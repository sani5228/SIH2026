# Product Requirements Document
# Digital Farmer Procurement & Queue Management Platform

**Document Version:** 1.0
**Status:** Draft for Development
**Owner:** Product Management

---

## Table of Contents

1. Executive Summary
2. Problem Statement
3. Vision
4. Goals
5. Non-Goals
6. Stakeholders
7. User Roles
8. Personas
9. User Journeys
10. Functional Requirements
11. Feature Specifications (Farmer / Operator / Admin Apps)
12. Slot Booking Engine
13. Real-Time Queue Management
14. Procurement Module
15. Payment Tracking
16. Notification System
17. Database Architecture
18. API Specification
19. System Architecture
20. Security & Privacy
21. Performance & Scalability
22. Offline / Low-Connectivity Strategy
23. Analytics & Reporting
24. Testing Strategy
25. Deployment & Infrastructure
26. Monitoring & Observability
27. Backup & Disaster Recovery
28. Development Phases
29. MVP Definition
30. Future Scope
31. User Stories
32. Acceptance Criteria
33. Team Responsibilities
34. Cost Considerations
35. Risks & Mitigations
36. Success Metrics
37. Glossary
38. Appendices
39. Ready for Development Checklists
40. Next Development Prompts

---

## 1. Executive Summary

The **Digital Farmer Procurement & Queue Management Platform** ("the Platform") is a multi-interface digital system that lets farmers register, book procurement slots at designated centres, track their queue position in real time, and follow their produce through verification, weighing, grading, procurement, and payment status — while procurement centre operators, centre managers, administrators, and government officials get dedicated tools to run and monitor the process.

The Platform digitizes scheduling, queueing, record-keeping, and communication, while keeping physical verification, weighing, quality inspection, and produce handover at the centre — it does not attempt to replace in-person procurement, only to remove the friction and uncertainty around it.

This PRD is written to be **development-ready**: every module includes data model, API surface, workflow, UX requirements, and acceptance criteria sufficient for an engineering team (or a sequence of AI-assisted development prompts) to implement the system without further product clarification for the vast majority of cases. Where domain-specific values (prices, subsidy amounts, legal/regulatory rules) are not specified, they are explicitly called out as **admin-configurable** rather than invented.

## 2. Problem Statement

Farmers currently must travel to a physical procurement centre and wait — often for hours — with no visibility into their position in the queue, no advance knowledge of centre load, and no digital record of their transaction until it is manually completed. During peak season this produces:

- Large physical crowds and unsafe congestion at centres
- Traffic build-up around centre access roads
- Long, unpredictable waiting times for farmers
- Manual, error-prone queue management by centre staff
- No visibility for farmers into queue status or expected wait
- Communication gaps (farmers do not know if a centre is closed, full, or delayed)
- Fragmented or paper-based procurement records, prone to loss and duplication
- Delayed or opaque payment status, causing farmer distress and repeated inquiries
- High administrative overhead for staff reconciling paper logs

## 3. Vision

A farmer should be able to plan a produce sale from home: pick a centre and a slot, arrive at (close to) their scheduled time, complete a short physical verification/weighing/grading process, and leave with a digital receipt and a transparent, trackable payment status — with government and administrative stakeholders able to see procurement activity, load, and payment health across all centres in real time.

## 4. Goals

- **G1.** Let farmers book procurement slots in advance and avoid unplanned queuing.
- **G2.** Give farmers a live, self-service view of their queue position and estimated wait.
- **G3.** Digitize the procurement record (weight, grade, rate, amount) at the point of transaction.
- **G4.** Provide transparent, always-available payment-status tracking (not necessarily payment processing).
- **G5.** Reduce manual paperwork and duplicate/incorrect entries through structured data capture.
- **G6.** Give administrators and government officials real-time and historical visibility into procurement, queue, and payment activity across centres.
- **G7.** Work acceptably on low-end devices and unreliable network connections, in at least English and Hindi.
- **G8.** Be auditable: every state-changing action must be traceable to a user, role, and timestamp.

## 5. Non-Goals

- **NG1.** The Platform does not set or determine minimum support prices, subsidy rules, or any government policy value — these are admin-configured inputs, not product logic.
- **NG2.** The Platform does not perform bank-to-bank payment processing by default; it tracks payment status and exposes integration points for a future payment gateway/banking system.
- **NG3.** The Platform does not replace physical verification, weighing, or quality inspection — these remain human, in-person activities that the Platform records.
- **NG4.** The Platform does not attempt full offline operation of every workflow — some operations intentionally require live connectivity (see Section 22).
- **NG5.** Legal/regulatory compliance (e.g., mandated e-KYC, data-residency law) is out of scope for this PRD and must be reviewed by the adopting organization's legal team before production launch.

## 6. Stakeholders

| Stakeholder | Interest |
|---|---|
| Farmers | Fast, transparent, low-friction procurement experience |
| Procurement Centre Operators | Efficient, low-error daily operations tooling |
| Centre Managers | Capacity planning, staff oversight, local reporting |
| Administrators | System-wide configuration, user/centre management, reporting |
| Government/Department Officials | Oversight, statistics, anomaly monitoring, policy input |
| Engineering Team | Build and maintain the system against clear specifications |
| Support/Helpdesk | Resolve farmer and operator issues, manage complaints |


## 7. User Roles

### 7.1 Farmer
Registration, login, profile, farm/land details, produce details, centre selection, slot booking, cancellation/reschedule, digital token, live queue tracking, notifications, procurement history, payment tracking, digital receipts, complaints, feedback.

### 7.2 Procurement Centre Operator
Login, view today's bookings, view live queue, call next farmer, verify identity, inspect produce, enter weight, enter quality/grade, calculate amount, confirm procurement, update procurement/payment status, handle cancellations and walk-ins, generate shift-level reports.

### 7.3 Centre Manager (superset of Operator)
Manage centre capacity, manage slot templates, assign/deactivate operators, monitor daily procurement and queue in real time, override selected operational settings (e.g., emergency slot closure), generate centre-level reports.

### 7.4 Administrator
User management (all roles), farmer management, centre management, operator/manager management, slot configuration, procurement/crop/price configuration, notification template configuration, system-wide reports, audit log access, global system configuration.

### 7.5 Government/Department Official (read-mostly)
View procurement statistics, monitor centres and regions, monitor farmer participation, monitor aggregate payment status, view/export regional reports, view anomaly/exception flags. No write access to operational data.

### 7.6 System/Operations (non-human actor)
Background jobs, notification dispatch, queue-state reconciliation, scheduled reports, backup jobs — represented in audit logs as a `system` actor.

**Role → Permission Matrix (summary)**

| Capability | Farmer | Operator | Manager | Admin | Gov. Official |
|---|:---:|:---:|:---:|:---:|:---:|
| Manage own profile | ✅ | – | – | – | – |
| Book/cancel own slot | ✅ | – | – | – | – |
| View own queue position | ✅ | – | – | – | – |
| Check-in / call / process farmer | – | ✅ | ✅ | – | – |
| Enter weight/grade/procurement | – | ✅ | ✅ | – | – |
| Configure slots/capacity (own centre) | – | – | ✅ | ✅ | – |
| Assign operators | – | – | ✅ | ✅ | – |
| Configure crops/prices/templates (global) | – | – | – | ✅ | – |
| Manage users & roles | – | – | – | ✅ | – |
| View audit logs | – | – | partial | ✅ | – |
| View cross-centre statistics | – | – | – | ✅ | ✅ |
| Export regional reports | – | – | – | ✅ | ✅ |

## 8. Personas

**Ramesh (Farmer, 45)** — Owns 3 acres, grows wheat and mustard, has a basic Android smartphone with intermittent 3G/4G, is more comfortable in Hindi than English, wants to know "when should I actually go" and "when will I get paid."

**Sunita (Centre Operator, 29)** — Works at a single procurement centre during season, processes 100+ farmers/day, needs a fast, low-click interface that tolerates network drops without losing data.

**Vikram (Centre Manager, 38)** — Oversees 2–3 centres, needs to see today's load across centres, reassign operators, and close a centre early if needed (e.g., weather).

**Anita (Administrator, 33)** — Manages centre onboarding, crop/price configuration, and system-wide reporting; not a developer, needs a clear admin UI, not database access.

**Mr. Desai (Government Official, 50)** — Reviews weekly/monthly regional procurement volume, payment-pending numbers, and anomaly flags; needs export-ready reports, not day-to-day operational tools.

## 9. User Journeys

**Journey A — Farmer books and completes a procurement (happy path)**
Register → verify OTP → complete profile → add farm/produce → select centre → view slot calendar → book slot → receive booking ID/token + notification → track queue on booking day → arrive, check in → operator verifies, weighs, grades → system calculates amount → procurement confirmed → payment status becomes "Processing" then "Successful" → farmer views/downloads digital receipt.

**Journey B — Operator processes a walk-in**
Operator opens dashboard → searches farmer by mobile number (or registers a minimal new farmer record) → creates a same-day walk-in queue entry → proceeds through verification/weighing/grading/procurement identical to a booked farmer, flagged internally as `walk_in`.

**Journey C — Admin configures a new season's pricing**
Admin logs in → opens Pricing Configuration → adds/updates crop, grade, and rate entries with an effective-date range → publishes → change is versioned and audit-logged → new procurements after the effective date use the new rule automatically.

**Journey D — Government official reviews regional health**
Official logs in (read-only) → opens Regional Dashboard → filters by district/date range → views aggregate volume, payment-pending %, no-show rate, and any anomaly flags → exports a CSV/PDF report.

## 10. Functional Requirements

Priority legend: **P0 = Critical (MVP)**, **P1 = High (MVP or immediately post-MVP)**, **P2 = Medium**, **P3 = Future**.

| Req ID | Description | Role | Priority | Dependencies |
|---|---|---|---|---|
| FR-001 | Farmer can register with mobile number and OTP verification | Farmer | P0 | Notification (SMS) |
| FR-002 | Farmer can log in via mobile+OTP or mobile/email+password | Farmer | P0 | FR-001 |
| FR-003 | Farmer can create/edit profile (name, DOB, address, language) | Farmer | P0 | FR-002 |
| FR-004 | Farmer can add/edit one or more farm/land records | Farmer | P0 | FR-003 |
| FR-005 | Farmer can add/edit produce/crop records tied to a farm | Farmer | P0 | FR-004 |
| FR-006 | Farmer can browse active procurement centres | Farmer | P0 | Centre config |
| FR-007 | Farmer can view a centre's slot calendar with live availability | Farmer | P0 | Slot Engine |
| FR-008 | Farmer can book an available slot atomically | Farmer | P0 | Slot Engine |
| FR-009 | Farmer receives a unique booking ID and digital token on booking | Farmer | P0 | FR-008 |
| FR-010 | Farmer can cancel a future booking within policy window | Farmer | P0 | Admin config |
| FR-011 | Farmer can reschedule a booking to another available slot | Farmer | P1 | FR-008 |
| FR-012 | Farmer can view live queue position and estimated wait | Farmer | P0 | Queue Engine |
| FR-013 | Farmer receives notifications for key lifecycle events | Farmer | P0 | Notification System |
| FR-014 | Farmer can view procurement history and receipts | Farmer | P0 | Procurement Module |
| FR-015 | Farmer can view payment status and history | Farmer | P0 | Payment Tracking |
| FR-016 | Farmer can submit a complaint/feedback | Farmer | P1 | – |
| FR-017 | Operator can view today's booking list for their centre | Operator | P0 | Slot Engine |
| FR-018 | Operator can view/manage the live queue for their centre | Operator | P0 | Queue Engine |
| FR-019 | Operator can check in a farmer (booked or walk-in) | Operator | P0 | FR-018 |
| FR-020 | Operator can call the next farmer in queue | Operator | P0 | FR-018 |
| FR-021 | Operator can record weight, quality/grade, and confirm procurement | Operator | P0 | Procurement Module |
| FR-022 | System calculates gross/net amount from configured pricing rules | System | P0 | Admin pricing config |
| FR-023 | Operator can mark/update payment status | Operator | P0 | Payment Tracking |
| FR-024 | Operator can register a walk-in farmer and create a same-day entry | Operator | P1 | FR-019 |
| FR-025 | Manager can configure centre capacity and slot templates | Manager | P0 | – |
| FR-026 | Manager can assign/deactivate operators for a centre | Manager | P1 | – |
| FR-027 | Admin can manage users, roles, and permissions | Admin | P0 | – |
| FR-028 | Admin can manage centres (create/edit/deactivate) | Admin | P0 | – |
| FR-029 | Admin can configure crops, grades, and pricing rules with effective dates | Admin | P0 | – |
| FR-030 | Admin can configure notification templates | Admin | P1 | – |
| FR-031 | Admin can view audit logs with filters | Admin | P1 | Audit Logging |
| FR-032 | Admin/Official can view/export reports and analytics | Admin, Official | P1 | Reporting |
| FR-033 | System prevents two farmers from being confirmed into the same slot unit beyond capacity | System | P0 | Slot Engine concurrency control |
| FR-034 | System supports English and Hindi UI with a pluggable i18n layer | System | P0 | – |
| FR-035 | System logs every state-changing action with actor, action, before/after values | System | P0 | Audit Logging |

## 11. Feature Specifications

### 11.1 Farmer Application (Responsive Web / PWA)
- Mobile-first, installable PWA; works down to low-end Android browsers.
- Core flows: Auth → Profile/Farm/Produce setup → Centre & Slot selection → Booking confirmation with digital token (QR + alphanumeric code) → Live queue screen (WebSocket-driven, polling fallback) → Procurement/Payment status → History/Receipts → Complaints/Support → Settings (language, notification preferences).
- All list/detail screens must show clear status chips (e.g., *Booked*, *Checked-in*, *In Queue*, *Processing*, *Completed*, *Cancelled*, *No-show*).

### 11.2 Procurement Centre Operator Dashboard (Web)
- Optimized for a desktop or tablet at the centre; large touch targets for quick, repetitive daily use.
- Core screens: Today's Bookings, Live Queue (call/check-in/complete controls), Farmer Search, Verification, Weighing & Grading entry, Procurement confirmation, Payment status update, Daily Summary.
- Must tolerate brief connectivity loss without losing in-progress entry (see Section 22).

### 11.3 Admin Dashboard (Web)
- Core modules: Overview (KPIs), Centres, Farmers, Operators/Managers, Bookings, Queue (read/monitor), Procurement, Payments, Reports, Notification Templates, System Configuration (crops/grades/pricing/slot rules/holidays), Audit Logs.
- All configuration changes are versioned and require a confirmation step; destructive actions (deactivating a centre, deleting a user) require a secondary confirmation and are audit-logged.

### 11.4 Government/Official Portal
- Read-only dashboard: regional/centre filters, date-range filters, KPI summary, exportable reports (CSV/PDF), anomaly flag list (e.g., unusually high cancellation rate at a centre).

## 12. Slot Booking Engine

### 12.1 Concepts
- A **Slot Template** defines recurring time windows for a centre (e.g., 09:00–10:00, capacity 20 farmers) on given weekdays.
- A **Slot Instance** is a concrete date+time+centre record generated from a template (or created manually), with a fixed **capacity** (max bookings) and optionally a **produce-quantity capacity**.
- A **Booking** reserves exactly one unit of a Slot Instance's capacity for one farmer.

### 12.2 Rules
- A farmer may hold at most **N active bookings** at a time (N configurable; default assumption: 1 active booking per crop-season — **Assumption**).
- Booking a slot must be **atomic**: capacity is decremented and the booking row is created in a single transaction; if capacity is exhausted, the booking is rejected with `SLOT_UNAVAILABLE`.
- Cancellation and rescheduling are allowed until an admin-configured cutoff (e.g., 2 hours before slot start — **Assumption, admin-configurable**).
- No-show handling: if a farmer does not check in within an admin-configured grace period after slot start, the booking is auto-marked `no_show` and capacity is released back to the pool if the centre allows re-release (configurable).
- Walk-ins are supported by creating a same-day queue entry not tied to a pre-existing slot capacity unit, subject to a centre's `allow_walk_ins` and remaining daily capacity.

### 12.3 Concurrency Control (preventing double-booking of the last slot unit)
Two farmers must never both be confirmed into the same last-available capacity unit. Recommended approach:
1. Model slot capacity as a counter column (`slots.available_capacity`) with a database-level **conditional update**: `UPDATE slots SET available_capacity = available_capacity - 1 WHERE id = :slotId AND available_capacity > 0`. If the update affects 0 rows, the booking fails with `SLOT_UNAVAILABLE`.
2. Wrap the conditional update and the booking insert in a single database transaction so both succeed or both roll back.
3. This relies on PostgreSQL's row-level locking under `READ COMMITTED`/`REPEATABLE READ` to serialize concurrent decrements on the same row — no application-level distributed lock is required for single-database deployments.
4. For very high-contention slots (rare), a short-lived Redis lock (`SETNX` with TTL) keyed on `slot:{id}` can be used as a pre-check to fail fast before hitting the database, purely as an optimization — the database constraint remains the source of truth.
5. A unique constraint on `(farmer_id, slot_id)` in `bookings` prevents the same farmer from double-booking the same slot.

### 12.4 Slot Lifecycle States
`draft → published → open → full → closed → completed` (a slot can also be `cancelled` by a manager, e.g., due to centre closure — all existing bookings for a cancelled slot transition to `booking.slot_cancelled` and trigger notifications).

## 13. Real-Time Queue Management

### 13.1 Token & Queue Number
- On check-in (or walk-in registration), a **queue token** is generated per centre per day, formatted as `{CentreCode}{SequentialNumber}` (e.g., `A1025`), reset daily.
- A `queue_entries` row tracks: `token`, `booking_id` (nullable for walk-ins), `centre_id`, `status`, `position`, `checked_in_at`, `called_at`, `processing_started_at`, `completed_at`.

### 13.2 Status Lifecycle
`waiting → called → processing → completed`
Alternative terminal states: `no_show`, `cancelled`.

### 13.3 Live Data Farmers See
```
Token: A1025
Current serving token: A1018
Farmers ahead of you: 7
Estimated waiting time: 42 minutes
```

### 13.4 Estimated Waiting Time Formula
`estimated_wait_minutes = farmers_ahead × rolling_avg_processing_time_minutes`
where `rolling_avg_processing_time_minutes` is a trailing average (e.g., last 20 completed entries at that centre today, falling back to the centre's historical average if fewer than 5 entries exist today).

### 13.5 Architecture
- **WebSocket layer** (Socket.IO or native `ws` behind a gateway): each connected farmer/operator client joins a room `centre:{centreId}`.
- On any queue-state change (check-in, call, complete, no-show), the backend publishes an event to the centre's room; connected clients update the UI without a page refresh.
- **Redis** is used as (a) the Socket.IO adapter's pub/sub backplane for multi-instance backend scaling, and (b) a fast read cache for "current serving token" and "queue length" per centre, refreshed on every state change and periodically reconciled against PostgreSQL (source of truth).
- **Reconnection handling:** on reconnect, the client re-fetches full current state via `GET /queue/{centreId}/position` (REST) before resuming WebSocket updates, so a missed event during disconnect cannot leave stale UI.
- **Offline fallback:** if WebSocket is unavailable, the farmer app falls back to polling `GET /queue/{centreId}/position` every 20–30 seconds.
- **Queue consistency / race conditions:** all queue-state transitions (call, complete, no-show) go through a single backend service method that performs a row-level `SELECT … FOR UPDATE` on the `queue_entries` row (or an atomic conditional update) before changing state, preventing two operators from calling the same farmer simultaneously (`FR-related edge case`, see Section 24).
- Queue **position** is computed, not stored as a mutable rank on every row; it is derived at read time as `COUNT(*) WHERE centre_id = X AND status = 'waiting' AND checked_in_at < this_entry.checked_in_at` (or by ordering on a monotonically increasing sequence), avoiding expensive re-ranking writes on every change.

## 14. Procurement Module

### 14.1 Captured Data
| Field | Notes |
|---|---|
| Farmer | FK to farmer |
| Booking ID | FK to booking (nullable for walk-in) |
| Centre | FK to centre |
| Crop / Produce type | FK to crop |
| Quantity declared | Farmer-entered at booking time (informational) |
| Actual weight | Operator-entered at weighing, required |
| Unit | kg/quintal — admin-configurable per crop |
| Quality / Grade | Operator-selected from configured grade list |
| Rate | Looked up from active pricing rule for crop+grade+date; **never hardcoded** |
| Gross amount | `actual_weight × rate` |
| Deductions | Optional, admin-configured deduction rules (e.g., moisture) — value only, not policy |
| Net payable amount | `gross_amount − deductions` |
| Operator | FK to user who confirmed |
| Timestamp | Server time, immutable once confirmed |
| Transaction ID | System-generated unique reference |

### 14.2 Workflow States
`pending_verification → verified → weighed → graded → calculated → confirmed → (payment states, see Section 15)`
A confirmed procurement record is **append-only**: corrections after confirmation are made via a linked `procurement_corrections` audit entry, not by silently overwriting the original row, so the audit trail is preserved (see Section 24, "Operator accidentally submits procurement").

### 14.3 Price/Rate Rules
All prices, rates, deduction formulas, and grade definitions are stored in an admin-managed `pricing_rules` table with `effective_from`/`effective_to` dates. The Platform performs the arithmetic; it never invents or hardcodes a rate. **No specific procurement prices are defined in this PRD.**

## 15. Payment Tracking

The Platform tracks payment status; it does not process bank transfers unless a future gateway is integrated (see Section 36).

### 15.1 Statuses
`pending → processing → successful` | `failed` | `reversed` | `on_hold`

### 15.2 Tracked Fields
Payment reference, transaction ID, amount, payment method (e.g., bank transfer, cheque — configurable list), date, status, failure reason (when applicable), linked procurement record.

### 15.3 Integration Points (future)
A `payment_provider_adapter` interface is defined so a real payment/banking system can later push status updates via a webhook (`POST /webhooks/payments/{provider}`), with idempotency handling (see Section 24, "duplicate payment callback").

## 16. Notification System

### 16.1 Events & Preferred Channels
| Event | Priority | Channels |
|---|---|---|
| Registration successful | Low | In-app |
| Slot booked | High | Push, SMS, In-app |
| Slot reminder (e.g., T-1 day, T-2 hours) | High | Push, SMS |
| Queue approaching (e.g., 5 farmers ahead) | High | Push, In-app |
| Farmer called | Critical | Push, SMS |
| Farmer delayed / centre running behind | Medium | Push, In-app |
| Procurement completed | High | Push, SMS, In-app |
| Payment initiated | Medium | Push, In-app |
| Payment completed | High | Push, SMS, In-app |
| Booking cancelled | High | Push, SMS |
| Centre closed (emergency) | Critical | Push, SMS, In-app |
| Slot changed/rescheduled | High | Push, SMS |

### 16.2 Template Examples (placeholders in `{{ }}`)
- Slot booked (SMS/EN): `"Your slot at {{centre_name}} is booked for {{date}} {{time}}. Token: {{token}}. Booking ID: {{booking_id}}."`
- Slot booked (SMS/HI): `"{{centre_name}} पर आपकी स्लॉट {{date}} {{time}} के लिए बुक हो गई है। टोकन: {{token}}."`
- Farmer called: `"It's your turn at {{centre_name}}. Please proceed to counter now. Token: {{token}}."`
- Payment completed: `"Payment of {{amount}} for {{crop_name}} ({{transaction_id}}) has been marked successful."`

### 16.3 Delivery & Failure Handling
Notifications are queued (background worker) with per-channel retry (e.g., 3 attempts, exponential backoff); failures are logged and, for Critical-priority events, an in-app notification is always additionally created as a guaranteed fallback since it does not depend on external SMS/push providers.

## 17. Database Architecture (PostgreSQL)

### 17.1 Entity Relationship Overview (description)
- `users` is the identity table for every human actor (farmer, operator, manager, admin, official); `roles`/`permissions` implement RBAC.
- A `farmers` row extends a `users` row 1:1 with farmer-specific fields.
- `farms` (1 farmer : many farms), `crops`/`produce` (catalog + per-farmer declared produce).
- `procurement_centres` (1) : many `centre_operators` (assignment), many `slots`.
- `slots` (1 centre : many slots) : many `bookings` (1 slot : many bookings up to capacity).
- `bookings` (1) : 0..1 `queue_entries` (created at check-in) : 0..1 `procurements` : 0..1 `payments`.
- `audit_logs` references any entity generically via `entity_type` + `entity_id`.

### 17.2 Core Tables

**users**
| Column | Type | Constraints |
|---|---|---|
| id | UUID | PK, default gen_random_uuid() |
| mobile_number | VARCHAR(15) | UNIQUE, NOT NULL |
| email | VARCHAR(255) | UNIQUE, NULL |
| password_hash | VARCHAR(255) | NULL (OTP-only users may have none) |
| role_id | UUID | FK → roles.id, NOT NULL |
| preferred_language | VARCHAR(5) | DEFAULT 'en' |
| status | VARCHAR(20) | DEFAULT 'active' (active/suspended/deleted) |
| created_at | TIMESTAMPTZ | DEFAULT now() |
| updated_at | TIMESTAMPTZ | DEFAULT now() |
Indexes: UNIQUE(mobile_number), UNIQUE(email) WHERE email IS NOT NULL, INDEX(role_id)

**roles** / **permissions** / **role_permissions**
| Table | Key Columns |
|---|---|
| roles | id (PK), name (UNIQUE: farmer/operator/manager/admin/official), description |
| permissions | id (PK), code (UNIQUE, e.g. `booking.create`), description |
| role_permissions | role_id (FK), permission_id (FK), PK(role_id, permission_id) |

**farmers**
| Column | Type | Constraints |
|---|---|---|
| id | UUID | PK |
| user_id | UUID | FK → users.id, UNIQUE, NOT NULL |
| full_name | VARCHAR(255) | NOT NULL |
| gender | VARCHAR(10) | NULL |
| date_of_birth | DATE | NULL |
| address | TEXT | NULL |
| district | VARCHAR(100) | NULL, INDEX |
| state | VARCHAR(100) | NULL |
| id_document_type | VARCHAR(50) | NULL |
| id_document_number | VARCHAR(100) | NULL, encrypted at rest |
| created_at / updated_at | TIMESTAMPTZ | DEFAULT now() |

**farms** / **land_records**
| Table | Key Columns |
|---|---|
| farms | id (PK), farmer_id (FK, INDEX), name, total_area, area_unit, village, district, created_at |
| land_records | id (PK), farm_id (FK, INDEX), survey_number, ownership_type, document_url, verified_status, created_at |

**crops** / **produce**
| Table | Key Columns |
|---|---|
| crops | id (PK), name (UNIQUE), category, default_unit, active (BOOL) |
| produce | id (PK), farmer_id (FK), farm_id (FK), crop_id (FK), season, expected_quantity, unit, created_at |

**procurement_centres**
| Column | Type | Constraints |
|---|---|---|
| id | UUID | PK |
| name | VARCHAR(255) | NOT NULL |
| code | VARCHAR(10) | UNIQUE, NOT NULL (used in token prefix) |
| district | VARCHAR(100) | INDEX |
| address | TEXT | |
| geo_lat / geo_lng | DECIMAL | NULL |
| working_hours_json | JSONB | working hours per weekday |
| allow_walk_ins | BOOLEAN | DEFAULT true |
| status | VARCHAR(20) | DEFAULT 'active' |
| created_at / updated_at | TIMESTAMPTZ | |

**centre_operators**
id (PK), centre_id (FK, INDEX), user_id (FK, INDEX), role_at_centre (operator/manager), active (BOOL), assigned_at

**centre_capacity**
id (PK), centre_id (FK, UNIQUE with date), date, max_daily_bookings, max_daily_quantity, notes

**slots**
| Column | Type | Constraints |
|---|---|---|
| id | UUID | PK |
| centre_id | UUID | FK, INDEX |
| date | DATE | NOT NULL, INDEX(centre_id, date) |
| start_time / end_time | TIME | NOT NULL |
| capacity | INT | NOT NULL |
| available_capacity | INT | NOT NULL, CHECK >= 0 |
| status | VARCHAR(20) | draft/published/open/full/closed/completed/cancelled |
| created_from_template_id | UUID | NULL FK |
| created_at / updated_at | TIMESTAMPTZ | |
Index: UNIQUE(centre_id, date, start_time)

**bookings**
| Column | Type | Constraints |
|---|---|---|
| id | UUID | PK |
| booking_code | VARCHAR(20) | UNIQUE, NOT NULL (public-facing ID) |
| farmer_id | UUID | FK, INDEX |
| slot_id | UUID | FK, INDEX |
| produce_id | UUID | FK, NULL |
| status | VARCHAR(20) | booked/cancelled/rescheduled/no_show/completed |
| booking_type | VARCHAR(10) | scheduled/walk_in |
| created_at / updated_at | TIMESTAMPTZ | |
Index: UNIQUE(farmer_id, slot_id) WHERE status='booked'

**booking_status_history**
id (PK), booking_id (FK, INDEX), old_status, new_status, changed_by (FK users), changed_at

**queue_entries**
| Column | Type | Constraints |
|---|---|---|
| id | UUID | PK |
| token | VARCHAR(20) | NOT NULL |
| centre_id | UUID | FK, INDEX |
| booking_id | UUID | FK, NULL (walk-in has none) |
| status | VARCHAR(20) | waiting/called/processing/completed/no_show/cancelled |
| checked_in_at | TIMESTAMPTZ | NOT NULL |
| called_at / processing_started_at / completed_at | TIMESTAMPTZ | NULL |
| operator_id | UUID | FK users, NULL |
Index: UNIQUE(centre_id, token, checked_in_at::date); INDEX(centre_id, status)

**queue_events**
id (PK), queue_entry_id (FK, INDEX), event_type, actor_id (FK users), payload_json, created_at

**check_ins**
id (PK), booking_id (FK, NULL), queue_entry_id (FK), method (qr/manual/self), verified_by (FK users), created_at

**procurements**
| Column | Type | Constraints |
|---|---|---|
| id | UUID | PK |
| transaction_id | VARCHAR(30) | UNIQUE, NOT NULL |
| booking_id | UUID | FK, NULL |
| queue_entry_id | UUID | FK, NOT NULL |
| farmer_id | UUID | FK, INDEX |
| centre_id | UUID | FK, INDEX |
| crop_id | UUID | FK |
| actual_weight | DECIMAL(10,2) | NOT NULL |
| unit | VARCHAR(10) | NOT NULL |
| quality_grade | VARCHAR(20) | NOT NULL |
| rate | DECIMAL(10,2) | NOT NULL |
| gross_amount | DECIMAL(12,2) | NOT NULL |
| deductions | DECIMAL(12,2) | DEFAULT 0 |
| net_amount | DECIMAL(12,2) | NOT NULL |
| operator_id | UUID | FK users |
| status | VARCHAR(20) | pending_verification/verified/weighed/graded/calculated/confirmed |
| confirmed_at | TIMESTAMPTZ | NULL |
| created_at / updated_at | TIMESTAMPTZ | |

**produce_quality** / **weighing_records**
| Table | Key Columns |
|---|---|
| produce_quality | id (PK), procurement_id (FK, INDEX), grade, inspector_id (FK), notes, inspected_at |
| weighing_records | id (PK), procurement_id (FK, INDEX), weight, weighed_by (FK), device_ref, weighed_at |

**pricing_rules**
id (PK), crop_id (FK, INDEX), grade, rate, unit, effective_from (DATE), effective_to (DATE, NULL), created_by (FK), created_at — INDEX(crop_id, grade, effective_from)

**payments**
| Column | Type | Constraints |
|---|---|---|
| id | UUID | PK |
| procurement_id | UUID | FK, UNIQUE |
| payment_reference | VARCHAR(50) | UNIQUE |
| amount | DECIMAL(12,2) | NOT NULL |
| method | VARCHAR(30) | |
| status | VARCHAR(20) | pending/processing/successful/failed/reversed/on_hold |
| failure_reason | TEXT | NULL |
| initiated_at / completed_at | TIMESTAMPTZ | NULL |

**payment_status_history**
id (PK), payment_id (FK, INDEX), old_status, new_status, changed_by, changed_at, source (operator/webhook/system)

**notifications** / **notification_templates**
| Table | Key Columns |
|---|---|
| notification_templates | id (PK), event_code (UNIQUE), channel, language, body_template |
| notifications | id (PK), user_id (FK, INDEX), template_id (FK), payload_json, channel, status (queued/sent/delivered/failed), created_at, read_at |

**complaints** / **feedback**
| Table | Key Columns |
|---|---|
| complaints | id (PK), farmer_id (FK), booking_id (FK, NULL), category, description, status (open/in_progress/resolved/closed), assigned_to (FK), created_at |
| feedback | id (PK), farmer_id (FK), booking_id (FK, NULL), rating (1-5), comment, created_at |

**documents**
id (PK), owner_type, owner_id, document_type, file_url, uploaded_by (FK), created_at

**audit_logs**
| Column | Type | Constraints |
|---|---|---|
| id | UUID | PK |
| user_id | UUID | FK, NULL (system actions) |
| role | VARCHAR(20) | |
| action | VARCHAR(100) | e.g. `procurement.weight_updated` |
| entity_type | VARCHAR(50) | |
| entity_id | UUID | |
| old_value | JSONB | NULL |
| new_value | JSONB | NULL |
| ip_address | VARCHAR(45) | NULL |
| device_info | TEXT | NULL |
| created_at | TIMESTAMPTZ | DEFAULT now(), INDEX |

### 17.3 Key Relationships & Cardinality
- users (1) — (1) farmers/centre_operators (role-dependent)
- farmers (1) — (N) farms — (N) land_records
- farmers (1) — (N) produce
- procurement_centres (1) — (N) slots — (N) bookings (bounded by `capacity`)
- bookings (1) — (0..1) queue_entries — (0..1) procurements — (0..1) payments
- procurement_centres (1) — (N) centre_operators — (1) users

### 17.4 Important Indexes
`bookings(farmer_id)`, `bookings(slot_id)`, `slots(centre_id, date)`, `queue_entries(centre_id, status)`, `procurements(centre_id, created_at)`, `payments(status)`, `audit_logs(entity_type, entity_id)`, `audit_logs(created_at)`.

### 17.5 Data Retention Considerations
- `audit_logs` and `procurements`/`payments` should be retained per the adopting organization's regulatory requirement (**Assumption: minimum 7 years for financial records** — confirm with legal/compliance before production).
- `notifications` and `queue_events` are high-volume/low-value-long-term; recommend partitioning by month and archiving/purging after e.g. 12 months (configurable).
- Personally identifiable fields (`id_document_number`) should be encrypted at rest and access-logged.

## 18. API Specification

Base URL: `/api/v1`. All responses use the standard envelope (see Section 30). All endpoints except `/auth/*` and `GET /centres*` require `Authorization: Bearer <JWT>`.

### 18.1 Authentication
| Method | Endpoint | Auth | Role | Body | Response | Status | Errors |
|---|---|---|---|---|---|---|---|
| POST | /auth/register | None | Any | `{mobile_number, full_name}` | `{user_id}` | 201 | `MOBILE_ALREADY_REGISTERED` (409) |
| POST | /auth/send-otp | None | Any | `{mobile_number}` | `{otp_sent: true, expires_in}` | 200 | `RATE_LIMITED` (429) |
| POST | /auth/verify-otp | None | Any | `{mobile_number, otp}` | `{access_token, refresh_token, user}` | 200 | `OTP_INVALID` / `OTP_EXPIRED` (400) |
| POST | /auth/login | None | Any | `{mobile_or_email, password}` | `{access_token, refresh_token, user}` | 200 | `INVALID_CREDENTIALS` (401), `ACCOUNT_LOCKED` (423) |
| POST | /auth/refresh | Refresh token | Any | `{refresh_token}` | `{access_token, refresh_token}` | 200 | `TOKEN_INVALID` (401) |
| POST | /auth/logout | Bearer | Any | – | `{success: true}` | 200 | – |

### 18.2 Farmer
| Method | Endpoint | Role | Notes |
|---|---|---|---|
| GET | /farmers/me | Farmer | Full profile |
| PUT | /farmers/me | Farmer | Update profile |
| GET | /farmers/me/bookings | Farmer | Query: status, from, to |
| GET | /farmers/me/procurements | Farmer | History |
| GET | /farmers/me/payments | Farmer | History |

### 18.3 Centres
| Method | Endpoint | Role | Notes |
|---|---|---|---|
| GET | /centres | Any | Filter by district; public list |
| GET | /centres/{id} | Any | Detail |
| GET | /centres/{id}/slots | Any | Query: date range; returns availability |

### 18.4 Booking
| Method | Endpoint | Role | Request | Response | Errors |
|---|---|---|---|---|---|
| POST | /bookings | Farmer | `{slot_id, produce_id}` | `{booking_id, booking_code, token}` | `SLOT_UNAVAILABLE` (409), `DUPLICATE_BOOKING` (409) |
| GET | /bookings/{id} | Farmer/Operator | – | Booking detail | `NOT_FOUND` (404) |
| PATCH | /bookings/{id}/cancel | Farmer/Operator | `{reason}` | Updated booking | `CANCEL_WINDOW_CLOSED` (400) |
| PATCH | /bookings/{id}/reschedule | Farmer | `{new_slot_id}` | Updated booking | `SLOT_UNAVAILABLE` (409) |

### 18.5 Queue
| Method | Endpoint | Role | Notes |
|---|---|---|---|
| GET | /queue/{centreId} | Operator/Manager | Full live queue list |
| GET | /queue/{centreId}/position | Farmer | Own position, current serving token, est. wait |
| POST | /queue/{id}/check-in | Operator/Farmer(self) | Marks `waiting` |
| POST | /queue/{id}/call | Operator | Marks `called`; publishes WS event |
| POST | /queue/{id}/complete | Operator | Marks `completed`; releases queue slot |

### 18.6 Procurement
| Method | Endpoint | Role | Request | Notes |
|---|---|---|---|---|
| POST | /procurements | Operator | `{queue_entry_id, crop_id, weight, grade}` | Creates + auto-calculates amount |
| GET | /procurements/{id} | Farmer/Operator/Admin | – | Detail |
| PATCH | /procurements/{id} | Operator (pre-confirm only) | `{weight?, grade?}` | `ALREADY_CONFIRMED` (409) if locked |

### 18.7 Payment
| Method | Endpoint | Role | Notes |
|---|---|---|---|
| GET | /payments/{id} | Farmer/Operator/Admin | Detail |
| GET | /farmers/me/payments | Farmer | List |
| PATCH | /payments/{id}/status | Operator/Admin | Manual status update, audit-logged |
| POST | /webhooks/payments/{provider} | System (signed) | Future gateway callback, idempotent by `payment_reference` |

### 18.8 Notifications
| Method | Endpoint | Role | Notes |
|---|---|---|---|
| GET | /notifications | Any authenticated | Own notifications, paginated |
| PATCH | /notifications/{id}/read | Any authenticated | Mark read |

### 18.9 Admin
| Method | Endpoint | Role | Notes |
|---|---|---|---|
| GET/POST/PUT | /admin/users | Admin | User management |
| GET/POST/PUT | /admin/centres | Admin | Centre management |
| GET/POST/PUT | /admin/crops | Admin | Crop/grade catalog |
| GET/POST/PUT | /admin/pricing-rules | Admin | Rate configuration with effective dates |
| GET/POST/PUT | /admin/notification-templates | Admin | Template management |
| GET | /admin/audit-logs | Admin | Filterable, paginated |
| GET | /admin/reports/{type} | Admin, Official | Query params for date range/centre/district |

## 19. System Architecture

```mermaid
flowchart TB
    FA[Farmer App - PWA] -->|HTTPS/WSS| GW[API Gateway / Load Balancer]
    OD[Operator Dashboard] -->|HTTPS/WSS| GW
    AD[Admin Dashboard] -->|HTTPS| GW
    GP[Gov Official Portal] -->|HTTPS| GW
    GW --> AUTH[Auth Service - JWT/OTP/RBAC]
    GW --> API[Backend API - Business Logic]
    API --> DB[(PostgreSQL - Primary)]
    API --> CACHE[(Redis - Cache and PubSub)]
    API --> WS[WebSocket Layer]
    WS --> CACHE
    API --> QUEUE[Background Job Queue]
    QUEUE --> NOTIF[Notification Service]
    NOTIF --> SMS[SMS Gateway]
    NOTIF --> FCM[Push - FCM]
    QUEUE --> BACKUP[Backup Worker]
    API --> STORAGE[Object Storage - Documents/Receipts]
    DB --> REPLICA[(Read Replica)]
    API --> LOGS[Logging and Metrics]
    LOGS --> MON[Monitoring/Alerting]
```

**Component notes:**
- **API Gateway / Load Balancer** (Nginx): TLS termination, routing, rate limiting.
- **Auth Service**: issues/validates JWT access tokens, rotates refresh tokens, enforces RBAC on every request.
- **Backend API**: stateless application servers; horizontally scalable behind the load balancer.
- **PostgreSQL**: system of record; a read replica offloads reporting/analytics queries from the primary.
- **Redis**: WebSocket pub/sub backplane, queue-state cache, OTP storage with TTL, rate-limit counters.
- **WebSocket layer**: co-located with API servers; uses Redis adapter so any server instance can broadcast to any connected client.
- **Background job queue** (e.g., BullMQ on Redis): notification dispatch, report generation, slot-template expansion, no-show auto-marking.
- **Object storage** (S3-compatible): uploaded documents, generated PDF receipts.
- **Logging/Monitoring**: centralized structured logs + metrics + health checks (Section 26).

## 20. Security & Privacy

- **Authentication:** OTP (mobile) as primary farmer auth; password+session or JWT for staff/admin roles; short-lived access tokens (e.g., 15 min) with rotating refresh tokens (e.g., 7–30 days), refresh-token reuse detection.
- **Authorization:** RBAC enforced server-side on every endpoint (never trust client-side role checks); centre-scoped authorization so an operator can only act on their assigned centre's data.
- **Encryption in transit:** TLS 1.2+ everywhere; WSS for WebSocket.
- **Encryption at rest:** database-level encryption (e.g., managed PostgreSQL encryption) plus application-level encryption for highly sensitive fields (ID document numbers).
- **Secrets management:** environment variables injected via a secrets manager/vault in production, never committed to source control.
- **Input validation:** schema validation (e.g., Zod/Joi or Pydantic) on every request body; parameterized queries/ORM only — no raw string SQL concatenation (SQL injection protection).
- **XSS protection:** output encoding on frontend, Content-Security-Policy headers, sanitization of any user-generated text before render.
- **CSRF:** not required for pure Bearer-token API usage from SPA/PWA clients (no cookies for auth); if cookies are ever used, apply CSRF tokens + SameSite=strict.
- **CORS:** explicit allow-list of frontend origins per environment.
- **Password hashing:** bcrypt/argon2 with per-user salt.
- **OTP security:** 6-digit OTP, 5–10 minute expiry, max 5 verification attempts, exponential cooldown between resend requests, per-mobile-number rate limiting.
- **Account lockout:** temporary lockout after repeated failed login/OTP attempts, with unlock via support flow.
- **API abuse prevention:** global and per-IP/per-user rate limiting at the gateway.
- **Audit logging:** every create/update/delete on sensitive entities recorded per Section 26.
- **Privacy:** collect only fields required for procurement and identity verification; do not require Aadhaar/e-KYC unless a future integration explicitly adds it under legal review (see NG5, Section 36).

## 21. Performance & Scalability

### 21.1 Performance Targets

| Metric | MVP Target | Production-Scale Target |
|---|---|---|
| API p95 response time | < 500ms | < 200ms |
| Queue update latency (WS) | < 3s | < 1s |
| Concurrent users | ~500 | 50,000+ |
| Concurrent bookings/sec | ~10 | 500+ |
| Notification delivery (SMS) | < 60s | < 15s |
| Availability | 99.0% | 99.9% |
| Recovery Time Objective (RTO) | < 4 hours | < 30 minutes |
| Recovery Point Objective (RPO) | < 24 hours | < 5 minutes |

### 21.2 Scalability Approach
- **Horizontal scaling** of stateless API/WebSocket servers behind a load balancer.
- **Database indexing** per Section 17.4; **read replicas** for reporting/analytics traffic separated from transactional traffic.
- **Redis** for caching hot reads (centre list, today's queue snapshot) and as the pub/sub backplane.
- **Background workers** scaled independently from web-facing API servers.
- **Event/queue architecture** (BullMQ/Redis Streams, or Kafka at large scale) decouples notification dispatch and report generation from request/response paths.
- **Object storage** for documents/receipts keeps large binary data out of the database.
- Path to production scale: MVP (1–5 centres, single DB) → pilot (tens of centres, read replica + Redis cluster) → production (hundreds/thousands of centres, sharding/partitioning by region if needed, CDN for static assets, multi-region considered only if latency/regulatory need justifies it).

## 22. Offline / Low-Connectivity Strategy

Procurement centres may have unreliable connectivity, especially in rural areas. The strategy distinguishes what **must** be online vs. what can **tolerate brief offline operation**.

### 22.1 Must Require Live Connectivity
- Slot booking confirmation (capacity must be checked/decremented against the single source of truth).
- Final procurement confirmation (financial record; must not be created twice or lost).
- Payment status changes.
- Authentication (OTP verification, token issuance).

### 22.2 Can Tolerate Temporary Offline Operation
- **Operator data entry** (weight, grade) can be captured into local browser storage (IndexedDB) while connectivity is down, then **synced** once connectivity returns — but the record only becomes an official `procurement` once synced and confirmed server-side, never before.
- **Farmer queue view**: falls back to last-known cached state with a visible "reconnecting…" indicator rather than blank/error screens.

### 22.3 Mechanisms
- **Local caching:** operator dashboard caches today's booking list and in-progress entries client-side.
- **Retry queues:** failed writes (e.g., a weight submission during a network blip) are queued client-side with automatic retry and exponential backoff.
- **Idempotency:** every write operation from operator clients includes a client-generated idempotency key; the server deduplicates retried submissions so a retried request cannot create a duplicate procurement/payment record.
- **Offline operator workflow:** operator can continue entering weight/grade for the farmer currently being served even if the network briefly drops; submission is disabled/queued (not silently lost) until confirmed.
- **Synchronization & conflict resolution:** on reconnect, queued writes are submitted in order; if a conflicting server-side change occurred (e.g., booking was cancelled from another device), the client is shown a conflict resolution prompt rather than silently overwriting.
- **Temporary token storage:** the day's queue tokens can be generated from a locally reserved token-range per centre (coordinated with the server periodically) so check-in can continue very briefly during a total outage, then reconciled — this is a deliberate small-risk tradeoff and should be scoped conservatively (**Assumption**: full offline check-in is a P2/P3 capability, not MVP).

## 23. Analytics & Reporting

### 23.1 Dashboards Show
Total registered farmers, today's bookings, active queue length, average waiting time, average processing time, completed/pending procurements, total quantity procured, total procurement value, payment pending/completed totals, centre utilization, no-show rate, cancellation rate. Reports available daily/weekly/monthly/yearly, filterable by centre/district/crop.

### 23.2 Queue Analytics Formulas
| Metric | Formula |
|---|---|
| Average waiting time | `AVG(processing_started_at − checked_in_at)` over completed entries in period |
| Average processing time | `AVG(completed_at − processing_started_at)` over completed entries |
| Average farmers/hour | `COUNT(completed entries) / operating_hours` |
| Centre utilization | `SUM(bookings) / SUM(slot_capacity)` over period |
| Peak hours | Hour-of-day bucket with highest `COUNT(check-ins)` |
| No-show percentage | `COUNT(status='no_show') / COUNT(all bookings)` × 100 |
| Average delay | `AVG(called_at − slot.start_time)` for booked (non-walk-in) entries |
| Estimated waiting time | See Section 13.4 |

## 24. Edge Cases

| Scenario | System Behaviour |
|---|---|
| Farmer books twice for the same slot | Rejected by `UNIQUE(farmer_id, slot_id)` constraint → `DUPLICATE_BOOKING` |
| Slot becomes unavailable during booking | Conditional-update capacity check fails → `SLOT_UNAVAILABLE`, client shown next available slot |
| Farmer cancels | Booking → `cancelled`; capacity released per policy; notification sent |
| Farmer doesn't arrive | Auto-marked `no_show` after grace period; capacity optionally released |
| Farmer arrives late | Operator can still check in if within a configurable late-arrival window; otherwise treated as no-show unless overridden by operator/manager |
| Internet disconnects (farmer app) | Falls back to cached state + polling; reconnects and resyncs via REST |
| Operator loses connection | Local queue + idempotent retry per Section 22 |
| Server restart | Stateless API servers restart behind load balancer with zero data loss (DB is source of truth); WebSocket clients auto-reconnect |
| Duplicate payment callback | Deduplicated by `payment_reference` + idempotency key; second callback is a no-op that logs a duplicate event |
| Payment failure | Status → `failed` with `failure_reason`; farmer notified; operator/admin can retry or escalate |
| SMS failure | Retried with backoff; in-app notification guaranteed as fallback for Critical events |
| Push notification failure | Same retry logic; SMS remains the higher-reliability channel for Critical events |
| Centre temporarily closes | Manager marks centre/slots `cancelled`; affected bookings notified and offered reschedule |
| Farmer changes mobile number | Requires OTP verification of new number; old number is deactivated for auth, history preserved |
| Incorrect farmer information | Farmer can self-correct pre-verification fields; verified fields require operator/admin correction (audit-logged) |
| Wrong weight entered (pre-confirmation) | Operator can edit via `PATCH /procurements/{id}` while status is pre-`confirmed` |
| Wrong weight entered (post-confirmation) | Requires a `procurement_correction` record (see 14.2), audit-logged, does not silently overwrite |
| Operator accidentally submits procurement | Same correction workflow; a manager/admin approval step is recommended for post-confirmation corrections (**Assumption**) |
| Concurrent queue updates | Row-level locking / atomic conditional update on `queue_entries` (Section 13.5) |
| Multiple operators calling farmers | Same atomic transition guard prevents two operators from calling the same entry; second attempt returns `ALREADY_CALLED` |
| Database failure | Automated failover to standby (production) or restore-from-backup (MVP); API returns `SERVICE_UNAVAILABLE` rather than partial writes |
| Redis failure | System degrades gracefully: WebSocket falls back to REST polling, rate limiting/caching temporarily bypassed with in-memory fallback; core booking/procurement flows (backed by PostgreSQL) continue to function |

## 25. Testing Strategy

| Type | Scope |
|---|---|
| Unit testing | Business logic (pricing calculation, wait-time formula, RBAC checks), pure functions, ≥80% coverage target on core services |
| Integration testing | Service-to-DB, service-to-Redis, booking transaction atomicity |
| API testing | Every endpoint: happy path + documented error codes (contract tests) |
| Database testing | Migration correctness, constraint enforcement (unique/foreign keys), seed data integrity |
| Frontend testing | Component tests for critical flows (booking, queue view, procurement entry) |
| End-to-end testing | Full farmer journey (Journey A) and full operator journey (Journey B) via automated E2E (e.g., Playwright) |
| Load testing | Concurrent booking on a near-full slot; concurrent queue calls; target MVP/production numbers from Section 21.1 |
| Security testing | AuthZ boundary tests (role cannot access another centre's data), OTP brute-force protection, dependency vulnerability scanning |
| Mobile/responsive testing | Farmer PWA on low-end Android devices/browsers, offline install |
| Offline testing | Simulated network loss during operator data entry; verify idempotent resync with no duplicate procurement records |
| Real-time queue testing | Two simultaneous "call next" attempts on the same entry resolve to exactly one winner |

**Key test cases to always include:** double-booking the last slot unit; two operators calling the same queue entry; cancel-after-cutoff rejection; no-show auto-transition; duplicate payment webhook; OTP expiry/attempt-limit enforcement; role trying to access another centre's data (must be denied).

## 26. Deployment & Infrastructure

### Development
```mermaid
flowchart LR
    Dev[Developer Laptop] --> Docker[Docker Compose]
    Docker --> PG[(PostgreSQL container)]
    Docker --> RD[(Redis container)]
    Docker --> BE[Backend container]
    Docker --> FE[Frontend dev server]
```

### Production
```mermaid
flowchart TB
    Internet --> NGINX[Nginx - TLS/Reverse Proxy]
    NGINX --> FE_STATIC[Frontend static/CDN]
    NGINX --> API1[Backend instance 1]
    NGINX --> API2[Backend instance N]
    API1 --> PG_PRIMARY[(PostgreSQL Primary)]
    API2 --> PG_PRIMARY
    PG_PRIMARY --> PG_REPLICA[(PostgreSQL Replica)]
    API1 --> REDIS[(Redis Cluster)]
    API2 --> REDIS
    API1 --> WORKERS[Background Workers]
    WORKERS --> NOTIFSVC[Notification Services - SMS/FCM]
    WORKERS --> BACKUPSVC[Automated Backup]
```

- **SSL:** managed certificates (e.g., Let's Encrypt) auto-renewed at the Nginx/load-balancer layer.
- **DNS:** environment-specific subdomains (e.g., `app.`, `api.`, `admin.`), low-TTL during cutovers.
- **Environment variables:** injected per environment via the deployment platform's secret store (never baked into images).
- **Backups:** automated per Section 27.
- **Monitoring:** health checks + metrics per Section 26 below feed alerting.
- **Deployment process:** build → run automated test suite → build container images → push to registry → deploy to staging → smoke test → promote to production (blue/green or rolling deploy to avoid downtime).

## 27. Monitoring & Observability

- **Application logs:** structured JSON logs per request (request ID, user ID, latency, status).
- **Error logs:** captured with stack traces, grouped/deduplicated (e.g., Sentry-style tooling).
- **Audit logs:** as defined in Section 17.2/28, queryable by admins.
- **Metrics:** request rate, error rate, latency percentiles, queue length per centre, notification delivery rate, background job queue depth.
- **Health checks:** `/healthz` (liveness) and `/readyz` (readiness, checks DB/Redis connectivity) on every service.
- **Database monitoring:** connection pool usage, slow query log, replication lag.
- **Queue monitoring:** background job queue depth and failure rate; WebSocket connection count per instance.
- **Notification monitoring:** delivery/failure rate per channel/provider.
- **Critical alerts:** API error rate above threshold; DB replication lag above threshold; background job queue depth above threshold; SMS/push delivery failure rate above threshold; any centre's queue with zero "completed" transitions for an extended period during operating hours (possible stuck queue).

## 28. Backup & Disaster Recovery

- **Database backups:** automated daily full backup + continuous WAL archiving for point-in-time recovery.
- **Retention:** daily backups retained 30 days, weekly retained 12 weeks, monthly retained 12 months (**Assumption, adjust to org policy**).
- **Recovery process:** documented runbook — restore latest full backup + replay WAL to target point in time, on a recovery instance, verified before promoting to production.
- **Disaster recovery:** standby replica promotable to primary; documented RTO/RPO targets per Section 21.1.
- **Restoration testing:** scheduled (e.g., quarterly) restore drills to confirm backups are actually usable.

## 29. Development Phases

### Phase 1 — Foundation
**Objectives:** Establish project skeleton, DB, auth, roles.
**Features:** Repo/monorepo setup, CI skeleton, PostgreSQL schema (core tables), JWT+OTP auth, RBAC.
**DB work:** `users`, `roles`, `permissions`, `role_permissions`.
**APIs:** `/auth/*`.
**Frontend:** Login/OTP/registration screens (shared shell).
**Testing:** Unit tests for auth/RBAC.
**Deliverables:** Working auth across all three frontends.
**Dependencies:** None.

### Phase 2 — Farmer Core
**Objectives:** Farmer profile and produce data capture.
**Features:** Profile, farm, land, produce CRUD.
**DB work:** `farmers`, `farms`, `land_records`, `crops`, `produce`.
**APIs:** `/farmers/me*`.
**Frontend:** Profile/Farm/Produce screens.
**Testing:** Unit + integration on CRUD.
**Deliverables:** Farmer can complete profile setup end-to-end.
**Dependencies:** Phase 1.

### Phase 3 — Centres & Slot Booking
**Objectives:** Centre catalog + booking engine.
**Features:** Centre CRUD (admin), slot templates/instances, atomic booking, cancel/reschedule.
**DB work:** `procurement_centres`, `centre_capacity`, `slots`, `bookings`, `booking_status_history`.
**APIs:** `/centres*`, `/bookings*`.
**Frontend:** Centre selection, slot calendar, booking confirmation + token screen.
**Testing:** Concurrency test on last-slot-unit booking.
**Deliverables:** Farmer can book/cancel/reschedule a real slot.
**Dependencies:** Phase 1–2.

### Phase 4 — Real-Time Queue
**Objectives:** Live queue for farmers and operators.
**Features:** Check-in, token generation, WebSocket queue updates, operator queue dashboard.
**DB work:** `queue_entries`, `queue_events`, `check_ins`.
**APIs:** `/queue/*`.
**Frontend:** Farmer live queue screen; Operator queue dashboard (call/check-in/complete).
**Testing:** Race-condition tests (Section 25).
**Deliverables:** End-to-end live queue experience.
**Dependencies:** Phase 3.

### Phase 5 — Procurement
**Objectives:** Verification → weighing → grading → calculation → confirmation.
**Features:** Procurement entry flow, pricing-rule lookup/calculation.
**DB work:** `procurements`, `produce_quality`, `weighing_records`, `pricing_rules`.
**APIs:** `/procurements*`, admin pricing endpoints.
**Frontend:** Operator procurement entry screens; Farmer procurement status.
**Testing:** Correct amount calculation across grade/rate combinations; correction workflow.
**Deliverables:** Full physical-to-digital procurement record.
**Dependencies:** Phase 4.

### Phase 6 — Payment Tracking
**Objectives:** Payment-status lifecycle and history.
**DB work:** `payments`, `payment_status_history`.
**APIs:** `/payments*`, `/webhooks/payments/*` (stub).
**Frontend:** Payment status/history screens; operator manual status update.
**Testing:** Idempotent webhook handling, duplicate-callback test.
**Deliverables:** Transparent payment tracking.
**Dependencies:** Phase 5.

### Phase 7 — Notifications
**Objectives:** Event-driven notification dispatch.
**DB work:** `notifications`, `notification_templates`.
**APIs:** `/notifications*`, admin template endpoints.
**Frontend:** In-app notification center; notification preferences.
**Testing:** Delivery retry/failure fallback tests.
**Deliverables:** All lifecycle events notify farmers correctly.
**Dependencies:** Phases 3–6 (event sources).

### Phase 8 — Admin, Reporting & Analytics
**Objectives:** System configuration and visibility.
**DB work:** `audit_logs`, reporting views/materialized views.
**APIs:** `/admin/*`, `/admin/reports/*`.
**Frontend:** Admin dashboard, Official read-only portal.
**Testing:** RBAC boundary tests, report accuracy tests.
**Deliverables:** Admin can run the system without engineering support; officials can view/export reports.
**Dependencies:** All prior phases (data sources).

### Phase 9 — Security & Testing Hardening
**Objectives:** Production-readiness.
**Features:** Rate limiting, dependency scanning, load testing, penetration-test remediation.
**Testing:** Full load test at MVP targets; security test suite (Section 25).
**Deliverables:** Security sign-off.
**Dependencies:** All prior phases.

### Phase 10 — Deployment
**Objectives:** Ship to production.
**Features:** Dockerization, Nginx config, domain/SSL, monitoring/alerting, backup automation.
**Deliverables:** Production environment live, documented runbooks.
**Dependencies:** Phase 9.

## 30. MVP Definition

**MVP includes:** Farmer registration/login/profile, centre selection, slot booking, digital token, live queue, operator dashboard (check-in/call/complete), procurement entry (weight/grade/calculation), payment-status tracking (manual updates), core notifications (booking, queue-called, procurement/payment completed) via SMS + in-app, basic admin dashboard (centres/slots/pricing/users), and basic reports (daily bookings, queue stats, procurement totals).

**Postponed beyond MVP:** Rescheduling (P1, soon after), walk-in flow (P1), WhatsApp notifications, advanced analytics/anomaly detection, offline operator resync (beyond basic retry), payment gateway integration, multi-language beyond English/Hindi, government official portal exports (can initially reuse admin reports), AI/forecasting features.

## 31. Future Scope (Not MVP)

AI-based demand prediction and queue forecasting; automatic slot optimization; fraud/anomaly detection; voice assistant/IVR; WhatsApp bot; digital document verification; Aadhaar/e-KYC integration *where legally permitted and reviewed*; GIS/map integration; weather integration; crop yield estimation; predictive centre load balancing; advanced analytics; direct government system API integrations; real payment gateway/banking integration.

## 32. User Stories

### Farmer (15)
1. As a farmer, I want to register with my mobile number, so that I can access the platform without needing an email.
2. As a farmer, I want to verify my identity via OTP, so that my account is secure without needing a password.
3. As a farmer, I want to complete my profile in Hindi or English, so that I can use the app comfortably.
4. As a farmer, I want to add my farm and land details, so that my produce records are linked to a real farm.
5. As a farmer, I want to add my expected produce/crop, so that I can book a matching procurement slot.
6. As a farmer, I want to browse nearby procurement centres, so that I can pick the most convenient one.
7. As a farmer, I want to see available slots on a calendar, so that I can pick a time that works for me.
8. As a farmer, I want to book a slot and instantly get a booking ID and token, so that I have proof of my reservation.
9. As a farmer, I want to cancel or reschedule my booking, so that I'm not penalized for a change of plan.
10. As a farmer, I want to see my live queue position and estimated wait time, so that I know when to leave for the centre.
11. As a farmer, I want to receive a notification when I'm being called, so that I don't miss my turn.
12. As a farmer, I want to view my procurement history, so that I can track past sales.
13. As a farmer, I want to see my payment status clearly, so that I know whether I've been paid.
14. As a farmer, I want to download a digital receipt, so that I have a record of the transaction.
15. As a farmer, I want to submit a complaint if something goes wrong, so that I can get help.

### Operator (15)
1. As an operator, I want to see today's bookings for my centre, so that I can plan my day.
2. As an operator, I want to see the live queue, so that I know who to call next.
3. As an operator, I want to call the next farmer with one click, so that the process is fast.
4. As an operator, I want to search for a farmer by mobile number, so that I can find their record quickly.
5. As an operator, I want to check in a farmer, so that they enter the queue.
6. As an operator, I want to verify a farmer's identity, so that fraudulent bookings are caught.
7. As an operator, I want to enter the actual weight of produce, so that the payable amount is accurate.
8. As an operator, I want to select a quality grade from a fixed list, so that grading is consistent.
9. As an operator, I want the system to calculate the amount automatically, so that I don't make manual arithmetic errors.
10. As an operator, I want to confirm a procurement, so that it becomes an official record.
11. As an operator, I want to update payment status, so that farmers can see accurate information.
12. As an operator, I want to register a walk-in farmer, so that I'm not forced to turn away unbooked farmers.
13. As an operator, I want my in-progress entry preserved if my connection drops, so that I don't lose work.
14. As an operator, I want to see a daily summary, so that I can reconcile my shift.
15. As an operator, I want clear error messages, so that I understand why an action failed.

### Administrator (10)
1. As an admin, I want to manage centres, so that I can onboard new procurement locations.
2. As an admin, I want to configure slot templates and capacity, so that centres reflect real operating constraints.
3. As an admin, I want to configure crops, grades, and pricing rules with effective dates, so that rates are always accurate and versioned.
4. As an admin, I want to manage users and roles, so that access is correctly scoped.
5. As an admin, I want to assign operators to centres, so that only authorized staff can process procurements there.
6. As an admin, I want to configure notification templates, so that messaging stays consistent and can be updated without a code change.
7. As an admin, I want to view audit logs, so that I can investigate disputes or errors.
8. As an admin, I want to view system-wide reports, so that I can monitor overall health.
9. As an admin, I want to deactivate a centre or user, so that I can respond to operational issues.
10. As an admin, I want configuration changes to be versioned, so that I can see what changed and when.

### Government Official (5)
1. As a government official, I want to view regional procurement statistics, so that I can monitor program performance.
2. As a government official, I want to monitor payment-pending totals by region, so that I can flag delays.
3. As a government official, I want to view farmer participation numbers, so that I can assess program reach.
4. As a government official, I want to export reports, so that I can share them with my department.
5. As a government official, I want to see anomaly flags (e.g., unusually high cancellations at a centre), so that I can investigate potential issues.

## 33. Acceptance Criteria (selected)

**Authentication**
Given a new user enters a valid mobile number, when they request an OTP, then an OTP is sent and expires after the configured window; when they submit a correct OTP within that window, then they receive valid access and refresh tokens.

**Slot Booking**
Given a farmer is logged in, when an available slot is selected, then the system must reserve it atomically, generate a unique booking ID and token, and display confirmation. If the slot becomes unavailable between selection and confirmation, the system must reject the booking with `SLOT_UNAVAILABLE` and offer the next available slot.

**Queue**
Given a checked-in farmer is in `waiting` status, when an operator calls them, then their status becomes `called`, a WebSocket event is broadcast to their session, and a Critical-priority notification is sent. A second operator attempting to call the same entry must receive `ALREADY_CALLED` and no state change.

**Check-in**
Given a farmer has an active booking for today's slot, when they check in (self-service QR or operator manual), then a queue entry with a unique token is created and the farmer immediately sees their queue position.

**Procurement**
Given an operator has entered weight and grade for a checked-in farmer, when they confirm the procurement, then the system calculates gross/net amount from the active pricing rule, locks the record from direct edits, and creates a linked payment record with status `pending`.

**Payment**
Given a procurement is confirmed, when the payment status changes (manually or via webhook), then the change is recorded in `payment_status_history` and the farmer is notified for `successful` and `failed` transitions.

**Notifications**
Given a Critical-priority event occurs, when SMS/push delivery fails after retries, then an in-app notification must still exist for that user as a guaranteed fallback.

**Admin**
Given an admin updates a pricing rule, when the change is saved, then it takes effect only from its `effective_from` date and does not alter already-confirmed procurements.

**Reports**
Given a date range and centre/district filter, when a report is generated, then all displayed totals must reconcile against the underlying `procurements`/`payments`/`bookings` tables for that filter.

## 34. Non-Functional Requirements

- **Performance/Reliability/Availability:** per Section 21.1 targets; graceful degradation over hard failure (Section 24).
- **Scalability:** horizontal-scale-first architecture (Section 21.2).
- **Security:** per Section 20.
- **Accessibility:** WCAG 2.1 AA-oriented target for farmer-facing screens — large touch targets, sufficient color contrast, text alternatives for icons.
- **Maintainability:** modular backend (clear service boundaries per module), documented API contracts, migration-based schema changes.
- **Usability:** minimal steps to complete core farmer tasks; plain-language error messages in the user's selected language.
- **Compatibility:** modern evergreen browsers + older Android WebView versions common on budget devices.
- **Localization:** all UI strings externalized (no hardcoded text) to support English/Hindi at launch and additional languages later.
- **Observability:** per Section 27.
- **Disaster recovery:** per Section 28.

## 35. Complete Project Directory Structure

```
digital-farmer-platform/
├── frontend/
│   ├── farmer-app/            # React/Next.js PWA
│   ├── operator-dashboard/    # React/Next.js
│   ├── admin-dashboard/       # React/Next.js
│   └── shared-ui/             # Shared components/design tokens
├── backend/
│   ├── src/
│   │   ├── modules/
│   │   │   ├── auth/
│   │   │   ├── farmers/
│   │   │   ├── centres/
│   │   │   ├── bookings/
│   │   │   ├── queue/
│   │   │   ├── procurement/
│   │   │   ├── payments/
│   │   │   ├── notifications/
│   │   │   └── admin/
│   │   ├── common/            # guards, interceptors, RBAC, validation
│   │   ├── websocket/
│   │   └── main.ts
│   └── test/
├── database/
│   ├── migrations/
│   ├── seeds/
│   └── schema.sql
├── workers/
│   ├── notification-worker/
│   ├── slot-generator/
│   └── backup-worker/
├── docs/
│   ├── prd.md
│   ├── api-spec.yaml
│   └── runbooks/
├── infra/
│   ├── terraform/ (optional)
│   └── nginx/
├── docker/
│   ├── docker-compose.dev.yml
│   ├── Dockerfile.backend
│   └── Dockerfile.frontend
└── tests/
    ├── e2e/
    └── load/
```

## 36. Git Workflow

- **Repository:** monorepo as above.
- **Branch strategy:** `main` (production), `staging`, `develop`; feature branches `feature/<module>-<short-desc>`; hotfix branches `hotfix/<desc>`.
- **Commit conventions:** Conventional Commits (`feat:`, `fix:`, `chore:`, `docs:`, `test:`).
- **Pull requests:** required for merges into `develop`/`staging`/`main`; at least one review; CI must pass (lint, unit tests, build).
- **Code reviews:** focus on RBAC correctness, transaction/atomicity correctness for booking/queue/procurement, and test coverage.
- **Issue tracking:** linked to PRD Requirement IDs (e.g., `FR-008`) for traceability.
- **Environment branches:** `develop` → staging environment auto-deploy; `main` → production deploy on tag/release.

## 37. Environment Configuration (examples — no real credentials)

```
DATABASE_URL=postgresql://user:password@host:5432/farmer_platform
REDIS_URL=redis://host:6379
JWT_ACCESS_SECRET=<set-in-secrets-manager>
JWT_REFRESH_SECRET=<set-in-secrets-manager>
OTP_EXPIRY_SECONDS=300
SMS_PROVIDER_API_KEY=<set-in-secrets-manager>
SMS_PROVIDER_SENDER_ID=<set-in-secrets-manager>
FCM_PROJECT_ID=<set-in-secrets-manager>
FCM_PRIVATE_KEY=<set-in-secrets-manager>
OBJECT_STORAGE_BUCKET=farmer-platform-documents
OBJECT_STORAGE_ACCESS_KEY=<set-in-secrets-manager>
OBJECT_STORAGE_SECRET_KEY=<set-in-secrets-manager>
NODE_ENV=production
LOG_LEVEL=info
CORS_ALLOWED_ORIGINS=https://app.example.org,https://admin.example.org
```

## 38. Cost Estimation (categories only — verify current vendor pricing before committing budget)

| Category | Student/Local Dev | Small Pilot | Production |
|---|---|---|---|
| Compute/Server | Free (local Docker) | Low-cost single VM/managed container | Multiple instances + load balancer |
| Database | Local Postgres container | Small managed PostgreSQL instance | Managed PostgreSQL with replica(s) |
| Cache | Local Redis container | Small managed Redis instance | Managed Redis cluster |
| SMS | Free-tier/test credits | Pay-per-message, budget cap | Negotiated volume pricing |
| Push (FCM) | Free | Free | Free (FCM has no send cost) |
| Object storage | Local disk | Small managed bucket | Managed bucket with CDN |
| Domain | Optional | Single domain + SSL (often free via Let's Encrypt) | Domain + SSL, possibly multiple subdomains |
| Monitoring | Free/open-source | Free tier of a hosted APM | Paid APM/monitoring tier |
| Email (ops alerts) | Free tier | Free/low tier | Low-volume transactional email tier |

*No specific vendor prices are asserted here; costs must be confirmed against current vendor pricing pages at planning time.*

## 39. Team Task Distribution (student team assumption)

- **Backend Developer:** Auth, RBAC, booking engine (concurrency-safe), queue service, procurement/payment services, API contracts, background workers.
- **Frontend Developer:** Farmer PWA (booking, queue, history), shared component library, i18n wiring.
- **Database Developer:** Schema design/migrations, indexing, seed data, query optimization, reporting views.
- **UI/UX Developer:** Wireframes/high-fidelity designs for all three apps, accessibility review, Hindi/English content review.
- **Integration Developer** (SMS/Push/optional WhatsApp): notification worker, provider adapters, delivery monitoring.
- **DevOps/Testing:** Docker/CI-CD setup, staging/production environments, monitoring/alerting, backup automation, load/security testing coordination.

## 40. Complete Project Roadmap

```mermaid
flowchart LR
    A[Idea] --> B[Requirements/PRD]
    B --> C[Architecture]
    C --> D[UI/UX Design]
    D --> E[Database Schema]
    E --> F[Backend Development]
    F --> G[Frontend Development]
    G --> H[Integration]
    H --> I[Testing]
    I --> J[Deployment]
    J --> K[Pilot - 1 to 5 centres]
    K --> L[Production Rollout]
```
Dependencies: Architecture depends on finalized requirements; UI/UX and Database can proceed in parallel once architecture is set; Backend depends on Database schema; Frontend depends on Backend API contracts (can start against mocked contracts in parallel); Integration depends on Backend + Frontend reaching feature completeness per module; Testing runs continuously but a dedicated hardening pass precedes Deployment; Pilot precedes full Production rollout to validate assumptions under real conditions.

## 41. Diagrams

### 41.1 Farmer Booking Flow
```mermaid
flowchart TD
    Start([Farmer opens app]) --> Login[Login/OTP]
    Login --> Profile{Profile complete?}
    Profile -->|No| SetupProfile[Complete Profile/Farm/Produce]
    SetupProfile --> SelectCentre
    Profile -->|Yes| SelectCentre[Select Centre]
    SelectCentre --> ViewSlots[View Slot Calendar]
    ViewSlots --> PickSlot[Pick Slot]
    PickSlot --> Confirm{Slot still available?}
    Confirm -->|No| ViewSlots
    Confirm -->|Yes| Reserve[Reserve Atomically]
    Reserve --> Token[Generate Booking ID and Token]
    Token --> Notify[Send Confirmation Notification]
    Notify --> End([Booking Complete])
```

### 41.2 Procurement Flow
```mermaid
flowchart TD
    CheckIn[Farmer Checked In] --> Verify[Operator Verifies Identity]
    Verify --> Weigh[Record Actual Weight]
    Weigh --> Grade[Record Quality Grade]
    Grade --> Calc[System Calculates Amount]
    Calc --> Confirm[Operator Confirms Procurement]
    Confirm --> PayInit[Payment Status: Pending]
    PayInit --> PayProcess[Payment Status: Processing]
    PayProcess --> PaySuccess[Payment Status: Successful]
    PayProcess --> PayFail[Payment Status: Failed]
    PaySuccess --> Receipt[Digital Receipt Generated]
```

### 41.3 Queue Flow
```mermaid
stateDiagram-v2
    [*] --> Waiting: Check-in
    Waiting --> Called: Operator calls
    Called --> Processing: Farmer arrives at counter
    Processing --> Completed: Procurement confirmed
    Waiting --> NoShow: Grace period expires
    Waiting --> Cancelled: Farmer/Operator cancels
    Completed --> [*]
    NoShow --> [*]
    Cancelled --> [*]
```

### 41.4 Authentication Flow
```mermaid
sequenceDiagram
    participant F as Farmer App
    participant API as Backend API
    participant R as Redis
    F->>API: POST /auth/send-otp
    API->>R: Store OTP with TTL
    API-->>F: otp_sent true
    F->>API: POST /auth/verify-otp
    API->>R: Validate OTP
    API-->>F: access_token, refresh_token
```

### 41.5 Sequence — Slot Booking
```mermaid
sequenceDiagram
    participant F as Farmer
    participant API as Backend API
    participant DB as PostgreSQL
    F->>API: POST /bookings {slot_id}
    API->>DB: BEGIN TRANSACTION
    API->>DB: UPDATE slots SET available_capacity -=1 WHERE id=? AND available_capacity>0
    alt capacity available
        DB-->>API: 1 row updated
        API->>DB: INSERT INTO bookings
        API->>DB: COMMIT
        API-->>F: 201 booking_id, token
    else no capacity
        DB-->>API: 0 rows updated
        API->>DB: ROLLBACK
        API-->>F: 409 SLOT_UNAVAILABLE
    end
```

### 41.6 Sequence — Queue Update (WebSocket)
```mermaid
sequenceDiagram
    participant O as Operator
    participant API as Backend API
    participant DB as PostgreSQL
    participant Redis as Redis PubSub
    participant Farmers as Connected Farmer Clients
    O->>API: POST /queue/{id}/call
    API->>DB: Atomic status update waiting to called
    API->>Redis: Publish queue update event
    Redis-->>Farmers: Broadcast to centre room
    Farmers->>Farmers: Update live queue UI
```

### 41.7 Payment Status Flow
```mermaid
stateDiagram-v2
    [*] --> Pending
    Pending --> Processing
    Processing --> Successful
    Processing --> Failed
    Failed --> Processing: Retry
    Successful --> Reversed: Manual reversal
    Pending --> OnHold
    OnHold --> Processing
    Successful --> [*]
    Reversed --> [*]
```

## 42. Error Handling — Standard Format & Error Code Catalogue

All API errors use:
```json
{
  "success": false,
  "error": {
    "code": "SLOT_UNAVAILABLE",
    "message": "This slot is no longer available."
  }
}
```

| Code | HTTP Status | Meaning |
|---|---|---|
| VALIDATION_ERROR | 400 | Request body failed schema validation |
| OTP_INVALID | 400 | Submitted OTP does not match |
| OTP_EXPIRED | 400 | OTP window has passed |
| CANCEL_WINDOW_CLOSED | 400 | Cancellation attempted past policy cutoff |
| INVALID_CREDENTIALS | 401 | Login failed |
| TOKEN_INVALID | 401 | Access/refresh token invalid or expired |
| FORBIDDEN | 403 | Authenticated but not authorized for this resource |
| NOT_FOUND | 404 | Entity does not exist |
| MOBILE_ALREADY_REGISTERED | 409 | Duplicate registration attempt |
| DUPLICATE_BOOKING | 409 | Farmer already booked for that slot |
| SLOT_UNAVAILABLE | 409 | Slot capacity exhausted |
| ALREADY_CALLED | 409 | Queue entry already transitioned by another operator |
| ALREADY_CONFIRMED | 409 | Procurement already confirmed, edit rejected |
| ACCOUNT_LOCKED | 423 | Too many failed auth attempts |
| RATE_LIMITED | 429 | Too many requests (e.g., OTP resend) |
| SERVICE_UNAVAILABLE | 503 | Downstream dependency (DB/Redis) unavailable |

## 43. Success Metrics (KPIs)

- Reduction in average farmer waiting time at centres (vs. pre-digitization baseline).
- Reduction in observed physical queue length/congestion.
- Booking completion rate (bookings started vs. confirmed).
- No-show rate (target: trending down over successive seasons).
- Average procurement processing time per farmer.
- Centre utilization rate (bookings vs. available capacity).
- Notification delivery success rate (target ≥ 95% for Critical events across channels combined, given in-app fallback).
- Payment-status visibility: percentage of procurements with a non-`pending` status within a configurable SLA window.
- Farmer satisfaction (survey/feedback rating).
- Operator efficiency (farmers processed per operator-hour).

## 44. Risks & Mitigations

| Risk | Category | Mitigation |
|---|---|---|
| Poor rural internet connectivity | Technical | Offline-tolerant operator entry, REST polling fallback, low-bandwidth-optimized frontend |
| Low digital literacy among farmers | Adoption | Simple language, large touch targets, icon+text UI, operator-assisted booking option |
| Incorrect farmer-entered data | Operational | Operator verification step before procurement; audit trail for corrections |
| Server downtime during peak season | Technical | Horizontal scaling, health checks, documented DR runbook, load testing ahead of season |
| SMS delivery failures | Technical | Multi-channel notifications with guaranteed in-app fallback for Critical events |
| Queue synchronization issues (race conditions) | Technical | Atomic DB-level transitions, tested concurrency scenarios (Section 25) |
| Fraud (e.g., proxy booking/procurement) | Operational | Identity verification at check-in, audit logging, anomaly flags for officials |
| Operator data-entry mistakes | Operational | Pre-confirmation edit window, post-confirmation correction workflow with audit trail |
| Scalability under rapid centre growth | Technical | Architecture designed to scale horizontally from day one (Section 21) |
| Privacy/security incidents | Security | Encryption, RBAC, minimal data collection, audit logging, incident response runbook |
| Resistance to adoption by centre staff | Adoption | Operator-first UX simplicity, phased rollout, training materials, feedback loop |

## 45. Glossary

- **Slot** — A bookable time window with limited capacity at a procurement centre.
- **Booking** — A farmer's reservation of one capacity unit of a slot.
- **Token** — The daily queue number issued to a farmer at check-in.
- **Queue Entry** — The record tracking a farmer's live position and status in a centre's queue on a given day.
- **Procurement** — The confirmed record of produce weighed, graded, and priced at a centre.
- **Walk-in** — A farmer processed at a centre without a pre-existing slot booking.
- **No-show** — A booking whose farmer did not check in within the grace period.
- **RBAC** — Role-Based Access Control.
- **RTO / RPO** — Recovery Time Objective / Recovery Point Objective.

## 46. Appendices

- **A. Assumptions Log** — All items in this document marked **Assumption** should be reviewed and confirmed with the adopting organization/department before final MVP sign-off, particularly: max active bookings per farmer, cancellation cutoff window, no-show grace period, data retention periods, and any offline check-in capability.
- **B. Out-of-Scope Legal/Policy Items** — Minimum support prices, subsidy rules, e-KYC mandates, and any other government policy values are explicitly outside this PRD's scope and must be supplied and maintained by administrators, not hardcoded by engineering.
- **C. Open Questions for Stakeholders** — Exact list of supported payment methods; whether a payment gateway integration is in scope for v2; which additional Indian languages to prioritize after Hindi; whether Aadhaar/e-KYC will be pursued and under what legal framework.

---

## 47. READY FOR DEVELOPMENT

### Backend Checklist
- [ ] Monorepo/backend project scaffolded with chosen framework
- [ ] Environment config loader + secrets handling in place
- [ ] Auth module (register, OTP, login, refresh, logout) implemented
- [ ] RBAC middleware/guards enforced on all protected routes
- [ ] Booking module with atomic capacity-decrement transaction
- [ ] Queue module with atomic state-transition guard
- [ ] Procurement module with pricing-rule lookup and locking after confirmation
- [ ] Payment module with status history and idempotent webhook handler stub
- [ ] Notification dispatch worker with per-channel retry
- [ ] Audit logging wired into every state-changing service method
- [ ] Standardized error envelope and error-code catalogue implemented

### Frontend Checklist
- [ ] Farmer PWA shell (installable, offline-tolerant shell)
- [ ] Auth screens (register/OTP/login)
- [ ] Profile/Farm/Produce screens
- [ ] Centre selection + slot calendar
- [ ] Booking confirmation + digital token screen
- [ ] Live queue screen (WebSocket + polling fallback)
- [ ] Procurement/payment status + history screens
- [ ] Notification center
- [ ] i18n wired (English/Hindi, no hardcoded strings)
- [ ] Operator dashboard (bookings, queue, verification, weighing, grading, procurement, payment update)
- [ ] Admin dashboard (centres, slots, pricing, users, templates, audit logs, reports)

### Database Checklist
- [ ] All tables from Section 17 created via migrations
- [ ] Foreign keys and unique constraints in place
- [ ] Indexes from Section 17.4 applied
- [ ] Seed data for roles/permissions/crops/grades
- [ ] Read replica configured (staging/production)

### API Checklist
- [ ] All endpoints from Section 18 implemented and documented (OpenAPI/Swagger)
- [ ] Contract tests covering happy path + documented error codes per endpoint

### Authentication Checklist
- [ ] OTP expiry + attempt-limit enforcement
- [ ] JWT access/refresh rotation with reuse detection
- [ ] Account lockout after repeated failures
- [ ] Rate limiting on `/auth/*`

### Queue Checklist
- [ ] Atomic call/complete transitions tested under concurrency
- [ ] WebSocket broadcast on every state change
- [ ] REST polling fallback implemented
- [ ] Reconnection re-sync logic implemented

### Notification Checklist
- [ ] All events in Section 16.1 wired to triggers
- [ ] Templates externalized and admin-editable
- [ ] Retry + guaranteed in-app fallback for Critical events

### Testing Checklist
- [ ] Unit test suite for business logic
- [ ] Integration tests for booking/queue/procurement transactions
- [ ] E2E tests for Journeys A and B
- [ ] Load test at MVP targets (Section 21.1)
- [ ] Security test suite (AuthZ boundaries, OTP brute force)

### Deployment Checklist
- [ ] Dockerfiles for backend/frontend/workers
- [ ] docker-compose for local development
- [ ] Nginx reverse proxy + TLS config
- [ ] CI/CD pipeline (test → build → deploy)
- [ ] Staging environment mirroring production topology

### Security Checklist
- [ ] TLS everywhere, HSTS enabled
- [ ] Secrets in a managed vault, not source control
- [ ] Input validation on all endpoints
- [ ] Rate limiting at gateway
- [ ] Audit logging verified for all sensitive entities
- [ ] Dependency vulnerability scan passing

---

## 48. NEXT DEVELOPMENT PROMPTS

The following prompts are intended to be given to Claude **one at a time, in order**, to build the project module-by-module from this PRD. **Every prompt below must be prefixed with an instruction to inspect the existing project state before making changes, preserve all previously implemented functionality, and implement only the next module without breaking prior modules.**

1. **Project Initialization** — Scaffold the monorepo structure from Section 35, set up the chosen backend framework and frontend framework(s), configure environment variables per Section 37, and set up base CI (lint + test on push).
2. **Database Schema** — Implement the full PostgreSQL schema from Section 17 as migrations, with seed data for roles, permissions, crops, and grades.
3. **Backend Architecture** — Set up the modular backend structure (Section 35), request validation, standardized error envelope (Section 42), and logging.
4. **Authentication** — Implement `/auth/*` endpoints, OTP flow, JWT issuance/rotation, and RBAC middleware.
5. **Farmer Module** — Implement farmer profile, farm, land, and produce CRUD and endpoints.
6. **Centre Module** — Implement procurement centre CRUD (admin) and public centre listing endpoints.
7. **Slot Booking** — Implement slot templates/instances and the atomic booking engine (Section 12.3), plus cancel/reschedule.
8. **Real-Time Queue** — Implement check-in, token generation, WebSocket layer, and the atomic call/complete state machine (Section 13).
9. **Procurement** — Implement verification/weighing/grading/calculation/confirmation workflow and pricing-rule lookup.
10. **Payment Tracking** — Implement payment status lifecycle, history, and the webhook stub with idempotency.
11. **Notifications** — Implement the notification worker, templates, and channel adapters (SMS/push/in-app) with retry and fallback.
12. **Admin Dashboard (Backend)** — Implement all `/admin/*` endpoints for user/centre/pricing/template management and audit-log querying.
13. **Farmer Frontend** — Build the full farmer PWA per Section 11.1, wired to the real API.
14. **Operator Frontend** — Build the operator dashboard per Section 11.2, wired to the real API and WebSocket layer.
15. **Testing** — Implement the full test suite per Section 25, including concurrency and idempotency test cases.
16. **Docker** — Containerize all services and provide docker-compose for local development.
17. **Deployment** — Set up staging/production infrastructure per Section 26, including Nginx, TLS, and environment configuration.
18. **Security Audit** — Run through the Security Checklist (Section 47), fix findings, and add missing rate limiting/validation.
19. **Performance Optimization** — Run load tests against Section 21.1 targets, optimize slow queries/endpoints, tune indexes and caching.
20. **Final Integration** — End-to-end verification of all Journeys (Section 9), full regression pass, and production readiness sign-off against the checklists in Section 47.

*(End of PRD)*
