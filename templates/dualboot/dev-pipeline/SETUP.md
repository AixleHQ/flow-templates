# Setup

The install creates the board, the agents, the workflows and the config items,
and leaves every trigger **inactive**. Work through the checklist in this order.

## 1. Attach the repository

The template needs one repository, `app`: the application the pipeline builds.
Connect GitHub for the project (the agents use `gh` for pull requests, checks
and runs) and attach the repository with **write** access — Implementation
pushes branches and opens pull requests, Merge and deploy pushes to the staging
branch, and Verify merges accepted pull requests when the merge policy is
`agent`.

Then check the install settings against the repository:

- the integration branch and the staging branch exist and are protected the way
  you want (agents never push to the integration branch directly);
- `CI_CHECK_NAME` is the GitHub Actions workflow (or check) that runs on pull
  requests, and `STAGING_DEPLOY_WORKFLOW` is the one that deploys the staging
  branch;
- `STAGING_BASE_URL` + `HEALTH_PATH` answer 200.

## 2. Add the secrets

`UAT_PASSWORD` and `UAT_ADMIN_PASSWORD` are the passwords of the two test users
Verify signs in as (`UAT_LOGIN`, `UAT_ADMIN_LOGIN`). Both accounts must exist on
staging. A product with more roles gets more pairs, added as config items named
`UAT_<ROLE>_LOGIN` / `UAT_<ROLE>_PASSWORD` and attached to the Verify step.

## 3. Add the agent members

A card entering an automated column **without an assignee never starts a run**.
Add the agent users (the identities whose credentials pay for runs) as members
of the project; the Planner, Scope Change and Chat Intake assign cards to them
round-robin, read from the board.

## 4. Decide the optional config items

Each is `unset` until you change it; the workflows behave sensibly without them.

| Config item | Set it when |
|---|---|
| `SPEC_PATH` | you keep a product spec / PRD in the repository. Scope Change edits it through pull requests; Merge and deploy files `Spec update:` cards when staging drifts from it. |
| `PLANNING_DIR` | you want planning artefacts (`epics.md`, `story-dependency-graph.md`, `sprint-status.yaml`). Run **Plan Bootstrap** once to create them from the spec; the Planner then also releases from the graph. |
| `DESIGN_SOURCE` | you have a UI contract: a `DESIGN.md` path, a Figma file, or the URL of a deployed reference front. Design and Verify compare against it. |
| `ESCALATION_CHANNEL`, `INTAKE_CHANNEL` | you connect Slack or Teams. Agents escalate to the first; Chat Intake listens on the second for messages containing `triage`, and the progress report lands there. |
| `PR_MERGE_POLICY` | `human` if people, not Verify, merge accepted pull requests. With `PLANNING_DIR` set, whoever merges then also sets the story to `done` in `sprint-status.yaml`. |

The product owner and delivery lead names you entered at install are baked into
the texts; change them by editing the step instructions.

## 5. Connect chat (optional)

Connect Slack or Teams, invite the bot to the intake channel, put the channel
ids into `INTAKE_CHANNEL` and `ESCALATION_CHANNEL`, then open the Chat Intake
trigger and pick the connected channel. The trigger fires on messages that
contain the word `triage`; tell the team that answers to escalations reach a
card only when said with that word or when a person moves the card.

## 6. Turn on the triggers

Activate the eight column triggers first (Needs Planning, Tech Design, Design,
Implementation, CI check, Code Review, Merge and deploy to staging, Verify).
Then the Planner schedule. Progress Report, Unblock Digest and Chat Intake only
once the channels are set. Adjust the schedules' cron and time zone to your
working hours — they install in UTC.

## 7. First run

Create a card in **Backlog** with a title, a description that states what is
wanted and how you would know it is done, and, if you use a tracker, the ticket
key at the start of the title. The next Planner cycle releases it into Tech
Design. Or run **Task Q&A** on it first to see what the analyst makes of it.

## Runtime notes

- Agent containers need outbound access to GitHub, to `npm` (the Playwright and
  Context7 MCP servers start through `npx`), and to the staging host.
- A run is billed to the card's assignee.
- Attach stack-specific skills to Tech Design, Implementation and Code Review;
  the template ships none.
