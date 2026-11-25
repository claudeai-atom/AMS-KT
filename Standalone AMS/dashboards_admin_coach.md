# Standalone AMS — Dashboards: Admin, Coach, Team Manager (v0.1 draft, 2026-11-24)

Status: draft. Completes the roadmap Phase 1 row "Basic performance dashboard" (KPIs, attendance, load summary; "no AI, just clean data display"; it is the sales-demo screen). Items **[PROPOSED]** await PM confirmation. Athlete and Guardian dashboards come after. O9 (guardian depth, age-18 transition) stays parked.

Inputs: `training_and_attendance.md`, `athlete_lifecycle_flow.md`, `centre_setup_flow.md`, `onboarding_and_rbac.md`, Phase 2 roadmap rows (wellness, load, readiness, alerts) for reserved slots.

---

## 1. Principles
1. **Every widget answers a decision**, not "shows data": who needs a nudge, what is unmarked, where is attendance slipping, who is unavailable today.
2. **Only data that exists.** v1 widgets draw from Programme, Session, Attendance, Roster, Availability status, Leave. Wellness, RPE/ACWR and readiness are **reserved slots** (Phase 2), hidden until their module ships, never empty boxes.
3. **Fixed layout in v1.** Configurable/drag-and-drop dashboards are roadmap Phase 2; here layouts are fixed per role.
4. **Role-safe.** Coach and Admin see availability **status only** (Fit / Restricted + permitted activity / Unfit), never the cause. Guardian/athlete content is not on these screens.
5. **Every number drills down** to the list behind it, and every list is exportable (CSV/PDF, roadmap Phase 1 reporting).
6. **Demo-ready:** one sensible default view, seeded demo data labelled "Demo data".

## 2. Metric definitions (inherit, do not redefine)
- **Attendance %** = Present ÷ (Present + Missed). **Engagement %** also counts excused. **Not Marked** never enters a denominator but is always shown as a raw count.
- Excused: Holiday, In-Competition, In-Recovery, In S&C, Medically Unavailable, Approved Leave. Missed: Absent, Leave.
- "Not Conducted" sessions leave all denominators.
- Self-reported Unfit not confirmed is excused but shown as "excused, unconfirmed".
- **Hours trained** = sum of Conducted session durations the athlete was marked Present for, by session type.

## 3. Global controls (all dashboards)
- **Date range:** Today / This week (default) / This month / Programme / Custom.
- **Filters:** Centre (if used), Sport, Roster, Programme. Filters persist per user per session.
- **Last updated** timestamp; manual refresh.
- Empty states say what to do ("No sessions scheduled, create one"), with the CTA.

---

## 4. Admin dashboard
**Purpose:** run the centre: is training happening, is it being recorded, where are the gaps.

| # | Widget | Content | Drill-down / CTA | Source |
|---|---|---|---|---|
| A1 | **Today at a glance** | Sessions today (Upcoming / Ongoing / Completed), athletes expected, athletes Unfit/Restricted today (counts only) | Session list | Session, Availability |
| A2 | **Unmarked attendance** | Completed sessions with Not Marked > 0, grouped by supervisor, oldest first | Open session; **nudge supervisor** | Attendance |
| A3 | **Centre attendance** | Attendance % and Engagement % for the range; trend by week; target line **[PROPOSED]** | By Roster / Sport / Programme table | Attendance |
| A4 | **Lowest attendance rosters** | Bottom 5 rosters by Attendance %, with Not Marked share | Roster report | Attendance, Roster |
| A5 | **Athletes needing attention** | Athletes below threshold or with N consecutive Absent (defaults in §8) | Athlete attendance record | Attendance |
| A6 | **Availability snapshot** | Count Fit / Restricted / Unfit now; list with permitted activity for Restricted; **no cause** | Athlete list (status only) | Availability |
| A7 | **Leave & excused** | Leave recorded this week; items awaiting **Approved Leave** decision; minors' leave flagged | Convert to Approved Leave | Leave |
| A8 | **Programmes** | Ongoing / Upcoming / ending in 14 days; any with "No sessions scheduled" | Programme page | Programme |
| A9 | **Hours trained by session type** | Stacked weekly hours (Training, S&C, Recovery, Competition, Assessment, Camp Admin) | Per roster | Session, Attendance |
| A10 | **Venue use** | Sessions per venue; count of **Shared venue** sessions | Shared sessions list | Session |
| A11 | **Set-up health** | Checklist: venues, rosters with members, athletes on no roster, invites pending, minors awaiting guardian consent, staff with no sessions | Fix link per item | Onboarding, Roster |
| A12 | **People** | Active athletes / staff by role; invites Pending / Expired | Import / invite | Onboarding |
| — | *Reserved (Phase 2)* | Readiness board, wellness completion, load/ACWR spikes | | |

Layout order: A1 + A2 (top, action), A3 + A4, A5 + A6, A7 + A8, A9 + A10, A11 + A12.

**Admin CTAs:** Create Programme, Schedule Session, Import athletes/staff, Create Roster, Add Venue, Export current view.

## 5. Coach dashboard
**Purpose:** my day, my squads, who needs me. Scope = sessions where the coach is a supervisor, plus rosters where they are a default coach **[PROPOSED]**.

| # | Widget | Content | Drill-down / CTA | Source |
|---|---|---|---|---|
| C1 | **Today's sessions** | My sessions today with time, venue, roster, expected count, Fit/Restricted/Unfit counts, **Mark attendance** (or Mark all Present) | Marking screen | Session, Availability |
| C2 | **Needs action** | Completed sessions I supervised with Not Marked; leave to record; sessions starting soon with no supervisors confirmed | Open item | Attendance |
| C3 | **My rosters** | Per roster: members, Attendance % (range), Not Marked count, Unfit/Restricted count | Roster page | Attendance, Roster |
| C4 | **Unavailable & restricted** | Athletes in my rosters currently Restricted (with permitted activity) or Unfit; status only | Athlete status card | Availability |
| C5 | **Athletes to follow up** | Low attendance or consecutive Absent in my rosters | Athlete attendance record; **Record leave** | Attendance |
| C6 | **This week calendar** | My sessions Mon-Sun, shared-venue badge, conflicts | Session | Session |
| C7 | **My programmes** | Programmes I own: progress (days elapsed, sessions Conducted vs planned), Phase label, "No sessions scheduled" flag; **Edit programme**, **Extend / Shrink**, **Add Phase**, **Add/remove participants**, **Cancel (Upcoming only)** | Programme page | Programme |
| C9 | **Org attendance (read)** | Centre-wide Attendance % and Engagement % by Roster / Sport / Programme, same definitions as Admin A3; read-only | Report (export) | Attendance |
| C8 | **Hours trained** | Weekly hours by session type for my rosters | Per roster | Session, Attendance |
| — | *Reserved (Phase 2)* | Readiness chips, today's wellness entry, RPE entry, load spikes | | |

Coach CTAs: Mark attendance, Schedule Session, Record leave, **Create Programme**, **Edit/Manage own programmes** (edit details, Phases, participants, extend/shrink, cancel while Upcoming). A coach can edit a programme they **own**; Admin can edit any. Ownership transfers only by Admin (existing rule: departed coach's programmes go to Admin).
Coach **org-wide attendance** is **read-only** (C9): coaches can browse and export reports across the organisation (decision 2026-11-24, resolves D1), but marking and editing stay limited to sessions they supervise.

## 6. Team Manager dashboard (minimal)
Scope = rosters that name them as manager. Pulled forward because Team Managers manage membership.
| # | Widget | Content |
|---|---|---|
| T1 | **My rosters** | Members, Fit/Restricted/Unfit counts (status only), no-members flag |
| T2 | **Upcoming sessions** | Next 7 days for my rosters, venue, supervisors, shared-venue badge |
| T3 | **Roster changes** | Recently added/removed members; athletes awaiting guardian consent on my rosters |
| T4 | **Logistics flags** | Sessions without venue/time confirmed, venue overlaps marked shared |
CTAs: Add members, Import members, Assign managers (own rosters), View sessions. Attendance: read-only summary per roster.

## 7. Behaviour details
- **Today** uses the org timezone; sessions spanning midnight appear on the start day.
- **Nudge supervisor** (A2) sends an in-app/email reminder; one nudge per supervisor per 24h; audited.
- Minors: never show guardian contact data on these dashboards; export follows the existing minor-data rule (guardian columns Admin only).
- Deactivated athletes: excluded from "now" counts, kept in historical figures with the name visible.
- Performance: dashboards read from pre-aggregated daily rollups; a stale-by-minutes figure is acceptable, labelled "Last updated".

## 8. Defaults (editable org settings **[PROPOSED]**)
| Setting | Default |
|---|---|
| Attendance target line | 85% |
| "Needs attention" threshold | Attendance % below 75% over the range, or 3 consecutive Absent |
| Unmarked escalation | Sessions with Not Marked older than 24h turn red on A2 |
| Ending-soon window (A8) | 14 days |

## 9. Reports behind the dashboards (v1)
Attendance by athlete / roster / programme / phase / session type; unmarked sessions by supervisor; hours trained by type; leave log; availability days lost (status counts only). PDF/CSV via the existing export rules. Detailed design inherits `../AMS/module_ams_attendance_reporting.md` (training-session grain).

## 10. Open points
- **D1 (resolved 2026-11-24).** Coach creates and manages own programmes, and reads org-wide attendance reports (read-only). RBAC grid updated to match.
- **D2.** Default thresholds in §8 (85% target, 75% / 3-absent attention): right for SAI HPC norms, or does SAI have a mandated attendance minimum?
- **D3.** Should a Coach see the supervisor-accountability view (A2-style) for peers, or Admin only (proposed)?
- **D4.** Nudge: is a manual nudge enough, or automatic reminders only (already specced at 30 min + next morning)?
- **D5.** A single multi-role user (e.g. Coach + Team Manager): separate dashboards per active role via the role switcher (proposed), not a merged view.
- **D6.** Phase-2 slots: show a teaser ("Readiness: coming soon") in the demo, or hide until built? Proposed hide.
- **D7.** Athlete and Guardian dashboards next, or go straight to the Phase 2 forms engine?
