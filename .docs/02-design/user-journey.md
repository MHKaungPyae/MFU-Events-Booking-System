# EventMFU — User Journey Maps

**Source:** `docs/APP_FLOW.md` (v18 target), `docs/REQUIREMENTS_DESIGN.md` §11
**Last updated:** 2026-09-08

Five journeys cover the primary user paths. Each maps the emotional arc, the app steps, and the pain it resolves.

---

## Journey 1 — Student discovers and attends an event

**Persona:** Ploy, 2nd-year Business student. Wants to find activities on campus but doesn't know where to look.
**Pain resolved:** P1 (scattered discovery), P2 (friction in booking)

```
Phase           Steps                                       Emotion
─────────────   ─────────────────────────────────────────   ──────────────────────
Awareness       Hears about EventMFU from a classmate       Curious
Signup          Enters university email → receives           Slight friction
                verification link → activates account
                → picks school + year
Discovery       Sees personalized feed filtered to her       Relieved ("finally
                school + open events                        one place")
Evaluation      Opens an event → reads description          Interested
                → checks Q&A thread for answers
                → sees organizer's public record
Decision        Clicks Book → receives QR code              Excited
Attendance      On event day: scans venue QR (self-scan)    Satisfied
                or shows QR to staff (staff-scan)           
Post-event      Submits a review (optionally anonymous)     Empowered
                → earns health reward + attendance points   
Record          My Record updates with the attended event   Proud
```

**Key app moments:**
1. Feed first load — immediately filtered by school/year, no setup needed
2. Organizer record visible before booking — trust signal
3. Single-tap check-in — no queue, no DM to organizer
4. Review with anonymous option — honest feedback without fear

---

## Journey 2 — Student becomes a first-time event organizer

**Persona:** Ton, 3rd-year Engineering student. Wants to run a hackathon for his faculty. Has never organized through an official channel.
**Pain resolved:** P7 (transparent pathway), P6 (venue coordination)

```
Phase               Steps                                   Emotion
─────────────────   ─────────────────────────────────────   ──────────────────────
Submission          Fills out Event Request form:            Hopeful
                    title, agenda, venue preference,
                    equipment needs, contact number,
                    date/time, audience (3rd-year + Eng)
Waiting             Status shows "pending" in dashboard     Uncertain
Needs info          Admin sends back feedback:              Frustrated →
                    "No room free that Saturday,            Motivated
                    can you do Sunday?"
Resubmit            Updates date → resubmits               Engaged
Approval            Notification: approved + venue          Excited ("it's real")
                    assigned (Eng Building 301, confirmed)
Preparation         Edits event draft → adds team via       In control
                    direct-add + posts open call for
                    Check-in Staff
Publishing          Publishes event → appears in feed       Proud
Running the event   Monitors live check-in view            Focused
Post-event          My Record gains Organizer history       Recognized
                    Earns "First-Time Organizer" title
```

**Key app moments:**
1. Event Request includes agenda/equipment fields — Admin has enough info to approve without calling
2. needs_info loop — request not rejected over missing details, real dialogue happens
3. Venue assigned at approval — no separate venue-confirmation step after the fact
4. Open call for Check-in Staff — doesn't have to personally recruit everyone

---

## Journey 3 — Student joins an event team via Staff Openings

**Persona:** May, 1st-year student. Wants to get involved but doesn't know any organizers personally.
**Pain resolved:** P9 (can only join via personal network)

```
Phase           Steps                                       Emotion
─────────────   ─────────────────────────────────────────   ──────────────────────
Discovery       Browses Staff Openings tab                  Hopeful
Evaluation      Sees "3 Presenter slots needed for          Interested
                Innovation Fair" — open call
                Sees "AV Technician — Application"
                with custom questions
Choice          Claims a Presenter slot (open call)         Immediate satisfaction
                → Booking auto-cancelled if she had one     
Contribution    Event day: credited as "Presenter"          Proud
                on the event roster
Record          My Record gains Contributor history         Recognized
                Earns "Team Player" badge after 5 events
```

**Key app moments:**
1. Staff Openings visible to all Users — no inside knowledge needed
2. One-tap claim for open calls — same immediacy as booking a seat
3. Application method for AV role — organizer can filter by answers about AV experience
4. Booking auto-cancelled so attendee list stays clean

---

## Journey 4 — Admin manages a busy event day

**Persona:** Admin account. Three events running today across different venues.
**Pain resolved:** P6 (no visibility across simultaneous events), P4 (no-shows)

```
Phase               Steps                                   Notes
─────────────────   ─────────────────────────────────────   ──────────────────────
Morning             Checks platform summary:                Flags pending, venues
                    2 pending requests, 1 open flag,        assigned, system healthy
                    3 events today
Request review      Opens first pending request             Eligibility snapshot:
                    — eligibility snapshot shows:           health score 85,
                    requester health 85/100,                no prior flags
                    1 prior event, no flags
                    Admin approves + picks venue
                    (Auditorium, free on that date)
Second request      Requester is organizer_restricted       Cannot approve;
                    App blocks approval action              sends a note to requester
Organizer flag      Opens OrganizerFlag for review:         Reviews 3 sample
                    ratio 38% negative over 12 reviews      negative reviews
                    → decides to warn the organizer        Admin warns; no auto-action
Post-event sweep    Triggers no-show sweep (or              HealthTransactions fire
(end of day)        scheduler runs automatically)           for no-shows; UserHealthFlags
                                                            may be raised
Activity log        Reviews today's log for any             Audit trail complete
                    suspicious actions
```

**Key app moments:**
1. Summary dashboard — one view to see everything requiring attention
2. Eligibility snapshot inline — no separate lookup to evaluate a request
3. Organizer-restricted block — system prevents approval, Admin sees why
4. Admin always decides on flags — no automatic restrictions

---

## Journey 5 — Student checks their recognition record

**Persona:** Nong, 4th-year student. Active organizer and frequent attendee over two years.
**Pain resolved:** P5 (no sharable proof of contribution)

```
Phase           Steps                                       Emotion
─────────────   ─────────────────────────────────────────   ──────────────────────
Access          Opens My Record page                        Curious
Overview        Sees primary title:                         Surprised and pleased
                "Seasoned Organizer"
                + badges: Fan Favorite, Team Player
Attendee tab    12 events attended, 0 no-shows,             Proud
                11 reviews written
                Earns "Reliable Attendee" badge
Organizer tab   8 events organized, avg 4.7 rating,         Very proud
                avg 89% attendance rate
Contributor tab 3 Contributor positions (Presenter 2x,      Satisfied
                General Staff 1x)
Share           Copies record URL for a club                Empowered
                committee application
Clickability    A committee member clicks Nong's name        Transparency
                from an event roster → opens this record
```

**Key app moments:**
1. Primary title visible beside name everywhere — persistent recognition, no setup
2. Organizer history shows per-event stats — useful as portfolio evidence
3. Print-friendly page — direct use in job/club applications
4. Public record via clickable name — others can verify the record
