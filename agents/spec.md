# Automation Workflows Specification (spec.md)

## Overview

This document defines the system architecture and workflow specifications for PropertyFollow automation systems. It covers both client-facing systems (villa/property automation) and internal agency automation.

---

# SECTION 1 — CLIENT-FACING SYSTEMS

## 1. Booking Engine Automation

### Objective

Convert incoming inquiries into qualified leads and confirmed bookings automatically.

### Triggers

* Incoming WhatsApp message (Webhook)
* Website form submission

### Core Workflow

1. Receive inquiry
2. Parse user intent:

   * Check-in date
   * Check-out date
   * Number of guests
3. Store lead in database (Google Sheets / Airtable)
4. Auto-response:

   * Greeting
   * Ask missing details
5. Availability check
6. Generate pricing
7. Send quotation
8. Tag lead status (new / qualified / hot)

### Integrations

* WhatsApp API
* n8n
* Google Sheets / Airtable

### Output

* Qualified lead
* Quotation sent

---

## 2. Booking + Operations System

### Objective

Manage bookings and synchronize operations across platforms.

### Triggers

* New booking (OTA / WhatsApp)
* Booking confirmation

### Core Workflow

1. Receive booking event
2. Update central calendar
3. Sync calendar across platforms
4. Block booked dates
5. Send confirmation to guest
6. Notify internal team:

   * Housekeeping
   * Operations

### Integrations

* OTA APIs (Airbnb, Booking platforms)
* Calendar system
* Internal notification (WhatsApp / Slack)

### Output

* Synced bookings
* Internal task notifications

---

## 3. Revenue & Guest Experience System

### Objective

Increase revenue and improve guest experience through automation.

### Triggers

* Booking confirmed
* Pre check-in timeline
* Check-out event

### Core Workflow

1. Booking confirmed → trigger guest journey
2. Pre check-in:

   * Send instructions
   * Send upsell offers
3. During stay:

   * Support messages
4. Check-out:

   * Thank you message
   * Request review
5. Post-stay:

   * Retarget guest for future stay

### Integrations

* WhatsApp API
* CRM / database

### Output

* Improved guest experience
* Increased upsell revenue
* More reviews

---

# SECTION 2 — INTERNAL AGENCY SYSTEMS

## 4. Sales Agent (Lead Qualification)

### Objective

Automatically qualify inbound leads via email.

### Triggers

* New email received

### Core Workflow

1. Parse email content
2. Extract:

   * Business type
   * Inquiry intent
3. Score lead quality
4. Auto-reply:

   * Ask qualification questions
5. Route lead:

   * High quality → sales pipeline
   * Low quality → nurture

### Integrations

* Email (IMAP / Gmail)
* CRM

### Output

* Qualified leads
* Segmented pipeline

---

## 5. Proposal Agent

### Objective

Generate tailored proposals based on client needs.

### Triggers

* Qualified lead in CRM

### Core Workflow

1. Retrieve lead data
2. Analyze requirements
3. Generate proposal draft:

   * Problem summary
   * Solution
   * Scope
   * Pricing estimate
4. Send draft for review or directly to client

### Integrations

* CRM
* Document generator

### Output

* Proposal document

---

## 6. Onboarding Agent

### Objective

Automate onboarding of new clients.

### Triggers

* Deal marked as closed

### Core Workflow

1. Send onboarding form
2. Collect required data:

   * Property details
   * Access credentials
3. Create project workspace
4. Assign internal tasks
5. Send onboarding confirmation

### Integrations

* Forms (Typeform / Google Forms)
* Project management tools

### Output

* Fully onboarded client

---

# SYSTEM ARCHITECTURE NOTES

## Core Engine

* n8n for orchestration

## Data Layer

* Google Sheets / Airtable (initial)
* Scalable to database (PostgreSQL)

## Communication Layer

* WhatsApp API
* Email

## Design Principles

* Modular workflows
* Reusable templates
* Event-driven architecture
* Clear separation between client systems and internal systems

---

# SECTION 3 — N8N NODE-BY-NODE IMPLEMENTATION

Below are production-ready workflow breakdowns for each system.

---

## 1. WhatsApp Booking Engine (n8n)

### Nodes Flow

1. **Webhook (Trigger)**

   * Method: POST
   * Source: WhatsApp API

2. **Set Node — Normalize Input**

   * Extract:

     * message
     * phone number

3. **Function Node — Parse Intent**

   * Extract:

     * check-in
     * check-out
     * guest count

4. **IF Node — Missing Info Check**

   * If missing → ask question
   * Else → continue

5. **Google Sheets / Airtable — Create/Update Lead**

   * Fields:

     * name
     * phone
     * dates
     * status

6. **HTTP Request — Availability Check**

   * Query calendar / database

7. **Function Node — Pricing Logic**

   * Calculate price based on:

     * nights
     * season

8. **Set Node — Format Message**

   * Build quotation text

9. **HTTP Request — Send WhatsApp Reply**

10. **Set Node — Update Lead Status**

* status = quoted

---

## 2. Lead Follow-up Automation

### Nodes Flow

1. **Cron Node (Scheduler)**

   * Run every 1 hour

2. **Google Sheets — Get Leads**

   * Filter:

     * status = quoted
     * no reply in 24h

3. **IF Node — Check Last Contact Time**

4. **Set Node — Generate Follow-up Message**

5. **HTTP Request — Send WhatsApp Message**

6. **Update Sheet — Mark Follow-up Sent**

---

## 3. Calendar Sync & Booking Handler

### Nodes Flow

1. **Webhook / API Trigger**

   * Source: OTA booking event

2. **Set Node — Normalize Booking Data**

3. **Database Node — Save Booking**

4. **HTTP Request — Update Other Calendars**

   * Push blocked dates

5. **IF Node — Conflict Check**

   * If conflict → alert admin

6. **HTTP Request — Send Confirmation to Guest**

7. **HTTP Request — Notify Internal Team**

---

## 4. Guest Journey Automation

### Nodes Flow

1. **Trigger — Booking Confirmed**

2. **Wait Node — Until H-1 Check-in**

3. **HTTP Request — Send Pre Check-in Info**

4. **Wait Node — During Stay Trigger**

5. **HTTP Request — Send Support Message**

6. **Wait Node — Check-out Date**

7. **HTTP Request — Send Thank You + Review Request**

8. **Set Node — Tag Guest for Retargeting**

---

## 5. Sales Agent (Email Qualification)

### Nodes Flow

1. **IMAP Email Trigger**

2. **Function Node — Parse Email Content**

3. **OpenAI Node — Classify Lead Quality**

4. **IF Node — High vs Low Quality**

5. **HTTP Node — Send Reply Email**

6. **CRM Node — Save Lead**

---

## 6. Proposal Agent

### Nodes Flow

1. **Trigger — New Qualified Lead (CRM)**

2. **Database Node — Fetch Lead Data**

3. **OpenAI Node — Generate Proposal**

4. **HTML / PDF Generator Node**

5. **Email Node — Send Proposal**

---

## 7. Onboarding Agent

### Nodes Flow

1. **Trigger — Deal Closed**

2. **Email Node — Send Onboarding Form**

3. **Webhook — Receive Form Submission**

4. **Set Node — Structure Data**

5. **Database Node — Create Client Record**

6. **Project Tool Node — Create Workspace**

7. **Slack / WhatsApp Node — Notify Team**

---

# IMPLEMENTATION NOTES

* Use environment variables for API keys
* Separate workflows per module for maintainability
* Use error workflows for failure handling
* Log all events for debugging

---

# SECTION 4 — SYSTEM ARCHITECTURE & DATA FLOW

## High-Level Architecture

```
                ┌────────────────────┐
                │   Client Channels   │
                │────────────────────│
                │ WhatsApp           │
                │ Website Forms      │
                │ OTA Platforms      │
                │ (Airbnb, Booking)  │
                └─────────┬──────────┘
                          │
                          ▼
                ┌────────────────────┐
                │   API Layer        │
                │────────────────────│
                │ WhatsApp API       │
                │ Webhooks           │
                │ OTA Integrations   │
                └─────────┬──────────┘
                          │
                          ▼
                ┌────────────────────┐
                │   Automation Core  │
                │       (n8n)        │
                │────────────────────│
                │ Booking Engine     │
                │ Follow-up System   │
                │ Ops Automation     │
                │ Guest Journey      │
                │ Internal Agents    │
                └─────────┬──────────┘
                          │
          ┌───────────────┼────────────────┐
          ▼               ▼                ▼
┌────────────────┐ ┌───────────────┐ ┌────────────────┐
│   Data Layer    │ │ Communication │ │ External Tools │
│────────────────│ │───────────────│ │────────────────│
│ Airtable       │ │ WhatsApp Send │ │ Payment Links  │
│ Google Sheets  │ │ Email         │ │ Calendar APIs  │
│ PostgreSQL(*)  │ │ Notifications │ │ CRM (optional) │
└────────────────┘ └───────────────┘ └────────────────┘

(*) scalable phase
```

---

## Data Flow — Booking Engine

```
User (WhatsApp)
   │
   ▼
Webhook (n8n)
   │
   ▼
Parse Message (Function Node)
   │
   ▼
Check Missing Info (IF)
   │
   ├── Missing → Ask Question → WhatsApp API → User
   │
   └── Complete
         │
         ▼
   Save Lead (DB)
         │
         ▼
   Check Availability (API/DB)
         │
         ▼
   Generate Pricing (Function)
         │
         ▼
   Send Quotation (WhatsApp API)
         │
         ▼
   Update Status (DB)
```

---

## Data Flow — Follow-up System

```
Cron Trigger
   │
   ▼
Fetch Leads (DB)
   │
   ▼
Filter شروط (no reply)
   │
   ▼
Generate Message
   │
   ▼
Send WhatsApp
   │
   ▼
Update Follow-up Status
```

---

## Data Flow — Booking & Operations

```
Booking Event (OTA / WA)
   │
   ▼
Webhook (n8n)
   │
   ▼
Normalize Data
   │
   ▼
Save Booking (DB)
   │
   ▼
Sync Calendar (API calls)
   │
   ▼
Conflict Check
   │
   ├── Conflict → Alert Admin
   │
   └── No Conflict
         │
         ▼
   Send Confirmation (Guest)
         │
         ▼
   Notify Internal Team
```

---

## Data Flow — Guest Journey

```
Booking Confirmed
   │
   ▼
Wait (H-1)
   │
   ▼
Send Check-in Info
   │
   ▼
Wait (During Stay)
   │
   ▼
Send Upsell / Support
   │
   ▼
Wait (Checkout)
   │
   ▼
Send Review Request
   │
   ▼
Store Guest Data (Retargeting)
```

---

## Data Flow — Internal Systems

### Sales Agent

```
Incoming Email → Parse → AI Classify → CRM → Auto Reply
```

### Proposal Agent

```
CRM Lead → Fetch Data → Generate Proposal → Send Email
```

### Onboarding Agent

```
Deal Closed → Send Form → Receive Data → Create Workspace → Notify Team
```

---

## System Design Principles

* Event-driven (Webhook + Trigger based)
* Stateless workflows (data stored in DB)
* Modular (each system isolated)
* Fail-safe (error handling workflows)
* Scalable (DB upgrade path)

---

# END OF SPEC
