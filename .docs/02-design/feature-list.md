# EventMFU — Feature List

**Source:** `docs/REQUIREMENTS_DESIGN.md` (v18)
**Last updated:** 2026-09-08

---

## Account & Authentication

| # | Feature | Notes |
|---|---------|-------|
| A1 | University-email signup with domain verification | `@mfu.ac.th` enforced; verification link mailed (mocked in dev) |
| A2 | Email + password login / session management | httpOnly cookie session; no SSO |
| A3 | Password reset via verified email link | Same pattern as signup verification |
| A4 | Consent capture at signup | PDPA: document version, hash, timestamp, source IP |
| A5 | Admin provisioning (out-of-band) | No self-registration path to Admin |

---

## Event Discovery

| # | Feature | Notes |
|---|---------|-------|
| D1 | Personalized event feed | Filtered by User's school/year; open events always included |
| D2 | Event detail page | Title, description, poster, date/time, venue, audience, check-in mode |
| D3 | Audience targeting on events | open / school / year / school+year — set at request time, confirmed at approval |
| D4 | Per-event Q&A thread | Any logged-in user can ask; Main Organizer/Co-Organizer answers |
| D5 | Q&A answer visibility | Answers visible to all event viewers (not just the asker) |

---

## Event Request & Approval

| # | Feature | Notes |
|---|---------|-------|
| R1 | Event Request form | title, description, agenda, equipment_needs, contact_phone, venue_preference, start/end, capacity, audience, checkin_mode, points_value |
| R2 | Admin request queue | Lists pending requests; each shows requester eligibility snapshot (organizer_restricted, health score, prior history) |
| R3 | Three review outcomes | approve / request-changes (needs_info + admin_feedback) / reject |
| R4 | needs_info / resubmit loop | Requester revises and resubmits; revision_count tracked; loops until Admin decides |
| R5 | Approval bundles venue + date/time | Admin picks a free registry venue and confirms date/time in the same step — no separate venue-assignment step for a fresh request |
| R6 | Venue overlap warning | Warns Admin on conflict; not a hard block; `force: true` to override |
| R7 | Admin direct event creation | Admin skips the request form; picks venue immediately; source_request_id = null |
| R8 | Venue registry management | Admin adds/edits/retires venue entries (name, building, capacity, notes) |
| R9 | Venue reassignment | Admin can change a venue/date after initial assignment |
| R10 | Publish gate | Event cannot reach `published` status while `venue_id` is null |

---

## Team Recruitment

| # | Feature | Notes |
|---|---------|-------|
| T1 | Direct add | Main Organizer searches user directory (live-filtered) and assigns role or Contributor position |
| T2 | Open call | Public posting with slot count; users claim first-come; auto-closes at capacity |
| T3 | Application method | Posting with custom organizer-written questions; applicants submit answers; Main Organizer accepts/rejects |
| T4 | Staff Openings browse | Any authenticated User can see and claim/apply to open calls and applications |
| T5 | Organizer role types | main_organizer / co_organizer / checkin_staff — each with distinct permissions |
| T6 | Contributor positions | No platform permissions; title set by Main Organizer (preset or custom); shown on event roster |
| T7 | Booking / team exclusivity | Holding a team row (organizer or Contributor) on an event is mutually exclusive with a Booking on the same event |
| T8 | Last-main-organizer guard | Server-side block on removing the last main_organizer row from an event |
| T9 | Main-Organizer-only team controls | Co-Organizers and Check-in Staff cannot post calls, review applications, or change the team |

---

## Check-In & Attendance

| # | Feature | Notes |
|---|---------|-------|
| C1 | Per-event check-in mode | Organizer chooses staff_scan or self_scan at request time; editable afterward |
| C2 | Staff-scan mode | Organizer scans attendee's `Booking.qr_token`; validates window |
| C3 | Self-scan mode | Student scans `Event.checkin_qr_token` via app; session user's booking looked up and marked attended |
| C4 | Staff-scan fallback on self-scan events | `Booking.qr_token` always generated; usable if student's phone won't cooperate |
| C5 | Check-in window enforcement | Scans outside the event's start_time–end_time window are rejected |
| C6 | Live attendance view | Organizers see coming/not-coming in real time during the event |
| C7 | Post-event no-show sweep | Any booking still `booked` after event ends → `no_show`; triggers HealthTransaction penalty |

---

## Reviews & Quality Signals

| # | Feature | Notes |
|---|---------|-------|
| Q1 | Post-event review submission | rating (1–5) + comment + anonymous flag; one review per attended booking |
| Q2 | Anonymous reviews | Reviewer's identity stored internally but redacted to null on every read path (organizer, other users, Admin) |
| Q3 | External sentiment analysis | Review comment sent to external service; result stored on Review record asynchronously |
| Q4 | Combined negative classification | rating ≤ cutoff OR sentiment label = negative + confidence ≥ threshold; both signals admin-configurable |
| Q5 | Organizer flag auto-generation | Ratio of negative reviews (Main Organizer only) exceeds threshold + volume gate → OrganizerFlag; Admin always decides the action |
| Q6 | Flag evidence view | Admin sees ratio, review count, window, sample of negative reviews (anonymity still enforced in this view) |
| Q7 | Admin flag resolution | dismiss / warn / restrict; restrict → organizer_restricted = true |

---

## Health System

| # | Feature | Notes |
|---|---------|-------|
| H1 | Health score ledger | HealthTransaction records every change; User.health_score = running total |
| H2 | No-show penalty | Configurable deduction per no-show booking |
| H3 | Review reward | Configurable credit for submitting a review |
| H4 | Health flag auto-generation | Score drops below threshold → UserHealthFlag raised; Admin decides |
| H5 | Admin health flag resolution | dismiss / warn / restrict; restrict → booking_restricted = true |
| H6 | Booking restriction scope | booking_restricted blocks new bookings only; login, browsing, and organizer roles unaffected |

---

## Recognition & Personal Record

| # | Feature | Notes |
|---|---------|-------|
| Re1 | Attendee track title | Newcomer / Regular Attendee / Super Attendee / Campus Legend (thresholds configurable) |
| Re2 | Organizer track title | First-Time Organizer / Event Organizer / Seasoned Organizer / Master Organizer |
| Re3 | Contributor track title | Contributor / Active Contributor / Veteran Contributor |
| Re4 | Primary title | Highest tier across tracks; tie-break Organizer > Contributor > Attendee |
| Re5 | 6 badges | Reliable Attendee, Voice of Campus, Big Stage, Community Builder, Fan Favorite, Team Player |
| Re6 | Computed on every read | No separate stored ledger; re-reads Booking/Review/EventOrganizer/EventContributor tables |
| Re7 | My Record page | Full history: attendee, organizer, contributor; print-friendly layout |
| Re8 | Public record view | Any authenticated User can view any other User's record; accessible by clicking credited name |

---

## Points System

| # | Feature | Notes |
|---|---------|-------|
| Po1 | Attendance points | Credited for every attended booking on a `points_value > 0` event |
| Po2 | Points history | User can view their own PointsTransaction ledger |
| Po3 | Sync-ready schema | `sync_status` / `synced_at` fields exist; external sync deferred to Section 14 |

---

## Admin Platform Tools

| # | Feature | Notes |
|---|---------|-------|
| Ad1 | Platform-wide summary dashboard | Total events, bookings, pending requests, unresolved flags, open health flags |
| Ad2 | Activity log viewer | Append-only; Admin-only read; all state-changing actions across the system |
| Ad3 | Admin-configurable thresholds | PlatformSettings: flag thresholds, health penalties/rewards, tier/badge cutoffs, check-in grace period |

---

## Compliance (enforced by rule.md)

| # | Feature | Notes |
|---|---------|-------|
| Co1 | Password stored as hash only | bcrypt/argon2; plaintext never logged |
| Co2 | Append-only ActivityLog | Never editable; Admin-only access; 90-day retention floor |
| Co3 | Account deletion with log carve-out | Profile data nulled/tombstoned; ActivityLog within 90-day window retained (Computer Crime Act §26) |
| Co4 | E-signature evidence on agreements | Consent, bookings, requests: authenticated user id + payload + timestamp + source IP stored |
| Co5 | Anonymous review redaction enforced server-side | Never leaks through any API response, log line, or Admin view |
| Co6 | Automated flags, never auto-restrictions | System signals; Admin always decides |
