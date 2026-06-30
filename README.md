# vMaint

**Maintenance and production in one app, for small and mid-sized manufacturers.**

[Live site](https://vmaint.com) &nbsp;·&nbsp; [Contact](https://vmaint.com/contact)

vMaint runs the maintenance and production sides of a shop floor from one place:
work orders, preventive maintenance, spare-parts inventory, live downtime
tracking, and machine performance (OEE), with an AI layer that troubleshoots the
faults that keep coming back. It does the work a small plant would otherwise
split across a CMMS, an MES, and a stack of spreadsheets, priced flat per site
with the whole team included.

Built for shops that have outgrown spreadsheets but do not need a heavy
enterprise system. Mobile-first, live the same day, no long rollout.

> This repository is a public showcase of the vMaint project. The product itself
> is proprietary and runs at **[vmaint.com](https://vmaint.com)**. This repo
> exists to document what vMaint is, how it is built, and who builds it.

## Try it, no signup

The homepage runs the actual product live in the browser. Switch between four
roles (operator, technician, ops lead, manager) and poke around a sample molding
plant. Nothing you do is saved.

**[vmaint.com](https://vmaint.com)**

## What it does

- **Work orders and preventive maintenance** — report a problem from the machine
  with a photo and a work order opens itself; recurring PMs schedule themselves.
- **Spare-parts inventory** — barcode and bin scanning from a phone, with
  low-stock alerts and reorder tracking.
- **Live OEE and downtime** — operators tap running, idle, or down; production
  tracks against target; OEE is computed per machine and per shift, with no MES
  and no hardware to start.
- **AiTech** — an AI layer that reads your manuals, fault history, and machine
  data to answer error codes, troubleshoot recurring faults, and open work
  orders by conversation.
- **One floor, four roles** — operators, technicians, ops leads, and managers
  each get their own view, sharing a real-time team board, andon alerts, and a
  shop-floor TV kiosk.
- **Connect your machines** — optional SCADA, PLC, and IoT integration set up per
  shop, with an on-prem and edge option that keeps machine data on site.

## How it is built

- **App** — **Next.js 16** (App Router) and **React 19**, with **Tailwind CSS v4**
  and **shadcn/ui**.
- **Data** — **Drizzle ORM** over **PostgreSQL** (Neon), with **row-level
  security** for hard multi-tenant isolation enforced by the database, not the
  application.
- **Auth, billing, email** — **NextAuth v5** (Google, Microsoft, email and
  password), **Stripe** for subscription billing, **Resend** for transactional
  email.
- **AI service** — a separate **Python service (FastAPI)** runs the AI layer,
  deployed independently from the web app so the heavier reasoning work scales on
  its own.
- **Real-time** — live updates over **WebSockets**, so andon calls, work-order
  changes, and floor-wide safety alerts reach every role and the shop-floor TV the
  instant they happen.
- **Hosting** — **Vercel**, with a Neon database branch per preview deploy.

A deeper engineering write-up is in **[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)**.

## Who builds it

vMaint is built by **Ender Celik**, a mechanical engineer who has worked every
side of the operation: machine operator, maintenance technician, and automation
and SCADA engineer.

On each side he saw the same gap, that engineering strategy and the day-to-day
reality of the floor rarely line up, so operations, maintenance, and engineering
end up working around each other. vMaint exists to put everyone on one shared
picture instead. It is built by someone who has stood in front of a faulted line
at 2 a.m., not by people who have never run a machine.

- Live product: **[vmaint.com](https://vmaint.com)**
- Get in touch: **[vmaint.com/contact](https://vmaint.com/contact)**

## Status

Early-stage and in production. The live app and the no-login demo above are real.

## License

The vMaint name, brand, product, and source code are proprietary. See
[LICENSE](LICENSE).
