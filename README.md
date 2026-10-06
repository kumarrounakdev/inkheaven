# Inkheaven — Tattoo Studio Management System

Inkheaven is a complete digital setup for a tattoo studio. It provides a modern booking website for clients and a clean management dashboard for studio staff, with zero monthly database or subscription fees.

---

## What Does This Project Do?

This system automates the entire process of booking and managing tattoo appointments—from a client filling out a form on their phone to the artist confirming their spot.

```
[ Customer Submits Form ] ──▶ [ Automated Checks & Storage ] ──▶ [ Studio Accepts / Changes Slot ] ──▶ [ Email Sent ]
```

---

## How It Helps the Studio

| Role | What They See & Do | Key Capabilities |
| --- | --- | --- |
| **For Clients** *(The Website)* | A clean, luxury website to explore tattoo styles, view past work, and request an appointment. | • Easy booking form with a validated phone number.<br>• Instant confirmation message upon submission.<br>• Option to pick a preferred session time. |
| **For Studio Staff** *(The Dashboard)* | An internal admin tool (like a digital planner) to view, organize, and manage appointments. | • View all incoming bookings in real-time.<br>• One-click confirmation, completion, or cancellation.<br>• Smart rescheduling: offers clients up to 3 alternative slots if the requested time is busy.<br>• Walk-in customer management & CSV list exporting. |
| **Behind the Scenes** *(The Automation Engine)* | An automated engine that processes bookings, organizes storage, and emails clients. | • **Zero Database Costs:** Runs on a self-hosted, lightweight background service.<br>• Sends automatic styled confirmation and rescheduling emails.<br>• Built-in spam protection (honeypot field + per-IP rate limiting). |

---

## Simple Feature Breakdown

* **Reschedule Without Hassle:** If a client requests a busy time slot, staff can click one button to suggest 3 alternative open slots and send an automated email offer.
* **No Software Lock-in:** The entire system runs locally or on private studio hardware—no third-party software subscriptions required.
* **Offline Demo Mode:** The admin dashboard includes a built-in demo mode for testing and staff training without affecting real data.

---

## Quickstart & Launch Commands

Run every command below from the **repository root**.

### Prerequisites
* **Node.js 20.19+ or 22.12+** (Vite 8 requires `"node": "^20.19.0 || >=22.12.0"`)
* **Docker**
* Free ports: `5678` (Automation Backend), `5173` (Website), `5174` (Admin Dashboard)

---

### Step 1: Start the Background Service (n8n)

Start the local automation engine:

```bash
docker run -d --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n -e N8N_HOST=localhost -e N8N_PORT=5678 -e N8N_PROTOCOL=http -e WEBHOOK_URL=http://localhost:5678/ -e N8N_SECURE_COOKIE=false docker.n8n.io/n8nio/n8n
```

The `-v n8n_data:/home/node/.n8n` volume is what makes bookings survive a container recreation.

1. Open `http://localhost:5678` in your browser.
2. Go to **Workflows** → **Import from File** → choose `n8n-automation/inkdesk-bookings.json`.
3. Click **Publish** (or toggle **Active**). An imported workflow is a draft — until it is published, the production URL returns **404**.
4. *(Optional, enables email)* Add an **SMTP** credential under **Credentials**, then select it on both the `Email: Confirmation` and `Email: Reschedule` nodes. Both ship with the placeholder `REPLACE_WITH_YOUR_SMTP_CREDENTIAL_ID`, so nothing sends until you replace it. Bookings work fine without this.

Verify connection:

```bash
curl -X POST http://localhost:5678/webhook/inkdesk-booking -H "Content-Type: text/plain;charset=utf-8" -d '{"action":"ping"}'
```

Full workflow documentation: [`n8n-automation/README.md`](n8n-automation/README.md)

---

### Step 2: Launch the Client Website

Open website folder (from the repo root):
```bash
cd inkheaven
```

Install components:
```bash
npm i
```

Run website:
```bash
npm run dev
```

* Open `http://localhost:5173`, click the gear icon (bottom-right), and enter:  
  `http://localhost:5678/webhook/inkdesk-booking`

---

### Step 3: Launch the Admin Dashboard

Open dashboard folder (from the repo root, or `cd ..` first):
```bash
cd inkdesk
```

Install components:
```bash
npm i
```

Run dashboard:
```bash
npm run dev
```

* Open `http://localhost:5174`, open **Settings** (gear icon), and enter:  
  `http://localhost:5678/webhook/inkdesk-booking`
* Click **Test connection**, then **Save**. Without an endpoint the header shows **DEMO** and the dashboard runs on local mock data.
