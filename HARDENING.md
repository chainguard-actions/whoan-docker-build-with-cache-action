<!-- markdownlint-disable -->

# Hardening Report: whoan--docker-build-with-cache-action/v7.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **whoan--docker-build-with-cache-action/v7.0.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In docker-build.sh, the final line writes `FULL_IMAGE_NAME=$(_get_full_image_name)` to `$GITHUB_OUTPUT` without sanitization. The `_get_full_image_name` function returns a value composed of `INPUT_REGISTRY`, `NAMESPACE`, and `INPUT_IMAGE_NAME` — all inherited environment variables from the calling workflow (user-controlled inputs). An attacker could inject newlines into these values to write arbitrary entries to `$GITHUB_OUTPUT`. The required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`) is not applied before the write.

Locations:

- `docker-build.sh:291`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection vulnerability in docker-build.sh at line 291. The original code wrote `FULL_IMAGE_NAME=$(_get_full_image_name)` directly to $GITHUB_OUTPUT without sanitization. The fix captures the full image name into a variable, sanitizes it with `printf '%s' "$_full_image_name" | tr -d '\n\r'` to strip newlines and carriage returns, then writes the sanitized value to $GITHUB_OUTPUT. This prevents attackers from injecting arbitrary entries into $GITHUB_OUTPUT via newlines in user-controlled inputs (INPUT_REGISTRY, NAMESPACE, INPUT_IMAGE_NAME).

