# ServiceNow Flow Fundamentals

This document provides a **clear, practical overview of Flow Designer** on the Now Platform.

Use this to understand:

- what flows are
- when to use them
- how they differ from scripts

---

## What is Flow Designer?

Flow Designer is ServiceNow’s **low-code / no-code automation engine**.

It allows you to:

- react to events
- orchestrate processes
- automate work across tables

> Think of Flow Designer as the _process layer_ of ServiceNow.

---

## When Should I Use a Flow?

Use Flow Designer when:

- multiple steps are involved
- approvals are required
- notifications must be sent
- actions span multiple records or tables

**Do NOT use when:**

- logic must run at save time
- data integrity is critical
- the logic is a simple field rule

> If it must always run → Business Rule

---

## Core Flow Components

### Trigger

Defines **when the flow starts**.

Common triggers:

- Record created
- Record updated
- Schedule
- Service Catalog events

**Example:**

- When an Incident is created

---

### Conditions

Optional checks that control whether the flow continues.

**Example:**

- Priority = 1

---

### Actions

Steps performed by the flow.

Common actions:

- Create Record
- Update Record
- Send Notification
- Ask for Approval

---

### Data Pills

- Dynamic values passed between steps
- Represent fields, records, or outputs

> Data pills replace variables in low-code flows.

---

## Flow Execution Context

- Runs on the **server**
- Executes asynchronously
- Does not block form submission

> Flows are not guaranteed to run instantly.

---

## Flow vs Script Comparison

| Tool          | Best For           | Runs   | Timing    |
| ------------- | ------------------ | ------ | --------- |
| Business Rule | Data enforcement   | Server | Immediate |
| Flow Designer | Process automation | Server | Async     |
| Client Script | UX logic           | Client | Immediate |

---

## Flow Logic Patterns

### Linear Flow

- Step-by-step execution

**Example:**

- Create incident → notify group → update state

---

### Conditional Branching

- Different paths based on conditions

**Example:**

- Priority 1 → notify manager
- Else → standard notification

---

### Looping

- Repeat actions for multiple records

**Example:**

- Notify all affected users

---

## Flow Designer vs Workflow (Legacy)

- Flow Designer is the modern replacement
- Workflows still exist in older instances

> Always prefer Flow Designer for new development.

---

## Error Handling

- Flow errors can be logged
- Some actions provide failure outputs

**Best Practice:**

- Design flows to fail gracefully

---

## Performance Considerations

- Avoid excessive flows on high-volume tables
- Keep flows focused on process, not validation
- Prefer scripts for tight logic loops

---

## Common Mistakes

- Using flows for field validation
- Expecting flows to run synchronously
- Overcomplicating simple logic
- Duplicating logic that belongs in scripts

---

## 1-Min Example

**Requirement:**
When an incident with priority 1 is created:

- notify on-call group
- request manager approval

**Solution:**

- Trigger: Incident created
- Condition: Priority = 1
- Actions:
  - Send notification
  - Ask for approval

---

## Relationship to Other Tools

- **UI Policy**: simple UI rules
- **Client Script**: complex UI behavior
- **Business Rule**: data enforcement
- **Flow Designer**: process orchestration

---

## Mental Model (Memorize This)

> **Flows move work forward**
> **Scripts enforce rules**

---

## Final Advice

If you’re unsure whether to use a Flow, ask:

1. Is this a process with multiple steps?
2. Does it need approvals or notifications?
3. Is async execution acceptable?

If yes → Flow Designer is likely the right choice.
