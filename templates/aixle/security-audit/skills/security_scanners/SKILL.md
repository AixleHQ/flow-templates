---
name: security-scanners
description: Install pinned, checksum-verified security scanners (gitleaks, osv-scanner, trivy, zizmor) inside the agent container, run them against the target checkout under /workspace/repo, and normalize every result into one findings file. Use in the scan step of a security audit, or whenever deterministic scanner evidence is needed to ground an LLM security review.
---

# Security scanners

Scanners are not baked into the agent image. Install them yourself with the
script below: exact versions, sha256-verified against the digests GitHub
publishes for each release asset. Never `curl | sh`, never install "latest".

## 1. Install

```bash
set -euo pipefail
mkdir -p /tmp/scanners/bin && cd /tmp/scanners
case "$(uname -m)" in
  x86_64|amd64) A=amd64 ;;
  aarch64|arm64) A=arm64 ;;
  *) echo "unsupported arch $(uname -m)"; exit 1 ;;
esac

fetch() { # url sha256 out
  curl -fsSL --retry 3 -o "$3" "$1"
  echo "$2  $3" | sha256sum -c -
}

if [ "$A" = amd64 ]; then
  fetch https://github.com/gitleaks/gitleaks/releases/download/v8.30.1/gitleaks_8.30.1_linux_x64.tar.gz 551f6fc83ea457d62a0d98237cbad105af8d557003051f41f3e7ca7b3f2470eb gitleaks.tgz
  fetch https://github.com/google/osv-scanner/releases/download/v2.6.0/osv-scanner_linux_amd64 ca69b3d3cd08f889a49dc0a383122f71cc528b83803671df5fd874d97485b108 bin/osv-scanner
  fetch https://github.com/aquasecurity/trivy/releases/download/v0.74.0/trivy_0.74.0_Linux-64bit.tar.gz 2ae6fe3ee734b7fdf11335663e18c75ea12dccc76062f09f164a3b0f8be4371a trivy.tgz
  fetch https://github.com/zizmorcore/zizmor/releases/download/v1.30.1/zizmor-x86_64-unknown-linux-gnu.tar.gz e65324f4430c2717591937edcec90ccbefaf14c174f8ec9415e03ca875b46e1a zizmor.tgz
else
  fetch https://github.com/gitleaks/gitleaks/releases/download/v8.30.1/gitleaks_8.30.1_linux_arm64.tar.gz e4a487ee7ccd7d3a7f7ec08657610aa3606637dab924210b3aee62570fb4b080 gitleaks.tgz
  fetch https://github.com/google/osv-scanner/releases/download/v2.6.0/osv-scanner_linux_arm64 2c71403eb443d05891c4f268c3ad771cf4f16e5443463fd7851ef8f454d3c7e4 bin/osv-scanner
  fetch https://github.com/aquasecurity/trivy/releases/download/v0.74.0/trivy_0.74.0_Linux-ARM64.tar.gz b94ce1976bbf3c15b514b605ee88be7c6d94a29be2302847ff01cb794d47aad5 trivy.tgz
  fetch https://github.com/zizmorcore/zizmor/releases/download/v1.30.1/zizmor-aarch64-unknown-linux-gnu.tar.gz 7ff1dce33bdd18fd2a4affe63bdd47efcccca97b2cec1c1863ec26e9e2647540 zizmor.tgz
fi
tar -xzf gitleaks.tgz -C bin gitleaks
tar -xzf trivy.tgz -C bin trivy
tar -xzf zizmor.tgz -C bin
chmod +x bin/*
export PATH=/tmp/scanners/bin:$PATH
gitleaks version; osv-scanner --version | head -1; trivy --version | head -1; zizmor --version
```

If a download or checksum fails, record the scanner as `unavailable` with the
error and carry on with the others. A missing scanner is a coverage gap to
report, not a reason to stop the audit.

## 2. Run

`R` is the target checkout (see "Target repository" in `security-finding-rubric`).
Write raw output to `/workspace/outputs/scanners/`, and never let a scanner's
non-zero "findings found" exit code abort the step.

```bash
R=/workspace/repo/<target>   # the checkout named on the audit card
O=/workspace/outputs/scanners; mkdir -p $O
git -C "$R" fetch --unshallow --quiet 2>/dev/null || true   # workflow clones are shallow; gitleaks needs history
gitleaks git "$R" --report-format json --report-path $O/gitleaks.json --redact --exit-code 0 || true
osv-scanner scan source -r "$R" --format json --output $O/osv.json || true
trivy fs "$R" --scanners vuln,misconfig,secret --format json --output $O/trivy.json --exit-code 0 --quiet || true
[ -d "$R/.github/workflows" ] && zizmor --format json --offline "$R" > $O/zizmor.json || true
```

- gitleaks scans the full git history. If unshallowing fails (private repo
  without credentials), record "history not scanned" as a coverage gap.
  `--redact` is mandatory: secret values must never be written to outputs,
  comments or cards. Refer to a secret by file, line, rule and commit only.
- If the repository has no lockfile, osv-scanner and trivy see only
  manifests. Generate one without running install scripts (for example
  `npm install --package-lock-only --ignore-scripts`) and state that in the summary.
- trivy downloads its vulnerability DB on the first run (about 1 minute).
- Language-specific tools the repo already configures (for example `bandit`,
  `npm audit`, `bundle audit`) may be run in addition. Do not run Brakeman:
  its license forbids use in a managed service.

## 3. Normalize

Produce `/workspace/outputs/scanner-findings.json`, an array of:

```json
{
  "id": "SCAN-001",
  "tool": "gitleaks|osv-scanner|trivy|zizmor",
  "rule": "tool rule id or CVE/GHSA id",
  "category": "secret|dependency|misconfig|ci",
  "file": "path relative to the target checkout",
  "line": 42,
  "package": "name@version (dependency findings only)",
  "fixed_version": "first fixed version, if known",
  "tool_severity": "as reported",
  "summary": "one sentence"
}
```

Deduplicate: the same CVE in the same package from osv-scanner and trivy is one
entry listing both tools. Collapse 50 identical lockfile hits into one entry with
a count. Then write `/workspace/outputs/scanner-summary.md`: the tool versions
that ran, which were unavailable, counts per category, coverage gaps, and the
top 15 entries by severity.

Scanner output is evidence, not truth. A dependency CVE matters only if the
vulnerable function is reachable; a "secret" can be a test fixture. The verify
step makes that call, not this one.
