<!-- markdownlint-disable -->

# Hardening Report: whoan--docker-build-with-cache-action/v7.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **whoan--docker-build-with-cache-action/v7.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In `docker-build.sh`, the final line writes `FULL_IMAGE_NAME=$(_get_full_image_name)` to `$GITHUB_OUTPUT` without sanitization. The `_get_full_image_name` function assembles its return value from inherited process env vars `$INPUT_REGISTRY`, `$NAMESPACE` (derived from `$INPUT_USERNAME`), and `$INPUT_IMAGE_NAME` — all of which are workflow-controlled inputs forwarded into the Docker container by the calling workflow. None of these values are passed through `printf '%s' ... | tr -d '\n\r'` before the write, so a newline embedded in any of those inputs could inject arbitrary key=value pairs into `$GITHUB_OUTPUT`. The fix is: `safe=$(printf '%s' "$(_get_full_image_name)" | tr -d '\n\r'); echo "FULL_IMAGE_NAME=$safe" >> "$GITHUB_OUTPUT"`.

Locations:

- `docker-build.sh:270`

### unpinned-uses (severity: high)

`.github/workflows/test.yml` references four GitHub Actions using mutable version tags instead of immutable 40-character commit SHAs, making the workflow vulnerable to supply-chain attacks if any of those tags are moved: `actions/checkout@v2` (used twice), `reviewdog/action-shellcheck@v1`, and `bewuethr/shellcheck-action@v2`. Each should be pinned to a full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v2`.

Locations:

- `.github/workflows/test.yml:11`
- `.github/workflows/test.yml:15`
- `.github/workflows/test.yml:26`
- `.github/workflows/test.yml:31`

### missing-permissions (severity: medium)

`.github/workflows/test.yml` has no top-level `permissions:` key and neither the `shellcheck` job nor the `docker` job defines its own `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions (which may be `write-all` for older repositories), granting the `GITHUB_TOKEN` broader access than needed. A minimal `permissions:` block (e.g. `contents: read`) should be added at the top level or per job.

Locations:

- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, unpinned-uses, missing-permissions

**Notes:**

1. docker-build.sh (line 270): Fixed github-env-injection by replacing `echo "FULL_IMAGE_NAME=$(_get_full_image_name)" >> "$GITHUB_OUTPUT"` with a two-line sanitized version: `safe=$(printf '%s' "$(_get_full_image_name)" | tr -d '\n\r')` then `echo "FULL_IMAGE_NAME=$safe" >> "$GITHUB_OUTPUT"`. 2. .github/workflows/test.yml: Pinned all four unpinned action references to full commit SHAs — actions/checkout@v2 → 0717577d45739eb3c851188b29f50ed6c0b2194e, reviewdog/action-shellcheck@v1 → 4c07458293ac342d477251099501a718ae5ef86e, bewuethr/shellcheck-action@v2 → 80bac2daa9fcf95d648200a793d00060857e6dc4 (actions/checkout used twice). 3. .github/workflows/test.yml: Added top-level `permissions: contents: read` block to restrict GITHUB_TOKEN to minimum needed access.

