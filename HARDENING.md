<!-- markdownlint-disable -->

# Hardening Report: oss-review-toolkit--ort-ci-github-action/v1.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **oss-review-toolkit--ort-ci-github-action/v1.2.0** was hardened automatically. 2 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Four `run:` blocks call `printenv >> "$GITHUB_ENV"` which dumps the entire process environment — including variables set from untrusted inputs such as SW_NAME (${{ inputs.sw-name }}), SW_VERSION (${{ inputs.sw-version }}), ORT_CLI_ARGS (${{ inputs.ort-cli-args }}), ORT_DOCKER_IMAGE (${{ inputs.image }}), POSTGRES_PASSWORD (${{ inputs.db-password }}), HTTP_FILE_SERVER_PASSWORD (${{ inputs.http-file-server-password }}), and many more — into $GITHUB_ENV without any newline sanitization (printf '%s' ... | tr -d '\n\r'). An attacker who controls any of these inputs can inject newlines to define arbitrary environment variables for subsequent steps.

Locations:

- `action.yml:253`
- `action.yml:281`
- `action.yml:316`
- `action.yml:381`

### script-injection (severity: high)

Multiple `run:` blocks expand input-derived environment variables unquoted in shell commands (rule b: unquoted shell variable expansion of untrusted data). Variables sourced from `inputs.*` via env: blocks are used without double-quoting: `$ORT_DOCKER_CLI_ARGS` (from inputs.docker-cli-args), `$ORT_DOCKER_IMAGE` (from inputs.image), `--$ORT_LOG_LEVEL` (from inputs.log-level), `$ORT_CLI_ARGS` (from inputs.ort-cli-args), `${ORT_CLI_DOWNLOAD_ARGS}` (built from inputs.vcs-*), `${ORT_CLI_ANALYZE_ARGS}` (from inputs.ort-cli-analyze-args), `$ORT_ADVISORS` (from inputs.advisors), `$ORT_REPORT_FORMATS` (from inputs.report-formats), `${ORT_CLI_ADVISE_ARGS}`, `${ORT_CLI_EVALUATE_ARGS}`, `${ORT_CLI_REPORT_ARGS}`. These unquoted expansions allow shell metacharacters (`;`, `|`, `&`, `$(...)`) in input values to be interpreted by the shell, enabling command injection. Affected steps: 'Download project sources' (docker run with $ORT_DOCKER_CLI_ARGS $ORT_DOCKER_IMAGE --$ORT_LOG_LEVEL $ORT_CLI_ARGS), 'Run ORT Analyzer', 'Run ORT Scanner', 'Run ORT Advisor' (-a $ORT_ADVISORS), 'Run ORT Evaluator', 'Run ORT Reporter' (-f $ORT_REPORT_FORMATS).

Locations:

- `action.yml:358`
- `action.yml:375`
- `action.yml:399`
- `action.yml:430`
- `action.yml:455`
- `action.yml:479`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed all four `printenv >> "$GITHUB_ENV"` calls by replacing them with explicit, newline-sanitized writes (using `printf '%s' "$VAR" | tr -d '\n\r'`) of only the specific variables needed by subsequent steps. Fixed all six docker run command blocks by converting multi-argument string variables into bash arrays (`array=($VAR)` then `"${array[@]}"`) and properly double-quoting single-value variables like `$ORT_DOCKER_IMAGE`, `--${ORT_LOG_LEVEL}`, `$ORT_ADVISORS`, and `$ORT_REPORT_FORMATS`.

### Iteration 2

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed two security findings in action.yml:

1. github-env-injection (high): Replaced all 11 unsanitized `echo "key=${VAR}" >> "$GITHUB_OUTPUT"` writes in the 'Init Workspace' (ort-init) step with sanitized versions using `printf 'key=%s\n' "$(printf '%s' "${VAR}" | tr -d '\n\r')" >> "$GITHUB_OUTPUT"`. This prevents newline injection via the attacker-controlled `inputs.ort-home-path` input from poisoning $GITHUB_OUTPUT with arbitrary key=value pairs.

2. script-injection (high): Added double quotes around `$ORT_CONFIG_VCS_URL` and `$ORT_CONFIG_VCS_REVISION` in the 'Shallow clone ort-config repository' (ort-config) step's `git remote add origin` and `git fetch` commands. This prevents shell metacharacter injection from the attacker-controlled `inputs.ort-config-repository` and `inputs.ort-config-revision` inputs.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed 5 instances of unquoted shell variable expansions of untrusted input-derived data in action.yml:
1. `cd $PROJECT_PATH` → `cd "$PROJECT_PATH"` in SW_NAME assignment (Init Workspace step)
2. `cd $PROJECT_PATH` → `cd "$PROJECT_PATH"` in SW_VERSION assignment (Init Workspace step)
3. `echo $SW_NAME` → `echo "$SW_NAME"` in SW_NAME_SAFE assignment (Init Workspace step)
4. `cd $ORT_CONFIG_PATH` → `cd "$ORT_CONFIG_PATH"` in Shallow clone ort-config step
5. `cd $ORT_CONFIG_PATH` → `cd "$ORT_CONFIG_PATH"` in Capture ORT config URL and revision step

All variables (PROJECT_PATH, SW_NAME, ORT_CONFIG_PATH) are already passed via the step's env: block from inputs, so no structural changes were needed — only adding double quotes around the variable expansions to prevent word splitting and glob expansion on attacker-controlled values.

