# ServiceNow Fundamentals

This document is a **platform-wide mental model** of ServiceNow.

It answers three questions:

> **What is ServiceNow?**
> **How do the pieces fit together?**
> **How do I know which tool to use?**

---

## What is ServiceNow?

ServiceNow is a **cloud-based enterprise platform** used to:

- manage work
- store and relate data
- enforce business rules
- automate processes
- present controlled user experiences

At its core, ServiceNow is:

> **A relational database + execution engine + UI framework**

Everything you configure maps to **one or more** of these layers.

> **Everything in ServiceNow is a record in a table.**
> **Every record is represented as a JavaScript object at runtime.**

---

## Core Platform Architecture

### The Three Layers

| Layer       | Responsibility                         |
| ----------- | -------------------------------------- |
| Data Layer  | Stores records in tables               |
| Logic Layer | Enforces rules and automation          |
| UI Layer    | Displays and controls user interaction |

Every feature you use lives in **at least one** of these layers.

---

## Data Layer

### Tables

- Tables store records (rows)
- Tables contain fields (columns)
- Tables can **extend** other tables (inheritance)

**Example:**

- `task` (parent)
- `incident` (extends `task`)

> Child tables inherit all parent fields automatically.

---

### Records

- A single row in a table
- Represent a unit of work or data

**Examples:**

- One incident
- One request item
- One user
- One CI

---

### Fields

- Attributes of a record
- Defined in the **Dictionary**
- Common types:
  - String
  - Reference
  - Choice
  - Date/Time
  - Boolean

**Examples:**

- `state` (choice)
- `caller_id` (reference)

---

### Reference Fields & Relationships

- Reference fields point to another table
- Enable **dot-walking**

**Example:**

```js
incident.caller_id.email;
```
