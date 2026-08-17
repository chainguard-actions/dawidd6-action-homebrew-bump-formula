<!-- markdownlint-disable -->

# Hardening Report: dawidd6--action-homebrew-bump-formula/v7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **dawidd6--action-homebrew-bump-formula/v7** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The `run:` block in close-pr.yml directly interpolates `${{github.event.pull_request.head.ref}}` — an attacker-controlled value from a pull request — into a shell command string. An attacker can craft a branch name containing shell metacharacters (e.g. `;`, `$(...)`) to execute arbitrary commands on the runner. Offending line: `run: git push origin --delete ${{github.event.pull_request.head.ref}}`

Locations:

- `.github/workflows/close-pr.yml:22`

### unpinned-uses (severity: high)

All `uses:` references in the workflow files use a mutable tag (`@v4`) instead of a pinned 40-character commit SHA. This exposes the action to supply-chain attacks if the referenced tag is moved or overwritten. Failing references: `actions/checkout@v4` in close-pr.yml (line 18) and `actions/checkout@v4` in test.yml (lines 30, 57, 79).

Locations:

- `.github/workflows/close-pr.yml:18`
- `.github/workflows/test.yml:30`
- `.github/workflows/test.yml:57`
- `.github/workflows/test.yml:79`

### missing-permissions (severity: medium)

Neither `.github/workflows/close-pr.yml` nor `.github/workflows/test.yml` declares a `permissions:` key at the top level or at the job level. Without explicit permissions, workflows run with the default (often broad) token permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/close-pr.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across close-pr.yml and test.yml:
1. script-injection (close-pr.yml line 22): Moved `${{github.event.pull_request.head.ref}}` into an `env:` block as `HEAD_REF` and referenced it as `"$HEAD_REF"` in the shell command to prevent shell metacharacter injection.
2. unpinned-uses: Pinned all four `actions/checkout@v4` references to the full SHA `34e114876b0b11c390a56381ad16ebd13914f8d5` with a `# v4` comment for readability.
3. missing-permissions: Added `permissions: contents: write` to close-pr.yml (required for `git push --delete`) and `permissions: contents: read` to test.yml (minimum needed for checkout).

