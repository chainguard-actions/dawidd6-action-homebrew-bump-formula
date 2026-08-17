<!-- markdownlint-disable -->

# Hardening Report: dawidd6--action-homebrew-bump-formula/v5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **dawidd6--action-homebrew-bump-formula/v5** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The `run:` block in the 'Delete' step directly interpolates the attacker-controlled GitHub Actions expression `${{github.event.pull_request.head.ref}}` into a shell command: `git push origin --delete ${{github.event.pull_request.head.ref}}`. Since this workflow triggers on `pull_request` events, a malicious actor can craft a branch name containing shell metacharacters (e.g. `;`, `&&`, `|`) to achieve arbitrary command execution on the runner.

Locations:

- `.github/workflows/close-pr.yml:22`

### permissions (severity: medium)

missing-permissions: Neither workflow file has a top-level `permissions:` key, and no job within either file defines its own `permissions:` block. This means the workflows run with the default (potentially broad) token permissions. Explicit minimal permissions should be declared.

Locations:

- `.github/workflows/close-pr.yml:1`
- `.github/workflows/test.yml:1`

### unpinned-uses (severity: high)

All `uses:` references in both workflow files pin to the mutable tag `@v4` rather than an immutable 40-character commit SHA. If the `actions/checkout` repository is compromised or the tag is moved, the action could execute arbitrary malicious code. Affected references: `actions/checkout@v4` (appears in close-pr.yml and three times in test.yml).

Locations:

- `.github/workflows/close-pr.yml:20`
- `.github/workflows/test.yml:28`
- `.github/workflows/test.yml:44`
- `.github/workflows/test.yml:60`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, permissions, unpinned-uses

**Notes:**

Fixed all three findings across both workflow files: (1) script-injection in close-pr.yml: moved `github.event.pull_request.head.ref` into an env var `HEAD_REF` and referenced it as `"$HEAD_REF"` in the shell command; (2) permissions: added `permissions: contents: write` to close-pr.yml (required for git push --delete) and `permissions: contents: read` to test.yml; (3) unpinned-uses: pinned all four `actions/checkout@v4` references to the immutable SHA `34e114876b0b11c390a56381ad16ebd13914f8d5` with a `# v4` comment for readability.

