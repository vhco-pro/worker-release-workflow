# Maintenance for Worker security automation

## Definition

Outcome: Enable weekly dependency update checks and CodeQL for worker-release-workflow.
Preserved contract: Existing build, release and deployment workflow behavior and executable dependency pins.
Task base: 4976e2176f6760f9cab1db4ae920355aa83a1b71
Affected paths: .github/dependabot.yml,.github/workflows/codeql-actions.yml,.github/workflows/worker-release.yml
Verification inputs: The workflows, dependency updater configuration, shared CodeQL v1.2.4 interface, and existing repository source and lockfiles.
Proposed change: Reuse the org Dependabot baseline and pinned CodeQL workflow.
Required checks: actionlint, YAML policy checks, and existing runtime checks where available. Confirm hosted scans and Dependabot runs after merge.

## Evidence

No Dependabot version config or CodeQL results were present at the base. Local actionlint and updater policy checks passed. Existing delivery workflow YAML parses to the same values as the base.

## Reviews

### R-1

Target: 2474284083116ac72fc518b253375c6ee05af8e2
Model: unknown
Isolation: procedural

```text
NO FINDINGS
```

## State

Local checks and independent review passed. Hosted verification is pending.
