# Standalone AMS

Separate product (own identity, onboarding, orgs, RBAC), integrated with ATOM only later. Target: High Performance Centres (multi-sport, all ages, both genders).

**Scope rule:** work happens here only when the PM says "Standalone AMS". Otherwise "AMS" means ATOM's AMS (`../AMS/`). Do not mix the two; do not assume ATOM onboarding/identity exists here.

Source roadmap: Google Sheet "AMS Roadmap" (id 1ZdKa3JGtnVPlqOwfijsDiXl_kJ3cGAaWTF4j0X5H5yg), ATOM-aligned, used as reference only.

## Files
- `onboarding_and_rbac.md` — locked decisions, role catalog (12 roles), onboarding flows, RBAC matrix, open points.
- `import_design.md` — v0.1 (2026-11-24): staff + athlete templates, validation, duplicates/cross-org, invite lifecycle, error catalogue, 7 open points (O-I1..O-I7).
- `training_and_attendance.md` — v0.1 (2026-11-24): Programme/Pattern/Session/Attendance adapted from ATOM-era decisions + HPC additions (session types, Approved Leave, Medically Unavailable, athlete clash warning, leave flow, optional RPE load); open points O-T1..O-T9.
- `athlete_lifecycle_flow.md` — v0.1 (2026-11-24): 9-stage HPC athlete lifecycle map + detailed Plan/Train (Programme & Session) flow with availability-derived attendance; open points O-L1..O-L6.
- `2026-11-24-ams-medical-and-injury-cli-context-v1.md` — Shrey's Medical & Injury module context (reviewed; 3 blockers open).
- `centre_setup_flow.md` — v0.2 (2026-11-24): Venues (overridable clash = "Shared venue" tag), Rosters (Admin creates), coaches on sessions not athletes, set-up readiness gate; open points O-S2..O-S8.
- `dashboards_admin_coach.md` — v0.1 (2026-11-24): Admin (12 widgets), Coach (8), Team Manager (4) dashboards, metric definitions, defaults; closes roadmap P1 "Basic performance dashboard"; open points D1..D7.
- `market_benchmark_and_gap_analysis.md` — v0.1 (2026-11-24): ATIUM + AthleteMonitoring + Smartabase/Kitman benchmark, 9 market pillars, gap vs roadmap, proposed monitoring-first re-sequencing; open points B-1..B-6.
- `athlete_journey_reference.md` — v0.1 (2026-11-25): living research/reference doc: real HPC journey, stakeholder profiles, journey requirements JR-1..20, sources, competitors, SME/PM feedback log.
- `wellness_and_rpe.md` — v0.2 (2026-11-25): athlete-level daily Hooper check (athlete self-entry, morning notification, Rosters only group the heatmap), coach-entered session RPE and load, baseline signals; open points O-W1..O-W16.
- `readiness_board.md` — v0.1 (2026-11-25): coach morning board: availability chip kept separate from advisory readiness signal (In range/Deviating/Marked deviation/Insufficient data), explainable rules, acknowledgement, daily snapshots, alert event hooks; open points O-R1..O-R12.

## Next
- Import screen spec (upload → preview → result) or Admin dashboard, then other role dashboards.
- Role dashboards (Admin, Coach, Athlete first).
- Parked: O9 guardian dashboard / age-18 transition. Open: legal check on minor self-consent (DPDP).
