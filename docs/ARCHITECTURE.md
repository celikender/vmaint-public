# Architecture

vMaint is a single Next.js application that runs the maintenance and production
floor for small manufacturers. This is an overview of the engineering decisions
behind it. It describes the design, not the proprietary source.

## Stack

- **Next.js 16** (App Router) and **React 19**, one codebase for the marketing
  site, the auth flow, and the role app.
- **Tailwind CSS v4** with **shadcn/ui** components.
- **Drizzle ORM** over **PostgreSQL** (Neon), with Postgres row-level security
  for tenant isolation.
- **NextAuth v5** (Google, Microsoft, and email/password) with JWT sessions.
- **Stripe** for subscription billing, **Resend** for transactional email.
- A **FastAPI** service for the AI layer.
- **Vercel** for hosting, with a Neon database branch per preview deploy.

## Tenant isolation at the database, not the application

Every tenant-scoped table (assets, work orders, parts, labels) has Postgres
row-level security enabled, with policies that filter on a per-request setting.
The application cannot read another tenant's rows even if a query forgets to
filter, because the database refuses to return them.

All tenant queries pass through a single choke point that opens a transaction,
sets the tenant id for that transaction only, and runs the query inside it:

```ts
// Outside withTenant(), RLS returns zero rows. That is the safety guarantee.
const list = await withTenant(org.id, (tx) => tx.select().from(assets));
```

The runtime database role does not bypass RLS; a separate admin role is used
only for migrations and the billing webhook. Multi-tenancy is therefore a
property of the database, not a discipline the application has to remember on
every query.

## One app, four roles

Operators, technicians, ops leads, and managers all run the same application,
gated by capability flags rather than separate builds. A manager working the
floor on a phone gets the technician shell; the full management dashboard is the
wide desktop view. This keeps one code path instead of four, and lets a role's
surface be hidden in the UI without starving it of data.

## Mobile-first by default

The floor runs on phones, so the role app is a fixed-height, installable PWA with
its own swipe-tab navigation, tuned for fast input (photo to work order, one tap
to set machine status) instead of dense desktop forms. The same app installs to a
home screen and broadcasts to a shop-floor TV in kiosk mode.

## A no-login demo, isolated from production

The "see it live" demo is the real app running against an in-memory seed, served
from an isolated deployment that never touches production data or credentials. It
is ephemeral (nothing is saved), blocked from search crawlers, and seeded with a
single consistent sample plant so every role and screen looks real. The marketing
site embeds it so a visitor can try the product without an account.

## The AI layer

Because maintenance, production, vendor work, and machine data live in one
record, the AI layer can reason over the actual shop instead of guessing. It
reads equipment manuals, fault history, and machine readings, answers error
codes and troubleshooting questions, and, with a confirmation step, can act, such
as opening a work order or logging a part used. It runs as its own service and
can also be reached through a standard AI assistant.

## Real-time floor

Work-order updates, andon calls, and safety alerts propagate over WebSockets, so
every role and the shop-floor TV see the floor change live rather than on a
refresh.

## Deploys

Hosting is on Vercel. Each pull request gets its own preview deploy backed by a
Neon database branch, so a change can be reviewed against an isolated copy of the
schema before it reaches production. The marketing site, the comparison and
content pages, and an `llms.txt` / `llms-full.txt` pair are statically rendered
for fast loads and clean retrieval by search and answer engines.
