# Inkheaven — Tattoo Studio Management System

A portfolio project: a booking website for clients and a dashboard for studio
staff, both powered by a single self-hosted n8n workflow — no database, no
subscription.

```
[ Client submits form ] ──▶ [ Validated & stored ] ──▶ [ Studio confirms / reschedules ] ──▶ [ Email sent ]
```

---

## What I Built

| Piece | What it does |
| --- | --- |
| `inkheaven/` | Single-page site with styles, portfolio, process, testimonials, FAQ and a booking form |
| `inkdesk/` | Admin dashboard — list, filter, confirm, reschedule, export appointments |
| `n8n-automation/` | One workflow serving as both the booking intake and the admin API, plus customer emails |

Both apps hit the same webhook URL. The workflow tells them apart by payload:
no `action` field = website booking, `action` field = admin API call.
Full backend reference: [`n8n-automation/README.md`](n8n-automation/README.md)

---

## Highlights

- **Website:** GSAP masked line reveals, pinned horizontal process section, Lenis smooth scroll — all disabled under `prefers-reduced-motion`.
- **Booking form:** per-field validation, fixed `+91` ten-digit phone, masked date field, hidden honeypot for bots.
- **Dashboard:** sortable list, status chips, scope filters, search, one-click confirm/complete/cancel, internal notes, client history, CSV export, keyboard shortcuts (`/`, `n`, `Esc`).
- **Reschedule offer:** pick up to 3 alternative slots and email the client — the original slot stays put until they accept.
- **Demo mode:** no endpoint configured → the whole dashboard runs on local mock data.
- **Slot engine:** rejects unreadable dates, past days, closed days, blocked dates, out-of-hours times and double-bookings. Rejected requests are still stored and the client is emailed the reason plus 3 alternatives.
- **Hardened proxy:** field allowlist, 16KB body cap, per-IP rate limiting, 8s upstream timeout, generic errors.
- **Emails:** three styled templates built as inline-styled tables so they survive Gmail and Outlook.

---

## Tech Stack

### `inkheaven/` — Website

| Layer | Technology |
| --- | --- |
| UI | React 19 + Vite 8 |
| Styling | Plain CSS design tokens |
| Animation | GSAP 3, ScrollTrigger, Lenis |
| Images | WebP / AVIF via `sharp` |

### `inkdesk/` — Dashboard

| Layer | Technology |
| --- | --- |
| UI | React 19 + Vite 8 |
| Styling | Plain CSS design tokens |
| Routing | Hash-based (no router dependency) |
| Data | API client + local mock backend |

### `n8n-automation/` — Backend

| Layer | Technology |
| --- | --- |
| Runtime | n8n (Docker) |
| Logic | Single Code node — routing, slot engine, validation |
| Storage | Workflow static data — no database |
| Email | SMTP via the Email Send node |

---

## Prerequisites

| Requirement | Version / Detail |
| --- | --- |
| **Node.js** | 20.19+ or 22.12+ |
| **npm** | Ships with Node |
| **Docker** | For the n8n backend |
| **Free ports** | `5678` (n8n), `5173` (website), `5174` (dashboard) |

---

## Run the Demo

Run everything from the repository root.

### 1. Start n8n

```bash
docker run -d --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n -e N8N_HOST=localhost -e N8N_PORT=5678 -e N8N_PROTOCOL=http -e WEBHOOK_URL=http://localhost:5678/ -e N8N_SECURE_COOKIE=false docker.n8n.io/n8nio/n8n
```

In the n8n UI at `http://localhost:5678`:

1. **Workflows** → **Import from File** → `n8n-automation/inkdesk-bookings.json`
2. **Publish** it (or toggle **Active**) — a draft returns `404` on the production URL.
3. *Optional:* add an SMTP credential and select it on `Email: Confirmation` and `Email: Reschedule` (both ship with a placeholder ID). Without it, bookings work but no email sends.

```bash
curl -X POST http://localhost:5678/webhook/inkdesk-booking -H "Content-Type: text/plain;charset=utf-8" -d '{"action":"ping"}'
```

### 2. Start the website

```bash
cd inkheaven
```

```bash
npm i
```

```bash
npm run dev
```

Open `http://localhost:5173`, click the gear icon (bottom-right), paste:

```
http://localhost:5678/webhook/inkdesk-booking
```

### 3. Start the dashboard

New terminal, back at the repo root:

```bash
cd inkdesk
```

```bash
npm i
```

```bash
npm run dev
```

Open `http://localhost:5174` → **Settings** → paste the same URL → **Test connection** → **Save**.
Skip this to stay in **demo mode** (shows a `DEMO` badge).

### 4. End-to-end check

1. Submit a booking at `http://localhost:5173/#booking`
2. Refresh `http://localhost:5174` — it appears as `new` / `website`
3. Open it → **Confirm**

---

## Scripts

| Where | Script | What it does |
| --- | --- | --- |
| `inkheaven` | `npm run dev` | Dev server + booking API on `:5173` |
| `inkheaven` | `npm run build` / `lint` | Production build / ESLint |
| `inkheaven` | `npm run preview:prod` | Dependency-free production server |
| `inkheaven` | `npm run audit` | Lighthouse target on `:4173` |
| `inkdesk` | `npm run dev` / `build` / `preview` | Dev, build, static preview |

