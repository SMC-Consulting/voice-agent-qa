# Aurion QA — Test Case Registry & Testing Guide

Manual QA tracking for the **Aurion** platform — an AI-powered voice agent for IT support + customer service SaaS.

**~580 test cases** across 26 sections, organized by user journey.

---

## Table of Contents

1. [Getting Started](#getting-started)
2. [Environments & Access](#environments--access)
3. [Testing Workflow](#testing-workflow)
4. [Test Case Sections](#test-case-sections)
5. [Voice Agent Subsections](#section-4--voice-agent-usage-subsections)
6. [Aurion CS Subsections](#section-9--aurion-cs-platform-subsections)
7. [Labels & Tracking](#labels--tracking)
8. [Test Data Reference](#test-data-reference)
9. [Product Context](#product-context)

---

## Getting Started

### What is Aurion?

Aurion is a **dual-product SaaS platform**:

- **Aurion Voice Agent** — AI voice agent that integrates with ITSM providers (Freshservice, HaloITSM, ServiceNow, JSM) and CS providers (Freshdesk, Zendesk) via real-time voice calls. Users call in, authenticate by name + badge, and manage tickets/KB/assets via natural conversation.
- **Aurion CS** — Customer service platform with inbox, conversations, AI copilot, workflows, SLA management, and multi-channel support (email, WhatsApp, voice widget).

Tenants choose a **product profile** during onboarding:
| Profile | What's enabled |
|---------|---------------|
| `voice` | Voice Agent only (ITSM tools, call management) |
| `platform` | Aurion CS only (inbox, conversations, AI features) |
| `complete` | Both Voice Agent + Aurion CS |

### Where to Start Testing

Follow this order for a complete test run:

```
1. Website & Signup        → Can users find and sign up?
2. Onboarding              → Does the 5-step wizard work for each profile?
3. Dashboard & Config      → Are all settings accessible and functional?
4. Voice Agent             → Make a call, authenticate, create tickets, search KB
5. Widget                  → Test the embeddable voice + chat widget
6. Help Center             → Test the public KB portal as an end user
7. Aurion CS               → Inbox, conversations, AI features, channels
8. Mobile App              → End-user ticket management via mobile
9. Billing                 → Plans, Stripe checkout, invoicing
10. Ops Console            → Super-admin fleet management (ops-staging)
```

For **smoke testing** (quick sanity check), focus on sections marked `priority:critical`:
- Signup flow (TC-005→010)
- Onboarding wizard (TC-012→016, TC-566→573)
- Voice call + authentication (TC-041→052)
- Ticket creation via voice (TC-053→058)
- CS inbox basics (TC-204→219)
- Billing activation (TC-348→356)

---

## Environments & Access

### Staging URLs

| App | URL | Purpose |
|-----|-----|---------|
| **Admin Dashboard** | `https://apps-staging.aurionai.net` | Tenant admin panel |
| **Ops Console** | `https://ops-staging.aurionai.net` | Super-admin / platform ops |
| **Help Center** | `https://help-staging.aurionai.net` | Public KB portal (end-user facing) |
| **LiveKit** | `wss://lk-staging.aurionai.net` | WebRTC media server (voice calls) |
| **API Base** | `https://apps-staging.aurionai.net/api` | Backend API |
| **Marketing Website** | `https://aurionai.net` | Public website (Vercel, production) |

### Local Development URLs

For testing against local dev (requires `./dev.sh` running):

| App | URL | Notes |
|-----|-----|-------|
| Admin Dashboard | `http://localhost:3000` | Tenant mode |
| Ops Console | `http://localhost:3001` | Super-admin mode |
| Help Center | `http://localhost:3002` | Public KB portal |
| Admin API | `http://localhost:8000` | Backend |
| Twilio Webhooks | `http://localhost:8001` | Phone event handlers |
| Developer Portal | `http://localhost:3003` | Manual start: `cd apps/developer-portal && pnpm dev --port 3003` |
| Widget Dev | `http://localhost:5173` | `cd apps/widget && pnpm dev` |
| LiveKit Agent | `http://localhost:8081` | Voice agent health endpoint |

### Authentication

**Staging dashboard login:** Use your staging credentials (Google SSO or email/password).

**Local dev bypass:** The dashboard supports dev auth bypass for local testing:
- Set `NEXT_PUBLIC_DEV_AUTH_BYPASS=true` in `apps/admin-dashboard/.env.local`
- Set `ALLOW_DEV_AUTH_BYPASS=true` in `apps/admin-api/.env`
- A "Dev Login" button appears on the login page

**Ops console:** Azure AD SSO required (staging: `ops-staging.aurionai.net`).

### ITSM Provider Test Instances

| Provider | Instance | Notes |
|----------|----------|-------|
| **Freshservice** | `cspohelpdesk.freshservice.com` | Dev sandbox — tickets created here are real |
| **HaloITSM** | `smcconsultingdev.haloitsm.com` | Dev sandbox |
| **ServiceNow** | Configure per-tenant | Requires instance URL + OAuth or Basic auth |
| **JSM** | Configure per-tenant | Requires Atlassian Cloud instance |

### CS Provider Test Instances

| Provider | Notes |
|----------|-------|
| **Freshdesk** | Configure per-tenant (domain + API key) |
| **Zendesk** | Configure per-tenant (subdomain + email + API token) |

### Voice Testing

| Method | How |
|--------|-----|
| **Phone call** | Call the Twilio number assigned to the tenant |
| **Voice widget** | Open the widget on a test page or via `localhost:5173` |
| **Mobile app** | Connect Expo Go to staging API (`API_URL=https://apps-staging.aurionai.net`) |

**Test phone number (dev):** `+322622211` (Belgium, configured in `.env`)

---

## Testing Workflow

### How to Execute a Test Case

1. **Open the issue** — Each TC is a GitHub issue in this repo
2. **Check prerequisites** — Look at the "Prerequisites" section in the issue
3. **Follow test steps** — Execute each numbered step
4. **Record results** — Fill in the "Test Execution" table at the bottom of the issue:

```markdown
| Run | Date | Tester | Result | Notes |
|-----|------|--------|--------|-------|
|  1  | 2026-03-11 | EY | PASS | All steps verified on staging |
```

5. **Update labels:**
   - `status:passed` — All steps verified
   - `status:failed` — One or more steps failed (add bug details in notes)
   - `status:blocked` — Cannot test (dependency missing, environment issue)

### How to Report a Bug

When a TC fails:
1. Change the issue label to `status:failed`
2. Add a comment with:
   - **Step that failed** (e.g., "Step 7 — expected redirect to dashboard, got 500 error")
   - **Screenshot or screen recording** (attach to the comment)
   - **Browser console errors** (if UI issue)
   - **Environment** (staging URL, browser, OS)
3. Create a **bug issue** in the main repo ([SMC-Consulting/voice-agent](https://github.com/SMC-Consulting/voice-agent)) referencing the TC number

### Test Case Issue Template

Every TC issue follows this structure:

```markdown
## Scenario
What is being tested and why.

## Prerequisites
- [ ] Logged in as tenant admin on staging
- [ ] ITSM provider configured (Freshservice)
- [ ] At least one requester synced

## Test Steps
1. Navigate to [URL]
2. Click [element]
3. Verify [expected behavior]
...

## Expected Results
- [ ] Result 1
- [ ] Result 2

## Staging URLs
| Page | URL |
|------|-----|
| Target page | https://apps-staging.aurionai.net/... |

## Test Execution
| Run | Date | Tester | Result | Notes |
|-----|------|--------|--------|-------|
|  1  |      |        |        |       |
```

---

## Test Case Sections

### Section Index

| # | Section | Labels | TC Count | Priority |
|---|---------|--------|----------|----------|
| 1 | [Website & Signup](#1-website--signup) | `section:website`, `section:signup`, `section:website-extended` | ~16 | High |
| 2 | [Onboarding](#2-onboarding) | `section:onboarding` | ~19 | Critical |
| 3 | [Admin Dashboard & Configuration](#3-admin-dashboard--configuration) | `section:dashboard`, `section:integration-config`, `section:config-templates`, `section:authentication`, `section:email-notifications`, `section:dashboard-extended` | ~36 | High |
| 4 | [Voice Agent Usage](#4-voice-agent-usage) | `section:call-setup` through `section:health` | ~93 | Critical |
| 5 | [KB Configuration & Testing](#5-kb-configuration--testing) | `section:kb-creator`, `section:aurion-kb`, `section:kb-approval`, `section:kb-tags-import` | ~27 | High |
| 6 | [Chat & Voice Widget](#6-chat--voice-widget) | `section:widget`, `section:widget-text` | ~12 | High |
| 7 | [Twilio & Telephony](#7-twilio--telephony) | `section:twilio-webhooks`, `section:twilio-extended`, `section:telephony` | ~15 | High |
| 8 | [Help Center](#8-help-center) | `section:help-center` | 20 | Medium |
| 9 | [Aurion CS Platform](#9-aurion-cs-platform) | `section:cs-inbox` through `section:aurion-native` | ~144 | Critical |
| 10 | [Billing & Invoicing](#10-billing--invoicing) | `section:billing`, `section:invoicing`, `section:stripe-webhooks` | ~24 | Critical |
| 11 | [Public API & Developer Portal](#11-public-api--developer-portal) | `section:public-api`, `section:api-metering`, `section:developer-portal`, `section:devportal-extended` | ~21 | Medium |
| 12 | [Security & Compliance](#12-security--compliance) | `section:security`, `section:rbac-security`, `section:consent-legal` | ~28 | Critical |
| 13 | [Sync & CronJobs](#13-sync--cronjobs) | `section:sync`, `section:cronjobs` | ~20 | Medium |
| 14 | [Analytics & Monitoring](#14-analytics--monitoring) | `section:analytics` | 8 | Medium |
| 15 | [Data Management & Admin Scripts](#15-data-management--admin-scripts) | `section:data-export`, `section:admin-scripts`, `section:admin-scripts-ext`, `section:config-backup` | ~18 | Low |
| 16 | [Infrastructure & Provisioning](#16-infrastructure--provisioning) | `section:infra-policies`, `section:pricing-v2`, `section:46-status-page-management`, `section:tenant-provisioning` | ~12 | Low |
| 17 | [Ops Console & Super-Admin](#17-ops-console--super-admin) | `section:super-admin`, `section:super-admin-ops`, `section:ops-console`, `section:fleet` | ~33 | High |
| 18 | [End-User Auth & Flows](#18-end-user-auth--flows) | `section:end-user-auth` | 4 | High |
| 19 | [Meeting Scheduler](#19-meeting-scheduler) | `section:meeting-scheduler` | 5 | Low |
| 20 | [Text Agent](#20-text-agent) | `section:text-agent` | 4 | Medium |
| 21 | [Email Pipeline](#21-email-pipeline) | `section:email-pipeline` | 6 | Medium |
| 22 | [Mobile App](#22-mobile-app) | `section:mobile`, `section:mobile-extended`, `section:end-user-mobile` | ~21 | High |

---

### 1. Website & Signup

**What:** Public marketing website (`aurionai.net`) and account registration flow.

**Where to test:** `https://aurionai.net` (website), `https://apps-staging.aurionai.net/signup` (registration)

**Labels:** `section:website`, `section:signup`, `section:website-extended`

| Subsection | TCs | What to verify |
|------------|-----|----------------|
| Website pages | TC-001→004 | Homepage, pricing, features, integrations pages load correctly, i18n works |
| Signup flow | TC-005→010 | Email registration, Google SSO signup, email verification, tenant creation |
| Website SEO & content | TC-011, TC-520→524 | Meta tags, blog, comparison pages, contact form |

---

### 2. Onboarding

**What:** 5-step tenant onboarding wizard that runs after signup. Adapts based on product profile (voice/platform/complete).

**Where to test:** `https://apps-staging.aurionai.net/onboarding`

**Labels:** `section:onboarding`

| Subsection | TCs | What to verify |
|------------|-----|----------------|
| Step 1 — Organization & Plan | TC-012, TC-566 | Org name, slug, timezone, industry, plan selection |
| Step 2 — Provider Setup | TC-567 | ITSM provider (FS/Halo/SNOW/JSM) + CS provider (FD/ZD) credential entry & test |
| Step 3 — Voice Configuration | TC-013 | TTS voice, STT provider, language, agent personality |
| Step 4 — Helpdesk Setup | TC-568 | CS platform configuration (business hours, categories, templates) |
| Step 5 — Review & Activate | TC-569 | Review cards, legal acceptance, contact import, tenant activation |
| Billing page | TC-570 | Plan selection, Stripe checkout, payment |
| Navigation flows | TC-571→573 | Voice profile flow, Platform profile flow, Complete profile flow |
| Test call & checklist | TC-014→016 | Test call diagnostics, checklist completion, free trial activation |

**Key test variations:** Test onboarding for each product profile (`voice`, `platform`, `complete`) — the wizard shows/hides steps based on the selected profile.

---

### 3. Admin Dashboard & Configuration

**What:** Core dashboard navigation, tenant settings, ITSM/CS integration configuration, user management.

**Where to test:** `https://apps-staging.aurionai.net/` (post-login)

**Labels:** `section:dashboard`, `section:integration-config`, `section:config-templates`, `section:authentication`, `section:email-notifications`, `section:dashboard-extended`

| Subsection | TCs | What to verify |
|------------|-----|----------------|
| Dashboard home & nav | TC-017→021 | Dashboard loads, sidebar navigation, profile menu, responsive layout |
| ITSM integration config | TC-022→027 | Provider credentials, connection test, sync settings, category mapping |
| CS provider config | TC-028→030 | CS provider setup, credential validation, feature toggling |
| Configuration templates | TC-031→035 | Template CRUD, template application, default values |
| Auth & account | TC-036→037, TC-052 | Login, password reset, SSO, tenant switching |
| Email notifications | TC-038→040 | Notification settings, email templates, alert preferences |
| Dashboard extended | TC-540→544 | Activity log, search, breadcrumbs, keyboard shortcuts, dark mode |

**Key pages:**
- `/configuration/voice-settings` — TTS/STT provider, voice selection, language
- `/configuration/llm-config` — LLM provider (Claude/GPT/DeepSeek/Groq/Gemini/Kimi)
- `/configuration/integrations` — ITSM + CS provider credentials
- `/configuration/business-hours` — Operating hours
- `/configuration/users` — Tenant user management + RBAC roles
- `/tenant/billing` — Subscription & plan management

---

### 4. Voice Agent Usage

**What:** The core voice AI experience — making calls, authenticating, managing tickets, searching KB, and using ITSM tools via natural conversation.

**Where to test:** Call the test Twilio number, or use the voice widget at `localhost:5173` / staging embed.

**Labels:** `section:call-setup`, `section:authentication`, `section:tickets`, `section:kb`, `section:assets`, `section:approvals`, `section:mcp-advanced`, `section:shortcuts`, `section:multi-provider`, `section:cs-providers`, `section:multilang`, `section:recording`, `section:errors`, `section:conversations`, `section:outbound-calls`, `section:health`

See [Voice Agent Subsections](#section-4--voice-agent-usage-subsections) below for the full breakdown (16 subsections, 93 TCs).

**Quick smoke test:**
1. Call the test number
2. Authenticate as John Lambert, badge 789456
3. Say "I need help with my laptop" → verify ticket creation
4. Say "Search knowledge base for password reset" → verify KB results
5. Say "Check my tickets" → verify ticket list

---

### 5. KB Configuration & Testing

**What:** Knowledge base article creation pipeline, article management, KB approval workflows, tags & import.

**Where to test:** `https://apps-staging.aurionai.net/tools/kb-article-creator` (creation), `/configuration/kb-settings` (config)

**Labels:** `section:kb-creator`, `section:aurion-kb`, `section:kb-approval`, `section:kb-tags-import`

| Subsection | TCs | What to verify |
|------------|-----|----------------|
| KB article creator (v2) | TC-134→138 | Upload PDF/DOCX/MD, LLM structuring, preview, publish to ITSM |
| Aurion KB management | TC-139→146 | Article CRUD, categories, collections, search indexing |
| KB approval workflows | TC-147→151 | Draft → Review → Approve → Publish flow, reviewer assignment |
| KB tags & import | TC-152→156 | Tag management, bulk import, tag-based filtering |

---

### 6. Chat & Voice Widget

**What:** Embeddable Preact widget with voice + chat capabilities. Served as a single JS file.

**Where to test:** Widget dev server `localhost:5173`, or embed on any HTML page via:
```html
<script>
  window.AurionSettings = {
    appId: "{tenant-app-id}",
    publicKey: "{tenant-public-key}",
    apiBaseUrl: "https://apps-staging.aurionai.net"
  };
</script>
<script src="https://apps-staging.aurionai.net/api/v1/voice-widget/embed/aurion.js"></script>
```

**Labels:** `section:widget`, `section:widget-text`

| Subsection | TCs | What to verify |
|------------|-----|----------------|
| Voice widget | TC-157→161 | Widget loads, mic permission, voice call connects, real-time transcript |
| Chat widget | TC-162→168 | Text input, message display, file attachments, widget styling |

---

### 7. Twilio & Telephony

**What:** Twilio phone integration, SIP trunks, inbound/outbound call routing.

**Where to test:** `/configuration/telephony`, `/configuration/sip-trunks`

**Labels:** `section:twilio-webhooks`, `section:twilio-extended`, `section:telephony`

| Subsection | TCs | What to verify |
|------------|-----|----------------|
| Twilio webhooks | TC-169→173 | Inbound call webhook, status callback, recording webhook |
| Twilio extended | TC-174→177 | Multi-number support, call forwarding, voicemail |
| Telephony config | TC-178→183 | SIP trunk setup, phone number management, routing rules |

**Note:** Twilio account is in **Ireland (IE1)**. Both `TWILIO_REGION=ie1` and `TWILIO_EDGE=dublin` are required.

---

### 8. Help Center

**What:** Public-facing KB portal where end users browse articles, search, submit tickets, and track existing requests.

**Where to test:** `https://help-staging.aurionai.net` (or `localhost:3002` locally)

**Labels:** `section:help-center`

| Subsection | TCs | What to verify |
|------------|-----|----------------|
| Public browsing | TC-184→190 | Homepage, category listing, article pages, breadcrumbs |
| Search | TC-191→195 | Keyword search, AI-powered summaries, search suggestions |
| End-user tickets | TC-196→200 | Create ticket, view ticket list, ticket detail, add comment |
| Auth & localization | TC-201→203 | Login/signup, language switching (11 locales), branding |

**Supported locales:** da, de, en, es, fi, fr, it, nl, pl, pt, sv

---

### 9. Aurion CS Platform

**What:** Full customer service platform — inbox, conversations, contacts, AI copilot, multi-channel (email, WhatsApp), workflows, SLA, CSAT.

**Where to test:** `https://apps-staging.aurionai.net/inbox` (inbox), `/conversations` (history), `/contacts` (directory)

**Labels:** See subsections below.

**Requires:** Tenant with `platform` or `complete` product profile.

See [Aurion CS Subsections](#section-9--aurion-cs-platform-subsections) below for the full breakdown (19 subsections, 144 TCs).

---

### 10. Billing & Invoicing

**What:** Stripe integration, subscription management, quota tracking, payment processing.

**Where to test:** `/tenant/billing` (dashboard), Stripe test mode

**Labels:** `section:billing`, `section:invoicing`, `section:stripe-webhooks`

| Subsection | TCs | What to verify |
|------------|-----|----------------|
| Subscription management | TC-348→356 | Plan selection, upgrade/downgrade, quota display, billing status banner |
| Invoicing | TC-357→362 | Invoice history, PDF download, payment method update |
| Stripe webhooks | TC-363→371 | Payment success/failure, subscription renewal, past-due handling, grace period |

---

### 11. Public API & Developer Portal

**What:** REST API for integrators, API key management, webhook subscriptions, developer documentation.

**Where to test:** `/configuration/api-keys` (key management), `/tenant/settings/api` (API settings), Developer Portal `localhost:3003`

**Labels:** `section:public-api`, `section:api-metering`, `section:developer-portal`, `section:devportal-extended`

| Subsection | TCs | What to verify |
|------------|-----|----------------|
| Public API | TC-372→381 | Ticket CRUD via API, auth with API keys, pagination, error responses |
| API metering | TC-382→384 | Usage tracking, rate limiting, quota enforcement |
| Developer portal | TC-385→392 | Documentation pages, API explorer, code examples, webhook docs |

---

### 12. Security & Compliance

**What:** RBAC, audit logging, GDPR compliance, security settings, data protection.

**Where to test:** `/configuration/security-settings`, `/configuration/roles`, `/configuration/audit-log`

**Labels:** `section:security`, `section:rbac-security`, `section:consent-legal`

| Subsection | TCs | What to verify |
|------------|-----|----------------|
| Security controls | TC-393→404 | Password policies, session management, IP allowlisting, 2FA |
| RBAC | TC-405→410 | Role CRUD, permission matrix, role assignment, access enforcement |
| Legal & consent | TC-411→420 | GDPR data export, consent tracking, privacy policy, data retention |

---

### 13. Sync & CronJobs

**What:** Background jobs that sync data from ITSM/CS providers and perform scheduled maintenance.

**Where to test:** Trigger via API (`POST /api/sync/...`) or verify via K8s CronJob logs.

**Labels:** `section:sync`, `section:cronjobs`

| Subsection | TCs | What to verify |
|------------|-----|----------------|
| Data sync | TC-421→430 | Requester sync (8h), contact sync (8h), service catalog (daily), storage usage |
| CronJobs | TC-431→440 | Quota alerts (hourly), suspension sweep (hourly), pod snapshots (2min), LLM aggregation (daily), attachment cleanup, CSAT survey send, notification digest |

---

### 14. Analytics & Monitoring

**What:** Call analytics dashboards, usage patterns, SLA tracking, export reports.

**Where to test:** `/analytics/voice`, `/analytics/channels`, `/analytics/performance`

**Labels:** `section:analytics`

| TCs | What to verify |
|-----|----------------|
| TC-441→448 | Dashboard loads with real data, date range filtering, chart rendering, CSV/PDF export, resolution rate metrics, SLA quality tracking |

---

### 15. Data Management & Admin Scripts

**What:** Data export, admin CLI scripts, configuration backup/restore.

**Where to test:** Admin scripts run via CLI; backup/restore via `/configuration/backup-restore`

**Labels:** `section:data-export`, `section:admin-scripts`, `section:admin-scripts-ext`, `section:config-backup`

| Subsection | TCs | What to verify |
|------------|-----|----------------|
| Data export | TC-449→451 | Tenant data export (GDPR), conversation export, ticket export |
| Admin scripts | TC-452→456 | Cartesia voice backfill, config audit, OpenAPI export, sandbox reset, Stripe bootstrap |
| Admin scripts ext | TC-457→461 | Search vector backfill, SLA backfill, KB capabilities, image URL fix, article recount |
| Config backup | TC-462→466 | Backup creation, restore, version history, cross-tenant restore validation |

---

### 16. Infrastructure & Provisioning

**What:** K8s tenant provisioning, infrastructure policies, pricing tiers, status page management.

**Where to test:** Ops console (`ops-staging.aurionai.net`), status page UI

**Labels:** `section:infra-policies`, `section:pricing-v2`, `section:46-status-page-management`, `section:tenant-provisioning`

| Subsection | TCs | What to verify |
|------------|-----|----------------|
| Infrastructure policies | TC-467→470 | Resource limits, HPA configuration, namespace isolation |
| Pricing tiers | TC-471→473 | Plan feature matrix, tier-based limits, upgrade paths |
| Status page | TC-474→477 | Create/update/resolve incidents, impact levels, component status |
| Tenant provisioning | TC-478 | Per-tenant K8s pod creation, sidecar injection, health monitor |

---

### 17. Ops Console & Super-Admin

**What:** Platform-wide operations console for super-admins. Tenant lifecycle, fleet health, billing oversight, rollout management.

**Where to test:** `https://ops-staging.aurionai.net` (requires Azure AD SSO)

**Labels:** `section:super-admin`, `section:super-admin-ops`, `section:ops-console`, `section:fleet`

| Subsection | TCs | What to verify |
|------------|-----|----------------|
| Tenant management | TC-479→492 | Tenant list, create/suspend/activate, config templates, impersonation |
| Ops operations | TC-493→497 | Platform health dashboard, system alerts, maintenance mode |
| Ops console UI | TC-498→506 | Navigation, search, filtering, bulk actions, LLM cost tracking |
| Fleet management | TC-507→511 | Fleet rollout (pause/resume/cancel/rollback), pod snapshots, HPA scaling |

---

### 18. End-User Auth & Flows

**What:** End-user authentication for mobile app, help center, and voice widget. Separate JWT from admin auth.

**Where to test:** Help center login, mobile app login, widget auth

**Labels:** `section:end-user-auth`

| TCs | What to verify |
|-----|----------------|
| TC-516→519 | End-user login (email/password), magic link auth, JWT refresh, SSO (Google/Microsoft) |

---

### 19. Meeting Scheduler

**What:** Calendar integration for scheduling meetings from within the platform.

**Where to test:** `/configuration/meetings`, profile calendar tab

**Labels:** `section:meeting-scheduler`

| TCs | What to verify |
|-----|----------------|
| TC-525→529 | Calendar connection (Google/Microsoft), availability slots, meeting creation, notifications |

---

### 20. Text Agent

**What:** AI-powered text-based agent (non-voice) for handling written support requests.

**Where to test:** Chat widget text mode, CS inbox AI features

**Labels:** `section:text-agent`

| TCs | What to verify |
|-----|----------------|
| TC-530→533 | Text conversation flow, intent detection, tool calling via text, handoff to human |

---

### 21. Email Pipeline

**What:** Inbound/outbound email processing for CS conversations.

**Where to test:** `/configuration/email`, CS inbox (email channel)

**Labels:** `section:email-pipeline`

| TCs | What to verify |
|-----|----------------|
| TC-534→539 | Inbound email → conversation creation, reply threading, attachments, bounce handling, sender verification, email templates |

---

### 22. Mobile App

**What:** React Native (Expo) end-user app for ticket management, voice calls, and conversations.

**Where to test:** Install via Expo Go, point to staging API

**Setup:**
```bash
cd apps/mobile-app
API_URL=https://apps-staging.aurionai.net npx expo start
# Scan QR with Expo Go (iOS/Android)
```

**Labels:** `section:mobile`, `section:mobile-extended`, `section:end-user-mobile`

| Subsection | TCs | What to verify |
|------------|-----|----------------|
| Core mobile | TC-545→555 | Login, ticket list, ticket detail, create ticket, voice call, settings |
| Mobile extended | TC-556→560 | Push notifications, offline mode, deep links, biometric auth |
| End-user mobile | TC-561→565 | Magic link login, org selection, tenant picker, profile management |

---

## Section 4 — Voice Agent Usage (Subsections)

| # | Subsection | TCs | Label | What to verify |
|---|------------|-----|-------|----------------|
| 4a | Call Setup | TC-041→046 | `section:call-setup` | Inbound call connects, LiveKit room created, agent greeting, language detection, caller ID lookup |
| 4b | Authentication | TC-047→052 | `section:authentication` | Name collection, badge entry (voice + DTMF), fuzzy name match, bcrypt verification, 3-attempt lockout (30min) |
| 4c | Tickets | TC-053→058 | `section:tickets` | Create ticket via voice, search tickets, get ticket details, update priority/status, add note |
| 4d | KB Search | TC-059→062 | `section:kb` | Search KB via voice, article summary read-back, follow-up questions, mark as resolved |
| 4e | Assets | TC-063→065 | `section:assets` | Search CMDB assets, get asset relationships, report asset issue |
| 4f | Approvals & Changes | TC-066→070 | `section:approvals` | Create approval request, check approval status, manager pending list, create/check change request |
| 4g | MCP Advanced Tools | TC-071→082 | `section:mcp-advanced` | KEDB search, problem linking, service catalog browse, service request creation, ticket history, group listing, reassignment |
| 4h | Shortcuts | TC-083→090 | `section:shortcuts` | "Quick ticket: [desc]", "Check my tickets", "Check my assets" — regex + fuzzy matching (0.8 threshold) + LLM fallback |
| 4i | Multi-Provider | TC-091→098 | `section:multi-provider` | Same voice flows against Freshservice, HaloITSM, ServiceNow, JSM — verify provider-agnostic behavior |
| 4j | CS Voice Routing | TC-099→104 | `section:cs-providers` | Calls routed to CS provider (Freshdesk/Zendesk) instead of ITSM when tenant uses CS profile |
| 4k | Multilanguage | TC-105→113 | `section:multilang` | French, Dutch, German, Spanish, Italian support. Language switching mid-call. Cartesia TTS localization |
| 4l | Recording | TC-114→117 | `section:recording` | Call recording toggle, S3 upload, retention policy, playback in conversation detail |
| 4m | Error Handling | TC-118→122 | `section:errors` | LLM timeout fallback, MCP connection failure, ITSM API errors, graceful degradation |
| 4n | Conversations | TC-123→125 | `section:conversations` | Conversation history saved, transcript accuracy, metadata (duration, tools used) |
| 4o | Outbound Calls | TC-126→129 | `section:outbound-calls` | Scheduled callback, outbound call initiation, recipient pickup, conversation flow |
| 4p | Health Checks | TC-130→133 | `section:health` | Agent health endpoint (`/`), MCP subprocess health, LLM connectivity, Redis/DB health |

---

## Section 9 — Aurion CS Platform (Subsections)

| # | Subsection | TCs | Label | What to verify |
|---|------------|-----|-------|----------------|
| 9a | CS Inbox | TC-204→219 | `section:cs-inbox` | Inbox list, filters, assignment, status changes, snooze, merge, sidebar info panel |
| 9b | Conversation Management | TC-220→233 | `section:conversation-mgmt` | Reply, internal note, attachments, canned responses, conversation timeline, collaborators, shared links |
| 9c | Contacts & Companies | TC-234→241 | `section:contacts-companies` | Contact CRUD, company CRUD, contact-company linking, import/export, merge duplicates |
| 9d | AI Features | TC-242→254 | `section:ai-features` | AI draft reply, AI resolution notes, KB search with query translation, sentiment analysis, AI copilot panel, AI routing, AI classification, text rephrasing |
| 9e | Email Channel | TC-255→262 | `section:email-channel` | Email channel setup (Google/Microsoft OAuth), inbound email parsing, reply-to threading, signature stripping |
| 9f | WhatsApp Channel | TC-263→268 | `section:whatsapp-channel` | WhatsApp Business API setup, inbound message handling, template messages, media support |
| 9g | Workflows | TC-269→278 | `section:workflows` | Workflow builder, trigger conditions, actions (assign, notify, update), workflow execution log |
| 9h | SLA | TC-279→286 | `section:sla` | SLA policy CRUD, first response time, resolution time, breach alerts, business hours integration |
| 9i | CSAT & Saved Replies | TC-287→292 | `section:csat-replies` | CSAT survey config, survey delivery (post-resolution), response collection, saved reply CRUD |
| 9j | Custom Fields & Rules | TC-293→300 | `section:custom-fields-rules` | Custom field CRUD (text/dropdown/date/checkbox), field visibility rules, business rule triggers |
| 9k | Workforce Management | TC-301→308 | `section:workforce` | Agent shifts, on-call rotation, skill tags, availability tracking, capacity management |
| 9l | Notifications | TC-309→316 | `section:notifications` | In-app notifications, email digest, webhook delivery, notification preferences, quiet hours |
| 9m | Business Hours & Flags | TC-317→320 | `section:business-hours` | Business hours CRUD, timezone handling, holiday calendar, feature flag toggles |
| 9n | Realtime | TC-321→324 | `section:realtime` | Live typing indicator, presence status, real-time message delivery, connection recovery |
| 9o | Bulk & Analytics | TC-325→329 | `section:bulk-analytics` | Bulk assign, bulk status change, CS analytics dashboard, team performance metrics |
| 9p | Products & Catalog | TC-330→333 | `section:products-catalog` | Product CRUD, product-ticket linking, catalog browsing, product search |
| 9q | CS Voice Tools | TC-334→339 | `section:cs-tools` | CS-specific MCP tools (create/get CS ticket), CS provider routing, tool dispatch |
| 9r | CS Voice Auth | TC-340→343 | `section:cs-voice-auth` | CS contact authentication (email-based), contact lookup, conversation linking |
| 9s | Aurion Native | TC-344→347 | `section:aurion-native` | Aurion as its own ITSM provider (no external integration), native ticket storage |

---

## Labels & Tracking

### Label Categories

Each issue carries labels from three categories:

**Priority** (required):
| Label | When to use |
|-------|-------------|
| `priority:critical` | Core user flow — signup, auth, ticket creation, billing |
| `priority:high` | Important feature — config, KB, widget, mobile |
| `priority:medium` | Supporting feature — analytics, sync, developer portal |
| `priority:low` | Edge case — admin scripts, infrastructure, data export |

**Status** (updated during testing):
| Label | Meaning |
|-------|---------|
| `status:not-started` | Not yet tested |
| `status:in-progress` | Currently being tested |
| `status:passed` | All steps verified successfully |
| `status:failed` | One or more steps failed |
| `status:blocked` | Cannot test (dependency, environment, or access issue) |

**Section** (auto-assigned at creation):
- Format: `section:<name>` (e.g., `section:onboarding`, `section:cs-inbox`)
- Matches the subsection structure above

**Special labels:**
| Label | Purpose |
|-------|---------|
| `qa` | General QA test case |
| `bug` | Bug found during testing |
| `e2e-test` | Has automated E2E test coverage |
| `test-run` | Part of a specific test run |

### Filtering Issues

```bash
# All critical, not-started test cases
gh issue list --repo SMC-Consulting/voice-agent-qa --label "priority:critical,status:not-started"

# All onboarding TCs
gh issue list --repo SMC-Consulting/voice-agent-qa --label "section:onboarding"

# All failed TCs
gh issue list --repo SMC-Consulting/voice-agent-qa --label "status:failed"

# Count by status
gh issue list --repo SMC-Consulting/voice-agent-qa --label "status:passed" --json number --jq length
```

---

## Test Data Reference

### Voice Agent Test User

| Field | Value |
|-------|-------|
| **Name** | John Lambert |
| **Email** | john@smc.consulting |
| **Badge ID** | `789456` |
| **Wrong Badge** (for failure tests) | `000000` |
| **Language** | French (fr-FR) |
| **Provider** | Freshservice (cspohelpdesk) |

### Voice Auth Flow

The voice agent uses **LLM-native 2-factor authentication**:

1. Agent asks: *"Bonjour, comment puis-je vous aider?"*
2. Caller says their **name** → LLM extracts via fuzzy matching
3. Agent asks for **badge number** → Caller speaks digits or enters via DTMF keypad
4. System verifies: fuzzy name search + bcrypt badge hash
5. **3 failed attempts** → 30-minute lockout
6. Badge IDs are never logged (security requirement)

### Sandbox Reset

To reset test data to a clean state:
```bash
cd apps/admin-api
.venv/bin/python -m src.scripts.reset_sandbox
```
This deletes conversations, tickets, and test data for sandbox tenants while preserving tenant configuration.

### Automated Test Coverage

Some TCs have corresponding automated tests in the main repo (`SMC-Consulting/voice-agent`):

| Area | Automated Tests | Location |
|------|----------------|----------|
| Voice auth | Unit + integration | `apps/livekit-agent/tests/unit/test_caller_auth_service.py` |
| MCP tools | Integration | `apps/livekit-agent/tests/integration/` (~65 files) |
| Admin API | E2E + unit | `apps/admin-api/tests/e2e/`, `apps/admin-api/tests/unit/` |
| KB pipeline | E2E | `apps/admin-api/tests/e2e/test_kb_provider_e2e.py` |
| Billing | E2E | `apps/admin-api/tests/e2e/billing/` |
| Dashboard | Playwright | `apps/admin-dashboard/e2e/` |
| MCP sidecar | Provider tests | `apps/mcp-sidecar/tests/providers/` |

TCs with automated coverage are labeled `e2e-test`. Manual QA is still required for UX verification, visual correctness, and cross-browser testing.

---

## Product Context

### Architecture Overview

```
Phone/Mobile/Widget → LiveKit Room → Voice Agent (Python)
                                       ↓
                      VAD → Whisper (STT) → Claude/GPT (LLM) → Cartesia/Polly (TTS)
                                              ↓
                                    MCP Sidecar (Node.js) → ITSM/CS Provider API
```

### Apps in the Monorepo

| App | Tech | Purpose |
|-----|------|---------|
| `website` | Next.js 16 | Marketing site (aurionai.net, Vercel) |
| `admin-dashboard` | Next.js 16 | Admin panel + ops console (EKS) |
| `admin-api` | Python FastAPI | Backend API |
| `livekit-agent` | Python 3.13+ | Voice AI orchestration |
| `mcp-sidecar` | Node.js 24 | Provider-agnostic ITSM/CS bridge |
| `twilio-webhooks` | Python FastAPI | Twilio event handlers |
| `help-center` | Next.js 16 | Public KB portal |
| `widget` | Preact | Embeddable voice + chat widget |
| `mobile-app` | React Native (Expo) | End-user mobile app |
| `developer-portal` | Next.js | API documentation |

### ITSM Providers Supported

| Provider | Status | Key Difference |
|----------|--------|----------------|
| Freshservice | Production | Default; integer IDs, HTML responses |
| HaloITSM | Production | 4-tier categories, `value` field for categories |
| ServiceNow | Production | `sys_id` GUIDs, `u_*` custom fields, OAuth |
| JSM | Production | 3 API surfaces (JSM + Jira + Confluence), ADF rich text |

### CS Providers Supported

| Provider | Status | Key Difference |
|----------|--------|----------------|
| Freshdesk | Production | Integer IDs, 50 req/min rate limit |
| Zendesk | Production | String statuses, `hold` not settable via API, 400 req/min |
