<!-- markdownlint-disable -->

# Hardening Report: oss-review-toolkit--ort-ci-github-action/v1.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **oss-review-toolkit--ort-ci-github-action/v1.2.0** was hardened automatically. 2 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Four `run:` blocks call `printenv >> "$GITHUB_ENV"` which dumps the entire process environment — including all `inputs.*`-derived env vars (e.g. ORT_CLI_ARGS, ORT_DOCKER_IMAGE, SW_NAME, SW_VERSION, HTTP_FILE_SERVER_PASSWORD, POSTGRES_PASSWORD, etc.) — into $GITHUB_ENV without any newline sanitization (`tr -d '\n\r'`). A calling workflow can supply an input value containing embedded newlines to inject arbitrary key=value pairs into the runner environment for subsequent steps. This matches check pattern (e): inherited env vars forwarded to GITHUB_ENV without sanitization. Affected steps: 'Init Workspace' (ort-init), 'Capture ORT config URL and revision' (ort-config-url-and-revision), 'Compute ORT labels' (ort-labels), and 'Run ORT Analyzer' (ort-analyzer).

Locations:

- `action.yml:271`
- `action.yml:291`
- `action.yml:320`
- `action.yml:374`

### script-injection (severity: high)

Multiple `run:` blocks expand env vars that hold `inputs.*`-derived values without double-quoting, violating rule (b). Examples across steps: `$ORT_DOCKER_CLI_ARGS` (from inputs.docker-cli-args), `$ORT_DOCKER_IMAGE` (from inputs.image), `--$ORT_LOG_LEVEL` (from inputs.log-level), `$ORT_CLI_ARGS` (from inputs.ort-cli-args), `${ORT_CLI_DOWNLOAD_ARGS}` (built from inputs.vcs-*), `$ORT_ADVISORS` (from inputs.advisors), `$ORT_REPORT_FORMATS` (from inputs.report-formats) are all passed unquoted to `docker run` commands. An attacker-controlled input value containing shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) can break out of the intended argument and execute arbitrary commands. Affected steps: 'Download project sources' (project-sources), 'Run ORT Analyzer' (ort-analyzer), 'Run ORT Scanner' (ort-scanner), 'Run ORT Advisor' (ort-advisor), 'Run ORT Evaluator' (ort-evaluator), 'Run ORT Reporter' (ort-reporter).

Locations:

- `action.yml:349`
- `action.yml:365`
- `action.yml:385`
- `action.yml:406`
- `action.yml:427`
- `action.yml:449`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed all four `printenv >> "$GITHUB_ENV"` calls by replacing them with explicit per-variable writes using `printf '%s' "${!_var}" | tr -d '\n\r'` to sanitize newlines before writing to GITHUB_ENV. Fixed all unquoted argument list variables in docker run commands across six steps (project-sources, ort-analyzer, ort-scanner, ort-advisor, ort-evaluator, ort-reporter) by tokenizing ORT_DOCKER_CLI_ARGS, ORT_CLI_ARGS, and per-command CLI args (ORT_CLI_DOWNLOAD_ARGS, ORT_CLI_ANALYZE_ARGS, ORT_CLI_SCAN_ARGS, ORT_CLI_ADVISE_ARGS, ORT_CLI_EVALUATE_ARGS, ORT_CLI_REPORT_ARGS) into bash arrays using the xargs/read-loop pattern. Single-value arguments ORT_DOCKER_IMAGE, ORT_LOG_LEVEL, ORT_ADVISORS, and ORT_REPORT_FORMATS are now properly double-quoted.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed three script-injection findings in hardened/action/action.yml:
1. ort-config step (lines 349-350): Added double quotes around `$ORT_CONFIG_VCS_URL` and `$ORT_CONFIG_VCS_REVISION` in `git remote add` and `git fetch` commands.
2. project-sources step (lines 444-447): Replaced the unsafe string-building pattern (embedding unquoted variables into ORT_CLI_DOWNLOAD_ARGS string) with direct array construction. The `ort_cli_download_args` array is now populated directly with properly double-quoted individual values (`"$PROJECT_VCS_TYPE"`, `"$PROJECT_VCS_URL"`, `"$PROJECT_VCS_REVISION"`, `"$PROJECT_VCS_PATH"`) prepended to any existing args from the xargs-tokenized ORT_CLI_DOWNLOAD_ARGS env var.
3. ort-analyzer step (line 487): Added double quotes around the `-P` flag value: `-P "ort.analyzer.allowDynamicVersions=${ORT_ALLOW_DYNAMIC_VERSIONS}"` to prevent shell metacharacter injection.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed all 6 categories of unquoted variable expansions in action.yml:
1. Init Workspace: `mkdir -p $HOME/$ORT_HOME_PATH/{...}` → expanded brace expansion into individual quoted `mkdir -p "$HOME/$ORT_HOME_PATH/cache" ...` calls and quoted `chmod` path.
2. Init Workspace: `cd $PROJECT_PATH` (×2) → `cd "$PROJECT_PATH"` in both SW_NAME and SW_VERSION command substitutions.
3. Init Workspace: `echo $SW_NAME | sed` → `echo "$SW_NAME" | sed`.
4. ort-config step: `cd $ORT_CONFIG_PATH` → `cd "$ORT_CONFIG_PATH"`.
5. ort-config-url-and-revision step: `cd $ORT_CONFIG_PATH` → `cd "$ORT_CONFIG_PATH"`.
6. All docker run steps (project-sources, ort-analyzer, ort-scanner, ort-advisor, ort-evaluator, ort-reporter): `-o ${PROJECT_PATH/$USER/ort}`, `-i ${ORT_RESULTS_CURRENT_PATH/$USER/ort}`, `-o ${ORT_RESULTS_PATH/$USER/ort}` → all wrapped in double quotes.

### Iteration 4

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed two security findings in hardened/action/action.yml:

1. github-env-injection: Replaced all 11 bare `echo "key=${VAR}" >> "$GITHUB_OUTPUT"` statements in the 'Init Workspace' step with sanitized `printf 'key=%s\n' "$(printf '%s' "${VAR}" | tr -d '\n\r')" >> "$GITHUB_OUTPUT"` forms. This prevents newline injection into GITHUB_OUTPUT from user-controlled `inputs.ort-home-path`.

2. script-injection: Fixed the unquoted `--repository-configuration-file ${ORT_YML_PATH}` in the 'Compute ORT labels' step by adding escaped double-quotes around `${ORT_YML_PATH}` in all three CLI args assignments (ORT_CLI_ANALYZE_ARGS, ORT_CLI_EVALUATE_ARGS, ORT_CLI_REPORT_ARGS), preventing word-splitting on user-controlled path values.

