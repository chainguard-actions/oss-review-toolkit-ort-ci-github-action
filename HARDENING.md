<!-- markdownlint-disable -->

# Hardening Report: oss-review-toolkit--ort-ci-github-action/v1.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **oss-review-toolkit--ort-ci-github-action/v1.1.0** was hardened automatically. 2 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in action.yml are pinned to mutable version tags (`@v4`) instead of immutable 40-character SHA commit hashes. This exposes the action to supply-chain attacks if the upstream action is compromised or the tag is moved. Failing references:
- `uses: actions/cache@v4` (Cache dependencies step)
- `uses: actions/cache@v4` (Cache ORT scan results step)
- `uses: actions/upload-artifact@v4` (Upload ORT results step)
- `uses: actions/upload-artifact@v4` (Upload ORT advisor-result.json step)
- `uses: actions/upload-artifact@v4` (Upload ORT evaluation-result.json step)
- `uses: actions/upload-artifact@v4` (Upload ORT scan-result.json step)

Locations:

- `action.yml:295`
- `action.yml:318`
- `action.yml:430`
- `action.yml:438`
- `action.yml:446`
- `action.yml:454`

### github-env-injection (severity: high)

Four `run:` steps call `printenv >> "$GITHUB_ENV"` which dumps the entire process environment — including all caller-controlled `inputs.*`-derived env vars (e.g. `ORT_CLI_ARGS`, `SW_NAME`, `SW_VERSION`, `POSTGRES_PASSWORD`, `HTTP_FILE_SERVER_TOKEN`, `ORT_DOCKER_IMAGE`, etc.) — directly to `$GITHUB_ENV` without any newline sanitization (`printf '%s' ... | tr -d '\n\r'`). An attacker-controlled input value containing embedded newlines (e.g. `\nFOO=bar`) can inject arbitrary key=value pairs into the GitHub environment, potentially overwriting security-sensitive variables for all subsequent steps in the calling workflow. Affected steps:
1. 'Init Workspace' (id: ort-init): `printenv >> "$GITHUB_ENV"` — env vars include `HTTP_FILE_SERVER_PASSWORD`, `HTTP_FILE_SERVER_TOKEN`, `ORT_CLI_ARGS`, `SW_NAME`, `SW_VERSION`, `POSTGRES_PASSWORD`, and many others sourced from `inputs.*`.
2. 'Capture ORT config URL and revision' (id: ort-config-url-and-revision): `printenv >> "$GITHUB_ENV"` — propagates all inherited env including `ORT_CONFIG_VCS_URL` and `ORT_CONFIG_VCS_REVISION` from `inputs.*`.
3. 'Compute ORT labels' (id: ort-labels): `printenv >> "$GITHUB_ENV"` — propagates `ORT_CLI_ANALYZE_ARGS`, `ORT_CLI_EVALUATE_ARGS`, `ORT_CLI_REPORT_ARGS` which are built from inherited untrusted env vars.
4. 'Run ORT Analyzer' (id: ort-analyzer): `printenv >> "$GITHUB_ENV"` — propagates `ORT_ALLOW_DYNAMIC_VERSIONS` from `inputs.allow-dynamic-versions` and all other inherited env vars.

Locations:

- `action.yml:230`
- `action.yml:258`
- `action.yml:290`
- `action.yml:385`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection

**Notes:**

Fixed 6 unpinned `uses:` references by replacing mutable version tags with immutable 40-character SHA commit hashes: actions/cache@v4 → @0057852bfaa89a56745cba8c7296529d2fc39830 (2 occurrences) and actions/upload-artifact@v4 → @ea165f8d65b6e75b540449e92b4886f43607fa02 (4 occurrences). Fixed 4 github-env-injection vulnerabilities by replacing `printenv >> "$GITHUB_ENV"` with explicit, sanitized writes: (1) ort-init step now uses a loop writing only the specific needed variables with `printf '%s' "${!VAR}" | tr -d '\n\r'` sanitization; (2) ort-config-url-and-revision step writes only ORT_CONFIG_VCS_URL and ORT_CONFIG_VCS_REVISION with newline stripping; (3) ort-labels step writes only ORT_CLI_ANALYZE_ARGS, ORT_CLI_EVALUATE_ARGS, ORT_CLI_REPORT_ARGS with newline stripping; (4) ort-analyzer step writes only ORT_CLI_ANALYZE_EXIT_CODE with newline stripping.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all 8 script-injection findings in action.yml:

1. **Init Workspace (ort-init)**: Quoted all `$ORT_HOME_PATH`, `$PROJECT_PATH`, and `$SW_NAME` expansions. Changed `mkdir -p $HOME/$ORT_HOME_PATH/{...}` brace expansion to explicit quoted paths. Changed `echo $SW_NAME | sed` to `printf '%s' "$SW_NAME" | sed`. Quoted `cd $PROJECT_PATH` and backtick subshell in git URL detection.

2. **Shallow clone ort-config (ort-config)**: Quoted `$ORT_CONFIG_PATH` in `ls -A`, `cd`, `git remote add origin`, and `git fetch` commands. Now uses `"$ORT_CONFIG_VCS_URL"` and `"$ORT_CONFIG_VCS_REVISION"` with double quotes.

3. **Download project sources (project-sources)**: Used `read -ra` arrays for `ORT_DOCKER_CLI_ARGS`, `ORT_CLI_ARGS`, and `ORT_CLI_DOWNLOAD_ARGS`. Quoted `$ORT_DOCKER_IMAGE` and `--${ORT_LOG_LEVEL}`. Quoted `$HOME` and path substitutions.

4. **Run ORT Analyzer (ort-analyzer)**: Used `read -ra` arrays for multi-word args. Quoted `$ORT_DOCKER_IMAGE`, `--${ORT_LOG_LEVEL}`, `-P` argument, and all path substitutions. Quoted file test and ln arguments.

5. **Run ORT Scanner (ort-scanner)**: Used `read -ra` arrays for multi-word args. Quoted `$ORT_DOCKER_IMAGE`, `--${ORT_LOG_LEVEL}`, SSH socket volume mount, and all path substitutions. Quoted file test and ln arguments.

6. **Run ORT Advisor (ort-advisor)**: Used `read -ra` arrays for multi-word args. Quoted `$ORT_DOCKER_IMAGE`, `--${ORT_LOG_LEVEL}`, `-a "$ORT_ADVISORS"`, and all path substitutions. Quoted file test and ln arguments.

7. **Run ORT Evaluator (ort-evaluator)**: Used `read -ra` arrays for multi-word args. Quoted `$ORT_DOCKER_IMAGE`, `--${ORT_LOG_LEVEL}`, and all path substitutions. Quoted file test and ln arguments.

8. **Run ORT Reporter (ort-reporter)**: Used `read -ra` arrays for multi-word args. Quoted `$ORT_DOCKER_IMAGE`, `--${ORT_LOG_LEVEL}`, `-f "$ORT_REPORT_FORMATS"`, and all path substitutions.

The `read -ra` array approach is used for intentionally multi-word variables (CLI arg lists) to preserve word-splitting behavior while preventing shell metacharacter injection from attacker-controlled input values.

### Iteration 3

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed two security findings in action.yml:
1. github-env-injection (line 248): Replaced all direct `echo "key=${VAR}" >> "$GITHUB_OUTPUT"` writes in the 'Init Workspace' step with sanitized versions. Each path variable (results-path, results-advisor-path, results-analyzer-path, results-evaluator-path, results-html-report-path, results-scanner-path, results-sbom-cyclonedx-xml-path, results-sbom-cyclonedx-json-path, results-sbom-spdx-json-path, results-sbom-spdx-yml-path, results-web-app-path) is now sanitized with `printf '%s' "${VAR}" | tr -d '\n\r'` before being written to $GITHUB_OUTPUT using `printf 'key=%s\n'`.
2. script-injection (line 340): Changed `cd $ORT_CONFIG_PATH` to `cd "$ORT_CONFIG_PATH"` in the 'Capture ORT config URL and revision' step to prevent shell metacharacter interpretation from the user-controlled ORT_CONFIG_PATH variable.

