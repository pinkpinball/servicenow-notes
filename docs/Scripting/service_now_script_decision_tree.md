# ServiceNow: Which Tool Do I Use?

This page is your **mental shortcut** for choosing the _right_ tool on the Now Platform.

If you memorize nothing else, memorize this flow.

---

```
mermaid
flowchart TD
A[Where must the logic run?]

A -->|Always / Server-side| BR[Business Rule]

A -->|UI only| B[Is it form behavior?]

B -->|Yes| C[Can it be done without JavaScript?]
C -->|Yes| UIP[UI Policy]
C -->|No| CS[Client Script]

B -->|No| D[Is this a multi-step process or automation?]
D -->|Yes| FD[Flow Designer]
D -->|No| E[Is the logic reused in multiple places?]
E -->|Yes| SI[Script Include]
E -->|No| R[Re-evaluate requirements]
```

---

## Step 1: Where does this logic need to run?

### Does it need to run **no matter how the record is changed**?

- UI
- API
- Import
- Background script

➡️ **YES** → **Business Rule**

➡️ **NO (UI only)** → Go to Step 2

---

## Step 2: Is this only about form behavior (UX)?

Examples:

- Show / hide fields
- Make fields mandatory or read-only
- React to user input

➡️ **YES** → Go to Step 3

➡️ **NO** → Go to Step 4

---

## Step 3: Can this be done without JavaScript?

Ask yourself:

- Is the logic simple?
- Is it condition-based?
- No calculations?

➡️ **YES** → **UI Policy**

➡️ **NO** → **Client Script**

---

## Step 4: Is this process-driven rather than record-driven?

Examples:

- Multi-step logic
- Approvals
- Notifications
- Cross-table actions

➡️ **YES** → **Flow Designer**

➡️ **NO** → Go to Step 5

---

## Step 5: Does this logic need to be reused across scripts?

Examples:

- Shared utility functions
- Reusable business logic
- Centralized calculations

➡️ **YES** → **Script Include**

➡️ **NO** → Re-evaluate requirements

---

## Quick Summary Table

| Tool           | Runs   | Best For           | Avoid When          |
| -------------- | ------ | ------------------ | ------------------- |
| UI Policy      | Client | Simple UI rules    | Logic, calculations |
| Client Script  | Client | Complex UI logic   | Data enforcement    |
| Business Rule  | Server | Data integrity     | UI behavior         |
| Flow Designer  | Server | Process automation | Fine-grained logic  |
| Script Include | Server | Reusable logic     | One-off rules       |

---

## One-Line Rules (Memorize These)

- **If it must always happen → Business Rule**
- **If it’s UI and simple → UI Policy**
- **If it’s UI and complex → Client Script**
- **If it’s a process → Flow Designer**
- **If logic is reused → Script Include**

---

## Common Anti-Patterns

❌ UI logic in Business Rules
❌ Business logic in Client Scripts
❌ Client Scripts when UI Policies work
❌ Flows for single-field validation
❌ Copy-pasting logic instead of Script Includes

---

## Final Mental Model

> **Client = experience**
> **Server = truth**
> **Flows = orchestration**

When in doubt, choose the tool that:

- runs in the right place
- has the smallest scope
- enforces the rule at the correct layer
