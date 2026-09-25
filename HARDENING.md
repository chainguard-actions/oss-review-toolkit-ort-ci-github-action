<!-- markdownlint-disable -->

# Hardening Report: oss-review-toolkit--ort-ci-github-action/v1.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **oss-review-toolkit--ort-ci-github-action/v1.1.0** was hardened automatically. 3 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Four run: blocks call `printenv >> "$GITHUB_ENV"` without any sanitization, dumping the entire process environment — including all inputs.*-derived env vars (SW_NAME, SW_VERSION, ORT_CLI_ARGS, ORT_YML_PATH, PROJECT_PATH, HTTP_FILE_SERVER_PASSWORD, POSTGRES_PASSWORD, etc.) — directly into $GITHUB_ENV. Any newline characters in those values can inject arbitrary environment variable definitions into subsequent steps. This affects: (1) 'Init Workspace' step, (2) 'Capture ORT config URL and revision' step, (3) 'Compute ORT labels' step, and (4) 'Run ORT Analyzer' step.

Locations:

- `action.yml:243`
- `action.yml:271`
- `action.yml:302`
- `action.yml:362`

### unpinned-uses (severity: high)

All six uses: references in action.yml use mutable version tags (@v4) instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if those upstream actions are compromised or their tags are moved. Failing references: actions/cache@v4 (×2), actions/upload-artifact@v4 (×4).

Locations:

- `action.yml:307`
- `action.yml:318`
- `action.yml:449`
- `action.yml:456`
- `action.yml:463`
- `action.yml:470`

### script-injection (severity: high)

Multiple run: blocks expand workflow-controllable env vars (sourced from inputs.*) without double-quoting, violating sub-rule (b). Unquoted expansions allow shell metacharacter injection (`;`, `|`, `&`, `$(...)`, glob chars, whitespace). Offending unquoted expansions include: `$ORT_DOCKER_CLI_ARGS` and `$ORT_DOCKER_IMAGE` (passed as bare unquoted args to docker run in the Download, Analyzer, Scanner, Advisor, Evaluator, and Reporter steps); `$ORT_CLI_ARGS`, `$ORT_LOG_LEVEL`, `$ORT_ADVISORS`, `$ORT_REPORT_FORMATS`, `$SW_NAME_SAFE`, `$PROJECT_VCS_TYPE`, `$PROJECT_VCS_URL`, `$PROJECT_VCS_REVISION`, `$PROJECT_VCS_PATH` — all derived from inputs.* env vars set in the env: block.

Locations:

- `action.yml:337`
- `action.yml:338`
- `action.yml:355`
- `action.yml:356`
- `action.yml:385`
- `action.yml:386`
- `action.yml:407`
- `action.yml:408`
- `action.yml:430`
- `action.yml:431`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, unpinned-uses, script-injection

**Notes:**

Fixed all three findings in hardened/action/action.yml:

1. github-env-injection (4 locations): Replaced all `printenv >> "$GITHUB_ENV"` calls with selective, sanitized exports. Each variable is sanitized with `printf '%s' "${!_var}" | tr -d '\n\r'` before writing to GITHUB_ENV. The Init Workspace step now exports only the ~40 specific variables needed by subsequent steps; the other three steps export only the 1-3 variables they compute.

2. unpinned-uses (6 locations): Pinned actions/cache@v4 to SHA 0057852bfaa89a56745cba8c7296529d2fc39830 (×2) and actions/upload-artifact@v4 to SHA ea165f8d65b6e75b540449e92b4886f43607fa02 (×4), preserving the tag in a comment.

3. script-injection (10 locations across 6 docker run steps - Download, Analyzer, Scanner, Advisor, Evaluator, Reporter): List-style args ($ORT_DOCKER_CLI_ARGS, $ORT_CLI_ARGS) are now tokenized via xargs into bash arrays and expanded as "${arr[@]}". Single-value vars ($ORT_DOCKER_IMAGE, $ORT_LOG_LEVEL, $ORT_ADVISORS, $ORT_REPORT_FORMATS) are now double-quoted.

### Iteration 2

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed three security findings in hardened/action/action.yml:

1. github-env-injection: Replaced all 10 direct `echo "key=${VAR}" >> "$GITHUB_OUTPUT"` writes in the 'Init Workspace' step with `printf 'key=%s\n' "$(printf '%s' "${VAR}" | tr -d '\n\r')"` to sanitize newlines before writing to GITHUB_OUTPUT, preventing injection of arbitrary key=value pairs via attacker-controlled newlines in inputs.ort-home-path.

2. script-injection (git commands): Added double-quotes around the three unquoted variables in the 'Shallow clone ort-config repository' step: `cd "$ORT_CONFIG_PATH"`, `git remote add origin "$ORT_CONFIG_VCS_URL"`, and `git fetch -q --depth 1 origin "$ORT_CONFIG_VCS_REVISION"`.

3. script-injection (docker run args): For each of the five docker run steps, replaced unquoted `${ORT_CLI_*_ARGS}` expansions with properly tokenized bash arrays using the xargs pattern (quote-aware tokenization), then expanded them as `"${array[@]}"`. This handles the list-type inputs correctly while preventing shell metacharacter injection.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed all unquoted shell variable expansions identified in the script-injection finding:
1. ort-init step (lines ~271-273): Quoted `$PROJECT_PATH` in both `cd` commands and `$SW_NAME` in the `echo` command used for SW_NAME_SAFE computation.
2. ort-config-url-and-revision step (line ~310): Quoted `$ORT_CONFIG_PATH` in the `cd` command.
3. project-sources step (line ~349): Quoted `${PROJECT_PATH/$USER/ort}` in the docker run `-o` argument.
4. ort-analyzer step (lines ~393, ~421): Quoted `${PROJECT_PATH/$USER/ort}` in the docker run `-i` argument and wrapped `ort.analyzer.allowDynamicVersions=${ORT_ALLOW_DYNAMIC_VERSIONS}` in double quotes for the `-P` flag.

