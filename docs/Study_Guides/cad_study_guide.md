# ServiceNow CAD Certification — Master Study Guide

> Deep Dives · Code Examples · Exam Traps · Practice Questions

---

## Table of Contents

1. [Exam Overview](#exam-overview)
2. [Domain 1: Designing & Creating an Application (20%)](#domain-1-designing--creating-an-application-20)
3. [Domain 2: Application User Interface (20%)](#domain-2-application-user-interface-20)
4. [Domain 3: Security & Restricting Access (20%)](#domain-3-security--restricting-access-20)
5. [Domain 4: Application Automation (20%)](#domain-4-application-automation-20)
6. [Domain 5: Working with External Data (10%)](#domain-5-working-with-external-data-10)
7. [Domain 6: Managing Applications (10%)](#domain-6-managing-applications-10)
8. [Glide API Quick Reference](#glide-api-quick-reference)
9. [Master Quick Reference](#master-quick-reference)
10. [All Exam Traps — Combined List](#all-exam-traps--combined-list)
11. [Mnemonics & Memory Tricks](#mnemonics--memory-tricks)
12. [7–10 Day Study Plan](#710-day-study-plan)

---

## Exam Overview

| Detail          | Info                                          |
| --------------- | --------------------------------------------- |
| Total Questions | ~60                                           |
| Time Allowed    | 90 minutes (~1.5 min/question)                |
| Passing Score   | 70% (42+ correct)                             |
| Focus           | Conceptual understanding + hands-on scripting |

> All six domains carry equal or near-equal weight. Do not neglect any domain — even Import Sets (10%) can swing a borderline result.

---

## Domain 1: Designing & Creating an Application (20%)

### Application Scope — Deep Dive

Every artifact you create in ServiceNow belongs to exactly one scope. The scope is determined by what is selected in the **App Picker** at the moment of creation — not where you navigate afterward.

| Prefix | When it appears                          | Example                  |
| ------ | ---------------------------------------- | ------------------------ |
| `x_`   | Tables/fields in a **scoped app**        | `x_myco_myapp_incident`  |
| `u_`   | Custom tables/fields in **global scope** | `u_custom_field`         |
| `sys_` | Native ServiceNow platform tables        | `sys_user`, `sys_script` |

> ⚠️ **EXAM TRAP:** `u_` is for **GLOBAL** customizations. `x_` is for **SCOPED** apps. Confusing them is the #1 Domain 1 trap.

**How to verify an artifact's scope:** Open the record and check the **Application** field — it shows exactly which scope owns it.

**Privately-scoped app default behavior:**

- **Read** → all scopes can read by default ✅
- **Write/Update/Delete** → restricted to owning scope only ✅

**Guided Application Creator (GAC)** → fastest/easiest way to create a new scoped app.

**Delegated Developer** → "admin" abilities within a specific scoped app only — NOT system admin. Cannot modify artifacts outside their assigned app.

### Cross-Scope Access — How it Really Works

When App A calls a Script Include in App B, ServiceNow checks the **"Accessible from"** field on the Script Include record.

| Setting                     | Meaning                                                   | Exam signal                   |
| --------------------------- | --------------------------------------------------------- | ----------------------------- |
| This application scope only | Only code in the same scope can call it                   | Default — restrictive         |
| All application scopes      | Any scope including global can call it                    | Required for shared utilities |
| Caller restriction          | Called resource controls access via `sys_scope_privilege` | Advanced — check the table    |

> 🧠 **MEMORY TIP:** To diagnose a cross-scope failure: open the Script Include record and look at "Accessible from". 9 times out of 10, that's the problem.

### Practice Questions — Domain 1

> 📋 **PRACTICE Q:** A developer creates a new table while the **Global** app is selected in the App Picker. The table prefix is `u_`. Later, the developer switches to a scoped app and creates a Script Include. Which scope does the Script Include belong to?

> ✅ **ANSWER:** The Script Include belongs to the **scoped app** — scope is determined at creation time by the App Picker. The earlier `u_` table stays in global scope.

---

> 📋 **PRACTICE Q:** A scoped app's Script Include has "Accessible from" set to "This application scope only". A Business Rule in a **different** scoped app tries to call it. What happens?

> ✅ **ANSWER:** The call **silently fails**. No error is thrown, but the Script Include does not execute. Fix by changing "Accessible from" to "All application scopes".

---

## Domain 2: Application User Interface (20%)

### Client Script Types — Full Breakdown

| Type         | When it fires                   | Key use case                 | Common trap                              |
| ------------ | ------------------------------- | ---------------------------- | ---------------------------------------- |
| `onLoad`     | Form first renders              | Pre-populate, set visibility | Runs even if user never touches the form |
| `onChange`   | A watched field's value changes | Show/hide dependent fields   | Does NOT run on form load                |
| `onSubmit`   | User clicks Save/Submit         | Last-chance validation       | Return `false` to cancel submit          |
| `onCellEdit` | Cell edited in **LIST VIEW**    | Inline list editing logic    | **Not for forms — list only!**           |

> ⚠️ **EXAM TRAP:** `onCellEdit` fires in **LIST VIEW only**. If an exam question describes a form field change, the answer is `onChange`, not `onCellEdit`.

> 💡 **Alert = JavaScript = Client Script** — `alert()` is a JS method, so it's client-side!

### GlideRecord — The Definitive Patterns

GlideRecord is the most heavily tested scripting topic. Memorize the correct pattern and the failure modes.

```javascript
// ✅ CORRECT — standard query pattern
var gr = new GlideRecord("incident");
gr.addQuery("priority", "1");
gr.addQuery("field", "!=", "value"); // not equal
gr.addEncodedQuery("active=true"); // encoded query string
gr.query(); // NEVER skip this
while (gr.next()) {
  // while, not if
  gs.log(gr.getValue("number")); // getValue(), not gr.number
  gs.log(gr.getDisplayValue("field")); // human-readable display value
  gs.log(gr.getUniqueValue()); // get sys_id of current record
}

// ✅ CORRECT — insert a record
var gr = new GlideRecord("incident");
gr.initialize();
gr.setValue("short_description", "Test");
gr.setValue("priority", "2");
gr.insert();

// ✅ CORRECT — update matching records
var gr = new GlideRecord("incident");
gr.addQuery("state", "1");
gr.query();
while (gr.next()) {
  gr.setValue("state", "2");
  gr.update(); // call update() inside the loop
}
```

**Additional GlideRecord methods:**

```javascript
gr.get("sys_id", "value"); // get single record by field
gr.getRowCount(); // count of results
gr.hasNext(); // check if more records exist
gr.deleteRecord(); // delete current record
gr.isNewRecord(); // is this a new record?
gr.isValidRecord(); // did query return a record?
```

| Mistake                           | Why it's wrong                                                       | Fix                                            |
| --------------------------------- | -------------------------------------------------------------------- | ---------------------------------------------- |
| Using `if` instead of `while`     | Processes only the first record                                      | Always use `while(gr.next())`                  |
| Skipping `.query()`               | Query never executes — `gr.next()` returns false immediately         | Always call `gr.query()`                       |
| Using `gr.field_name` directly    | Returns a GlideElement object, not a string — causes comparison bugs | Use `gr.getValue('field_name')`                |
| `current.update()` in a Before BR | Triggers another save event, causing an infinite loop                | Just set values; the platform handles the save |
| GlideRecord in a Client Script    | GlideRecord is server-side only — will throw ReferenceError          | Use GlideAjax instead                          |

> 🧠 **MEMORY TIP:** `getValue()` returns a **string** (DB value). `getDisplayValue()` returns the **human-readable label** (e.g., a person's name for a reference field). Read with `getValue()`, write with `setValue()`. Always.

### GlideAjax — Client-to-Server Communication

When a client script needs to run a server-side database query, use **GlideAjax**. It calls a client-callable Script Include asynchronously without reloading the form.

```javascript
// SERVER — Script Include (must extend AbstractAjaxProcessor)
var MyAjaxSI = Class.create();
MyAjaxSI.prototype = Object.extendsObject(AbstractAjaxProcessor, {
  getManagerName: function () {
    var userId = this.getParameter("sysparm_user_id");
    var gr = new GlideRecord("sys_user");
    if (gr.get(userId)) {
      return this.newItem("result").setAttribute(
        "manager",
        gr.getValue("manager.name"),
      );
    }
  },
  type: "MyAjaxSI",
});

// CLIENT — Client Script calling the Script Include
function onChange(control, oldValue, newValue, isLoading) {
  if (isLoading) return;
  var ga = new GlideAjax("MyAjaxSI");
  ga.addParam("sysparm_name", "getManagerName");
  ga.addParam("sysparm_user_id", newValue);
  ga.getXMLAnswer(function (answer) {
    // async callback
    g_form.setValue("manager", answer);
  });
}
```

**GlideAjax Flow:**

```
Client Script
  → GlideAjax (the messenger)
    → Script Include (extends AbstractAjaxProcessor)
      → GlideRecord (queries the DB)
  → GlideAjax callback
    → g_form.setValue() (updates the form)
```

> 🧠 **MEMORY TIP:** The 3-step GlideAjax pattern: **(1)** `new GlideAjax('ScriptIncludeName')`, **(2)** `addParam` for `sysparm_name` + data params, **(3)** `getXMLAnswer(callback)`. The Script Include **must** extend `AbstractAjaxProcessor`.

> 💡 Use **g_scratchpad** when you need server data on **form load** (no extra call). Use **GlideAjax** when you need server data **after user interaction**.

### UI Policy Deep Dive

A **UI Policy** is a parent record holding a condition. A **UI Policy Action** is a child record saying what happens to a specific field when that condition is true.

| Setting                | What it controls                        | Gotcha                                                                          |
| ---------------------- | --------------------------------------- | ------------------------------------------------------------------------------- |
| Condition              | When the policy activates               | If condition is already true on load, nothing fires unless "On Load" is checked |
| **On Load** (checkbox) | Also evaluate when the form first loads | Leave unchecked = policy only fires when fields change during the session       |
| Action: Mandatory      | `true` / `false` / leave alone          | "Leave alone" = this policy doesn't touch it                                    |
| Action: Visible        | `true` / `false` / leave alone          | Hidden field still submits its value                                            |
| Action: Read Only      | `true` / `false` / leave alone          | UI only — doesn't protect data on server                                        |
| **Reverse if False**   | Auto-reverses when condition is false   | No need for a second UI Policy!                                                 |

> ⚠️ **EXAM TRAP:** UI Policy controls the **browser form only**. It does not protect data from server-side scripts, imports, or REST API calls. Use **Data Policy** for server-side enforcement.

### UI Policy vs Client Script vs Data Policy

|                            | UI Policy        | Client Script                     | Data Policy                     |
| -------------------------- | ---------------- | --------------------------------- | ------------------------------- |
| **Runs on**                | Browser          | Browser                           | Server                          |
| **Can show/hide fields**   | ✅               | ✅ (`g_form.setVisible`)          | ❌                              |
| **Can make mandatory**     | ✅               | ✅ (`g_form.setMandatory`)        | ✅                              |
| **Can make read only**     | ✅               | ✅ (`g_form.setReadOnly`)         | ✅                              |
| **Bypassed by API/import** | ✅ Yes           | ✅ Yes                            | ❌ No                           |
| **Use when**               | Simple condition | Complex logic / needs server data | Must enforce via API/Import too |

> 💡 **Rule:** Simple condition using only form data → **UI Policy**. Complex logic or needs server data → **Client Script**. Must enforce via API/Import Sets too → **Data Policy**.

### ACL vs UI Policy (Read Only)

| Scenario                                      | Tool                                                            |
| --------------------------------------------- | --------------------------------------------------------------- |
| Hide data from certain roles entirely         | **ACL**                                                         |
| Make a field read only based on a role        | **Client Script** (`g_user.hasRole()` + `g_form.setReadOnly()`) |
| Make a field read only based on another field | **UI Policy**                                                   |
| Restrict write access at the data level       | **ACL**                                                         |

> ⚠️ **ACLs grant or deny access entirely** — they don't have a "read only" mode. Use Client Scripts for read-only behavior based on roles.

### Practice Questions — Domain 2

> 📋 **PRACTICE Q:** A developer wants a field to become mandatory when Priority is set to "High", **including when the form first loads** with Priority already set to High. What must be configured?

> ✅ **ANSWER:** Create a UI Policy with Condition: `Priority = High` **AND** check the **"On Load"** checkbox. Add a UI Policy Action targeting the field with `Mandatory = true`.

---

> 📋 **PRACTICE Q:** A client script needs to look up a user's department from the database when a field changes. What is the correct approach?

> ✅ **ANSWER:** Use **GlideAjax** with a client-callable Script Include (extending `AbstractAjaxProcessor`). GlideRecord cannot run in a client script — it is server-side only.

---

## Domain 3: Security & Restricting Access (20%)

### ACL Evaluation — The Full Mental Model

ACL evaluation flows strictly from **broad to specific**. Failing any level stops evaluation entirely.

```
Access Request comes in
       |
       v
  [TABLE ACL]  ------FAIL------>  Access Denied. STOP. No record/field ACLs checked.
       |
      PASS
       v
  [RECORD ACL] ------FAIL------>  Access Denied. STOP. No field ACLs checked.
       |
      PASS
       v
  [FIELD ACL]  ------FAIL------>  Field is hidden or read-only.
       |
      PASS
       v
  Access fully granted

  KEY RULE: admin role bypasses ALL levels. security_admin does NOT.
```

> 🧠 **MEMORY TIP:** "Broad to specific — Table → Record → Field. Fail any gate, you're blocked."

### ACL Diagnosis — Symptom to Cause

| Symptom                                        | Root Cause                                              | Where to look                                       |
| ---------------------------------------------- | ------------------------------------------------------- | --------------------------------------------------- |
| User cannot see the table in nav or list       | Table-level ACL blocking read                           | ACL record: `name = table_name`, `operation = read` |
| User sees list but can't open a record         | Record-level ACL blocking read                          | ACL record with a condition or script on the record |
| User opens record but a field is blank/missing | Field-level ACL blocking read                           | ACL record: `name = table_name.field_name`          |
| User can see a field but can't edit it         | Field-level ACL blocking write, or dictionary read-only | Check write ACL AND dictionary entry                |

### ACL Evaluation — Role → Condition → Script

Within a single ACL record, all three checks must pass in order: **Role → Condition → Script**.

| Check     | Fails when...                              | Practical note                                      |
| --------- | ------------------------------------------ | --------------------------------------------------- |
| Role      | User does not have the required role       | Leave blank to allow any authenticated user         |
| Condition | Record does not match the condition filter | Example: only allow if `assigned_to = current user` |
| Script    | Script returns `false`                     | Most powerful — can implement any logic             |

> ⚠️ **EXAM TRAP:** `admin` bypasses **ALL** ACL evaluation entirely. `security_admin` only allows managing ACL records — it does **not** bypass them. Requires `security_admin` role to create/modify ACLs.

### Practice Questions — Domain 3

> 📋 **PRACTICE Q:** A user with the `itil` role reports they can see a list of incidents but clicking any record shows an error. The `admin` role works fine. What is the most likely cause?

> ✅ **ANSWER:** A **record-level ACL** is blocking read access for the `itil` role. The table-level ACL is passing (they see the list), but a record-level ACL is denying access to individual records.

---

> 📋 **PRACTICE Q:** A new field is added to the incident table. Users with `itil` can open incident records but the new field is not visible. What should the admin check first?

> ✅ **ANSWER:** Check for a **field-level ACL** on `incident.new_field_name`. Also verify the field is present on the **Form Layout** — a missing field from the layout looks identical to a field-level ACL from the user's perspective.

---

## Domain 4: Application Automation (20%)

### Business Rule Timing — Visual Flow

```
User clicks Save on a form
         |
         v
   [BEFORE BR]  <── Can read & modify 'current'. Can abort with current.setAbortAction(true).
         |          Do NOT call current.update() here — causes an infinite loop!
         v
   DATABASE WRITE  (record is committed to DB)
         |
         v
   [AFTER BR]   <── 'current' is effectively read-only here. Use to update RELATED records.
         |          User is still waiting. Keep this fast.
         v
   [ASYNC BR]   <── Runs in a background thread. User does NOT wait.
         |          Use for emails, integrations, heavy processing.
         v
   [DISPLAY BR] <── Fires when form LOADS for display (not on save).
                    Use g_scratchpad to pass data to client scripts.
```

**Full Execution Order (when a record is saved):**

```
1. UI Policy         (browser)
2. Client Script     (browser)
3. Before BR         (server)
4. DATABASE SAVE     ← record written here
5. After BR          (server)
6. Async BR          (background)
```

> 💡 **Browser first → Server second → Background last**

| Type       | Modify `current`?               | Blocks user?               | Best used for                             |
| ---------- | ------------------------------- | -------------------------- | ----------------------------------------- |
| Before BR  | Yes — directly set field values | Yes                        | Validate/transform fields before save     |
| After BR   | No (already saved)              | Yes — user waits           | Update related records, trigger workflows |
| Async BR   | No                              | **No** — background thread | Notifications, integrations, slow tasks   |
| Display BR | Via `g_scratchpad` only         | Yes (form load)            | Pass server data to client scripts        |

> ⚠️ **EXAM TRAP:** Async BRs do **NOT** block the user. If an exam question asks for **immediate feedback** after a save, Async is wrong — use After BR. If the task is a notification or integration, Async is the right answer.

> 🧠 **MEMORY TIP:** "Before = shape it. After = react to it. Async = fire and forget. Display = feed the form."

### Business Rule Specific APIs

```javascript
current; // record that triggered the BR
previous; // record values BEFORE the update
current.setAbortAction(true); // stop the save / abort transaction
current.update(); // save changes (⚠️ don't use in Before BR!)
current.isNewRecord(); // is this a new record?

// Check if field was changed
if (current.priority != previous.priority) {
} // compare old vs new
if (current.priority.changed()) {
} // built-in method
if (current.priority.nil()) {
} // is field empty?
```

### Flow Designer — Key Concepts

Flow Designer is the no-code/low-code replacement for legacy Workflow.

| Concept         | What to know                                                                     |
| --------------- | -------------------------------------------------------------------------------- |
| Trigger         | What starts the flow: record created/updated, schedule, inbound email, or manual |
| Action          | A single step — create record, update field, send email, call spoke              |
| Spoke           | A pre-built integration package (e.g., Slack Spoke, JIRA Spoke)                  |
| Subflow         | A reusable flow callable from other flows — like a Script Include for flows      |
| Script step     | Run arbitrary JavaScript inside a flow when no built-in action exists            |
| Action Designer | Where you build custom actions that don't exist out of the box                   |
| Flow Logic      | If/Else, For Each loops, Wait for condition                                      |

> 💡 **No-code automation = Flow Designer** (not Business Rule)
> 💡 **Subflow** = reusable logic YOU built | **Spoke** = pre-built npm-like package for external platforms
> 📝 **NOTE:** Flow Designer runs **after** the database write, similar to an After BR. To **prevent** a save, use a Before BR — you cannot do that from Flow Designer.

### Scheduled Jobs vs Async Business Rules

|              | Scheduled Job                               | Async Business Rule                            |
| ------------ | ------------------------------------------- | ---------------------------------------------- |
| Triggered by | A time schedule (daily, hourly, cron)       | A DB event (insert, update, delete)            |
| When to use  | Nightly batch processing, periodic cleanup  | Background work triggered by a record change   |
| Example      | Close resolved incidents older than 30 days | When a new incident is inserted, post to Slack |

### Practice Questions — Domain 4

> 📋 **PRACTICE Q:** A Business Rule sets a field value on the current record. After deployment, users report the save operation is stuck in an infinite loop. What is the most likely cause?

> ✅ **ANSWER:** The Business Rule is calling `current.update()`. This triggers another save event, which fires the Business Rule again. **Fix:** remove `current.update()` from a Before BR — the platform handles the save automatically.

---

> 📋 **PRACTICE Q:** A notification email must be sent after a high-priority incident is created, but the user should not have to wait for it. Which Business Rule type is most appropriate?

> ✅ **ANSWER:** **Async Business Rule.** It runs in a background thread after the DB commit, so the user is not blocked.

---

## Domain 5: Working with External Data (10%)

### Import Sets — Full Pipeline

```
External Data Source (CSV, Excel, JDBC, REST)
         |
         v
   [Import Set Table]   <── Staging table (u_import_*). Raw data lands here first.
         |
         v
   [Transform Map]      <── Maps source columns to target table fields.
         |                  Contains Field Maps (1 per source→target mapping).
         |                  Optional: coalesce field to match/update existing records.
         |                  Optional: "Run business rules" checkbox.
         v
   [Target Table]       <── Final destination (e.g., incident, sys_user).
```

### Transform Map — Key Settings

| Setting                    | What it does                                          | Note                                                  |
| -------------------------- | ----------------------------------------------------- | ----------------------------------------------------- |
| Run business rules         | Whether BRs fire on the target table during transform | Unchecked by default — check it if BRs must run       |
| Coalesce field             | Match existing records to prevent duplicates          | If match found, updates; otherwise inserts            |
| Script (field map)         | Transform source value before writing to target       | Access source with `source.field_name`                |
| Run script (transform map) | Runs once before/after the entire transform           | `onBefore`, `onAfter`, `onStart`, `onComplete` events |

### UI Policy vs Data Policy — Critical Distinction

|                     | UI Policy                              | Data Policy                                     |
| ------------------- | -------------------------------------- | ----------------------------------------------- |
| Runs where?         | Browser (client-side)                  | Server (database layer)                         |
| Enforces on import? | **NO** — imports bypass the browser    | **YES** — enforced at DB level                  |
| Controls            | Mandatory, visible, read-only on forms | Mandatory, read-only (no visibility control)    |
| Enforcement scope   | Current user's form session only       | All data writes — forms, imports, REST, scripts |
| When to use         | Dynamic form UX                        | Enforcing data integrity across all channels    |

> ⚠️ **EXAM TRAP:** UI Policies do **NOT** enforce data imported via Import Sets. If a question asks how to ensure a field is mandatory for both form saves AND imports, the answer is **Data Policy**.

### Practice Questions — Domain 5

> 📋 **PRACTICE Q:** An administrator imports a CSV file of user records. A field on `sys_user` is defined as mandatory in a UI Policy. After the import, records exist with that field empty. Why?

> ✅ **ANSWER:** UI Policies are browser-side only and do not apply to server-side imports. To enforce the mandatory rule on imports, a **Data Policy** must be created on the `sys_user` table for that field.

---

## Domain 6: Managing Applications (10%)

### Update Sets vs App Repository

|                    | Update Set                                        | App Repository (Studio)                 |
| ------------------ | ------------------------------------------------- | --------------------------------------- |
| Best for           | Global/legacy configurations                      | Scoped apps (preferred)                 |
| Captures           | Configuration changes (scripts, forms, workflows) | Full scoped app including all artifacts |
| Does NOT capture   | Data — records, incidents, users, attachments     | Instance-specific settings              |
| Source control     | Manual export XML → import to other instance      | Git integration supported               |
| Deployment flow    | Dev → Test → Prod (Preview before Commit)         | Dev → Test → Prod via app version       |
| Conflict detection | Preview step shows conflicts before commit        | Compare with published version          |

> ⚠️ **EXAM TRAP:** Update Sets capture **CONFIGURATION**, not **DATA**. Creating a new incident record, uploading an attachment, or adding a user will **NOT** appear in your Update Set.

### Update Set Deployment — Step by Step

```
1. In SOURCE instance:
   System Update Sets > Local Update Sets
   Create / select your Update Set > mark Complete
   Export to XML file

2. In TARGET instance:
   System Update Sets > Retrieved Update Sets
   Import XML file

3. PREVIEW the Update Set (mandatory best practice)
   Identifies conflicts with existing customizations
   Review any problem records before committing

4. COMMIT the Update Set
   Applies all configuration changes to the target
   This action cannot be easily undone — always preview first!
```

> 🧠 **MEMORY TIP:** "**Complete → Export → Import → Preview → Commit**". If any exam option says to skip Preview, that's a trap — always preview first.

### Service Catalog

| Term            | Description                                                            |
| --------------- | ---------------------------------------------------------------------- |
| Catalog Item    | A requestable item; creates records in `sc_request` / `sc_req_item`    |
| Record Producer | Creates a record in **any table you specify** through the catalog      |
| Order Guide     | Bundles multiple catalog items together (e.g., New Employee Setup)     |
| Variable        | A form field on a catalog item/record producer                         |
| Variable Set    | A **reusable group of variables** attachable to multiple catalog items |

> 💡 **sc\_ = Service Catalog** — easy way to remember the table prefix!
> 💡 **Variable Set** is to variables what a **Script Include** is to scripts — both exist for reusability!

### REST APIs

The URL structure is predictable:

```
https://instance.service-now.com/api/now/table/{table_name}

GET    /api/now/table/incident          → get all incidents
GET    /api/now/table/incident/{sys_id} → get specific incident
POST   /api/now/table/incident          → create new incident
PUT    /api/now/table/incident/{sys_id} → update incident
DELETE /api/now/table/incident/{sys_id} → delete incident
```

| API               | Direction             | Use case                                        |
| ----------------- | --------------------- | ----------------------------------------------- |
| Table API         | External → ServiceNow | External system reads/writes ServiceNow records |
| REST Message      | ServiceNow → External | ServiceNow calls an external REST API           |
| Scripted REST API | External → ServiceNow | Custom endpoints YOU build in ServiceNow        |

> 💡 **Exam key:** External talking TO ServiceNow → **Table API** | ServiceNow talking TO external → **REST Message** | Need a custom endpoint in ServiceNow → **Scripted REST API**

### Dictionary Attributes

Added as `attribute_name=true` or `attribute_name=value` in a field's dictionary entry.

| Attribute                      | What it does                                   |
| ------------------------------ | ---------------------------------------------- |
| `no_sort=true`                 | Prevents column from being sorted in list view |
| `no_filter=true`               | Prevents field from being used as a filter     |
| `no_negative=true`             | Prevents negative numbers                      |
| `show_if_empty=true`           | Shows field even if it has no value            |
| `edge_encryption_enabled=true` | Encrypts field data                            |

> 💡 **Pattern:** Attributes that **disable** something usually start with `no_`!

### Key Roles

| Role               | What it does                                                        |
| ------------------ | ------------------------------------------------------------------- |
| `admin`            | Full system access — keys to the kingdom                            |
| `security_admin`   | Required to create/modify ACL rules (elevated, separate from admin) |
| `itil`             | Standard role for service desk agents (incidents, requests)         |
| `import_set_admin` | Manages data imports into ServiceNow                                |
| `catalog_admin`    | Manages the Service Catalog                                         |

### Delegated Development

| Concept                  | What to know                                                           |
| ------------------------ | ---------------------------------------------------------------------- |
| Delegated Developer role | Grants development access to a specific scoped app                     |
| Code Review              | A designated approver must review changes before they are promoted     |
| Scope isolation          | Delegated developer cannot modify artifacts outside their assigned app |
| Why use it               | Allows team-based development without handing out admin access         |

### Practice Questions — Domain 6

> 📋 **PRACTICE Q:** A developer completes workflow and script changes in a dev instance. They export the Update Set and import it to test. During Preview, a conflict is shown on a Business Rule. What should happen next?

> ✅ **ANSWER:** **Resolve the conflict before committing.** Review both versions, choose which to keep (or merge manually), then proceed to Commit. Never commit a conflicted Update Set without reviewing it.

---

## Glide API Quick Reference

### GlideSystem / gs (Server Side) — System & Session Info

```javascript
// Current user
gs.getCurrentUserID(); // current user's sys_id
gs.getCurrentUserName(); // current user's username
gs.getUser().getFirstName(); // current user's first name
gs.getUser().getEmail(); // current user's email
gs.hasRole("admin"); // check if user has role (returns true/false)

// Logging
gs.log("message"); // write to log
gs.info("message"); // info level log
gs.warn("message"); // warning level log
gs.error("message"); // error level log

// Messages (server side)
gs.addInfoMessage("message"); // show info message to user
gs.addErrorMessage("message"); // show error message to user

// Utility
gs.now(); // current date/time
gs.nowDateTime(); // current datetime string
gs.tableExists("table_name"); // check if table exists
gs.getProperty("property.name"); // get system property
```

### GlideForm / g_form (Client Side) — Form Manipulation

```javascript
// Getting values
g_form.getValue("field"); // get field value
g_form.getDisplayValue("field"); // get display value

// Setting values
g_form.setValue("field", "value"); // set field value

// Visibility
g_form.setVisible("field", true); // show/hide field
g_form.setDisplay("field", true); // show/hide field AND label

// Field behavior
g_form.setMandatory("field", true); // make field mandatory
g_form.setReadOnly("field", true); // make field read only

// Messages
g_form.addInfoMessage("message"); // show info message on form
g_form.addErrorMessage("message"); // show error message on form
g_form.clearMessages(); // clear all messages

// Sections
g_form.setSectionDisplay("name", true); // show/hide a form section

// Misc
g_form.save(); // save the form
g_form.submit(); // submit the form
g_form.getUniqueValue(); // get current record's sys_id
```

### g_user (Client Side) — Current User Info

```javascript
g_user.userID; // current user's sys_id
g_user.userName; // current user's username
g_user.firstName; // current user's first name
g_user.lastName; // current user's last name
g_user.email; // current user's email
g_user.hasRole("role_name"); // check if user has role
g_user.hasRoleExactly("role_name"); // check role without admin override
```

### g_scratchpad (Server → Client Bridge)

```javascript
// In a Display Business Rule (server side) — runs BEFORE form loads
g_scratchpad.anyName = "any value"; // set any property

// In a Client Script (client side) — reads on form load
var val = g_scratchpad.anyName; // read the property
```

> 💡 **g_scratchpad = lunchbox packed at home, opened at school** (server packs it → client reads it on load)

---

## Master Quick Reference

### Client vs Server API Cheat Sheet

| Object / API       | Client or Server        | Purpose                                               | Key Gotcha                                         |
| ------------------ | ----------------------- | ----------------------------------------------------- | -------------------------------------------------- |
| `g_form`           | Client only             | Get/set field values, visibility, mandatory on a form | Not available in Business Rules                    |
| `g_user`           | Client only             | Current logged-in user info                           | Use `gs` on server side instead                    |
| `current`          | Server only             | GlideRecord in a BR representing the current record   | Do not call `current.update()` in a Before BR      |
| `previous`         | Server only             | Pre-update field values in a Business Rule            | Compare with `current` to detect changes           |
| `GlideRecord`      | Server only             | Query / insert / update any table                     | Use `getValue()`, not `gr.field_name` directly     |
| `GlideAjax`        | Client → Server         | Call a server Script Include from a client script     | Script Include must extend `AbstractAjaxProcessor` |
| `gs` (GlideSystem) | Server only             | Logging, user info, system properties                 | `gs.getUser()` returns a GlideUser object          |
| `g_scratchpad`     | Bridge: server → client | Pass data from Display BR to client scripts           | Set in Display BR, read in client script           |
| Script Include     | Server                  | Reusable server-side functions                        | Set "Accessible from" for cross-scope use          |

> 💡 **Memory trick:** Everything starting with `g_` is **client-side**. `gs` and `gr` are **server-side**.

### BR Timing Quick Reference

| Type        | Runs                         | Can abort? | Notes                                 |
| ----------- | ---------------------------- | ---------- | ------------------------------------- |
| Before BR   | Before DB write              | ✅ Yes     | Do NOT call `current.update()`        |
| After BR    | After DB commit              | ❌ No      | Update related records; user waits    |
| Async BR    | After DB commit (background) | ❌ No      | Non-blocking — fire and forget        |
| Display BR  | On form load                 | ❌ No      | Use `g_scratchpad` to pass to client  |
| Data Policy | Before DB write              | ✅ Yes     | Enforces imports (UI Policy does not) |
| UI Policy   | Client/browser only          | ❌ No      | Form only — cannot enforce imports    |

### Prefix Quick Reference

| Prefix | Meaning                          |
| ------ | -------------------------------- |
| `x_`   | Scoped app table/field           |
| `u_`   | Global scope custom table/field  |
| `sys_` | ServiceNow native platform table |
| `sc_`  | Service Catalog tables           |

### Script Include Quick Reference

- **Server-side only** reusable logic
- Called from: Business Rules, other Script Includes, Scheduled Jobs, REST APIs
- Called from Client Scripts via **GlideAjax** (must extend `AbstractAjaxProcessor`)
- **Regular** → server-side only
- **Ajax-enabled** → extends `AbstractAjaxProcessor`, callable from Client Scripts via GlideAjax

---

## All Exam Traps — Combined List

| Scenario / Symptom                          | Trap                                     | Correct Answer                                                 |
| ------------------------------------------- | ---------------------------------------- | -------------------------------------------------------------- |
| Form is slow after save                     | Thinking it's a client issue             | Check for heavy After BR — consider Async instead              |
| Field not saving despite no error           | Checking only UI settings                | Check dictionary read-only flag                                |
| Client Script needs DB data                 | Using GlideRecord in client script       | Use GlideAjax + client-callable Script Include                 |
| Mandatory not enforced on import            | Using UI Policy                          | Use Data Policy — it's server-side                             |
| Cross-scope call fails silently             | Assuming it's a code bug                 | Check "Accessible from" on the Script Include                  |
| User sees list, can't open record           | Checking field-level ACLs first          | Record-level ACL is blocking — check that first                |
| User can't see list at all                  | Checking record/field ACLs               | Table-level ACL is blocking — start there                      |
| Button on form needs server logic           | Using a Business Rule                    | Use a UI Action with Form Button checked                       |
| Field change needs silent server call       | Using a UI Action                        | Use GlideAjax + client-callable Script Include                 |
| Move config between instances               | Manually recreating scripts              | Use Update Set or App Repository                               |
| Artifact in wrong scope                     | Blaming the script                       | Check App Picker — was the correct scope selected at creation? |
| BR causing infinite loop                    | Adding more conditions to the BR         | Remove `current.update()` from the Before BR                   |
| `onCellEdit` used on a form                 | Thinking it works on forms               | `onCellEdit` is list view only — use `onChange` on forms       |
| `security_admin` assumed = `admin`          | Thinking `security_admin` bypasses ACLs  | Only `admin` bypasses ACLs — `security_admin` manages them     |
| Update Set expected to include data records | Expecting incident records in Update Set | Update Sets capture config only, not data                      |
| UI Policy expected to enforce on import     | Using UI Policy for import validation    | UI Policies are browser-side — use Data Policy                 |
| Making a field read-only based on role      | Using an ACL                             | ACLs have no "read-only" mode — use Client Script              |
| `gr.field_name` used for comparison         | Thinking it returns the value            | Returns GlideElement object — use `getValue()` instead         |
| `getValue()` vs `getDisplayValue()`         | Using wrong one for reference fields     | `getValue()` = sys_id; `getDisplayValue()` = human label       |

---

## Mental Checklist for Exam Decisions

- **ACL issue?** → Table deny → stop; Record deny → stop; Field deny → restrict field only
- **BR Timing?** → Before → DB write → After → Async (background)
- **Client vs Server?** → `g_form` / `g_user` = client; `current` / `GlideRecord` / `gs` = server
- **Cross-scope failure?** → Check "Accessible from" on the Script Include
- **Button on form?** → UI Action (not Business Rule)
- **Silent server call from client?** → GlideAjax + client-callable Script Include
- **Mandatory on import?** → Data Policy (not UI Policy)
- **Move config between instances?** → Update Set or App Repository
- **Wrong scope on artifact?** → Check App Picker, verify Application field on the record
- **Read a field in GlideRecord?** → `getValue()`, not `gr.field_name`
- **Read-only behavior based on role?** → Client Script, NOT an ACL
- **Need data on form load?** → g_scratchpad. Need data after interaction? → GlideAjax

---

## Mnemonics & Memory Tricks

- **Alert = JavaScript = Client Script** (alert() is a JS method → it's client side!)
- **`g_` prefix = client side** (g_form, g_user, g_scratchpad)
- **`gs` = system/session** (server side)
- **`gr` = records/database** (server side)
- **`no_` prefix = disabling a dictionary attribute** (no_sort, no_filter, no_negative)
- **sc\_ = Service Catalog** (sc_request, sc_req_item)
- **UI Policy = simple form conditions** | **Client Script = complex logic**
- **ACL = security (grants/denies access)** | **UI Policy = appearance only**
- **Data Policy = enforces beyond the UI** (API, Import Sets too)
- **g_scratchpad = lunchbox packed at home, opened at school** (server → client on load)
- **Subflow = utility function you wrote** | **Spoke = npm package someone else built**
- **x\_ = scoped** | **u\_ = global (user-created)** | **sys\_ = system/native**
- **Before = shape it. After = react to it. Async = fire and forget. Display = feed the form.**
- **Complete → Export → Import → Preview → Commit** (Update Set deployment order)

---

## 7–10 Day Study Plan

| Days | Focus                                                                              | Goal                                                                    |
| ---- | ---------------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| 1–2  | Domain 1 & 2: Scoped apps, tables, client scripts, UI Policy, GlideRecord          | Build a small scoped app with 2 tables, client scripts, UI Policies     |
| 3–4  | Domain 3 & 4: ACLs, Business Rules, Flow Designer, Scheduled Jobs                  | Create ACLs at all 3 levels, all 4 BR types, observe timing differences |
| 5    | Domain 2 deep dive: GlideAjax, UI Actions, `g_scratchpad`, Script Includes         | Build a GlideAjax call from scratch — this is exam-heavy content        |
| 6    | Domain 5 & 6: Import Sets, Transform Maps, Update Sets, App Repository             | Import a CSV, test Data Policy, export and import an Update Set         |
| 7    | Timed practice: 60 questions in 90 minutes                                         | Identify weak areas — revisit those domain sections                     |
| 8    | Quick-reference drills: ACL levels, BR timing, prefix table, client vs server APIs | Can you recall all tables and flows without looking?                    |
| 9–10 | Light review, mental walkthroughs, exam-day confidence                             | Re-read all exam traps, sleep well, trust your prep                     |
