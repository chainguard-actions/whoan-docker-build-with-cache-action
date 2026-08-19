<!-- markdownlint-disable -->

# Hardening Report: whoan--docker-build-with-cache-action/v8.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **whoan--docker-build-with-cache-action/v8.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In `docker-build.sh`, the final line writes `FULL_IMAGE_NAME=$(_get_full_image_name)` to `$GITHUB_OUTPUT` without sanitization. `_get_full_image_name()` returns a value composed of `INPUT_REGISTRY`, `NAMESPACE` (derived from `INPUT_USERNAME`), and `INPUT_IMAGE_NAME` — all user-controlled action inputs that can contain newlines. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the write, allowing an attacker to inject arbitrary key=value pairs into the GitHub output environment.

Locations:

- `docker-build.sh:313`

### missing-permissions (severity: medium)

The workflow file `.github/workflows/test.yml` has no top-level `permissions:` key, and neither the `shellcheck` job nor the `docker` job defines its own `permissions:` block. Without explicit permissions, the workflow runs with the default (potentially broad) token permissions.

Locations:

- `.github/workflows/test.yml:1`

### unpinned-uses (severity: high)

The workflow file `.github/workflows/test.yml` references four actions using mutable version tags instead of immutable 40-character commit SHAs, making the workflow vulnerable to supply-chain attacks if those tags are moved:
- `actions/checkout@v2` (lines 10 and 27)
- `reviewdog/action-shellcheck@v1` (line 14)
- `bewuethr/shellcheck-action@v2` (line 23)

Locations:

- `.github/workflows/test.yml:10`
- `.github/workflows/test.yml:14`
- `.github/workflows/test.yml:23`
- `.github/workflows/test.yml:27`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, missing-permissions, unpinned-uses

**Notes:**

1. docker-build.sh (line 313): Fixed github-env-injection by capturing _get_full_image_name() output into a variable and sanitizing it with `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_OUTPUT. 2. .github/workflows/test.yml: Added top-level `permissions: contents: read` block plus per-job permissions (shellcheck: contents:read; docker: contents:read + packages:write for ghcr.io push). 3. .github/workflows/test.yml: Pinned all four action references to full commit SHAs — actions/checkout@v2 → 0717577d45739eb3c851188b29f50ed6c0b2194e, reviewdog/action-shellcheck@v1 → 4c07458293ac342d477251099501a718ae5ef86e, bewuethr/shellcheck-action@v2 → 80bac2daa9fcf95d648200a793d00060857e6dc4 — with original tags preserved in comments.

