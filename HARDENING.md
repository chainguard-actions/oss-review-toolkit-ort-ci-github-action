<!-- markdownlint-disable -->

# Hardening Report: oss-review-toolkit--ort-ci-github-action/v1.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **oss-review-toolkit--ort-ci-github-action/v1.1.0** was hardened automatically. 16 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The composite steps in action.yml reference actions by mutable tag instead of a full 40-character commit SHA: 'uses: actions/cache@v4' (Cache dependencies), 'uses: actions/cache@v4' (Cache ORT scan results), and 'uses: actions/upload-artifact@v4' four times (Upload ORT results, advisor-result, evaluation-result, scan-result). The 'image' input also defaults to the mutable tag 'ghcr.io/oss-review-toolkit/ort:latest'.

Locations:

- `action.yml:382`
- `action.yml:413`
- `action.yml:602`
- `action.yml:609`
- `action.yml:616`
- `action.yml:623`

### github-env-injection (severity: high)

Init Workspace step: 'printenv >> "$GITHUB_ENV"' writes the whole process environment to GITHUB_ENV without sanitising it. That environment includes values taken straight from inputs (ORT_CLI_ARGS, ORT_DOCKER_IMAGE, ORT_HOME_PATH, SW_NAME, SW_VERSION, PROJECT_PATH, db and HTTP credentials, and others) and any env inherited from the calling workflow. A value containing a newline can inject arbitrary environment variables such as LD_PRELOAD or BASH_ENV into later steps. The same step also writes values built from inputs to GITHUB_OUTPUT with no 'printf | tr -d \n\r' step, for example 'echo "results-path=${ORT_RESULTS_PATH}" >> "$GITHUB_OUTPUT"', where ORT_RESULTS_PATH comes from inputs.ort-home-path.

Locations:

- `action.yml:288`
- `action.yml:320`

### github-env-injection (severity: high)

Capture ORT config URL and revision step: 'printenv >> "$GITHUB_ENV"' writes the full process environment to GITHUB_ENV without sanitising it. This includes inherited, workflow-controlled variables and the git remote URL, which comes from inputs.ort-config-repository.

Locations:

- `action.yml:348`

### github-env-injection (severity: high)

Compute ORT labels step: 'printenv >> "$GITHUB_ENV"' writes the environment to GITHUB_ENV without sanitising it. That includes ORT_CLI_ANALYZE_ARGS, ORT_CLI_EVALUATE_ARGS and ORT_CLI_REPORT_ARGS, which are built from inputs (ort-yml-path, sw-version, the image, the config URL and revision) and from GITHUB_ACTOR.

Locations:

- `action.yml:379`

### github-env-injection (severity: high)

Run ORT Analyzer step: '&& printenv >> "$GITHUB_ENV"' writes the whole process environment, including inherited values that come from inputs, to GITHUB_ENV without the newline-stripping step.

Locations:

- `action.yml:471`

### github-env-injection (severity: high)

Run ORT Advisor step: 'echo "exit-code=${ORT_CLI_EVALUATE_EXIT_CODE}" >> "$GITHUB_OUTPUT"' writes ORT_CLI_EVALUATE_EXIT_CODE to GITHUB_OUTPUT without sanitising it. This step never sets that variable, so the value comes from the inherited environment.

Locations:

- `action.yml:535`

### script-injection (severity: high)

Init Workspace step, rule (b): env vars holding input data are expanded without quotes, e.g. 'mkdir -p $HOME/$ORT_HOME_PATH/{cache,...}; chmod -R aug+w ${HOME}/$ORT_HOME_PATH/', 'SW_NAME=$(cd $PROJECT_PATH; ...)', 'SW_NAME_SAFE=$(echo $SW_NAME | sed ...)' and 'SW_NAME=${SW_NAME:-"unknown"}'.

Locations:

- `action.yml:282`
- `action.yml:313`
- `action.yml:316`

### script-injection (severity: high)

Shallow clone ort-config step, rule (b): input data is expanded without quotes, e.g. 'ls -A ${ORT_CONFIG_PATH}', 'cd $ORT_CONFIG_PATH', 'git remote add origin $ORT_CONFIG_VCS_URL' and 'git fetch -q --depth 1 origin $ORT_CONFIG_VCS_REVISION'. These come from inputs.ort-config-path, inputs.ort-config-repository and inputs.ort-config-revision.

Locations:

- `action.yml:328`
- `action.yml:334`

### script-injection (severity: high)

Capture ORT config URL and revision step, rule (b): 'cd $ORT_CONFIG_PATH' expands an inherited value that comes from an input, without quotes.

Locations:

- `action.yml:343`

### script-injection (severity: high)

Download project sources step, rule (b): docker arguments are expanded without quotes, e.g. '$ORT_DOCKER_CLI_ARGS', '$ORT_DOCKER_IMAGE', '--$ORT_LOG_LEVEL', '$ORT_CLI_ARGS', '-o ${PROJECT_PATH/$USER/ort}' and '${ORT_CLI_DOWNLOAD_ARGS}'. ORT_CLI_DOWNLOAD_ARGS is built from inputs.vcs-url, vcs-revision, vcs-path and vcs-type.

Locations:

- `action.yml:440`

### script-injection (severity: high)

Run ORT Analyzer step, rule (b): input data is expanded without quotes, e.g. '$ORT_DOCKER_IMAGE', '--$ORT_LOG_LEVEL', '-P ort.analyzer.allowDynamicVersions=${ORT_ALLOW_DYNAMIC_VERSIONS}', '-i ${PROJECT_PATH/$USER/ort}' and '${ORT_CLI_ANALYZE_ARGS}'.

Locations:

- `action.yml:460`

### script-injection (severity: high)

Run ORT Scanner step, rule (b): docker arguments that come from inputs are expanded without quotes, e.g. '$ORT_DOCKER_CLI_ARGS', '$ORT_DOCKER_IMAGE', '--$ORT_LOG_LEVEL', '$ORT_CLI_ARGS' and '${ORT_CLI_SCAN_ARGS}'.

Locations:

- `action.yml:499`

### script-injection (severity: high)

Run ORT Advisor step, rule (b): input data is expanded without quotes, e.g. '-a $ORT_ADVISORS' (inputs.advisors), '$ORT_DOCKER_IMAGE', '$ORT_CLI_ARGS' and '${ORT_CLI_ADVISE_ARGS}'.

Locations:

- `action.yml:531`

### script-injection (severity: high)

Run ORT Evaluator step, rule (b): input data is expanded without quotes, e.g. '$ORT_DOCKER_CLI_ARGS', '$ORT_DOCKER_IMAGE', '--$ORT_LOG_LEVEL', '$ORT_CLI_ARGS' and '${ORT_CLI_EVALUATE_ARGS}'.

Locations:

- `action.yml:551`

### script-injection (severity: high)

Run ORT Reporter step, rule (b): input data is expanded without quotes, e.g. '-f $ORT_REPORT_FORMATS' (inputs.report-formats), '$ORT_DOCKER_IMAGE', '$ORT_CLI_ARGS' and '${ORT_CLI_REPORT_ARGS}'.

Locations:

- `action.yml:590`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index a85c120..485b3a7 100644
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
@@ -184,13 +178,8 @@ inputs:
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
@@ -206,7 +195,6 @@ inputs:
       Project version number or release name (use the version from package metadata, not VCS revision).
       By default, the Git short SHA is used.
     required: false
-
 outputs:
   evaluator-exit-code:
     description: The exit code of the evaluator CLI command.
@@ -244,10 +232,26 @@ outputs:
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
@@ -285,39 +289,83 @@ runs:
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
-        echo "results-evaluator-path=${ORT_RESULTS_EVALUATOR_PATH}" >> "$GITHUB_OUTPUT"
+        printf 'results-evaluator-path=%s\n' "$(printf '%s' "${ORT_RESULTS_EVALUATOR_PATH}" | tr -d '\n\r')" >> "$GITHUB_OUTPUT"
         export ORT_RESULTS_HTML_REPORT_PATH="${ORT_RESULTS_HTML_REPORT_PATH}/ort-results/scan-report.html"
-        echo "results-html-report-path=${ORT_RESULTS_SCANNER_PATH}" >> "$GITHUB_OUTPUT"
+        printf 'results-html-report-path=%s\n' "$(printf '%s' "${ORT_RESULTS_SCANNER_PATH}" | tr -d '\n\r')" >> "$GITHUB_OUTPUT"
         export ORT_RESULTS_SCANNER_PATH="${ORT_RESULTS_PATH}/scan-result.json"
-        echo "results-scanner-path=${ORT_RESULTS_SCANNER_PATH}" >> "$GITHUB_OUTPUT"
+        printf 'results-scanner-path=%s\n' "$(printf '%s' "${ORT_RESULTS_SCANNER_PATH}" | tr -d '\n\r')" >
... (truncated)
```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection, script-injection, prior-fix

**Notes:**

I fixed every finding in action.yml. Nothing has been run or tested yet.

**Pinning**
- actions/cache is pinned to 0057852bfaa89a56745cba8c7296529d2fc39830 (v4), in both places it is used.
- actions/upload-artifact is pinned to ea165f8d65b6e75b540449e92b4886f43607fa02 (v4), in all four places.
- The default for the `image` input is now `ghcr.io/oss-review-toolkit/ort:latest@sha256:fadf830125e7cfba5d54376809ea4763e88671802cbb83009d2fb3f3caeedc00`. The tag stays inline, so the image no longer changes when `latest` moves.

**Writes to GITHUB_ENV**
- The four steps that dumped the whole environment into GITHUB_ENV now write only the variables later steps use. Each one uses the multi-line form with a fresh random delimiter from /dev/urandom, so a newline in a value cannot add extra variables.
- The Init step's list includes `HOME`, because the scan-results cache path reads `env.HOME`. It also includes the database and HTTP server credentials and all the ORT_* paths and arguments, as upstream did.
- The other three steps write only what they produce: the ort-config URL and revision, the three analyze/evaluate/report argument strings, and the analyzer exit code plus the allow-dynamic-versions setting.

**Writes to GITHUB_OUTPUT**
- All values written to GITHUB_OUTPUT have newlines stripped first, including both `exit-code` outputs.

**Unquoted variables (script injection)**
- Single values are now double-quoted: paths, `cd` targets, the git remote URL and revision, the image, `--$ORT_LOG_LEVEL`, the allow-dynamic-versions setting, `-a` advisors, `-f` report formats, the ssh socket, and the bind mount.
- The argument-list inputs (docker-cli-args, ort-cli-args and the ort-cli-*-args) are split into bash arrays by unquoted assignment. That splits on whitespace and expands globs exactly as upstream's unquoted use did, without evaluating anything in the value.
- The download arguments are now built as an array, one quoted value per flag.

**Behaviour differences to check**
- Advisors and report formats now reach ORT as one argument each. If a user writes them with spaces after the commas (`OSV, VulnerableCode`), ORT gets one value instead of the two words upstream produced.
- Because each download flag value is now one argument, a value containing spaces is no longer split the way upstream split it.
- The safe software name still collapses and trims whitespace like upstream's unquoted `echo` did, but it no longer expands glob characters in the name.

**Upstream bugs left as they were**
- The Advisor step's `exit-code` output still reports the Evaluator's exit code, which is empty at that point. As a result, `fail-on: issues` never fails the action. Fixing it would change behaviour, so it is only sanitised.
- `results-html-report-path` still reports the scanner path, which is not yet set when it is written.

