# Inkheaven — Tattoo Studio Management System

A full-stack portfolio project: a luxury booking website for clients and an
internal dashboard for studio staff, both powered by a single self-hosted
automation workflow — no database, no subscription, no monthly fees.

```
[ Client submits form ] ──▶ [ Validated & stored ] ──▶ [ Studio confirms / reschedules ] ──▶ [ Email sent ]
```

---

## What I Built

| Piece | What it does |
| --- | --- |
| **`inkheaven/`** — the website | A single-page editorial site (styles, portfolio, process, testimonials, FAQ) ending in a booking form. |
| **`inkdesk/`** — the dashboard | An admin tool for viewing, filtering, confirming, rescheduling and exporting appointments. |
| **`n8n-automation/`** — the backend | One workflow that is both the public booking intake and the admin API, plus the customer emails. |

The two apps talk to the *same* webhook URL. The workflow tells them apart by
the payload: no `action` field = a website booking, an `action` field = an
admin API call. That one design decision is what let me ship a backend with no
server code and no database.

---

## Highlights

**For clients**
- Immersive one-page site with masked line reveals, a pinned horizontal process section, and smooth Lenis scrolling — all disabled under `prefers-reduced-motion`.
- Booking form with per-field validation, a fixed `+91` ten-digit phone input, a masked date field, and a hidden honeypot that silently swallows bots.
- Submitting shows an instant confirmation; nothing leaves the browser except the same-origin `POST /api/bookings`.

**For studio staff**
- Live appointment list with sortable columns, status chips, scope filters and debounced search across name, phone, email and ID.
- One-click confirm / complete / cancel, plus internal notes and a client-history view.
- **Reschedule offer**: pick up to 3 alternative slots, add a note, send it — the original slot stays untouched until the client accepts.
- Walk-in bookings, CSV export, keyboard shortcuts (`/`, `n`, `Esc`).
- **Demo mode**: with no endpoint configured the whole dashboard runs on local mock data, so it can be explored without touching anything real.

**Behind the scenes**
- Slot engine with a single source of truth: rejects unreadable dates, past days, closed days, blocked dates, out-of-hours times and double-bookings.
- Unavailable requests are never dropped — they're stored with a flag and the client is emailed the reason plus 3 real alternatives.
- Three styled transactional emails (confirmation / reschedule / unavailable), hand-built as inline-styled tables so they survive Gmail and Outlook.
- Proxy hardening: field allowlist, 16KB body cap, per-IP rate limiting, 8s upstream timeout, generic errors that never leak the upstream URL.

---

## Tech Stack

| Layer | Technology |
| --- | --- |
| UI | React 19, Vite 8, plain CSS with custom-property design tokens |
| Animation | GSAP 3 + `@gsap/react`, ScrollTrigger, Lenis |
| Fonts / images | Self-hosted Bodoni Moda + Manrope; WebP/AVIF generated with `sharp` |
| Backend | n8n (single Code-node workflow, webhook trigger) |
| Storage | n8n workflow static data — no database |
| Email | n8n Email Send node over SMTP |
| Lint | ESLint 10 flat config |

---

## Run the Demo

Everything below runs from the repository root. You'll need **Node.js 20.19+
or 22.12+** and **Docker**, with ports `5678`, `5173` and `5174` free.

### 1. Start n8n

```bash
docker run -d --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n -e N8N_HOST=localhost -e N8N_PORT=5678 -e N8N_PROTOCOL=http -e WEBHOOK_URL=http://localhost:5678/ -e N8N_SECURE_COOKIE=false docker.n8n.io/n8nio/n8n
```

Then in the n8n UI at `http://localhost:5678`:

1. **Workflows** → **Import from File** → `n8n-automation/inkdesk-bookings.json`
2. **Publish** it (or toggle **Active**) — an imported workflow is a draft, and
   a draft returns **404** on the production URL.
3. *Optional, only if you want the demo to send real email:* add an SMTP
   credential and select it on the `Email: Confirmation` and
   `Email: Reschedule` nodes. Both ship with a placeholder ID
   (`REPLACE_WITH_YOUR_SMTP_CREDENTIAL_ID`), so nothing sends until you do.

Check it's alive:

```bash
curl -X POST http://localhost:5678/webhook/inkdesk-booking -H "Content-Type: text/plain;charset=utf-8" -d '{"action":"ping"}'
```

### 2. Start the website

```bash
cd inkheaven
npm i
npm run dev
```

Open `http://localhost:5173`, click the **gear icon** (bottom-right) and paste:

```
http://localhost:5678/webhook/inkdesk-booking
```

### 3. Start the dashboard

In a new terminal, back at the repo root:

```bash
cd inkdesk
npm i
npm run dev
```

Open `http://localhost:5174`, go to **Settings** (gear icon), paste the same
URL, then **Test connection** → **Save**.

> Skip this step entirely to stay in **demo mode** — the dashboard will show
> the `DEMO` badge and run on local mock data.

### 4. See it work end to end

1. Submit a booking at `http://localhost:5173/#booking`
2. Refresh `http://localhost:5174` — it's there as `new` / `website`
3. Open it and hit **Confirm**

---

## Scripts

| Where | Script | What it does |
| --- | --- | --- |
| `inkheaven` | `npm run dev` | Dev server + booking API on `:5173` |
| `inkheaven` | `npm run build` / `lint` | Production build / ESLint |
| `inkheaven` | `npm run preview` | Serve `dist/` with CSP headers |
| `inkheaven` | `npm run preview:prod` | Dependency-free production server |
| `inkheaven` | `npm run audit` | Build + serve for Lighthouse on `:4173` |
| `inkdesk` | `npm run dev` / `build` / `preview` | Dev server, build, static preview |

---

## Notes

- Bookings live in n8n's workflow static data. Re-importing the workflow or
  recreating the container without the `n8n_data` volume wipes them — fine for
  a demo, which is exactly why the Docker command above includes the volume.
- The n8n webhook has no authentication, and the webhook URL the site sends is
  client-supplied. This is a local/portfolio setup, not something to expose to
  the public internet as-is.
- Full backend reference (every action, the email templates, slot logic):
  [`n8n-automation/README.md`](n8n-automation/README.md)
