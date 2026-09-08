# EventMFU — Product Backlog

**Source:** `docs/REQUIREMENTS_DESIGN.md` (v18)
**Team:** Nay Lin Htet, Lin Myat Oo, Min Htet Kaung Pyae, Myo Min Min Oo
**Last updated:** 2026-09-08

Every item below is traced to a real user pain observed during research. Pain codes:

| Code | User Pain |
|------|-----------|
| P1 | Events are scattered across LINE groups, notice boards, and department pages — students miss events relevant to them |
| P2 | No single place to register; students DM organizers or fill out Google Forms per-event |
| P3 | Organizers have no reliable way to track who actually showed up vs. who just registered |
| P4 | No-shows are invisible — organizers over-prepare for attendance that never comes |
| P5 | Students have no way to prove they organized or contributed to an event (no sharable record) |
| P6 | Admins have no visibility across events running on campus simultaneously; venue double-bookings happen |
| P7 | Becoming an event organizer has no transparent process — it depends on who you know |
| P8 | Negative event experiences go unreported because students fear retaliation from organizers |
| P9 | Organizers recruit team members via personal networks only; no public way to join a team |

---

## Epic 1 — Authentication & Accounts

### US-101 · University-verified signup
**As a** student, **I want to** create an account using my university email, **so that** the platform is scoped to the campus community without needing IT to set up SSO.
**Pain:** P1 (any outside tool can be used; this keeps it campus-only)
**Acceptance criteria:**
- Email must match the university domain (`@mfu.ac.th`)
- Verification link sent before account is activated
- `name`, `school`, `year` collected at signup for audience targeting
**Priority:** P0 · **Phase:** B1, F0

### US-102 · Email + password login / password reset
**As a** registered user, **I want to** log in with email and password and reset my password via email link, **so that** I can access my account without needing SSO.
**Pain:** P1
**Acceptance criteria:**
- Login returns a session token stored in an httpOnly cookie
- Password reset follows the same verified-email-link pattern as signup
**Priority:** P0 · **Phase:** B1, F0

### US-103 · Out-of-band Admin provisioning
**As an** Admin, **I want** my account to be created by an existing Admin or at deployment, **so that** there is no self-registration path to Admin.
**Pain:** P6 (Admin integrity)
**Priority:** P0 · **Phase:** B1

---

## Epic 2 — Event Discovery & Audience Targeting

### US-201 · Filtered event feed
**As a** student, **I want to** see only events relevant to my school and year (plus open events), **so that** I'm not overwhelmed by events that don't apply to me.
**Pain:** P1
**Acceptance criteria:**
- Feed filters by `audience_type`: open / school / year / school+year
- School and year from User profile drive the filter automatically
**Priority:** P0 · **Phase:** B2, F1

### US-202 · Event detail page with Q&A
**As a** student, **I want to** read event details and ask questions before deciding to book, **so that** I have enough information to commit.
**Pain:** P2 (friction in getting information)
**Acceptance criteria:**
- Public Q&A thread per event; any logged-in user can post
- Main Organizer or Co-Organizer can answer; answers visible to all viewers
**Priority:** P1 · **Phase:** B2, F1

---

## Epic 3 — Event Request & Organizer Pathway

### US-301 · Event Request submission
**As a** student who wants to run an event, **I want to** submit a structured proposal (title, description, agenda, venue preference, date/time, equipment needs, audience, check-in mode), **so that** Admin has everything needed to make a decision without back-and-forth.
**Pain:** P7 (transparent process for becoming an organizer)
**Acceptance criteria:**
- Fields: title, description, agenda, equipment_needs, contact_phone, venue_preference, start/end, capacity, audience_type, checkin_mode, points_value
- Submitted request appears with status `pending` in user's dashboard
**Priority:** P0 · **Phase:** B2, F1

### US-302 · Admin reviews requests with eligibility snapshot
**As an** Admin, **I want to** see each requester's `organizer_restricted` status, health score, and prior organizing/flag history alongside the request, **so that** I can make an informed decision without a separate lookup.
**Pain:** P6 (visibility), P7 (transparent decision-making)
**Acceptance criteria:**
- Eligibility snapshot computed at read time; no new stored field
- A request from an `organizer_restricted` user cannot be approved
**Priority:** P0 · **Phase:** B2, F1

### US-303 · Three-outcome review: approve / request-changes / reject
**As an** Admin, **I want to** send a request back for clarification (needs_info) instead of only approve or reject, **so that** I don't have to reject good proposals over missing details.
**Pain:** P7
**Acceptance criteria:**
- `needs_info` → requester notified → revises and resubmits → re-enters queue with `revision_count` incremented
- Loop repeats until Admin approves or rejects
**Priority:** P0 · **Phase:** B2, F1

### US-304 · Approval bundles venue + date/time confirmation
**As an** Admin, **I want to** pick a free venue and confirm the date/time in the same step as approving a request, **so that** approved events always have a venue immediately (no "approved but venue TBD" limbo).
**Pain:** P6
**Acceptance criteria:**
- Admin sees registry venues matching the stated preference, each flagged free/conflict for the requested window
- Overlap warning on conflict (not a hard block); `force: true` to proceed
- Event is created with `venue_id` and `start_time`/`end_time` already set; `status = draft`; requester becomes Main Organizer
**Priority:** P0 · **Phase:** B2, F1

### US-305 · Admin direct event creation
**As an** Admin, **I want to** create an event directly (without a student request), **so that** top-down university-wide events don't require Admin to write and approve its own request.
**Pain:** P6
**Acceptance criteria:**
- Same fields as Event Request; Admin picks venue directly
- `source_request_id = null`; Admin added as Main Organizer (`joined_via: admin_direct_create`)
- Same venue-overlap warning applies
**Priority:** P1 · **Phase:** B2, F1

### US-306 · Venue registry management
**As an** Admin, **I want to** maintain a registry of campus rooms (name, building, capacity, notes), **so that** venue assignment uses real, up-to-date room data.
**Pain:** P6 (double-booking)
**Acceptance criteria:**
- Admin can add/edit/retire venue entries
- Retired venues are no longer assignable
**Priority:** P0 · **Phase:** B2, F1

---

## Epic 4 — Team Recruitment

### US-401 · Direct add a team member
**As a** Main Organizer, **I want to** search users by name or email and assign them a role or Contributor position directly, **so that** I can immediately add people I've already lined up.
**Pain:** P9
**Acceptance criteria:**
- Live-filtered directory search as you type
- Creates EventOrganizer (role) or EventContributor (position) row; cancels any existing Booking by that user on this event
**Priority:** P1 · **Phase:** B3, F2

### US-402 · Post an open call
**As a** Main Organizer, **I want to** post a public opening with a slot count, **so that** interested students can self-select on a first-come basis without my involvement per person.
**Pain:** P9
**Acceptance criteria:**
- Appears under Staff Openings; any User can claim a slot
- Auto-closes once `slots_filled` reaches `slots_total`
- Main Organizer only can post; Co-Organizer and Check-in Staff cannot
**Priority:** P1 · **Phase:** B3, F2

### US-403 · Application-method posting with custom questions
**As a** Main Organizer, **I want to** post a role/position that requires an application with my own questions, **so that** I can select contributors based on relevant answers (not just first-come).
**Pain:** P9
**Acceptance criteria:**
- Custom questions attached to the call; applicants answer each one
- Main Organizer reviews applications, accepts/rejects; accept creates the role/position row
**Priority:** P1 · **Phase:** B3, F2

### US-404 · Browse Staff Openings
**As a** student, **I want to** browse open calls and applications for events looking for helpers, **so that** I can join an event team even if I don't know the organizers personally.
**Pain:** P9
**Acceptance criteria:**
- Staff Openings visible to any authenticated User
- Claim (open call) or Apply (application method) from the same view
**Priority:** P1 · **Phase:** B3, F2

### US-405 · Booking / team mutual exclusivity
**As a** student on an event team, **I want** the app to not let me also hold a booking as an attendee on the same event, **so that** the organizer's team roster and the attendee list don't overlap.
**Pain:** P3 (clean attendance data)
**Acceptance criteria:**
- Adding a team row auto-cancels any existing Booking by that user on the event
- Booking blocked if user already holds an EventOrganizer or EventContributor row on the event
**Priority:** P0 · **Phase:** B3

---

## Epic 5 — Check-In & Attendance

### US-501 · Staff-scan check-in
**As an** organizer with check-in access, **I want to** scan a student's personal booking QR code to check them in, **so that** I confirm each attendee individually with high integrity.
**Pain:** P3, P4
**Acceptance criteria:**
- `Booking.qr_token` scanned by Main Organizer, Co-Organizer, or Check-in Staff
- Valid only within the event's check-in window
- Booking status → `attended`; timestamp recorded
**Priority:** P0 · **Phase:** B4, F3

### US-502 · Self-scan check-in
**As a** student attending a large event, **I want to** scan a venue QR code myself to check in, **so that** there's no queue bottleneck at the door.
**Pain:** P3, P4
**Acceptance criteria:**
- One `Event.checkin_qr_token` displayed at the venue
- Session user's own booking is looked up and marked attended; repeat scan is a no-op
- Valid only within the check-in window
**Priority:** P0 · **Phase:** B4, F3

### US-503 · Post-event no-show sweep
**As an** organizer, **I want** unchecked bookings to automatically become no-shows after the event ends, **so that** the attendance record is complete without manual cleanup.
**Pain:** P4
**Acceptance criteria:**
- Scheduled sweep (or admin dev trigger): any `booked` booking past `end_time` → `no_show`
- Triggers `HealthTransaction` penalty for each no-show user
**Priority:** P0 · **Phase:** B4

---

## Epic 6 — Reviews & Quality Signals

### US-601 · Post-event review with anonymous option
**As a** student who attended an event, **I want to** leave a rating and comment — optionally anonymously — **so that** I can give honest feedback without risking retaliation.
**Pain:** P8
**Acceptance criteria:**
- One review per user per attended event
- `is_anonymous` choice at submission; `user_id` stored but redacted to null on every read (organizer, other users, Admin)
- Submitting a review grants a health score reward
**Priority:** P0 · **Phase:** B5, F4

### US-602 · Sentiment-enriched review flagging
**As an** Admin, **I want** the system to automatically flag organizers whose reviews trend negative (by rating + sentiment), **so that** I can intervene before a pattern becomes a problem.
**Pain:** P8
**Acceptance criteria:**
- External sentiment service called per review; result stored on Review record
- Ratio threshold + volume gate checked on each new review or sentiment result
- OrganizerFlag raised for Main Organizers only; Admin decides to dismiss/warn/restrict — never auto-restricted
**Priority:** P1 · **Phase:** B5

### US-603 · Admin resolves organizer and health flags
**As an** Admin, **I want to** see flag details (ratio, review sample, flag history) and choose dismiss/warn/restrict, **so that** I always make the final call — the system only surfaces signals.
**Pain:** P6, P8
**Acceptance criteria:**
- Flag queue with evidence view
- Restrict → `User.organizer_restricted = true` (blocks new requests and team adds); does not affect booking-as-user
**Priority:** P1 · **Phase:** B5, F4

---

## Epic 7 — Health System

### US-701 · Health score tracks no-shows and positive activity
**As a** student, **I want** my health score to reflect my booking reliability, **so that** I have an incentive to cancel bookings I can't make instead of silently no-showing.
**Pain:** P4
**Acceptance criteria:**
- `health_noshow_penalty` deducted per no-show; `health_review_reward` credited per review submitted
- Penalty/reward amounts admin-configurable via PlatformSettings
**Priority:** P1 · **Phase:** B4–B5

### US-702 · Admin reviews and acts on health flags
**As an** Admin, **I want** to be notified when a user's health drops below the threshold, **so that** I can decide whether to restrict their ability to make new bookings.
**Pain:** P4
**Acceptance criteria:**
- UserHealthFlag created automatically when health crosses threshold; Admin decides dismiss/warn/restrict
- Restrict → `User.booking_restricted = true`; user can still browse and view history
**Priority:** P1 · **Phase:** B5

---

## Epic 8 — Recognition & Personal Record

### US-801 · Computed title and badges
**As a** student, **I want** my participation to generate a visible title and badges automatically, **so that** my activity is recognized without me having to do anything extra.
**Pain:** P5
**Acceptance criteria:**
- Three tracks: Attendee / Organizer / Contributor (thresholds from PlatformSettings)
- Six badges (Reliable Attendee, Voice of Campus, Big Stage, Community Builder, Fan Favorite, Team Player)
- Primary title = highest tier reached; tie-break Organizer > Contributor > Attendee
- All computed live from existing tables; nothing stored separately
**Priority:** P2 · **Phase:** B6, F5

### US-802 · My Record page
**As a** student, **I want** a personal record page showing my attending, organizing, and contributing history, **so that** I can share it as a reference in a resume or job application.
**Pain:** P5
**Acceptance criteria:**
- Attendee history (events booked, attended, no-shows, reviews written)
- Organizer history per event (role, date, capacity, attendance rate, avg rating)
- Contributor history per event (position, date)
- Print-friendly layout; no PDF required for this pass
**Priority:** P2 · **Phase:** B6, F5

### US-803 · Public record view (clickable names)
**As any** logged-in user, **I want to** click an organizer or contributor's name anywhere in the app and see their record, **so that** I can evaluate their track record before joining or attending their event.
**Pain:** P7 (transparency about organizers)
**Acceptance criteria:**
- `GET /api/user/:id/record` available to any authenticated user
- Clicking a credited name (event roster, Q&A, review byline) opens that user's record
**Priority:** P2 · **Phase:** B6, F5

---

## Epic 9 — Points System

### US-901 · Local attendance points
**As a** student, **I want to** earn points for attending events, **so that** my participation is quantified locally on the platform.
**Pain:** P5 (quantified activity)
**Acceptance criteria:**
- `PointsTransaction` credited for every `attended` booking where `points_value > 0`
- User can view their own points history
- `sync_status` field exists but no external sync in this pass (Section 14 deferred)
**Priority:** P2 · **Phase:** B4–B6

---

## Epic 10 — Compliance & Audit

### US-1001 · Consent-gated signup
**As a** user, **I want to** explicitly agree to the privacy notice before my account is created, **so that** my consent is recorded with a document version and timestamp.
**Pain:** Legal requirement (PDPA)
**Acceptance criteria:** consent version + timestamp + source IP stored; no account created without it
**Priority:** P0 · **Phase:** B1

### US-1002 · Append-only activity log
**As an** Admin, **I want** every state-changing action logged with actor, action, target, and timestamp, **so that** I can audit any incident and meet the 90-day log retention requirement.
**Pain:** P6 (accountability)
**Acceptance criteria:**
- ActivityLog row on every login, request decision, venue assignment, booking, check-in, review, staff action, flag resolution
- Logs retained ≥ 90 days; account deletion does not cascade-delete logs within that window
**Priority:** P0 · **Phase:** B1+

---

## Backlog status summary

| Phase | Items | Priority |
|-------|-------|----------|
| B0 | Data model migration | prerequisite |
| B1, F0 | Auth (US-101–103, US-1001) | P0 |
| B2, F1 | Event Request + Venue (US-201–306) | P0–P1 |
| B3, F2 | Team recruitment (US-401–405) | P0–P1 |
| B4, F3 | Check-in (US-501–503) | P0 |
| B5, F4 | Reviews + flags + health (US-601–702) | P0–P1 |
| B6, F5 | Recognition (US-801–803, US-901) | P2 |
