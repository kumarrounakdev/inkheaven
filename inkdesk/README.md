# Inkdesk — Studio Admin Dashboard

The internal side of the Inkheaven system: one screen for every appointment
the studio has, with confirm / reschedule / cancel actions, client history and
CSV export. Talks directly to the same n8n webhook as the website.

```
[ Open :5174 ] ──▶ [ List & filter bookings ] ──▶ [ Confirm / Reschedule / Cancel ] ──▶ [ Client emailed ]
```

---

## What It Does

| Area | Feature |
| --- | --- |
| **Appointments** | Sortable list with status chips, scope filters (Upcoming / Today / Archive / All) and debounced search across name, phone, email and ID |
| **Dashboard** | Stat cards for pending requests, upcoming sessions, this week and today |
| **Actions** | One-click confirm, complete, cancel, delete + internal notes |
| **Rescheduling** | Offer up to 3 alternative slots with a note — the original slot stays put until the client accepts |
| **Client history** | Full appointment record for a phone number or email |
| **Walk-ins** | Manual booking modal that saves as `confirmed` |
| **Export** | CSV of the current list |
| **Shortcuts** | `/` search, `n` new booking, `Esc` close |
| **Demo mode** | No endpoint configured → runs entirely on local mock data |

---

## Tech Stack

| Layer | Technology |
| --- | --- |
| UI | React 19 + Vite 8 |
| Styling | Plain CSS design tokens (`src/index.css`) |
| Routing | Hash-based — `#/appt/<id>`, `#/client/<q>` (no router dependency) |
| Data | API client (`src/lib/api.js`) + local mock backend for demo mode |
| Fonts | Self-hosted Bodoni Moda + Manrope (variable `.woff2`) |

---

## Prerequisites

| Requirement | Version / Detail |
| --- | --- |
| **Node.js** | 20.19+ or 22.12+ |
| **npm** | Ships with Node |
| **Free port** | `5174` |
| **n8n backend** | Optional — demo mode works without it (see below) |

---

## Run It

From the repository root:

```bash
cd inkdesk
```

```bash
npm i
```

```bash
npm run dev
```

Open `http://localhost:5174`.

### Connect the backend (optional)

1. Gear icon → **Settings** → paste the webhook URL:
   ```
   http://localhost:5678/webhook/inkdesk-booking
   ```
2. **Test connection** (sends `{action:'ping'}`)
3. **Save**

With no endpoint saved the header shows a **DEMO** badge and the dashboard runs
on local mock data — useful for exploring the UI without touching real bookings.

> Start n8n first if you want a live backend: see
> [`../README.md`](../README.md) or [`../n8n-automation/README.md`](../n8n-automation/README.md).

---

## Scripts

| Script | What it does |
| --- | --- |
| `npm run dev` | Dev server on `:5174` |
| `npm run build` | Production build to `dist/` |
| `npm run preview` | Serve the built dashboard |

