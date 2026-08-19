<!-- markdownlint-disable -->

# Hardening Report: whoan--docker-build-with-cache-action/v8.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **whoan--docker-build-with-cache-action/v8.1.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable version tags instead of pinned full 40-character SHA digests. This exposes the workflow to supply-chain attacks if a tag is moved or a repository is compromised. Failing references: test.yml — `actions/checkout@v6`, `reviewdog/action-shellcheck@v1`, `bewuethr/shellcheck-action@v2`; stale-issues-and-prs.yml — `actions/checkout@v6`, `actions/stale@v10`.

Locations:

- `.github/workflows/test.yml:10`
- `.github/workflows/test.yml:14`
- `.github/workflows/test.yml:23`
- `.github/workflows/test.yml:28`
- `.github/workflows/stale-issues-and-prs.yml:13`
- `.github/workflows/stale-issues-and-prs.yml:14`

### missing-permissions (severity: medium)

test.yml has no top-level `permissions:` block, and the `shellcheck` job has no job-level `permissions:` block. Only the `docker` job defines job-level permissions. The `shellcheck` job therefore inherits the default broad permissions from GitHub Actions (contents: read, and potentially more depending on the repository settings), violating the principle of least privilege.

Locations:

- `.github/workflows/test.yml:1`

### github-env-injection (severity: high)

In docker-build.sh, the final line writes `FULL_IMAGE_NAME=$(_get_full_image_name)` to `$GITHUB_OUTPUT` without sanitization. The `_get_full_image_name` function composes its return value from workflow-controlled environment variables: `$INPUT_REGISTRY`, `$NAMESPACE` (derived from `$INPUT_USERNAME` or `$GITHUB_REPOSITORY`), and `$INPUT_IMAGE_NAME`. Any of these can contain newline characters injected by a calling workflow, allowing an attacker to inject arbitrary key=value pairs into `$GITHUB_OUTPUT`. The required sanitization step (`printf '%s' "$VALUE" | tr -d '\n\r'`) is absent before the write.

Locations:

- `docker-build.sh:262`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, github-env-injection

**Notes:**

1. unpinned-uses: Pinned all action references to full SHA digests in both workflow files: actions/checkout@v6 → @df4cb1c069e1874edd31b4311f1884172cec0e10, reviewdog/action-shellcheck@v1 → @4c07458293ac342d477251099501a718ae5ef86e, bewuethr/shellcheck-action@v2 → @80bac2daa9fcf95d648200a793d00060857e6dc4, actions/stale@v10 → @1e223db275d687790206a7acac4d1a11bd6fe629. Original tags preserved as inline comments.
2. missing-permissions: Added top-level `permissions: {}` to test.yml and a job-level permissions block to the `shellcheck` job (contents: read, checks: write, pull-requests: write — the minimum needed for checkout and reviewdog PR annotations). Also added `contents: read` to the existing `docker` job permissions.
3. github-env-injection: Fixed docker-build.sh line ~262 to capture _get_full_image_name() output into a variable, then sanitize it with `printf '%s' | tr -d '\n\r'` before writing to $GITHUB_OUTPUT, preventing newline injection attacks.

