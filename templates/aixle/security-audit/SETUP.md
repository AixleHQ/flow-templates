# Setup

## 1. Attach the repository to audit

The template needs one repository, `target`. When you install, map it to a
repository already attached to the project, or attach one:

- **Audit only:** read-only access is enough, and a public repository can be
  attached by URL.
- **Audit and fix PRs:** the repository must be private and connected through
  the GitHub (or GitLab) integration with write access. Otherwise the fix
  workflow attaches a `.patch` to the card instead of opening a PR.

To audit several repositories from one project, attach them all and add them
to both workflows' repositories. Each audit card then names its target on the
`Repository:` line.

## 2. Turn on the triggers

Triggers install **inactive**. Enable both of them:

- **Audit card enters Auditing** starts *Repository security audit*.
- **Fix card enters Fixing** starts *Fix security finding*.

## 3. Check the runtime

- The steps are written for Claude Code. Other runtimes work, but they have not
  been measured.
- Agent containers need outbound HTTPS to `github.com` and
  `objects.githubusercontent.com` (scanner downloads), and to the trivy and OSV
  advisory databases.
- A run is billed to the card's assignee, so assign audit cards to the person
  whose agent credential should pay.

## 4. Run your first audit

Create a card in **Audit Queue** using the format in the column purpose, set
the depth to `quick`, and move it to **Auditing**.
