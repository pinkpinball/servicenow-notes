# UI Policy

## Purpose

- Control **simple form behavior** without scripting
- Provide declarative UI rules

> UI Policies exist to handle common UI logic cleanly and safely.

---

## Runs

- **Client**
- Evaluated:
  - on form load
  - when watched fields change

---

## Use When

- Show / hide fields
- Make fields mandatory
- Make fields read-only
- Logic is straightforward and rule-based

**Do NOT use when:**

- Calculations are required
- Complex conditional logic is needed
- Cross-field math or comparisons
- Server-side enforcement is required

---

## Components

### UI Policy

- Condition-based rule
- Determines _when_ actions apply

### UI Policy Actions

- Define _what_ happens
  - Visible
  - Mandatory
  - Read-only

> UI Policies can exist **without scripting**

---

## Touches

- **Tables**: current form table
- **Fields**: via UI Policy Actions
- **APIs**: none (unless using UI Policy Scripts)

---

## UI Policy vs Client Script

UI Policy:

- Declarative
- No JavaScript
- Easier to maintain
- Faster performance

Client Script:

- JavaScript-based
- More powerful
- Higher risk
- More flexible

> Always try UI Policy first

---

## Common Mistakes

- Using Client Scripts when UI Policy is sufficient
- Overlapping multiple UI Policies
- Forgetting UI Policy order matters
- Mixing UI Policies and Client Scripts unnecessarily

---

## 1-Min Example

**Requirement:**
If category = Hardware → make serial number mandatory

**Where:**
UI Policy on the table

**How:**

- Condition: category == Hardware
- UI Policy Action: serial_number → mandatory = true

---

## Mental Shortcut

> **UI Policy = simple UI rules**
> **Client Script = advanced UI logic**
