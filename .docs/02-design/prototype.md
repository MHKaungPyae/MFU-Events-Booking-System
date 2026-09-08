# EventMFU — Prototype Reference

**Last updated:** 2026-09-08

---

## Working prototype

The interactive HTML prototype lives at `docs/eventmfu_demo.html` in the project root. It is a self-contained, single-file app that runs all flows in the browser with no backend.

**What it covers (fully clickable):**
- Standalone auth screens (signup, verify, login)
- User feed, event detail, booking, Q&A, review submission (including anonymous option)
- Main Organizer: Event Request form, team management (all three recruitment methods), check-in scanner (both modes), live attendance view
- Admin: request queue (with eligibility snapshot), venue registry, approve/request-changes/reject flow, direct event creation, flag queues, activity log
- My Record page with recognition titles and badges

**How to open it:**
```
open docs/eventmfu_demo.html
# or just double-click the file in Finder / your file manager
```

No server needed. All state is in-memory; refreshing resets it.

**When to use it:**
- As the behavioral reference for any ambiguous flow: "what should happen here?" → check the demo
- As the comparison baseline for end-to-end (Playwright) tests (`plan.md` T7)
- For user interviews and usability testing before the real backend is ready

---

## Key screen inventory

| Screen | Role | Notes |
|--------|------|-------|
| Signup / Verify / Login | User | University-email gate; verification link flow mocked |
| Event Feed | User | Filtered by school + year; shows organizer/contributor status instead of Book if applicable |
| Event Detail + Q&A | User | Booking, Q&A thread, see organizer's primary title |
| My Bookings + History | User | QR code, cancel, booking status |
| My Record | User | Full attendee/organizer/contributor history + badges |
| Staff Openings | User | Browse, claim (open call), apply (application method) |
| Event Request Form | User | Full v18 fields including agenda, equipment_needs, contact_phone, venue_preference |
| Request Status | User | Pending / needs_info (with admin feedback) / approved / rejected |
| Manage Event | Main Organizer | Edit, publish, team roster, staff call posting, live check-in view |
| Check-in Scanner | Organizer / Check-in Staff | Staff-scan QR scanner; self-scan fallback |
| Admin Request Queue | Admin | Eligibility snapshot per row; approve/request-changes/reject |
| Admin Venue Registry | Admin | Add/edit/retire venues |
| Admin Create Event | Admin | Direct creation with venue picker |
| Admin Flag Queues | Admin | Organizer flags + health flags with evidence view |
| Admin Activity Log | Admin | Full append-only audit trail |

---

## Design principles carried into the prototype

1. **No separate Creator account.** Any user who holds an organizer role on an event sees a "Manage" view for that event — the same session, same login.
2. **Computed recognition.** Titles and badges appear live; no separate "update my profile" step.
3. **Admin always decides.** No screen in the prototype auto-restricts or auto-rejects — flags and warnings are surfaced; actions require a click.
4. **Anonymous reviews indistinguishable to external readers.** The prototype redacts the author in every view, including Admin's flag evidence panel.

---

## Prototype limitations (known gaps vs. real backend)

- No persistent storage — state resets on reload
- Sentiment analysis returns a mocked result immediately (no async delay)
- Verification email is logged to the browser console, not sent
- No real QR code scanning — the scanner accepts any typed token for demo purposes
- University-domain check is hardcoded; production will read from PlatformSettings
