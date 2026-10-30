# Agentic Dev Pipeline

A Kanban delivery pipeline for one application repository, run by agents and
gated by people. A card enters Backlog; the Planner releases it when nothing it
depends on is outstanding and nothing in flight touches the same files; agents
write the tech design and the UI spec, implement on a branch, wait for CI, review
the diff, merge to staging and walk the acceptance criteria in a real browser.
People make product decisions, run hand-run operations on staging, and merge
when they want to.

It is the pipeline Dualboot Partners runs on client and internal projects,
merged from three live projects into one project-agnostic template.

## What you get

- **A 13-column board**: Backlog → Needs Planning → Tech Design → Design →
  Implementation → CI check → Code Review → Merge and deploy to staging →
  After Deploy (Human action) → Verify → Done, with Blocked and Human-gated to
  the side. Every column purpose spells out who moves a card in, what fires,
  and what moves it out.
- **12 agents** with distinct personas: planner, scope manager, intake triager,
  tech lead, UX designer, developer, code reviewer, release engineer,
  acceptance tester, progress reporter, unblock coordinator, card analyst.
- **15 workflows**, 8 of them bound to columns:

| Workflow | Fires on | What it does |
|---|---|---|
| Planner | schedule, every 2 h | Builds the dependency map from the board (and the planning artefacts when you keep them), releases the next independent batch under the WIP cap, assigns round-robin, triages Blocked, re-triggers stuck cards, reports drift. |
| Scope Change | card enters Needs Planning | Slices an ask into buildable cards, triages a bug report, or amends the spec through a pull request; escalates product decisions. |
| Tech Design | card enters Tech Design | Writes the implementation-ready spec (`tech_design` comment) and routes to Design or Implementation. |
| Design | card enters Design | Writes the UI/UX spec from the design source; draws what is missing; big new screens wait for a person's approval. |
| Implementation | card enters Implementation | Branches from the integration branch, implements with tests, opens the PR, creates the CI gate. |
| CI Check | card enters CI check | Reads the gate. Green → Code Review; red → back with the failing job; base red or CI down → Blocked, never blamed on the card. |
| Code Review | card enters Code Review | Adversarial static review: every acceptance criterion maps to a test; blocking findings go back. |
| Merge and deploy to staging | card enters that column | Merges the branch into the staging branch, watches the deploy, proves the app answers, checks the spec for drift, hands over an `ops_runbook` when a person must act. |
| Verify | card enters Verify | Walks every acceptance criterion and edge case in a headless browser against staging, compares screens with the design source, then merges the PR (or leaves it for a person) and closes the card. |
| Blocked Sweep → After Deploy | manual | Classifies everything in Blocked and Human-gated and applies the moves you approve. |
| Task Q&A | manual | Answers a question about a card from the code and the spec, read-only. |
| Chat Intake | chat message containing `triage` | Turns a message into the right board action: an answer, a comment, an unblock, a bug card, a planning item. |
| Progress Report | schedule, twice a week | Evidence-based report for stakeholders: what shipped, how, what is blocked, what is left. |
| Unblock Digest | schedule, weekdays | Works the Human-gated column: asks the open questions, validates answers, moves cards that are fully unblocked. |
| Plan Bootstrap | manual, once | Derives epics, a story dependency graph and a sprint status file from your product spec. |

## How it keeps itself honest

- **CI is the gate, not the agent's word.** Implementation creates a gate for the
  PR's CI run; nothing moves until GitHub reports it. A missing run is never green.
- **One piece of work, one card.** Before creating or releasing, the Planner
  searches every column, open PRs and branches.
- **Blocked means the pipeline is stuck.** Retryable and work-gated cards stay in
  Blocked and come back on their own; decisions, credentials and access go to
  Human-gated and wait for a person; hand-run operations wait in After Deploy
  with a runbook.
- **Edge cases are part of the contract.** Any stage adds them to the card
  (`## Edge cases`), and Verify walks every one.
- **The spec is read before anyone is asked.** Spec file → tracker ticket →
  repository docs → `decision` comments on the card. Only Scope Change edits the
  spec, and only through a pull request.
- **Agents never write to the integration branch directly, to the production
  branch, or to the tracker.**

## Install settings

| Setting | Default | Used for |
|---|---|---|
| Integration branch | `develop` | branch cards branch from, PR target |
| Staging branch | `staging` | the branch whose push deploys staging |
| Staging base URL, health path | — / `/up` | deploy proof and the browser walk |
| CI workflow / check name, staging deploy workflow | `ci` / `deploy-staging` | the PR gate and the deploy to watch |
| Work-in-flight cap | 15 | Planner ceiling and per-cycle maximum |
| Who merges an accepted PR | `agent` | `human` keeps PRs open for review |
| Product owner, delivery lead | generic | who agents name for decisions and escalations |
| Product spec path, planning directory, design source | unset | optional: spec file, planning artefacts, UI contract |
| Escalation channel, intake channel | unset | optional: Slack or Teams channel ids |
| UAT logins | unset | the test users Verify signs in as |

Everything else is a config item you can change after installing. See
[SETUP.md](SETUP.md) for what to connect before the triggers go on.

## Assumptions

- The repository is on GitHub and the agents use `gh` (pull requests, checks,
  runs). CI is GitHub Actions.
- Staging is deployed by a workflow on push to the staging branch and is shared
  and long-lived: the tester creates only what a walk needs, namespaced to the
  card, and deletes nothing.
- The steps are written for Claude Code; other runtimes work but were not
  measured.
- The template bundles no stack-specific skills. Attach your own (Rails,
  Django, React, …) to Tech Design, Implementation and Code Review after
  installing.
