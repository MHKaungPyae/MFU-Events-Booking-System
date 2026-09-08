# Diagram 3 — Event Request Workflow

**Type:** Process / sequence diagram
**Source:** `docs/REQUIREMENTS_DESIGN.md` §3.3, §11; `docs/APP_FLOW.md` §3a

This is the primary path for a student to become an event organizer. It is also the main venue-coordination flow — Admin assigns the actual room.

---

## Flow diagram

```
Student (User app)                Backend                     Admin (Admin app)
        │                            │                               │
        │  POST /api/user/           │                               │
        │  event-requests            │                               │
        │  {title, description,      │                               │
        │   agenda,                  │                               │
        │   equipment_needs,         │                               │
        │   contact_phone,           │                               │
        │   venue_preference,        │                               │
        │   start_time, end_time,    │                               │
        │   capacity, audience_type, │                               │
        │   checkin_mode,            │                               │
        │   points_value}            │                               │
        │───────────────────────────►│                               │
        │                            │  EventRequest{status:pending} │
        │  status: pending           │                               │
        │◄───────────────────────────│                               │
        │                            │                               │
        │                            │  GET /api/admin/              │
        │                            │  event-requests?status=pending│
        │                            │◄──────────────────────────────│
        │                            │  per row: eligibility snapshot│
        │                            │  (organizer_restricted,       │
        │                            │   health_score,               │
        │                            │   prior organizing + flags)   │
        │                            │───────────────────────────────►
        │                            │                               │
        │                            │  GET /api/admin/              │
        │                            │  venues?matching=:requestId   │
        │                            │◄──────────────────────────────│
        │                            │  registry venues matching     │
        │                            │  venue_preference, each flagged│
        │                            │  free/conflict for start/end  │
        │                            │───────────────────────────────►
        │                            │                               │
        │                            │               ┌───────────────┴────────────────┐
        │                            │               │      Admin chooses an outcome   │
        │                            │               └──┬────────────────┬─────────────┘
        │                            │                  │                │
        │          ┌─────────────────┤ APPROVE          │ REQUEST CHANGES│ REJECT
        │          │                 │                  │                │
        │          │  POST /approve  │◄─────────────────┘                │
        │          │  {venue_id,     │                                   │
        │          │   start, end}   │                                   │
        │          │                 │  overlap check                    │
        │          │                 │  (409 if conflict; force:true     │
        │          │                 │   to proceed anyway)              │
        │          │                 │                                   │
        │          │      ┌──────────┤  ● Event{status:draft,           │
        │          │      │          │    venue_id: set,                 │
        │          │      │          │    start/end: confirmed}          │
        │          │      │          │  ● EventOrganizer{main_organizer, │
        │          │      │          │    joined_via:event_request_approval}
        │          │      │          │  ● EventRequest.status → approved │
        │          │      │          │                                   │
        │  notified│      │          │        POST /request-changes      │
        │  + event │      │          │        {feedback}◄────────────────┘
        │  created │      │          │                                   │ POST /reject
        │          │      │          │  EventRequest.status → needs_info │ {reason}
        │          │      │          │  admin_feedback: <set>            │◄────────────
        │          │      │          │                                   │
        │          │      │  notify  │                                   │
        │◄─────────┘      │◄─────────│  "revise and resubmit"            │
        │                 │          │                                   │
        │  POST /api/user/│          │                                   │
        │  event-requests/│          │                                   │
        │  :id/resubmit   │          │                                   │
        │  {...updates}   │          │                                   │
        │────────────────►│          │                                   │
        │                 │  EventRequest.revision_count += 1            │
        │                 │  status → pending  (re-enters queue above)   │
        │                 │          │                                   │
        │                 │          │                          notify   │
        │◄────────────────┘ notify   │  rejected ────────────────────────►
        │  approved: event│          │                                   │
        │  ready to publish          │                                   │
```

---

## State machine: EventRequest.status

```
                    submit
  [not created] ──────────────► [pending]
                                    │
                    ┌───────────────┼────────────────┐
                    │               │                │
               Admin rejects   Admin sends      Admin approves
                    │           for changes          │
                    ▼               │                ▼
               [rejected]           ▼           [approved]
                              [needs_info]    (→ Event created)
                                    │
                              requester
                              resubmits
                                    │
                                    ▼
                               [pending]  ──── (loops until Admin decides)
```

---

## Mermaid sequence diagram

```mermaid
sequenceDiagram
    participant U as Student
    participant B as Backend
    participant A as Admin

    U->>B: POST /event-requests {fields}
    B-->>U: EventRequest{status: pending}

    A->>B: GET /event-requests?status=pending
    B-->>A: list + eligibility snapshots

    A->>B: GET /venues?matching=:requestId
    B-->>A: free/conflict venue candidates

    alt Admin approves
        A->>B: POST /event-requests/:id/approve {venue_id, start, end}
        B->>B: overlap check (warn if conflict)
        B->>B: create Event, EventOrganizer{main_organizer}
        B-->>U: notify — event created, you are Main Organizer
    else Admin requests changes
        A->>B: POST /event-requests/:id/request-changes {feedback}
        B-->>U: notify — revise and resubmit
        U->>B: POST /event-requests/:id/resubmit {updates}
        B->>B: revision_count++, status → pending
        Note over U,A: Loop repeats until Admin approves or rejects
    else Admin rejects
        A->>B: POST /event-requests/:id/reject {reason}
        B-->>U: notify — rejected
    end
```
