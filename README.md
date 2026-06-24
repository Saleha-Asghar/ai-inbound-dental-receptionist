# 🦷 Maya — AI Inbound Receptionist for Dental Clinics

An AI-powered inbound voice receptionist that answers patient calls, books appointments through natural conversation, creates Google Calendar events automatically, and logs every call to Google Sheets — with zero human involvement.

Built with VAPI, n8n, Google Calendar, and Google Sheets.

---

## 📽️ Demo

> [Insert your demo video link here — YouTube or LinkedIn]

---

## 🏗️ System Architecture

```
Patient calls the clinic number
          ↓
Maya (AI Voice Receptionist) answers instantly via VAPI
          ↓
Conversation:
  → New or existing patient?
  → Reason for visit
  → Full name + contact
  → Preferred date & time
  → Confirms appointment details
          ↓
Call ends → VAPI fires end-of-call-report webhook
          ↓
n8n catches webhook → extracts patient details from transcript
          ↓
Gemini AI parses natural language date/time
→ "This Friday at 10am" → 2026-06-27T10:00:00
          ↓
If appointment booked:
  → Google Calendar event created automatically
  → Row logged to Google Sheets
If not booked (cancelled/incomplete):
  → Row logged to Google Sheets only
```

---

## ✨ Features

- **Instant call answering** — Maya picks up every call, zero wait time, 24/7
- **Natural conversation** — handles new patients, existing patients, cancellations, reschedules, and emergency triage
- **Smart patient detection** — auto-detects if caller is new or existing from conversation
- **Reason for visit classification** — checkup, whitening, root canal, extraction, emergency, etc.
- **AI date/time parsing** — Gemini converts natural speech ("sometime Friday morning") to exact ISO datetime
- **Google Calendar integration** — creates real appointment events with patient details and full transcript in description
- **Dual-path routing** — booked appointments → Calendar + Sheets, incomplete calls → Sheets only
- **Emergency detection** — flags emergency cases separately for priority handling
- **Null-safe extraction** — handles missing transcripts, failed recordings, and incomplete calls gracefully

---

## 🛠️ Tech Stack

| Tool | Purpose |
|---|---|
| VAPI | AI voice infrastructure — answers inbound calls |
| n8n | Workflow orchestration + webhook handling |
| Gemini 2.5 Flash | Natural language date/time parsing |
| Google Calendar | Appointment event creation |
| Google Sheets | Call log + appointment tracker |
| Twilio | Phone number (inbound) |
| Cloudflare Tunnel | Expose local n8n webhook publicly |

---

## 📁 Repository Structure

```
maya-ai-receptionist/
│
├── README.md
│
├── n8n/
│   └── maya_appointment_logger.json     # Complete n8n workflow
│
├── vapi/
│   └── maya_system_prompt.md            # Full Maya system prompt
│
└── docs/
    ├── architecture.png                 # n8n workflow screenshot
    └── sheet_sample.png                 # Sample Google Sheet output
```

---

## 🚀 Setup Guide

### Prerequisites
- [VAPI account](https://vapi.ai) — free $10 credit
- [n8n](https://n8n.io) — self-hosted (Docker) or cloud
- [Twilio account](https://twilio.com) — for inbound phone number
- Google account — for Calendar + Sheets
- [Gemini API key](https://aistudio.google.com/apikey) — free tier
- [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps) — free, no account needed

---

### Step 1 — Create Maya in VAPI

1. Log in to [vapi.ai](https://vapi.ai) → Assistants → Create Assistant → Blank Template
2. Name it **"Maya"**
3. Set **First Message**:
   ```
   Thank you for calling Bright Smile Dental Clinic.
   This is Maya, your scheduling assistant.
   How can I help you today?
   ```
4. Paste contents of `vapi/maya_system_prompt.md` into System Prompt
5. Go to **Advanced** tab → **Server Messages** → enable only:
   - ✅ `end-of-call-report`
   - ✅ `hang`
6. Set **Server URL** to your Cloudflare tunnel URL (see Step 4):
   ```
   https://YOUR-TUNNEL-URL.trycloudflare.com/webhook/maya-callback
   ```
7. Connect your Twilio number under **Phone Numbers**

---

### Step 2 — Set Up Google Sheet

Create a new Google Sheet called **"Maya - Appointments"** with these exact headers in Row 1:

```
Timestamp | Patient Name | Patient Type | Reason for Visit | 
Appointment DateTime | Booking Status | Call Duration | Full Transcript
```

---

### Step 3 — Set Up Google Calendar

1. Go to [calendar.google.com](https://calendar.google.com)
2. Create a new calendar named **"Bright Smile Dental Clinic"**
3. Go to its Settings → copy the **Calendar ID**
   (format: `xxxxxxxx@group.calendar.google.com`)

---

### Step 4 — Import n8n Workflow

1. In n8n → New Workflow → Import from File
2. Upload `n8n/maya_appointment_logger.json`
3. Fill in these placeholders inside the workflow:

**In "Code in JavaScript" node** — find and replace:
```
PASTE-YOUR-GEMINI-API-KEY-HERE → your actual Gemini API key
```

**In "Create Calendar Event" node:**
```
PASTE-YOUR-CALENDAR-ID-HERE → your Google Calendar ID
```

**In "Log to Google Sheets" node:**
```
PASTE-YOUR-GOOGLE-SHEET-URL-HERE → your Sheet URL
```

4. Connect credentials:
   - Google Calendar → Create new → sign in with Google
   - Google Sheets → Create new → same Google account
5. Toggle workflow to **Active**

---

### Step 5 — Expose Webhook with Cloudflare Tunnel

Open a terminal and run:
```bash
cloudflared tunnel --url http://localhost:5678
```

Copy the generated URL and paste into VAPI → Advanced → Server URL:
```
https://random-words.trycloudflare.com/webhook/maya-callback
```

Keep this terminal open while the system is running.

---

### Step 6 — Test

Call your Twilio number (or use VAPI's browser Talk button) and book a test appointment. After the call ends, verify:

- ✅ n8n workflow executed green (all nodes)
- ✅ Google Calendar shows new appointment event
- ✅ Google Sheet shows new row with patient details

---

## 📊 Google Sheet Output Example

| Timestamp | Patient Name | Patient Type | Reason for Visit | Appointment DateTime | Booking Status | Call Duration | Full Transcript |
|---|---|---|---|---|---|---|---|
| 24/06/2026, 9:36 am | Ahmed | New Patient | General Checkup & Cleaning | Friday 27 June at 10:00 AM | Booked | 2m 15s | Maya: Thank you for... |
| 24/06/2026, 10:12 am | Sarah | Existing Patient | Root Canal | Tomorrow at 2:00 PM | Booked | 3m 40s | Maya: Thank you for... |
| 24/06/2026, 11:05 am | Unknown | Unknown | General Checkup | Tomorrow at 10:00 AM | Incomplete | N/A | No transcript available |

---

## 📅 Google Calendar Event Example

```
Title:    General Checkup & Cleaning — New Patient
Date:     Friday, June 27, 2026
Time:     10:00 AM – 10:30 AM
Calendar: Bright Smile Dental Clinic

Description:
Patient Type: New Patient
Reason: General Checkup & Cleaning
Booked via: Maya AI Receptionist

Full Transcript:
Maya: Thank you for calling...
```

---

## 🔧 Customization

**Change the clinic name:**
Update Maya's system prompt and first message with your client's clinic name.

**Add more appointment types:**
In the "Extract Appointment Data" code node, add keywords to the `reasonForVisit` section.

**Change appointment duration:**
In the "Code in JavaScript" node, find `end.setMinutes(end.getMinutes() + 30)` and change 30 to your preferred duration in minutes.

**Add SMS confirmation:**
After the Google Sheets node, add a Twilio SMS node to send the patient a confirmation text with their appointment details.

---

## 💰 Cost Breakdown

| Item | Cost |
|---|---|
| VAPI free credit | $0 (included) |
| Browser call demo | $0 |
| Gemini API (free tier) | $0 |
| Google Calendar | $0 |
| Google Sheets | $0 |
| n8n self-hosted | $0 |
| Cloudflare Tunnel | $0 |
| **Total for demo** | **$0** |

Production: ~$0.10–0.15/min per call via VAPI + Twilio

---

## 🔒 Security Notes

- Never commit your Gemini API key to GitHub — use environment variables in production
- The workflow file uses placeholder values — fill them in after importing to n8n
- Cloudflare tunnel URL changes on each restart — update VAPI Server URL accordingly

---

## 📬 Contact

Built by **Saleha** — AI automation & voice agent specialist

- LinkedIn: [your LinkedIn URL]
- Fiverr: [your Fiverr gig URL]
- Email: your.email@example.com

---

## 📄 License

MIT — free to use and modify for personal and commercial projects.
