<!-- markdownlint-disable -->

# Hardening Report: dawidd6--action-homebrew-bump-formula/v4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **dawidd6--action-homebrew-bump-formula/v4** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

In .github/workflows/close-pr.yml, the `run:` block directly interpolates the attacker-controlled expression `${{github.event.pull_request.head.ref}}` into a shell command: `git push origin --delete ${{github.event.pull_request.head.ref}}`. This is triggered on `pull_request` events, meaning any contributor opening a PR against a Formula file can supply a crafted branch name containing shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) that will be executed on the runner. This is a rule-(a) script injection violation. Fix: move the value into an `env:` variable and double-quote the expansion, e.g. `env: { HEAD_REF: "${{github.event.pull_request.head.ref}}" }` then `run: git push origin --delete "$HEAD_REF"`

Locations:

- `.github/workflows/close-pr.yml:22`

### missing-permissions (severity: medium)

Neither .github/workflows/test.yml nor .github/workflows/close-pr.yml declares a top-level `permissions:` key, and no individual job within either file declares its own `permissions:` block. Without explicit permissions, workflows inherit the repository's default token permissions (often `write-all`), granting unnecessarily broad access. Each workflow should declare the minimal required permissions (e.g. `permissions: contents: read` or `pull-requests: write` as needed).

Locations:

- `.github/workflows/test.yml:1`
- `.github/workflows/close-pr.yml:1`

### unpinned-uses (severity: high)

All `uses:` references in the workflow files pin to a mutable version tag (`@v4`) rather than an immutable 40-character commit SHA. If the `actions/checkout` repository is compromised or the tag is moved, the action will silently execute attacker-controlled code. All four occurrences should be pinned to a full SHA, e.g. `uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Failing references:
- `.github/workflows/close-pr.yml`: `uses: actions/checkout@v4`
- `.github/workflows/test.yml`: `uses: actions/checkout@v4` (appears 3 times)

Locations:

- `.github/workflows/close-pr.yml:18`
- `.github/workflows/test.yml:24`
- `.github/workflows/test.yml:43`
- `.github/workflows/test.yml:60`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, missing-permissions, unpinned-uses

**Notes:**

Fixed all three findings across .github/workflows/close-pr.yml and .github/workflows/test.yml:
1. script-injection (close-pr.yml line 22): Moved `${{github.event.pull_request.head.ref}}` into an `env:` block as `HEAD_REF` and changed the run command to `git push origin --delete "$HEAD_REF"`.
2. missing-permissions: Added `permissions: contents: write` to close-pr.yml (requires write to delete remote branches) and `permissions: contents: read` to test.yml (only needs repo read access).
3. unpinned-uses: Pinned all four `actions/checkout@v4` references to the immutable SHA `11d5960a326750d5838078e36cf38b85af677262 # v4` — one in close-pr.yml and three in test.yml.

