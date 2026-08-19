<!-- markdownlint-disable -->

# Hardening Report: oss-review-toolkit--ort-ci-github-action/v1.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **oss-review-toolkit--ort-ci-github-action/v1.2.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Four `run:` steps call `printenv >> "$GITHUB_ENV"` which dumps the entire process environment — including all inputs-derived env vars (SW_NAME, SW_VERSION, ORT_CLI_ARGS, POSTGRES_PASSWORD, HTTP_FILE_SERVER_TOKEN, ORT_DOCKER_IMAGE, ORT_DOCKER_CLI_ARGS, etc.) — into $GITHUB_ENV without any sanitization (no `printf '%s' | tr -d '\n\r'`). An attacker-controlled input containing newlines can inject arbitrary environment variable definitions into GITHUB_ENV, enabling environment variable hijacking for subsequent steps. This affects: (1) 'Init Workspace' step — exports all inputs-derived vars then calls `printenv >> "$GITHUB_ENV"`; (2) 'Capture ORT config URL and revision' step — calls `printenv >> "$GITHUB_ENV"` after setting ORT_CONFIG_VCS_URL/REVISION from git; (3) 'Compute ORT labels' step — calls `printenv >> "$GITHUB_ENV"` after building ORT_CLI_ANALYZE_ARGS from inputs; (4) 'Run ORT Analyzer' step — calls `printenv >> "$GITHUB_ENV"` inline in the docker run chain.

Locations:

- `action.yml:360`
- `action.yml:390`
- `action.yml:440`
- `action.yml:540`

### script-injection (severity: high)

Multiple `run:` blocks expand env vars that hold untrusted `inputs.*` values without double-quoting, violating sub-rule (b). In the 'Download project sources' step, `$ORT_DOCKER_CLI_ARGS`, `$ORT_DOCKER_IMAGE`, `--$ORT_LOG_LEVEL`, `$ORT_CLI_ARGS`, and `${ORT_CLI_DOWNLOAD_ARGS}` are all unquoted in the `docker run` command — these are set from `inputs.docker-cli-args`, `inputs.image`, `inputs.log-level`, `inputs.ort-cli-args`, and `inputs.vcs-*` respectively. The same pattern repeats in 'Run ORT Analyzer' (`$ORT_DOCKER_CLI_ARGS`, `$ORT_DOCKER_IMAGE`, `--$ORT_LOG_LEVEL`, `$ORT_CLI_ARGS`, `${ORT_CLI_ANALYZE_ARGS}`), 'Run ORT Scanner' (same plus `${ORT_CLI_SCAN_ARGS}`), 'Run ORT Advisor' (`$ORT_ADVISORS`, `${ORT_CLI_ADVISE_ARGS}`), 'Run ORT Evaluator' (`${ORT_CLI_EVALUATE_ARGS}`), and 'Run ORT Reporter' (`$ORT_REPORT_FORMATS`, `$SW_NAME`, `${ORT_CLI_REPORT_ARGS}`). Unquoted expansion allows shell metacharacters (`;`, `|`, `&`, `$(...)`, whitespace, globs) in any of these inputs to break out of the intended command and execute arbitrary shell commands.

Locations:

- `action.yml:475`
- `action.yml:515`
- `action.yml:548`
- `action.yml:585`
- `action.yml:617`
- `action.yml:646`

### missing-permissions (severity: medium)

The workflow file 'renovate-validation.yml' has no top-level `permissions:` key and the single job `renovate-validation` also has no job-level `permissions:` key. Without explicit permissions, the workflow inherits the repository's default token permissions (which may be `write-all` for some repositories), granting the GITHUB_TOKEN broader access than necessary. A minimal permissions block (e.g., `contents: read`) should be added.

Locations:

- `.github/workflows/renovate-validation.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, script-injection, missing-permissions

**Notes:**

Fixed all three findings in action.yml and .github/workflows/renovate-validation.yml:

1. github-env-injection: Replaced all four `printenv >> "$GITHUB_ENV"` calls with explicit, sanitized writes. Each variable is now written individually using `printf '%s' "$VAR" | tr -d '\n\r'` to strip newlines before writing to GITHUB_ENV, preventing environment variable injection attacks.

2. script-injection: Fixed all six docker run command blocks (Download, Analyzer, Scanner, Advisor, Evaluator, Reporter) by: (a) using `read -ra _array <<< "$VAR"` to split multi-word args into bash arrays and expanding them as `"${_array[@]}"`, (b) quoting single-value variables like `"$ORT_DOCKER_IMAGE"`, `"$ORT_ADVISORS"`, `"$ORT_REPORT_FORMATS"`, and (c) changing `--$ORT_LOG_LEVEL` to `"--${ORT_LOG_LEVEL}"` to prevent shell metacharacter injection.

3. missing-permissions: Added `permissions: contents: read` top-level block to renovate-validation.yml to restrict the GITHUB_TOKEN to the minimum required permissions.

### Iteration 2

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed three security findings in hardened/action/action.yml:

1. github-env-injection: Replaced all 11 bare `echo "key=${VAR}" >> "$GITHUB_OUTPUT"` writes in the Init Workspace step with `printf 'key=%s\n' "$(printf '%s' "$VAR" | tr -d '\n\r')" >> "$GITHUB_OUTPUT"` to strip newlines and prevent injection of arbitrary output variables via a crafted inputs.ort-home-path value.

2. script-injection (ort-config step, line ~345): Added double-quotes around `$ORT_CONFIG_VCS_URL` and `$ORT_CONFIG_VCS_REVISION` in the `git remote add origin` and `git fetch` commands to prevent shell metacharacter injection.

3. script-injection (download step, line ~435): Replaced string-concatenation-based ORT_CLI_DOWNLOAD_ARGS building (which left values unquoted) with a bash array `_ort_cli_download_args` where each value is properly double-quoted (e.g., `_ort_cli_download_args+=(--vcs-type "$PROJECT_VCS_TYPE")`), preventing command injection via attacker-controlled vcs-type, vcs-url, vcs-revision, and vcs-path inputs.

