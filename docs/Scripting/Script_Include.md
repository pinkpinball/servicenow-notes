# Script Include

## Purpose:

- Reusable server-side logic

## Runs:

- Server
- Called by other scripts

## Use When:

- Logic reused across BRs, Flows, APIs
- NOT for UI logic

## Touches:

- Tables: varies
- APIs: GlideRecord, gs

## Common Mistakes:

- Putting business logic directly in BRs
- Forgetting client-callable flag

1-Min Example:

- calculate SLA based on priority
