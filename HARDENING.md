<!-- markdownlint-disable -->

# Hardening Report: whoan--docker-build-with-cache-action/v8.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **whoan--docker-build-with-cache-action/v8.0.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In docker-build.sh, the final line writes `FULL_IMAGE_NAME=$(_get_full_image_name)` to `$GITHUB_OUTPUT` without sanitization. `_get_full_image_name()` returns a value composed of `INPUT_IMAGE_NAME` and `INPUT_REGISTRY`, which are environment variables set by the calling workflow from action inputs — i.e., caller-controlled (untrusted) data. An attacker could inject newlines into these values to poison subsequent `$GITHUB_OUTPUT` entries. The required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`) is absent before the write.

Locations:

- `docker-build.sh:295`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in docker-build.sh at line 295. The original code wrote `FULL_IMAGE_NAME=$(_get_full_image_name)` directly to $GITHUB_OUTPUT without sanitization. The fix captures the value into a local variable, strips newlines and carriage returns using `printf '%s' "$_full_image_name" | tr -d '\n\r'`, and then writes the sanitized value to $GITHUB_OUTPUT. This prevents an attacker from injecting newlines via INPUT_IMAGE_NAME or INPUT_REGISTRY to poison subsequent $GITHUB_OUTPUT entries.

