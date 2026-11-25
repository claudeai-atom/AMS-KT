# Standalone AMS — Daily Hooper Check & Session RPE (Monitoring core, v0.2 draft, 2026-11-25)

Status: draft v0.2. First monitoring module after Training & Attendance. **v0.2 changes (PM, 2026-11-25):** the Hooper check is an **athlete-level daily log filled by the athlete every morning**, independent of Roster; Rosters only group the coach view. This **supersedes the 2026-11-24 "wellness is coach-entered" rule for the Hooper check**. Session RPE stays coach-entered until the PM says otherwise (O-W15). Items **[PROPOSED]** await PM confirmation; **[SETTLED]** are decided.

Builds on `training_and_attendance.md` §8, `market_benchmark_and_gap_analysis.md` (monitoring-first), `athlete_lifecycle_flow.md` (Stage 5), `dashboards_admin_coach.md` (reserved Phase-2 slots), `athlete_journey_reference.md` (JR-7, JR-8, JR-10).

Scope: (1) **daily Hooper check**, (2) **morning notification**, (3) **session RPE and load**, (4) **signal layer** the readiness board and alerts will read. Out of scope: readiness board, alert rules, injury cases, assessments.

---

## 1. Principles

1. **The check belongs to the athlete, not to a Roster, Programme or session** [SETTLED 2026-11-25]. One entry per athlete per day. Every active athlete can fill it and is expected to, every morning, including rest days.
2. **Athlete self-entry is primary; staff entry is the fallback** [PROPOSED]. Every record carries `entered_by`, `entered_role`, `source` (`athlete_self`, `staff_on_behalf`) and is audited.
3. **A Hooper entry is a signal, never a status.** It never changes availability (Fit/Restricted/Unfit) and never changes attendance. Only the Medical flow changes availability. A bad score can prompt a human to act; it does not create a case on its own.
4. **Compare to the athlete's own baseline**, not to a fixed "good" number.
5. **No default values.** Every item must be tapped. A blank is "Not logged", never zero and never a bad score.
6. **Missing data is never penalised** and is not a disciplinary signal.
7. **Forms-engine minimum.** The questionnaire is stored as a versioned definition plus responses, though v1 ships one fixed questionnaire. First step of the forms engine reused by assessments and intake.

---

## 2. The Hooper check

### 2.1 Questionnaire (v1, fixed) [PROPOSED]
Hooper Index: four items on a **7-point scale**, summed. Search of published studies (2026-11-25) shows consistent scoring: sleep quality 1 = very, very good to 7 = very, very bad; stress, fatigue and muscle soreness 1 = very, very low to 7 = very, very high. **Higher on every item is worse**, so the sum (4-28) rises as the athlete feels worse. Some studies use a 10-point variant; do not compare scores across variants. Sources: [study 1](https://reference-global.com/article/10.2478/tperj-2024-0007), [study 2](https://pmc.ncbi.nlm.nih.gov/articles/PMC7534939) [R:HOOPER-SEARCH, summary read from search results, original 1995 paper not read: TBV].

| Item | 1 | 7 |
|---|---|---|
| Sleep quality (last night) | Very, very good | Very, very bad |
| Stress | Very, very low | Very, very high |
| Fatigue | Very, very low | Very, very high |
| Muscle soreness | Very, very low | Very, very high |

- **Hooper Index** = sum of the four items (4-28). Shown with the items, never instead of them.
- Optional add-ons (not part of the index): **sleep hours** (number), **pain/illness flag** (yes/no), **note** (short text, §7). Mood is **not** in v1 so the index stays the recognised four-item version; it can be added as a separate extra item later (O-W1).
- Athlete-facing wording is neutral and plain (and translatable); the 1-7 labels must be shown at both ends and ideally the middle.
- Athlete sees their own index and trend but **no verdict language** such as "bad" or "at risk".

### 2.2 Who is expected [PROPOSED]
- **Every athlete with an Active account**, every day of the week, rest days and off-days included. Rest-day entries are valuable (illness and recovery show up there).
- **Not expected:** athletes who are Invited or Awaiting Guardian Consent, Deactivated, or on **Approved Leave** that date (no notification; entry still allowed).
- **Unfit athletes remain expected**: their daily scores track recovery. They are not excused from the check.
- Roster, Programme and session membership do **not** affect expectation. An athlete in no Roster is expected and is shown under "No roster" in staff views.
- Admin can **pause the check org-wide** for closures or the off-season (dates, reason); paused days show "Paused", not "Not logged".

### 2.3 Entry [PROPOSED]
| Mode | Who | How |
|---|---|---|
| **Athlete self-entry** (primary) | The athlete | 30-second phone-first form from the morning notification, four taps plus optional fields, one entry per day. Same-day edits by the athlete are allowed and audited. |
| **Staff on behalf** (fallback) | Coach, S&C Coach, Medic/Physio, Admin | Roster grid (rows = athletes, columns = four items) or call-out mode for athletes without a device, signal or account. Flagged `source = staff_on_behalf`. Cannot overwrite an athlete's own entry without an audited edit. |
| **Minors** | The athlete (guardian consent at activation) | Same form; see §7 for the legal condition. |

- A staff entry made when the athlete later logs the same day: the **athlete's entry becomes the record**; the staff entry is kept in history.
- **Time window:** an entry is "on time" if made on the date it describes. Entries made later (up to 7 days) are accepted but flagged `late_entry` and can be excluded from baselines.
- No offline in v1 [SETTLED]; the form keeps a draft on the device and submits when the connection returns.
- No "copy yesterday" mode [PROPOSED, dropped]: with self-entry it only invites fake data.

### 2.4 Pain / illness handoff to Medical
- If the athlete answers **yes** to pain/illness, the form offers **"Tell the medical team"** which opens the athlete self-report flow in the Medical module (`2026-11-24-ams-medical-and-injury-cli-context-v1.md` §7.1). Declining keeps the flag on the entry without creating a case.
- The Hooper entry stores only the flag and the linked case id. Nothing clinical is stored here.
- A Hooper score **never** auto-creates a case or sets availability, because subjective scores are noisy and automatic cases would flood the Medic.

---

## 3. Morning notification [PROPOSED]

| Aspect | Proposal |
|---|---|
| **When** | Org default morning time (suggested 06:30 org timezone), set by Admin. Athlete may choose their own time between set limits. |
| **Who** | Every expected athlete (§2.2). |
| **Content** | Neutral, no health detail: "Your morning check-in is ready." Deep link to the form. |
| **Reminder** | At most **one** reminder (default +2 h) if not done, then nothing more that day. No streak guilt, no escalating nags. |
| **Channels** | v1: web push and email. SMS/WhatsApp considered later (cost, reliability, India usage). **Open O-W14.** No native athlete app in v1. |
| **Minors** | Notification goes to the athlete only. Guardians do **not** get a non-completion alert from this module (they already get the absence digest). |
| **Coach/Admin** | See completion in dashboards. **No per-athlete push to coaches** for a missed check, to avoid turning a signal into surveillance. A daily completion summary is optional [PROPOSED]. |
| **Login** | Link opens a short-lived signed session if the athlete already has an active login on the device; otherwise normal sign-in. A magic-link option is an SME/engineering question (O-W14). |
| **Do not disturb** | Notifications are not sent on Approved Leave, paused days, or after the athlete has already logged. |

---

## 4. Session RPE and training load

Carries over `training_and_attendance.md` §8, unchanged by the Hooper decision.

### 4.1 Capture [PROPOSED detail on SETTLED rules]
- Available on **Conducted** sessions in a Programme with **load capture on** (off by default per Programme).
- **Coach-entered** [SETTLED]: session supervisors plus Admin, same scope as marking. **Open O-W15:** now that athletes log the Hooper themselves, should session RPE also be athlete-logged (standard practice is roughly 30 minutes after the session, per athlete)? Coach entry stays until the PM decides.
- **RPE value:** 1-10 per athlete [SETTLED; O-W2 on the 0-10 convention]. Entered **only for athletes marked Present**.
- **Group-then-adjust** [SETTLED]; records from the group value are marked `bulk_default`.
- Prompt on session completion along with the Not-Marked reminder; missing RPE is **not penalised**. Entries after 24 h are `late_entry`.

### 4.2 Minutes
**Load = RPE × minutes.** Default minutes = scheduled duration; supervisor may override **per athlete** (late, left early). Attendance stays Present; this resolves part of O-L5 for load only.

### 4.3 Derived measures
| Measure | Definition | v1? |
|---|---|---|
| Session load | RPE × minutes per athlete per session | Yes |
| Daily / weekly load | Sums by date and Mon-Sun week (week start is an org setting) | Yes |
| 7-day and 28-day load average | Rolling daily averages | Yes |
| Acute:chronic ratio (ACWR) | 7-day ÷ 28-day average; only when ≥ 28 days of data and the 28-day average > 0 | **Indicator only** |
| Monotony / strain | Weekly mean ÷ SD; × weekly load | Later |

**Caution on ACWR** [OUR knowledge, SME/literature check pending]: widely used but contested as an injury predictor. The product shows it as a load-change indicator and **never labels it "injury risk"** or acts on it automatically.

### 4.4 What RPE does not do
Not part of Attendance % or Engagement % [SETTLED]. Never changes availability.

---

## 5. Baseline and signals (read by the readiness board later)

| Signal | Definition | Notes |
|---|---|---|
| `hooper_index` | Sum of four items, 4-28 | Per athlete per day; higher = worse |
| `hooper_baseline` | Mean of the athlete's last 28 days of non-late entries; needs ≥ 10 entries else "Not enough data" | O-W3 |
| `hooper_delta` | Today minus baseline (points; also SD units when available) | Positive delta = worse than usual |
| `item_delta` | Same per item | One item moving is often the real signal |
| `weekly_load`, `acute_7`, `chronic_28`, `acwr` | §4.3 | |
| `log_completion` | Logged ÷ expected, per athlete, Roster and org | Data-quality signal |
| `rpe_completion` | Athletes with RPE ÷ Present athletes per session | |
| `data_confidence` | Entries in last 28 days and flags (`late`, `staff_on_behalf`, `bulk_default`, `uniform_entry`) | Shown beside every number |

Display colours are **relative to the athlete's baseline** (within usual range / worse than usual / much worse than usual). Cut-offs are configurable defaults to settle with an SME. No "risk" naming until the readiness-board design.

---

## 6. Screens (v1)

| Screen | Audience | Content |
|---|---|---|
| **Morning check form** | Athlete | Four taps, optional sleep hours, pain flag, note; "Tell the medical team" shortcut |
| **My log** | Athlete | Own entries and trend, own load, read-only, no verdict language; line stating who can see this data |
| **Roster heatmap** | Coach, S&C, Medic | Athletes × last 14 days, coloured by delta vs baseline; athletes appear in every Roster they belong to; **"No roster" group** for the rest |
| **Athlete monitoring page** | Coach, S&C, Medic, Psych (item-level); Admin (summary) | 28-day item trend, load bars with 7/28-day lines, ACWR indicator, data-confidence flags, links to availability and attendance |
| **Staff entry grid / call-out** | Coach, S&C, Medic, Admin | For athletes who cannot log themselves; flagged on behalf |
| **Session RPE entry** | Session supervisors | Present athletes, group value, per-athlete adjust, minutes override |
| **Admin completion view** | Admin | Completion by Roster and org, athletes with no recent entries, no-account athletes needing staff entry, pause control, notification settings |

Fills the reserved Phase-2 slots in `dashboards_admin_coach.md`.

---

## 7. Visibility, privacy, compliance

### 7.1 Who can see what [PROPOSED]
| Data | Athlete | Admin | Coach | S&C | Medic/Physio | Psych / Nutritionist | Team Mgr | Guardian (minor) |
|---|---|---|---|---|---|---|---|---|
| Hooper items and index | Own | Summary only | Yes | Yes | Yes | Yes | **No** | **No in v1** |
| Pain/illness flag and linked case | Own | Flag only | Flag only | Flag only | Case per Medical rules | Per Medical rules | No | No |
| Note | Own | Staff roles only | Yes | Yes | Yes | Yes | No | No |
| RPE and load | Own | Yes | Yes | Yes | Yes | Load only | No | No |
| Completion metrics | No | Yes | Own rosters | Own rosters | No | No | No | No |

Coaches see the scores because they are the primary users, so **the athlete must be told this plainly** on the form footer and in My log. Whether an athlete may keep the stress item private from the coach is open (O-W9).

### 7.2 Compliance
- **Minors now self-report health-adjacent data.** This reopens the DPDP legal check that coach entry had avoided (market doc P9). Condition [PROPOSED]: guardian consent at activation names the daily check explicitly, and the legal check clears before the minor-facing form ships. Adults are unaffected. **Blocker for minors only (O-W13).**
- At age 18, consent moves to the athlete and the guardian's consent record is closed.
- Retention and right-of-access for Hooper/RPE data are open and shared with the Medical module (JR-9). Do not finalise here.
- The note field invites clinical or personal detail; placeholder says "observations only, no diagnosis", length limit, staff-visible only. Dropping it is an SME option (O-W5).
- Audit every staff view of item-level data for minors [PROPOSED].

---

## 8. Edge cases
| Case | Behaviour |
|---|---|
| Athlete in no Roster | Expected, notified, can log; shown under "No roster" |
| Athlete in several Rosters | One entry, shown in every Roster heatmap |
| Athlete added or removed from a Roster | History follows the athlete, not the Roster |
| Athlete with no account yet | Not expected; staff may enter on behalf once Active only (no entry before activation) |
| Minor awaiting guardian consent | No check until consent and legal condition are met |
| Athlete logs after staff entered on behalf | Athlete's entry becomes the record; staff entry kept in history |
| Athlete logs twice in one day | Second submission edits the first, audited |
| Approved Leave | No notification; entry allowed |
| Org pause day | No notification, shown as Paused |
| Athlete deactivated | Past records kept, name visible in reports (matches O-T9) |
| Athlete turns 18 | Records unchanged; consent moves to the athlete |
| Athlete changes timezone or travels | Check belongs to the athlete's local date for notification; stored with that date and org timezone for reporting [PROPOSED] |
| All items identical for many days | Flagged `uniform_entry` in data-confidence, not blocked |
| Fewer than 10 entries in 28 days | "Not enough data", no colouring |
| Session Not Conducted after RPE entered | RPE retained, excluded from load, flagged |
| Attendance changed from Present to Absent after RPE | RPE excluded from load until resolved |
| RPE for athlete not Present | Rejected |
| Two sessions in one day | Loads summed for the day |
| Athlete reports pain, declines to tell medical | Flag kept, no case, Medic sees flag only per §7.1 |

---

## 9. Data model (logical)
- `questionnaire` (id, name, version, items[], scale, status)
- `hooper_entry` (id, athlete_id, local_date, questionnaire_id+version, item_scores{}, sleep_hours?, pain_flag, linked_case_id?, note?, entered_by, entered_role, source, late_entry, entered_at, edit history)
- `wellness_settings` (org-level: enabled, notify_time, reminder_offset, paused_ranges[])
- `athlete_notification_pref` (athlete_id, preferred_time, channels)
- `session_rpe` (attendance_record_id, rpe, minutes, load, bulk_default, late_entry, entered_by, entered_role, source, valid)
- Derived values are computed views. All writes audited (actor, active role, time, old → new).
- There is no `roster_monitoring_config`: Rosters do not control expectation.

---

## 10. Requirement trace
| Journey requirement | Covered by |
|---|---|
| JR-7 per-athlete diary/history | My log and athlete monitoring page |
| JR-8 availability separate from wellness | Principle 3, §2.4 |
| JR-10 flagging without leaking detail | §2.4, §7.1 |
| JR-20 upward reporting | Completion metrics seed, no export design yet |

Also added as feedback to `athlete_journey_reference.md` §8 (2026-11-25).

---

## 11. Open points
| # | Question | Proposed answer |
|---|---|---|
| O-W1 | Final items: four-item Hooper only, with optional sleep hours and note; add Mood as a separate extra? | Four items only in v1 |
| O-W2 | RPE scale: 1-10 (settled) or 0-10 | Keep 1-10 for now |
| O-W3 | Baseline window (28 days), minimum entries (10), colour cut-offs | Defaults as stated, later Admin-configurable |
| O-W5 | Keep or drop the free-text note | Keep with guardrails |
| O-W6 | Guardian visibility of the check for minors | None in v1 |
| O-W7 | Staff on behalf: roles and reasons required? | Coach, S&C, Medic, Admin, reason "athlete cannot log" |
| O-W8 | Notification time: org default plus athlete choice, one reminder at +2 h | As stated |
| O-W9 | May an athlete hide the stress item from the coach? | No in v1 (transparency instead) |
| O-W10 | Retention and DPDP treatment of Hooper/RPE data | Shared with Medical, open |
| O-W12 | Will athletes actually complete it daily? Usability and completion test with a real centre | Pilot before sign-off |
| O-W13 | Legal check for minors self-reporting; guardian consent wording | Blocker for minors only |
| O-W14 | Channels (web push and email in v1; SMS/WhatsApp later?) and magic-link sign-in | Web push + email |
| O-W15 | Should session RPE also be athlete-logged (about 30 min post-session)? | Open; coach-entered until decided |
| O-W16 | Cultural and fear risk: athletes under-report to avoid selection or benching | Mitigate by never using scores in selection or attendance; SME to test |
| Resolved | Per Roster vs per athlete | **Per athlete** (PM 2026-11-25) |

## 12. SME questions
1. Do athletes at your centre already fill a daily check? On what, and how reliably?
2. Which time in the morning is realistic, and do hostel or residential athletes have phones available?
3. Do athletes under-report when they believe scores affect selection? What reduces that?
4. Should coaches see stress and sleep for every athlete, or only trends?
5. For minors, what do parents and centres accept regarding self-reporting?
6. How do you use RPE today, and who collects it?

## 13. Risks
1. **Completion and honesty:** athlete self-report is only as good as trust. Fear of selection effects can bias scores down. Mitigation: signals never touch selection or attendance, transparent visibility, SME and pilot test.
2. **Access:** athletes with no phone, data or account leave holes. Mitigation: staff on behalf, visible "no account" list, completion view.
3. **Notification fatigue or intrusion:** one reminder maximum, neutral content, pause control.
4. **Minors and compliance:** reopened DPDP question; minor-facing form is gated on the legal check.
5. **False certainty:** Hooper totals and ACWR read as predictions. Indicator language only until alerts are designed with SME input.
6. **Cultural and language:** form must work in regional languages for teenagers.

## 14. Suggested next
1. PM confirms O-W1, O-W7, O-W8 and O-W15 (change screens), and who owns the O-W13 legal check.
2. Readiness board design reading §5 signals plus availability and attendance.
3. Settle Medical blockers B1-B3 with Shrey in parallel, since readiness and alerts depend on availability being trustworthy.
