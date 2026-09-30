---
name: security-finding-rubric
description: The shared contract for security audit findings — engagement context, target repository resolution, the candidate/finding JSON schema, how scanner evidence joins a finding, severity and confidence rubric, hard exclusions, and the verification standard a finding must meet before it is reported. Use in every step of a security audit so all agents speak the same format and bar.
---

# Security finding rubric

Adapted from Anthropic's claude-code-security-review (MIT) and Trail of Bits'
fp-check method. Every step of the audit uses this contract.

## Engagement context

This is an **authorized, defensive white-box review**. The project owner
attached the repository to this project and asked for the audit, so that the
issues can be fixed. The work happens on source code only:

- Read code and run local static tools. Never send traffic to a deployed
  system, a third party, or any host other than package and advisory mirrors.
- Describe weaknesses the way a code-review report does: what is wrong, where,
  why it is reachable, what an attacker would observe, and how to fix it.
- An exploit sketch is one line naming the input and the observable result,
  for example "GET /api/orders/42 as user B returns user A's order". Never
  write working payload chains, shellcode, credential-stuffing or evasion
  steps, or anything usable against systems other than this repository.
- Write long documents **section by section**: write the first section, then
  append each next section with a separate edit, so that no single write holds
  the whole document. If a write is interrupted or refused, do not abandon the
  step. Rewrite that section more concisely, in remediation-oriented language,
  and continue. Report a section as missing only after two attempts.

## Target repository

Repositories are cloned into `/workspace/repo/<name>/`. The audit target is the
checkout named on the audit card's `Repository:` line. If only one checkout
exists, that checkout is the target. Wherever step instructions say
`/workspace/repo`, read it as the target checkout directory. Paths in every
finding are relative to that directory.

## Candidate schema (hunt output)

```json
{
  "id": "H-ACCESS-003",
  "title": "Order lookup returns any user's order (IDOR)",
  "class": "authz|authn|injection|ssrf|xss|deserialization|path-traversal|crypto|secrets|config|business-logic|ci-supply-chain|dependency|other",
  "cwe": "CWE-639",
  "owasp": "A01:2025",
  "location": [{"file": "routes/order.ts", "line": 57, "symbol": "getOrder"}],
  "source": "where attacker-controlled input enters (entry point from the threat model)",
  "sink": "where it does damage",
  "path": "source -> ... -> sink, one hop per line, file:line each",
  "precondition": "what the attacker needs: anonymous | any user | admin | local | network position",
  "impact": "what they get, concretely",
  "evidence": "the code that proves it, quoted briefly",
  "confidence": 0.0,
  "related_scan_ids": ["SCAN-012"]
}
```

## Scanner evidence joins a finding, it never replaces one

A scanner entry (`SCAN-*`) is evidence, not a report. Scanner entries never get
fix cards of their own, so a real issue that only a scanner saw would otherwise
go untracked.

- When a candidate describes the same issue as a scanner entry, it is **one
  finding**. Put the entry's id in `related_scan_ids`. Never exclude a
  candidate because a scanner saw the same thing.
- A scanner entry that is exploitable in this code becomes a candidate even if
  no hunter raised it. Examples: a live committed secret, a reachable vulnerable
  dependency, a CI workflow that runs untrusted input with secrets. Verify it
  like any other candidate.
- Several scanner hits with one root cause become one finding. For example,
  one leaked key found in both the tree and the history is a single finding.

## Hard exclusions — never report

- Denial of service, resource exhaustion, missing rate limiting, ReDoS without a proven catastrophic input.
- Secrets or credentials that are not committed to the repository or its history.
- Missing hardening with no concrete exploit ("could add CSP", "should use a WAF").
- Issues only in tests, fixtures, examples, docs or dev-only tooling, unless that code ships or runs in CI with secrets.
- Memory-safety issues in memory-safe languages.
- Log spoofing, missing audit logging, verbose error messages without secret leakage.
- Theoretical race conditions without a concrete interleaving that changes security state.
- Outdated dependencies with no known vulnerability, or a CVE whose vulnerable function is not reachable from this code.
- Client-side-only checks when the server enforces the same rule.
- Anything that requires the attacker to already hold the privilege being "escalated to".

## Confidence

- 0.9–1.0: complete source→sink path traced in code, no guard found, exploit sketch works on paper.
- 0.8–0.9: path traced, one assumption about runtime configuration stated explicitly.
- < 0.8: do not report. Hunt steps may emit candidates at ≥ 0.5; verify decides.

## Severity (after verification)

| Severity | Meaning |
|---|---|
| critical | Unauthenticated remote code execution, auth bypass to admin, forging any user's credentials, or mass data exfiltration |
| high | Authenticated user reaches other tenants' or users' data or actions; stored XSS on privileged pages; SSRF to internal network or cloud metadata; live secret in git history |
| medium | Needs unusual preconditions or user interaction; limited data exposure; reflected XSS; CSRF on a sensitive action |
| low | Defense-in-depth gaps that have a concrete but narrow exploit |

## Verification standard (verify step)

Work through each candidate independently, reading the code fresh:

1. Re-read every hop of `path` at the cited lines. A hop that does not exist makes the candidate FALSE.
2. Search for guards the hunter missed: middleware, before_action / decorators, ORM scoping, framework auto-escaping, schema validation, allowlists, feature flags, and config that disables the path in production.
3. Check reachability: is the entry point routed and exposed? Is the dependency function actually called?
4. Write the one-line exploit sketch (input and observable result). Nothing may be executed against real systems.
5. Look for variants: the same bug pattern elsewhere. Emit each as its own verified finding, citing the original.
6. Give a verdict:
   - `TRUE_POSITIVE`: confidence ≥ 0.8 and no hard exclusion applies.
   - `FALSE_POSITIVE` or `EXCLUDED`: a hard exclusion applies or the code does not prove the issue. Give a one-line reason.
   - `MERGED`: the candidate duplicates another candidate. Name the finding it merged into.
   Overlap with a scanner entry is never a reason to exclude (see above).

## Verified finding schema (verify output)

The candidate fields plus:

```json
{
  "finding_id": "SA-007",
  "verdict": "TRUE_POSITIVE",
  "severity": "high",
  "confidence": 0.9,
  "exploit_sketch": "GET /rest/order/123 as user B returns user A's order",
  "guards_checked": ["auth middleware on /rest/*: authenticates only, no ownership check"],
  "remediation": "Scope the query to the current user: Order.where(id:, user_id: current_user.id)",
  "regression_test": "Request user A's order as user B and expect 404",
  "variants_of": null
}
```

Secret values never appear in any field. Cite the file, line, commit and rule only.
