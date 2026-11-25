# Standalone AMS — Readiness Board (Monitoring core, v0.1 draft, 2026-11-25)

Status: first draft. Second monitoring module, after `wellness_and_rpe.md` v0.2. Items **[PROPOSED]** await PM confirmation; **[SETTLED]** are earlier decisions. Inputs: `wellness_and_rpe.md` §5 signals, availability from the Medical module (`2026-11-24-ams-medical-and-injury-cli-context-v1.md`), attendance (`training_and_attendance.md`), `market_benchmark_and_gap_analysis.md` (readiness pillar P2: AthleteMonitoring traffic-light board, "start with 4 fixed states, no ML"), `dashboards_admin_coach.md` (reserved slots).

Purpose: one screen where a coach, S&C coach or medic sees, **before training**, which athletes need a closer look today and why. It is a **decision aid**, not a gate. Out of scope here: alert routing (next doc), injury lifecycle, assessments.

---

## 1. Principles

1. **Two separate things on the board, never merged:** the **availability chip** (Fit / Restricted / Unfit, authoritative, owned by Medical) and the **readiness signal** (derived, advisory, owned by this module). One combined colour would imply a good wellness score can override a medical status, or a bad one can bench an athlete. This deliberately differs from the single traffic light in the AthleteMonitoring benchmark.
2. **Advisory only.** Readiness never changes availability, never blocks marking attendance, never selects or drops an athlete from a session, squad or competition.
3. **Explainable.** Every non-normal state lists exactly which rules fired and the numbers behind them. No hidden score, no ML in v1 [SETTLED: monitoring-first, "no ML"].
4. **Relative to the athlete's own baseline** (see `wellness_and_rpe.md` §5), with data-quality shown beside every number.
5. **Missing data is its own state, not a bad one.** "Not logged" and "Building baseline" are never coloured as concern.
6. **Few flags beat many.** A board that flags half the squad is ignored. Defaults aim for a small number of flagged athletes on a typical day (see §8, calibration).
7. **No "risk" or "injury" language.** The states describe what the data shows, not a prediction.

---

## 2. Inputs

| Input | Source | Used for |
|---|---|---|
| Availability status and permitted activity (Fit / Restricted / Unfit) | Medical | The chip; excludes Unfit athletes from readiness colouring |
| Hooper index and four items, today and recent days | Athlete daily log | Wellness rules |
| Hooper baseline and delta | `wellness_and_rpe.md` §5 | Wellness rules |
| Weekly load, 7-day and 28-day averages, ACWR | Session RPE | Load rules (only when data is complete) |
| Log completion and RPE completion | Wellness / RPE modules | Data-quality rules |
| Attendance in the last 7 days | Attendance | Context column only, not a rule input in v1 |
| Pain/illness flag on the day's entry | Hooper entry | Context chip "Reported pain/illness" (flag only) |

Not inputs in v1: assessments, wearables, nutrition, free-text notes.

---

## 3. States [PROPOSED]

**Readiness signal (advisory), for athletes whose availability is Fit or Restricted:**
| State | Meaning | How it is reached |
|---|---|---|
| **In range** | Data is within the athlete's usual variation | No rule fired |
| **Deviating** | One thing differs from the athlete's baseline | One rule fired on the day |
| **Marked deviation** | Worth a conversation today | Two independent rule groups fired, or one rule fired for 3 or more consecutive days |
| **Insufficient data** | Not enough to judge | Not yet logged today (before the cut-off), no log for N days, or baseline not built |

**Separate availability chip:** Fit, Restricted (with permitted activity), Unfit. **Unfit athletes show the chip only**, with no readiness colour, and a small recovery sparkline of their Hooper trend.

Naming (**PM chose Option A, 2026-11-25**): In range / Deviating / Marked deviation / Insufficient data. Chosen because every rule compares the athlete with their own baseline, so the labels describe deviation from baseline rather than recovery, readiness or risk; avoids clinical terms (overreaching, fatigued) and clearance wording (ready/not ready). The board column is headed "Status vs baseline"; the module keeps the name Readiness Board. Wording to be confirmed with an SME; the four-state count follows the market synthesis (O-R1).

---

## 4. Rules (v1 defaults, all Admin-configurable later) [PROPOSED]

Defaults are **placeholders to calibrate with an SME and pilot data**; none are validated cut-offs.

### 4.1 Wellness rule group
| Rule | Fires when |
|---|---|
| W1 Index above usual | Today's Hooper index is ≥ 1 SD above the athlete's baseline (or ≥ 3 points above when SD is unavailable) |
| W2 Item extreme | Any single item scores 6 or 7 |
| W3 Persistent drift | Hooper index above baseline on 3 consecutive days |

### 4.2 Load rule group (only evaluated when load data is complete, see 4.4)
| Rule | Fires when |
|---|---|
| L1 Load jump | Weekly load ≥ 30% above the previous week's, and above the athlete's 28-day weekly average |
| L2 ACWR outside band | ACWR above 1.5 or below 0.8 (shown as "load change", never as injury risk) |
| L3 Load drop | Weekly load ≥ 40% below the 28-day weekly average (possible detraining or missed sessions) |

### 4.3 Data-quality rule group
| Rule | Fires when | Effect |
|---|---|---|
| D1 No log | No Hooper entry by the daily cut-off (default 10:00 org time) | State **Insufficient data** for the day, labelled "Not logged yet" before the cut-off and "Not logged" after |
| D2 Log gap | No Hooper entry for 3 or more consecutive days | **Insufficient data** ("Log gap"), shown in the completion view |
| D3 Building baseline | Fewer than 10 entries in the last 28 days | **Insufficient data**, labelled "Building baseline" |

### 4.4 Guards
- **Load rules are suppressed** unless RPE completion for the athlete's Present sessions in the window is ≥ 70% **and** the window has ≥ 14 days of data; the board shows "Load incomplete" instead of a possibly misleading ratio.
- **Competition load is not captured** unless RPE is entered for those sessions; load rules say so when an athlete had competition sessions without RPE.
- **Baseline excludes days when the athlete was Unfit** and the first 3 days after return; a Medic can mark a longer "return period" that only affects display [PROPOSED, open O-R5].
- A state is calculated from **flags that are independent**: W1 and W3 are both wellness, so together they count as one group, not two.
- **Approved Leave** days produce no rules and no state (greyed).

### 4.5 Combination examples
| Fired | State |
|---|---|
| W2 only | Deviating |
| W1 + L1 | Marked deviation (two groups) |
| W1 on 3 consecutive days | Marked deviation (persistence) |
| L2 only | Deviating |
| D1 only | Insufficient data |
| Unfit | Chip only |

---

## 5. Screens (v1)

### 5.1 Morning board (primary)
Roster and sport filters, date picker (default today), "as of" time shown. Columns:
| Column | Content |
|---|---|
| Athlete | Name, photo if available, Rosters, age group for minors |
| Availability chip | Fit / Restricted + permitted activity / Unfit (status only, never cause for non-clinical roles) |
| Status vs baseline | In range / Deviating / Marked deviation / Insufficient data, with the **reason summary** on hover or tap |
| Hooper today | Index and delta vs baseline, four item dots |
| Trend | 14-day Hooper sparkline |
| Load | This week vs usual (bar), ACWR chip if load is complete |
| Flags | Reported pain/illness chip, uniform entries, late entries |
| Today | Sessions scheduled today (so the coach knows who matters now) |

- **Sort** default: Marked deviation first, then Deviating, then Insufficient data, then In range; Unfit athletes grouped separately at the bottom with their chip.
- **Athletes in several Rosters** appear in each; **athletes in no Roster** appear under "No roster".
- **Top strip** per Roster: counts per state, completion today (`logged / expected`), load distribution.

### 5.2 Athlete detail
Opened from the board: the **athlete monitoring page** from `wellness_and_rpe.md` §6 with the rule history (which rules fired on which days), availability history, attendance, and the **acknowledgement thread** (§6).

### 5.3 Pre-session view
From a session page, the same board filtered to the session's athletes, showing only: availability chip, readiness, Hooper today, and "Needs a word before the session". Opened by the supervisor; no extra data.

### 5.4 Admin view
Counts only: states by Roster, completion, athletes with a log gap, athletes with no account. Admin does not see item-level scores [consistent with `wellness_and_rpe.md` §7.1].

---

## 6. Acknowledgement [PROPOSED]
- A Coach, S&C Coach or Medic can **acknowledge** a Deviating/Marked deviation state for the day with an optional short note ("spoke to athlete, lighter session", "referred to physio").
- Acknowledged athletes drop out of the "Needs attention" count for that day but the **signal and its reasons stay visible**; acknowledgement never clears or alters the state.
- Each acknowledgement stores who, active role, time and note, and appears on the athlete page and in the audit. Notes follow the same visibility as the Hooper note (staff roles).
- Purpose: close the loop and give review and retention (Stage 8) evidence that signals were seen and acted on. It also counters "the system flagged him and nobody did anything".
- No auto-assignment and no escalation in v1; escalation belongs to the alerts module.

---

## 7. Who can see what [PROPOSED]
| Data | Athlete | Admin | Coach | S&C | Medic/Physio | Psych / Nutritionist | Team Manager | Guardian (minor) |
|---|---|---|---|---|---|---|---|---|
| Availability chip | Own | Yes | Status only | Status only | Full per Medical | Status only | Status only | Per Medical rules |
| Readiness state and reasons | **No label; sees own trend only** | Counts only | Yes | Yes | Yes | Yes | **No** | **No** |
| Rule detail with item scores | Own scores | No | Yes | Yes | Yes | Yes | No | No |
| Acknowledgements | No | Counts only | Yes | Yes | Yes | Yes | No | No |
| Completion view | No | Yes | Own rosters | Own rosters | No | No | No | No |

Athletes do not see the state label so that a "Marked deviation" tag is never the first thing a teenager reads about themselves; the athlete page shows their own trend and tells them their coach and medical team can see it (consistent with Wellness §7.1). Whether to show athletes a gentle summary is open (O-R6).

Team Manager continues to see **availability only** for competition and travel readiness [Medical doc A3, still to be confirmed there].

---

## 8. Calibration and success measures
- **Target:** on a typical day, roughly **10-15%** of athletes with complete data are Deviating or Marked deviation [PROPOSED aim, to calibrate]. Much higher means thresholds are too tight; near zero means they are useless.
- **Track:** share of flagged athletes who were acknowledged the same day; completion rate of the Hooper check; share of states driven by data-quality rather than data; how often a Marked deviation state precedes a new Medical case (as description, not as claimed predictive value).
- **Pilot** with a real centre before locking defaults. Keep rule changes versioned so past days are not rewritten (state history stores the rule version).
- Evidence caution [OUR]: the link between subjective-wellness drift or load ratios and injury is debated; the board is justified as a prompt for a conversation, not as an injury predictor, and the product should never claim otherwise.

---

## 9. Daily snapshot and history
- Board recalculates whenever an input changes (new Hooper entry, RPE, availability change).
- A **daily snapshot** of each athlete's state, reasons and rule version is stored at the cut-off (default 10:00) and again at end of day [PROPOSED], so a later review sees what was known on the morning, not the corrected data. Late entries update the live board but the morning snapshot keeps what the coach saw.
- Snapshots feed the Review/Retention pack (Stage 8) and the later alerts module.

---

## 10. Hooks for the alerts module (next doc)
Events this module emits and does not itself act on:
- `readiness.state_changed` (to Marked deviation, from anything)
- `readiness.deviation_persisting` (Marked deviation for N days)
- `readiness.no_data_streak` (log gap)
- `availability.changed` (from Medical, passed through)
- `readiness.acknowledged`

Alert routing, recipients, channels, quiet hours and escalation are designed separately.

---

## 11. Edge cases
| Case | Behaviour |
|---|---|
| New athlete (< 10 entries) | "Building baseline", no rule evaluation |
| Athlete returns after injury | Unfit days and first 3 days after return excluded from baseline; Medic may extend the display window |
| Athlete only trains occasionally | Load rules suppressed if RPE incomplete or < 14 days data |
| Athlete in several programmes or sports | Load summed across all programmes the athlete is in |
| Competition sessions without RPE | Load rules say "competition load not recorded" |
| Athlete on Approved Leave | Grey, no state, no rules |
| Athlete not in any Roster | Appears under "No roster" |
| Athlete logs after the cut-off | Live state updates, morning snapshot unchanged |
| Hooper entry by staff on behalf | Included, flagged `staff_on_behalf` in data-confidence |
| Uniform item entries for several days | Flagged `uniform_entry`; rules still run but confidence shown lower |
| Athlete reports pain, declines to tell medical | Chip "Reported pain/illness", no case, no state change |
| Coach is also Medic | Sees what their active role allows; audit shows the role |
| Org pause day (closure) | No expectations, no D1/D2 fired, state not computed |
| Athlete deactivated | Past snapshots kept, name visible |
| Athlete turns 18 | No change |
| Rule set changed by Admin | Applies from the change date; past snapshots keep the old rule version |
| Sport with no meaningful load (e.g., some skill sports) | Load group can be switched off per Programme; wellness rules still run |

---

## 12. Requirement trace
| Journey requirement | Covered by |
|---|---|
| JR-7 per-athlete diary used at review | Daily snapshots, athlete detail, acknowledgements (§6, §9) |
| JR-8 availability separate from wellness | Principle 1; two separate elements (§1, §3) |
| JR-10 cross-discipline flagging without leaking detail | §7 visibility, chip only for non-clinical roles |
| JR-11, JR-12 review and retention evidence | §9 snapshots feed the Review pack (Stage 8) |

---

## 13. Open points
| # | Question | Proposed answer |
|---|---|---|
| O-R1 | State names and count: In range / Deviating / Marked deviation / Insufficient data, with availability chip separate | **RESOLVED (PM 2026-11-25): names accepted** |
| O-R2 | Thresholds (W1, W2, W3, L1-L3, 70% RPE completeness, 10:00 cut-off) | Defaults as placeholders; SME and pilot to calibrate |
| O-R3 | Are the Admin-configurable thresholds in v1, or fixed defaults first? | **RESOLVED (PM 2026-11-25): fixed thresholds in v1** |
| O-R4 | Acknowledgement: keep, and does it remove the athlete from the "Needs attention" count? | **RESOLVED (PM 2026-11-25): keep; removes from the count only, signal stays visible** |
| O-R5 | Baseline exclusion around injury: 3 days after return, Medic-extendable | As stated |
| O-R6 | Should athletes see any readiness summary of themselves? | **PM 2026-11-25: athlete sees only their own data.** Whether the In range/Deviating/Marked deviation label counts as their data is being confirmed; default stays no label |
| O-R7 | ACWR in the rules at all, given contested predictive value | Keep L2 as "load change" Deviating only; remove if SME objects |
| O-R8 | Coach visibility beyond own rosters (flat org visibility) | Allowed, audited |
| O-R9 | Per-sport rule sets (e.g., different cut-offs for endurance vs skill sports) | Later; single default set in v1 |
| O-R10 | Does the board also serve Team Manager (availability only)? | Availability-only list, no readiness |
| O-R11 | Export of snapshots for federation/SAI reporting | Later with reporting design |
| O-R12 | Daily cut-off time and org timezone handling for travelling athletes | 10:00 org time; athlete's local date for entry |

## 14. SME questions
1. What do you look at before a session to decide who to push, modify or rest? How do you do it today?
2. Which signs would you actually act on: wellness drop, load jump, missed logs, anything else?
3. How many athletes flagged per day is useful before it becomes noise?
4. Should coaches see why an athlete is flagged, or only the Medic?
5. What should happen when an athlete is flagged and the coach does nothing?
6. How do you handle athletes coming back from injury in monitoring?

## 15. Risks
1. **Over-reliance:** coaches treat "In range" as clearance or "Marked deviation" as a ban. Mitigation: advisory wording, availability stays Medical, acknowledgement notes show human judgement.
2. **Alert fatigue and false confidence:** thresholds uncalibrated. Mitigation: calibration target, pilot, versioned rules, SME input.
3. **Misuse in selection or discipline:** readiness used to drop athletes. Mitigation: no auto-selection, athletes cannot be disadvantaged by system rules, Team Manager and guardians excluded.
4. **Garbage in:** low Hooper completion or RPE completion makes rules meaningless. Mitigation: data-quality states, load suppression, completion view.
5. **Athlete behaviour change:** athletes who learn flags exist may game scores. Mitigation: athletes do not see state labels; scores never used in selection; SME input.
6. **Privacy:** reasons show Hooper items to coaches. Mitigation: same role gating and audit as Wellness §7.

## 16. Suggested next
1. PM confirms O-R1, O-R3, O-R4, O-R6.
2. **Alerts module**: routing, recipients, quiet hours, escalation, built on §10 events.
3. In parallel, settle Medical blockers B1-B3 with Shrey since the availability chip depends on them.
4. Prototype the morning board and athlete form for an SME pilot (O-W12).
