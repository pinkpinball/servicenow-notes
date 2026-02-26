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

### Guided Application Creator (GAC) — Full Deep Dive

GAC is a step-by-step wizard that walks you through building the skeleton of a scoped application. It is intentionally low-code and guided — you don't need to know where everything lives in the platform. Once GAC finishes, you continue development in **Studio**.

#### How to Access GAC

Two entry points:

1. **System Applications > My Company Applications > Create New**
2. **Studio > Create Application button**

> 💡 The GAC welcome screen appears **only the first time** it is used. To see it again, delete the `sn_g_app_creator.has_viewed_gac` record from the `sys_user_preference` table.

#### Who Can Use GAC

Users with the **`sn_g_app_creator.app_creator`** role can access GAC. This is intended for System Administrators, Developers, and Business Analysts.

> ⚠️ **EXAM TRAP:** By default, GAC does **not** offer the option to create a **Global** application — it only creates scoped apps. To enable global app creation in GAC, the system property `sn_g_app_creator.allow_global` must be set to `true`.

#### GAC Wizard Steps — In Order

GAC walks you through these steps sequentially:

```
1. Application Configuration  → Name, scope ID (max 18 chars), description
2. User Role                  → Assign existing roles or create new ones for the app
3. User Experience            → Choose up to 3: Workspace, Mobile, Classic
4. Table                      → Designate data tables (see below)
5. Field Inputs               → Add fields to the table
6. Table Configuration        → Configure access controls, extensibility, etc.
7. Next Steps                 → Opens Studio for further development
```

> 💡 **Best practice:** Always create at least one scoped role in the User Role step — don't skip it and rely on global roles alone.

#### The Three Ways to Designate a Table in GAC

This is a **frequently tested exam question** — know all three options and when to use each.

| Method                       | What it does                                                          | When to use it                                                    |
| ---------------------------- | --------------------------------------------------------------------- | ----------------------------------------------------------------- |
| **Create a new table**       | Builds a brand-new table from scratch with only default system fields | No similar table exists; you want full control over the schema    |
| **Extend an existing table** | Creates a child table that inherits all fields and logic of a parent  | Parent table has useful fields/logic (e.g., `task`, `cmdb_ci`)    |
| **Upload a spreadsheet**     | Turns a CSV/Excel file into a new table, mapping columns to fields    | You have existing data to import and use as the table's structure |

> ⚠️ **EXAM TRAP:** "Create table from template" is **NOT** a valid GAC option. If you see it in a question, it's a distractor — the correct options are create from scratch, extend, or upload spreadsheet.

> ⚠️ **EXAM TRAP:** The exam also asks which file types can be uploaded. Only **spreadsheets (CSV/Excel)** are supported. PDFs and Word documents are **NOT** valid upload types in GAC.

#### Create a New Table vs. Extend a Table — Decision Guide

| Signal in the scenario                                 | Right choice          |
| ------------------------------------------------------ | --------------------- |
| No similar table exists in ServiceNow                  | Create new table      |
| Table will hold reference/lookup data only             | Create new table      |
| You want to script all behaviors yourself              | Create new table      |
| A similar table already has useful fields and logic    | Extend existing table |
| You want approval workflows (other than User Approval) | Extend `task`         |
| You're modeling a Configuration Item type              | Extend `cmdb_ci`      |
| You want to reuse fields without rebuilding them       | Extend existing table |

> 💡 **Key rule on extending:** The parent table must have **Extensible = true** in its dictionary entry. You **cannot** extend a system table or database view. Inherited fields cannot be deleted from the child table.

> 💡 **Approval workflow note:** The **User Approval** workflow activity works with ALL tables. All **other** approval activities (e.g., group approval, manager approval) only work with tables that extend **`task`**.

#### Table Configuration Options Set During GAC

When creating or extending a table in GAC, these options are configured:

| Option                 | What it does                                                                         |
| ---------------------- | ------------------------------------------------------------------------------------ |
| Create access controls | Auto-generates baseline ACLs for the new table — **must be checked for scoped apps** |
| Extensible             | Allows other tables to extend this one in the future                                 |
| Create module          | Automatically adds the table to the app's navigation menu                            |
| Auto-number            | Adds an auto-incrementing number field (like INC0001234)                             |

> ⚠️ **EXAM TRAP:** If **Create access controls** is not checked when creating a table in a scoped app, no ACLs are generated and access may be unexpectedly open or blocked depending on the app's default access settings.

#### What GAC Does NOT Do

GAC builds the skeleton — it doesn't do everything. After GAC, you go to Studio to:

- Add Business Rules, Client Scripts, Script Includes
- Create UI Policies and UI Actions
- Configure Form Layouts in detail
- Set up Flow Designer automations
- Add additional tables beyond what GAC created

> 🧠 **MEMORY TIP:** **GAC = foundation pour**. It sets up the structure. Studio = where you build the house on top.

#### Practice Questions — GAC

> 📋 **PRACTICE Q:** A developer is using GAC to create a new application. When asked to designate a data table, they want to reuse the state management, assignment, and approval logic already built into ServiceNow's task management system. Which table option should they choose, and what should it extend?

> ✅ **ANSWER:** Choose **Extend an existing table** and extend the **`task`** table. This inherits all task fields, state management logic, and enables the full suite of approval workflow activities.

---

> 📋 **PRACTICE Q:** Which of the following is NOT a valid way to designate a data table in GAC? (A) Create a new table on the platform (B) Upload an existing spreadsheet (C) Create a table from a template (D) Use an existing table on the platform

> ✅ **ANSWER:** **(C) Create a table from a template.** Templates are not an option in GAC. The three valid methods are: create from scratch, extend a table, and upload a spreadsheet.

---

> 📋 **PRACTICE Q:** A developer opens GAC but does not see an option to create a Global application. What must be configured to enable this?

> ✅ **ANSWER:** The system property **`sn_g_app_creator.allow_global`** must be set to `true`. By default, GAC only allows scoped application creation.

### Cross-Scope Access — How it Really Works

When App A calls a Script Include in App B, ServiceNow checks the **"Accessible from"** field on the Script Include record.
If it says "This application scope only", the call silently fails at runtime — no obvious error, just no result.

| Setting                     | Meaning                                                   | Exam signal                   |
| --------------------------- | --------------------------------------------------------- | ----------------------------- |
| This application scope only | Only code in the same scope can call it                   | Default — restrictive         |
| All application scopes      | Any scope including global can call it                    | Required for shared utilities |
| Caller restriction          | Called resource controls access via `sys_scope_privilege` | Advanced — check the table    |

> 🧠 **MEMORY TIP:** To diagnose a cross-scope failure: open the Script Include record and look at "Accessible from". 9 times out of 10, that's the problem.

### Delegated Development — Full Deep Dive

Delegated Development lets you give a developer admin-level power over **one specific scoped app** without granting them system admin. This is the right tool for team-based or contractor-based development.

#### How to Set It Up

```
1. Open the scoped application record (via Studio or App Manager)
2. Click "Manage Developers" on the application record
3. Add users to the "Delegated Developers" related list
4. Those users now have developer access scoped to that app only
```

The delegated developer works entirely inside Studio, which filters to show only their assigned app. They cannot navigate to other scoped apps or to global scope artifacts.

#### What a Delegated Developer CAN Do

- Create and edit Tables, Fields, Business Rules, Client Scripts, Script Includes within their app
- Create UI Policies, UI Actions, and ACLs within their app
- Publish and version the app to the App Repository
- View the application's update history and versions

#### What a Delegated Developer CANNOT Do

- Modify any artifact outside their assigned scope
- Access System Administration menus (User Management, Update Sets, etc.)
- Change instance-level settings or system properties
- Grant themselves `admin` or `security_admin`
- Edit global scope scripts or other scoped apps

#### Delegated Developer vs Admin — Key Comparison

| Capability                          | Admin | Delegated Developer     |
| ----------------------------------- | ----- | ----------------------- |
| Modify artifacts in any scope       | ✅    | ❌ (own scope only)     |
| Create/edit tables in assigned app  | ✅    | ✅                      |
| Access System Admin menus           | ✅    | ❌                      |
| Grant other users the admin role    | ✅    | ❌                      |
| Publish app to App Repository       | ✅    | ✅ (unless Code Review) |
| Bypass ACLs                         | ✅    | ❌                      |
| Manage Update Sets                  | ✅    | ❌                      |
| View artifacts in other scoped apps | ✅    | ❌                      |

> 🧠 **MEMORY TIP:** Think of Delegated Developer as a **room key**, not a master key. They can only open the one door they've been assigned.

#### Code Review for Delegated Developers

When **Code Review** is enabled on an application, delegated developers cannot directly publish their changes. Every change must be approved by a designated reviewer first.

```
Delegated Developer saves/commits a change
         |
         v
Change enters "Pending Review" state
         |
         v
Designated Code Reviewer inspects the change
         |
   [APPROVE] ──────> Change is promoted and published
         |
   [REJECT]  ──────> Developer must revise and resubmit
```

> ⚠️ **EXAM TRAP:** Code Review is configured **per application**, not globally. Enabling it on one app does not affect other apps.

> ⚠️ **EXAM TRAP:** A delegated developer stuck in "Pending Review" does NOT need `admin` access — they need an assigned **Code Reviewer** to approve or reject their submission.

#### Practice Questions — Delegated Development

> 📋 **PRACTICE Q:** A company wants a contractor to build a new scoped app without being able to access HR data stored in a separate scoped app on the same instance. What should the admin configure?

> ✅ **ANSWER:** Create the scoped app and add the contractor as a **Delegated Developer** on that app only. They get full development access to the assigned app but cannot see or modify artifacts in other scopes.

---

> 📋 **PRACTICE Q:** A delegated developer reports they cannot publish their changes to the App Repository even though their work is complete. They have not been given the `admin` role. What is the most likely cause?

> ✅ **ANSWER:** **Code Review is enabled** on the application. Their changes are in "Pending Review" state and require a designated reviewer to approve them before publishing is allowed.

---

### Application Access Settings — Deep Dive

**Application Access** controls how a scoped app's tables interact with the rest of the platform. Found on the table record under the **Application Access** tab.

| Setting                           | What it does                                                                    | Key exam note                                                 |
| --------------------------------- | ------------------------------------------------------------------------------- | ------------------------------------------------------------- |
| **Can read**                      | Other scopes can read records from this table                                   | If unchecked, Can create/update/delete become unavailable too |
| **Can create**                    | Other scopes can insert new records                                             | Requires Can read to be checked first                         |
| **Can update**                    | Other scopes can modify existing records                                        | Requires Can read to be checked first                         |
| **Can delete**                    | Other scopes can delete records                                                 | Requires Can read to be checked first                         |
| **Allow configuration**           | Out-of-scope apps can create Business Rules, Client Scripts, etc. on this table | ⚠️ Opens the table to configuration from other scopes         |
| **Allow access via web services** | Table is accessible via REST/SOAP APIs                                          | Does NOT bypass ACLs — callers still need correct permissions |

> ⚠️ **EXAM TRAP:** If **Can read** is unchecked, **Can create, Can update, and Can delete** become unavailable in the UI. "Allow access via web services" is NOT affected by Can read.

> ⚠️ **EXAM TRAP:** **Allow configuration** = out-of-scope apps **can create Business Rules** on the table. This is a direct exam question. If Allow configuration is NOT checked, only in-scope scripts can configure the table.

> ⚠️ **EXAM TRAP:** "Allow access via web services" does **not** mean anyone can access the table — the caller still needs correct ACL permissions. It only controls whether the table is reachable via web service at all.

> 💡 **Cross-scope default for new files:** When creating new application files in a scoped app, **REST Messages** have cross-scope access turned on by default. Script Includes, Tables, and Workflows do NOT.

---

### Modules — Link Types and Configuration

A **Module** is a navigation link inside an **Application Menu**. Every module must have a **Link type** that defines what happens when a user clicks it.

**Valid Link Types (memorize this list — it appears directly on the exam):**

| Link Type            | What it opens                                                      |
| -------------------- | ------------------------------------------------------------------ |
| List of Records      | A filtered list of records from a table                            |
| Content Page         | A custom UI page                                                   |
| URL (from arguments) | Any URL — used to open the Application Properties page for the app |
| Assessment           | A survey or assessment                                             |
| Separator            | A visual divider between modules (not a link — just a line)        |
| Timeline Page        | A timeline view of records                                         |

> ⚠️ **EXAM TRAP:** Common distractor answers include **"Catalog Type"** and **"Roles"** — these are NOT valid link types. The correct list is: Assessment, List of Records, Separator, Timeline Page, Content Page, URL (from arguments).

> 💡 **Application Properties module:** The way to expose the Application Properties page from the nav menu is to create a module with link type **URL (from arguments)** pointing to the sys_properties page filtered by the app's category.

**Module configuration options:**

| Option                              | What it does                                                                          |
| ----------------------------------- | ------------------------------------------------------------------------------------- |
| **Override application menu roles** | Users with the module's role can access it even without access to the parent app menu |
| Order                               | Controls display order within the application menu                                    |
| Active                              | Whether the module appears in navigation                                              |
| Roles                               | Restricts who can see the module                                                      |

> 💡 If a module is created when a table is created in GAC, the **default behavior** is to open a list of all records from that table.

---

### Application Properties

**Application Properties** are system properties scoped to your application. They let admins and developers change an app's behavior without editing its code or artifacts.

**Key facts:**

- Stored in the `sys_properties` table, like all system properties
- Grouped by a **Category** that matches the application name
- Accessed in the nav via a module with link type **URL (from arguments)**
- Read in scripts via `gs.getProperty('x_myco_myapp.property_name')`

> Application Properties allow a developer or admin to make changes to an application's behavior **without modifying application artifacts**. Use them for feature flags, configurable URLs, thresholds, default values — anything that should be changeable without a code deploy.

```javascript
// ✅ Reading an application property in a server-side script
var threshold = gs.getProperty("x_myco_myapp.escalation_threshold", "5"); // second arg = default
var apiUrl = gs.getProperty("x_myco_myapp.external_api_url");

// ✅ Setting a property programmatically
gs.setProperty("x_myco_myapp.feature_flag", "true");
```

> ⚠️ **EXAM TRAP:** Application Properties are NOT a separate artifact type — they are ordinary `sys_properties` records that appear on the Application Properties page because they share the app's category name.

> 🧠 **MEMORY TIP:** Application Properties = **config file for your app**. Change behavior without touching code.

---

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
gr.addEncodedQuery("active=true"); // encoded query string, can pass in a pre-built query!
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

gr.addNullQuery("assignment_group"); // ✅ use when BUILDING a query
gr.addQuery("assignment_group", ""); // ❌ unreliable for null checks

gr.orderByDesc("field"); // descending order
gr.orderBy("field"); // ascending order
gr.setLimit(10); // limit results
gr.addEncodedQuery("active=true^priority=1"); // pre-built query string

// ^ means AND
active=true^priority=1^state=2

// Translates to:
// active = true AND priority = 1 AND state = 2
```

**The encoded query operators:**

```
^        → AND
^OR      → OR
^NQ      → starts a new query (advanced)
=        → equals
!=       → not equal
LIKE     → contains
STARTSWITH → starts with
ENDSWITH   → ends with
ISEMPTY  → is empty
ISNOTEMPTY → is not empty
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

### UI Actions — Full Deep Dive

A **UI Action** creates a button, context menu item, or link on a form or list. It is the correct tool when a **user interaction** should trigger logic — not a Business Rule.

#### UI Action Types

| Type               | Where it appears                        | When to use                                  |
| ------------------ | --------------------------------------- | -------------------------------------------- |
| Form Button        | Button bar at top of a form             | Most common — user clicks a button on a form |
| Context Menu Item  | Right-click menu on a form              | Secondary actions, less prominent            |
| List Action        | Dropdown on a list row                  | Actions taken on a record from a list view   |
| List Banner Button | Button above a list (acts on selection) | Bulk actions on multiple selected records    |

#### Client vs Server in a UI Action

A UI Action script can run **client-side**, **server-side**, or **both** — depending on configuration.

| Configuration           | What happens                                                                |
| ----------------------- | --------------------------------------------------------------------------- |
| `Client` checkbox OFF   | Script runs **server-side only** — `current` object is available            |
| `Client` checkbox ON    | Script runs **client-side first** — `g_form`, `g_user` available            |
| Both (via `gsftSubmit`) | Client runs first, then triggers the server-side portion of the same script |

> ⚠️ **EXAM TRAP:** UI Actions do NOT always run server-side. If the **Client** checkbox is checked, the script (or at least the `onclick` function) runs in the browser. This is a direct exam question.

#### Server-Only UI Action (most common)

```javascript
// ✅ Server-side only — Client checkbox unchecked
// 'current' is the record. action.setRedirectURL() controls where to go after.
current.state = 6; // closed
current.close_code = "Solved";
current.update();
action.setRedirectURL(current); // redirect back to the same record
```

**Key server-side UI Action APIs:**

```javascript
action.setRedirectURL(gr); // redirect to a specific record after submit
action.setReturnURL(url); // return to a specific URL
action.getGlideURI(); // get the current URI
action.getURL(); // get full URL
```

#### Client + Server UI Action (using `gsftSubmit`)

When you need to validate on the client first, then run server logic:

```javascript
// ✅ Client checkbox ON, Onclick field set to: confirmAction();
// The script field contains both client and server functions

// Client-side function (runs first when button is clicked)
function confirmAction() {
  if (!confirm("Are you sure?")) return false; // abort if user cancels
  gsftSubmit(null, g_form.getFormElement(), "my_action_name"); // trigger server side
}

// Server-side function (runs after gsftSubmit)
if (typeof window == "undefined") runServerSide(); // guard: only run on server
function runServerSide() {
  current.state = 3;
  current.update();
  action.setRedirectURL(current);
}
```

> 🧠 **MEMORY TIP:** `gsftSubmit(null, g_form.getFormElement(), 'action_name')` — the third argument **must match** the UI Action's **Action name** field exactly, not the display name.

> ⚠️ **EXAM TRAP:** `typeof window == 'undefined'` is the guard that prevents server-side code from running in the browser. Without it, the server portion throws a ReferenceError client-side.

---

### Form Designer vs Form Layout

Two tools exist for editing forms — the exam tests which capabilities belong to which.

| Feature                          | Form Designer                                      | Form Layout (classic) |
| -------------------------------- | -------------------------------------------------- | --------------------- |
| Interface                        | Drag-and-drop graphical                            | List-based            |
| Add existing fields              | ✅ (Fields tab)                                    | ✅                    |
| Create new fields                | ✅ (Field Types tab)                               | ✅                    |
| Remove fields from layout        | ✅ (hover → X button)                              | ✅                    |
| Add sections                     | ✅ (Field Types tab)                               | ✅                    |
| Delete a field from the table    | ❌ Cannot delete system fields                     | ❌                    |
| Edit field labels on child table | Changes label on child table only — NOT the parent |                       |

> ⚠️ **EXAM TRAP:** Removing a field from a form layout does **NOT** delete the field from the table. The field still exists in the schema — it's just not displayed on the form.

> ⚠️ **EXAM TRAP:** In Form Designer, editing the label of a field on a **child table** changes the label on the child table only — not the parent. Inherited fields can be relabeled per table.

> 💡 **Schema Map** is a separate tool (not part of Form Designer) that shows relationships between tables. It's accessed via System Definition > Tables, not from Form Designer.

---

### Debugging Client-Side Scripts

The exam asks which tools are used for client-side vs server-side debugging. Know the distinction cold.

| Tool                       | Client or Server | What it does                                                                   |
| -------------------------- | ---------------- | ------------------------------------------------------------------------------ |
| `jslog()`                  | **Client only**  | Writes a message to the browser console                                        |
| `g_form.addInfoMessage()`  | **Client only**  | Displays a blue info banner on the form                                        |
| Field Watcher              | **Client only**  | Shows current and previous values of watched fields as they change on the form |
| Debug Business Rule        | **Server only**  | Shows which Business Rules ran and in what order for a given transaction       |
| Debug Business Rule Detail | **Server only**  | Shows full script execution details for Business Rules                         |
| `gs.info()` / `gs.log()`   | **Server only**  | Writes to the system log (`gs.log()` unavailable in scoped apps)               |

> ⚠️ **EXAM TRAP:** `gs.log()` is **NOT** a client-side debugging strategy — it runs on the server. A direct exam question asks which of the following is NOT a client-side debugging tool, and `gs.log()` is the correct answer.

> ⚠️ **EXAM TRAP:** **Field Watcher** debugs **client-side** field changes on a form. It cannot debug server-side Business Rules or Scheduled Jobs.

> ⚠️ **EXAM TRAP:** **Debug Business Rule** vs **Debug Business Rule Detail** — BR shows which rules ran; BR Detail shows the full script output and variable values. Know both exist and that they are server-side tools.

```javascript
// ✅ Client-side debugging
jslog("Field value is: " + g_form.getValue("priority")); // browser console
g_form.addInfoMessage("Debug: state = " + g_form.getValue("state")); // visible on form

// ✅ Server-side debugging (NOT in scoped apps for gs.log)
gs.info("BR fired. Current state: " + current.getValue("state"));
gs.warn("Unexpected value encountered");
gs.error("Critical failure in script include");
```

---

### GlideDateTime in Scoped Apps

> ⚠️ **EXAM TRAP:** In scoped applications, the correct object for working with dates and times is **`GlideDateTime`**, not `GlideDate`. Some functions available in global scope (like `datediff`) are **not available** in scoped apps.

```javascript
// ✅ CORRECT — use GlideDateTime in scoped apps
var now = new GlideDateTime(); // current date/time
var dt = new GlideDateTime("2025-01-15 10:00:00"); // specific date/time
gs.info(dt.getDisplayValue()); // human-readable string
gs.info(dt.getValue()); // internal value (UTC)
dt.addDays(7); // add 7 days
dt.addSeconds(3600); // add 1 hour

// Comparing dates
var start = new GlideDateTime();
var end = new GlideDateTime();
end.addDays(5);
var diff = GlideDateTime.subtract(start, end); // returns GlideDuration
gs.info(diff.getDayPart()); // number of days difference
```

| Method              | What it returns                                |
| ------------------- | ---------------------------------------------- |
| `getValue()`        | Internal UTC string (for DB storage)           |
| `getDisplayValue()` | User's local time formatted string             |
| `getDate()`         | Returns a GlideDate object (date portion only) |
| `addDays(n)`        | Adds n days to the datetime                    |
| `addSeconds(n)`     | Adds n seconds to the datetime                 |
| `before(other)`     | Returns true if this datetime is before other  |
| `after(other)`      | Returns true if this datetime is after other   |

---

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

Business Rules have checkboxes for Insert, Update, Delete, Query
Checking Insert only → BR fires ONLY on new record creation
Simple, clean, no scripting needed

```
When:   Before / After / Async / Display
Insert:  ☑ (fires on new records)
Update:  ☑ (fires on updates)
Delete:  ☑ (fires on deletions)
Query:   ☑ (fires on queries)
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

| Concept         | What to know                                                                                        |
| --------------- | --------------------------------------------------------------------------------------------------- |
| Trigger         | What starts the flow: record created/updated, schedule, inbound email, application event, or manual |
| Action          | A single step — create record, update field, send email, call spoke                                 |
| Spoke           | A pre-built integration package (e.g., Slack Spoke, JIRA Spoke)                                     |
| Subflow         | A reusable flow callable from other flows — like a Script Include for flows                         |
| Script step     | Run arbitrary JavaScript inside a flow when no built-in action exists                               |
| Action Designer | Where you build custom actions that don't exist out of the box                                      |
| Flow Logic      | If/Else, For Each loops, Wait for condition                                                         |

> 💡 **No-code automation = Flow Designer** (not Business Rule)
> 💡 **Subflow** = reusable logic YOU built | **Spoke** = pre-built npm-like package for external platforms
> 📝 **NOTE:** Flow Designer runs **after** the database write, similar to an After BR. To **prevent** a save, use a Before BR — you cannot do that from Flow Designer.

### Scheduled Jobs vs Async Business Rules

|              | Scheduled Job                               | Async Business Rule                            |
| ------------ | ------------------------------------------- | ---------------------------------------------- |
| Triggered by | A time schedule (daily, hourly, cron)       | A DB event (insert, update, delete)            |
| When to use  | Nightly batch processing, periodic cleanup  | Background work triggered by a record change   |
| Example      | Close resolved incidents older than 30 days | When a new incident is inserted, post to Slack |

### Email Notifications — Full Breakdown

Email Notifications automate communication by sending emails when specified conditions or events occur. Configured under **System Notification > Email > Notifications**.

#### The Three Configuration Tabs

Every Email Notification is built around three tabs — this is a **direct exam question**:

| Tab      | What you configure                                       |
| -------- | -------------------------------------------------------- |
| **Who**  | Recipients — users, groups, roles, event parameters      |
| **What** | Content — subject, body, template, mail scripts          |
| **When** | Trigger — record-based conditions or event-based trigger |

> ⚠️ **EXAM TRAP:** The exam asks which three things are configured in a notification. The answer is **Who, What, When** — NOT "How" (delivery method is not a configurable tab).

#### Trigger Types: Record vs Event

| Trigger Type     | How it fires                                           | Example                                                  |
| ---------------- | ------------------------------------------------------ | -------------------------------------------------------- |
| **Record-based** | When a record is inserted or updated matching a filter | Priority changes to 1 on an incident                     |
| **Event-based**  | When a specific event is fired in the Event Registry   | `incident.commented` event fires when a comment is added |

#### The Weight Field — Most Tested Email Notification Concept

When multiple notifications target the **same record** and the **same recipients** simultaneously, ServiceNow uses **Weight** to decide which ones to send:

```
Weight = 0          → ALWAYS sent (regardless of other notifications)
Weight > 0          → Only the notification with the HIGHEST weight is sent
                       All others are moved to the "Skipped" mailbox
```

> ⚠️ **EXAM TRAP:** A Weight value of zero does **NOT** mean no email is sent — it means the notification is **always** sent. "A weight of zero means no email will be sent" is a known wrong answer on the exam.

> ⚠️ **EXAM TRAP:** The default Weight value is **0** — meaning by default, all notifications are always sent.

#### Referencing Fields and Events in Email Content

```
${field_name}               → Value of a field on the triggering record
${event.parm1}              → First parameter passed when the event was fired
${event.parm2}              → Second parameter
<mail_script>               → Opening tag for a server-side mail script block
  template.print('text');   → Print text into the email body
</mail_script>              → Closing tag
```

#### Watermark

The **watermark** is a hidden identifier added to every outbound email by default. It allows ServiceNow to match incoming reply emails back to the correct record. It always includes `Ref:` followed by a prefix (default `MSG`) and the record's auto-number.

> 💡 The watermark is what enables **inbound email** to update the correct ticket automatically.

---

### Events and the Event Registry

An **Event** is a named notification that something happened. Events decouple the "something happened" from "what to do about it" — scripts fire events, and other artifacts (notifications, script actions) react to them.

#### Event Flow

```
Server-side script fires an event:
  gs.eventQueue('incident.priority.changed', current, current.priority, previous.priority);
         |
         v
  Event Queue (processed asynchronously)
         |
         v
  Event Registry (must have matching event registered to be recognized)
         |
         v
  Email Notifications with matching event trigger → send email
  Script Actions with matching event trigger → run script
```

#### Registering an Event

Before ServiceNow can respond to an event, it must be **registered** in the Event Registry (`sysevent_register`). In Studio: **Create New > Event Registration**.

```javascript
// ✅ Firing an event from a Business Rule or Script Include
gs.eventQueue(
  "x_myco_myapp.record.escalated", // event name (must match registry)
  current, // GlideRecord that caused the event
  current.getValue("priority"), // parm1 — available as ${event.parm1} in notifications
  gs.getUserID(), // parm2 — available as ${event.parm2}
);
```

| `gs.eventQueue` Parameter | Purpose                                             |
| ------------------------- | --------------------------------------------------- |
| Event name                | Must match a registered event in the Event Registry |
| GlideRecord               | The record associated with the event                |
| parm1                     | First custom value — accessible in notifications    |
| parm2                     | Second custom value — accessible in notifications   |

> ⚠️ **EXAM TRAP:** ServiceNow can only respond to events that are **registered** in the Event Registry. Firing an unregistered event does nothing — no notification, no script action.

> 💡 **Events = publish/subscribe pattern.** The script fires and forgets. Notifications and Script Actions subscribe and react. They are decoupled.

---

### Automated Test Framework (ATF)

ATF allows you to create and run **automated tests** on ServiceNow — no manual clicking required. It's free with the Now Platform.

#### Key ATF Concepts

| Concept            | What it is                                                                     |
| ------------------ | ------------------------------------------------------------------------------ |
| Test               | A single automated test — contains a sequence of test steps                    |
| Test Suite         | A collection of tests grouped together to run as one batch                     |
| Test Step          | A single action or assertion within a test (e.g., set a field, click a button) |
| Step Configuration | A reusable template that defines how a test step works                         |
| Test Schedule      | Runs a test or suite automatically on a defined schedule                       |

#### What ATF Is Good For

- **Regression testing** — verify existing functionality still works after an upgrade or change
- **Functional testing** — simulate user actions: create records, set values, click buttons, check results
- **Scheduled testing** — run tests automatically and receive results by email

#### What ATF Is NOT For

- **Unit testing** — ATF is not the recommended tool for rapidly changing features (tests break every time the feature changes)
- **Load/performance testing** — ATF cannot simulate high-volume traffic
- **Production testing** — ATF should only be run in **sub-production** instances

> ⚠️ **EXAM TRAP:** ATF tests should **never** be run in production. Always run in sub-production (dev, test, UAT).

> ⚠️ **EXAM TRAP:** The ATF test step to create a user for testing is **"Create a user"** — this creates a temporary user deleted at the end of the test. There is no "Create a role" or "Create a group" test step — those use "Create a record" with the appropriate table.

#### Common ATF Test Steps

| Step                | What it does                                                         |
| ------------------- | -------------------------------------------------------------------- |
| Create a user       | Creates a temp user with specified roles/groups for the test         |
| Impersonate         | Switches to an existing user (does NOT create one)                   |
| Open a form         | Navigates to a record's form                                         |
| Set field values    | Fills in field values on the open form                               |
| Submit a form       | Saves the form                                                       |
| Assert field values | Verifies a field contains an expected value (test passes/fails here) |
| Run server script   | Runs arbitrary server-side code as part of the test                  |

---

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

### REST APIs — Full Deep Dive

The URL structure is predictable:

```
https://instance.service-now.com/api/now/table/{table_name}

GET    /api/now/table/incident          → get all incidents
GET    /api/now/table/incident/{sys_id} → get specific incident
POST   /api/now/table/incident          → create new incident
PUT    /api/now/table/incident/{sys_id} → update incident (full replace)
PATCH  /api/now/table/incident/{sys_id} → update incident (partial update)
DELETE /api/now/table/incident/{sys_id} → delete incident
```

#### Three REST Directions — Know Which Is Which

| API               | Direction             | Use case                                            |
| ----------------- | --------------------- | --------------------------------------------------- |
| Table API         | External → ServiceNow | External system reads/writes ServiceNow records     |
| REST Message      | ServiceNow → External | ServiceNow calls an outbound REST API               |
| Scripted REST API | External → ServiceNow | Custom endpoints YOU build and expose in ServiceNow |

> 💡 **Exam key:**
>
> - External talking **TO** ServiceNow → **Table API**
> - ServiceNow talking **TO** external → **REST Message** (`sn_ws.RESTMessageV2`)
> - Need a **custom endpoint** in ServiceNow → **Scripted REST API**

---

#### Table API — Useful Query Parameters

When calling the Table API from an external system, these query params control what comes back:

| Parameter               | What it does                                | Example                                 |
| ----------------------- | ------------------------------------------- | --------------------------------------- |
| `sysparm_fields`        | Limit which fields are returned             | `?sysparm_fields=number,priority,state` |
| `sysparm_limit`         | Max number of records returned              | `?sysparm_limit=10`                     |
| `sysparm_offset`        | Pagination offset                           | `?sysparm_offset=10`                    |
| `sysparm_query`         | Encoded query to filter results             | `?sysparm_query=priority=1^active=true` |
| `sysparm_display_value` | Return display values instead of raw values | `?sysparm_display_value=true`           |

---

#### Outbound REST — `sn_ws.RESTMessageV2` (ServiceNow → External)

Use this when ServiceNow needs to call **an external REST API**. Configured under **System Web Services > Outbound > REST Message**, then called via script.

```javascript
// ✅ Calling an outbound REST Message via script (e.g., in a Business Rule or Script Include)
try {
  var sm = new sn_ws.RESTMessageV2("MyRESTMessage", "get"); // REST Message name + HTTP Method record
  sm.setStringParameterNoEscape("table", "incident"); // set a variable defined on the REST Message
  sm.setHttpTimeout(10000); // timeout in ms (10 seconds)

  var response = sm.execute(); // synchronous — blocks until response
  var httpStatus = response.getStatusCode(); // e.g., 200
  var body = response.getBody(); // response body as string
  var jsonBody = JSON.parse(body); // parse JSON

  gs.info("Status: " + httpStatus);
  gs.info("Response: " + body);
} catch (ex) {
  gs.error("REST call failed: " + ex.getMessage());
}

// ✅ Async (non-blocking) outbound REST call
var sm = new sn_ws.RESTMessageV2("MyRESTMessage", "post");
sm.setRequestHeader("Content-Type", "application/json");
sm.setRequestBody(JSON.stringify({ key: "value" }));
var response = sm.executeAsync(); // returns immediately
response.waitForResponse(60); // wait up to 60 seconds
var statusCode = response.getStatusCode();
```

**Key `sn_ws.RESTMessageV2` methods:**

```javascript
sm.setEndpoint("https://api.example.com/data"); // override the endpoint URL
sm.setHttpMethod("POST"); // GET, POST, PUT, PATCH, DELETE
sm.setRequestHeader("Authorization", "Bearer token"); // set a single header
sm.setRequestBody('{"key":"value"}'); // set raw body string
sm.setStringParameterNoEscape("param", value); // set a URL/body variable
sm.setQueryParameter("filter", "active"); // add a query string param
sm.execute(); // synchronous execution
sm.executeAsync(); // asynchronous execution
response.getStatusCode(); // HTTP status (200, 404, etc.)
response.getBody(); // response body as string
response.getHeader("Content-Type"); // get a response header
response.haveError(); // true if a transport error occurred
response.getErrorMessage(); // error details if haveError() is true
```

> ⚠️ **EXAM TRAP:** `sn_ws.RESTMessageV2` is for **outbound** calls — ServiceNow calling an external system. It is NOT used to receive inbound requests.

---

#### Scripted REST API — Building Custom Endpoints in ServiceNow

A **Scripted REST API** lets you create a **custom REST endpoint** that external systems can call, beyond what the Table API provides. You define the URL path, HTTP methods, and write the handler logic yourself.

**When to use it:**

- You need a custom URL structure (not `/api/now/table/...`)
- You want to return a transformed/aggregated response
- You need to expose business logic (not just raw table data)
- You want to restrict what callers can access

**How it's structured:**

```
Scripted REST API (parent)
  → defines base path:  /api/x_myco_myapp/myservice
  → Resources (children)
      → each Resource = one URL path + one or more HTTP methods
      → each method has a Script that handles the request and builds the response
```

**Full Example — Creating a custom endpoint:**

```javascript
// Scripted REST API Resource script
// This handles: GET /api/x_myco_myapp/incident_summary/{number}

(function process(/*RESTAPIRequest*/ request, /*RESTAPIResponse*/ response) {
  // 1. Read path parameters (e.g., /incident_summary/{number})
  var incidentNumber = request.pathParams.number;

  // 2. Read query parameters (e.g., ?include_comments=true)
  var includeComments = request.queryParams.include_comments;

  // 3. Read the request body (for POST/PUT)
  var body = request.body.data; // parsed JSON object (if Content-Type is application/json)
  var rawBody = request.body.dataString; // raw string body

  // 4. Read request headers
  var authHeader = request.headers.Authorization;

  // 5. Query ServiceNow data
  var gr = new GlideRecord("incident");
  gr.addQuery("number", incidentNumber);
  gr.query();

  if (!gr.next()) {
    // 6a. Return an error response
    response.setStatus(404);
    response.setBody({ error: "Incident not found", number: incidentNumber });
    return;
  }

  // 6b. Build and return a success response
  var result = {
    number: gr.getValue("number"),
    priority: gr.getDisplayValue("priority"),
    state: gr.getDisplayValue("state"),
    assigned_to: gr.getDisplayValue("assigned_to"),
    opened_at: gr.getValue("opened_at"),
  };

  response.setStatus(200);
  response.setBody(result); // automatically serialized to JSON
})(request, response);
```

**Key `RESTAPIRequest` methods:**

```javascript
request.pathParams.paramName; // path variables defined in the resource URL
request.queryParams.paramName; // URL query string params (?key=value)
request.headers.HeaderName; // inbound request headers
request.body.data; // parsed JSON body (object)
request.body.dataString; // raw body as string
request.getRequestBodyAsStream(); // for large/binary bodies
```

**Key `RESTAPIResponse` methods:**

```javascript
response.setStatus(200); // HTTP status code to return
response.setBody({ key: "value" }); // set response body (object auto-serialized to JSON)
response.setContentType("application/json"); // set response content type
response.setHeader("X-Custom", "val"); // set a response header
response.setError(error); // set an error object on the response
```

**Key `Scripted REST` methods:**

```javascript
// Inside Scripted REST API resource:
request.body.data; // parsed JSON object ✅
request.getBody(); // raw string body
request.getHeader("name"); // request header
request.getQueryParameter("name"); // URL query param
```

**Securing a Scripted REST API:**

| Option                  | How it works                                                          |
| ----------------------- | --------------------------------------------------------------------- |
| Requires Authentication | Checkbox on the API — enforces Basic Auth or OAuth                    |
| ACL on the endpoint     | Create an ACL with operation `REST_Endpoint` for fine-grained control |
| Role check in script    | Use `gs.hasRole("role_name")` inside the resource script              |
| OAuth 2.0               | Configure an OAuth provider and associate it with the API             |

> ⚠️ **EXAM TRAP:** The two REST objects to keep straight:

| Object                | Used in           | Purpose                                     |
| --------------------- | ----------------- | ------------------------------------------- |
| `RESTAPIResponse`     | Scripted REST API | Send a response **TO** an external caller   |
| `sn_ws.RESTMessageV2` | REST Message      | Call **FROM** ServiceNow to external system |

> 🧠 **MEMORY TIP:** **Scripted REST API = you're the server** (ServiceNow receives the call). **REST Message = you're the client** (ServiceNow makes the call).

#### Practice Questions — REST APIs

> 📋 **PRACTICE Q:** An external ticketing system needs to create incidents in ServiceNow by calling a URL. The team wants to expose only specific fields and add custom validation logic. Which ServiceNow feature should they use?

> ✅ **ANSWER:** A **Scripted REST API**. The Table API exposes all fields with no custom logic. A Scripted REST API lets you define exactly what fields are accepted, apply business logic, and return a tailored response.

---

> 📋 **PRACTICE Q:** A Business Rule needs to notify an external webhook URL every time a high-priority incident is created. Which API is used?

> ✅ **ANSWER:** **`sn_ws.RESTMessageV2`** (Outbound REST Message). ServiceNow is making the call to an external system, which means outbound REST.

---

> 📋 **PRACTICE Q:** An external system calls the ServiceNow Table API with `?sysparm_display_value=true`. What changes in the response?

> ✅ **ANSWER:** Fields return their **display values** (human-readable labels) instead of raw database values. For example, `state` returns `"In Progress"` instead of `"2"`, and a reference field returns the referenced record's display name instead of its `sys_id`.

---

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

| Concept                  | What to know                                                                  |
| ------------------------ | ----------------------------------------------------------------------------- |
| Delegated Developer role | Grants development access to a specific scoped app only                       |
| Code Review              | A designated approver must review changes before they are promoted/published  |
| Scope isolation          | Delegated developer cannot modify artifacts outside their assigned app        |
| Why use it               | Allows team-based development without handing out admin access                |
| How to assign            | Open the application record → "Manage Developers" → add users to related list |
| Code Review config       | Per application — enabling it on one app does not affect others               |

### Git / Source Control Integration

ServiceNow Studio supports linking a scoped application to a **Git repository** for proper version control and team collaboration.

#### Linking to a Git Repo — What You Need

| Required            | NOT Required     |
| ------------------- | ---------------- |
| Repository URL      | Application name |
| Username            |                  |
| Password / token    |                  |
| Branch name         |                  |
| Authentication type |                  |

> ⚠️ **EXAM TRAP:** The **application name is not required** to link to a Git repo — the scope name is used automatically. A common distractor is listing the app name as required.

#### Source Control Operations — Know Which Work Where

Some operations are available from **Studio only**, some from **the Git repo side only**, and some from **both**:

| Operation          | Available in Studio | Available in Git repo  |
| ------------------ | ------------------- | ---------------------- |
| Create Branch      | ✅                  | ✅ (available in both) |
| Create Tag         | ✅                  | ✅ (available in both) |
| Create Repository  | ✅                  | ✅                     |
| Create Credentials | ✅                  | ❌                     |
| Grant Access       | ❌                  | ✅                     |
| Stash Changes      | ✅                  | ❌                     |
| Pull               | ✅                  | ❌                     |
| Push               | ✅                  | ❌                     |

> ⚠️ **EXAM TRAP:** **Create Branch** is the operation available in **both** Studio and the Git repository. This is a direct exam question — "which operation is common to both?"

#### The Stash Operation

**Stash** stores local uncommitted changes on the instance temporarily so you can switch branches or pull updates without losing your work.

> 💡 **Stash = save your work in a drawer** so you can do something else, then come back and "pop" the stash to restore it.

> ⚠️ **EXAM TRAP:** The exam asks what the source control "stash" operation does. Answer: **stores local changes on the instance for later application** — it does NOT commit or push to the repo.

#### Roles Required for Source Control

Users need either the **`admin`** or **`source_control`** role to link an application to a Git repository.

---

### Application Files Related List

To see **all artifacts that will be included when publishing an application**, use the **Application Files Related List** on the application record.

> ⚠️ **EXAM TRAP:** This is a direct exam question. The correct answer is always **"Examine the Application Files Related List in the application to be published"** — NOT the Global search bar, NOT Update Sets, NOT opening each artifact individually.

The Application Files list groups artifacts by type (Business Rules, Client Scripts, Tables, etc.) and shows the scope each belongs to. It's the single source of truth for what's in the app.

---

### Record Producer — Additional Notes

> 💡 **Record Producer variable access syntax:** In a Record Producer's script, access form variable values using `producer.variable_name` (NOT `current.variable_name`). This is a direct exam question.

```javascript
// ✅ CORRECT — Record Producer script accessing variables
var shortDesc = producer.short_description; // variable on the Record Producer form
current.short_description = shortDesc; // set on the target record
current.caller_id = gs.getUserID();
current.insert();
```

> ⚠️ **EXAM TRAP:** In Inbound Action scripts (not Record Producer), the available objects are `current` and `email` — NOT `previous`, NOT `event`, NOT `producer`.

---

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

| Object / API          | Client or Server        | Purpose                                               | Key Gotcha                                          |
| --------------------- | ----------------------- | ----------------------------------------------------- | --------------------------------------------------- |
| `g_form`              | Client only             | Get/set field values, visibility, mandatory on a form | Not available in Business Rules                     |
| `g_user`              | Client only             | Current logged-in user info                           | Use `gs` on server side instead                     |
| `current`             | Server only             | GlideRecord in a BR representing the current record   | Do not call `current.update()` in a Before BR       |
| `previous`            | Server only             | Pre-update field values in a Business Rule            | Compare with `current` to detect changes            |
| `GlideRecord`         | Server only             | Query / insert / update any table                     | Use `getValue()`, not `gr.field_name` directly      |
| `GlideAjax`           | Client → Server         | Call a server Script Include from a client script     | Script Include must extend `AbstractAjaxProcessor`  |
| `gs` (GlideSystem)    | Server only             | Logging, user info, system properties                 | `gs.getUser()` returns a GlideUser object           |
| `g_scratchpad`        | Bridge: server → client | Pass data from Display BR to client scripts           | Set in Display BR, read in client script            |
| Script Include        | Server                  | Reusable server-side functions                        | Set "Accessible from" for cross-scope use           |
| `sn_ws.RESTMessageV2` | Server only             | Make outbound REST calls to external systems          | ServiceNow is the CLIENT calling an external API    |
| `RESTAPIRequest`      | Server only             | Read inbound data in a Scripted REST API resource     | Available as `request` variable in resource script  |
| `RESTAPIResponse`     | Server only             | Send response from a Scripted REST API resource       | Available as `response` variable in resource script |

> 💡 **Memory trick:** Everything starting with `g_` is **client-side**. `gs`, `gr`, and `sn_ws` are **server-side**.

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

### REST API Quick Reference

| Tool                  | Direction             | When to use                                        |
| --------------------- | --------------------- | -------------------------------------------------- |
| Table API             | External → ServiceNow | External system does basic CRUD on standard tables |
| Scripted REST API     | External → ServiceNow | Custom logic, custom URL, aggregated/filtered data |
| `sn_ws.RESTMessageV2` | ServiceNow → External | ServiceNow needs to call an external API           |

### Script Include Quick Reference

- **Server-side only** reusable logic
- Called from: Business Rules, other Script Includes, Scheduled Jobs, REST APIs
- Called from Client Scripts via **GlideAjax** (must extend `AbstractAjaxProcessor`)
- **Regular** → server-side only
- **Ajax-enabled** → extends `AbstractAjaxProcessor`, callable from Client Scripts via GlideAjax

---

## All Exam Traps — Combined List

| Scenario / Symptom                          | Trap                                     | Correct Answer                                                                                                               |
| ------------------------------------------- | ---------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| Form is slow after save                     | Thinking it's a client issue             | Check for heavy After BR — consider Async instead                                                                            |
| Field not saving despite no error           | Checking only UI settings                | Check dictionary read-only flag                                                                                              |
| Client Script needs DB data                 | Using GlideRecord in client script       | Use GlideAjax + client-callable Script Include                                                                               |
| Mandatory not enforced on import            | Using UI Policy                          | Use Data Policy — it's server-side                                                                                           |
| Cross-scope call fails silently             | Assuming it's a code bug                 | Check "Accessible from" on the Script Include                                                                                |
| User sees list, can't open record           | Checking field-level ACLs first          | Record-level ACL is blocking — check that first                                                                              |
| User can't see list at all                  | Checking record/field ACLs               | Table-level ACL is blocking — start there                                                                                    |
| Button on form needs server logic           | Using a Business Rule                    | Use a UI Action with Form Button checked                                                                                     |
| Field change needs silent server call       | Using a UI Action                        | Use GlideAjax + client-callable Script Include                                                                               |
| Move config between instances               | Manually recreating scripts              | Use Update Set or App Repository                                                                                             |
| Artifact in wrong scope                     | Blaming the script                       | Check App Picker — was the correct scope selected at creation?                                                               |
| BR causing infinite loop                    | Adding more conditions to the BR         | Remove `current.update()` from the Before BR                                                                                 |
| `onCellEdit` used on a form                 | Thinking it works on forms               | `onCellEdit` is list view only — use `onChange` on forms                                                                     |
| `security_admin` assumed = `admin`          | Thinking `security_admin` bypasses ACLs  | Only `admin` bypasses ACLs — `security_admin` manages them                                                                   |
| Update Set expected to include data records | Expecting incident records in Update Set | Update Sets capture config only, not data                                                                                    |
| UI Policy expected to enforce on import     | Using UI Policy for import validation    | UI Policies are browser-side — use Data Policy                                                                               |
| Making a field read-only based on role      | Using an ACL                             | ACLs have no "read-only" mode — use Client Script                                                                            |
| `gr.field_name` used for comparison         | Thinking it returns the value            | Returns GlideElement object — use `getValue()` instead                                                                       |
| `getValue()` vs `getDisplayValue()`         | Using wrong one for reference fields     | `getValue()` = sys_id; `getDisplayValue()` = human label                                                                     |
| Delegated developer can't publish           | Assuming they need `admin`               | Check if Code Review is enabled — they need a reviewer to approve                                                            |
| Outbound REST call from a Business Rule     | Using Table API or Scripted REST API     | Use `sn_ws.RESTMessageV2` — ServiceNow is the client here                                                                    |
| Custom external-facing endpoint needed      | Using only the Table API                 | Build a Scripted REST API for custom paths and logic                                                                         |
| `RESTAPIResponse` vs `RESTMessageV2`        | Confusing inbound vs outbound            | `RESTAPIResponse` = respond to caller; `RESTMessageV2` = call out                                                            |
| GAC table option: "from template"           | Thinking it's a valid GAC option         | Not valid — the three options are create, extend, or upload spreadsheet                                                      |
| GAC file upload type                        | Thinking PDF or Word doc can be uploaded | Only spreadsheets (CSV/Excel) are valid upload types in GAC                                                                  |
| GAC creates global apps by default          | Thinking GAC defaults to global scope    | GAC creates scoped apps only — set `sn_g_app_creator.allow_global` to enable global                                          |
| `gs.log()` in a scoped app                  | Using `gs.log()` in scoped scripts       | `gs.log()` is NOT available in scoped apps — use `gs.info()`, `gs.warn()`, `gs.error()`                                      |
| Approval workflows on non-task tables       | Expecting all approvals to work anywhere | Only **User Approval** works on all tables; all others require extending `task`                                              |
| GAC completes the whole application         | Thinking GAC is the complete dev tool    | GAC builds the skeleton only — Business Rules, Client Scripts, flows are added in Studio                                     |
| `Allow configuration` checkbox              | Thinking it's an in-scope-only setting   | Allow configuration = **out-of-scope** apps CAN create Business Rules on the table                                           |
| `Can read` unchecked → other fields gone    | Thinking other fields are independent    | Can create/update/delete become unavailable when Can read is unchecked                                                       |
| `Allow web services` = bypasses ACLs        | Thinking it grants open access           | Still requires correct ACL permissions — only controls whether table is web-accessible                                       |
| Module link type includes "Catalog Type"    | Selecting Catalog Type or Roles          | Not valid link types — valid ones: Assessment, List of Records, Separator, Timeline Page, Content Page, URL (from arguments) |
| Application Properties = special artifact   | Thinking they're a unique object type    | They are ordinary `sys_properties` records grouped by a category matching the app                                            |
| UI Actions always run server-side           | Assuming no client code possible         | If Client checkbox is checked, the onclick runs in the browser first                                                         |
| `gsftSubmit` third arg = display name       | Using the button label as the action arg | Third arg must match the **Action name** field exactly, not the display name                                                 |
| Removing field from form = deletes field    | Thinking layout removal = schema change  | Removing from layout hides it on the form only — field still exists in the table                                             |
| `gs.log()` = client debugging tool          | Including it in client debug strategies  | `gs.log()` is server-side — NOT a client debugging strategy (direct exam question)                                           |
| Notification Weight 0 = no email sent       | Thinking zero suppresses the email       | Weight 0 = notification is **always** sent; higher weight wins among competing notifications                                 |
| Notification tabs include "How"             | Listing "How to send" as a tab           | The three tabs are **Who, What, When** — there is no "How" tab                                                               |
| Unregistered events get processed           | Firing event without registering it      | Events must be registered in the Event Registry — unregistered events do nothing                                             |
| ATF runs in production                      | Testing in prod for accuracy             | ATF must only run in **sub-production** instances — never production                                                         |
| ATF "Create a role" step exists             | Using it to set up test roles            | No such step — use "Create a record" on `sys_user_role` table instead                                                        |
| Git link requires application name          | Including app name in required fields    | App name NOT required — URL, username, password, branch, and auth type are required                                          |
| Stash = commit to repo                      | Thinking stash pushes code upstream      | Stash stores local changes on the **instance** temporarily — does NOT push to Git                                            |
| Both Studio and Git: only one operation     | Guessing a Studio-only operation         | **Create Branch** is the operation available in BOTH Studio and the Git repository                                           |
| App artifacts found in Update Sets          | Checking Update Sets to review app files | Use **Application Files Related List** on the app record — Update Sets don't show this                                       |
| Record Producer uses `current.variable`     | Using current to access form variables   | Use `producer.variable_name` — `current` refers to the target record, not the form fields                                    |
| Inbound Action has `previous` object        | Expecting same objects as Business Rules | Inbound Actions only have `current` and `email` — no `previous`, no `event`, no `producer`                                   |

---

## Scripting Pattern

```code
Side:
  g_ prefix    → always client side
  gs.          → always server side
  gr.          → always server side

Fields:
  .getValue()        → raw DB value / sys_id
  .getDisplayValue() → human readable
  .nil()             → is it empty?
  .changed()         → did it change?

Reference fields:
  current.field.method()    → dot INTO the field first!!

Catalog:
  producer.variablename     → what user filled out
  current.field             → what gets saved

Dates:
  gs.now()          → date only
  gs.nowDateTime()  → date + time
  gs.daysAgo(n)     → n days ago
```

---

## User Patter

```code
CLIENT SIDE:
g_user.userID          → sys_id
g_user.userName        → username
g_user.firstName       → first name
g_user.hasRole()       → check role
g_user.hasRoleExactly() → strict role check

SERVER SIDE:
gs.getUserID()         → sys_id
gs.getUserName()       → username
gs.getUser().getFirstName() → first name
gs.hasRole()           → check role
gs.hasRoleExactly()    → strict role check

The pattern is IDENTICAL — just different prefix!!

Client → g_user.property
Server → gs.method()

The one gotcha:
g_user.userID          → property (no parentheses!!)
gs.getUserID()         → method (parentheses!!)

Quick explanations:
 Q: Get the sys_id of the current logged in user (server side)
 g_user.userID → client side only!! ❌
 gs.getUserID() → ✅ correct!!
 gs.getCurrentUser() → returns GlideUser OBJECT not sys_id ❌
 gr.getUserID() → doesn't exist!! ❌

The pattern:
gs.getUserID()        → sys_id (string)
gs.getCurrentUser()   → GlideUser object
gs.getUserName()      → username string

Memory trick:

g_user → properties you ACCESS like an object
gs → methods you CALL like a function
```

1. The prefix rule saves everything:

```code
   g\_ → client side
   gs. → server side, system methods
   gr. → GlideRecord operations
```

2. Parentheses rule:

```code
   g_user.userID → no parentheses (property)
   gs.getUserID() → parentheses (method)
```

3. When two answers look identical:
   Ask yourself:
   - Server or client context?
   - Property or method?
   - Field level or record level?

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
- **ServiceNow calling an external API?** → `sn_ws.RESTMessageV2`
- **External system calling a custom ServiceNow endpoint?** → Scripted REST API
- **Delegated developer stuck, can't publish?** → Code Review is enabled — needs a reviewer
- **Delegated developer accessing another app's artifacts?** → Not possible — scope isolation
- **What's in my published app?** → Application Files Related List (not Update Sets, not Global search)
- **Email notification question about tabs?** → Who / What / When (NOT How)
- **Notification Weight 0?** → Always sent; Weight > 0 → only highest wins
- **Fire an event?** → `gs.eventQueue()` — must be registered in Event Registry first
- **ATF for testing?** → Sub-production only, regression and functional testing only
- **Git stash?** → Stores local changes on the instance, does NOT push to repo
- **Which operation in both Studio and Git?** → Create Branch
- **Allow configuration on a table?** → Out-of-scope apps CAN create Business Rules on it
- **Can read unchecked?** → Can create/update/delete become unavailable
- **Which tool to debug client-side?** → jslog(), g_form.addInfoMessage(), Field Watcher (NOT gs.log)
- **Record Producer variable access?** → `producer.variable_name` (NOT `current.variable_name`)
- **Date/time in scoped app?** → GlideDateTime (not GlideDate; datediff not available in scoped)

---

## Mnemonics & Memory Tricks

- **Alert = JavaScript = Client Script** (alert() is a JS method → it's client side!)
- **`g_` prefix = client side** (g_form, g_user, g_scratchpad)
- **`gs` = system/session** (server side)
- **`gr` = records/database** (server side)
- **`sn_ws` = web services outbound** (server side — ServiceNow calling out)
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
- **Scripted REST API = you're the server** (ServiceNow receives) | **REST Message = you're the client** (ServiceNow sends)
- **Delegated Developer = room key, not master key** (one app only)
- **GAC = foundation pour** (sets up structure; Studio builds the house on top)
- **GAC table options = Create, Extend, Spreadsheet** (no templates, no PDFs, no Word docs)
- **`gs.log()` = global only** — use `gs.info()`, `gs.warn()`, `gs.error()` in scoped apps
- **User Approval = any table | All other approvals = must extend `task`**
- **Notification tabs = Who + What + When** (no "How")
- **Weight 0 = always send | Weight > 0 = only highest weight wins**
- **Events = fire and forget; registry = the subscriber list**
- **ATF = automated regression testing, sub-production only**
- **Stash = save in a drawer (local instance), not pushed to Git**
- **Create Branch = the operation in BOTH Studio and Git**
- **Application Files Related List = what's in my app before publishing**
- **Allow configuration = OUT-of-scope apps can write Business Rules to your table**
- **Can read unchecked = Can create/update/delete disappear**
- **Module link types: Assessment, List of Records, Separator, Timeline Page, Content Page, URL** (no Catalog Type, no Roles)
- **producer.variable_name** (Record Producer) vs **current.field_name** (Business Rule)

---

## 7–10 Day Study Plan

| Days | Focus                                                                              | Goal                                                                    |
| ---- | ---------------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| 1–2  | Domain 1 & 2: Scoped apps, tables, client scripts, UI Policy, GlideRecord          | Build a small scoped app with 2 tables, client scripts, UI Policies     |
| 3–4  | Domain 3 & 4: ACLs, Business Rules, Flow Designer, Scheduled Jobs                  | Create ACLs at all 3 levels, all 4 BR types, observe timing differences |
| 5    | Domain 2 deep dive: GlideAjax, UI Actions, `g_scratchpad`, Script Includes         | Build a GlideAjax call from scratch — this is exam-heavy content        |
| 6    | Domain 5 & 6: Import Sets, Transform Maps, Update Sets, App Repository             | Import a CSV, test Data Policy, export and import an Update Set         |
| 6.5  | Domain 6 deep dive: Scripted REST API, REST Message, Delegated Development         | Build a Scripted REST API resource; practice outbound REST call script  |
| 7    | Timed practice: 60 questions in 90 minutes                                         | Identify weak areas — revisit those domain sections                     |
| 8    | Quick-reference drills: ACL levels, BR timing, prefix table, client vs server APIs | Can you recall all tables and flows without looking?                    |
| 9–10 | Light review, mental walkthroughs, exam-day confidence                             | Re-read all exam traps, sleep well, trust your prep                     |
