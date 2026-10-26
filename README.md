# 👁️ Aarogya Drishti

**An AI-assisted oral cancer early-detection, telemedicine and patient-care platform for remote tribal communities.**

Built for the **AAROHAN Inter-Centre Hackathon 2026**, organised by **ITDA Paderu**, Alluri Sitharama Raju District, Andhra Pradesh, India.

> ⚠️ **HACKATHON DEMONSTRATION ONLY: NOT FOR CLINICAL USE.**
> This project uses **synthetic data only**. Any AI output is a research/demo label, is advisory, and must never be used to diagnose, stage or rule out cancer.

| | |
|---|---|
| **Event dates** | 27 October – 4 November 2026 |
| **Code freeze** | 4 November 2026, 10:00 AM IST |
| **Theme** | AI for Public Health · Technology for Tribal Communities · From Screening to Treatment |
| **Team** | `[Team Name]`, `[Member 1 – Member 5]`, `[Participating AAROHAN Centre]` |
| **Live demo** | `[Deployed HTTPS URL]` |
| **Demo video** | `[5-minute video link]` |

---

## Table of contents

1. [The problem](#1-the-problem)
2. [What Aarogya Drishti does](#2-what-aarogya-drishti-does)
3. [End-to-end patient workflow](#3-end-to-end-patient-workflow)
4. [Non-negotiable safety and design rules](#4-non-negotiable-safety-and-design-rules)
5. [Users, roles and permissions](#5-users-roles-and-permissions)
6. [Feature modules](#6-feature-modules)
7. [System architecture](#7-system-architecture)
8. [Technology stack](#8-technology-stack)
9. [Repository structure](#9-repository-structure)
10. [Case state machine](#10-case-state-machine)
11. [Database design](#11-database-design)
12. [API overview](#12-api-overview)
13. [Image quality and AI pipeline](#13-image-quality-and-ai-pipeline)
14. [Telugu and offline-first design](#14-telugu-and-offline-first-design)
15. [Security and privacy](#15-security-and-privacy)
16. [Testing strategy](#16-testing-strategy)
17. [Getting started](#17-getting-started)
18. [Configuration](#18-configuration)
19. [Deployment and recovery](#19-deployment-and-recovery)
20. [Screen map](#20-screen-map)
21. [Build plan (9 days)](#21-build-plan-9-days)
22. [Live demo script](#22-live-demo-script)
23. [Judging criteria mapping](#23-judging-criteria-mapping)
24. [Submission checklist](#24-submission-checklist)
25. [Known limitations](#25-known-limitations)
26. [Roadmap to a clinical pilot](#26-roadmap-to-a-clinical-pilot)
27. [AI tools and third-party disclosure](#27-ai-tools-and-third-party-disclosure)
28. [Team and acknowledgements](#28-team-and-acknowledgements)
29. [License](#29-license)

---

## 1. The problem

Oral cancer is a significant public-health concern, and early detection is difficult in remote tribal areas where access to specialists is limited. A villager with a persistent ulcer or white patch may be examined by a frontline health worker, but then:

- **Limited access:** specialists are scarce and far away.
- **Weak documentation:** oral findings and screening data are recorded inconsistently, often without photos.
- **Delays:** referrals, diagnostics and follow-up are slow.
- **No coordination:** nobody has a single view of the care journey, so patients are **lost between screening, referral and treatment**.

The gap is **coordination, not only detection**.

## 2. What Aarogya Drishti does

Aarogya Drishti is a secure, deployable digital platform that connects **village-level screening → doctor review → telemedicine → referral → diagnostics → treatment follow-up → district monitoring** in a single continuous patient record.

**Objectives**

1. Digital patient registration and screening documentation.
2. Image quality checks and research-only AI analysis.
3. Connected clinicians: health workers linked to Medical Officers and specialists.
4. Tracking of referrals, appointments, investigations and follow-up.
5. Secure records and district-level aggregate monitoring.
6. Telugu/English UI with offline workflows for low connectivity.

**Before vs. after**

| Before (disconnected) | After (one connected platform) |
|---|---|
| Paper notes, no standard photos | Registration + consent + guided photo capture |
| Referral by phone, no tracking | Structured referral and appointment booking |
| No view of attendance | Attendance and missed-visit tracking |
| Patient lost to follow-up | Navigator tasks, reminders and escalation |
| No overview for administrators | Aggregate district dashboard |

## 3. End-to-end patient workflow

```
Register patient → Consent → Medical history → Oral photo capture → Image quality check
   → Research-demo AI (advisory) → Independent Medical Officer review
   → Specialist referral → Video / physical consultation → Simulated investigation
   → Clinician-entered result → Navigation & follow-up → Aggregate dashboard
```

**Exception branches**

| Situation | System behaviour |
|---|---|
| Poor-quality image | Request recapture |
| AI service unavailable | Workflow continues without AI; doctor reviews manually |
| Specialist opinion needed | Medical Officer creates a referral |
| Missed appointment | A navigator follow-up task is created automatically |
| Network failure | Draft saved offline, synced once on reconnect |

> **No AI result can bypass clinician review or advance a case by itself.**

## 4. Non-negotiable safety and design rules

These rules are built into the design and covered by tests.

1. AI output is **advisory** and stored **separately** from the signed clinician conclusion.
2. The app **never confirms or stages cancer by itself** and **never tells a patient they are cancer-free**. Only a clinician-entered pathology result can mark a case as confirmed (in the simulation).
3. Every mock or research model output carries the label **NOT FOR CLINICAL USE**, the **model version** and a **timestamp**.
4. The clinical workflow **keeps working if the AI service is down**.
5. Every API call is **authenticated and authorized at patient/facility level**, not just by role (**deny by default**).
6. **No bounding boxes or heatmaps** unless a real supervised localization model exists.
7. **No secrets in Git.** Only `.env.example` with placeholders is committed.
8. Offline records are marked **synced only after server acknowledgement**.

**Disqualifier checklist:** real patient data, presenting AI as clinical, exposed records, plagiarism, hardcoded secrets.

## 5. Users, roles and permissions

| Capability | Health Worker | Medical Officer | Specialist | Navigator | District Admin |
|---|---|---|---|---|---|
| Register patient, consent | Yes (own facility) | No | No | No | No |
| Screening intake, photos | Yes (assigned) | View | View referred only | No | No |
| Run AI analysis | Yes | View | View | No | No |
| Clinical review and sign | No | **Yes** | Consult notes only | No | No |
| Create referral | No | Yes | Yes (onward) | No | No |
| Book video/physical appointment | No | Yes | Confirm | Yes | No |
| Investigations / pathology result | No | Order/view | **Yes** | View status only | No |
| Enter final diagnosis | No | Yes | Yes | No | No |
| Navigation tasks | View assigned follow-up | View | No | **Yes** | No |
| View images | Own cases | Assigned cases | Referred cases | No (unless needed) | No |
| Dashboard | No | Facility summary | No | Own queue | **Aggregates only** |
| Audit log | No | No | No | No | Aggregate/system view |

MFA is required for Medical Officer, Specialist and District Admin accounts.

## 6. Feature modules

| Module | Description | Done when |
|---|---|---|
| **A. Registration & history** | Multi-step Telugu/English form, exposure and symptom checklist, consent gating, duplicate check (name + age + village + phone). `patient_code` generated server-side. | Patient and visit saved; duplicate warning shown |
| **B. Smartphone screening** | Guided camera with framing overlay, preview, retake, site tag, quality check. | Real phone captures 5 views; blurry image rejected |
| **C. AI assessment** | Pluggable adapter, mock model, stored outputs, warnings, timeouts, failure handling. | Image → AI → persisted result; AI down does not block |
| **D. MO & specialist review** | Role-scoped queue, case viewer with zoom, signed review, referral. | Doctor overrides AI and referral is created |
| **E. Video consultation** | Slots, booking, per-user time-limited tokens, recording off. | Two accounts join; expired token denied |
| **F. Physical appointment** | Specialty, hospital, slot, printable Telugu-capable slip, missed-visit flow. | Slip prints in Telugu; missed flow works |
| **G. Diagnostics** | Investigation state machine, report upload, clinician-only diagnosis. | Simulated biopsy → pathology → diagnosis |
| **H. Longitudinal record** | Single timeline of all patient events with actor, time and version. | Full journey visible in one timeline |
| **I. Navigation** | Task queue, contact log, overdue escalation, transport support. | Overdue shown and escalated |
| **J. District dashboard** | Aggregate cards, filters, CSV export with no identifiers. | Seeded 12 cases / 4 referrals show exact counts |

## 7. System architecture

```
 ┌───────────────────────── Client applications (PWA) ─────────────────────────┐
 │ Health Worker (mobile) · Medical Officer · Specialist · Navigator · Admin   │
 └──────────────────────────────────┬──────────────────────────────────────────┘
                                    │ HTTPS (REST)
                            ┌───────▼────────┐
                            │  FastAPI API   │  authentication · RBAC + patient/facility checks
                            │   (gateway)    │  validation · idempotency · audit middleware
                            └───┬─────┬──────┘
          ┌─────────────────────┘     │      └──────────────────────────┐
   ┌──────▼───────┐          ┌────────▼─────────┐              ┌─────────▼────────┐
   │ PostgreSQL   │          │ Private object   │              │  AI service      │
   │ (app + audit)│          │ storage (images, │              │  OpenCV QC +     │
   └──────────────┘          │ reports, signed  │              │  mock/research   │
                             │ short-lived URLs)│              │  adapter (can    │
                             └──────────────────┘              │  fail safely)    │
                                                               └──────────────────┘
   External: Video SDK (time-limited tokens) · SMS/email notifications (non-clinical text)
```

**Data flow:** Health worker PWA → authenticated FastAPI → PostgreSQL + private image storage → AI adapter (research only) → stored AI observation → Medical Officer → video or physical specialist → diagnostics → navigator → aggregate dashboard.

**Key architectural properties**

- **Fault isolation:** AI down ≠ workflow down. Manual review continues.
- **Security boundary:** authentication and patient-level checks at the API; private storage; no public links.
- **Private vs. aggregate:** images and records stay private; the dashboard shows aggregates only.

## 8. Technology stack

| Layer | Choice |
|---|---|
| Frontend | React + TypeScript + Vite PWA, `react-i18next` (Telugu/English) |
| Backend | Python, FastAPI, SQLAlchemy, Alembic, Pydantic |
| Database | PostgreSQL |
| Image storage | Private S3-compatible bucket (MinIO locally), short-lived signed URLs |
| Image QC | OpenCV (blur, brightness, glare) |
| AI | Pluggable adapter: `mock-v1` by default, optional EfficientNet-B0 research model |
| Video | Approved SDK (e.g. Jitsi JaaS, Daily, LiveKit) with time-limited tokens |
| Offline | Service worker + IndexedDB queue with idempotency keys |
| Deployment | Docker Compose locally; managed container/VM + managed Postgres online |
| Monitoring | Structured logs (no PHI), `/healthz`, uptime check |
| Testing | pytest, API integration tests, Playwright (mobile E2E) |

## 9. Repository structure

```
aarogya-drishti/
├── apps/
│   └── web/                # PWA: health worker, clinician, navigator, admin UI
├── services/
│   ├── api/                # FastAPI, auth, workflow, Alembic migrations
│   └── ai/                 # quality checker + mock/research adapters
├── infra/                  # docker-compose, deployment manifests
├── docs/                   # PRD, architecture, ER, API, threat model, test report
├── tests/                  # unit, integration, E2E, security
├── .env.example
└── README.md
```

## 10. Case state machine

```
DRAFT → CONSENTED → SCREENED → AWAITING_CLINICIAN → REVIEWED → REFERRAL_PLANNED
      → APPOINTED → ASSESSED → DIAGNOSTICS_PENDING → RESULT_REVIEWED → CARE_LINKED | CLOSED

Tracked exceptions: RESCHEDULED · MISSED · DECLINED · LOST_CONTACT
```

| Transition | Allowed actor | Guard |
|---|---|---|
| DRAFT → CONSENTED | Health worker | Imaging consent granted |
| CONSENTED → SCREENED | Health worker | Screening saved; at least one acceptable image or manual note |
| SCREENED → AWAITING_CLINICIAN | System | Submitted for review |
| AWAITING_CLINICIAN → REVIEWED | Medical Officer | Review signed |
| REVIEWED → REFERRAL_PLANNED | Medical Officer | Decision is referral |
| REFERRAL_PLANNED → APPOINTED | MO / Navigator | Appointment CONFIRMED |
| APPOINTED → ASSESSED | Specialist | Appointment COMPLETED + outcome |
| ASSESSED → DIAGNOSTICS_PENDING | Specialist | Investigation ordered |
| DIAGNOSTICS_PENDING → RESULT_REVIEWED | Clinician | Report received and reviewed |
| RESULT_REVIEWED → CARE_LINKED / CLOSED | Clinician / Navigator | Diagnosis entered by clinician |

Any transition not in this table is rejected with **HTTP 409**. No AI result may move a case forward.

## 11. Database design

PostgreSQL with UUID primary keys. Core tables:

| Table | Purpose | Key rules |
|---|---|---|
| `facilities` | PHC, hospital, diagnostic centre | Mandal/PHC filters |
| `users` | Accounts and roles | Unique login, active flag, MFA for privileged |
| `patients` | Demographics and residence | UUID, duplicate hint, PII restricted |
| `consents` | Purpose-specific consent | **Append-only**, versioned |
| `screenings` | One screening visit | Clinician findings separate from AI |
| `images` | Oral photo metadata | Private key, checksum, consent gate |
| `ai_analyses` | Model outputs | `clinical_use = false` enforced by CHECK constraint |
| `clinical_reviews` | MO signed review | Versioned; only clinicians can finalize |
| `referrals` | Specialist referral | Explicit clinical decision |
| `appointments` | Video and physical visits | Status history, no public links |
| `appointment_status_history` | Status transitions | **Append-only** |
| `investigations` | Biopsy, pathology, imaging | Separate from AI label |
| `diagnoses` | Clinician-entered result | Clinician roles only |
| `navigation_tasks` | Follow-up work items | Overdue escalation |
| `contact_attempts` | Navigator calls/visits | Linked to task |
| `idempotency_keys` | Safe retries | Unique per user + key |
| `case_state_history` | Case state transitions | **Append-only**, enforced transitions |
| `audit_events` | Who did what | **Append-only**, no PHI in metadata |

**Relationships:** `facilities 1-N users | patients` · `patients 1-N consents | screenings` · `screenings 1-N images | clinical_reviews` · `images 1-N ai_analyses` · `clinical_reviews 1-N referrals` · `referrals 1-N appointments | investigations | navigation_tasks` · `investigations 1-N diagnoses` · `navigation_tasks 1-N contact_attempts`.

Append-only tables (`audit_events`, `consents`, `case_state_history`, `appointment_status_history`) are protected by revoking `UPDATE`/`DELETE` from the app database role, or by a trigger that raises an exception.

The full DDL lives in `docs/` and `services/api/migrations/`.

## 12. API overview

All routes require authentication, patient/facility authorization, validation, pagination for lists, standard error codes, and an `Idempotency-Key` header on retryable writes.

| Method + route | Purpose | Who |
|---|---|---|
| `POST /auth/login` | Session/JWT with expiry | All |
| `POST /auth/mfa/verify` | MFA for privileged users | MO, specialist, admin |
| `GET /me` | Current user, role, facility | All |
| `POST /patients` | Register patient (returns duplicate warnings) | Health worker |
| `GET /patients/{id}` | Longitudinal record | Assigned care team |
| `POST /patients/{id}/consents` | Purpose-specific consent | Health worker |
| `POST /screenings` | Create screening + history (accepts `client_uuid`) | Health worker |
| `POST /screenings/{id}/images` | Upload photo (needs consent) | Assigned health worker |
| `GET /images/{id}/url` | Short-lived signed URL, audited | Authorized clinicians |
| `POST /ai/analyze` | Queue non-clinical analysis | Screening user |
| `GET /ai/analyses?image_id=` | AI history | Care team |
| `GET /queue` | Role-scoped work queue | MO, specialist, navigator |
| `POST /reviews` | Save and sign clinical review | Medical Officer |
| `POST /referrals` | Create referral + printable slip | Medical Officer |
| `GET /slots` | Available appointment slots | MO, navigator |
| `POST /appointments` | Book / request | MO, navigator |
| `PATCH /appointments/{id}` | Confirm, reschedule, cancel, attendance, outcome | Clinician / navigator |
| `POST /appointments/{id}/videotoken` | Time-limited token | Participants only |
| `POST /investigations` | Order biopsy/pathology | Specialist, MO |
| `PATCH /investigations/{id}` | Update state, attach report | Specialist |
| `POST /diagnoses` | Clinician-entered result | MO, specialist |
| `PATCH /navigation/{id}` | Contact attempt, status | Navigator |
| `GET /dashboard/summary` | Aggregate counts and rates | District admin |
| `GET /dashboard/export` | Aggregate export only | District admin |
| `GET /healthz` | Health check | Public |

**Standard AI response (research demo)**

```json
{
  "analysis_id": "DEMO-AI-009",
  "screening_id": "DEMO-CASE-001",
  "image_quality": "acceptable",
  "provider": "mock",
  "model_version": "mock-v1",
  "research_label": "suspicious_example",
  "score": null,
  "clinical_use": false,
  "requires_clinician_review": true,
  "warning": "NOT FOR CLINICAL USE"
}
```

Interactive OpenAPI docs are served by FastAPI at `/docs` once the API is running.

## 13. Image quality and AI pipeline

**Pipeline:** Image upload → validation → quality assessment (OpenCV) → mock/research AI adapter → structured output → Medical Officer review.

**Image checks (`services/ai`)**

1. Validate extension and decode with Pillow/OpenCV; reject non-images (e.g. a renamed PDF).
2. Enforce max size (e.g. 8 MB) and minimum dimensions.
3. Fix EXIF orientation, strip metadata, compute SHA-256.
4. **Blur:** variance of Laplacian below threshold → recapture.
5. **Brightness:** mean grayscale too low or too high → recapture.
6. **Glare:** fraction of saturated pixels above threshold → recapture.
7. Thresholds are calibrated on the demo images and recorded in `docs/`.

**AI adapter levels**

| Level | What | When |
|---|---|---|
| 1 (required) | Mock adapter with fixed demo labels | Day 4 |
| 2 (optional) | EfficientNet-B0 fine-tuned on a licensed dataset | Only if the core system is complete |
| 3 (never) | Real clinical use | **Not allowed** |

**If a research model is trained:** include `DATASET_CARD.md`; remove corrupt files; split original source images into train/val/test **before** augmentation (no leakage); fine-tune on train only; report sensitivity, specificity, precision, F1, confusion matrix and sample sizes; include `MODEL_CARD.md` with limitations. Output labels stay research-only.

**What the AI will never do:** show a score as a probability, draw heatmaps or boxes, say "healthy" or "cancer", or make accuracy/validation claims.

## 14. Telugu and offline-first design

| Area | Implementation |
|---|---|
| i18n | `react-i18next` with `te.json` and `en.json`; no hardcoded English; toggle always visible |
| Fonts | Noto Sans Telugu bundled locally; tested on Android and in PDF |
| Usability | 44px+ touch targets, icon + label buttons, progress steps, saved confirmation, minimal typing |
| Offline storage | IndexedDB drafts; browser storage is not fully protected, so payloads are encrypted with WebCrypto where feasible |
| Sync queue | Each write has a client UUID + `Idempotency-Key`; states: pending → uploading → confirmed / failed |
| Conflicts | Resolution screen; clinician notes are never discarded |

**Jury test:** airplane mode → create patient and draft → close/reopen → reconnect → sync once → force retry → **exactly one** server record.

## 15. Security and privacy

| Area | Requirement |
|---|---|
| Auth | Argon2/bcrypt, short session expiry, refresh handling, MFA for MO/specialist/admin, no shared accounts |
| Authorization | Role **and** patient/facility check in every query; deny by default; tests for each endpoint |
| Transport & storage | HTTPS, encrypted DB and backups, private bucket, signed URLs under 5 minutes |
| Uploads | Decode, size limit, safe filenames, no path traversal, strip metadata |
| Audit | Log image views, exports, clinical edits, diagnosis, permission changes; no PHI in logs |
| Secrets | Env vars or secret manager; `.env` is gitignored; rotate after the event |
| Privacy UI | No patient names in browser titles, notifications or URLs; session-timeout screen |
| Governance | Synthetic data only; demo reset script; disable test users after the event; DPDP Act and telemedicine compliance in the roadmap |

**Threats covered:** stolen phone, expired/replayed token, IDOR, mass assignment, leaked signed URL, malicious upload, shared video link, backup leak.

See `docs/THREAT_MODEL.md` for details.

## 16. Testing strategy

**Approach:** vertical slices, each with passing tests before moving on. Security tests are written failing first, then fixed.

| Test | Input | Expected |
|---|---|---|
| Normal journey | Consent → screening → photo → review → video → biopsy | One continuous record |
| Blurred photo | Defocused image | Recapture warning |
| Invalid file | PDF renamed `.jpg` | Rejected after decode |
| Missing consent | Upload before consent | Rejected with explanation |
| Wrong patient | DoctorA opens unrelated case | 403 |
| AI down | Stop AI container | Case intact; manual review works |
| Network loss | Disconnect during upload | Retry, no duplicate image |
| Double submit | Same POST + same key | Single record |
| False-negative AI | AI low, clinician suspicious | Referral still possible |
| False-positive AI | AI suspicious, clinician disagrees | Discrepancy documented |
| Missed appointment | Mark missed | Navigation task created |
| Bad video token | Expired or wrong token | Join denied; reschedule possible |
| Long Telugu text | Switch language, PDF, mobile | No overflow |
| Concurrent edit | Two users edit one review | Conflict/version error |
| Dashboard | 12 cases, 4 referrals seeded | Exact counts |
| Audit | Change pathology status | Actor/time/action stored |
| Invalid transition | SCREENED → CARE_LINKED | 409 |
| Oversized image | Over limit | 413 |
| Traversal filename | `../../x.jpg` | Sanitized or rejected |

**Evidence produced:** pytest unit tests, API integration tests, Playwright mobile E2E, authorization tests, offline demo recording, and a known-issues list. Run with:

```bash
# planned commands (created during the build)
make test            # backend unit + integration
make test-e2e        # Playwright mobile E2E
make test-security   # authorization and abuse-case tests
```

## 17. Getting started

> The repository starts empty. Follow these steps to scaffold the project, then use the `make` targets as they are added.

### Prerequisites

- Git, Docker + Docker Compose
- Node.js 20+ and npm
- Python 3.11+

### Scaffold

```bash
git init aarogya-drishti && cd aarogya-drishti
mkdir -p apps/web services/api services/ai docs tests infra

# Frontend
cd apps && npm create vite@latest web -- --template react-ts
cd web && npm install && cd ../..

# Backend
cd services/api && python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install fastapi 'uvicorn[standard]' sqlalchemy alembic \
            'psycopg[binary]' pydantic-settings python-multipart
pip freeze > requirements.txt      # pin and review versions
```

### Run the full stack

```bash
cp .env.example .env               # fill in placeholders locally; never commit .env
docker compose up --build -d       # web, api, ai, postgres, minio
# run Alembic migrations, then seed synthetic data
make seed
curl http://localhost:8000/healthz # expect HTTP 200
```

### Recommended build order

1. Docker stack + health check + migrations + seed data
2. Auth, roles, facility scoping, audit middleware
3. Patient, consent, screening APIs and Telugu/English forms
4. Image upload, quality checks, mock AI, stored results
5. Review queue, signed review, referral
6. Appointments, video tokens, physical slip
7. Investigations, diagnosis, navigation tasks
8. Timeline and dashboard aggregates
9. Offline sync, security tests, deployment, documentation

### Demo accounts (synthetic, for evaluators)

| Role | Login | Password |
|---|---|---|
| Health Worker | `[to be added]` | `[to be added]` |
| Medical Officer | `[to be added]` | `[to be added]` |
| Specialist | `[to be added]` | `[to be added]` |
| Navigator | `[to be added]` | `[to be added]` |
| District Admin | `[to be added]` | `[to be added]` |

Do **not** commit real credentials. Share evaluator accounts through the submission form and disable them after the event.

## 18. Configuration

All configuration comes from environment variables. Commit only `.env.example`.

| Variable | Purpose | Example (placeholder) |
|---|---|---|
| `DATABASE_URL` | PostgreSQL connection string | `postgresql+psycopg://user:CHANGE_ME@db:5432/aarogya` |
| `JWT_SECRET` | Token signing key | `CHANGE_ME` |
| `ACCESS_TOKEN_TTL_MIN` | Session expiry | `15` |
| `S3_ENDPOINT` | Object storage endpoint | `http://minio:9000` |
| `S3_BUCKET` | Private bucket name | `aarogya-private` |
| `S3_ACCESS_KEY` / `S3_SECRET_KEY` | Storage credentials | `CHANGE_ME` |
| `SIGNED_URL_TTL_SEC` | Signed URL lifetime (under 300) | `120` |
| `AI_PROVIDER` | `mock` or `research` | `mock` |
| `AI_SERVICE_URL` | AI service address | `http://ai:8100` |
| `VIDEO_PROVIDER_*` | Video SDK keys and room settings | `CHANGE_ME` |
| `VIDEO_TOKEN_TTL_MIN` | Per-user video token expiry (15–30) | `20` |
| `MAX_UPLOAD_MB` | Image size limit | `8` |
| `CORS_ORIGINS` | Allowed web origins | `https://your-demo-url` |

## 19. Deployment and recovery

Free hosting tiers may sleep or wipe files. Use a **persistent database and object storage**, and keep a **local Docker route as backup**.

| Step | Action |
|---|---|
| 1 | `docker compose up --build -d` (web, api, ai, postgres, minio) |
| 2 | Run Alembic migrations, then seed synthetic data (`make seed`) |
| 3 | Verify `GET /healthz` returns 200 |
| 4 | Open the HTTPS URL on Android; test camera, reload, offline |
| 5 | Restart the API; confirm data persists |
| 6 | Confirm the bucket is private and signed URLs expire |
| 7 | Run automated tests against the deployed URL |
| 8 | Take a DB backup and restore into a fresh environment |
| 9 | Provide `make reset` for demo data |
| 10 | After the event: disable test users, rotate secrets |

## 20. Screen map

Every screen has an ID. Health worker (HW) screens are mobile-first; Medical Officer, Specialist, Navigator and Admin screens (MO, SP, NV, AD) target laptop/tablet but must work on a phone.

**Health worker (10 screens, 1 retake loop)**

`HW-01 Login → HW-02 Home → HW-03 Register → HW-04 Consent → HW-05 History → HW-06 Camera → HW-07 Photo check (blurry/dark → back to HW-06) → HW-08 AI result → HW-09 Submit → HW-10 Sync → MO-01 Queue`

**Medical Officer and specialist (12 screens, 1 decision)**

`MO-01 Queue → MO-02 Case view → MO-03 Review (sign) → Decision? → [refer] MO-04 Referral → MO-05 Book visit → SP-01 Queue → SP-02 Consult → SP-03 Order test → SP-04 Report → SP-05 Diagnosis (sign) → NV-01 Follow-up`
Decision branches: *recapture* → HW-06; *routine* → reminder only.

**Navigator and district admin (5 screens, 1 decision)**

`NV-01 Task queue → NV-02 Task detail → Attended? → [yes] NV-03 Treatment | [no] MO-05 Reschedule`
`AD-01 Dashboard → AD-02 Export`

**Cross-cutting:** `TL-01 Timeline` is a read-only longitudinal view reachable from any case.

**Design rules**

- Mobile: top bar (back, title, TE/EN toggle, sync badge), progress dots, one content card, sticky 48px+ full-width Next button.
- Touch targets 44px+, body text 16px+, icon + label on every button.
- Status colours: grey draft, blue pending, green confirmed/signed, amber warning/AI advisory, red failed/urgent. **Never colour a patient as "cancer" or "healthy".**
- AI card is always amber with *NOT FOR CLINICAL USE*, model version and timestamp; no score, heatmap or box.
- Clinician-signed items show a green badge with doctor name, time and version.
- Visible "Saved" confirmation after every write; clear retry on failure; offline banner when disconnected.
- No patient names in browser titles, notifications or URLs.

## 21. Build plan (9 days)

| Day | Date | Goal | Highlights |
|---|---|---|---|
| 1 | 27 Oct | Plan | Repo, PRD, schema draft, wireframes, i18n keys, risk register, CI setup |
| 2 | 28 Oct | Foundation | FastAPI skeleton, migrations, login, RBAC, PWA shell, language toggle, Docker stack, healthz |
| 3 | 29 Oct | Registration | Patient/consent/screening APIs, Telugu registration and checklist, adapter contract, auth tests |
| 4 | 30 Oct | Images & AI | Upload, private storage, signed URLs, camera/retake, OpenCV QC, mock adapter, AI panel |
| 5 | 31 Oct | Review | Review, referral, state machine, audit, offline draft queue, MO queue and case viewer. **Minimum end-to-end system working.** |
| 6 | 1 Nov | Video & booking | Appointments, slots, token endpoint, sync status UI, optional research model, two-device video test |
| 7 | 2 Nov | Diagnostics & dashboard | Investigations, diagnoses, tasks, aggregates, navigator, timeline, Telugu polish, model/dataset cards |
| 8 | 3 Nov | Harden & deploy | Bug fixes, idempotency, backups, real-phone testing, AI-down drill, Telugu PDF, staging tests, test report |
| 9 | 4 Nov | Demo | **Code freeze 10:00 AM IST**, final checks, rehearsal, backup route, final live demo and awards |

**Team practice:** 15-minute daily stand-up, merge twice a day, daily development log with commits.

## 22. Live demo script

10 minutes:

1. Open the deployed app in Telugu on a phone.
2. Register a synthetic patient: 46-year-old from G. Madugula, tobacco chewing, persistent white cheek patch.
3. Record consent and symptoms.
4. Upload a good demo image, then a blurry one (recapture shown).
5. Run the mock AI; stop the AI service and show graceful recovery.
6. Log in as Medical Officer, review, disagree with AI if relevant, sign the decision.
7. Book a video consultation and join with a second account.
8. Create a physical referral, mark attendance, upload simulated pathology.
9. Specialist enters the final simulated diagnosis.
10. Navigator task updated; dashboard counts correct.
11. Show unauthorized access blocked, audit trail, and offline sync.
12. Explain limitations and the roadmap to an approved clinical pilot.

## 23. Judging criteria mapping

| Criterion | Marks | How this project earns points |
|---|---|---|
| End-to-end workflow and integration | 20 | Full journey with one continuous record |
| AI engineering, image quality, safe integration | 15 | Quality checks, versioned adapter, non-clinical labels, AI-down resilience |
| MO review, video, appointments | 15 | Signed review, working two-party call, status tracking |
| Records, diagnostics, navigation | 15 | Timeline, investigation states, navigator follow-up |
| Security, privacy, access | 15 | Patient-level authorization, audit, private images, MFA |
| Telugu and offline reliability | 10 | Full Telugu flow, offline sync with no duplicates |
| Deployment, testing, documentation | 10 | Live URL, tests, reset script, complete docs |

**Tie-breakers:** patient workflow, then security/privacy, then AI safety/integration. A safety violation can disqualify a team regardless of technical score.

## 24. Submission checklist

- [ ] Git repo with daily commits and contribution record
- [ ] Deployed HTTPS demo URL + evaluator test accounts
- [ ] README (install, architecture, stack, credentials, known limits)
- [ ] Architecture diagram with trust boundaries; ER diagram
- [ ] Role-access matrix
- [ ] OpenAPI docs with examples
- [ ] Environment variable table (no secrets)
- [ ] AI model card, dataset card, validation and limitations
- [ ] Threat model
- [ ] Test report with failure cases and offline tests
- [ ] Deployment, backup and rollback guide
- [ ] Synthetic dataset + reset command
- [ ] Telugu UI demonstration
- [ ] 5-minute demo video
- [ ] Disclosure of AI tools, templates, third-party code
- [ ] Roadmap from demo to approved clinical pilot
- [ ] Live presentation on 4 November 2026

## 25. Known limitations

- Demonstration with **synthetic data only**; not validated for any clinical purpose.
- The default AI is a **mock adapter** with fixed demo labels. Any optional research model has no clinical validation.
- Image quality checks flag blur, brightness and glare. They **cannot prove a lesion is absent**.
- No bounding boxes, heatmaps or probability scores are shown.
- Browser-side offline storage is not fully protected; WebCrypto encryption is applied where feasible.
- Notification delivery (SMS/email) is simulated or limited, and reminder text deliberately contains no clinical details.
- Free-tier hosting may sleep; a local Docker route is provided as backup.

Update this section with the actual known-issues list before submission.

## 26. Roadmap to a clinical pilot

1. Clinical governance: engage the state health department, obtain ethics approval and define the pilot protocol.
2. Compliance review: India's DPDP Act, telemedicine practice guidelines, and medical-device/software regulations as applicable.
3. Data: collect consented, locally representative, ethically sourced data with expert-labelled ground truth.
4. Model validation: prospective evaluation with sensitivity/specificity reporting, bias analysis across populations and devices; no deployment without regulatory approval.
5. Integration: ABDM/health-facility integration, official SMS gateways, production-grade MFA and key management.
6. Operations: formal security audit and penetration test, DR drills, local-language usability testing with ASHA/ANM workers.
7. Scale: phased rollout across PHCs with district monitoring.

## 27. AI tools and third-party disclosure

The hackathon is open-book and permits AI-assisted coding with proper disclosure. Complete this section before submission.

| Item | Details |
|---|---|
| AI coding assistants used | `[e.g. Antigravity, other]` and what they were used for |
| Templates / boilerplate | `[list]` |
| Third-party libraries | See `package.json` and `requirements.txt` |
| Datasets / images | `[source, licence]`; **synthetic or licensed demo images only** |
| Fonts | Noto Sans Telugu (SIL Open Font License) |

All AI-generated code is reviewed, tested and committed only after acceptance. No real records or secrets were shared with any AI tool.

## 28. Team and acknowledgements

**Team `[Team Name]`**, `[Centre]`

| Name | Role / focus |
|---|---|
| `[Member 1]` | Engineering lead (repo, schema, API) |
| `[Member 2]` | Frontend (PWA, i18n, offline) |
| `[Member 3]` | AI / image quality |
| `[Member 4]` | Clinical UI (MO, specialist, navigator) |
| `[Member 5]` | QA / DevOps |

**Acknowledgements**

- **ITDA Paderu**, Alluri Sitharama Raju District, for organising AAROHAN 2026.
- **AAROHAN** (AI & Software Skills for a Brighter Tomorrow) training centres.
- Inspiration: **IISc TANUH** and the **Aarogya Aarohan** oral cancer screening concept using smartphone images and AI-assisted assessment.

## 29. License

`[Choose a license, e.g. MIT or Apache-2.0, and add a LICENSE file.]`

Datasets and demo images carry their own licences, which are listed in the dataset card.

---

*Technology for People. Health for a Brighter Tomorrow.*
*Hackathon demonstration only. NOT FOR CLINICAL USE.*
