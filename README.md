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
| **For Clients** *(The Website)* | A clean, luxury website to explore tattoo styles, view past work, and request an appointment. | • Easy booking form with phone verification.<br>• Automatic date checking (prevents booking closed days).<br>• Instant confirmation message upon submission. |
| **For Studio Staff** *(The Dashboard)* | An internal admin tool (like a digital planner) to view, organize, and manage appointments. | • View all incoming bookings in real-time.<br>• One-click confirmation, completion, or cancellation.<br>• Smart rescheduling: offers clients up to 3 alternative dates if the requested time is busy.<br>• Walk-in customer management & CSV list exporting. |
| **Behind the Scenes** *(The Automation Engine)* | An automated engine that processes bookings, organizes storage, and emails clients. | • **Zero Database Costs:** Runs on a self-hosted, lightweight background service.<br>• Sends automatic styled confirmation and rescheduling emails.<br>• High-grade security protection to prevent spam submissions. |

---

## Simple Feature Breakdown

* **Smart Calendar Rules:** The system automatically knows open hours, blocks past dates, and hides Sundays or fully booked days.
* **Reschedule Without Hassle:** If a client requests a busy time slot, staff can click one button to suggest 3 alternative open dates and send an automated email offer.
* **No Software Lock-in:** The entire system runs locally or on private studio hardware—no third-party software subscriptions required.
* **Offline Demo Mode:** The admin dashboard includes a built-in demo mode for testing and staff training without affecting real data.

---

## Quickstart & Launch Commands

### Prerequisites
* **Node.js 22+**
* **Docker**
* Free ports: `5678` (Automation Backend), `5173` (Website), `5174` (Admin Dashboard)

---

### Step 1: Start the Background Service (n8n)

Start the local automation engine:

```bash
docker run -d --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n -e N8N_HOST=localhost -e N8N_PORT=5678 -e N8N_PROTOCOL=http -e WEBHOOK_URL=http://localhost:5678/ -e N8N_SECURE_COOKIE=false docker.n8n.io/n8nio/n8n
```

1. Open `http://localhost:5678` in your browser.
2. Go to **Workflows** → **Import from File** → choose `n8n-automation/inkdesk-bookings.json`.
3. Click **Publish** (or toggle **Active**).

Verify connection:

```bash
curl -X POST http://localhost:5678/webhook/inkdesk-booking -H "Content-Type: text/plain;charset=utf-8" -d '{"action":"ping"}'
```

---

### Step 2: Launch the Client Website

Open website folder:
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

Open dashboard folder:
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
