# Diagram 2 — Data Model (Entity-Relationship)

**Type:** ER diagram
**Source:** `docs/REQUIREMENTS_DESIGN.md` §10, `docs/DATA_MODEL.md`

---

## Entity relationship overview

```
                              ┌──────────────────┐
                              │      User         │
                              │──────────────────-│
                              │ id (PK)           │
                              │ email (verified)  │
                              │ password_hash     │
                              │ name, school, year│
                              │ role: user|admin  │
                              │ health_score      │
                              │ booking_restricted│
                              │ organizer_restricted
                              │ status            │
                              └──────┬────────────┘
                                     │
          ┌──────────────────────────┼────────────────────────────────────┐
          │                          │                                    │
          │ submits                  │ creates (admin only)               │ logs actions
          ▼                          ▼                                    ▼
┌──────────────────┐      ┌───────────────────┐               ┌─────────────────┐
│  EventRequest    │      │      Venue         │               │  ActivityLog    │
│──────────────────│      │───────────────────│               │─────────────────│
│ id               │      │ id                │               │ id              │
│ requested_by(FK) │      │ name, building    │               │ actor_id (FK)   │
│ title, description│     │ capacity, notes   │               │ action_type     │
│ agenda           │ approves+assigns │ status: active|retired│ target_type/id  │
│ equipment_needs  ├─────►│                   │               │ metadata        │
│ contact_phone    │      └────────┬──────────┘               │ created_at      │
│ venue_preference │               │ assigned to              └─────────────────┘
│ start/end_time   │               │
│ capacity, audience│              ▼
│ checkin_mode     │      ┌───────────────────┐
│ points_value     │      │      Event        │
│ status:          ├─────►│───────────────────│
│  pending|         creates│ id               │
│  needs_info|     │      │ source_request_id │◄── null for Admin-created
│  approved|       │      │  (FK, nullable)   │
│  rejected        │      │ venue_id (FK)     │
│ admin_feedback   │      │ created_by (FK)   │
│ revision_count   │      │ title, description│
│ resulting_event_id│     │ start/end_time    │
└──────────────────┘      │ capacity, audience│
                          │ checkin_mode      │
                          │ checkin_qr_token  │
                          │ points_value      │
                          │ status: draft|    │
                          │  published|cancelled
                          └──────┬────────────┘
                                 │
          ┌──────────────────────┼────────────────────────────────────┐
          │ has many             │ has many                           │ has many
          ▼                      ▼                                    ▼
┌─────────────────┐   ┌──────────────────┐               ┌─────────────────────┐
│ EventOrganizer  │   │ EventContributor │               │     Booking          │
│─────────────────│   │──────────────────│               │─────────────────────│
│ id              │   │ id               │               │ id                  │
│ event_id (FK)   │   │ event_id (FK)    │               │ event_id (FK)       │
│ user_id (FK)    │   │ user_id (FK)     │               │ user_id (FK)        │
│ role:           │   │ position_title   │               │ status: booked|     │
│  main_organizer │   │ joined_via       │               │  cancelled|attended │
│  co_organizer   │   │ assigned_by (FK) │               │  |no_show           │
│  checkin_staff  │   │ assigned_at      │               │ qr_token            │
│ joined_via      │   └──────────────────┘               │ booked_at           │
│ assigned_by (FK)│                                      │ cancelled_at        │
│ assigned_at     │           ┌────────────────────────► │ checked_in_at       │
└─────────────────┘           │                          └─────────────────────┘
                              │ created from                        │ generates
          ┌───────────────────┤                                     ▼
          │                   │                          ┌─────────────────────┐
          ▼                   │                          │     Review          │
┌──────────────────┐          │                          │─────────────────────│
│   StaffCall      │          │                          │ id                  │
│──────────────────│          │                          │ event_id (FK)       │
│ id               │          │                          │ user_id (FK)        │
│ event_id (FK)    │          │                          │ is_anonymous        │
│ target_type:     │          │                          │ rating (1–5)        │
│  organizer_role  │          │                          │ comment             │
│  contributor_pos │          │                          │ sentiment_label     │
│ organizer_role   │          │                          │ sentiment_score     │
│ position_title   │          │                          │ sentiment_status    │
│ method:          │          │                          └──────────────┬──────┘
│  open_call       │          │                                         │ triggers
│  application     │          │                                         ▼
│ slots_total/filled│         │                          ┌─────────────────────┐
│ questions[]      │          │                          │  OrganizerFlag      │
│ status: open|closed         │                          │─────────────────────│
└───────┬──────────┘          │                          │ user_id (FK)        │
        │ has many            │                          │ negative_ratio      │
        ▼                     │                          │ review_count        │
┌──────────────────┐          │                          │ status/resolution   │
│ StaffApplication │──────────┘                          └─────────────────────┘
│──────────────────│
│ id               │
│ staff_call_id(FK)│    ┌──────────────────────────────────────────────────────┐
│ applicant_id(FK) │    │  Health & Points ledgers (per User)                  │
│ answers[]        │    │                                                      │
│ status: pending| │    │  HealthTransaction: user_id, booking_id, delta, reason│
│  accepted|rejected    │  UserHealthFlag: user_id, health_score_at_flag, status│
│ reviewed_by (FK) │    │  PointsTransaction: user_id, event_id, booking_id,   │
└──────────────────┘    │    points, sync_status                               │
                        │                                                      │
                        │  PlatformSettings: key/value (admin-configurable)    │
                        └──────────────────────────────────────────────────────┘
```

---

## Mermaid source

```mermaid
erDiagram
    User {
        string id PK
        string email
        string password_hash
        string name
        string school
        string year
        enum role "user|admin"
        number health_score
        boolean booking_restricted
        boolean organizer_restricted
        enum status
    }

    Venue {
        string id PK
        string name
        string building
        number capacity
        string notes
        enum status "active|retired"
        string created_by FK
    }

    EventRequest {
        string id PK
        string requested_by FK
        string title
        string agenda
        string equipment_needs
        string venue_preference
        datetime start_time
        datetime end_time
        enum status "pending|needs_info|approved|rejected"
        string admin_feedback
        number revision_count
        string resulting_event_id FK
    }

    Event {
        string id PK
        string source_request_id FK
        string venue_id FK
        string created_by FK
        string title
        enum status "draft|published|cancelled"
        enum checkin_mode "staff_scan|self_scan"
        string checkin_qr_token
        number points_value
    }

    EventOrganizer {
        string id PK
        string event_id FK
        string user_id FK
        enum role "main_organizer|co_organizer|checkin_staff"
        enum joined_via
    }

    EventContributor {
        string id PK
        string event_id FK
        string user_id FK
        string position_title
        enum joined_via
    }

    Booking {
        string id PK
        string event_id FK
        string user_id FK
        enum status "booked|cancelled|attended|no_show"
        string qr_token
        datetime checked_in_at
    }

    Review {
        string id PK
        string event_id FK
        string user_id FK
        boolean is_anonymous
        number rating
        string comment
        enum sentiment_label
        number sentiment_score
    }

    StaffCall {
        string id PK
        string event_id FK
        enum target_type "organizer_role|contributor_position"
        enum method "open_call|application"
        json questions
        enum status "open|closed"
    }

    StaffApplication {
        string id PK
        string staff_call_id FK
        string applicant_id FK
        json answers
        enum status "pending|accepted|rejected"
    }

    OrganizerFlag {
        string id PK
        string user_id FK
        number negative_ratio
        number review_count
        enum status "open|dismissed|actioned"
    }

    User ||--o{ EventRequest : "submits"
    User ||--o{ EventOrganizer : "holds role on"
    User ||--o{ EventContributor : "holds position on"
    User ||--o{ Booking : "makes"
    User ||--o{ Review : "writes"
    User ||--o{ OrganizerFlag : "flagged by"
    EventRequest ||--o| Event : "produces"
    Venue ||--o{ Event : "hosts"
    Event ||--o{ EventOrganizer : "has team"
    Event ||--o{ EventContributor : "has contributors"
    Event ||--o{ Booking : "has bookings"
    Event ||--o{ Review : "has reviews"
    Event ||--o{ StaffCall : "posts calls"
    StaffCall ||--o{ StaffApplication : "receives"
```
