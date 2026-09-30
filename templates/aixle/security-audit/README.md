# Security Audit

A white-box security audit of a repository that ends in work, not a PDF. Every
finding that survives independent verification becomes a fix card on the board,
and moving a card to **Fixing** has an agent write the regression test and the fix.

## What you get

- `SECURITY-AUDIT.md` on the audit card: executive summary, findings table
  (severity, CWE, OWASP Top 10:2025), one section per finding, scope, method and
  coverage gaps.
- One **Fix Backlog** card per verified finding. Each card holds the location,
  the source → sink path, a one-line exploit sketch, a concrete fix, a regression
  test and an acceptance checklist.
- Intermediate evidence as run outputs: threat model, entry-point inventory,
  normalized scanner results, every rejected candidate with the reason it was
  rejected.

## How it works

```
Audit Queue → Auditing ──────────────────────────────────────────→ Audit Complete
               │ recon + threat model ─┐
               │ pinned scanners ──────┼→ hunt: access control ─┐
               │                       ├→ hunt: injection ──────┼→ skeptic verify → report
               │                       └→ hunt: secrets/config ─┘        │
                                                                         ▼
                                  Fix Backlog → Fixing → Fix In Review → Done / Won't Fix
```

1. **Recon** maps every entry point, trust boundary and security mechanism.
2. **Scanners** give the review deterministic evidence: gitleaks (full history),
   osv-scanner, trivy and zizmor. Each is installed at a pinned version and
   sha256-verified inside the agent container, so nothing needs to be baked into
   an image.
3. **Three hunts run in parallel**, each tracing data flow from the entry-point
   map rather than grepping for sinks.
4. **The Skeptic Verifier** re-derives every candidate from the code, looks for
   the guard the hunter missed, and applies a fixed rubric: confidence of at least
   0.8, plus hard exclusions such as DoS, theoretical races and missing hardening.
   It sweeps the critical and high scanner entries, so an exploitable issue that
   only a scanner saw still becomes a finding, and it searches for variants of
   each confirmed bug.
5. **The report step** writes the report and creates the fix cards. It skips a
   finding that already has a card, so re-audits do not create duplicates.

The pipeline follows the structure shared by Anthropic's
claude-code-security-review, OpenAI Aardvark and Trail of Bits' methodology:
context first, then candidates, then independent verification of each one.

## What it costs

Measured on two real runs:

| Target | Depth | Time | Model usage | Candidates → verified |
|---|---|---|---|---|
| Rails + React monolith, ~500 entry points | standard | 53 min | ~$53 | 23 → 15 |
| OWASP Juice Shop (Node/Express) | quick | 39 min | ~$18 | 56 → 41 |

On Juice Shop the audit found the benchmark's classic flaws from the code alone: SQL injection, admin self-registration, JWT forgery, code injection, XXE, zip slip, SSRF, NoSQL injection, forged coupons and a set of IDORs. It did not read the challenge list. Fix runs are priced per card.

## Starting an audit

Create a card in **Audit Queue**. The column's purpose shows the description
format:

```markdown
## Scope
- Repository: <repository name as attached to the project>
- Include: <paths, or "everything">
- Exclude: test/, docs/, vendor/
## Depth
standard
## Context
What the system does, who its users and tenants are, how it is deployed,
and what matters most to protect.
```

Move the card to **Auditing** to start. The context section matters: the
hunters use it to decide what "cross-tenant" or "privileged" means for your
system.

## Safety and scope

- The audit only reads code and runs local static tools. Nothing is sent to
  deployed systems.
- Exploit sketches are one line each (the input and the observable result),
  never working payloads.
- Secret values are never written anywhere. gitleaks runs with `--redact`, and
  findings cite file, line, commit and rule only.
- On a public repository the fix workflow does not push. It attaches a
  `.patch` to the card, because a public PR would disclose the vulnerability.

## Third-party content

This template bundles snapshots of these skills, unmodified:

- **Trail of Bits** (https://github.com/trailofbits/skills), licensed
  CC BY-SA 4.0 (https://creativecommons.org/licenses/by-sa/4.0/):
  `audit-context-building`, `insecure-defaults`, `fp-check`,
  `variant-analysis`.
- **Sentry** (https://github.com/getsentry/skills), licensed Apache-2.0:
  `security-review`, `gha-security-review`. Their LICENSE file is included.
- **OpenAI** (https://github.com/openai/skills): `security-threat-model`. Its
  LICENSE.txt is included.

The rubric is adapted from Anthropic's claude-code-security-review (MIT). The
scanner binaries are downloaded from their upstream releases at run time and
are not redistributed. Brakeman and Semgrep/Opengrep rule packs are left out on
purpose, because their licenses restrict use in managed services.
