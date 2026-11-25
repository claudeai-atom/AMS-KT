# Standalone AMS — Market Benchmark & Gap Analysis (v0.1, 2026-11-24)

**References studied:** ATIUM Sports, AthleteMonitoring.com (the two named), plus Smartabase / Teamworks AMS and Kitman Labs as enterprise anchors.
**Our baseline:** AMS Roadmap sheet (Roadmap / Backlogs / Dummy / AMS V1 / Dummy 2 / Sheet5) + existing Standalone AMS specs (onboarding_and_rbac, import_design, training_and_attendance, athlete_lifecycle_flow, medical & injury context).

**Evidence caveat:** all findings come from vendor marketing, help-centre pages and third-party listings (G2, app stores). Neither reference site could be read page-by-page; no pricing is public for either. Treat as "what the market claims", not verified product behaviour. ATIUM attendance / gym-load / hydration claims were not confirmed on a primary page.

---

## 1. What "market standard" looks like (synthesis)

| # | Pillar | What the references do |
|---|---|---|
| P1 | **Daily athlete-reported monitoring** | Wellness questionnaire (fatigue, sleep, stress, mood, soreness, pain, illness). ATIUM uses the **Hooper Index** (sleep, motivation, hunger…) with coach-assignable questionnaires to athlete/team and a year look-back. Session RPE → training load. |
| P2 | **Readiness & alerting** | AthleteMonitoring's team dashboard is a **traffic-light board**: red = injury risk, yellow = workload struggle, blue = under-performing, green = ready. Preset **and configurable** thresholds; risk-score rules choose which metrics feed it. New health problem → email/SMS to linked practitioners. |
| P3 | **Workload science** | RPE × duration, acute:chronic ratio, monotony/strain; GPS/wearable ingestion. Kitman adds ML risk (add-on) and exposure & availability analytics. |
| P4 | **Configurable forms / testing engine** | Smartabase's core = custom mobile forms with offline entry. AthleteMonitoring: "your own tests, formulas, standards, questionnaires, grading systems"; concussion baseline/post tests. ATIUM has an "AssessPro" assessment tool (e.g. weight + vertical jump). |
| P5 | **Medical / injury lifecycle (EMR-lite)** | Injury & illness reports, diagnoses with codes, SOAP notes, return-to-play protocols and expected RTP date, medical assessment forms; availability status feeds the coach view. |
| P6 | **Role-restricted data** | Per-role control of who sees which form, data and dashboard (medical data walled off from coaches). |
| P7 | **Mobile + offline + push** | Athlete app is the data-capture surface (wellness, attendance, load uploads). Offline sync is table stakes for field use. |
| P8 | **Integrations & BI** | Wearables, GPS, 3rd-party apps (Teamworks claims 100+); API/Tableau/Power BI connectors; league/surveillance feeds. |
| P9 | **Compliance** | HIPAA/GDPR claims, encryption at rest/in transit. Our equivalent: DPDP Act (minors, guardian consent). |

---

## 2. Gap vs our roadmap

Roadmap Phase 1 (MVP) = onboarding, athlete profile, coach mgmt, institution mgmt, invitations, training programme/sessions/attendance, basic dashboard, RBAC, export. Backlog = clash/conflict detection. All Completed except dashboard.

> **CORRECTION (2026-11-24, later same day):** the first version of this section was written from a truncated 9-row sample of the Roadmap tab (only Phase 1 visible). The full tab has **4 phases / 35 rows**: P1 MVP (ops, ATOM-reused), **P2 Growth (wellness, injury, testing, load, squads, dashboards, alerts, competition)**, P3 Scale (mobile, analytics, wearables, EMR-lite, IDP, talent ID, API, multi-sport/centre), P4 Differentiation (AI/ML risk + predictive, video, compliance, marketplace, languages, white-label). The roadmap **is** monitoring-aligned: Phase 2 already holds P1/P2/P3/P4/P5 of the pillars below. The "Our status" column below describes the *Standalone AMS specs written so far*, not the roadmap. The real gaps are in §3.

**Reading (corrected):** the roadmap order is sound. What is missing is (a) a foundation epic for configurable forms, (b) decisions where the roadmap contradicts later decisions (self-reporting, compliance timing, wearables, multi-centre) — see §3.

| Pillar | Our status | Gap |
|---|---|---|
| P1 Wellness / RPE | RPE is **optional** in training_and_attendance (O-T open points); no wellness questionnaire | **Large.** Highest value-per-effort item missing. |
| P2 Readiness board + alerts | Roadmap dashboard = attendance, load summary, KPIs, "no AI yet" | **Large.** No readiness status, no configurable thresholds, no alert routing. |
| P3 Workload analytics | "load summary" only | Medium — ACWR/monotony are simple formulas once P1 data exists. |
| P4 Form / test engine | Dummy-tab Phase 5 "Assessment Engine"; nothing specced | **Large and foundational** — P1, P5 and assessments are all just forms + scoring + standards. Build once. |
| P5 Medical/injury | Shrey's Medical & Injury doc reviewed, 3 blockers open; Dummy Phase 4 | Medium — in flight; align scope to SOAP/diagnosis-code/RTP/availability above. |
| P6 Role-restricted data | RBAC locked (12 roles, 9-resource grid) | **Ahead or at par.** Verify medical resource is hard-walled from Coach. |
| P7 Mobile/offline | Dummy Phase 8, not started | Large, but only the athlete capture surface is needed first (PWA acceptable). |
| P8 Integrations | Import (CSV) designed; ATOM link deferred; no device ingestion | Defer; keep external-ID + generic import. |
| P9 Compliance | DPDP legal check open (minor self-consent) | Open — blocks athlete self-reported health data for minors. |

**Where we are already competitive:** multi-org memberships, guardian-as-user for minors, bulk import with partial commit, programme/session clash detection (zone, athlete, coach, venue — not evident in references), availability-derived attendance, India-specific Excel/PDF export for federations.

---

## 3. Roadmap vs decisions & buyer (SAI HPC) — what to change

The roadmap's phase order stands (P2 = monitoring). Proposed changes:

| # | Roadmap says | Conflict | Proposal |
|---|---|---|---|
| R1 | P2 wellness = "athlete self-reporting via web form" | PM decision: coach enters for all athletes | Change to coach/staff entry (quick grid); keep `source` field for later self-entry |
| R2 | No forms-engine epic | Wellness, testing, injury forms, configurable dashboards all need it → 4 bespoke builds | Add "Forms & scoring engine" as first P2 epic |
| R3 | Compliance in P4 ("start design in P2"); named PDPB | Health data lands in P2; SAI/Ministry contracts require it; PDPB was withdrawn — law is DPDP Act 2023 | Audit trail, role-gated medical data, consent record in P2; rename to DPDP |
| R4 | Wearables (Catapult/Polar) in P3, yet "deal requirement" for SAI centres | Buyer is SAI HPC | Keep P3 for integrations, but ship manual device-file bulk upload in P2 (roadmap already hints) and confirm with SAI which devices are mandatory |
| R5 | Multi-sport + multi-centre in P3 ("contract requirement" at SAI scale) | Tenancy/centre hierarchy is architectural; sport-specific fields (80+) need a sport source standalone | Decide centre hierarchy + sport taxonomy now even if UI ships P3 |
| R6 | Injury in P2, EMR-lite in P3 | Fine, but injury data must be role-walled from coaches from day one | Keep; add medical-resource walling to P2 RBAC check |
| R7 | Native mobile in P3 | Consistent with coach entry | Keep; coach web quick-entry must work offline-tolerant in P2 |
| R8 | Phase 2+ Status column blank; P1 "Completed" | P1 completion was ATOM-based; Standalone has its own build | Clarify status per product, or the roadmap overstates progress |
| R9 | P4 AI/ML, video, marketplace, white-label | Agree — needs 6+ months of data | Leave in P4; log `entered_by`/timestamps now so training data is usable later |

Original proposal below (superseded for sequencing; kept for the vertical-slice detail). A thin vertical slice of monitoring right after attendance:

1. **Form & scoring engine (v1, minimal)** — field types, scoring formula, optional standard/norm table, versioned. Powers everything below.
2. **Daily wellness (Hooper-style) + session RPE** — athlete-submitted, coach-assignable to athlete/squad; promote RPE from optional to core.
3. **Readiness board** — traffic-light per athlete/squad from wellness + RPE load + availability; admin-configurable thresholds; start with 4 fixed states, no ML.
4. **Alert routing** — new injury/illness or red status → linked medical staff + coach (in-app + email/SMS).
5. **Injury lifecycle & RTP** — fold in Shrey's doc; availability status feeds step 3 and attendance (already designed as Medically Unavailable).
6. **Assessments & benchmarking** — on the step-1 engine; India HPC test batteries.
7. Athlete mobile (offline capture) in parallel from step 2.

Defer: ML injury-risk (Kitman sells it as an add-on; accuracy unvalidated), GPS/wearable integrations, nutrition planner, team-selection workflow.

---

## 4. Where we can differentiate (India HPC)
- Federation/HPC-ready exports and Excel-like bulk entry (roadmap: "they will ask in meeting #1").
- Offline + low-bandwidth athlete capture; regional-language wellness forms.
- Minor/guardian consent model built in (DPDP) — references market HIPAA/GDPR, not DPDP.
- Multi-sport, all-age, both-gender norms (bio-banding hooks for later; ties to CSC benchmarking vision).
- Pricing transparency (both references hide pricing) — optional, commercial call.

---

## 5. Open points
- **B-1** Confirm re-sequencing: does monitoring slice (steps 1–4) jump ahead of Phase 4/5 as listed in Dummy tab?
- **B-2** Is RPE/wellness athlete-self-reported only, or can coaches enter on behalf (affects guardian consent for minors)?
- **B-3** DPDP legal outcome on minor self-submitted health data gates P1 for under-18s.
- **B-4** Buyer type still open (federation vs academy vs school) — readiness board depth differs a lot.
- **B-5** Obtain demo access / trial for ATIUM and AthleteMonitoring to verify claims in §1 before locking scope.
- **B-6** Medical doc blockers B1–B3 (from the Shrey review) still open.

## 6. Decisions & roadmap alignment (added 2026-11-24, PM input)
- **B-1 RESOLVED:** monitoring-first slice accepted.
- **B-4 RESOLVED:** primary buyer = SAI High Performance Centres; secondary = state sports departments (not federations/academies/schools).
- **B-2 RESOLVED:** wellness/RPE for **all** athletes is entered by the coach (or staff), not self-submitted. Consequences: coach quick-entry/bulk grid per squad, offline-tolerant web first, athlete app no longer gates monitoring, DPDP minor self-consent not a blocker for wellness (consent sits at enrolment). Risk: coach-entered subjective scores are a proxy for athlete self-report; keep an `entered_by` / `source` field so athlete self-entry can be added later without migration.
- **B-5:** no demo access; PM's 2022 ATIUM internship is the only first-hand reference (to be tapped for recall, not treated as current).
- **Roadmap alignment finding (corrected):** the Roadmap tab IS monitoring-aligned (Phase 2 = wellness, injury, testing, load, alerts). An earlier version of this note claimed otherwise based on a truncated read; that was wrong. The Dummy tab is a separate RFP-style 8-phase list and is not the roadmap. Real deltas are R1–R9 in §3.

## Sources
- ATIUM: [features](https://www.atiumsports.com/features), [Hooper Index dashboard](https://atiumsuccess.freshdesk.com/support/solutions/articles/89000018497-how-to-view-wellness-questionnaire-hooper-index-dashboard-on-atium-sports-mobile-app), [coach wellness questionnaire](https://atiumsuccess.freshdesk.com/support/solutions/articles/89000019827-how-to-take-wellness-questionnaire-from-coaches-environment-mobile-app-), [AssessPro](https://atiumsuccess.freshdesk.com/support/solutions/articles/89000018698-how-to-conduct-weight-vertical-jump-assessment-in-assesspro), [G2](https://www.g2.com/products/atium-sports/reviews)
- AthleteMonitoring: [home](https://www.athletemonitoring.com/), [pricing](https://www.athletemonitoring.com/pricing/), [team dashboard](https://support.en.athletemonitoring.com/support/solutions/articles/13000039236-the-team-dashboard), [alert & risk scoring](https://support.en.athletemonitoring.com/support/solutions/articles/13000119547-how-to-configure-your-alert-and-risk-scoring-system), [testing](https://www.athletemonitoring.com/sport-testing-assessment/), [health/EMR](https://www.athletemonitoring.com/health/)
- Smartabase / Teamworks: [pro teams](https://smartabase.com/solutions/pro-sports-teams/), [Sports Intel (AIS) platform](https://support.sportsintel.ausport.gov.au/hc/en-us/articles/4899177324815)
- Kitman Labs: [performance optimization](https://www.kitmanlabs.com/gb/platform/performance-optimisation/)
