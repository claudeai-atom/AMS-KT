# Standalone AMS — Athlete Journey & Stakeholder Reference (living doc)

Status: **living reference, v0.1, created 2026-11-25.** Not a spec. This is the evidence base the specs are checked against: the real athlete journey in a High Performance Centre (HPC), what each stakeholder does and needs, and the requirements the system must meet to support it.

Related specs (the *design* side): `athlete_lifecycle_flow.md` (our 9-stage draft), `onboarding_and_rbac.md`, `import_design.md`, `training_and_attendance.md`, `2026-11-24-ams-medical-and-injury-cli-context-v1.md` (Shrey's Medical doc).

## How to use this document

1. Every claim carries a **source tag**. Never add an untagged claim.
2. Journey and stakeholder sections describe **reality**. Requirements (Section 5) describe **what the system must do about it**. Keep them separate so a requirement can always be traced to a real-world fact.
3. When an SME or the PM gives feedback, log it in Section 8 first (dated, verbatim-ish), then update the affected sections and the changelog.
4. If research and SME disagree, keep both in the text and record the conflict in Section 9. Do not silently pick one.

### Source tags
| Tag | Meaning | Trust |
|---|---|---|
| `[R:id]` | Published research / policy / govt document, listed in Section 6 | High for what it states; check date |
| `[C:id]` | Competitor or comparable product, listed in Section 7 | Medium (vendor claims are marketing) |
| `[SME:name/date]` | Subject-matter-expert conversation, logged in Section 8 | High for practice, may be one centre only |
| `[PM:date]` | Product-owner decision or feedback, logged in Section 8 | Authoritative for our scope |
| `[OUR]` | Our own inference/draft, **not yet validated** by research or SME | Low until tagged otherwise |
| `[TBV]` | To be verified | Do not build on it |

---

## 1. The athlete journey in an HPC

Reference model: a centre such as an SAI National Centre of Excellence (NCOE). Stages are the **real-world** sequence; the "AMS today" column says how our current draft handles each.

| # | Stage | What actually happens | Key evidence | AMS today |
|---|---|---|---|---|
| 1 | **Identification & selection** | Talent is identified and selected through trials/scouting; a committee decides admission. | SAI: all inductions decided by the Talent Identification and Development Committee (TIDC) `[R:SAI-RET]`. Sport NI lists selection/deselection within its talent processes `[R:SNI]`. Athletics NZ: selection on performance and potential `[R:ANZ]`. | **Out of AMS in v1** `[PM:2026-11-24]` (O-L4 to re-confirm) |
| 2 | **Admission checks** | Medical examination and age verification before admission. | SAI NCOE brief: medical checkup and age verification essential `[R:SAI-NCOE]`; admitted athletes must be medically and physically fit `[R:SAI-AR]`. | **Conflict:** draft says "everyone starts Fit, medical baseline not required" `[OUR]`. See Section 9, G1 |
| 3 | **Induction** | Programme duration, content and expectations are confirmed with the athlete; psychological screening starts at induction. | Sport NI induction phase `[R:SNI]`; SAI mandates psychological screening on induction into NCOEs/STCs `[R:SAI-WB]`. | Onboarding done; screening not designed |
| 4 | **Baseline assessment** | Anthropometrics, fitness tests, initial specialist assessments. | SAI recommends scientific assessment of admitted athletes `[R:SAI-NCOE]`; AIS AMS holds S&C, nutrition, psychology, medical modules `[C:AIS]`. | Not designed |
| 5 | **Individual plan** | An individual performance plan sets goals and competition commitments; sessions are scheduled within programmes. | NSW Future Champions: shared Individual Athlete Performance Plan `[R:NSW]`. | **Gap:** we plan at Roster/Programme level only `[OUR]`. See G3 |
| 6 | **Daily training & monitoring** | Sessions, attendance, load, wellbeing/injury monitoring. A training diary is kept per athlete. | SAI: training diary of each athlete is maintained and used for retention decisions `[R:SAI-AR]`; AIS AMS captures athlete-entered questionnaires and integrated monitoring tools `[C:AIS]`. | Designed (training, attendance); Medical reviewed, blockers B1-B3 open |
| 7 | **Health & wellbeing support** | Injury/illness management by medical staff; psychology, nutrition and S&C coordinate for flagged athletes. | SAI: counsellors coordinate with nutritionists and sports scientists `[R:SAI-WB]`. AIS: AMS meets medico-legal needs for medical records `[R:AIS-INJ]`. | Medical doc v1.0 (Shrey), not integrated |
| 8 | **Periodic assessment** | Recurring testing; SAI recommends half-yearly. | `[R:SAI-NCOE]` | Not designed. No cadence `[OUR]` (G4) |
| 9 | **Competition** | Selection, availability, travel, results. | Athletics NZ ties plan to domestic/international seasons `[R:ANZ]`. | Undecided: AMS vs TMS (O-L3) |
| 10 | **Review & retention** | Periodic review decides retain / promote / weed out, using the training diary and assessment results. A committee or Regional Director decides. | `[R:SAI-RET]`, `[R:SAI-AR]`; Athletics NZ reviews once or twice yearly after seasons `[R:ANZ]`. | **Gap:** treated as a coach/admin move, no decision-maker or record `[OUR]` (G2) |
| 11 | **Transition & exit** | Graduation, release, retirement; support through transitions; alumni network. | AIS Clearinghouse transition support and alumni network `[R:AIS-TR]`; Sport Performance Centres report: well-managed exit keeps people in the sport `[R:SPC]`. | Partly (age 18, deactivation); parked O9 |

**Known systemic failure** (affects design): identification is mistaken for development, i.e. thousands are tested and selected but progression is not tracked through the whole pathway `[R:VIN]` (opinion piece, treat as a hypothesis). Implication `[OUR]`: the system's value is in tracking progression, not just recording sessions.

---

## 2. Stakeholder profiles

Each profile: **what they do in the journey / what they need from the system / what they must not see or do / evidence status.** Pain points are hypotheses until an SME confirms them.

### 2.1 Athlete
- **Does:** trains, reports daily status, takes assessments, is reviewed, may compete.
- **Needs:** own schedule, own attendance and load, own availability status, review outcome and goals.
- **Must not:** edit attendance, see other athletes' data, see another person's health detail.
- **Special case, minor:** a guardian co-owns visibility until age 18.
- **Evidence:** `[OUR]` plus AIS questionnaires `[C:AIS]`. Needs SME on how much athletes self-report in practice.

### 2.2 Guardian (minors)
- **Does:** consents at intake; receives notifications and digests.
- **Needs:** absences, unmarked sessions, injury notification in neutral wording, review summaries.
- **Must not:** edit anything; access continues only until the athlete turns 18.
- **Evidence:** `[PM:2026-11-24]` (minor/guardian kept). Depth of guardian access is parked (O9).

### 2.3 Coach
- **Does:** designs and runs sessions, marks attendance, selects for competition, recommends in reviews.
- **Needs:** roster and availability chips, attendance trends, load, assessment results, session notes.
- **Must not:** see the cause of an injury or medical detail (status only); override an athlete's self-reported Unfit without a Medic or return request (proposed fix to blocker B2).
- **Evidence:** `[OUR]`; SAI training diary implies coach-maintained records `[R:SAI-AR]`. Needs SME.

### 2.4 Team Manager
- **Does:** logistics, rosters they manage, travel; may mark attendance.
- **Needs:** roster and availability for travel, schedule clashes.
- **Must not:** create rosters (Admin only) `[PM:2026-11-24]`.

### 2.5 Medic / Physio
- **Does:** injury and illness cases, availability decisions, return-to-play.
- **Needs:** full case history, status control, audit trail, medico-legal record retention.
- **Must not:** be bypassed by coach overrides. Physio sees care-team scope only.
- **Evidence:** AIS AMS must satisfy medico-legal record requirements `[R:AIS-INJ]`. Shrey's Medical doc is the working source. Needs SME.

### 2.6 S&C coach, Biomechanist, Performance Analyst
- **Do:** testing, load management, movement and performance analysis.
- **Need:** assessment capture, benchmarks, trend views, load data.
- **Evidence:** module list in AIS AMS `[C:AIS]`; Assessments module is not yet designed.

### 2.7 Nutritionist, Psychologist / Counsellor
- **Do:** screening at induction and regular checks; coordinate on flagged athletes.
- **Need:** a flag mechanism that does not expose health detail to the wrong role.
- **Evidence:** `[R:SAI-WB]`. Privacy rules for mental-health data need SME and legal input `[TBV]`.

### 2.8 Admin (centre head / office)
- **Does:** imports athletes, creates rosters, programmes, handles exits, oversees data quality.
- **Needs:** centre-wide attendance, unmarked sessions by supervisor, import tools, audit.
- **Evidence:** `[PM:2026-11-24]`.

### 2.9 Retention / selection committee (**not yet modelled**)
- **Does:** decides admission and retention (SAI TIDC), reviews athletes using diary and assessments.
- **Needs:** a review pack per athlete, a decision record with reason and date.
- **Evidence:** `[R:SAI-RET]`. Whether the committee is a user of AMS, or the Admin records its decision, is **open** (Q1).

### 2.10 Funder / federation / govt oversight (**not yet modelled**)
- **Buyer context:** buyer is SAI HPCs, then state departments `[PM:2026-11-24]`. Reporting upward (utilisation, outcomes) is likely required. `[TBV]`

---

## 3. Responsibility matrix (who acts at which stage)

A = accountable, R = does the work, C = consulted/contributes, I = informed, – = not involved. This is `[OUR]` pending SME review.

| Stage | Athlete | Guardian | Coach | Team Mgr | Medic/Physio | S&C / Bio / Analyst | Nutr. / Psych | Admin | Committee |
|---|---|---|---|---|---|---|---|---|---|
| 1 Selection | I | I | C | – | – | – | – | R | A |
| 2 Admission checks | R | R (minor) | – | – | R | – | C | A | I |
| 3 Induction | R | I | C | – | C | C | R (screen) | A | – |
| 4 Baseline | R | I | C | – | C | R | C | I | – |
| 5 Plan | C | I | A/R | R | C | C | C | R | – |
| 6 Train & monitor | R | I | A/R | R | C | C | – | I | – |
| 7 Health support | R | I | I (status only) | I | A/R | C | R | I | – |
| 8 Periodic assessment | R | I | C | – | C | A/R | C | I | I |
| 9 Competition | R | I | A/R | R | C (availability) | C | – | I | – |
| 10 Review & retention | C | I (minor) | R | – | C | C | C | R | A |
| 11 Transition & exit | R | I | C | – | C | – | C | R | A |

---

## 4. The system must fulfil: journey requirements

Requirement IDs are **JR-n** (journey requirements) to avoid clashing with the spec IDs. Status: ✅ in spec, 🟡 partly, ❌ not designed, ⚠ conflict.

| ID | Requirement | Journey stage | Stakeholders | Basis | Status |
|---|---|---|---|---|---|
| JR-1 | Record admission checks (age verification, medical clearance) as completed items before an athlete is marked admitted | 2 | Admin, Medic | `[R:SAI-NCOE]` | ⚠ contradicts "everyone starts Fit" |
| JR-2 | Psychological screening captured at induction, with results visible to care roles only | 3 | Psych, Admin | `[R:SAI-WB]` | ❌ |
| JR-3 | Baseline assessment capture and storage | 4 | S&C, Bio, Analyst | `[R:SAI-NCOE]`, `[C:AIS]` | ❌ |
| JR-4 | Individual plan per athlete (goals, competition commitments) linked to programmes | 5 | Coach, Athlete | `[R:NSW]` | ❌ |
| JR-5 | Programme / roster / session scheduling with validation | 5-6 | Admin, Coach | `[PM:2026-11-24]` | ✅ |
| JR-6 | Attendance marking, snapshots, audit | 6 | Coach, Team Mgr | `[PM]` | ✅ |
| JR-7 | Per-athlete training diary / history viewable at review | 6, 10 | Coach, Committee | `[R:SAI-AR]` | 🟡 attendance and load exist, no unified diary view |
| JR-8 | Daily availability (Fit/Restricted/Unfit) and injury cases with care-team-only detail | 6-7 | Medic, Coach, Athlete | `[R:AIS-INJ]`, Medical doc | 🟡 blockers B1-B3 open |
| JR-9 | Medico-legal record retention for medical data | 7 | Medic | `[R:AIS-INJ]` | ❌ retention rule not defined `[TBV]` |
| JR-10 | Cross-discipline flagging (psych/nutrition/S&C coordinate on a flagged athlete) without leaking detail | 7 | Psych, Nutr., S&C | `[R:SAI-WB]` | ❌ |
| JR-11 | Scheduled assessment cycles (e.g. half-yearly) with due and overdue tracking | 8 | S&C, Admin | `[R:SAI-NCOE]` | ❌ |
| JR-12 | Review/retention workflow: review pack, decision (retain/promote/move/release), decision-maker, reason, date | 10 | Committee, Coach, Admin | `[R:SAI-RET]` | ❌ |
| JR-13 | Review schedule (annual or post-season) with reminders | 10 | Admin | `[R:ANZ]` | ❌ |
| JR-14 | Progression tracking across the pathway (who moved level, when, why) | 10-11 | Admin, Committee | `[R:VIN]` (hypothesis) | ❌ |
| JR-15 | Exit handling: graduation, release, retirement with data retained and access restricted | 11 | Admin | `[R:AIS-TR]` | 🟡 |
| JR-16 | Alumni / transition support | 11 | Admin | `[R:AIS-TR]`, `[R:SPC]` | ❌ parked |
| JR-17 | Guardian visibility and consent for minors, ending at 18 | 1-11 | Guardian | `[PM]` | 🟡 depth parked (O9) |
| JR-18 | Role-scoped views and audit with active role at every stage | all | all | `[PM]` | ✅ |
| JR-19 | Competition selection and availability check | 9 | Coach, Team Mgr | `[OUR]` | ❌ scope open (O-L3) |
| JR-20 | Upward reporting for funders/federations | all | Admin, funder | `[TBV]` | ❌ |

---

## 5. Why Section 4 is separate from our specs

Specs describe *how we chose to build*. This section only says *what reality requires*. After each SME conversation, re-check every ⚠ and ❌ here before accepting a spec change.

---

## 6. Research references

| ID | Source | What it supports | Date/age note |
|---|---|---|---|
| `SAI-RET` | [SAI circular: authorising Regional Directors for retention and weeding out in NCOEs](https://www.sportsauthorityofindia.nic.in/sai_new/public/assets/uploads/circulars/1639128864_Authorising%20RDs%20for%20retention-weeding%20out%20of%20athletes%20in%20NCOEs.pdf) | TIDC decides inductions; RDs can review retention/weeding out | 2021, may be superseded `[TBV]` |
| `SAI-NCOE` | [SAI NCOE scheme brief](https://sportsauthorityofindia.gov.in/sai/public/assets/pdfs/schemes/NCOE_Brief.pdf) | Medical check and age verification before admission; half-yearly scientific assessment | undated |
| `SAI-AR` | [SAI annual report](https://sportsauthorityofindia.nic.in/sai_new/public/assets/uploads/annual_reports/472eee91-ec1e-4293-90a4-dfaf81071bd1.pdf) | Medical and physical fitness for admission on performance; training diary feeds retention | year not confirmed `[TBV]` |
| `SAI-WB` | [SAI Athlete Wellbeing Policy](https://sportsauthorityofindia.nic.in/sai_new/public/assets/uploads/news/a859afd1-1cb1-4211-8fda-13b6c44c3512.pdf) | Psych screening at induction; regular wellbeing checks; counsellor coordination | |
| `NSW` | [NSW Future Champions](https://www.sport.nsw.gov.au/sites/default/files/2021-04/future-champions-strategy-full-version_0.pdf) | Individual Athlete Performance Plan | 2021 |
| `SNI` | [Sport NI talent processes](https://www.sportni.net/performance/talent-id/talent-processes/) | Induction, progression profiles, selection/deselection, benchmarking | |
| `ANZ` | [Athletics NZ HP athlete pathway](https://athletics.org.nz/wp-content/uploads/2022/05/ANZ-HP-framework-23-May-2022.pdf) | Selection on performance and potential; review once or twice yearly | 2022 |
| `AIS-INJ` | [AIS injury reporting position statement](https://www.ausport.gov.au/ais/position_statements/injury-reporting) | AMS fulfils medico-legal medical record requirements | |
| `AIS-TR` | [AIS Clearinghouse: athlete transition in sport](https://www.ausport.gov.au/clearinghouse/evidence/athlete-transition-in-sport) and [AIS Alumni Network](https://www.ausport.gov.au/about/asc-restorative-program/alumni-network) | Transition support, alumni | |
| `SPC` | [Sport Performance Centres: Athlete Performance Pathways Report](https://sportperformancecentres.org/sites/default/files/Athlete%20Performance%20Pathways%20Report.pdf) | Transition quality supports progression and retention in sport | |
| `VIN` | [Vinciamo: Talent Is Everywhere, Pathways Are Not](https://vinciamo-sports.com/media/talent-is-everywhere-pathways-are-not/) | Identification mistaken for development | Opinion/commercial piece |
| `SE` | [Sport England Talent Plan](https://sportengland-production-files.s3.eu-west-2.amazonaws.com/s3fs-public/the-talent-plan-for-england.pdf) | Listed by search as a broad pathway reference; **not yet read** | `[TBV]` |

**Search gaps (as of 2026-11-25):** SAI's Individual Coaching Plan (ICP) and integrated support team documents were not found. UK Sport's pathway and performance data system were not found. SAI NCOE SOPs not found. Next searches: SAI NCOE guidelines/SOP, SAI "Khelo India Athlete" scheme norms, SAI Assessment/Sports Science protocols.

---

## 7. Competitor / comparable product reference

Vendor material is marketing; mark claims as such.

| ID | Product | What is confirmed so far | What we still need |
|---|---|---|---|
| `AIS` | [AIS Athlete Management System](https://ais.gov.au/ams) (built on Smartabase by Fusion Sport, contract awarded 2013 per vendor case study) | National shared platform for AIS, national sporting organisations and institute network; modules for medical, nutrition, S&C, psychology, coaching; captures athlete questionnaires and integrated monitoring tools; vendor reports 15,000+ users `[C:AIS]` | How athlete lifecycle/status is modelled; permissions between disciplines; how retention/review is handled |
| Smartabase (Fusion Sport) | Same platform as above | See AIS row | Configurable forms vs fixed workflows; pricing; India presence |
| Other AMS products | `[TBV]` not yet researched | | Candidates to evaluate: team-sport monitoring suites, medical-record platforms, club management systems. Choose by what the buyer (SAI HPCs, state depts) might already use or be shown. |

**Comparison dimensions to fill for each product:** lifecycle stages supported, role model, minors/guardian handling, medical privacy model, assessment capture, review/retention workflow, India compliance (DPDP), offline/mobile use, price.

---

## 8. Feedback log (PM and SME)

Newest first. Format: date, who, topic, what was said, effect on this doc. SME entries use a name or role plus centre, never personal contact details.

### Product-owner decisions already made (carried from earlier sessions)
| Date | Decision | Effect |
|---|---|---|
| 2026-11-24 | AMS is rebuilt as a separate product with its own identity/onboarding; ATOM link later | Scope of whole doc |
| 2026-11-24 | Buyer is SAI HPCs, then state departments; monitoring-first accepted | Prioritise stages 6-8 |
| 2026-11-24 | Fixed roles, multi-role switcher, bulk import by official email; minors and guardians kept | Section 2 role set |
| 2026-11-24 | Leave and RPE are coach-entered; athletes do not self-log in v1 | Athlete profile 2.1 |
| 2026-11-24 | Rosters created by Admin only; Team Manager manages members; coaches attach to sessions, not athletes | Profiles 2.3, 2.4, 2.8 |
| 2026-11-24 | Selection/trials intake stays outside AMS in v1 (proposed, to re-confirm) | Stage 1 |
| 2026-11-25 | Wanted real HPC lifecycle benchmarked against research; asked for this reference doc | Doc created |
| 2026-11-25 | Daily Hooper check belongs to the athlete (not a Roster), filled by the athlete every morning with auto-notification; Rosters only group the heatmap; every athlete can fill it. Supersedes coach-entered wellness for the Hooper check | Sections 1 stage 6, 2.1, 2.7; JR-8; spec `wellness_and_rpe.md` v0.2; reopens DPDP check for minors |

### SME and PM feedback entries
*(none yet. Add the first entry below this line. Template:)*

```
#### YYYY-MM-DD — <SME role, centre or "PM"> — <topic>
- Said: <what they said, close to their words>
- Contradicts / confirms: <JR-ids, stage numbers, or research IDs>
- Action: <doc sections changed, or "none yet">
```

---

## 9. Gaps and contradictions (current spec vs reality)

| ID | Gap | Evidence | Proposed change | State |
|---|---|---|---|---|
| G1 | Induction assumes everyone starts Fit, no medical baseline | SAI requires medical fitness and age verification at admission `[R:SAI-NCOE]`, `[R:SAI-AR]` | Add admission checklist (JR-1, JR-2) | Open, needs PM + SME |
| G2 | Retention is a coach/admin action, no decision record | `[R:SAI-RET]` committee/RD decides | Add Retention Review step and decision record (JR-12) | Open |
| G3 | No individual athlete plan | `[R:NSW]` | Add athlete-level plan (JR-4) | Open |
| G4 | No assessment or review cadence | `[R:SAI-NCOE]`, `[R:ANZ]` | Scheduled cycles with due/overdue (JR-11, JR-13) | Open |
| G5 | Medical blockers B1-B3 | Shrey doc review | See `athlete_lifecycle_flow.md` | Open, needs Shrey |
| G6 | Funder/federation reporting not modelled | `[TBV]` | Ask buyer-side SME | Open |

---

## 10. Questions for SMEs (bring to the next conversation)

Group by who to ask. Keep answers in Section 8.

**HPC director / admin**
1. Who actually decides admission and retention at your centre, and how is the decision recorded today?
2. How often are athletes formally reviewed, and what documents go into the review?
3. What do you report upward (to SAI or the state), and in what format?
4. What happens to an athlete's data after release or retirement?

**Coach**
5. What do you record per session today, and where (diary, Excel, WhatsApp)?
6. What do you need to know about an athlete's health, and what do you currently get told?
7. How do you handle athletes who train in more than one sport or programme?

**Medic / physio**
8. How are injuries recorded and who can see them today? How long are records kept?
9. Who decides return-to-play, and do coaches ever pressure that decision?

**S&C / sports science**
10. Which tests are done, how often, and how are results stored and compared?

**Psychologist / counsellor / nutritionist**
11. What can be shared with coaches, and what must stay confidential?

**Athlete (and guardian for a minor)**
12. What do you want to see about yourself, and what would you not want a coach to see?

---

## Changelog
| Date | Change |
|---|---|
| 2026-11-25 | v0.1 created. 11-stage real journey, 10 stakeholder profiles, responsibility matrix, 20 journey requirements, 12 research sources, AIS AMS as the one competitor confirmed, feedback log seeded with PM decisions, SME log empty. |
