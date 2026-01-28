# ServiceNow Fundamentals

This document is a **platform-wide overview** of ServiceNow. It’s meant to answer:

> _What is this thing, how do the pieces fit together, and when do I use what?_

---

## What is ServiceNow?

ServiceNow is a **cloud-based platform** for:

- managing work
- enforcing business rules
- automating processes
- storing and acting on data

At its core, ServiceNow is:

> **A relational database + workflow engine + UI layer**

Everything you do maps to one (or more) of those layers.

---

## Core Platform Concepts

### Tables

- Tables store records (rows)
- Tables contain fields (columns)
- Tables can extend other tables

**Example:**

- `task` → parent table
- `incident` → extends `task`

> Inheritance means child tables automatically get parent fields.

---

### Records

- A single row in a table
- Represent real-world work items

**Example:**

- One incident ticket
- One request
- One user

---

### Fields

- Attributes of a record
- Can be:
  - string
  - reference
  - choice
  - date/time

**Example:**

- `caller_id` (reference)
- `state` (choice)

---

### Reference Fields

- Point to another table
- Enable dot-walking

**Example:**

```
incident.caller_id.email
```

> Reference fields create relationships between tables.

---

## UI Layer

### Forms

- Used to view/edit a single record
- Display fields from a table

**Example:**

- Incident form

---

### Lists

- Table views showing many records
- Filterable and sortable

**Example:**

- Incident list view

---

### UI Policies

- Declarative UI rules
- No scripting required

**Use for:**

- show/hide fields
- mandatory
- read-only

---

### Client Scripts

- Client-side JavaScript
- Enhance form behavior

**Types:**

- onLoad
- onChange
- onSubmit

---

## Server Layer

### Business Rules

- Server-side logic
- Run on database operations

**Types:**

- Before
- After
- Async

---

### Script Includes

- Reusable server-side logic
- Called by other scripts

**Use for:**

- shared utilities
- centralized logic

---

### Glide APIs

- ServiceNow server-side APIs

**Common ones:**

- GlideRecord
- GlideSystem (`gs`)
- GlideDateTime

---

## Process Automation

### Flow Designer

- No/low-code automation tool
- Trigger-based workflows

**Use for:**

- approvals
- notifications
- record updates

---

### Workflows (Legacy)

- Older automation engine
- Still used in some instances

> Prefer Flow Designer for new work.

---

## Security & Access

### Roles

- Control access to features

**Example:**

- `admin`
- `itil`

---

### ACLs (Access Control Lists)

- Control access to tables and fields

**Example:**

- Read/write restrictions

---

## Application Development

### Application Scope

- Logical boundary for apps
- Controls access and isolation

---

### App Engine Studio

- Guided app creation
- Uses tables, flows, scripts

---

## Reporting & Analytics

### Reports

- Visualize data
- Based on tables

**Types:**

- bar
- pie
- list

---

### Dashboards

- Collections of reports

---

## Data Management

### Import Sets

- Bring data into ServiceNow
- Transform maps map source → target

---

### CMDB (Configuration Management Database)

- Stores configuration items (CIs)
- Tracks relationships

---

## Update Sets

- Capture configuration changes
- Move changes between instances

---

## Integration Basics

### REST / SOAP APIs

- External system communication

---

## Mental Model (Remember This)

- **Tables store data**
- **Forms & Lists show data**
- **Scripts enforce behavior**
- **Flows automate processes**
- **ACLs protect data**

---

## Final Advice

If you ever feel lost, ask:

1. Where is the data?
2. Who is changing it?
3. Does this need to always run?

Those questions will guide you to the correct tool every time.
