# AMS Standalone — Onboarding & RBAC (v0.1 draft, 2026-11-24)

Status: decisions captured from founder/PM; items marked **[PROPOSED]** are my recommendations awaiting confirmation. AMS is a standalone product; ATOM link comes later (keep an `external_id` on Person and Organization, nothing else ATOM-specific).

## 0. Target customer (PM, 2026-11-24)
Primary use case: a **High Performance Centre** developing professional athletes across **multiple sports, all age groups and both genders**. So sport, age group and gender are first-class dimensions for athletes, batches and programs, and minors are a core population, not an edge case.

## 1. Locked decisions (from PM)

1. **Top-down provisioning.** Master Admin onboards an Organization Admin. Admin builds their own ecosystem: add/invite athletes, coaches and all other user types.
2. **Bulk import is the main onboarding path** for athletes and staff, keyed on the user's official email id. Once added, the user logs in with that same id.
3. **No certificates / experience / credential capture.** Org-based product; the org vouches for its people.
4. **Minor/guardian check stays** in onboarding.
5. **RBAC is fixed.** No custom roles. Each role is a defined persona with its own dashboard (Medic, Physio, Psychologist, Biomechanist, Team Manager, Coach, etc.).
6. **No partial-permission mixing.** A user may hold multiple roles (e.g. Medic + Coach); each role is a complete, separate dashboard and the user switches between them. Permissions are never merged or blended.

7. **Email is the only login identity** (2026-11-24 round 2). No phone login, so the later ATOM integration maps cleanly on email. Phone is a **non-mandatory** profile field for athletes.
8. **Athlete core fields:** name, email, gender, DOB (age derived), sport; phone optional.
9. **Coach is one role among the catalog**, not a base type.
10. **Org visibility is flat:** any role in an org can see all athletes in that org and perform its own role's tasks. Task design per role comes later.
11. **Same email may belong to two organizations** (kept as a known risk, see O3; going ahead regardless).
12. **Minors get full access to their own dashboard** (no limited view). Guardian consent still gates activation.
13. **Organization hierarchy tree UI: skipped / out of scope** (PM decision 2026-11-24). Org structure stays flat: Organization with optional Centres; no visual tree.

## 2. Model

- **Account** = login (email). One per person.
- **Person** = profile (name, gender, DOB, address, photo). No credentials fields.
- **Organization** = tenant. Created by Master Admin. **[PROPOSED]** optional Centres inside it for scope; no deeper hierarchy in v1.
- **Role assignment** = (Person, Organization, Role from fixed catalog, Scope, Status).
- Role is chosen from the catalog, never authored. Scope (org-wide / centre / assigned teams) is data, not a new role.

### Role catalog (confirm the "etc.")
| Level | Roles |
|---|---|
| Platform | Master Admin |
| Org | Admin |
| Staff | Coach, Team Manager, Medic (Doctor), Physiotherapist, Psychologist, Biomechanist, Nutritionist, S&C Coach, Performance Analyst |
| Participants | Athlete, Guardian |

Staff roles are all assigned to a Person by the Admin. "Coach" is just one role in the catalog, not a parent type: a person can hold Coach, Medic, or both.

### Role switching
- Login lands on the user's last-used role (first login: highest-priority role).
- Header switcher lists only the roles the user holds. Switching reloads the whole dashboard; no cross-role data on one screen.
- Every action is audited with the active role.

## 3. Onboarding flows

### 3.1 Master Admin → Admin
Master Admin creates Organization (name, type, sports) and enters the Admin's official email → invite email → Admin sets up login → lands on first-run checklist (add centres [optional], import staff, import athletes, create first program). **[PROPOSED]** an org may have several Admins; Master Admin creates the first, an Admin may add more.

### 3.2 Admin → staff (bulk import)
CSV/XLSX template: `name, official_email, phone, gender, DOB, roles (one or more from catalog), centre (optional)`.
- Validate: email format + uniqueness within org, role values in catalog, required fields.
- Preview with per-row errors; commit only valid rows (downloadable error file for the rest).
- On commit: account created in `Invited` state, email sent → first login = email OTP, then set password **[PROPOSED]** → `Active`.
- Admin can also add one user manually, resend invite, change roles, deactivate.

### 3.3 Admin → athletes (bulk import)
Template columns: `name, email (required), gender, DOB, sport, phone (optional), team/batch (optional)`; plus `guardian_name, guardian_email, guardian_phone(optional)` required when age < 18.
- Email is the login, so **every athlete needs a unique email, including minors**. A minor's email must differ from the guardian's (one email = one account). If a minor has none, the guardian may supply one they create for the child (O2).
- Row-level rule: **DOB < 18 and no guardian contact → row rejected.**
- Minor: guardian invited first, gives consent (who, when, how recorded); only then does the athlete record go `Active`. Minor logs in only after consent, then has **full access to their own dashboard**; the Guardian dashboard is read-mostly (O9).
- Adult athlete: normal invite → `Active`.
- Turning 18: guardian access ends unless the athlete re-grants it. **[PROPOSED]**

### 3.4 Statuses
`Invited → Active → Deactivated`; minors add `Awaiting Guardian Consent` before Active. Import rows: `Valid / Error / Duplicate`.

## 4. RBAC — fixed role matrix (to be filled per module)

Principle: each role sees only what its job needs; permissions are fixed per role.

| Data | Admin | Coach | Medic/Physio | Psychologist | Biomechanist | Team Mgr | Athlete | Guardian |
|---|---|---|---|---|---|---|---|---|
| Athlete profile | Full | Assigned athletes | Org/centre athletes | Org/centre athletes | Org/centre athletes | Assigned | Own | Child |
| Training programs/attendance | Full | Create/manage **own programs** + mark own sessions; org-wide attendance **read** | Read | Read | Read | Logistics | Own | Child (read) |
| Medical/injury | Availability status only **[PROPOSED]** | Fit/Restricted/Unfit only | Full | None | None | None | Own | Child |
| Psych notes | None **[PROPOSED]** | None | None | Full | None | None | Own summary only | Per policy |
| Biomechanics | Read | Summary | Read | None | Full | None | Own | Child |
| Import / user mgmt | Full | None | None | None | None | None | None | None |

Medical and psychological content is the highest-risk area: coaches see availability only; psych notes are not visible even to Admin by default. Needs PM sign-off.

## 5. Build phasing implication
Each role has its own dashboard, so cost scales with role count. **[PROPOSED]** ship dashboards in the order the underlying module ships: Admin, Coach, Athlete, Guardian first; Medic/Physio with Health & Injury; Psychologist and Biomechanist with Assessments; Team Manager with Squads/Selection. Roles can be assigned only once their dashboard exists (otherwise a user lands in an empty shell).

## 6. Open points
- ~~O1~~ Resolved 2026-11-24: catalog = the 12 roles in section 2 (PM's list + Nutritionist, S&C Coach, Performance Analyst).
- ~~O2~~ Resolved: Admin enters an email for every athlete incl. minors; the minor gives consent at login. **Risk flagged to PM:** under India's DPDP Act, consent for a child's data must come from the parent/guardian, so a minor self-consenting may not be legally valid. Recommended minimum: keep guardian name+email on the minor's row and record guardian consent too (or have the minor's login consent step also notify/confirm the guardian). PM to confirm with legal.
- O3. **Dicey, accepted:** same email in 2 orgs. **[PROPOSED]** safeguards: account = login only, profile data stored per organization membership so org B never sees org A's data; org B's import sends an invite the person must accept (no silent attach); deactivation in one org never touches the other; org switcher appears only when 2+ memberships; import must not reveal that an email already exists elsewhere.
- ~~O4~~ Resolved: Coach is one role among the catalog.
- ~~O5~~ Resolved: org-wide athlete visibility for all roles. Confirm this covers the athlete list/profile only, while medical and psych content stays restricted per matrix in section 4.
- ~~O6~~ Resolved: minors get full dashboard access.
- ~~O7~~ Resolved: deactivation preserves history; **Admin owns a departed coach's athletes and programs** until reassigned.
- ~~O8~~ Dropped: hierarchy tree skipped.
- O9 (PARKED by PM 2026-11-24). Guardian role after a minor gets full access: consent + read view of the child, or more? What happens at age 18?
