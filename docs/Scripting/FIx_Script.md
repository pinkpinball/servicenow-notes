# Fix Script

## Purpose:

- One-time data correction

## Runs:

- Server
- Manual execution

## Use When:

- Backfilling data
- Correcting bad records
- NEVER for ongoing logic

## Touches:

- Tables: target tables
- APIs: GlideRecord

## Common Mistakes:

- Running in prod without test
- Forgetting rollback plan

1-Min Example:

- Set missing assignment groups
