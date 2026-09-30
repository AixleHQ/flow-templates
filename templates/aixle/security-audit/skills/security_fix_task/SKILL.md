---
name: security-fix-task
description: How to turn a verified security finding into a fix task card on the board, and how a fix step implements one (regression test first, minimal fix, then a pull request or a patch). Use in the report step of a security audit when creating fix cards, and in the fix workflow when implementing a card.
---

# Security fix task

## Creating a fix card (report step)

Create one card per verified finding with `board_create_task` in the column
**Fix Backlog**. Never create a card for a FALSE_POSITIVE, EXCLUDED or
MERGED candidate.

Re-audits must not duplicate cards. Before creating a card, run `board_list_tasks` on
Fix Backlog, Fixing, Fix In Review, Done and Won't Fix. A finding already has a
card when a card with the same CWE, file and symbol exists, whatever its SA
number. In that case, comment on the existing card instead of creating a new one.

- **title**: `[<SEVERITY>] <short title> (<CWE>)`, for example `[HIGH] Order lookup returns any user's order (CWE-639)`
- **priority**: set it to the severity, one to one: critical→`critical`,
  high→`high`, medium→`medium`, low→`low`.
- **tags**: `security`, `sev:<severity>`, `<class>`, `audit:<audit card id>`
- **assignee**: the audit card's assignee. A fix run is billed to the card's
  assignee, and a card without one cannot start the fix workflow.
- **description** (markdown, exactly these sections):

```markdown
## Finding
<finding_id> from audit #<audit card id>. <2–3 sentence description of the bug and impact.>
Scanner evidence: <related SCAN ids and tools, or "none">

## Where
- `file:line` — symbol (one line per location)

## Attack path
<source -> sink, one hop per line, file:line each>

**Precondition:** <anonymous / any user / admin ...>
**Exploit sketch:** <request/input and observable result>

## Classification
Severity: <severity> · Confidence: <0.x> · <CWE> · <OWASP 2025 id>

## Fix
<remediation, concrete: what to change and where. Name the idiomatic
framework mechanism when one exists.>

## Regression test
<the test that must fail before the fix and pass after>

## Acceptance
- [ ] Regression test added and failing on the unfixed code
- [ ] Fix applied; regression test passes; existing tests pass
- [ ] Variants listed below checked
```

Secret values never go on a card. For a leaked secret, the fix section says
"rotate the credential" first, because removing it from git history does not
un-leak it.

Then add a comment on the **audit** card listing every fix card created (id + title).

## Implementing a fix card (fix workflow)

1. Read the card (`board_get_task`) and its comments. If the finding no longer
   reproduces on the current default branch, comment with the evidence, move the
   card to **Won't Fix**, and stop.
2. Create branch `security/<card id>-<short-slug>` from the default branch.
3. Write the regression test from the card first and run it: it must fail. If
   the project has no runnable test setup, write the test anyway and state in
   the delivery that it was not executed.
4. Apply the smallest fix that closes the root cause (not only the reported
   instance). Check the listed variants.
5. Run the regression test and the relevant existing suite. Both must be green.
6. Commit with `fix(security): <title> (<CWE>)`.
7. Deliver the fix:
   - **Private repository that accepts pushes:** open a PR. Its body is the
     card's Finding / Attack path / Fix sections plus "Test evidence" with the
     before/after test output.
   - **Public or read-only repository:** never push. Attach
     `git format-patch` output to the card, and ask a human to land it through
     a private path (security advisory / private fork).
8. Comment the PR link or patch name on the card and move it to **Fix In Review**.
