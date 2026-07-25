<!-- markdownlint-disable -->

# Hardening Report: dawidd6--action-homebrew-bump-formula/v8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **dawidd6--action-homebrew-bump-formula/v8** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The `Delete` step in close-pr.yml directly interpolates the attacker-controlled expression `${{github.event.pull_request.head.ref}}` into a `run:` shell command: `git push origin --delete ${{github.event.pull_request.head.ref}}`. A pull request author can craft a branch name containing shell metacharacters (e.g. `;`, `&&`, `|`) to execute arbitrary commands on the runner. The value should be passed via an `env:` variable and then double-quoted in the shell command.

Locations:

- `.github/workflows/close-pr.yml:22`

### unpinned-uses (severity: high)

Multiple workflow files reference `actions/checkout@v4` using a mutable tag instead of a pinned 40-character commit SHA. If the tag is moved (e.g. by a supply-chain compromise), the action will silently execute different code. Affected references: `actions/checkout@v4` in close-pr.yml (1 occurrence) and `actions/checkout@v4` in test.yml (3 occurrences). Each should be pinned to a full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/close-pr.yml:18`
- `.github/workflows/test.yml:21`
- `.github/workflows/test.yml:38`
- `.github/workflows/test.yml:55`

### missing-permissions (severity: medium)

Neither `.github/workflows/close-pr.yml` nor `.github/workflows/test.yml` declares a top-level `permissions:` block, and no individual job within either file declares its own `permissions:` block. Without explicit permissions, workflows inherit the default repository token permissions (which may be `write-all` depending on repository settings), violating the principle of least privilege. Each workflow should declare the minimal permissions required (e.g. `permissions: contents: write` for the push/delete operation in close-pr.yml).

Locations:

- `.github/workflows/close-pr.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across close-pr.yml and test.yml:
1. script-injection (close-pr.yml line 22): Moved `github.event.pull_request.head.ref` into an `env:` block as `HEAD_REF` and used `"$HEAD_REF"` (double-quoted) in the shell command to prevent shell metacharacter injection.
2. unpinned-uses: Pinned all 4 occurrences of `actions/checkout@v4` to `actions/checkout@11d5960a326750d5838078e36cf38b85af677262 # v4` (1 in close-pr.yml, 3 in test.yml).
3. missing-permissions: Added `permissions: contents: write` to close-pr.yml (required for the git push --delete operation) and `permissions: contents: read` to test.yml (minimal read-only access for checkout and testing).

