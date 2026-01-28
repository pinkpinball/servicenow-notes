# Scheduled Script

## Purpose:

- Run logic on a schedule

## Runs:

- Server
- Time-based (cron)

## Use When:

- Cleanup jobs
- Batch updates
- NOT for real-time logic

## Touches:

- Tables: any
- APIs: GlideRecord, gs

## Common Mistakes:

- Heavy logic during business hours
- No logging

1-Min Example:

- Nightly close stale incidents
