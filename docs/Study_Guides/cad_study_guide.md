# ServiceNow CAD Study Guide

This Markdown document combines the full CAD Study Guide, Last-Minute Drill Sheet, execution diagrams, exam traps, and hands-on drills. It is designed for notes and offline review.

---

## Table of Contents

1. [CAD Overview](#cad-overview)
2. [Domain 1: Designing & Creating an Application](#domain-1-designing--creating-an-application)
3. [Domain 2: Application User Interface](#domain-2-application-user-interface)
4. [Domain 3: Security & Restricting Access](#domain-3-security--restricting-access)
5. [Domain 4: Application Automation](#domain-4-application-automation)
6. [Domain 5: Working with External Data](#domain-5-working-with-external-data)
7. [Domain 6: Managing Applications](#domain-6-managing-applications)
8. [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)
9. [Execution Flow & ACL Diagram](#execution-flow--acl-diagram)
10. [Hands-On Drill Checklist](#hands-on-drill-checklist)
11. [Mental Checklist for Exam Decisions](#mental-checklist-for-exam-decisions)
12. [7–10 Day Prep Plan](#7-10-day-prep-plan)

---

## CAD Overview

- Total Questions: ~60
- Time: 90 minutes
- Passing: 70%
- Focus: Conceptual understanding + hands-on practice

---

## Domain 1: Designing & Creating an Application (20%)

**Key Concepts:**

- Application Fit: Scoped vs Global
- Data Model: Tables, dictionary entries, field types, relationships
- Modules: Navigation, list, form
- Application Scope: Cross-scope access, artifact protection

**Table & Field Prefixes — Know These Cold:**

| Prefix | Where it comes from                              | Example                  |
| ------ | ------------------------------------------------ | ------------------------ |
| `x_`   | Tables/fields created in a **scoped app**        | `x_myco_myapp_mytable`   |
| `u_`   | Custom fields/tables created in **global scope** | `u_custom_field`         |
| `sys_` | **ServiceNow's own** platform tables             | `sys_user`, `sys_script` |

> ⚠️ **Exam Trap:** `u_` is for global customizations, `x_` is for scoped apps. Don't mix these up!

**Application Scope — Key Rules:**

- Every artifact created while a scoped app is selected in the **App Picker** belongs to that scope
- The App Picker is in the top banner of the ServiceNow UI and Studio
- **Studio** is the recommended place to build scoped apps — it locks your scope and reduces accidental global artifact creation
- If you forget to switch the App Picker, new artifacts go to the wrong scope!

**How to verify an artifact's scope:** Open the record and check the **Application** field — it tells you exactly which scope owns it.

**Cross-Scope Access:**

| Setting                       | What it means                                                                            |
| ----------------------------- | ---------------------------------------------------------------------------------------- |
| "This application scope only" | Only code in the same scope can call this resource                                       |
| "All application scopes"      | Any scope (including global) can call this resource                                      |
| Caller Restriction            | The called resource decides — check the Cross-Scope Access table (`sys_scope_privilege`) |

**Exam Traps:**

- Confusing cross-scope Script Includes
- Table vs field restrictions in scoped apps
- Creating artifacts in the wrong scope by forgetting to check the App Picker
- `u_` vs `x_` prefix confusion

**Hands-On Drill:**

- Build a small scoped app (2 tables, 1 extended, 1 custom)
- Add modules for list & form view
- Experiment with cross-scope calls
- Check the Application field on artifacts to verify scope ownership

---

## Domain 2: Application User Interface (20%)

**Key Concepts:**

- Forms & Views: Add/remove fields, layout, sections
- Client Scripts: UI Policy, `onLoad`, `onChange`, `onSubmit`, `g_form`
- Server Scripts: BRs (Before, After, Async), Script Includes, GlideRecord
- Record Producers as UI

**Client Script Types — Know All Four:**

| Type         | When it fires                            | Common use case                                |
| ------------ | ---------------------------------------- | ---------------------------------------------- |
| `onLoad`     | When the form first loads                | Pre-populate fields, set initial visibility    |
| `onChange`   | When a specific field's value changes    | React to field changes, show/hide other fields |
| `onSubmit`   | When user clicks Save/Submit             | Validate before save, prevent bad data         |
| `onCellEdit` | When a cell is edited in a **list view** | Inline list editing logic                      |

> ⚠️ **Exam Trap:** `onCellEdit` is for **list view only**, not forms. Don't confuse with `onChange`.

**UI Policy vs UI Policy Action:**

- **UI Policy** = the _parent_ — holds the **condition** (e.g. `Priority = High`) and "On Load" setting
- **UI Policy Action** = the _child_ — defines **what happens to a specific field** when the condition is true

Each UI Policy Action controls one field and sets:

- **Mandatory** — true, false, or leave alone
- **Visible** — true, false, or leave alone
- **Read Only** — true, false, or leave alone

One UI Policy can have multiple UI Policy Actions (one per field). In Studio they appear grouped under the UI Policy, so you may have used them without knowing the name!

> ⚠️ **Exam Trap:** Always check "On Load" if the condition could already be true when the form opens. Without it, the policy only fires on field changes during the session.

**GlideRecord Best Practices:**

```javascript
// CORRECT pattern — always follow this order
var gr = new GlideRecord("incident");
gr.addQuery("priority", "1");
gr.query(); // ← NEVER forget this!
while (gr.next()) {
  // ← while, not if (unless you only want 1 record)
  gs.log(gr.getValue("number")); // ← getValue() not gr.number
}
```

| Mistake                                            | Why it's wrong                                                         |
| -------------------------------------------------- | ---------------------------------------------------------------------- |
| Using `if` instead of `while`                      | Only processes the first matching record                               |
| Forgetting `.query()`                              | The database query never executes                                      |
| Using `gr.field` instead of `gr.getValue('field')` | Returns a GlideElement object, not a plain string — causes subtle bugs |
| Using `gr.setValue()` to read                      | `setValue()` is for writing, `getValue()` is for reading               |

> 🧠 **Memory Tip:** Read with `getValue()`, write with `setValue()`. Always.

**Script Includes:**

- Server-side reusable functions — call them from Business Rules, other Script Includes, etc.
- Set **"Accessible from"** to control cross-scope access
- Can be made **client-callable** (for use with GlideAjax)

**GlideAjax vs UI Action — When to use each:**

| Scenario                                                                | Use                                            |
| ----------------------------------------------------------------------- | ---------------------------------------------- |
| User clicks a **button** on a form → run server logic                   | **UI Action** (Form button checked)            |
| Field changes → silently call server logic → update form without reload | **GlideAjax** + client-callable Script Include |

> ⚠️ **Exam Trap:** Business Rules fire on **database events** (insert/update/delete), NOT button clicks. Use UI Actions for buttons.

**Exam Traps:**

- UI Policy does not enforce imports (use Data Policy instead)
- Before BR does not need `current.update()`
- Async vs After BR timing
- `onCellEdit` is list view only

**Hands-On Drill:**

- Create a form with 3 fields
- Add UI Policy & UI Policy Actions
- Add Before, After, Async BRs affecting fields & related records
- Create Record Producer
- Test GlideAjax with a client-callable Script Include

---

## Domain 3: Security & Restricting Access (20%)

**Key Concepts:**

- ACLs: Table → Record → Field (broad to specific)
- Role → Condition → Script evaluation order
- Application & Module access
- GlideSystem security methods
- Application Scope protections

**ACL Evaluation Order — Broad to Specific:**

| Level      | Controls                                       | If blocked...                                  |
| ---------- | ---------------------------------------------- | ---------------------------------------------- |
| **Table**  | Can user access this table at all? (list view) | Stops here — record & field ACLs not evaluated |
| **Record** | Can user access this specific record?          | Stops here — field ACLs not evaluated          |
| **Field**  | Can user see/edit this specific field?         | Field is hidden or read-only                   |

> 🧠 **Memory Tip:** "Broad to specific — Table → Record → Field. Fail any gate, you're blocked."

**Diagnosing ACL Issues:**

| Symptom                                    | Likely cause              |
| ------------------------------------------ | ------------------------- |
| User can't see the list at all             | Table-level ACL blocking  |
| User sees the list but can't open a record | Record-level ACL blocking |
| User opens a record but a field is missing | Field-level ACL blocking  |

**Key Roles:**

| Role             | What it does                                                                                                                 |
| ---------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| `admin`          | Full platform access, **bypasses ACLs entirely**                                                                             |
| `security_admin` | Manages ACL records and security settings — required when High Security plugin is enabled. Does NOT grant full admin access. |
| `itil`           | Standard service desk role — incidents, problems, changes                                                                    |

> ⚠️ **Exam Trap:** `security_admin` sounds powerful but doesn't bypass ACLs like `admin` does. Don't confuse them!

**Exam Traps:**

- Table deny stops field ACL evaluation
- Client vs server misconceptions
- `security_admin` vs `admin` role confusion

**Hands-On Drill:**

- 2 table ACLs (read/write), 1 field ACL with script
- Test with different roles & users
- Observe how table deny blocks record/field evaluation

---

## Domain 4: Application Automation (20%)

**Key Concepts:**

- Workflows & Flow Designer
- Application Properties
- Events, Scheduled Jobs, Utility Script Includes
- Email notifications

**Business Rule Timing — Full Reference:**

| Type           | When it runs                       | Can modify `current`? | Blocks user? | Common use                               |
| -------------- | ---------------------------------- | --------------------- | ------------ | ---------------------------------------- |
| **Before BR**  | Before DB save                     | ✅ Yes                | ✅ Yes       | Modify/validate field values before save |
| **After BR**   | After DB save                      | ⚠️ Limited            | ✅ Yes       | Update related records                   |
| **Async BR**   | After DB save, **separate thread** | ❌ No                 | ❌ No        | Notifications, integrations, heavy tasks |
| **Display BR** | When form loads for display        | ✅ via `g_scratchpad` | ✅ Yes       | Pass server data to client scripts       |

> 🧠 **Memory Tip:** "Before = shape it. After = react to it. Async = fire and forget. Display = feed the form."

> ⚠️ **Exam Trap:** Async BRs do NOT block the user — they run in a background thread. Never use Async when you need the result to appear immediately on the form.

**Scheduled Jobs vs Async BRs:**

|                  | Scheduled Job                         | Async BR                                |
| ---------------- | ------------------------------------- | --------------------------------------- |
| **Triggered by** | A time schedule (daily, hourly, etc.) | A database event (insert/update/delete) |
| **Use when**     | You need logic to run on a timer      | You need background logic after a save  |

**Exam Traps:**

- Scheduled jobs vs Async BRs confusion
- Flow designer execution timing
- Using After BR when Async would be more appropriate (performance)

**Hands-On Drill:**

- Scheduled job updating a field daily
- Flow on record insert
- Event triggered via BR or Script Include

---

## Domain 5: Working with External Data (10%)

**Key Concepts:**

- Import Sets (CSV, Excel)
- Transform Maps & Field Maps
- Data Policy enforcement on imports
- REST API integrations

**UI Policy vs Data Policy — Critical Distinction:**

|                       | UI Policy                                       | Data Policy                               |
| --------------------- | ----------------------------------------------- | ----------------------------------------- |
| **Runs on**           | Client (browser)                                | Server (database)                         |
| **Enforces imports?** | ❌ No                                           | ✅ Yes                                    |
| **Controls**          | Field visibility, mandatory, read-only on forms | Mandatory and read-only at the data level |

> ⚠️ **Exam Trap:** UI Policies do NOT enforce imports. If you need mandatory/read-only to apply to imported data, use Data Policy.

**Exam Traps:**

- Forgetting "Run business rules" checkbox on Transform Maps
- UI Policies do not enforce imports

**Hands-On Drill:**

- Import CSV, map fields, optionally script transform
- Test REST GET/POST integration

---

## Domain 6: Managing Applications (10%)

**Key Concepts:**

- Install/download apps
- Delegated Development & code reviews
- ServiceNow Git integration

**Update Sets — Key Facts:**

- Capture **configuration changes** (scripts, forms, workflows) — NOT data (records, users, incidents)
- Changes go to the **Default Update Set** if you don't create a custom one
- Migration flow: **Dev → Test → Production**
- Always **Preview** before **Commit** in the target instance to catch conflicts

> ⚠️ **Exam Trap:** Update Sets capture config, not data. A new incident record will NOT be in your Update Set.

**Update Set vs App Repository:**

|                    | Update Set                   | App Repository            |
| ------------------ | ---------------------------- | ------------------------- |
| **Best for**       | Legacy/global customizations | Scoped apps (preferred)   |
| **Source control** | Manual export/import         | Git integration supported |

**Exam Traps:**

- Update Set vs App Repository confusion
- Delegated permissions misunderstood
- Thinking Update Sets capture data records

**Hands-On Drill:**

- Push/pull app via Git
- Create delegated developer account

---

## Quick Reference Cheat Sheet

**BR Timing:**

| Type        | Runs                         | Can Abort | Notes                             |
| ----------- | ---------------------------- | --------- | --------------------------------- |
| Before BR   | Before DB                    | ✅        | Do not call `current.update()`    |
| After BR    | After DB commit              | ❌        | Update related records            |
| Async BR    | After DB commit (background) | ❌        | Non-blocking, fire and forget     |
| Data Policy | Before DB                    | ✅        | Enforces imports                  |
| UI Policy   | Client only                  | ❌        | Form only, cannot enforce imports |

**Client vs Server APIs:**

| Object             | Scope           | Notes                                                       |
| ------------------ | --------------- | ----------------------------------------------------------- |
| `g_form`           | Client          | Only on forms — get/set field values, visibility, mandatory |
| `current`          | Server          | GlideRecord in BR/Script Include                            |
| `GlideRecord`      | Server          | Query/update DB — always use `getValue()` / `setValue()`    |
| `GlideAjax`        | Client → Server | Call server-side Script Include from a Client Script        |
| `gs` (GlideSystem) | Server          | Logging, user info, system utilities                        |
| `g_scratchpad`     | Bridge          | Pass data from Display BR (server) to Client Script         |
| Script Include     | Server          | Reusable; can be made client-callable for GlideAjax         |

**ACL Evaluation:**

- Table → Record → Field (broad to specific)
- Fail any level = access denied, evaluation stops
- `admin` role bypasses all ACLs
- `security_admin` manages ACL records — does NOT bypass them

**Prefixes:**

| Prefix | Meaning                         |
| ------ | ------------------------------- |
| `x_`   | Scoped app table/field          |
| `u_`   | Global scope custom table/field |
| `sys_` | ServiceNow platform table       |

**Cross-Scope / Scoped App:**

- Set "Accessible from" = "All application scopes" to allow cross-scope calls
- UI Policies do not cross scope
- Check the **Application** field on any artifact to verify its scope

**Transform Map / Import:**

- "Run business rules" checkbox controls BR execution on import
- Field Maps: source → target
- Data Policy enforces mandatory/readonly on imports (UI Policy does NOT)

**Deployment / Source Control:**

- App Repository → preferred for scoped apps
- Update Set → legacy, captures config not data
- Delegated Development → code reviews
- Git → commit/pull/push

**Common Exam Traps — Quick List:**

| Symptom / Scenario                          | Answer                                              |
| ------------------------------------------- | --------------------------------------------------- |
| Form is slow after save                     | Check After vs Async BR — use Async for heavy tasks |
| Field not saving                            | Check dictionary read-only                          |
| Client Script needs DB query                | Use GlideAjax, not GlideRecord directly             |
| Mandatory field not enforced on import      | Use Data Policy, not UI Policy                      |
| Cross-scope call failing                    | Check "Accessible from" / Cross-Scope Access table  |
| User sees list but can't open record        | Record-level ACL blocking                           |
| User can't see list at all                  | Table-level ACL blocking                            |
| Need button on form to trigger server logic | UI Action (not Business Rule)                       |
| Need field change to silently call server   | GlideAjax + client-callable Script Include          |
| Move config between instances               | Update Set or App Repository                        |
| Wrong scope on new artifact                 | Forgot to check App Picker before creating          |

---

## Execution Flow & ACL Diagram (ASCII)

```
Record Save Flow:

Client Form Submit
       |
       v
  [Before BR] ---> can modify `current`, can abort
       |
       v
   Database Commit
       |
       v
   [After BR] ---> update related records, user waits
       |
       v
   [Async BR] ---> background thread, user continues

ACL Evaluation Flow:

Access Request
       |
       v
  Table ACL? --> FAIL --> Access Denied (stops here)
       |
      PASS
       v
  Record ACL? --> FAIL --> Access Denied (stops here)
       |
      PASS
       v
  Field ACL? --> FAIL --> Field hidden/read-only
       |
      PASS
       v
  Access Granted

Rules:
- Table deny stops ALL further evaluation
- Must pass ALL levels for full access
- admin role skips all ACL evaluation
```

---

## Hands-On Drill Checklist

- [ ] Scoped app with 2 tables (1 extended, 1 custom) — verify `x_` prefix on tables
- [ ] Check App Picker before creating each artifact — verify Application field on records
- [ ] Create modules (list/form)
- [ ] Client Scripts (onLoad, onChange, onSubmit, onCellEdit)
- [ ] UI Policies + UI Policy Actions (with and without "On Load")
- [ ] Before, After, Async BRs — observe timing differences
- [ ] Display BR with `g_scratchpad` passing data to Client Script
- [ ] Data Policies for mandatory/readonly — test on import
- [ ] ACLs (table & field, scripts, roles) — test table deny blocking record/field evaluation
- [ ] Transform Map + Import Set + optional script
- [ ] REST integration test
- [ ] Scheduled Job / Event trigger
- [ ] Record Producer
- [ ] Script Include — test cross-scope access with "Accessible from" settings
- [ ] GlideAjax with a client-callable Script Include
- [ ] UI Action with Form button triggering server-side script
- [ ] Git integration & Delegated Development test
- [ ] Update Set — export config, import to another instance, preview before commit

---

## Mental Checklist for Exam Decisions

- **ACL issue?** → Table deny → stop; Record deny → stop; Field deny → restrict further
- **BR Timing?** → Before → DB → After → Async (background)
- **Client vs Server?** → `g_form` = client; `current`/`GlideRecord`/`gs` = server
- **Cross-scope failure?** → Check "Accessible from" / Cross-Scope Access table
- **Button on form?** → UI Action (not Business Rule)
- **Silent server call from client?** → GlideAjax + client-callable Script Include
- **Mandatory on import?** → Data Policy (not UI Policy)
- **Move config between instances?** → Update Set or App Repository
- **Wrong scope on artifact?** → Check App Picker, verify Application field on record
- **Read a field value in GlideRecord?** → `getValue()` not `gr.field`
- **Multi-select exam question?** → Commit first instinct, don't second-guess

---

## 7–10 Day Prep Plan

**Days 1–3:** Build small scoped app, BRs, ACLs, UI Policies, Data Policies. Pay attention to App Picker and scope prefixes.

**Days 4–5:** Cross-scope, GlideAjax, UI Actions, Flow Designer, Scheduled Jobs, REST

**Days 6–7:** Timed practice exams, commit instinct answers

**Day 8:** Quick-reference drills (ACL levels, BR timing, prefix table, client vs server APIs)

**Days 9–10:** Light review, mental walkthroughs, confidence reinforcement
