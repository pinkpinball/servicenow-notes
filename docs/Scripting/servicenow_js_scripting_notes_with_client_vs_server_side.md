# ServiceNow JavaScript: Server Side vs Client Side

This document consolidates **all major scripting types and JS usage on the Now Platform** into one
reference. Use it as your active note-taking and recall guide.

---

## Universal JS on Now Platform Template

```
Feature:

Purpose:
- Why this script exists
- What problem it solves

Runs:
- Client | Server | Both
- Trigger / Event / Flow

APIs / Globals:
- List of platform APIs (g_form, GlideRecord, GlideAjax, GlideFlow, etc.)

Inputs:
- Fields, variables, Flow inputs, or app variables

Outputs:
- Updated fields, records, logs, Flow outputs

Use Cases:
- Concrete scenarios where this should be used
- Where NOT to use it

Common Mistakes / Gotchas:
- Misfires, infinite loops, security issues, context mistakes

1-Min Example:
- Step-by-step scenario with input → JS → output
```

---

## Client-Side Scripting

### Client Script

```
 Client Script (onChange example)

Purpose:
- Control form field behavior

Runs:
- Client, triggered by field change

APIs / Globals:
- g_form, g_user

Inputs:
- Field value changed

Outputs:
- Field visibility / mandatory / value set

Use Cases:
- Show/hide fields dynamically
- Mandatory logic on form
- NOT for DB changes

Common Mistakes / Gotchas:
- Trying to modify database fields directly
- Forgetting runs only in the browser

1-Min Example:
- If “Category” = Hardware → make “Serial Number” mandatory
```

### UI Policy

```
 UI Policy

Purpose:
- Declarative form behavior (low/no script)

Runs:
- Client, on form load/change

APIs / Globals:
- UI Policy Actions

Inputs:
- Field values

Outputs:
- Show/hide fields, mandatory logic

Use Cases:
- Simple form logic without scripts
- NOT for complex validations

Common Mistakes / Gotchas:
- Using script when policy can do it
- Only client-side execution

1-Min Example:
- If category = Hardware → show serial number
```

### UI Action

```
 UI Action

Purpose:
- Add buttons / actions to UI

Runs:
- Client, Server, or Both, on click

APIs / Globals:
- g_form, current

Inputs:
- Button clicked

Outputs:
- Updates fields, triggers logic

Use Cases:
- User-triggered actions
- NOT for background enforcement

Common Mistakes / Gotchas:
- Mixing client/server logic incorrectly
- No condition guarding visibility

1-Min Example:
- "Escalate Incident" button
```

### UI Page

```
 UI Page

Purpose:
- Custom UI with HTML/JS

Runs:
- Client + Server (Jelly)

APIs / Globals:
- GlideAjax, DOM

Inputs:
- Form data, user input

Outputs:
- Custom display, server updates

Use Cases:
- Custom dashboards or forms
- NOT for simple forms

Common Mistakes / Gotchas:
- Reinventing OOB UI
- Security issues

1-Min Example:
- Custom dashboard input form
```

---

## Server-Side Scripting

### Business Rule

```
 Business Rule

Purpose:
- Enforce data rules at save time

Runs:
- Server, Before / After / Async

APIs / Globals:
- current, previous, GlideRecord, gs

Inputs:
- Record fields

Outputs:
- Updated record, notifications, logs

Use Cases:
- Data integrity enforcement
- Audit logs
- NOT for UI behavior

Common Mistakes / Gotchas:
- Doing UI logic
- Wrong timing (Before vs After)
- Infinite loops

1-Min Example:
- If incident.state = Closed → set closed_at timestamp
```

### Script Include

```
Script Include

Purpose:
- Reusable server-side logic

Runs:
- Server, callable by other scripts

APIs / Globals:
- GlideRecord, gs

Inputs:
- Varies by use

Outputs:
- Return value, updated records

Use Cases:
- Reusable logic across BRs, Flows, APIs
- NOT for UI logic

Common Mistakes / Gotchas:
- Putting logic directly in BRs
- Forgetting client-callable flag

1-Min Example:
- Calculate SLA based on priority
```

### Scheduled Script

```
Scheduled Script

Purpose:
- Run logic on a schedule

Runs:
- Server, time-based (cron)

APIs / Globals:
- GlideRecord, gs

Inputs:
- Scheduled trigger

Outputs:
- Updated records, logs

Use Cases:
- Cleanup jobs, batch updates
- NOT for real-time logic

Common Mistakes / Gotchas:
- Heavy processing during business hours
- Missing logging

1-Min Example:
- Nightly close stale incidents
```

### Fix Script

```
Fix Script

Purpose:
- One-time data correction

Runs:
- Server, manual execution

APIs / Globals:
- GlideRecord

Inputs:
- Target records

Outputs:
- Updated records

Use Cases:
- Backfilling data, correcting bad records
- NEVER for ongoing logic

Common Mistakes / Gotchas:
- Running in prod without test
- Forgetting rollback plan

1-Min Example:
- Set missing assignment groups
```

---

## Flow / Automation

### Flow Script Step

```
Flow Script Step

Purpose:
- Custom logic inside Flow Designer

Runs:
- Server, triggered by Flow

APIs / Globals:
- GlideRecord, Flow inputs/outputs

Inputs:
- Flow variables

Outputs:
- Updated Flow variables, records

Use Cases:
- Conditional logic not supported declaratively
- NOT for UI logic

Common Mistakes / Gotchas:
- Overly complex logic inside Flow
- Not returning outputs correctly

1-Min Example:
- If "priority" > 3 → assign to escalation group
```

### App Engine / Scoped App Script

```
Scoped App Script (Server-side)

Purpose:
- Encapsulate app-specific logic

Runs:
- Server, callable from other modules

APIs / Globals:
- GlideRecord, gs, Script Includes

Inputs:
- App variables, table fields

Outputs:
- Updated records, app variables

Use Cases:
- Reusable app logic, integrations
- NOT for unrelated tables

Common Mistakes / Gotchas:
- Forgetting scope protection
- Using global objects incorrectly

1-Min Example:
- Calculate SLA for custom app records
```

### Report Script

```
Report Script

Purpose:
- Customize report logic / data retrieval

Runs:
- Server during report generation

APIs / Globals:
- GlideRecord, gs

Inputs:
- Table data, filters

Outputs:
- Report dataset

Use Cases:
- Advanced filtering, calculated fields
- NOT for UI actions

Common Mistakes / Gotchas:
- Slow queries
- Not respecting permissions

1-Min Example:
- Calculate overdue incidents grouped by assignment group
```

---

## Server vs Client Method Tips

### Quick Guide to Remember

- **g_form** → Client only (form manipulation, field visibility, mandatory fields)
- **g_user** → Client only (user info in browser)
- **current** → Server only (record being saved or queried)
- **previous** → Server only (record state before update)
- **GlideRecord** → Server only (query/update database)
- **gs** → Server only (logs, notifications, user info)
- **GlideAjax** → Client-to-server bridge (client calls server methods)
- **Flow APIs** → Server (Flow Script Step)
- **UI Policy Actions** → Client (show/hide, mandatory, read-only)

### Mental Shortcut

- **Touching the database = server**
- **Touching the form/UI = client**
- **Reusable logic = Script Include / Scoped App**
- **Scheduled/Timed = Scheduled Script**
- **One-time fix = Fix Script**

---

## Key JS Patterns to Memorize

- If it touches the DB → Server
- If it touches the form → Client
- If it’s reusable logic → Script Include / Scoped App
- If it’s a process / workflow → Flow Script Step
- Always check API scope: g_form = client, current = server, GlideAjax = client→server bridge

---

This document is designed for **active note-taking and hands-on reinforcement**. Fill in the `1-Min Example` and `Use Cases` sections with examples from your dev instance to lock in memory.
