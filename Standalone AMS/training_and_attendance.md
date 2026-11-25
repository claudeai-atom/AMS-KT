# Standalone AMS — Training Programs, Sessions & Attendance (v0.1 draft, 2026-11-24)

Status: first draft. Adapts the ATOM-era decisions in `../AMS/` (lifecycle, scheduling, attendance, all settled) to the standalone product, and adds what a High Performance Centre needs. Items **[PROPOSED]** await PM confirmation. Builds on `onboarding_and_rbac.md` (fixed roles, flat org visibility, role switcher, minors with full dashboards, guardian consent).

## 1. What carries over unchanged (do not re-debate)
- Grain: **Programme → Schedule Pattern → Session → Attendance record.** Pattern is the series master; Session is the operational unit; attendance is per athlete per session.
- Pattern generates **real Session rows** at creation; multiple concurrent patterns per Programme; pattern edit scope is "this session only" vs "this and following", pattern-scoped, Upcoming-only; `stopped` (pattern) vs `detached` (session) are different things; skip-dates on the pattern; batched conflict check at generation.
- Attendance is **its own entity with its own permission**, gated only on Session status (`Ongoing`/`Completed` markable). Completed never closes marking. **Audit is the only control.**
- **Roster snapshot** at session start.
- **AP3: academy-directed absence is excused; athlete-directed absence is a miss.** `Attendance % = Present ÷ (Present + Missed)`, plus `Engagement %` including excused. Not Marked never enters a denominator.
- Cancelled (called off in advance) vs Not Conducted (abandoned on the day, with reason) stay distinct.
- Cancel/Delete rules on Upcoming; Ongoing has no emergency stop; Completed/Cancelled are terminal.

## 2. What changes for standalone
| Topic | ATOM-era | Standalone |
|---|---|---|
| Cross-org marking / "session assignment beats ownership" | Needed | **Dropped.** One org, flat visibility; marking right = Mark Attendance permission + assignment to the session |
| Zone from VMS | Required | **Venue** is a simple org-owned list (name, centre, optional capacity). No external dependency. (PM briefly removed Venues on 2026-11-24, then reversed: **Venues stay**) |
| Roles | Admin + Supervisor, custom roles | **Fixed catalog.** Marking held by Admin, Coach, Team Manager, S&C Coach (see 6). Role switcher, action audited with active role |
| Participants | Roster + individuals | Same, plus **Squad/Team** as the Roster (name kept: **Roster**). Athlete belongs to many Rosters (multi-sport HPC) |
| Minors | Not modelled | Guardian sees child's attendance (read), is notified of absence (see 7) |

## 3. Entities
- **Programme**: name, sport(s), period (start/end), owner (Coach or Admin), status (Upcoming/Ongoing/Completed/Cancelled), optional **Phases** (name, date range, focus; e.g. General Prep / Pre-Season / Competition / Transition). Phases are labels on a date range for planning and reporting; they never gate anything. **[PROPOSED]** v1 includes Phases because periodisation is the HPC's mental model.
- **Schedule Pattern**: label, days, start/end time, Venue, default Rosters/athletes, default supervisors, **default session type**, date range within Programme, skip-dates, status (active/stopped). Entry point stays on "Schedule Session" (One-time / Recurring toggle), management via Patterns list on Programme details (as settled).
- **Roster** (squad/team; decided 2026-11-24): name, sport(s), optional age-group / gender category, members (athlete + start/end date, so history is kept), status (Active/Archived), created_by. **Created, edited and archived by Admin only**; Team Manager adds/removes members on rosters that name them as manager; Coach reads rosters and uses them in sessions but does not change them. Optional default coaches on a roster only prefill session supervisors; coaches are never assigned to individual athletes. An athlete can be in many Rosters.
- **Session**: date, time, Venue, **Session Type**, Rosters + individual athletes (frozen at start), supervisors (multiple), lifecycle status, conduct outcome, `detached` flag, link to pattern, objectives/notes.
- **Attendance record**: (session, athlete, status, reason, marked_by, marked_role, timestamps, `reconstructed` flag).

### Session types (fixed list) **[PROPOSED]**
`Training` (sport skills), `Strength & Conditioning`, `Recovery`, `Competition/Match`, `Assessment/Testing` (links to Assessments module when it ships), `Meeting/Camp Admin`.
- Type is a **fixed enum**, not authorable (consistent with "no custom anything" stance), and feeds reporting (hours by type) and the excused logic below.
- Why it matters: in the ATOM model "In S&C" and "In-Competition" were athlete statuses because they were *elsewhere*. With typed sessions, an athlete in a S&C session is simply Present *at that session*; the status "In S&C" is only needed when they are excused from a Training session because they are at S&C. Both exist, but the athlete-status list is kept for the "elsewhere" case.

## 4. Statuses
**Session lifecycle:** Upcoming → Ongoing → Completed, or Cancelled (Upcoming only). `Published` transient, internal, never shown. Conduct outcome: Conducted / Not Conducted (reason: Holiday, Weather, Facility Unavailable, Staff Unavailable, Other).

**Athlete attendance status:** `Present, Absent, Leave, Holiday, In-Competition, In-Recovery, In S&C, Medically Unavailable` + system `Not Marked`. Reason mandatory on all but Present.
- **New: Medically Unavailable** (academy-directed, excused). Set by Medic/Physio or Admin. Coach and everyone else see **only the status**, never the cause (RBAC rule: coach sees availability only). Until Health & Injury ships, only Medic/Physio/Admin may set it manually and the reason field is hidden from non-clinical roles.
- Excused bucket: Holiday, In-Competition, In-Recovery, In S&C, Medically Unavailable. Missed bucket: Absent, Leave.
- **Leave counts against attendance (athlete-directed)**, per AP3. **[PROPOSED]** add an *Approved Leave* sub-state set only by Admin (e.g. exams, family emergency, selection trials elsewhere), which is **excused**. Without it an HPC punishes athletes for exam leave. See O-T2.

## 5. Session creation and conflicts
- Conflict checks at creation and pattern generation, batched into **one summary** before commit:
  - **Coach/supervisor double-booking** (hard block, as before).
  - **Venue double-booking** (**overridable warning**, changed 2026-11-24): scheduler may "continue as shared venue"; both sessions get a session-level **Shared venue** badge. No venue-level shared-use flag, no open hours. See `centre_setup_flow.md` S1.
  - **Athlete double-booking (NEW, Warning, not block)**: an athlete in two overlapping sessions across Rosters. Multi-sport athletes make this real; blocking would be wrong since it may be intentional (e.g. shared drill), so surface it with the athlete list.
- **Backdated sessions** permitted for onboarding backfill **[PROPOSED]**: max 30 days back, reason required, records carry `reconstructed = true` and reports can filter them out. (Resolves the open backdating question from the lifecycle doc.)
- Roster change after a session is created but before it starts flows into it; after start it does not (snapshot). Athlete **added to Roster mid-programme appears from next Upcoming session only** (closes the lifecycle doc's retroactive question).
- Removed/deactivated athlete: history preserved, disappears from future sessions. Departed coach: **Admin owns** their sessions and programmes until reassigned (O7 already locked).

## 6. Who can do what
| Action | Admin | Coach | Team Mgr | S&C Coach | Medic/Physio | Other staff | Athlete | Guardian |
|---|---|---|---|---|---|---|---|---|
| Create/edit Programme, Pattern, Session | Full | Own programmes (**confirmed**: create + manage) | Logistics fields only (venue/time) | Own S&C programmes | None | None | None | None |
| Create/edit/archive Roster | Full | No | No (read) | No | No | No | No | No |
| Add/remove Roster members | Full | Managed rosters only | No | No | No | No | No | No |
| Mark attendance | Any | Sessions they supervise | Sessions they supervise | Sessions they supervise | None | None | None | None |
| Set Medically Unavailable | Yes | **No** | No | No | Yes | No | No | No |
| View session attendance | All | All (org-wide visibility) | All | All | All | All | Own | Child |
| Record leave for an athlete | Yes | Yes (own rosters/sessions) | No | No | No | No | No (view only) | No (view only) |
| Set Approved Leave (excused) | Yes | No | No | No | No | No | No | No |

- Marking right = permission **and** being listed as a supervisor on that session (or Admin). Non-supervising coaches can **view** but not mark (flat visibility, scoped action).
- Coach creating programmes themselves changes the ATOM-era "Admin-only". Standalone HPCs have head coaches who own their programme calendar. Confirm in O-T3.

## 7. Athlete & guardian experience
- **Athlete dashboard**: my sessions (calendar + list), my attendance %, engagement %, upcoming load. Read-only: the athlete does not submit leave or RPE (coach-entry rule).
- **Leave flow [DECIDED 2026-11-24: coach enters]**: the athlete/guardian tells the coach outside the system; the **Coach (or Admin) records Leave** for the athlete with dates + reason → affected sessions pre-mark `Leave` (athlete-directed, i.e. missed) → **Admin may convert to Approved Leave** (excused) for exams, family emergency, trials elsewhere. Minors: the **guardian is notified** of every leave recorded for their child.
- **Guardian**: read-only attendance for each child; **notification on Absent/Not Marked-after-session** for minors **[PROPOSED]**, which is the safeguarding minimum when a 14-year-old does not turn up. Digest rather than per-session push.

## 8. Training load (RPE) **[DECIDED 2026-11-24: fields reserved in v1, capture screen in Phase 2, coach-entered]**
After a Conducted session the **supervisor (coach/staff) enters RPE (1-10) for athletes** (one value for the group, then adjust exceptions). Athletes do not self-log (PM: wellness and RPE are coach-entered for all athletes). **Session load = RPE × session duration (minutes)**. Stored per attendance record with `entered_by` / `source` so athlete self-entry can be added later. Surfaced as weekly load per athlete to Coach/S&C/Medic. Not part of the percentages. **Off by default per programme.** v1 reserves the fields; the capture screen ships with Phase 2 load monitoring (roadmap). Bridge to Health & Injury (load spikes → injury risk).

## 9. Marking UX principles
- Marked **on a phone at the ground**: default all athletes to `Not Marked`, one-tap **"Mark all Present"** then flip exceptions. Not Marked stays visible as a count.
- **No true offline in v1 (decided 2026-11-24).** Marking is a lightweight web screen that works on a poor connection; real offline queue + sync arrives with mobile (roadmap Phase 3). Conflicting edits resolved latest-write with both audited.
- Edit history is displayed on the record itself (actor, role, time, old → new, reason).

## 10. Reporting (v1 minimum)
Per athlete, per Roster, per Programme, per Phase: Attendance %, Engagement %, Not Marked count, sessions by type, hours trained, `reconstructed` filter. Export respects the minor-data rules (guardian contact columns only for Admin). Detailed report design inherits `../AMS/module_ams_attendance_reporting.md`.

## 11. Edge cases to carry into the build
- Session spans midnight; DST not relevant (India), but org timezone is a setting.
- Athlete turns 18 mid-programme: guardian notifications stop with their access (onboarding doc).
- Athlete on two Rosters in the same session: one attendance record, not two.
- Session Not Conducted after attendance was marked: records retained but excluded from denominators; flagged.
- Supervisor removed from a session after it has records: records stay, audit preserved.
- Same person is Coach and Medic: marking as Coach, Medically Unavailable only from the Medic role; audit shows the active role.
- Pattern extended on an Ongoing programme: generate from every `active` pattern (settled).

## 12. Open points — ALL RESOLVED 2026-11-24
| # | Decision |
|---|---|
| O-T1 | Fixed six session types kept. `Competition/Match` is a **label only**; full competition handling is TMS later |
| O-T2 | **Approved Leave** (Admin-set, excused): yes |
| O-T3 | **Coaches own their programmes**; Admin can edit any |
| O-T4 | Guardian absence notifications for minors: **mandatory**, sent as a daily digest |
| O-T5 | RPE: fields reserved v1, capture Phase 2, **coach-entered** (see §8) |
| O-T6 | Offline marking: **not in v1**; arrives with mobile (Phase 3) |
| O-T7 | **Phases in v1** (labels only) |
| O-T8 | Athlete double-booking: **warning** |
| O-T9 | After deactivation: **history kept, name visible** in Programme reports |
| New | **Leave is coach-entered** (athlete/guardian do not submit); Admin may mark Approved Leave |
Still parked: O9 guardian dashboard depth / age-18 transition. Shrey's doc still differs (Batch/Individual option, attendance lock + correction approvals, 30-min snapshot, athlete absence notices).

### Original list (for history)
- **O-T1. Session types.** Is the fixed 6-type list right, and does `Competition/Match` belong in AMS at all, or only in TMS later?
- **O-T2. Approved Leave** as an excused state set only by Admin. Confirm, since it changes headline percentages.
- **O-T3. Coach self-serve programmes.** Admin-only (ATOM-era) vs coaches own their programmes. Proposed: coaches own their own.
- **O-T4. Guardian absence notifications** for minors: opt-out or mandatory? Proposed mandatory for minors.
- **O-T5. RPE/load in v1** or reserve the field only?
- **O-T6. Offline marking** in v1? Depends on mobile app scope.
- **O-T7. Phases** in v1 or later? Proposed v1 (cheap, labels only).
- **O-T8. Athlete double-booking** as warning (proposed) vs hard block.
- **O-T9. Attendance after athlete deactivation:** confirm history stays in Programme reports with the name visible (versus anonymised).
- Parked from before: O9 guardian dashboard depth / age-18 transition.

## 13. Suggested next
Admin and Coach dashboards (these two screens use almost everything above), or Health & Injury, whose Medically Unavailable status and load data plug straight into this module.
