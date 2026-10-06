# Inkheaven — Booking Website

The public side of the Inkheaven system: a single-page studio site that ends
in a booking form. Submissions go through a same-origin `/api/bookings` proxy
straight to the n8n webhook.

```
[ Client fills the form ] ──▶ [ Validated by the proxy ] ──▶ [ Stored in n8n ] ──▶ [ Confirmation email ]
```

---

## What It Does

| Area | Feature |
| --- | --- |
| **Site** | Hero, portfolio gallery, tattoo styles, artist bio, process, testimonials, FAQ and booking form — one page, no router |
| **Navigation** | Floating dock with magnification on desktop, compact bottom nav on mobile, active section via `IntersectionObserver` |
| **Animation** | GSAP masked line reveals, pinned horizontal process section, Lenis smooth scroll — all disabled under `prefers-reduced-motion` |
| **Booking form** | Per-field validation, fixed `+91` ten-digit phone, masked date field, hidden honeypot for bots |
| **Proxy** | Field allowlist, 16KB body cap, per-IP rate limit (5 / 10 min), 8s upstream timeout, generic errors |
| **Settings panel** | Gear icon (bottom-right) sets the webhook URL per browser — no `.env` edit needed |
| **Images** | Responsive WebP/AVIF via `sharp`, lazy below the fold, hero LCP eager |
| **Performance** | Lazy below-the-fold sections, split vendor chunks, CLS-safe placeholders |

---

## Tech Stack

| Layer | Technology |
| --- | --- |
| UI | React 19 + Vite 8 |
| Styling | Plain CSS design tokens (`src/index.css`) |
| Animation | GSAP 3, ScrollTrigger, Lenis |
| Images | WebP / AVIF via `sharp` |
| Server | `scripts/serve.mjs` + `scripts/bookingProxy.mjs` |
| Fonts | Self-hosted Bodoni Moda + Manrope (variable `.woff2`) |

---

## Prerequisites

| Requirement | Version / Detail |
| --- | --- |
| **Node.js** | 20.19+ or 22.12+ |
| **npm** | Ships with Node |
| **Free port** | `5173` |
| **n8n backend** | Required for real bookings (see below) |

---

## Run It

From the repository root:

```bash
cd inkheaven
```

```bash
npm i
```

```bash
npm run dev
```

Open `http://localhost:5173`.

### Connect the backend

1. Gear icon (bottom-right) → paste the webhook URL:
   ```
   http://localhost:5678/webhook/inkdesk-booking
   ```
2. **Save**

The URL is stored in that browser and sent with every booking, so no `.env`
edit is required.

<details>
<summary>Optional: server-side fallback URL</summary>

```bash
cp .env.example .env
```

Then uncomment `BOOKING_WEBHOOK_URL` in `.env`. It is only used when the
browser sends nothing. Never prefix a secret with `VITE_` — Vite inlines those
into the client bundle.

</details>

> Start n8n first: see [`../README.md`](../README.md) or
> [`../n8n-automation/README.md`](../n8n-automation/README.md).

---

## Scripts

| Script | What it does |
| --- | --- |
| `npm run dev` | Dev server + booking API on `:5173` |
| `npm run build` | Production build to `dist/` |
| `npm run lint` | ESLint |
| `npm run preview` | Serve `dist/` with CSP headers |
| `npm run preview:prod` | Dependency-free production server |
| `npm run audit` | Build + serve — Lighthouse target on `:4173` |
