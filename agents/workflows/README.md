# PropertyFollow — n8n Workflow Files

Folder ini berisi 7 workflow n8n yang siap di-import, sesuai dengan `spec.md`.

---

## Daftar Workflow

| File | Nama | Tipe | Trigger |
|------|------|------|---------|
| `01_whatsapp_booking_engine.json` | WhatsApp Booking Engine | Client-facing | Webhook (POST dari WhatsApp API) |
| `02_lead_followup_automation.json` | Lead Follow-up Automation | Client-facing | Schedule (setiap 1 jam) |
| `03_calendar_sync_booking_handler.json` | Calendar Sync & Booking Handler | Client-facing | Webhook (POST dari OTA / WA) |
| `04_guest_journey_automation.json` | Guest Journey Automation | Client-facing | Webhook (dipanggil saat booking confirmed) |
| `05_sales_agent_email_qualification.json` | Sales Agent (Email Qualification) | Internal | IMAP Email Trigger |
| `06_proposal_agent.json` | Proposal Agent | Internal | Webhook (qualified lead) |
| `07_onboarding_agent.json` | Onboarding Agent | Internal | Webhook (deal closed) |

---

## Cara Import ke n8n

1. Buka n8n → **Workflows** → **+ New**
2. Klik tombol `⋮` (menu) → **Import from File**
3. Pilih file `.json` dari folder ini
4. Klik **Save**

> Ulangi untuk setiap file.

---

## Environment Variables yang Dibutuhkan

Set semua variabel berikut di n8n: **Settings → Variables**

| Variable | Deskripsi | Contoh |
|----------|-----------|--------|
| `WHATSAPP_PHONE_NUMBER_ID` | Phone Number ID dari Meta Business | `123456789012345` |
| `WHATSAPP_ACCESS_TOKEN` | Token akses WhatsApp Cloud API | `EAAxxxxxxx...` |
| `GOOGLE_SHEET_ID` | ID Google Spreadsheet utama | `1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgVE2upms` |
| `CALENDAR_API_KEY` | API key PMS / channel manager Anda | `your_api_key` |
| `PROPERTY_ID` | ID properti di sistem kalender Anda | `villa_001` |
| `ADMIN_PHONE` | Nomor WA admin (format internasional) | `6281234567890` |
| `HOUSEKEEPING_PHONE` | Nomor WA housekeeping | `6281234567891` |
| `INTERNAL_TEAM_PHONE` | Nomor WA tim internal | `6281234567892` |
| `AGENCY_EMAIL` | Email pengirim (SMTP) | `hello@aiautomationagent.id` |
| `ONBOARDING_FORM_URL` | URL form onboarding (Typeform/Google Forms) | `https://forms.gle/xxx` |

---

## Credentials yang Dibutuhkan di n8n

Buat credentials berikut di **n8n → Credentials**:

### 1. Google Sheets
- Type: `Google Sheets OAuth2`
- Atau: `Google Sheets Service Account`

### 2. OpenAI (untuk workflow 05 & 06)
- Type: `OpenAI`
- API Key: dari [platform.openai.com](https://platform.openai.com)

### 3. Email / IMAP (untuk workflow 05)
- Type: `IMAP`
- Host: `imap.gmail.com` (untuk Gmail)
- Port: `993`
- **Catatan Gmail**: Aktifkan IMAP di Gmail settings, gunakan App Password bukan password biasa

### 4. Email / SMTP (untuk kirim email, workflow 05, 06, 07)
- Type: `SMTP`
- Host: `smtp.gmail.com`
- Port: `587`

---

## Struktur Google Sheets yang Dibutuhkan

Buat satu Google Spreadsheet dengan tabs/sheets berikut:

### Sheet: `Leads`
| Kolom | Tipe |
|-------|------|
| Phone | Text |
| CheckIn | Date |
| CheckOut | Date |
| GuestCount | Number |
| Status | Text (`new`, `quoted`, `booked`) |
| CreatedAt | DateTime |
| LastMessage | Text |
| QuotedAt | DateTime |
| QuotedPrice | Text |
| FollowUpSent | Text (`yes` / kosong) |
| FollowUpAt | DateTime |

### Sheet: `Bookings`
| Kolom | Tipe |
|-------|------|
| BookingID | Text |
| GuestName | Text |
| GuestPhone | Text |
| GuestEmail | Text |
| CheckIn | Date |
| CheckOut | Date |
| Platform | Text |
| PropertyID | Text |
| Status | Text |
| CreatedAt | DateTime |

### Sheet: `Guests`
| Kolom | Tipe |
|-------|------|
| Phone | Text |
| GuestName | Text |
| LastStay | Date |
| TotalStays | Number |
| RetargetTag | Text |
| UpdatedAt | DateTime |

### Sheet: `Pipeline`
| Kolom | Tipe |
|-------|------|
| Email | Text |
| Name | Text |
| BusinessType | Text |
| Intent | Text |
| Quality | Text (`HIGH`, `MEDIUM`, `LOW`) |
| NumProperties | Number |
| Stage | Text |
| CreatedAt | DateTime |
| ProposalSentAt | DateTime |

### Sheet: `Clients`
| Kolom | Tipe |
|-------|------|
| ClientID | Text |
| Name | Text |
| Email | Text |
| Phone | Text |
| Package | Text |
| NumProperties | Number |
| Status | Text |
| FormSentAt | DateTime |
| FormCompleted | Text |
| PropertyName | Text |
| Address | Text |
| WhatsAppNumber | Text |
| CheckInTime | Text |
| CheckOutTime | Text |
| BaseRate | Number |
| Amenities | Text |
| FormCompletedAt | DateTime |

---

## Alur Antar Workflow

```
Workflow 01 (Booking Engine)
  └─► Workflow 04 (Guest Journey) — dipanggil saat booking confirmed

Workflow 03 (Calendar Sync)
  └─► Workflow 04 (Guest Journey) — dipanggil saat booking confirmed

Workflow 05 (Sales Agent)
  └─► Workflow 06 (Proposal Agent) — dipanggil saat lead HIGH quality

Workflow 06 (Proposal Agent)
  └─► Workflow 07 (Onboarding Agent) — dipanggil saat deal closed
```

---

## Error Handling

Setiap workflow sudah dikonfigurasi dengan `errorWorkflow: "error-handler-workflow"`.

Buat satu workflow terpisah bernama `Error Handler` yang:
1. Menerima error trigger
2. Kirim notifikasi WhatsApp ke admin
3. Catat error ke Google Sheets untuk audit

---

## Checklist Setup

- [ ] Import semua 7 file JSON ke n8n
- [ ] Set semua Environment Variables
- [ ] Buat semua Credentials (Google Sheets, OpenAI, IMAP, SMTP)
- [ ] Buat Google Spreadsheet dengan semua sheets yang dibutuhkan
- [ ] Configure WhatsApp Cloud API dan dapatkan tokens
- [ ] Buat Error Handler workflow
- [ ] Test setiap workflow satu per satu dengan data dummy
- [ ] Aktifkan semua workflow
