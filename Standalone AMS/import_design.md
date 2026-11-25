# AMS Standalone — Bulk Import Design (v0.1 draft, 2026-11-24)

Status: builds on `onboarding_and_rbac.md` (locked decisions 1-13). Items marked **[PROPOSED]** await PM confirmation. Import is the **main onboarding path**, so its failure modes are the product's first impression.

## 1. Scope
- Two import types: **Staff** and **Athletes**. Separate templates, separate screens. (Guardians are never imported directly; they are created from athlete rows, see 4.)
- Format: XLSX and CSV (UTF-8). Template downloadable from the screen, with a read-only "Instructions" sheet and an "Allowed values" sheet.
- Who: **Admin only**. Master Admin can act as an Admin of any org (audited as such).
- **[PROPOSED]** Limits: 2,000 rows per file, 5 MB. Larger orgs split files. Revisit after first customer.
- Insert-only in v1: import creates people; it never edits existing ones (see 5).

## 2. The flow (same for both types)
1. **Download template** (versioned; file carries a template-version cell, old versions rejected with a clear message).
2. **Upload** file.
3. **Parse + validate** (async job with progress; runs in under a minute for 2,000 rows).
4. **Preview**: summary tiles `Valid / Error / Duplicate / Warning` + row table with filter by state. Nothing is written yet.
5. **Commit**: **[PROPOSED]** partial commit, valid rows only; the user explicitly chooses "Import N valid rows" (not silent). Blocking all-or-nothing would punish a 500-row file for one typo.
6. **Result screen + invites** sent per rules in section 6.
7. **Error report**: download of the original file's failed rows + `error_code` and `error_message` columns, same column layout, so it can be fixed in place and re-uploaded directly.

Every import is an **Import Job** record: who, when, role, file name + hash, counts per state, status. Visible in an Import History list. Re-uploading a file with a hash already committed shows a warning ("this exact file was imported on <date>").

## 3. Staff template
`name*, official_email*, phone, gender*, dob, roles*, centre`

| Column | Rule |
|---|---|
| name | required, 2-100 chars, trimmed |
| official_email | required, valid format, lowercased + trimmed; unique in file and in org |
| phone | optional, E.164 or 10-digit Indian mobile, normalised |
| gender | required, from allowed list (see O-I1) |
| dob | optional for staff **[PROPOSED]** (not needed for any staff function); DD/MM/YYYY and true Excel dates both accepted |
| roles | required, one or more from the 12-role catalog, separated by `;` (e.g. `Coach;Medic`). Case-insensitive match. Master Admin / Admin / Athlete / Guardian **not allowed** here |
| centre | optional; must match an existing Centre name in the org, else error (no auto-create) |

Multi-role row = one person, several role assignments (honours decision 6). A role whose dashboard is not yet shipped is **rejected** with `ROLE_NOT_AVAILABLE` (per section 5 of the RBAC doc: no empty-shell logins).

## 4. Athlete template
`name*, email*, gender*, dob*, sport*, phone, team_batch, centre, guardian_name, guardian_email, guardian_phone`

| Column | Rule |
|---|---|
| email | required, unique in file and org; must not equal any guardian_email in the file |
| dob | required (age drives the minor rule); must be in the past, age 3-60 sanity range else Warning |
| sport | required; must be one of the org's configured sports (set at org creation). Multi-sport athletes: **[PROPOSED]** one primary sport in v1, second sport added in app |
| team_batch | optional; must already exist, else error (no auto-create) |
| guardian_* | **required when age < 18 on import date**; ignored (with Warning) for adults |

**Minor row rules**
- Age < 18 and no guardian name+email → **Error `GUARDIAN_REQUIRED`**.
- guardian_email = athlete email → **Error** (one email = one account).
- **Sibling case:** several rows may share one guardian_email. They resolve to **one Guardian account with multiple children**, not duplicates. Name mismatch between rows with the same guardian_email → Warning, first row wins.
- Minor row commits as `Awaiting Guardian Consent`; guardian invite goes out **first**; athlete invite is withheld until consent is recorded (decision 12 + O2).
- **Existing guardian email that is already an Athlete or staff account** → allowed (one person, multiple roles), no error.

## 5. Duplicates and cross-org handling
Order of checks per row: format → in-file duplicate → in-org duplicate → cross-org.

| Case | Result |
|---|---|
| Same email twice in the file | 2nd occurrence = Duplicate (first wins) |
| Email already in this org as the same role | Duplicate, skipped, listed in report |
| Staff import: email exists in org with different roles | **[PROPOSED]** Warning "existing user, add role?"; roles are added only if the Admin ticks "add new roles to existing users" on the preview. Default off |
| Athlete import: email exists in org as Athlete | Duplicate, skipped. No profile update in v1 |
| Email exists in **another org** (O3) | Row is **Valid**; the user sees nothing special. Result is an **invite the person must accept**, never a silent attach. The UI **must not reveal** that the email exists elsewhere (preview shows plain Valid) |

Edit-via-reimport (updating DOB, sport, etc.) is **out of v1**; corrections happen in-app per person. Flag as the first likely customer request.

## 6. Invites, states, and re-sending
Person states unchanged: `Invited → Active → Deactivated` (+ `Awaiting Guardian Consent` for minors).
- Invite email sent on commit, **batched via queue** (do not send 2,000 at once; sender reputation).
- Invite link expiry **[PROPOSED]** 7 days. Expired → state stays `Invited`, Admin sees "Expired" badge and can **bulk-resend** from the user list.
- First login = email OTP, then set password (as in 3.2 of the RBAC doc).
- **Bounce handling:** hard bounce flags the person `Invalid email` in the user list with a one-click "correct email and resend". This is the realistic failure for HPC rosters built from old spreadsheets.
- Shared mailboxes (`info@`, `admin@`, `coach@`) → Warning on the row, not blocked. **[PROPOSED]**
- Admin can **undo an import** within 24h only for people still in `Invited` (never logged in): removes them and logs it. After first login, deactivate instead.

## 7. Error catalogue (stable codes, one per failure, plain-language message with the offending value)
`REQUIRED_MISSING, EMAIL_INVALID, EMAIL_DUP_IN_FILE, EMAIL_DUP_IN_ORG, ROLE_UNKNOWN, ROLE_NOT_ALLOWED, ROLE_NOT_AVAILABLE, GENDER_INVALID, DOB_INVALID, DOB_FUTURE, SPORT_NOT_IN_ORG, CENTRE_NOT_FOUND, BATCH_NOT_FOUND, GUARDIAN_REQUIRED, GUARDIAN_EQUALS_ATHLETE, PHONE_INVALID, TEMPLATE_VERSION_OLD, FILE_TOO_LARGE`
Warnings (do not block): `DOB_AGE_UNUSUAL, SHARED_MAILBOX, GUARDIAN_IGNORED_ADULT, GUARDIAN_NAME_MISMATCH, ROLE_ADD_TO_EXISTING`.

## 8. Audit and privacy
- Import Job + per-row outcome retained; the uploaded file itself is **deleted after 30 days** **[PROPOSED]** (it contains minors' DOB and guardian contacts, so do not keep it indefinitely). The error report is generated on demand during that window.
- Row-level audit for each created person (created_by import job id, actor role).
- Only Admin sees Import History; export of the error file is logged.

## 9. Open points
- **O-I1. Gender list.** HPC is "both genders", but the allowed value set drives validation and category eligibility later. **[PROPOSED]** Male / Female / Other, with Male/Female used for gender-category logic. PM to confirm.
- **O-I2. Partial commit vs all-or-nothing.** Proposed partial commit with explicit confirmation. Confirm.
- **O-I3. Update-via-reimport.** Out of v1 (proposed). Confirm, since onboarding a real HPC will surface wrong-DOB fixes immediately.
- **O-I4. Auto-create of batches/centres/sports.** Proposed: never auto-create; error and fix. Cost: Admin must set up Centres/teams before importing. Alternative: a "create missing" toggle. Confirm.
- **O-I5. Roles whose dashboards are not shipped yet.** Proposed `ROLE_NOT_AVAILABLE` rejection. This means a Medic cannot be onboarded until Health & Injury ships. Is that acceptable for pilot customers, or should they be importable and parked as `Inactive until module live`?
- **O-I6. Minor without any email** (O2): the guardian creates one for the child. Where does the Admin put that in the file (athlete email = guardian-created address)? Proposed: just a normal email; no special flow.
- **O-I7. Invite expiry and OTP length** (7 days proposed) and whether Admin can set a custom welcome message. Low priority.

## 10. Suggested next
Role dashboards (Admin first, since its first-run checklist is where the import screen lives), or screen-level wireframe spec for the import flow (upload → preview → result), which is the highest-value single screen set.
