# RSD Cross-Channel Client Acquisition System
**Real Space Digital — Internal Operations Document**
Last updated: June 3, 2026 | Owner: Zackary D. Jackson

---

## System Vision

Automate lead qualification and discovery call booking across **Instagram DM** and **WhatsApp** — 24/7, around the clock. A prospect comments on a post or sends a message. The AI qualifies them through the GRAVITY framework, handles objections in RSD voice, and either books the call or flags a hot lead for immediate human follow-up. No human needed until the call is confirmed on the calendar.

**North Star:** Every inbound touchpoint from either channel flows into one unified pipeline with zero leads falling through.

---

## Channel Architecture

```
INSTAGRAM DM                    WHATSAPP
─────────────                   ─────────
Keyword comment                 Inbound message
      │                               │
  ManyChat                    WA Cloud API
      │                               │
      └──────────┬────────────────────┘
                 ▼
        RSD Backend (Node.js)
                 │
         ┌───────┴────────┐
         │   Claude AI    │
         │  (GRAVITY)     │
         └───────┬────────┘
                 │
    ┌────────────┼────────────┐
    ▼            ▼            ▼
 Supabase    Calendly     HighLevel
 (state)    (booking)      (CRM)
```

---

## Phase Roadmap

### Phase 1 — Instagram DM Agent
**Status:** Backend complete · Deploy pending | **Timeline:** June 2026

| Task | Status |
|---|---|
| Node.js backend (server, routes, services) | ✅ Done |
| Claude GRAVITY qualification engine | ✅ Done |
| Supabase conversation state | ✅ Done |
| HighLevel CRM sync | ✅ Done |
| Handoff alert system (email + SMS) | ✅ Done |
| After-hours protocol | ✅ Done |
| Deploy to Railway or Render | ⏳ Pending |
| Set all .env variables | ⏳ Pending |
| Run docs/schema.sql in Supabase | ⏳ Pending |
| ManyChat External Request config | ⏳ Pending |
| Persona testing (all 5 scripts) | ⏳ Pending |
| Go-live on first post | ⏳ Pending |

**Linear:** [RSD IG DM Agent Project](https://linear.app/zdj-ventures/project/rsd-ig-dm-agent-instagram-automation-system-095855426c12) · RSD-42 through RSD-48

---

### Phase 2 — WhatsApp Agent
**Status:** Planning | **Timeline:** July 2026 | **Linear:** RSD-50, RSD-51

#### 2A — WhatsApp Business API Setup

| Option | Cost | Best For |
|---|---|---|
| Meta Cloud API (direct) | Free (pay per conversation) | Full control, lower cost |
| Business Service Provider (BSP) | $50–150/mo | Faster setup, managed |

**Recommended:** Meta Cloud API.

**Setup Steps:**
1. Create Meta Business Account at business.facebook.com
2. Create a Meta App → Add WhatsApp product
3. Get a WhatsApp Business Account (WABA) + Phone number
4. Generate a permanent system user access token
5. Register the phone number in the Meta dashboard
6. Configure webhook: `POST /webhook/whatsapp` → verify token
7. Subscribe to `messages` webhook field
8. Submit message templates for Meta approval (24hr window)

**Required credentials (.env):**
```
WA_PHONE_NUMBER_ID=your_phone_number_id
WA_ACCESS_TOKEN=your_permanent_system_user_token
WA_VERIFY_TOKEN=your_webhook_verify_token
WA_BUSINESS_ACCOUNT_ID=your_waba_id
```

#### 2B — Backend Extension

**New files:**
- `routes/whatsapp.js` — POST/GET /webhook/whatsapp
- `services/whatsapp-api.js` — send messages via Meta Graph API

**Meta webhook payload structure:**
```json
{
  "entry": [{
    "changes": [{
      "value": {
        "messages": [{
          "from": "15551234567",
          "text": { "body": "I need a 3D tour for my property" },
          "id": "wamid.xxx"
        }]
      }
    }]
  }]
}
```

**Send reply via API:**
```bash
POST https://graph.facebook.com/v18.0/{phone_number_id}/messages
Authorization: Bearer {token}
{ "messaging_product": "whatsapp", "to": "15551234567",
  "type": "text", "text": { "body": "Thanks for reaching out..." } }
```

**Supabase schema update:**
```sql
ALTER TABLE conversations
ADD COLUMN channel TEXT DEFAULT 'instagram' CHECK (channel IN ('instagram','whatsapp')),
ADD COLUMN wa_phone TEXT,
ADD COLUMN wa_message_id TEXT;
CREATE INDEX idx_conversations_wa_phone ON conversations(wa_phone);
```

**WhatsApp message templates needed (submit for Meta approval):**
- `rsd_opener` — First response to new inbound
- `rsd_handoff_alert` — Internal alert to Zack/Joel
- `rsd_follow_up` — Re-engage after 23hr silence

**Cross-channel deduplication:**
Track `wa_phone` in Supabase — prevent same lead from counting twice across channels.

---

### Phase 3 — Booking Integration
**Status:** Planned | **Timeline:** July 2026 | **Linear:** RSD-52

**On booking (`invitee.created` Calendly webhook):**
1. Update Supabase: `status = "booked"`
2. Move GHL contact to "Discovery Booked" pipeline stage
3. Generate pre-call brief from conversation transcript
4. Email brief to Zack/Joel
5. Send prospect confirmation + reminder on IG/WA

**Pre-call brief format:**
```
PROSPECT BRIEF — [Name] — [Date/Time]
Channel: Instagram / WhatsApp
ICP Type: Commercial Developer
Recommended: Phase 2 Growth · Entry: Matterport + Drone Bundle

Qualifiers:
- Property: 40,000 sq ft office complex in Richmond
- Goal: Investor documentation package for Q3
- Timeline: August (Q3 deadline)
- Decision maker: Yes — building owner
- Budget: Aligned with $1,500–$3,000

Transcript: [Supabase link]
```

**Post-booking sequence:**

| Timing | Message | Channel |
|---|---|---|
| T-0 (immediately) | Confirmation + calendar details | IG / WA |
| T-24 hours | Reminder + "What's the one thing you most want to know?" | IG / WA |
| T-1 hour | "We're looking forward to speaking with you" | IG / WA |
| No-show (+30 min) | Follow-up + reschedule offer | IG / WA |
| Post-call | Thank you + next steps | Email + IG / WA |

---

## ICP Client Journey Maps

### ICP 01 — Commercial RE Developer

```
IG "TOUR" comment or WA inbound message
         │
G: "What type of property are you working with?"
   → "40,000 sq ft office complex — need it for investors"
         │
R: "What would a successful result look like?"
   → "Investor documentation package before Q3"
         │
A: "Do you have a target date?"
   → "Investors coming in Q3 — need by August"
         │
V: "Is this your decision to make?"
   → "Yes — I own the building"
         │
I: "Our combined packages range $1,500–$3,000.
    Does that align?"
   → "Yes, that works"
         │
T: Send Calendly link → BOOKING
         │
GHL: New contact → tag: commercial_developer
     Stage: "Discovery Booked"
         │
Pre-call brief → Zack/Joel email
```
**Path:** Matterport + Drone Bundle → Phase 2 Growth or Phase 3 Authority

---

### ICP 02 — Design-Build / Architect

```
IG "AERIAL" or "SCOPE" keyword
         │
G → R → A → V → I
Budget range: $797 (Drone Day) to $2,497+ (Origin Story)
         │
Path: Photo + Drone Day → project scope on call → retainer
```

---

### ICP 03 — Owner Needing Content Scale

```
IG "AVATAR" keyword or WA "can't keep up with content"
         │
Entry: Mascot Avatar $997 or Full Clone $9,000
Path: Phase 1 Entry + Avatar add-on
```

---

### ICP 04 — After-Hours / Missed Leads

```
IG "AFTER5" or WA after-hours inquiry
         │
Skip deep qualification — present Operations Authority Layer
+$1,997/mo + $2,188 setup
Path: Calendly → Operations consultation
```

---

## Keyword + Trigger Map

| Keyword | Channel | ICP Signal | Entry Offer |
|---|---|---|---|
| TOUR | IG | Matterport interest | Matterport $1,497 |
| AERIAL | IG | Drone/construction | Photo + Drone Day $797 |
| SCOPE | IG | Large commercial | Custom scope on call |
| SYSTEM | IG | AI/lead gen | Operations Authority Layer |
| AVATAR | IG + WA | Content scale | Mascot $997 / Clone $9k |
| AFTER5 | IG + WA | After-hours missed leads | Ops Layer +$1,997/mo |
| PRICING | IG + WA | High intent, budget ready | Phase 1/2/3 comparison |
| INFO | IG + WA | General — monitor closely | Discovery needed |

> **WhatsApp:** No keyword triggers — any inbound message opens a session. Opener is context-aware. Re-engagements pull prior Supabase history.

---

## Full Tech Stack

| Layer | Tool | Purpose |
|---|---|---|
| IG Interface | ManyChat Pro ($15/mo) | Keyword triggers, DM delivery |
| WA Interface | Meta WhatsApp Cloud API | Inbound/outbound messaging |
| AI Engine | Claude Sonnet (claude-sonnet-4-5) | GRAVITY qualification |
| Backend | Node.js + Express | Webhook handler, orchestration |
| State | Supabase (free tier) | Conversation history, lead status |
| CRM | HighLevel | Pipeline, contacts, follow-up |
| Booking | Calendly | 30-min discovery call scheduling |
| Notifications | Email + Twilio SMS | Handoff alerts, pre-call briefs |
| Hosting | Railway or Render (~$8/mo) | Backend server |

**Monthly cost at 200 conversations (both channels combined):** ~$50–75/mo

---

## Backend File Structure (Updated)

```
rsd-channel-backend/
├── server.js
├── .env.example
├── config/
│   ├── index.js
│   └── systemPrompt.js          — GRAVITY (shared across channels)
├── routes/
│   ├── webhook.js               — ManyChat (IG)
│   ├── whatsapp.js              — WhatsApp Cloud API       ← NEW
│   ├── calendly.js              — Booking events           ← NEW
│   └── admin.js                 — Morning briefing
├── services/
│   ├── claude.js                — Conversation engine (shared)
│   ├── conversation.js          — Supabase CRUD
│   ├── highlevel.js             — CRM sync
│   ├── whatsapp-api.js          — Send WA messages         ← NEW
│   ├── calendly.js              — Booking processing       ← NEW
│   └── notifications.js         — Alerts + pre-call briefs
└── utils/
    ├── afterHours.js
    ├── handoffDetector.js
    └── icpRouter.js
```

---

## Key Metrics to Track

| Metric | Target | Source |
|---|---|---|
| DM/WA → Qualification rate | >40% | Supabase |
| Qualification → Calendly sent | >60% | Supabase |
| Calendly sent → Booked | >30% | Calendly analytics |
| Booked → Showed | >70% | GHL pipeline |
| Showed → Proposal sent | >50% | GHL pipeline |
| Proposal → Closed | 20–30% | GHL pipeline |
| **Overall: DM/WA → Closed** | **>3%** | Combined |

---

## Implementation Checklist

### Phase 1 — IG DM Deploy (This Week) · [RSD-49](https://linear.app/zdj-ventures/issue/RSD-49)
- [ ] Push backend to private GitHub repo
- [ ] Deploy to Railway or Render
- [ ] Set all .env variables
- [ ] Run schema.sql in Supabase
- [ ] Confirm /health returns 200
- [ ] Configure ManyChat External Request block
- [ ] Run all 5 persona scripts
- [ ] Go live on first TOUR post

### Phase 2 — WhatsApp · [RSD-50](https://linear.app/zdj-ventures/issue/RSD-50) · [RSD-51](https://linear.app/zdj-ventures/issue/RSD-51)
- [ ] Create Meta Business Account + App
- [ ] Get WABA approval + phone number
- [ ] Generate permanent access token
- [ ] Add /webhook/whatsapp to backend
- [ ] Configure webhook in Meta dashboard
- [ ] Submit 3 message templates for approval
- [ ] Update Supabase schema (channel + wa_phone fields)
- [ ] Test with personal WA number
- [ ] Verify cross-channel deduplication

### Phase 3 — Booking · [RSD-52](https://linear.app/zdj-ventures/issue/RSD-52)
- [ ] Set up Calendly webhook subscription
- [ ] Add /webhook/calendly route to backend
- [ ] Build pre-call brief generator
- [ ] Configure GHL stage update on booking
- [ ] Set up post-booking message sequence (IG + WA)
- [ ] Test full flow: comment → qualify → book → brief → call

### Phase 4 — Monitoring · [RSD-53](https://linear.app/zdj-ventures/issue/RSD-53)
- [ ] Schedule daily GET /admin/briefing at 7AM ET
- [ ] Weekly: review DM → Calendly conversion rate
- [ ] Monthly: refine GRAVITY prompt after 50+ convos/channel
- [ ] Monthly: A/B test keyword CTAs on posts

---

## Notes + Decisions Log

**June 3, 2026 — Zack**
- Decided on custom Node.js backend over ManyChat native Claude for full control
- Phase naming: Phase 1 Entry / Phase 2 Growth / Phase 3 Authority (no reef-themed names)
- WhatsApp will share the same GRAVITY framework + system prompt as IG
- Calendly 30-min discovery call format confirmed
- HighLevel as single source of truth for all qualified leads across channels

---

*Real Space Digital — Digitizing Reality, Elevating Sales™*
*Internal use only · Not for client distribution*
