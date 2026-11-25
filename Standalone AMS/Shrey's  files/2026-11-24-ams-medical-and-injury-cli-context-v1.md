# AMS Medical and Injury Module: CLI Project Context

**Version:** 1.0 | **Prepared:** 24 November 2026 | **Owner (PM):** Shrey Prakash | **Status:** draft, ready to hand to the engineering CLI
**Purpose:** give a coding CLI a complete, durable understanding of the AMS **Medical and Injury** module so it can build it in step with the **Onboarding and RBAC** module built by a colleague. This file is self-contained: it repeats everything decided, so nothing depends on a chat history you cannot see.

---

## 0. How to use this document (rules for the CLI)

1. Read the whole file before changing anything. Then inspect the repository: framework, conventions, existing modules (especially onboarding, roles, persons, organisations, audit, notifications, file storage). **Do not assume a greenfield repo. Do not replace architecture.**
2. Build **one slice at a time** (section 20). Do not build later slices early.
3. Never invent a field, rule, role, integration, clinical list or number. If something is unclear or conflicts with the onboarding module, **stop and ask** with a focused question.
4. Treat every item here as one of these labels:
   - **[LOCKED]** a PM decision. Build it.
   - **[PROPOSED]** a recommendation not yet confirmed. Build it as a **setting** where possible, and report that it is unconfirmed.
   - **[DECISION NEEDED]** open. Build the default stated, and flag.
   - **[CLINICAL REVIEW]** a clinician must confirm before real use. Ship neutral starter lists labelled "review before use".
5. Label what you build as `Implemented`, `Mocked`, `Planned`, `Blocked` or `Needs Decision`. Never claim a test passed that you did not run.
6. Use only synthetic data labelled "Demo data". Never use real medical or personal data in development.
7. Security is a gate (section 18). Do not bypass access checks to make something work.
8. After each slice report: what was built, journeys supported, what is mocked, edge cases handled, what remains, assumptions to confirm.
9. This is a **record-keeping and workflow system**. It must never diagnose, prescribe, advise, or ship clinical thresholds, reference ranges, drug lists or protocols as authoritative. Clinicians own the lists; the system stores and routes.

---

## 1. Product context

- **Product:** **AMS Standalone**, an Athlete Management System. A later link to the wider ATOM platform is intended, but **only `external_id` on Person and Organization is ATOM-specific** [LOCKED by the onboarding draft]. Do not add other ATOM-specific naming.
- **Target customer:** a **High Performance Centre** developing professional athletes across **multiple sports, all age groups and both genders**. Minors are a core population.
- **Tenancy:** one **Organization** per customer. All data is organisation-scoped. The same person can belong to two organisations with separate data.
- **Roles are fixed** (no custom roles). A user holds one or more roles and **works in one active role at a time**; each role has its own dashboard; permissions are never merged.
- **Login:** email only (OTP on first login, then password) [from onboarding].
- **This module:** how athletes, coaches and medical staff record unfitness, injury and illness, manage cases and files, and return athletes to play.

### 1.1 What this module is not
Not a clinical decision tool. Not an electronic hospital record. No billing, insurance, imaging viewer (no DICOM), prescriptions database, or wearable data. Psychology, nutrition, labs, rehab programming and staged return are **Phase 2** (section 22).

---

## 2. Decisions locked (chronological summary)

| # | Decision | Source |
|---|---|---|
| D1 | **Everyone is Fit at onboarding.** No health declaration, screening, clearance, expiry or health gate on activation. | PM, 24 Nov |
| D2 | **Athlete or Coach can log Unfit, injury or illness.** An athlete's self-report takes effect immediately and cannot be refused. | PM |
| D3 | **Return to play: Coach or Medic changes status back to Fit**, "for now". The athlete can only request. | PM |
| D4 | **Medical hold:** when a **Medic** sets **Unfit**, a hold is **on by default**; while on, only a Medic can return the athlete to Fit. Coach can "Request review". | PM ("unfit") |
| D5 | **Case assignment is hybrid:** auto-assign, then pool with claim, then Admin fallback. | PM ("hybrid routing yes") |
| D6 | **Medics are assigned at institution level; multiple Medics are allowed.** | PM ("medic assignment per institution, multiple yes"). Interpretation: see section 23, Q1 |
| D7 | **A Coach cannot close a case when a Medic exists.** Only a Medic closes it. | PM ("no he can't") |
| D8 | **Athletes may attach a photo or document** to a report; it goes to a pending-review tray and only the Medic sees it. | PM ("yes") |
| D9 | **Daily "How is it today?" check applies to everyone** with an open case, not only serious ones. | PM ("for everyone") |
| D10 | Admin sees **availability status only**, never clinical notes, diagnosis or psychological notes. | Onboarding draft |
| D11 | Coach sees **Fit / Restricted / Unfit** and permitted activity only. | Onboarding draft |
| D12 | Minors get **full access to their own dashboard**; guardian consent gates activation; guardian is notified when a minor goes Unfit. | Onboarding draft + PM |
| D13 | No certificates, licences or credential capture anywhere (organisation vouches for its people). | Onboarding draft |
| D14 | Athletes see **no rank or comparison** with other athletes. | AMS V1 rule |
| D15 | Nothing is hard-deleted. Corrections add rows. History is preserved. | Platform rule |
| D16 | The Admin owns a departed person's cases administratively (no clinical access) until reassigned. | Extends onboarding O7 |

---

## 3. Alignment with the Onboarding and RBAC module

The colleague's onboarding module is the **source of truth** for identity, organisations, persons, roles and invitations. This module **consumes** them. Do not duplicate them.

### 3.1 What this module needs from onboarding (contracts)
| Need | Description |
|---|---|
| Organisation scope | `organization_id` from the server session on every request; never from client input |
| Person | `person_id`, name, gender, DOB (age derived), photo, sport, batch or team, status, `external_id` |
| Athlete minor flag | Computed from DOB against the organisation's `minor_age` setting (default 18) |
| Guardian links | Athlete to guardian relationships, link status, end date at 18 |
| Role assignments | Which Person holds which role in the organisation, status |
| Active role | The session's current role; every permission check and audit entry uses it |
| Role availability | The Master Admin setting that decides which roles can be assigned; Medic and Physio become assignable when this module's dashboard exists |
| Centre scope | Optional centre on a Person or role assignment |
| Audit service | One append-only audit writer that records actor, **active role**, organisation, action, entity, old and new, reason |
| Notification service | In-app now; email through a provider interface; supports neutral text templates |
| File storage service | Private object storage with short-lived signed links |
| Invitations and role switching | Handled by onboarding; this module just reads the role |
| Org switcher | Handled by onboarding; this module only sees the current organisation |

### 3.2 What this module provides back
- `availability_status(athlete)` and a status history, for rosters, attendance, nominations and dashboards.
- Events (emit even if nobody listens yet): `health.status_changed`, `health.case_reported`, `health.case_assigned`, `health.case_closed`, `health.return_requested`, `health.athlete_returned`.
- A "case owner" concept for Admin reassignment screens.

### 3.3 Points to confirm with the onboarding colleague (differences found)
| # | Onboarding draft says | This module assumes | Action |
|---|---|---|---|
| A1 | Matrix: "Medic/Physio: Full" medical/injury | **Medic** has organisation-wide clinical access; **Physio** sees only cases where the Medic or Coach adds them (care team) | Confirm |
| A2 | Athlete sees "Own" medical, Guardian "Child" | Athlete and guardian see **status, permitted activity, and items the Medic releases**; not raw notes | Confirm |
| A3 | Team Manager sees no medical data | Team Manager sees **status only** (readiness for competition and travel) | Confirm |
| A4 | Matrix: Coach sees "assigned athletes" for profile; decision 10 says flat org visibility | Coach reads athlete basics org-wide; **acts** only on own athletes; Coach reports and returns **own athletes only** | Confirm |
| A5 | Roles Psychologist, Biomechanist, etc. assignable with their modules | Same; none can change availability status | Aligned |
| A6 | Phone optional for athletes and guardians | Emergency contact phone is **optional but flagged** (Q6) | Confirm |
| A7 | Departed coach: athletes and programmes owned by Admin | Extended: departed Medic's open cases are owned administratively by Admin until reassigned | Confirm |
| A8 | Org flat with optional Centres | Cases carry the athlete's centre; no hierarchy needed | Aligned |
| A9 | Statuses `Invited / Active / Deactivated / Awaiting Guardian Consent` | Cases open for deactivated athletes stay visible to Medic, closed by Medic | Aligned |
| A10 | Terms: availability "Fit / Restricted / Unfit" | Use exactly these; "Unavailable" is retired | Aligned |

---

## 4. Roles and what they do in this module

| Role | Function in medical and injury |
|---|---|
| **Athlete** | Logs self as Unfit or Restricted, adds daily "How is it today?" updates, asks to return, disputes a return ("I'm not ready"), uploads documents (pending review), sees own status and released items |
| **Coach** | Reports athletes Unfit or Restricted, sets permitted activity, sets Fit when no hold applies, raises concerns, asks Medic for review, sees status and permitted activity only |
| **Medic (Doctor)** | Owns clinical cases: claims, triages, assesses, sets status and hold, writes notes, manages files, assigns care team, releases information, authorises return, closes cases |
| **Physiotherapist** | Works assigned cases: treatment notes, proposes return dates; cannot change status [PROPOSED] |
| **Team Manager** | Sees readiness (status) for squads, competitions and travel; no changes |
| **Admin** | Sees status counts; assigns or reassigns case owners (operational, no clinical access); sets the primary Medic per athlete; configures settings |
| **Guardian** | Gives consent (in onboarding); sees child's status; receives neutral notices; sees released items |
| **Master Admin** | No access to health data |
| **Psychologist, Nutritionist, S&C Coach, Biomechanist, Performance Analyst** | **Phase 2**, not assignable yet; none can change availability status |

A person with several roles (for example Coach and Medic) sees medical content **only in the Medic role**.

---

## 5. Availability status model

### 5.1 Values
| Status | Meaning |
|---|---|
| **Fit** | Can train and compete normally. **Default at onboarding.** |
| **Restricted** | Can train with limits. Has a **permitted activity** note. |
| **Unfit** | Cannot train. |
Every non-Fit status has a **reason type** (Injury, Illness, Other), a start date, who set it, and an optional short note.

### 5.2 Who can change status
| Action | Athlete | Coach | Medic | Physio | Admin | Team Mgr | Guardian |
|---|---|---|---|---|---|---|---|
| Fit → Unfit / Restricted (self) | Yes | | | | | | |
| Fit → Unfit / Restricted (athlete) | | Yes (own athletes) | Yes (any) | Propose | No | No | No |
| Unfit ↔ Restricted | Self: Restricted → Unfit only | Yes | Yes | Propose | No | No | No |
| Unfit or Restricted → Fit | **No** (request only) | Yes if no Medical hold | Yes | Propose | No | No | No |
| Set or remove Medical hold | | | Yes | | | | |

### 5.3 Transitions and history
Each change writes an immutable **status entry** (from, to, reason type, permitted activity, who, active role, time, hold flag, note). History is never edited.
"**Return requested**" is a flag set by the athlete, not a status; it notifies Coach and Medic.

### 5.4 Medical hold [LOCKED D4]
- **Default on** when a **Medic** sets **Unfit**. For **Restricted**, default **off** (Medic may switch on).
- While on, only a Medic can set Fit. A Coach sees "Medical hold" and a **Request review** button; it notifies the owning Medic.
- A Coach-set Unfit has **no hold**; a Medic can add one later.
- Removing a hold is a Medic action with an optional reason.

### 5.5 Effects of status elsewhere
- Attendance: Unfit athletes show **"Unfit", not "Absent"**; the coach may override with a reason.
- Competition nomination: Unfit blocks; Restricted needs a Coach or Medic confirmation.
- Rosters, dashboards and readiness lists use the status.
- Status follows the athlete across squads and batches.

---

## 6. Case lifecycle, assignment and routing

### 6.1 One report, one case
Every report (self, coach, or Medic-opened) creates a **Case**. Duplicate reports for the same event link to one case, and each reporter is recorded.

### 6.2 Layers built bottom up
| Layer | Stage | Owner | What happens |
|---|---|---|---|
| 0 | Trigger | Athlete, Coach, Medic | Status becomes Unfit or Restricted at once |
| 1 | Report | Athlete or Coach | Quick fields (section 7) |
| 2 | Auto-enrich | System | Athlete, age, minor flag, sport, squad, session, time, earlier cases |
| 3 | Route | System, Admin fallback | Assign owner |
| 4 | Triage | Medic | Priority, red flags |
| 5 | Assess | Medic | Diagnosis text, severity, mechanism, time-off band |
| 6 | Manage | Medic, Physio | Status, permitted activity, hold, review date, files, care team |
| 7 | Recover | Athlete, Physio | Daily updates, treatment notes |
| 8 | Return | Coach or Medic | Status Fit (hold rule) |
| 9 | Close | **Medic** (Coach only if no Medic) | Outcome, follow-up |
| 10 | Learn | System | Recurrence, aggregate reports |

### 6.3 Case states
`Reported → Unclaimed/Assigned → Accepted → Triaged → Under care → Returned to Fit → Closed → Reopened`
- **Unclaimed** only while waiting in the pool.
- **Coach-managed** (no Medic in the organisation) skips Accepted and Triaged.
- **Returned** and **Closed** are separate; a case can stay open for follow-up after the athlete is Fit.
- Reopen links to the earlier case.

### 6.4 Hybrid routing [LOCKED D5, D6]
1. **One Medic in the organisation** → auto-assign.
2. **Athlete has a primary Medic** (set by Admin or Medic) → auto-assign to that Medic.
3. **Several Medics, no primary** → case enters the **Medic pool**; all **on-duty** Medics are notified; **the first to claim owns it**. Optional routing by sport (a Medic covers certain sports) [PROPOSED setting].
4. **Unclaimed past the timer** → **Admin is alerted** with no clinical detail and may **assign a Medic**. Default timers: **Urgent 10 minutes, Routine 4 hours** (settings).
5. An assigned Medic may **Accept or Decline with a reason**; Decline returns the case to the pool.
6. **No Medic exists** → **coach-managed**; owner is the athlete's coach.
7. **Transfer** any time by the owner or by Admin, with a reason, audited.
8. Medics have an **on-duty toggle**; off-duty Medics are skipped in alerts; Urgent cases with nobody on duty alert all Medics and Admin.

### 6.5 What Admin sees and does
Sees: case id, athlete, status, owner, age of case, unclaimed alert. Does: assign, reassign, set primary Medic, edit routing settings. **Cannot** open notes, diagnosis, files, or the reporter's free text. Assignment never grants clinical visibility.

### 6.6 Red flags and urgency
- A short **red-flag list** (for example head injury, fainting, cannot put weight on it, chest pain, breathing trouble) is **owned and edited by the Medic** per organisation [CLINICAL REVIEW; PM did not answer who owns it, default used].
- Any tick sets **Urgent**, alerts every on-duty Medic immediately, shows the reporter a safety prompt ("Tell your coach or Medic now") and exposes the emergency button.
- The product ships a **neutral starter list** labelled "review before use".

---

## 7. Smart fields

### 7.1 Athlete self-report (30 seconds, phone first)
| Field | Type | Smart behaviour |
|---|---|---|
| Athlete | Prefilled | Logged-in athlete |
| What's happening | Chips: Injury / Illness / Other | Changes next questions |
| Can you train today? | No / With limits / Yes | No → **Unfit**; With limits → **Restricted** and asks "What can you do?" (chips plus text); Yes → case logged, status stays |
| When did it start? | Today / Yesterday / Earlier + date | Defaults to now |
| Where (injury) | **Body map tap** (front and back) + side | Fills body area list |
| How did it happen? | During training / competition / before training / outside the centre / gradually / not sure | "During training" links today's session |
| Symptoms (illness) | Chips: fever, cough, sore throat, vomiting, diarrhoea, headache, body ache, other | Multi-select |
| Pain level | 0 to 10 (optional) | Used in daily updates |
| Red-flag check | Medic-owned list | Any tick → Urgent |
| Short note | 300 characters | Medical class |
| Photo or document | Optional | **Pending review** (Medic only) |
Minors: same form; guardian gets a neutral notice.

### 7.2 Coach report
| Field | Smart behaviour |
|---|---|
| Athlete | Pick from roster, or tap in session |
| What did you see? | Chips: complained of pain, limping, fall or impact, collapsed or dizzy, unwell, not performing, other |
| Can the athlete continue? | No / With limits / Yes → Unfit / Restricted / unchanged |
| Session and time | Prefilled from live session |
| Body area (optional) | Body map |
| Red-flag check | Same list |
| Short note | Optional |

### 7.3 Auto-filled by the system
Athlete, age, minor flag, gender, sport, squad, coach, centre, session link, date and time, context (training, competition, travel, outside), earlier cases (same area, last 12 months), recurrence flag, case id, reporter role, time to claim, time to first response, time open, days lost, sessions missed (from attendance).
Case id format: `CASE-YYYY-0001`, configurable.

### 7.4 Medic fields
| Field | Notes |
|---|---|
| Triage priority | Urgent / Routine; prefilled from red flags |
| Kind | Injury / Illness / Other |
| Body area, side | Prefilled from reports |
| Mechanism | Contact, non-contact, overuse, fall, heat or environment, infection, other [CLINICAL REVIEW] |
| Activity at time | Warm-up, training, competition, conditioning, outside |
| Onset | Sudden or gradual |
| Diagnosis | Free text; optional code from an organisation-supplied list; type-ahead from past entries |
| Severity band | Configurable list |
| Expected time off | Bands (starter: 0 to 3 days, 4 to 7, 8 to 28, over 28) [CLINICAL REVIEW] |
| New / recurrence / exacerbation | Suggested from earlier cases; Medic confirms |
| Status decision | Fit, Restricted, Unfit |
| Permitted activity | **Chips** plus text (No running, No jumping, No contact, Upper body only, Lower body only, Reduced volume, Pool only, Skill work only); chip list editable |
| Medical hold | Toggle; default on for Unfit |
| Review date | Default from severity band |
| Share with coach | Permitted activity and review date (default on); nothing else |
| Care team | Add Physio and others with purpose and end date |
| Files | Category, visibility, version |
| Follow-up | Date and task; default from severity band |
| Guardian notice (minors) | Neutral message; Medic may edit; default on above a severity band |
| Athlete summary | Plain-language text, released when ready |
All lists are organisation **settings**, not code.

### 7.5 Daily update ("How is it today?") [LOCKED D9: everyone]
While a case is open, the athlete (and the athlete only) gets a daily prompt: **Better / Same / Worse**, optional pain 0 to 10, optional short note. **Worse** alerts the Medic (or Coach if coach-managed) and never changes status automatically. Reminder once per day; missed days are not penalised.

### 7.6 Closure fields
Outcome (recovered, referred, ongoing management, other), return date, days Unfit and Restricted (computed), sessions missed (computed), follow-up date, optional short lessons note, recurrence watch. **Medic closes** (Coach only if no Medic).

---

## 8. Medic workspace: logging records

### 8.1 Record types
Case; case note (initial assessment, progress, discharge, addendum, late entry); general health note (not tied to a case); measurement (name, value, unit, date); status entry; task and follow-up; release record. (Phase 2: referral, lab or test result, medication entry, appointment.)

### 8.2 Notes
- **Optional** structured template (Subjective, Objective, Assessment, Plan) [CLINICAL REVIEW]; free text always allowed.
- **Autosave** as draft (visible to author only). **Sign and save** records author, role, time.
- **Edit window** after saving: **15 minutes** (setting) by the author; afterwards only **addenda**, which attach to the original and show both.
- **Entered in error:** hidden by default, retained, reason required.
- **Late entry:** labelled with event time and writing time.
- The reporter's short text is stored unchanged and is separate from the Medic's notes.

### 8.3 Measurements
Name, value, unit, date, optional note. Organisation adds names and units. No ranges shipped; the Medic may type a range as text. Measurements never trigger status changes.

### 8.4 Tasks
Task text, due date, assignee (Medic or care team). Overdue tasks appear in the Medic's inbox.

### 8.5 Medic's day
1. Inbox: new reports (Urgent first), return requests, coach concerns, review requests, follow-ups due, pending files.
2. Claim or accept a case.
3. Read athlete summary (status history, earlier cases, recurrence flag).
4. Assess and write notes; set status and hold; add files.
5. Add care team; write athlete summary; release.
6. Follow up; return the athlete; close.
7. Mark off duty; hand over open cases.

---

## 9. File management

### 9.1 Categories (configurable)
Lab or test report; imaging report; external provider record; prescription or medication letter; medical certificate; injury photo (Medic only); consent or form scan; rehab document; other.

### 9.2 Upload rules
| Item | Rule |
|---|---|
| Types | PDF, JPG, PNG (settable); no executables, archives or macros |
| Size | **20 MB** per file, 10 files per upload (settings) |
| Checks | Content-based type check, malware scan, size and page limit; clear rejection message |
| Storage | **Private** object storage; random file ids; original name kept only as encrypted metadata |
| Photos | Location and device metadata stripped |
| Mobile | Capture in-app, upload directly, do not save to gallery |
| Checksum | Stored for integrity and duplicate warning |

### 9.3 Metadata
Athlete (required); case (optional); category (required); document date (required); provider (optional); **neutral title** (required, must not contain a diagnosis; default "Medical document"); description (medical, optional); visibility (required); allow download (default off for care team, on for released); uploaded by, role, time; version; status (Active, Superseded, Withdrawn, Pending review).
Visibility values: **Medic only (default)**, Care team, Released to athlete, Released to guardian.

### 9.4 Versions and corrections
Replacing creates version 2; version 1 becomes Superseded. A wrong file is **Withdrawn** with a reason, hidden from normal lists, kept in a withdrawn view. **No hard delete.**

### 9.5 Viewing
In-app viewer; short-lived signed links issued after a permission check; **every open, download and share is logged**. Optional view-only watermark. **Search covers titles and metadata, never file contents.** File names and titles never appear in notifications.

### 9.6 Uploads by athletes and guardians
Allowed on their own record or case. They go to **Pending review**; only the Medic sees them. The Medic **Accepts** (sets category and visibility) or **Rejects** with a reason; rejected files are retained. Coaches, Admin and Team Managers cannot upload health files [PROPOSED].

### 9.7 Retention
Default is **keep**; periods need legal review. Athlete leaving the organisation: records stay, restricted. No cross-organisation transfer.

---

## 10. Release, sharing and care team

- **Athlete summary:** the Medic writes a plain-language summary; raw notes are not released by default.
- **Release to athlete:** the Medic selects files or the summary; athlete sees them under "My health"; logged.
- **Release to guardian (minors):** separate toggle, **default on for minors** [DECISION NEEDED, default used].
- **Care team:** the Medic names members (Physio now; others in Phase 2), the items they may see, the purpose and an **end date**. Access ends automatically. Grant and revoke are audited.
- **Share with an outside provider (Phase 2):** needs athlete or guardian consent; time-limited secure link or PDF; logged.
- The athlete can ask for a correction; the Medic adds an addendum.

---

## 11. Persona journeys (from onboarding to end of journey)

### Master Admin
Create organisation and first Admin → control role availability (Medic, Physio on once this module exists) → monitor → suspend or reactivate. **No health data.**

### Admin
Accept invite (OTP, password) → first-run checklist → import staff and athletes (every athlete starts **Fit**) → assign roles → set **primary Medic** per athlete (optional) and routing settings → daily dashboard: counts Fit, Restricted, Unfit; unclaimed-case alerts → assign Medic when alerted → reports (aggregate) → deactivate users; reassign cases of departed Medics.

### Coach
Accept invite → roster with status chips → run session (Unfit shows "Unfit", not Absent) → **report** athlete (two taps) → set Restricted with permitted activity → get notified of self-reports and return requests → **set Fit** (unless hold; then "Request review") → raise concern → planning uses status → deactivated: athletes and cases go to Admin.

### Team Manager
Accept invite → squads and logistics → readiness list for competitions and travel (status only) → cannot change status → deactivated by Admin.

### Medic
Accept invite → dashboard (role switch if also Coach) → on-duty toggle → inbox → claim or accept case → triage and assess → set status and **hold** → files and care team → release summary → follow up → return (or approve coach's request) → close → hand over open cases at end.

### Physiotherapist
Accept invite → assigned cases only → treatment notes → propose return → cannot change status.

### Athlete (adult)
Invite → OTP, password → dashboard (**Fit**) → schedule and attendance → **"I'm not fit to train"** → status banner and what is allowed → daily "How is it today?" → **Ask to return** → coach or Medic sets Fit → optional "I'm not ready" → see released items → exit and alumni (history kept).

### Athlete (minor)
Imported → Awaiting Guardian Consent → guardian invited first and consents → athlete invited, full own dashboard → same as adult; **guardian receives neutral notices** on Unfit and on return to Fit → at 18 guardian access ends unless re-granted; consent reconfirmed.

### Guardian
Invite → OTP → consent (in onboarding) → dashboard: child's schedule, attendance, status → neutral alerts ("marked Unfit to train, please contact the centre", no detail) → released items → withdraw consent (child's access suspended, Admin told) → access ends at 18.

### Injury swimlane
| Step | Athlete | Coach | Medic | Physio | System |
|---|---|---|---|---|---|
| 1 Injury | Self-report Unfit, or | Reports Unfit | | | Status Unfit; history row |
| 2 Alert | | Notified if athlete reported | Notified (neutral) | | Guardian notice for minors |
| 3 Route | | | Claim or accept | | Hybrid routing |
| 4 Assess | | | Notes, diagnosis, hold | | Case Triaged |
| 5 Partial work | | May set Restricted | May set Restricted | Treats | Roster shows permitted activity |
| 6 Recover | Daily updates | | | Notes | Under care |
| 7 Request return | Asks | Notified | Notified | | Flag |
| 8 Return | | Sets Fit (no hold) | Sets Fit, removes hold | | Notices |
| 9 Close | | | Closes case | | Days lost computed |
| 10 Follow up | | | Reminder | | Recurrence watch |

---

## 12. Notifications (neutral wording; never any medical detail)
| Event | Recipients |
|---|---|
| Athlete self-reports Unfit | Coach, owning Medic or pool |
| Coach reports Unfit | Athlete, Medic, Guardian (minor) |
| Urgent report | All on-duty Medics; escalation to Admin if unclaimed |
| Unclaimed beyond timer | Admin |
| Case assigned | Medic |
| Medic sets or changes status | Athlete, Coach |
| Return requested | Coach, Medic |
| Review requested by Coach | Medic |
| Returned to Fit | Athlete, Coach, Guardian (minor) |
| "I'm not ready" after return | Coach, Medic |
| "Worse" daily update | Medic (or Coach if coach-managed) |
| File pending review | Medic |
| Case closed | Athlete (neutral) |
| Care-team grant | Granted staff |

---

## 13. Data model (conceptual; adapt to repo conventions)
Every table: `id`, `organization_id`, `created_at`, `created_by`, `created_in_role`, `updated_at`, `status`. Never hard delete.
| Entity | Key fields |
|---|---|
| `health_status_entry` | athlete_person_id, status, reason_type, permitted_activity, hold, set_by_person_id, set_in_role, note, effective_from |
| `health_case` | case_code, athlete_person_id, kind, body_area, side, mechanism, activity_at_time, onset, severity_band, time_off_band, diagnosis_text, diagnosis_code (nullable), classification (new, recurrence, exacerbation), recurrence_of_case_id, state, priority, owner_person_id, owner_type (medic, coach), claimed_at, closed_at, outcome, follow_up_date |
| `health_report` | case_id, reporter_person_id, reporter_role, occurred_at, context, session_id (nullable), can_continue, symptoms[], pain, free_text, red_flags[] |
| `health_note` | case_id (nullable), athlete_person_id, type, body, author_person_id, signed_at, supersedes_note_id, entered_in_error, late_entry |
| `health_measurement` | athlete, name, value, unit, date, note |
| `health_task` | case_id, text, due_date, assignee_person_id, done |
| `health_file`, `health_file_version` | athlete, case_id, category, document_date, provider, neutral_title, description, visibility, allow_download, storage_key, checksum, original_name_enc, status, uploaded_by, version |
| `health_care_team` | case_id or athlete, member_person_id, role, purpose, items, from, to, granted_by |
| `health_release` | athlete, items, summary_text, released_to (athlete, guardian), released_by, time |
| `health_daily_update` | case_id, athlete, date, trend (better, same, worse), pain, note |
| `health_assignment` | case_id, from_owner, to_owner, by, reason, type (auto, claim, admin, transfer, decline) |
| `health_access_log` | actor, active_role, athlete, data_class, record_type, record_id, action, purpose, time (append-only) |
| `health_primary_medic` | athlete, medic_person_id, set_by |
| `health_medic_duty` | medic_person_id, on_duty, since |
| `health_settings` | routing timers, edit window, file limits, hold default, recurrence window, guardian notice threshold |
| Lists | `body_area`, `kind`, `severity`, `time_off_band`, `mechanism`, `activity_chip`, `red_flag`, `file_category`, `symptom` (organisation-owned) |

---

## 14. Visibility matrix (what each role can see)
Key: **F** full, **R** read, **O** own, **C** child (guardian), **G** if on care team, **Rel** released only, **S** status and permitted activity only, **—** none.
| Data | Admin | Coach | Team Mgr | Medic | Physio | Athlete | Guardian |
|---|---|---|---|---|---|---|---|
| Status, permitted activity, review date | R | R | R | F | R | O | C |
| Case exists (open or closed) | counts + id, owner | R (own athletes) | S | F | G | O | C |
| Reporter's free text | — | own report only | — | F | G | own report | — |
| Diagnosis, notes, measurements | — | — | — | F | G | Rel | Rel |
| Files | — | — | — | F | G | Rel | Rel |
| Athlete summary | — | — | — | F | G | Rel | Rel |
| Medical hold flag | R | R | — | F | R | R | R |
| Case owner | R | — | — | F | R | — | — |
| Care team | — | — | — | F | G | — | — |
| Access log | counts only | — | — | own cases | — | — | — |
| Aggregate reports | R | R | R | R | R | — | — |
Aggregate groups smaller than **5** are hidden.
Always blocked: Admin, Coach and Team Manager never receive diagnosis, notes, files or free text via any screen, API, export, notification or search. Search never returns medical text.

---

## 15. Rules (numbered for tests)
| # | Rule |
|---|---|
| R1 | Every athlete starts Fit. No health step blocks activation or training. |
| R2 | An athlete can mark themselves Unfit or Restricted; it takes effect immediately and cannot be refused. |
| R3 | An athlete cannot set themselves Fit; they can only request. |
| R4 | A Coach can set Unfit, Restricted or Fit for their own athletes; a Medic for any athlete in the organisation. |
| R5 | When a Medic sets Unfit, Medical hold defaults on; only a Medic can remove it. |
| R6 | A Coach cannot set Fit while a hold is on; they may Request review. |
| R7 | Every status change records who, active role, time, from, to, reason type and permitted activity, and is never edited. |
| R8 | Every report creates a case; duplicate reports for one event link to one case. |
| R9 | Hybrid routing: single Medic auto-assign; primary Medic auto-assign; otherwise pool and claim; Admin alerted after timers; no Medic means coach-managed. |
| R10 | First Medic to claim a pooled case owns it. |
| R11 | An assigned Medic may Decline with a reason; the case returns to the pool. |
| R12 | Admin assignment is operational and grants no clinical access. |
| R13 | Only a Medic can close a case when any Medic exists in the organisation. |
| R14 | Any red-flag tick sets Urgent and alerts all on-duty Medics at once. |
| R15 | The reporter's free text is stored unchanged and visible only to the reporter, the Medic and Physio on the case. |
| R16 | Notes cannot be deleted; edits after the window are addenda; "entered in error" hides but retains. |
| R17 | Files are private, versioned, never hard deleted; every open, download and share is logged. |
| R18 | Uploads from athletes or guardians are Pending review until the Medic accepts. |
| R19 | File names, titles and notifications never carry diagnosis or note text. |
| R20 | Search indexes metadata only; never file contents or note text for non-clinical roles. |
| R21 | Care-team access needs a named purpose and end date, and ends automatically. |
| R22 | Only a Medic can release information to the athlete or guardian. Raw notes are not released by default. |
| R23 | For minors, the guardian gets a neutral notice when Unfit begins and when Fit returns. |
| R24 | Unfit athletes are not marked Absent automatically. |
| R25 | A nomination for an Unfit athlete is blocked; Restricted needs confirmation. |
| R26 | Days lost and sessions missed are computed, not typed. |
| R27 | Same body area within the recurrence window (default 90 days) is flagged and linked. |
| R28 | "Worse" in a daily update alerts the Medic and never changes status. |
| R29 | Exports of individual medical data are denied except released records or a consented external share (Phase 2). |
| R30 | A Medic cannot be on the care team or owner of their own record; another Medic handles it. |
| R31 | A departed Medic's open cases are owned administratively by Admin until reassigned. |
| R32 | A person with several roles sees medical data only in the Medic role. |
| R33 | All clinical lists, chips, bands and timers are settings. No clinical thresholds or ranges ship as authoritative. |
| R34 | Health data of one organisation is never visible in another, even for the same person. |
| R35 | Athletes never see rank or comparison. |
| R36 | Master Admin has no access to health data. |

---

## 16. Screens
| ID | Screen | Who |
|---|---|---|
| H-1 | "I'm not fit" form (body map, red flags) | Athlete |
| H-2 | Coach quick report | Coach |
| H-3 | My case and daily update | Athlete |
| H-4 | Coach roster with status chips, permitted activity, review date, concern and review request | Coach |
| H-5 | Medic inbox and pool: Claim, Accept, Decline, on-duty toggle | Medic |
| H-6 | Case list and board | Medic, Physio |
| H-7 | Case detail: header, status controls, timeline, notes, files, tasks, care team, release | Medic, Physio (assigned) |
| H-8 | Assessment form with chips and defaults | Medic |
| H-9 | Note editor | Medic, Physio |
| H-10 | File uploader and viewer, pending-review tray | Medic |
| H-11 | Release panel and athlete summary | Medic |
| H-12 | Care team editor | Medic |
| H-13 | Closure form | Medic (Coach if no Medic) |
| H-14 | Admin: unclaimed and aged cases, assign, primary Medic, settings | Admin |
| H-15 | Team Manager readiness list | Team Manager |
| H-16 | My health (released items) | Athlete, Guardian |
| H-17 | Access log (own cases) | Medic |
| H-18 | Settings: routing, timers, lists, red flags, bands, limits | Admin, Medic |
Every screen has loading, empty, error, success and permission-denied states. Responsive; reports and daily updates must work well on phones.

---

## 17. Dashboards and reports
| Dashboard | Tiles |
|---|---|
| Medic | New reports (Urgent first), return requests, concerns and review requests, open cases by state, follow-ups due, pending files, my on-duty status |
| Physio | My cases, tasks, return proposals |
| Coach | Athletes by status, permitted activity, review dates, concerns raised |
| Admin | Counts Fit, Restricted, Unfit; unclaimed and aged cases; time to claim (counts); days lost by sport (aggregate) |
| Team Manager | Readiness list |
| Athlete and Guardian | Status, next review date, released items |
Reports (aggregate, groups under 5 hidden): availability by sport and centre; injury and illness incidence by sport, body area, severity and period; days lost; recurrence rate; time to claim and first response; return duration; exports audited.
Every tile shows its definition and last update time.

---

## 18. Security, privacy and legal
- Server-side permission checks on every request using the **active role** and organisation from the session. Deny by default; log denials.
- Encrypt medical fields at rest; no medical data in logs, URLs, analytics or error messages.
- Files: content-type check, malware scan, private storage, short-lived links, access logged.
- Short idle session for Medic and Physio; second factor strongly recommended.
- Automated tests for every cell of the visibility matrix, including direct API calls.
- Spreadsheet-formula injection neutralised in any exported file.
- Minors: guardian consent from onboarding is a precondition; neutral guardian notices; legal review under India's DPDP Act for children's data **[DECISION NEEDED]**. This module reports "no health screening for minors" as a **legal and clinical risk** to the PM.
- Retention and deletion: Discovery required; default keep.
- Clinical safety: no automatic status changes from measurements, labs or check-ins; no clinical advice text.

---

## 19. Edge cases
| Case | Expected |
|---|---|
| Athlete marks Unfit, Coach sets Fit same day (no hold) | Allowed; athlete can press "I'm not ready" (notifies only) |
| Coach reports, athlete disagrees | Athlete may add a note; cannot reverse |
| Two reports for one event | One case, both reporters recorded |
| Two Medics claim at once | First claim wins; second sees "claimed by" |
| Medic off duty at an Urgent report | Next on-duty Medic and Admin alerted |
| No Medic in organisation | Coach-managed case; Coach may close |
| Medic leaves with open cases | Admin owns administratively, reassigns |
| Athlete changes squad while Unfit | Status follows athlete |
| Athlete turns 18 while Unfit | Guardian access ends unless re-granted; case continues |
| Status set by mistake | Correct with a new entry and reason |
| Wrong athlete selected for a note or file | Note marked entered in error; file withdrawn; both retained |
| Corrupt or oversized file | Rejected with message |
| Athlete deactivated with open case | Case stays visible to Medic; Medic closes |
| User holds Coach and Medic roles | Medical data visible only in Medic role |
| Athlete injured away from the centre | Context "outside"; same flow |

---

## 20. Build plan for the CLI

### 20.1 Slices (build in order)
| Slice | Scope | Notes |
|---|---|---|
| **H-S1** | Availability status and history: Fit default, athlete and coach log Unfit and Restricted, return rules, Medical hold, notifications, roster chips, attendance effect | Needs onboarding: persons, roles, active role, notification service |
| **H-S2** | Report and case: forms with smart fields, red flags, auto-enrichment, hybrid routing, pool, claim, accept and decline, unclaimed alerts, Admin assignment, coach-managed mode, on-duty toggle | |
| **H-S3** | Medic workspace: assessment fields, notes with edit window and addenda, measurements, tasks, status controls with hold, closure, recurrence | |
| **H-S4** | Files: upload checks, categories, versions, visibility, pending-review tray, viewer, access log | Needs file storage service |
| **H-S5** | Release and care team: athlete summary, release to athlete and guardian, care-team grants with end dates, "My health" views | |
| **H-S6** | Daily updates, return request and "I'm not ready", Team Manager readiness, nomination and attendance checks | |
| **H-S7** | Dashboards and aggregate reports with small-group suppression, settings screens, access log viewer | |
Phase 2 slices are listed in section 22.

### 20.2 Definition of done (per slice)
- All acceptance tests for the slice pass, automated where possible.
- Permissions tested for every role (positive and negative), including direct API calls.
- Audit and events emitted as specified; audit entries include the **active role**.
- States and transitions enforced on the server; no direct status edits.
- Loading, empty, error, success and denied states exist.
- Demo data loads; no real data.
- No hard deletes; history preserved.
- No medical text in logs or notifications.
- Documentation and change log updated.
- A human technical reviewer has checked access control and file handling.

### 20.3 Prompt template (per slice)
> Build slice {X} of the AMS Medical and Injury module using this context file and the onboarding module's contracts in section 3. Do only this slice. Follow the data model, states, rules and permissions exactly. Create the acceptance tests and make them pass. If anything is unclear or conflicts with the onboarding module, stop and ask. At the end report: what was built, journeys supported, what is mocked, edge cases handled, what remains, assumptions to confirm.

### 20.4 Demo data
2 organisations (to test isolation); 2 Medics, 1 Physio, 3 Coaches, 1 Team Manager, 1 Admin; 30 athletes (all Fit by default, plus seeded cases: 3 Unfit with hold, 2 Restricted, 1 Urgent unclaimed, 1 coach-managed organisation case, 1 recurrence), 5 minors with guardians; sample files (fictional, labelled demo); sample daily updates. All labelled **Demo data**.

---

## 21. Acceptance tests (Given / When / Then)

**Status (H-S1)**
1. Given a newly imported athlete, then status is Fit and no health step is pending.
2. Given an athlete, when they mark themselves Unfit (Injury), then status changes at once and Coach and Medic are notified.
3. Given an Unfit athlete, when they try to set Fit, then it is refused and "Ask to return" is offered.
4. Given a Coach, when they set their athlete Restricted with permitted activity, then the roster shows it.
5. Given an athlete outside the Coach's assignment, when the Coach tries to set status, then it is denied.
6. Given a Medic sets Unfit, then Medical hold is on by default; given Restricted, then it is off.
7. Given a hold on, when the Coach tries Fit, then it is blocked and "Request review" notifies the Medic.
8. Given a Medic removes the hold and sets Fit, then Coach, athlete and guardian (minor) receive neutral notifications.
9. Given a minor marked Unfit, then the guardian receives a neutral notice with no detail.
10. Given an Unfit athlete in a session, then attendance shows Unfit, not Absent.
11. Given a nomination for an Unfit athlete, then it is blocked.
**Case and routing (H-S2)**
12. Given one Medic in the organisation, when an athlete reports, then the case is auto-assigned to that Medic.
13. Given an athlete with a primary Medic, then the case goes to that Medic.
14. Given several Medics and no primary, then the case is in the pool, on-duty Medics are notified, the first claimer becomes owner and others see "claimed".
15. Given a Routine case unclaimed beyond the timer, then Admin receives an alert with no clinical text and can assign a Medic.
16. Given Admin assigns a Medic, then Admin still cannot open notes or the reporter's text.
17. Given an assigned Medic declines with a reason, then the case returns to the pool.
18. Given no Medic in the organisation, then the case is coach-managed and the Coach can close it.
19. Given a Medic exists, when the Coach tries to close a case, then it is denied.
20. Given a red-flag tick, then the case is Urgent, all on-duty Medics are alerted and the reporter sees a safety prompt.
21. Given "With limits", then status is Restricted and the coach sees the permitted activity.
22. Given two reports for one event, then they link to one case.
23. Given a body map tap, then body area and side are stored as list values.
24. Given a Medic goes off duty, then pooled alerts skip them; Urgent with nobody on duty alerts all Medics and Admin.
**Medic workspace (H-S3)**
25. Given a saved note, when edited within the window, then the edit is allowed; after the window, then an addendum is created and the original kept.
26. Given a note marked entered in error, then it is hidden by default and retained.
27. Given a recurrence in the same area within 90 days, then the case links to the earlier one and is flagged.
28. Given case closure by a Medic, then days lost and sessions missed are computed and a follow-up date is set.
29. Given a measurement, then no status change occurs automatically.
**Files (H-S4)**
30. Given an oversized or executable file, then upload is rejected.
31. Given a stored file, then its link is not public and expires in minutes.
32. Given a Coach, when they request a medical file by id, then access is denied and logged.
33. Given a replaced file, then version 1 is retained as Superseded.
34. Given an athlete uploads a document, then it is Pending review and invisible to others until accepted.
35. Given a Medic rejects it with a reason, then the file is retained and hidden from normal lists.
36. Given a search for a diagnosis word by a non-clinical user, then no medical text is returned.
**Release and care team (H-S5)**
37. Given a Medic releases a summary to a minor athlete, then the guardian sees it only if the guardian toggle is on.
38. Given a care-team grant with an end date, then access ends automatically.
39. Given a Physio not on the care team, when they open the case, then access is denied.
40. Given any view of a medical record by a clinician, then an access-log entry with purpose exists.
**Daily and returns (H-S6)**
41. Given an open case, then the athlete receives a daily "How is it today?" prompt (every athlete with an open case).
42. Given "Worse", then the Medic is alerted and status is unchanged.
43. Given a return, when the athlete presses "I'm not ready", then Coach and Medic are notified and status is unchanged.
**Privacy and reports (H-S7)**
44. Given a notification or email, then no diagnosis, file name or note text appears.
45. Given an aggregate report with fewer than five athletes in a group, then the group is hidden.
46. Given an export request for individual medical data, then it is denied.
47. Given a user with Coach and Medic roles, when they switch to Coach, then medical detail disappears.
48. Given an Admin of Organisation A, when they request any Organisation B record, then access is denied.
49. Given the Master Admin, then no health data is visible.
50. Given any state change, then an audit entry with the active role exists.

---

## 22. Phase 2 (not now; keep the design open)
Staged return-to-play (configurable stages, Medic approval at each), rehab plans and treatment sessions for the Physio, appointments, referrals and external records, laboratory and blood tests with clinician-entered reference ranges, body composition and nutrition (Nutritionist), wellness check-ins with colour flags, psychological wellbeing (Psychologist; stricter data class; referral carries no reason; "support in place" yes or no visible to staff), recovery interventions, medication and anti-doping flag (optional), external sharing with consent. These roles become assignable when their dashboards exist.

---

## 23. Open decisions and assumptions to confirm
| # | Item | Default used |
|---|---|---|
| Q1 | "Medic assignment per institution, multiple Medics yes": interpreted as Medics are assigned at institution level and several are allowed; per-athlete primary Medic and sport routing are optional settings | As stated |
| Q2 | Who owns the red-flag list | Medic edits; neutral starter list |
| Q3 | Athlete and guardian view: released items only vs the onboarding matrix "own / child" | Released items only |
| Q4 | Physio access: care team only vs onboarding "Full" | Care team only |
| Q5 | Team Manager sees status only | Yes |
| Q6 | Emergency contact phone and emergency button | Optional contact, flagged; simple emergency alert kept |
| Q7 | Guardian release default for minors | On |
| Q8 | Timers: Urgent 10 minutes, Routine 4 hours | As stated |
| Q9 | Note edit window | 15 minutes |
| Q10 | Daily check-in time and reminders | Once per day, non-penalising |
| Q11 | File size and types | 20 MB; PDF, JPG, PNG |
| Q12 | Who may upload files | Medic, Physio, athlete and guardian (pending); not Coach, Admin, Team Manager |
| Q13 | Legal basis for no screening of minors, retention, DPDP | Legal review required |
| Q14 | Coach can set Fit for a self-reported illness with no Medic involvement | Allowed "for now"; shown on Medic dashboard |
| Q15 | Body map graphic and language | Simple outline, English first |
| Q16 | Whether Admin may edit routing settings or only Medic | Both |
| Q17 | Terminology change from "Unavailable" to "Unfit" in older files | Use "Unfit" everywhere |

---

## 24. Glossary
**AMS:** Athlete Management System. **Organization:** the customer tenant. **Person:** a profile within one organisation. **Account:** a login (email). **Active role:** the role the user is working in now. **Fit / Restricted / Unfit:** availability statuses. **Permitted activity:** short note of what an athlete may do while Restricted. **Medical hold:** flag that only a Medic can lift. **Case:** one injury or illness episode. **Pool:** shared Medic queue. **Claim:** a Medic takes ownership of a pooled case. **Care team:** named staff given need-to-know access to one athlete's case. **Release:** Medic shares an item with the athlete or guardian. **Pending review:** uploaded file waiting for the Medic. **Addendum:** a later note attached to an earlier one. **Neutral wording:** text with no diagnosis or clinical detail.

---

## 25. Change log
| Date | Version | Change |
|---|---|---|
| 2026-11-24 | 1.0 | First self-contained context for the Medical and Injury module, aligned with the onboarding and RBAC draft |
