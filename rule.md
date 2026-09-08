# EventMFU — Legal & Compliance Rules (rule.md)  
**Note:written by Nay Lin Htet**
**Team:Lin Myat Oo, Min Htet Kaung Pyae,Myo Min Min Oo**

**Read this before writing any code that touches user data or user actions.**

Scope: EventMFU is a standalone university event-booking platform (see
`docs/REQUIREMENTS_DESIGN.md`). It stores student personal data, keeps activity logs, and records
user agreements ("I agree", bookings, reviews). Three Thai laws govern that: **PDPA**, **Computer
Crime Act §26**, and the **Electronic Transactions Act §9/26/28**. The rules below map each law onto
EventMFU's concrete entities (`User`, `ActivityLog`, `Review`, `Booking`, `EventRequest`, etc.).

> These are engineering rules derived from the statutes for day-to-day implementation. They are not
> a substitute for sign-off by a qualified Thai lawyer / the Data Protection Officer before launch.

---

## PDPA (Personal Data Protection Act, B.E. 2562 / 2019)

**What it is:** Thailand's general data-protection law (close to the EU GDPR): it governs how any
organization collects, uses, stores, and discloses the personal data of individuals in Thailand.

**What it requires:** a lawful basis (usually **consent**) · **purpose limitation** (use data only
for the stated purpose) · **data minimisation** · honoring data-subject rights to **access /
correct / delete** and withdraw consent · extra protection for **sensitive data**; plus security
safeguards and breach notification.

### Rules for the agent (write as many as you can):

- If the system stores **User.email, name, school, year** (Section 2 signup), it must first show a
  privacy notice and record explicit consent, storing the consent version + timestamp — no account
  is created without it.
- If the system stores **User.password**, it must persist only `password_hash` (bcrypt/argon2),
  never the plaintext, and must never write the password (or the hash) to any log, error message, or
  `ActivityLog.metadata`.
- If the system collects **school / year**, it must use them **only** for audience targeting and
  feed filtering (Section 4) — never repurpose them for profiling, marketing, or disclosure to other
  users without fresh consent (purpose limitation).
- If the system needs to identify a student, it must collect the **minimum** fields listed in the
  data model (Section 10) and nothing more — do not add phone, national ID, address, photo, or any
  field the spec doesn't require (minimisation).
- If the system exposes **My Record / booking history / health history / points** (Section 3.1,
  Section 15), that satisfies the **right of access** — every field stored about a user must be
  reachable by that user through a read path.
- If the system lets a user **edit or delete their own reviews and questions** (Section 3.1), it must
  actually mutate/remove the stored row (right to correct/delete), not just hide it in the UI.
- If a user requests **account deletion**, it must delete or irreversibly anonymise their personal
  data (name, email → nulled/tombstoned), **except** data a different law forces us to keep — see
  the Computer Crime Act §26 log-retention carve-out below; document that exception in the deletion
  routine.
- If the system sends **Review.comment text to the external sentiment service** (Section 8), it must
  treat that as disclosure to a third-party processor: a data-processing agreement must exist, the
  privacy notice must disclose it, and if that service is hosted abroad the cross-border-transfer
  conditions of the PDPA must be met before the call is made.
- If the system offers **anonymous reviews** (Section 8), it must enforce redaction-on-read on every
  path (organizer, other users, **and Admin**) while still using the stored `user_id` only internally
  for dedup and the health reward — the identity must never leak through any API response, log line,
  or flag-evidence view.
- If the system runs **automated health/organizer flagging that can restrict a user** (Sections 7–8),
  it must keep Admin as the human decision-maker (the system flags, never auto-restricts) and must
  log the flag evidence, because PDPA restricts decisions taken solely by automated processing.
- If the system stores a **health_score / behavioral signals**, it must not expose one user's health,
  no-show history, or flags to any other user — only to the user themselves and to Admin.
- If the system would ever collect **sensitive data** (health condition, religion, disability, etc.),
  it must **not** — none of EventMFU's features require a special-category field; if a new feature
  seems to, stop and escalate to the DPO rather than adding the column.
- If a **personal-data breach** is detected, the system must support notifying the DPO/authority
  within the statutory window (72 hours) — so security-relevant events must be logged with enough
  detail to scope a breach.
- If the system emails users (verification, password reset — Section 2), it must send only
  transactional messages tied to the consented purpose; no marketing send without a separate opt-in.

---

## Computer Crime Act §26 (B.E. 2550 / 2007, amended B.E. 2560 / 2017)

**What it is:** Thai law imposing duties on "service providers" (which EventMFU is) around computer
misuse and, specifically in §26, the retention of traffic/log data.

**What it requires:** keep **computer traffic (access) log data for at least 90 days** (extendable by
official order up to ~1–2 years), and keep it in a form that lets a **specific real user** be
identified from it.

### Rules for the agent:

- If the system has a **login endpoint** (Section 2), it must write an `ActivityLog` entry for every
  login with: `actor_id` (real user), source IP, user-agent, and timestamp.
- If the system performs any **user action that changes state** — Event Request submit/decision,
  venue assignment, event edit, booking, cancellation, check-in, review, Q&A, staff recruitment,
  flag resolution, admin action (Section 10 `ActivityLog`) — it must append a log row with actor,
  action_type, target, and timestamp.
- If a log entry is written, it must be **tied to a real, identifiable user** — the university-email
  verification at signup (Section 2) is what makes `actor_id` traceable, so anonymous/unverified
  accounts must never be able to take loggable actions.
- If a review is **anonymous** (Section 8), the `ActivityLog` entry records `actor_id = null` for
  display, but the system must still be able to identify the real actor internally if lawfully
  compelled — do not destroy that linkage; only redact it on read.
- If any process would **delete `ActivityLog` rows**, it must retain them for **at least 90 days**
  from creation — no retention job, account deletion, or "clear logs" action may remove a log entry
  younger than 90 days.
- If **account deletion** runs (PDPA), it must **not** cascade-delete `ActivityLog` traffic data
  inside the 90-day window — anonymise the user's *profile* data but keep the minimum access log the
  Computer Crime Act requires; this carve-out must be explicit in code and comments.
- If logs are stored, they must be **tamper-evident and Admin-only** — `ActivityLog` is append-only
  (Section 10), never editable through any route, and readable only by role = admin.
- If the system stores logs, it must record timestamps in a **consistent, unambiguous timezone**
  (UTC stored, Asia/Bangkok for display) so a 90-day window and any investigation are computable.
- If a scheduled purge/rotation is added, it must be configured to retain ≥90 days by default and
  the retention length must be an admin-visible setting, not a hardcoded value shorter than 90 days.

---

## Electronic Transactions Act §9 / 26 / 28 (B.E. 2544 / 2001)

**What it is:** the Thai law that gives electronic records, agreements, and signatures the same legal
effect as paper — so a user's "I agree", a submitted Event Request, or a confirmed booking is legally
binding when done correctly.

**What it requires:** a **valid e-signature test (§9)** — a method that identifies the signatory,
shows their approval, and is reliable/appropriate for the purpose; a **presumed-reliable signature
(§26)** — creation data uniquely linked to and controlled by the signatory, with any later alteration
detectable; and **CA duties (§28)** — obligations on a certification authority if certificates are
relied on.

### Rules for the agent:

- If the user clicks **"I agree" on the Terms / Privacy notice at signup** (Section 2), the system
  must record: the authenticated user id, the exact document **version (and a content hash)**, the
  timestamp, and the source IP — this is the §9 evidence that a specific person approved a specific
  text.
- If the user submits an **Event Request, a Booking, a Review, or a StaffApplication** (Section 11),
  the system must treat it as a signed electronic record: capture who (session user), what (the exact
  payload), and when, so the action can be attributed and its content is fixed.
- If any agreement/record is stored, it must be **linked to the authenticated session user** (§26:
  the signature is under the signatory's control and uniquely tied to them) — never accept an "agree"
  or a booking from an unauthenticated or spoofable identity (no `x-user-id` header trust in prod;
  use the real session/token from B1).
- If a stored agreement or e-record could be altered later, the system must make **alteration
  detectable** (§26) — store the document/version hash alongside the consent row, and keep records
  append-only or checksummed so a change is provable.
- If the Terms or Privacy notice **changes**, the system must version it and **re-record consent**
  against the new version — an old "I agree" does not cover new text.
- If the system relies on an **email-verification / certificate / CA** for identity (Section 2, or a
  future SSO in Section 14), it must verify that authority's validity before trusting it and follow
  §28 CA-reliance duties — EventMFU is not itself a certification authority, so do **not** build one;
  if a real digital-certificate flow is ever introduced, escalate for §28 review first.
- If a check-in or booking QR token (`Booking.qr_token`, `Event.checkin_qr_token`, Section 6) stands
  in for a signed attendance record, it must be unguessable, uniquely tied to the user+event, and
  validated server-side within the check-in window — so the electronic attendance record is reliable.
- If the system shows a user the result of a signed action (booking confirmation, request status,
  review posted), it must give them an accessible, retrievable copy/record of what they agreed to or
  submitted (electronic records must remain accessible for later reference).
