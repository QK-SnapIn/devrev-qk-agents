# Discovery Questions — Workflow

Use after `domain-classifier.md` resolves to `workflow`. Ask in focused rounds (3–5 questions per round).

---

## Round 1 — Trigger

1. What kicks this workflow off?
    - Manual (a user clicks "run")
    - Scheduled (e.g., every Monday 9am IST)
    - On an event (a record changes, a record is created)
2. If event-driven: which object? (`ticket`, `conversation`, `account`, custom)
3. If event-driven: what condition triggers? (e.g., severity changes to P0; status moves to closed)
4. Is this a new workflow or a fix/extension of an existing one?

## Round 2 — Inputs

1. What data does the workflow have access to when it starts? (the triggering record, related records, user input)
2. Are any inputs supplied by the user at run time (manual workflow)?
3. Any external data needed? (Slack channels, account IDs, contact info)

## Round 3 — Steps

1. Walk me through the steps in order. For each, what does it do?
   - Common building blocks: send notification, update field, create record, comment, branch, loop.
2. Which steps run for every iteration vs. once total?
3. Does the order matter? (some are sequential, some can parallelize)

> Tip: run `bin/devrev-api list-operations --filter <keyword>` to see what operations are actually available in the user's tenant before committing to a step.

## Round 4 — Branches & loops

1. Any conditional logic? (`if priority == high, also notify Slack`)
2. Any iteration over a list? (`for each ticket linked to this account, do X`)
3. Any "skip if already done" guard?

## Round 5 — Outputs / side effects

1. What does the workflow change in DevRev (fields updated, records created, comments added)?
2. What does it send out (Slack, email, webhook)?
3. Any logging or audit-trail needs?

## Round 6 — Failure handling

1. If a step fails, what should happen?
    - Retry (how many times?)
    - Alert someone (who?)
    - Continue with remaining steps
    - Fail the whole workflow loudly
2. Is partial completion acceptable?

---

## Common workflow shapes (quick patterns)

| Shape | Trigger | Steps | Example |
|---|---|---|---|
| Reminder | scheduled | for_each ticket → check stale → send Slack | weekly enhancement status |
| SLA escalation | event (severity changed) | update field → notify owner | severity-to-SLA mapping |
| Auto-close | scheduled | for_each ticket → check criteria → close | weekly auto-close completed tickets |
| Bulk update | manual | for_each in user-supplied list → update field | bulk reassign on team change |
