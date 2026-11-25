# AMS Training and Sessions Module: CLI Project Context

**Version:** 1.2 (Coach can create batches and programmes; Batch or Individual option) | **Prepared:** 24 November 2026 | **Owner (PM):** Shrey Prakash | **Status:** draft for PM review, then hand to the engineering CLI
**Purpose:** a self-contained brief for the **Training and Sessions** module (programmes, batches, sessions, scheduling, attendance), built to plug into the colleague's **Onboarding and RBAC** module and the **Medical and Injury** module. It reuses the vocabulary and rules of the current AMS (V1 and V2 PRDs) wherever they still apply.

---

## 0. How to use this document (rules for the CLI)
1. Read the whole file. Then inspect the repository (framework, conventions, onboarding, roles, persons, organisations, audit, notifications, health module). **Do not assume a greenfield repo. Do not replace architecture.**
2. Build **one slice at a time** (section 18). Do not build later slices early.
3. Never invent a field, rule, role or integration. If something is unclear or conflicts with onboarding or health, **stop and ask**.
4. Labels: **[LOCKED]** decided (from the current AMS PRDs, onboarding draft or PM). **[PROPOSED]** my recommendation; build as a **setting** where possible and report it as unconfirmed. **[DECISION NEEDED]** open; build the default stated and flag it.
5. Mark what you build as `Implemented`, `Mocked`, `Planned`, `Blocked`, `Needs Decision`. Never claim a test passed that you did not run.
6. Use only synthetic data labelled "Demo data".
7. After each slice report: what was built, journeys supported, what is mocked, edge cases handled, what remains, assumptions to confirm.
8. Health data stays in the health module. This module may show **status and permitted activity** only, never diagnosis or notes.

---

## 1. Product context
- **AMS Standalone** for **High Performance Centres**: many sports, all age groups, both genders; minors are core. Only `external_id` on Person and Organization is ATOM-specific [LOCKED, onboarding].
- **Organisation** is the tenant. Optional flat **Centres** inside it (no tree) [LOCKED].
- **Fixed roles**, one **active role** at a time, each with its own dashboard [LOCKED]. Roles used here: **Admin, Coach, Team Manager, Athlete, Guardian**, plus read access for **Medic** and **Physiotherapist**. Phase 2 roles (S&C Coach, Biomechanist, Performance Analyst, Nutritionist, Psychologist) are not assignable yet.
- **Login:** email only. Org flat visibility: every staff role can read all athletes in the organisation (basic profile); acting is limited to scope [LOCKED, onboarding decision 10 with matrix note].

### 1.1 Vocabulary (kept from the current AMS)
| Term | Meaning |
|---|---|
| **Programme** (Training Programme) | A plan of training for a period: for example "Senior Athletics 2026-27" or "Pre-season camp" |
| **Batch** (also called Roster or Team in the UI) | A group of athletes who train together (sport, age group, gender category) |
| **Session** | One scheduled training event: date, time, venue, batch, coach(es) |
| **Attendance** | Who was present at a session; written only through the Attendance service |
| **Venue** | A place where sessions happen |
| **Batch programme** | A programme whose sessions are for one or more Batches (a group) [LOCKED, PM] |
| **Individual programme** | A programme for **one athlete**; the system keeps a hidden Batch of one behind it [LOCKED, PM] |
The hierarchy is **Organization > (Centre) > Programme > Batch > Session**. The V1 term "Project" is not used in the standalone product; its role is played by the Organization.
V1 terms map as: Supervisor (V1 attendance marker) → **Coach**.

### 1.2 What this module is not
Not a performance or assessment engine (that is AMS V2 and Phase 2: sessions are the **hosts** for assessments later). Not a training-load or periodisation tool (Phase 2). Not a finance, inventory or hostel module. Not a health module.

---

## 2. Decisions and starting points

| # | Decision | Source |
|---|---|---|
| T1 | Hierarchy: Programme > Batch > Session; Batches group athletes | AMS V1 [LOCKED] |
| T2 | Attendance is **one service**, called through entry points (sessions now; staff and others later) | Attendance module decision [LOCKED] |
| T3 | Venue clash check exists and prevents overlap | AMS V1 [LOCKED] |
| T4 | A session uses a **roster snapshot** of the Batch as expected athletes | AMS V1 FR-RB-32 [LOCKED] |
| T5 | V1 lock is date based: a past-dated session is immutable, a today-dated session is editable all day | AMS V1 [LOCKED]; see section 10 for corrections |
| T6 | Corrections add a new row; nothing is overwritten | AMS V2 decision [LOCKED] |
| T7 | Unfit athletes are shown as **"Unfit"**, not counted Absent | Health module [LOCKED] |
| T8 | Athletes see **no rank or comparison** | AMS V1 [LOCKED] |
| T9 | Minors have full access to their own dashboard; guardian is read-mostly | Onboarding [LOCKED] |
| T10 | Roles are fixed; no custom roles; a user works in one role at a time | Onboarding [LOCKED] |
| T11 | Same email can belong to two organisations; data is per organisation | Onboarding [LOCKED] |
| T12 | Departed coach's batches and sessions are owned by Admin until reassigned | Onboarding O7 [LOCKED] |
| T13 | **A Coach can create programmes** and owns them; at creation the Coach chooses the **scope: Batch or Individual** [LOCKED, PM, later 24 Nov]. **A Coach can also create batches** and owns them [LOCKED, PM, later 24 Nov], within the guardrails in Flow T1c. Admin creates venues and may create or edit any batch or programme. Coach creates and edits sessions in their own programmes and own or assigned batches. Team Manager handles logistics and membership for assigned teams | PM; onboarding matrix (Coach "Own programs") |
| T14 | Attendance lock window default 24 hours after the session ends; later changes need a correction approved by Admin | **[PROPOSED]** (V2 open point, "24-hour soft lock") |
| T15 | Team Manager becomes assignable when this module's batch and logistics features ship | **[PROPOSED]** (onboarding said "with Squads/Selection") |
| T16 | **Individual programme** (one athlete, hidden Batch of one) is allowed, chosen as the "Individual" scope when a programme is created | [LOCKED, PM] (resolves the V2 open point) |

---

## 3. Alignment with other modules

### 3.1 Needs from onboarding
Organization id from the server session; Person (name, gender, DOB, photo, sport, centre, status, `external_id`); minor flag from the organisation's `minor_age`; guardian links; role assignments and the **active role**; Centre scope; audit service; notification service (in-app now, email via provider); the `team/batch` column in the athlete import (the Batch must exist, else the athlete is added without a Batch and a warning is shown).
**First-run checklist alignment:** onboarding's checklist step "create first programme" is this module's first slice.

### 3.2 Needs from the health module
`availability_status(athlete)` (Fit, Restricted, Unfit), permitted activity text, review date, **"Concern reported" marker** (Coach and Medic only), and the ability to open the health report form **pre-filled with the session** (athlete, session, time, context).
This module never reads clinical notes.

### 3.3 Provides to others
- Sessions, rosters and attendance to dashboards and reports.
- Events: `session.published`, `session.changed`, `session.cancelled`, `session.completed`, `attendance.submitted`, `attendance.locked`, `attendance.corrected`, `batch.membership_changed`.
- **Hosting** for assessments (V2: "assessment chips" on sessions; roster snapshot reused as expected athletes; results entry exempt from the attendance lock) [Phase 2 hook].
- Schedule data for competition clash checks (later).

### 3.4 Points to confirm with the onboarding colleague
| # | Point | Assumption |
|---|---|---|
| C1 | Matrix says Coach "Own programs" | **Confirmed by PM:** Coach creates programmes (Batch or Individual), **batches**, and sessions in them; onboarding's athlete import `team/batch` column then matches batches created by Admin or Coaches |
| C2 | Team Manager's matrix cell "Logistics" | Venue, time and travel logistics for assigned teams; manages batch membership |
| C3 | Flat visibility of athletes | Coaches can read other batches' schedules and athletes; edit only their own |
| C4 | Athlete import "team/batch" | Must match an existing Batch name or code |
| C5 | Role availability | Coach, Athlete, Guardian, Admin assignable first; Team Manager with this module |
| C6 | Departed coach | Batches and future sessions become "owned by Admin" with a reassign banner |

---

## 4. Roles in this module

| Role | Does |
|---|---|
| **Admin** | Manages venues, coach assignment, batch membership; can create and edit **any** batch and programme; sets the age-group and gender lists; approves coach-created batches if that setting is on; approves attendance corrections; sets lock, reminder and alert settings; sees all sessions and reports |
| **Coach** | **Creates batches and programmes (Batch or Individual scope) and owns them**; manages membership and assistant coaches of own batches; creates and edits sessions in own programmes and assigned batches; shares the plan; marks attendance; closes sessions; records delivered-vs-planned and observations; requests corrections; sees roster with health status; reports or declares health status (per the health module) |
| **Team Manager** | Handles logistics for assigned teams: venues, timing, rosters, travel readiness; manages batch membership; reads attendance |
| **Medic, Physiotherapist** | Read schedules and rosters to plan care; see permitted activity; no editing here |
| **Athlete** | Sees own schedule, details, reminders; gives advance absence notice; sees own attendance; no rank |
| **Guardian** | Sees the child's schedule and attendance; gives absence notice for a minor |
| **Master Admin** | No access |

---

## 5. Entities (conceptual; adapt to repo conventions)
Every table: `id`, `organization_id`, `created_at`, `created_by`, `created_in_role`, `updated_at`, `status`. No hard deletes.

| Entity | Key fields |
|---|---|
| `venue` | name, code, centre_id (optional), type, capacity, shared_use (yes/no), open_hours, notes, status |
| `programme` | name, sport, **scope (Batch or Individual)**, type (Regular, Camp, Fitness testing, Other), centre_id (optional), start, end, objective, phases[] (optional name, from, to, note), owner_person_id (the creating Coach or Admin), co_coaches[], batch_ids[] (Batch scope) or athlete_person_id (Individual scope), hidden_batch_id (Individual scope), status |
| `batch` | name, code, sport, age_group (from Admin-owned list), gender_category (from list), capacity, kind (Group or Individual), auto_created (true for Individual), owner_person_id, created_by_role, approval_state (Not required, Pending, Approved, Rejected), centre_id, status. Individual batches are hidden from batch lists and have exactly one member |
| `batch_membership` | batch_id, athlete_person_id, from, to, reason, added_by (history kept; never overwritten) |
| `batch_coach` | batch_id, coach_person_id, role (Primary, Assistant), from, to |
| `session` | session_code, programme_id, batch_id, date, start, end, venue_id, type (Training, Conditioning, Recovery, Camp, Test day, Other), objective, equipment_note, plan_blocks[], plan_visible_to_athletes (default off), mandatory (default yes), series_id (nullable), rescheduled_from, status, cancel_reason_code, cancel_note, created_by, owner |
| `session_coach` | session_id, coach_person_id, role |
| `session_roster_snapshot` | session_id, athlete_person_id, taken_at, batch_membership_id, health_status_at_snapshot (Fit/Restricted/Unfit), source (batch, guest, added late) |
| `attendance_record` | session_id, athlete_person_id, status, reason_code, minutes_late, marked_by, marked_in_role, marked_at, health_status_at_session, permitted_activity_shown, locked, supersedes_id |
| `attendance_correction` | record_id, requested_status, reason, requested_by, state (Requested, Approved, Rejected), decided_by, decided_at |
| `absence_notice` | session_id or date range, athlete_person_id, given_by (athlete or guardian), reason_code, note, state (Open, Acknowledged), acknowledged_by |
| `session_close` | session_id, delivered (Full, Partial, Not delivered), actual_start, actual_end, summary, closed_by, closed_at |
| `session_note` | session_id, athlete_person_id (nullable), body, author, class `normal` (no medical content) |
| `session_template` | name, sport, type, plan_blocks[], default_duration |
| `series` | pattern (weekly days, interval), start, end, exceptions[], created_by |
| `calendar_closure` | date range, reason, centre_id (nullable) (holidays, shutdowns) |
| Settings | lock hours, snapshot minutes, reminder times, alert thresholds, attendance statuses and reason-code lists, clash policy |
| Lists | `attendance_status`, `absence_reason`, `cancel_reason`, `session_type`, `plan_block_type` (organisation owned) |

Plan block fields: `type` (Warm-up, Main, Conditioning, Skills, Cool-down, Other), `title`, `duration_min`, `notes`, optional `intensity_tag` (Low, Medium, High; descriptive, not a clinical measure).

---

## 6. Core flows

### Flow T1: Set up (Admin)
| # | Step | Detail |
|---|---|---|
| 1 | Add venues | Name, type, centre, capacity, shared use, open hours |
| 2 | Add a programme | Admin or Coach; **Batch or Individual option** (Flow T1b) |
| 3 | Add batches | Admin or Coach (Flow T1c): sport, age group, gender category, capacity |
| 4 | Assign coaches | Primary and assistant coaches per batch with dates |
| 5 | Add athletes | From athlete import (`team/batch`) or manual; the owning Coach, the Team Manager and the Admin can manage membership; history kept |
| 6 | Optional | Closures (holidays), session templates, attendance reason-code lists |
| 7 | Checklist | System shows programme, batches, coaches and venues complete |
Rules: sport must exist in the organisation's sports list; capacity is a warning at 100 percent and a block beyond a configurable ceiling; an athlete can be in several batches; one **primary batch per sport** is flagged (reporting uses primary).

### Flow T1b: Create a programme (Coach or Admin), with the Batch or Individual option [LOCKED, PM]
| # | Step | Detail |
|---|---|---|
| 1 | Start "New programme" | Available to Coach (own programmes) and Admin |
| 2 | **Choose scope** | A switch at the top: **Batch** (a group) or **Individual** (one athlete) |
| 3a | Batch scope | Pick one or more **Batches**. A Coach sees only batches they are assigned to; an Admin sees all. Sport comes from the batch |
| 3b | Individual scope | Pick **one athlete**. A Coach sees athletes in batches they are assigned to; an Admin sees any athlete. Sport defaults to the athlete's primary sport and can be changed |
| 4 | Fill details | Name, type (Regular, Camp, Fitness testing, Other), start and end dates, objective, optional phases, centre (optional) |
| 5 | Co-coaches (optional) | Add assistant coaches; they can add sessions but not edit the programme |
| 6 | Save | Draft, or Activate once at least one session or a series exists |
| 7 | System actions | Owner is set to the creator. For **Individual** scope the system **creates a hidden Batch of one** with membership dates equal to the programme dates; the athlete and their guardian (minor) see it in their schedule as "Individual training" |
**Rules**
- **Batch scope:** sessions are created per batch; one programme can cover several batches (for example a camp).
- **Individual scope:** the athlete cannot be changed once sessions exist; to switch athlete, complete the programme and create a new one. The athlete's Fit, Restricted or Unfit status chip, the "Concern reported" marker and the normal session checks (venue, coach and athlete clash) all apply.
- **Who edits:** the owner and the Admin edit the programme; co-coaches add and edit sessions; Team Manager reads. An Admin may take over or reassign ownership.
- **A Coach cannot create a programme for an athlete or batch outside their assignment.** For a specialist session with another coach's athlete, the Admin creates it or adds the Coach as a co-coach [PROPOSED].
- **Departed coach:** programmes owned by that coach become "Owned by Admin: reassign".
- **Athlete leaves or is deactivated:** an Individual programme is completed with a reason; history is kept.
- **Reports:** Individual programmes appear under their own scope filter and count toward coach workload; they are excluded from batch capacity.

### Flow T1c: Create a batch (Coach or Admin) [LOCKED, PM]
**Approach chosen: free creation with guardrails** (rather than Admin-only, or approval for every batch).
| Option | Good | Bad |
|---|---|---|
| A. Coach creates freely, no limits | Fast; no Admin bottleneck | Messy names, duplicate batches, athletes overloaded, reports unreliable |
| B. Every coach batch needs Admin approval | Clean data and control | Slow; defeats the purpose; Admin bottleneck |
| **C. Free creation with guardrails (chosen)** | Fast and clean; Admin can switch on approval if needed | Slightly more rules |
**Steps**
| # | Step | Detail |
|---|---|---|
| 1 | Start "New batch" | Coach or Admin |
| 2 | Core fields | Sport (organisation list), **age group and gender category from Admin-owned lists**, capacity, centre (optional), name (auto-suggested "{Sport} {Age group} {Gender} {Centre}"; editable), code |
| 3 | Add athletes | Search all Active athletes in the organisation (flat read); filters for sport, age group, gender, centre; each row shows the athlete's **status chip** and **how many active batches** they are already in |
| 4 | Eligibility warnings | Age outside the age group or gender mismatch shows a warning; adding anyway needs a short reason |
| 5 | Assistant coaches (optional) | Add other coaches; they are notified and can add sessions |
| 6 | Save | Batch becomes **Active** (or **Pending** if "Coach batch approval" is on) |
| 7 | System actions | Owner is the creator and becomes **Primary coach**; athletes and, for minors, guardians are notified of membership (setting); audit written; Admin sees the batch under "Recently created by coaches" |
**Guardrails**
- Required: sport, age group, gender category, capacity. **Coaches cannot add new age-group or gender values; only Admin edits those lists** (keeps reporting standard).
- **Duplicate name** inside a centre shows a warning.
- **Overload warning** when an athlete is already in the configured number of active batches (starter: warn at 3, no hard limit).
- **Primary batch per sport** is set only by Admin or Team Manager, so a new coach batch never takes an athlete's primary slot automatically.
- Only **Active** athletes can be added; people in Invited, Awaiting Guardian Consent or Deactivated status cannot.
- **Owner rights:** the owner edits the batch and its membership and adds assistants; assistants add sessions only; Admin and Team Manager (assigned teams) can manage membership; others read.
- **Deactivate:** the owner can deactivate only when no future sessions exist; otherwise move or cancel them. Admin can deactivate any batch after handling sessions.
- **Departed coach:** batch becomes "Owned by Admin: reassign".
- **Approval setting (off by default):** when on, a Coach's new batch is Pending; the Coach can prepare programmes and draft sessions but cannot publish sessions until the Admin approves; rejection gives a reason.
- **Individual batches** (hidden Batch of one) are separate, created only through Flow T1b, and never go through this flow.

### Flow T2: Plan sessions (Coach, Admin, Team Manager)
1. Choose **single session**, **series** (for example Mon, Wed, Fri until a date) or **from template**.
2. Pick programme and batch, date, start and end, venue, session type, coaches, objective.
3. Optional **plan blocks** and equipment note; choose whether athletes can see blocks.
4. **Checks (on save):**
   - Venue clash: blocked unless the venue allows shared use [T3].
   - Coach clash: blocked.
   - Athlete clash (two sessions at once): warning with the conflicting session.
   - Closure or holiday: warning or block per setting.
   - Venue open hours: warning.
   - Health summary (non-clinical): count of Fit, Restricted, Unfit in the batch for that date.
5. Save as Draft or **Publish**. Publishing notifies coaches, athletes and, for minors, guardians; edits to published sessions notify again with a change summary.
6. Series edit: "this session", "this and following", or "all"; exceptions are stored.
Rule: sessions in the past can only be created by Admin with a reason (flagged late entry).

### Flow T3: Before the session
- Reminders to athletes (and guardians of minors) at configured times (default 24 hours and 2 hours).
- **Advance absence notice:** athlete or guardian picks the session or a date range, a reason chip (School or exam, Family, Travel, Personal, **Unwell**, Other) and a note. Choosing **Unwell** offers "Tell my coach about a health concern" (opens the health form; the absence notice itself carries no medical detail).
- Coach and Admin see notices on the roster as "Absence notice" and may set **Excused** at attendance time.
- **Roster snapshot:** taken when attendance opens (default 30 minutes before start; setting). It copies the batch's active members, their health status chip, and any guests. Athletes who join the batch afterwards are not on that session's snapshot unless added late.

### Flow T4: Session day (Coach)
| # | Step | Detail |
|---|---|---|
| 1 | Open "Today" | List of the coach's sessions with batch, venue, time, expected count and a health summary (Fit, Restricted, Unfit counts) |
| 2 | Open session | Roster with photo, name, **status chip**, permitted activity (Restricted), "Concern reported" marker, absence notice |
| 3 | Start session (optional) | Status In progress; actual start time recorded |
| 4 | Mark attendance | Present, Late (minutes), Absent, Excused, plus bulk "mark all present". **Unfit athletes default to "Unfit, not expected"**; the coach may mark "Present, observing or rehab" with a reason. Restricted athletes show their permitted activity |
| 5 | Add late or guest athlete | Search the organisation; guest needs a reason |
| 6 | Health during session | One tap opens the health form pre-filled with this session; the Coach may declare Unfit or Restricted per the health module |
| 7 | Observations | Per-athlete short note (normal class; a prompt reminds "Use the health report for injuries") and a session summary |
| 8 | Submit attendance | Records become Submitted; edits allowed until the lock |
| 9 | Close session | Delivered (Full, Partial, Not delivered), actual end, summary; status Completed |
Rules: attendance is written only through the Attendance service; a session cannot be Completed until attendance is Submitted or "no athletes attended" is recorded with a reason; coaches see only their own sessions to edit.

### Flow T5: After the session, lock and corrections
- Attendance editable by the marking coach until **lock time = session end + lock hours (default 24)** [T14].
- After the lock, changes use a **correction request**: coach states requested status and reason; **Admin approves or rejects**. Approval adds a new linked record; the original stays.
- **Late entry:** Admin may open attendance for a session with no attendance with a reason.
- Today-dated sessions follow V1: editable the same day; past-dated immutable except attendance corrections and result entry [Phase 2 assessments exempt, per V2 AS-5 open point].

### Flow T6: Cancel and reschedule
1. Coach (own sessions) or Admin cancels: reason code (Weather, Venue unavailable, Coach unavailable, Holiday, Low numbers, Other) and note.
2. Notifications to coaches, athletes, guardians (minors).
3. **Reschedule** creates a new session linked to the old one; athletes are re-snapshotted.
4. **Bulk cancel** for a closure date range or a venue closure; each session shows its cancel reason.
5. Cancelled sessions stay visible with status and are excluded from "planned" counts.
6. A venue deactivated with future sessions is blocked until sessions are moved or cancelled.

### Flow T7: Absence and attendance follow-up
- If attendance is not submitted within X hours of session end (default 2), the coach and Admin are reminded.
- **Low attendance alert** per athlete: configurable (starter: 3 consecutive unexcused absences, or attendance rate below a set percent over 30 days). Alerts go to the Coach and Admin; for minors a **guardian notice** is a setting, default **off** [PROPOSED].
- Alerts never mention health; Unfit and Excused do not count as unexcused absence.

### Flow T8: Programme lifecycle
`Draft → Active → Completed → Archived`. Activating requires at least one batch, one coach and one session or a series. Completing a programme keeps all sessions and attendance. **Individual programme:** created by a Coach or Admin through the Individual scope in Flow T1b; sessions work as usual [T16].
**Camps and special blocks:** a programme of type Camp with a date range; batch can be formed from selected athletes (selection engine is a later module).

### Flow T9: Departures and handovers
- **Coach deactivated:** batches, programmes and future sessions show "Owned by Admin: reassign". Admin reassigns; past records keep the original coach.
- **Athlete leaves a batch:** membership gets an end date; past attendance remains; future snapshots exclude them.
- **Athlete deactivated:** removed from future snapshots; history kept.
- **Batch merged or split:** Admin creates new batch and moves memberships with dates; history kept.

### Flow T10: Reporting and dashboards
See section 11.

---

## 7. Persona journeys (onboarding to end of journey)

### Admin
Accept invite (OTP, password) → first-run checklist → import athletes and staff → create venues → create programme → create batches and assign coaches → membership via import or manual → set settings (lock hours, reminders, alert thresholds, closures) → publish sessions or let coaches do so → monitor dashboard (sessions today, attendance not submitted, correction requests) → approve corrections → reports → reassign when coaches leave → archive programmes at season end.

### Coach
Accept invite → Coach dashboard: my batches, my programmes and today's sessions → **create a batch if needed** (sport, age group, gender, capacity, add athletes and assistants) → **create a programme: choose Batch or Individual** (pick batch or athlete) → create sessions (single, series, template) with checks → publish → before session: see absence notices and health summary → session day: roster with status chips and markers, mark attendance, add guests, start session → raise health report/declare per health rules → close session with delivered and summary → fix mistakes inside the lock window or request a correction → review attendance trend per athlete → end of season: hand over batches; deactivated → Admin owns.

### Team Manager
Accept invite → my teams → membership management (add and end memberships) → logistics: venues, timings, travel readiness (Fit, Restricted, Unfit list from health) → read attendance → handle bulk cancel for closures → cannot edit health or clinical data.

### Athlete
Invite (minors: guardian consents first) → dashboard (Fit) → **Schedule** (calendar and list; session details: venue, time, objective, equipment) → reminders → **absence notice** if cannot attend (Unwell offers the health form) → session day: see "Today" → after: own attendance history → sees no rank or comparison → leaving: membership ends, history stays.

### Guardian
Invite and consent → child's schedule and attendance → reminders for the child's sessions → give absence notice for the child → neutral alerts on cancellations and (setting) low attendance → access ends at 18 unless re-granted.

### Medic and Physiotherapist
See schedules and rosters to plan treatment; see permitted activity; cannot edit sessions or attendance.

### Master Admin
No role in this module.

---

## 8. Status models
| Object | States |
|---|---|
| Programme | Draft → Active → Completed → Archived |
| Batch | Active ↔ Inactive |
| Session | Draft → Published → In progress → Completed; Cancelled (any time before Completed); Rescheduled is a link, not a state |
| Attendance record | Draft → Submitted → Locked; Corrected = new linked record |
| Correction | Requested → Approved or Rejected |
| Absence notice | Open → Acknowledged |
| Series | Active → Ended |
Transitions only through defined actions with actor, time, reason; audited.

---

## 9. Settings (organisation level, edit by Admin)
| Setting | Starter |
|---|---|
| Attendance lock hours after session end | 24 |
| Roster snapshot minutes before start | 30 |
| Reminder times (athletes and guardians) | 24 h and 2 h before |
| Attendance-not-submitted reminder | 2 h after end |
| Attendance statuses | Present, Late, Absent, Excused (Unfit is system-derived) |
| Capacity warning and block | warn 100 percent; block above 110 percent |
| Low attendance alert | 3 consecutive unexcused, or below 75 percent over 30 days |
| Guardian notice on low attendance | Off |
| Closures: block or warn | Warn |
| Shared-use venues allowed | Per venue |
| Reason-code lists | Starter lists, organisation edits |
| Metric definition for attendance rate | Present + Late ÷ (rostered − Unfit not expected − Excused) |
| Coach batch approval required | Off |
| Warn when an athlete is in this many active batches | 3 |
| Notify athlete and guardian (minor) on batch add or remove | On |
| Age-group and gender category lists | Admin-owned starter lists |

---

## 10. Rules (numbered for tests)
| # | Rule |
|---|---|
| S-1 | **Coach and Admin can create batches and programmes** (Batch or Individual scope). Coach edits only their own batches and programmes and sessions in them or in assigned batches. Team Manager manages membership and logistics for assigned teams. [T13] |
| S-2 | A session needs programme, batch, date, start, end, venue and at least one coach. |
| S-3 | **Venue clash** is blocked unless the venue is shared-use. **Coach clash** is blocked. **Athlete clash** is a warning. |
| S-4 | A session cannot end before it starts; maximum duration is a setting. |
| S-5 | Roster snapshot is taken when attendance opens and is frozen at Submit. Athletes added later appear only as "added late". |
| S-6 | Attendance is written only through the Attendance service. No module copies attendance logic. |
| S-7 | An athlete with status **Unfit** is shown as "Unfit, not expected" and is not counted Absent unless the coach overrides with a reason. |
| S-8 | A Restricted athlete shows permitted activity on the roster; the coach cannot see why. |
| S-9 | The "Concern reported" marker is visible to the Coach and Medic only. |
| S-10 | A session cannot be Completed until attendance is Submitted or "no attendance" is recorded with a reason. |
| S-11 | Attendance is editable until session end + lock hours; afterwards changes need an Admin-approved correction. |
| S-12 | A correction adds a new linked record; the original stays. |
| S-13 | Past-dated sessions cannot be edited except attendance corrections (and, later, assessment results). Today-dated sessions can be edited until they start. |
| S-14 | Cancelling needs a reason code. Rescheduling creates a linked session. |
| S-15 | Publishing and changes notify the right people; changes include a summary. |
| S-16 | A venue or coach cannot be deactivated while future sessions use it without moving or cancelling them. |
| S-17 | Batch membership is history: ends get an end date; nothing is edited in place. |
| S-18 | An athlete can be in several batches; primary batch per sport is used for reports. |
| S-19 | Coach notes about athletes contain no medical content; injuries go through the health report. |
| S-20 | Athletes and guardians see their own schedule and attendance only; no rank, no comparison. |
| S-21 | Absence notices carry no medical detail. "Unwell" only prompts the health form. |
| S-22 | Low-attendance alerts exclude Unfit and Excused absences and never mention health. |
| S-23 | Aggregate reports hide groups smaller than five if they split by health status. |
| S-24 | A departed coach's batches and future sessions are owned by Admin until reassigned. |
| S-25 | Plan blocks are visible to athletes only if the coach turns that on; objective and equipment are visible by default. |
| S-26 | Every create, change, cancel, attendance mark and correction is audited with the **active role**. |
| S-27 | Server derives organisation and role from the session; client input is never trusted. |
| S-28 | Late creation of past sessions needs Admin and a reason. |
| S-29 | An Individual programme is a Programme with Individual scope: exactly one athlete and a hidden Batch of one created by the system. |
| S-30 | Time is stored with time zone; shown in the organisation's time zone (default Indian Standard Time). |
| S-31 | Every programme has a scope, Batch or Individual, chosen at creation. A Batch-scope programme needs at least one batch; an Individual-scope programme needs exactly one athlete. |
| S-32 | A Coach can pick only batches they are assigned to (Batch scope) or athletes in those batches (Individual scope). |
| S-33 | The scope of a programme cannot change once it has sessions. |
| S-34 | The athlete of an Individual programme cannot change once sessions exist. |
| S-35 | Hidden Individual batches are never shown in batch lists or counted for capacity, and have exactly one member. |
| S-36 | The owner of a programme is its creator until reassigned; co-coaches can add sessions but not edit the programme. |
| S-37 | An Individual programme is visible to the athlete, the guardian (minor), the owner, co-coaches, Admin, Team Manager (logistics read), Medic and Physio (read). Other athletes never see it. |
| S-38 | A Coach can create a batch; the creator becomes owner and Primary coach automatically. |
| S-39 | A batch requires sport, age group, gender category and capacity. Age group and gender values come from Admin-owned lists; a Coach cannot add values. |
| S-40 | A Coach may add any **Active** athlete in the organisation (flat read). Age or gender mismatch shows a warning; adding anyway needs a reason. |
| S-41 | Athletes, and guardians of minors, are notified when added to or removed from a batch (setting, default on). |
| S-42 | A Coach manages membership only in their own batches; Team Manager in assigned teams; Admin in any. |
| S-43 | If "Coach batch approval" is on, a Coach-created batch is Pending until the Admin approves; sessions in it cannot be published meanwhile. |
| S-44 | Only Admin or Team Manager can set an athlete's primary batch for a sport. |
| S-45 | A batch cannot be deactivated while future sessions exist; the owner must move or cancel them first. |
| S-46 | A duplicate batch name within the same centre shows a warning. |
| S-47 | When an athlete would exceed the configured number of active batches, a warning is shown (no hard limit by default). |
| S-48 | Admin can see "Recently created by coaches" and who created each batch; every create, edit, add and remove is audited with the active role. |
| S-49 | Athletes in Invited, Awaiting Guardian Consent or Deactivated status cannot be added to a batch. |

---

## 11. Permissions (starter matrix)
Key: **F** full, **C** create, **E** edit, **R** read, **S** scoped (assigned batches or teams), **O** own, **Ch** child, **A** approve, — none.
| Resource | Admin | Coach | Team Mgr | Medic | Physio | Athlete | Guardian |
|---|---|---|---|---|---|---|---|
| Venues | F | R | R | R | R | R (name) | R (name) |
| Programmes (create, edit) | F (any) | **C, E (own); co-coach R** | R | R | R | R (own, Individual included) | R (child) |
| Batches (create, edit) | F (any) | **C; E own**; edit sessions in assigned | S membership | R | R | O | Ch |
| Batch membership | F | **S manage (own batches)** | S manage | R | R | O | Ch |
| Coach assignment | F | **S (add assistants to own batches)** | R | — | — | — | — |
| Age-group and gender lists | F | R | R | — | — | — | — |
| Sessions (create, edit, publish, cancel) | F | S | S logistics | R | R | O read | Ch read |
| Session plan blocks | F | S | R | — | — | if shared | if shared |
| Attendance (mark) | F | S | R | R | R | O read | Ch read |
| Attendance correction | A | request | request (logistics) | — | — | — | — |
| Absence notice | R, acknowledge | R, acknowledge | R | — | — | C | C |
| Coach notes per athlete | R | S | — | R | — | — | — |
| Reports (aggregate) | F | S | S | R | R | — | — |
| Settings and lists | F | — | — | — | — | — | — |
| Audit | F | — | — | — | — | — | — |
Flat visibility [LOCKED]: staff roles can read all athletes and the organisation calendar; edit and act only in scope. Health status chips are shown to Admin, Coach, Team Manager, Medic, Physio and the athlete or guardian; clinical detail never.

---

## 12. Screens
| ID | Screen | Who |
|---|---|---|
| TR-1 | Venues list and form | Admin |
| TR-2 | Programme list, create with **Batch or Individual** switch, lifecycle | Admin, Coach (own) |
| TR-3 | Batch list, create (Flow T1c), coach assignment, approval queue | Admin, Coach (own) |
| TR-4 | Batch membership editor (add, end, move, bulk) with status chips and batch counts | Admin, Team Manager, Coach (own batches) |
| TR-5 | Calendar (day, week, month; by batch, coach, venue, centre) | All staff |
| TR-6 | Session create or edit with live clash feedback and health summary | Coach, Admin, Team Manager |
| TR-7 | Series editor | Coach, Admin |
| TR-8 | Session plan blocks and templates | Coach, Admin |
| TR-9 | Coach "Today" | Coach |
| TR-10 | Session roster and attendance sheet | Coach |
| TR-11 | Session close | Coach |
| TR-12 | Correction request and approval queue | Coach, Admin |
| TR-13 | Cancel, reschedule, bulk cancel | Coach, Admin |
| TR-14 | Absence notices | Athlete, Guardian, Coach, Admin |
| TR-15 | Athlete schedule and attendance history | Athlete, Guardian |
| TR-16 | Dashboards | Admin, Coach, Team Manager |
| TR-17 | Reports and exports | Admin, Coach (scoped) |
| TR-18 | Settings, closures, reason-code lists | Admin |
| TR-19 | Handover: owned-by-Admin items | Admin |
Every screen has loading, empty, error, success and permission-denied states; mobile first for TR-9, TR-10, TR-14, TR-15.

---

## 13. Notifications
| Event | Recipients |
|---|---|
| Session published or changed | Batch athletes, guardians of minors, session coaches |
| Session cancelled or rescheduled | Same, with reason |
| Reminder 24 h and 2 h | Athletes, guardians of minors |
| Absence notice received | Coach, Admin |
| Attendance not submitted | Coach, then Admin |
| Correction requested or decided | Requester, Admin |
| Low attendance alert | Coach, Admin; guardian only if setting on |
| Batch membership changed | Athlete, guardian (minor) |
| Coach deactivated, items to reassign | Admin |
Notification text carries no health information.

---

## 14. Dashboards and reports
| Dashboard | Tiles |
|---|---|
| **Admin** | Sessions today and this week; attendance not submitted; correction requests; unfilled sessions (no coach); batches over capacity; coach workload; venue utilisation; low-attendance alerts; owned-by-Admin items |
| **Coach** | Today's sessions; my programmes (Batch and Individual); my batches with Fit, Restricted, Unfit counts; attendance to submit; my athletes' attendance trend; notices |
| **Team Manager** | Team calendar; roster changes; venue conflicts; readiness list (status only) |
| **Athlete** | Next sessions, attendance this month, notices |
| **Guardian** | Child's next sessions, attendance, notices |
Reports: attendance by athlete, batch, sport, period; sessions planned vs delivered; coach workload and athlete-to-coach ratio per batch; venue utilisation; cancellations by reason; correction counts; low-attendance list.
Each metric shows its definition and last update time. Exports are permission-checked and audited; health splits hide groups under five.

---

## 15. Non-functional and security
- Server-side permission checks on every request using the active role and organisation from the session; deny by default.
- Organisation isolation tested (an Admin of Org A cannot read Org B).
- No health content stored in this module; health chips are fetched live from the health module.
- Spreadsheet-formula injection neutralised in exports.
- Notifications use neutral text; no personal data in logs or URLs.
- Pagination and search on every list; responsive design; basic accessibility.
- Offline entry is **not** in Phase 1; coach screens must fail gracefully and retry sending attendance.
- Time zone is a setting.

---

## 16. Edge cases
| Case | Expected |
|---|---|
| Venue clash with a shared-use venue | Allowed, with notice |
| Two coaches mark attendance at the same time | One wins; other sees a conflict message |
| Athlete joins batch after the snapshot | Not expected; coach can add as late |
| Athlete is Unfit but attends to observe | Coach marks "Present, observing" with a reason |
| Athlete turns Unfit mid-session | Attendance stays; status change affects future sessions |
| Session spans midnight | Allowed; date is start date; lock counts from end |
| Public holiday or closure | Warn or block per setting; bulk cancel available |
| Coach leaves with future sessions | Owned by Admin; reassign banner |
| Athlete in two batches with overlapping sessions | Athlete clash warning |
| Attendance entered for the wrong session | Correction with reason; original kept |
| Series edited after some sessions completed | Only future sessions change |
| Athlete deactivated | Removed from future snapshots; history kept |
| Same person is Coach and Athlete | Acts per active role; no mixing |
| Guest athlete from another batch | Added with a reason; counts in this session only |
| Minor's guardian not yet consented | No sessions visible to the athlete until Active |
| Weather cancellation affecting many sessions | Bulk cancel with one reason code |
| Capacity ceiling reached | Block with override by Admin and reason |
| Time zone change at organisation level | Existing sessions keep stored times; display changes |

---

## 17. Acceptance tests (Given / When / Then)
**Setup**
1. Given an Admin, when they create a programme, batch and assign a Primary coach, then the coach sees the batch on their dashboard.
2. Given a batch at capacity, when an athlete is added beyond the ceiling, then it is blocked unless Admin overrides with a reason.
3. Given an athlete import with an unknown batch, then the athlete is added without a batch and a warning is shown.
4. Given a membership end, then the athlete is excluded from future sessions and past attendance remains.
**Scheduling**
5. Given two sessions at the same non-shared venue and time, when the second is saved, then it is rejected.
6. Given a coach with overlapping sessions, then the second is rejected.
7. Given an athlete in two overlapping batches, then a warning appears and saving is allowed.
8. Given a weekly series, when "this and following" is edited, then earlier sessions are unchanged.
9. Given a closure date, then creating a session on it shows the configured warning or block.
10. Given a published session, when the time changes, then athletes and guardians receive a change notification.
11. Given a Coach, when they try to create a session in a batch they are not assigned to, then it is denied.
12. Given a Team Manager, when they try to edit a session plan, then it is denied.
**Session day and health**
13. Given a published session, then the roster snapshot is taken at the configured time and frozen at Submit.
14. Given an Unfit athlete, then the roster shows "Unfit" and attendance is not Absent by default.
15. Given an Unfit athlete who attends, when the coach marks "Present, observing" with a reason, then the record is saved with the health status at that time.
16. Given a Restricted athlete, then the roster shows the permitted activity and no diagnosis.
17. Given an athlete with a concern reported, then the Coach sees the marker and the Admin does not.
18. Given a coach opens the health form from a session, then it is pre-filled with the athlete and the session.
19. Given an athlete added late, then the record shows "added late".
20. Given a session with attendance not submitted, when the coach tries to complete it, then it is blocked unless "no attendance" is recorded with a reason.
**Lock and corrections**
21. Given attendance submitted, when the coach edits within the lock window, then it is allowed.
22. Given the lock window has passed, when the coach tries to edit, then it is blocked and a correction request is offered.
23. Given an Admin approves a correction, then a new linked record exists and the original stays.
24. Given a past-dated session, when anyone tries to edit time or venue, then it is denied.
**Cancel and reschedule**
25. Given a cancellation without a reason code, then it is rejected.
26. Given a reschedule, then a new linked session is created and athletes are re-snapshotted.
27. Given a bulk cancel over a date range, then each session shows the reason and notifications go out.
28. Given a venue deactivated with future sessions, then it is blocked until sessions are moved or cancelled.
**Athlete and guardian**
29. Given an athlete, then they see only their own schedule and attendance and no ranking.
30. Given a guardian, when they open another family's athlete, then access is denied.
31. Given a guardian gives an absence notice with reason "Unwell", then the notice contains no medical text and the guardian is offered the health form.
32. Given an unexcused absence streak reaching the threshold, then Coach and Admin are alerted and the alert excludes Unfit and Excused sessions.
**Handover and reports**
33. Given a coach is deactivated, then their batches and future sessions show "Owned by Admin" and Admin is notified.
34. Given an attendance report, then the rate follows the stated definition and excludes Unfit not expected and Excused.
35. Given a report split by health status with fewer than five athletes, then the group is hidden.
36. Given Admin of Org A, when they request any Org B session, then access is denied.
37. Given any create, edit, cancel or attendance mark, then an audit entry with the active role exists.
38. Given a user with Coach and Medic roles, when they switch to Medic, then session editing disappears.
39. Given an Individual programme, then a hidden Batch of one is created for the chosen athlete and sessions work as usual.
40. Given a session spanning midnight, then the lock time is calculated from the session end.
**Programme creation by Coach (new, 24 Nov)**
41. Given a Coach, when they create a programme and choose Batch scope, then they can pick only batches they are assigned to, and the programme is owned by them.
42. Given a Coach, when they choose Individual scope, then they can pick only athletes in their assigned batches; given an Admin, then any athlete.
43. Given an Individual programme is saved, then a hidden Batch of one exists, is absent from batch lists and capacity counts, and has exactly one member.
44. Given a Batch-scope programme with no batch selected, then saving is blocked.
45. Given a programme with sessions, when the Coach tries to change its scope or the athlete, then it is blocked.
46. Given a co-coach, when they add a session to the programme, then it is allowed; when they edit the programme itself, then it is denied.
47. Given another Coach, when they open an Individual programme they do not own, then they can read it as staff but cannot edit it.
48. Given an Individual programme for a minor, then the guardian sees "Individual training" in the child's schedule and other athletes see nothing.
49. Given an Individual programme for an Unfit athlete, then the roster shows Unfit, attendance is not Absent by default, and the session checks run as usual.
50. Given a programme owner is deactivated, then the programme shows "Owned by Admin: reassign".
51. Given an athlete in an Individual programme leaves the organisation, then the programme is completed with a reason and history is kept.
**Batch creation by Coach (new, 24 Nov)**
52. Given a Coach, when they create a batch, then they become owner and Primary coach and the batch appears under "Recently created by coaches" for the Admin.
53. Given a Coach creating a batch, then age group and gender can only be chosen from the Admin lists, and a Coach cannot add a new value.
54. Given a batch with a missing sport, age group, gender category or capacity, then saving is blocked.
55. Given a Coach adds an athlete whose age is outside the age group, then a warning is shown and adding needs a reason.
56. Given an athlete in Invited, Awaiting Guardian Consent or Deactivated status, then they cannot be added to a batch.
57. Given an athlete already in the configured number of active batches, then an overload warning is shown and adding is still allowed.
58. Given a Coach adds a minor to a batch, then the athlete and guardian are notified and the athlete's primary batch for that sport does not change.
59. Given a Coach, when they edit membership of another coach's batch, then it is denied; given a Team Manager on that team, then allowed.
60. Given "Coach batch approval" is on, when a Coach creates a batch, then it is Pending and its sessions cannot be published until the Admin approves.
61. Given a batch with future sessions, when the owner tries to deactivate it, then it is blocked until sessions are moved or cancelled.
62. Given a duplicate batch name in the same centre, then a warning is shown.
63. Given a coach is deactivated, then their batches show "Owned by Admin: reassign".

---

## 18. Build plan for the CLI

### 18.1 Slices (in order)
| Slice | Scope |
|---|---|
| **TR-S1** | Venues, **batch creation by Coach or Admin with guardrails (Flow T1c)**, coach assignment, membership (manual and import link), **programme creation by Coach or Admin with the Batch or Individual option (hidden Batch of one)**, settings, closures; first-run checklist hook |
| **TR-S2** | Sessions: create, edit, series, templates, plan blocks, clash checks, publish, notifications, calendar views |
| **TR-S3** | Session day: roster snapshot, Attendance service entry point, marking, health chips and marker, late and guest athletes, close session |
| **TR-S4** | Lock, corrections, cancel, reschedule, bulk cancel, absence notices, reminders |
| **TR-S5** | Athlete and guardian views, Team Manager dashboard, handover for departed coaches |
| **TR-S6** | Dashboards, reports, alerts, settings screens, audit viewer |
Phase 2 hooks (do not build now): assessment chips on sessions, training load and session RPE, periodisation phases beyond a simple list, S&C Coach planning, video, offline entry.

### 18.2 Definition of done (per slice)
- Acceptance tests for the slice pass and are automated where possible.
- Permissions tested for every role, positive and negative, including direct API calls.
- Audit entries include the **active role**; events are emitted.
- Transitions enforced on the server; no direct status edits.
- Loading, empty, error, success and denied states exist.
- Demo data loads; no real data.
- No hard deletes; history preserved.
- No health content stored or sent in notifications.
- A human technical reviewer has checked access control.
- Documentation and change log updated.

### 18.3 Prompt template
> Build slice {X} of the AMS Training and Sessions module using this context file and the contracts in section 3. Do only this slice. Follow the data model, states, rules and permissions exactly. Create the acceptance tests and make them pass. If anything is unclear or conflicts with the onboarding or health modules, stop and ask. At the end report: what was built, journeys supported, what is mocked, edge cases handled, what remains, assumptions to confirm.

### 18.4 Demo data
1 organisation with 2 centres and a second organisation (isolation test); 6 venues (one shared); 3 programmes (Regular, Camp, Individual); 6 batches across 4 sports; 4 Coaches, 1 Team Manager, 1 Admin, 1 Medic; 40 athletes (6 minors with guardians) with 3 Unfit, 2 Restricted, 1 concern reported; 4 weeks of sessions including a series, a cancellation, a reschedule, a closure; attendance with late, excused, unfit and corrections; absence notices. All labelled **Demo data**.

---

## 19. Open decisions and assumptions to confirm
| # | Item | Default |
|---|---|---|
| Q1 | Who creates programmes and sessions | **Resolved by PM:** Coach creates programmes (Batch or Individual) and sessions in them; Admin may create or edit any |
| Q2 | Attendance lock (T14): 24 hours and Admin approves corrections | As stated |
| Q3 | Team Manager assignable with this module (T15) | Yes |
| Q4 | Individual programme allowed (T16) | **Resolved by PM:** yes, as the Individual scope |
| Q5 | Can athletes mark themselves present (self check-in, QR)? | No, not in Phase 1 |
| Q6 | Guardian notice on low attendance | Off |
| Q7 | Are plan blocks visible to athletes by default | Off |
| Q8 | Should a Coach see other coaches' sessions (flat read) | Yes, read only |
| Q9 | Capacity ceiling number | 110 percent |
| Q10 | Attendance statuses and reason codes | Starter lists |
| Q11 | Metric definition for attendance rate | Stated in section 9 |
| Q12 | Holiday calendar source | Manual closures per organisation |
| Q13 | Mandatory vs optional sessions in reporting | Mandatory by default |
| Q14 | Mobile app vs responsive web | Responsive web |
| Q15 | Whether results entry can bypass the lock for assessments (V2 open point C2) | Yes later; not in this module |
| Q16 | Where "Present, observing" for Unfit athletes counts in reports | Counted as attended, flagged |
| Q17 | Naming: Batch, Squad or Team in the UI | "Batch" internally, label configurable |
| Q18 | Can a Coach also create **batches**? | **Resolved by PM:** yes, with the guardrails in Flow T1c (free creation, Admin-owned lists, warnings, optional approval setting) |
| Q24 | "Coach batch approval" default | **Confirmed by PM:** Off; Admin can switch it on |
| Q25 | May a Coach add any Active athlete in the organisation, or only athletes not in another coach's batch? | **Confirmed by PM (read of "yes"):** any Active athlete (flat read), with overload warning |
| Q26 | Should athletes or guardians be able to refuse a batch add? | **Confirmed by PM:** No; notification only |
| Q27 | Who owns age-group and gender lists | Admin |
| Q28 | Should a hard limit on batches per athlete exist? | Warning at 3, no hard limit |
| Q29 | Who sets an athlete's primary batch per sport | Admin or Team Manager |
| Q19 | Interpretation of "add an option in batch and individual" | Scope switch (Batch or Individual) when creating a programme |
| Q20 | Can a Coach create an Individual programme for an athlete outside their batches (specialist work)? | No; Admin does it or adds the Coach as co-coach |
| Q21 | Can one programme mix Batch and Individual scope? | No; one scope per programme |
| Q22 | Can an Individual programme have a second athlete later (turn into a pair)? | No; create a Batch programme instead |
| Q23 | Individual programme goals and progress | Later (V2.1 goals); not in this module now |

## 20. Glossary
**Programme:** training plan for a period. **Batch:** group of athletes training together. **Session:** one training event. **Roster snapshot:** frozen list of expected athletes. **Lock window:** time after a session when attendance can be edited without approval. **Correction:** approved change after the lock. **Absence notice:** advance notice from athlete or guardian. **Unfit, not expected:** system label for Unfit athletes at a session. **Concern reported:** marker shown to Coach and Medic. **Active role:** the role the user is working in.

## 21. Change log
| Date | Version | Change |
|---|---|---|
| 2026-11-24 | 1.0 | First Training and Sessions context, aligned with onboarding and health modules and AMS V1 and V2 vocabulary |
| 2026-11-24 | 1.2 | PM decision: Coach can create batches. Added Flow T1c (free creation with guardrails, optional approval), rules S-38 to S-49, tests 52 to 63, questions Q24 to Q29; updated T13, C1, roles, batch entity, permissions, settings, journeys, screens, slice TR-S1 |
| 2026-11-24 | 1.1 | PM decision: Coach can create programmes; added the Batch or Individual scope option (Flow T1b, rules S-31 to S-37, tests 41 to 51, questions Q18 to Q23); updated T13, T16, roles, entities, permissions, journeys, screens, dashboards, slice TR-S1 |
