<!-- markdownlint-disable -->

# Hardening Report: oss-review-toolkit--ort-ci-github-action/v1.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **oss-review-toolkit--ort-ci-github-action/v1.2.0** was hardened automatically. 16 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 'Init Workspace' writes input-derived values to special files with no sanitization. `printenv >> "$GITHUB_ENV"` dumps the whole process environment into GITHUB_ENV. That includes every input-derived env var (ORT_CLI_*_ARGS, ORT_DOCKER_IMAGE, ORT_DOCKER_CLI_ARGS, PROJECT_PATH, SW_NAME, SW_VERSION, ORT_HOME_PATH, POSTGRES_*, HTTP_FILE_SERVER_*, and others) plus any env inherited from the caller. A newline in any of them injects arbitrary env vars such as LD_PRELOAD or BASH_ENV. The step also writes ORT_HOME_PATH-derived paths to GITHUB_OUTPUT with no `printf '%s' | tr -d '\n\r'` step, e.g. `echo "results-path=${ORT_RESULTS_PATH}" >> "$GITHUB_OUTPUT"`.

Locations:

- `action.yml:293`
- `action.yml:325`

### github-env-injection (severity: high)

Step 'Capture ORT config URL and revision' runs `printenv >> "$GITHUB_ENV"`. This writes inherited, workflow-controlled env vars and the input-derived ORT_CONFIG_VCS_URL into GITHUB_ENV with no newline sanitization or random heredoc delimiter.

Locations:

- `action.yml:353`

### github-env-injection (severity: high)

Step 'Compute ORT labels' runs `printenv >> "$GITHUB_ENV"`. This writes ORT_CLI_ANALYZE_ARGS, ORT_CLI_EVALUATE_ARGS and ORT_CLI_REPORT_ARGS to GITHUB_ENV with no sanitization. Those values are built from inputs (ORT_YML_PATH, SW_VERSION, ORT_DOCKER_IMAGE, ORT_CONFIG_VCS_URL, ORT_CLI_*_ARGS) and from inherited env.

Locations:

- `action.yml:384`

### github-env-injection (severity: high)

Step 'Run ORT Analyzer' runs `&& printenv >> "$GITHUB_ENV"`. This writes the whole environment, including ORT_ALLOW_DYNAMIC_VERSIONS (inputs.allow-dynamic-versions) and other inherited input-derived vars, to GITHUB_ENV with no sanitization.

Locations:

- `action.yml:476`

### github-env-injection (severity: high)

Steps 'Run ORT Advisor' and 'Run ORT Evaluator' write `echo "exit-code=${ORT_CLI_EVALUATE_EXIT_CODE}" >> "$GITHUB_OUTPUT"` with no sanitization. In the Advisor step, ORT_CLI_EVALUATE_EXIT_CODE is never set by that script, so its value comes from the inherited environment, which the caller controls. In the Evaluator step, `${VAR:-0}` keeps an inherited value when the command succeeds.

Locations:

- `action.yml:540`
- `action.yml:566`

### script-injection (severity: high)

Rule (b), step 'Init Workspace': workflow-controlled env vars are expanded without quotes, e.g. `mkdir -p $HOME/$ORT_HOME_PATH/{cache,...}; chmod -R aug+w ${HOME}/$ORT_HOME_PATH/`, `SW_NAME=$(cd $PROJECT_PATH; basename ...)` and `SW_NAME_SAFE=$(echo $SW_NAME | sed ...)`.

Locations:

- `action.yml:287`
- `action.yml:318`
- `action.yml:321`

### script-injection (severity: high)

Rule (b), step 'Shallow clone ort-config repository': env vars from inputs and inherited env are expanded without quotes: `ls -A ${ORT_CONFIG_PATH}`, `cd $ORT_CONFIG_PATH`, `git remote add origin $ORT_CONFIG_VCS_URL`, `git fetch -q --depth 1 origin $ORT_CONFIG_VCS_REVISION`.

Locations:

- `action.yml:333`
- `action.yml:339`
- `action.yml:340`

### script-injection (severity: high)

Rule (b), step 'Capture ORT config URL and revision': `cd $ORT_CONFIG_PATH` expands an inherited, workflow-controlled env var without quotes.

Locations:

- `action.yml:348`

### script-injection (severity: high)

Rule (b), step 'Download project sources': input-derived vars are expanded without quotes in the docker command: `$ORT_DOCKER_CLI_ARGS`, `$ORT_DOCKER_IMAGE`, `--$ORT_LOG_LEVEL`, `$ORT_CLI_ARGS`, `-o ${PROJECT_PATH/$USER/ort}` and `${ORT_CLI_DOWNLOAD_ARGS}` (built from inputs vcs-type, vcs-url, vcs-revision and vcs-path).

Locations:

- `action.yml:445`
- `action.yml:451`

### script-injection (severity: high)

Rule (b), step 'Run ORT Analyzer': unquoted expansions `$ORT_DOCKER_CLI_ARGS`, `$ORT_DOCKER_IMAGE`, `--$ORT_LOG_LEVEL`, `-P ort.analyzer.allowDynamicVersions=${ORT_ALLOW_DYNAMIC_VERSIONS}`, `$ORT_CLI_ARGS`, `-i ${PROJECT_PATH/$USER/ort}` and `${ORT_CLI_ANALYZE_ARGS}`.

Locations:

- `action.yml:465`
- `action.yml:468`

### script-injection (severity: high)

Rule (b), step 'Run ORT Scanner': unquoted expansions `$ORT_DOCKER_CLI_ARGS`, `$ORT_DOCKER_IMAGE`, `--$ORT_LOG_LEVEL`, `$ORT_CLI_ARGS` and `${ORT_CLI_SCAN_ARGS}`. All are input-derived values inherited through GITHUB_ENV.

Locations:

- `action.yml:504`

### script-injection (severity: high)

Rule (b), step 'Run ORT Advisor': unquoted expansions `$ORT_DOCKER_CLI_ARGS`, `$ORT_DOCKER_IMAGE`, `--$ORT_LOG_LEVEL`, `$ORT_CLI_ARGS`, `-a $ORT_ADVISORS` (inputs.advisors) and `${ORT_CLI_ADVISE_ARGS}`.

Locations:

- `action.yml:529`
- `action.yml:536`

### script-injection (severity: high)

Rule (b), step 'Run ORT Evaluator': unquoted expansions `$ORT_DOCKER_CLI_ARGS`, `$ORT_DOCKER_IMAGE`, `--$ORT_LOG_LEVEL`, `$ORT_CLI_ARGS` and `${ORT_CLI_EVALUATE_ARGS}`.

Locations:

- `action.yml:556`

### script-injection (severity: high)

Rule (b), step 'Run ORT Reporter': unquoted expansions `$ORT_DOCKER_CLI_ARGS`, `$ORT_DOCKER_IMAGE`, `--$ORT_LOG_LEVEL`, `$ORT_CLI_ARGS`, `-f $ORT_REPORT_FORMATS` (inputs.report-formats) and `${ORT_CLI_REPORT_ARGS}`.

Locations:

- `action.yml:588`
- `action.yml:595`

### script-injection (severity: high)

Rule (b), step 'Remove current-result.yml file if present': `rm -f $ORT_RESULTS_CURRENT_PATH` expands an env var derived from inputs.ort-home-path without quotes.

Locations:

- `action.yml:602`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 1d46bc5..53fd0fe 100644
--- a/action.yml
+++ b/action.yml
@@ -1,4 +1,3 @@
----
 # Copyright (C) 2022 The ORT Project Authors
 #
 # Licensed under the Apache License, Version 2.0 (the "License");
@@ -15,11 +14,9 @@
 #
 # SPDX-License-Identifier: Apache-2.0
 # License-Filename: LICENSE
-
 name: 'GitHub Action for ORT'
 description: 'A GitHub Action workflow to run ORT.'
 author: 'The ORT Project Authors'
-
 inputs:
   vcs-type:
     default: 'git'
@@ -34,10 +31,7 @@ inputs:
     required: false
   vcs-revision:
     default: ''
-    description: |
-      SHA1 or tag to scan (do not use branch names as they can move). 
-      If vcs-type is 'git-repo', SHA1 must be unabbreviated.
-      Tag names must be prefixed with 'refs/tags/'.
+    description: "SHA1 or tag to scan (do not use branch names as they can move). \nIf vcs-type is 'git-repo', SHA1 must be unabbreviated.\nTag names must be prefixed with 'refs/tags/'.\n"
     required: false
   vcs-path:
     default: ''
@@ -188,13 +182,8 @@ inputs:
     required: false
   run:
     default: >
-      cache-dependencies,
-      labels,
-      analyzer,
-      evaluator,
-      advisor,
-      reporter,
-      upload-results
+      cache-dependencies, labels, analyzer, evaluator, advisor, reporter, upload-results
+
     description: |
       Comma-separated list of optional workflow steps to run.
     required: false
@@ -210,7 +199,6 @@ inputs:
       Project version number or release name (use the version from package metadata, not VCS revision).
       By default, the Git short SHA is used.
     required: false
-
 outputs:
   evaluator-exit-code:
     description: The exit code of the evaluator CLI command.
@@ -248,10 +236,26 @@ outputs:
   results-web-app-path:
     description: Path to the Web App Report file.
     value: "${{ steps.ort-init.outputs.results-web-app-path }}"
-
 runs:
   using: 'composite'
   steps:
     - name: Init Workspace
       id: ort-init
       shell: bash
@@ -284,45 +288,65 @@ runs:
         echo -e "\e[1;33m Initializing ORT in GitHub workspace... "
         mkdir -p $HOME/.cache/scancode-tk/; chmod -R aug+w ${HOME}/.cache/ || :
         mkdir -p $HOME/.config/jgit/; chmod -R aug+w ${HOME}/.config/ || :
-        mkdir -p $HOME/$ORT_HOME_PATH/{cache,config,ort-results,scanner/archive/,scanner/provenance/}; chmod -R aug+w ${HOME}/$ORT_HOME_PATH/ || :
+        mkdir -p "$HOME/$ORT_HOME_PATH/cache" "$HOME/$ORT_HOME_PATH/config" "$HOME/$ORT_HOME_PATH/ort-results" "$HOME/$ORT_HOME_PATH/scanner/archive/" "$HOME/$ORT_HOME_PATH/scanner/provenance/"; chmod -R aug+w "${HOME}/$ORT_HOME_PATH/" || :
         mkdir -p $HOME/.gradle/; chmod -R aug+w ${HOME}/.gradle/ || :
         mkdir -p $HOME/.rustup/{cache,download,tmp}; chmod -R aug+w ${HOME}/.rustup/ || :
         mkdir -p $HOME/go/; chmod -R aug+w ${HOME}/go/ || :
         export ORT_CONFIG_PATH=${ORT_CONFIG_PATH:-"$HOME/$ORT_HOME_PATH/config"}
         export ORT_RESULTS_PATH="$HOME/$ORT_HOME_PATH/ort-results"
-        echo "results-path=${ORT_RESULTS_PATH}" >> "$GITHUB_OUTPUT"
+        printf 'results-path=%s\n' "$(printf '%s' "${ORT_RESULTS_PATH}" | tr -d '\n\r')" >> "$GITHUB_OUTPUT"
         export ORT_RESULTS_ADVISOR_PATH="${ORT_RESULTS_PATH}/advisor-result.json"
-        echo "results-advisor-path=${ORT_RESULTS_ADVISOR_PATH}" >> "$GITHUB_OUTPUT"
+        printf 'results-advisor-path=%s\n' "$(printf '%s' "${ORT_RESULTS_ADVISOR_PATH}" | tr -d '\n\r')" >> "$GITHUB_OUTPUT"
         export ORT_RESULTS_ANALYZER_PATH="${ORT_RESULTS_PATH}/analyzer-result.json"
-        echo "results-analyzer-path=${ORT_RESULTS_ANALYZER_PATH}" >> "$GITHUB_OUTPUT"
+        printf 'results-analyzer-path=%s\n' "$(printf '%s' "${ORT_RESULTS_ANALYZER_PATH}" | tr -d '\n\r')" >> "$GITHUB_OUTPUT"
         export ORT_RESULTS_CURRENT_PATH="${ORT_RESULTS_PATH}/current-result.json"
         export ORT_RESULTS_EVALUATED_MODEL_PATH="${ORT_RESULTS_PATH}/evaluated-model.json"
         export ORT_RESULTS_EVALUATOR_PATH="${ORT_RESULTS_PATH}/evaluation-result.json"
-        echo "results-evaluator-path=${
... (truncated)
```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, script-injection, prior-fix

**Notes:**

I rewrote action.yml to fix the reported findings and re-applied the previous harden's changes. Nothing was run or tested.

**Environment and output writes (github-env-injection)**
- Every `printenv >> "$GITHUB_ENV"` is replaced by a small `write_env` helper. It writes only the variables named in that step, and only if they are set. Each one gets its own random heredoc delimiter built from /dev/urandom, so a multi-line value can't add extra env vars.
- The variables saved per step:
  - Init: all of its input-derived env plus the ORT_RESULTS_* paths, ORT_RESULTS_ARTIFACT_NAME, SW_NAME, SW_VERSION, SW_NAME_SAFE and SW_REVISION.
  - Capture: ORT_CONFIG_VCS_URL and ORT_CONFIG_VCS_REVISION.
  - Labels: ORT_CLI_ANALYZE_ARGS, ORT_CLI_EVALUATE_ARGS and ORT_CLI_REPORT_ARGS.
  - Analyzer: ORT_ALLOW_DYNAMIC_VERSIONS and ORT_CLI_ANALYZE_EXIT_CODE.
- Behaviour change: unlike upstream, the whole environment (PATH, HOME, GITHUB_*, and so on) is no longer dumped into GITHUB_ENV. GITHUB_RUN_URL is also no longer saved, since no later step reads it.
- Every GITHUB_OUTPUT write, including the Advisor and Evaluator `exit-code`, now strips newlines with `tr -d '\n\r'`.
- Two upstream bugs are left as they were: the Advisor still reports ORT_CLI_EVALUATE_EXIT_CODE as its exit code, and `results-html-report-path` still gets the scanner path.

**Unquoted variables (script-injection)**
- Single values are now double-quoted. This covers the paths in mkdir/chmod/cd/ln/rm and `[[ -f ]]`, `ls -A`, the git remote URL and revision, the Docker image, `--$ORT_LOG_LEVEL`, the allowDynamicVersions value, `-a $ORT_ADVISORS`, `-f $ORT_REPORT_FORMATS`, the `${X/$USER/ort}` paths, `$HOME` in `--mount`, `-u` and `SSH_AUTH_SOCK`.
- The brace expansions in mkdir are written out as separate quoted paths.
- `basename` and `echo $SW_NAME` now use quoted command substitution and `printf`.
- The argument-list inputs (docker-cli-args, ort-cli-args and each ort-cli-*-args) are turned into arrays with `arr=($VAR)` and passed as `"${arr[@]}"`. That splits and globs exactly as upstream's unquoted use did, without running anything inside the value.
- The download arguments are now built as an array, with each vcs-* value and the project name passed as its own quoted argument.

**Small behaviour changes**
- The project name (SW_NAME) is no longer passed through unquoted `echo`. Repeated spaces in it now each become a dash, and leading or trailing spaces are no longer trimmed.
- `advisors` and `report-formats` are now passed as one argument each. If you write them with spaces after the commas (e.g. `OSV, VulnerableCode`), they are no longer split into separate words.

