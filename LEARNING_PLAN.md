# Azure DevOps YAML Pipelines: Learning Plan

## Table of contents

- [Overview](#overview)
- [Proposed repository layout](#proposed-repository-layout)
- [Sample application and companion repositories](#sample-application-and-companion-repositories)
- [Module sequence](#module-sequence)
- [Capstone project](#capstone-project)
- [Reference appendices](#reference-appendices)
- [Azure DevOps setup checklist](#azure-devops-setup-checklist)
- [Implementation roadmap](#implementation-roadmap)
- [Summary and open questions](#summary-and-open-questions)

## Overview

This repository will teach Azure DevOps YAML Pipelines by building small, runnable examples, observing their behavior, and explaining why the service evaluates them that way. It assumes comfort with Git, CI/CD concepts, and Bash or PowerShell, but no prior Azure DevOps YAML experience.

### Prerequisites

- An Azure DevOps organization and a project in which the learner can create and run pipelines.
- Permission to create repositories, feeds, environments, variable groups, and service connections, or an administrator who can perform the setup checklist.
- Access to at least one Microsoft-hosted or self-hosted agent pool. As a planning baseline, the current Azure DevOps Services free grant for a private project is one Microsoft-hosted parallel job with up to 60 minutes per run and 1,800 minutes per month; the self-hosted grant and public-project entitlements differ. These grants and public-project policies can change, so verify **Configure and pay for parallel jobs** for the learner’s organization before planning labs.
- Git, a terminal, and the .NET SDK version selected for the sample app. The local SDK is for understanding and optionally running the sample; Azure Pipelines tasks will perform the pipeline build.
- For cross-repository GitHub exercises, a GitHub repository and an appropriately scoped Azure DevOps service connection. Azure Repos-only exercises do not require it.

### Time estimate

Plan for approximately **30–40 hours**: 1–2 hours for setup, 1–2 hours per core module, and 4–6 hours for the capstone. Time spent waiting for approvals, troubleshooting organization permissions, or provisioning agents is additional.

> **Accuracy policy:** Azure DevOps service behavior, task major versions, hosted images, limits, and free-tier entitlements can change. The documentation list at the end identifies authoritative Microsoft Learn pages; each implemented module should link the relevant page directly. Before implementing a lab, verify the current page and record the verification date in that module. In particular, recheck public-project entitlements and their announced retirement/transition policy, as well as hosted limits and task versions. Do not use an old limit or task version merely because it appears in an example.

## Proposed repository layout

```text
.
├── README.md
├── LEARNING_PLAN.md
├── docs/
│   ├── setup.md
│   └── reference/
│       ├── variables.md
│       ├── expressions.md
│       ├── checkout-paths.md
│       ├── artifacts.md
│       ├── timeouts.md
│       └── permissions-and-troubleshooting.md
├── modules/
│   ├── 00-first-run/
│   │   ├── README.md
│   │   └── azure-pipelines.yml
│   ├── 01-yaml-and-pipeline-anatomy/
│   ├── 02-expressions-and-processing-order/
│   ├── 03-variables-and-environment/
│   ├── 04-stages-jobs-steps-and-dependencies/
│   ├── 05-parameters-templates-and-reuse/
│   ├── 06-agents-workspaces-and-job-strategies/
│   ├── 07-conditions-timeouts-and-cancellation/
│   ├── 08-checkout-and-workspace-paths/
│   ├── 09-cross-repository-checkout-and-permissions/
│   ├── 10-pipeline-artifacts/
│   ├── 11-pipeline-resources-and-artifact-comparison/
│   ├── 12-consuming-azure-artifacts-feeds/
│   ├── 13-publishing-packages-and-cross-project-access/
│   ├── 14-deployment-jobs-environments-and-approvals/
│   └── 15-debugging-and-pipeline-security/
├── src/
│   ├── Tutorial.Library/
│   └── Tutorial.Console/
├── tests/
│   └── Tutorial.Library.Tests/
├── scripts/
│   ├── dump-env.sh
│   └── dump-env.ps1
├── templates/
│   └── shared/                 # Local examples; reusable repo is separate
├── capstone/
│   ├── README.md
│   └── azure-pipelines.yml
└── solutions/                  # Separate instructor/reference solutions
```

Each module will be self-contained where possible: a short README, one or more focused pipeline YAML files, and only the sample files needed to observe the concept. Deliberately broken variants should be named clearly (for example, `broken-expression.yml`) and must not be configured as the default pipeline. Solution files will live separately from learner instructions; they are teaching aids, not security-protected answers.

The two companion Azure DevOps repositories are:

1. **`azure-pipelines-shared-templates`** — a small set of step, job, and stage templates; an `extends` template; and a README describing parameters and versioned refs. Used to demonstrate template repository resources and cross-repo template reuse.
2. **`azure-pipelines-feed-consumer`** — a minimal consumer application and pipeline that checks out the shared-templates repo where needed, authenticates to the tutorial feed, restores the sample package, and publishes a runnable output. Used to demonstrate cross-repository and cross-pipeline boundaries.

The tutorial repository itself publishes the sample library package. This keeps the number of repositories small while still demonstrating a producer pipeline, a feed, shared templates, and a separate consumer pipeline.

## Sample application and companion repositories

Use a tiny **.NET class library plus console application**:

- `Tutorial.Library` contains a small greeting or formatting API and is packed as a NuGet package.
- `Tutorial.Console` references the library, prints a deterministic result, and produces a build output suitable for pipeline-artifact labs.
- `Tutorial.Library.Tests` contains a few unit tests and no external service dependency.

This is cheap to build, familiar to many learners, and gives the same source a meaningful test, package, feed restore, and downloadable build output. Keep the project deliberately small; pipeline behavior—not application design—is the subject. Pin the SDK in a `global.json` only after selecting a currently supported SDK, and use a current supported .NET task/SDK approach when implementation begins.

The separate consumer repository references a released package version from an Azure Artifacts feed rather than using a project reference. That makes feed authentication, version selection, and producer/consumer permissions observable instead of hidden by a local build.

## Module sequence

The modules are ordered so that each one introduces only a few new pipeline concepts. Unless stated otherwise, each module has a `README.md`, a runnable `azure-pipelines.yml`, a `solutions/` answer, and any small supporting scripts or source files listed below.

### Module 00 — First run and safe lab setup

- **Objectives**
  - Create a minimal pipeline and identify its run, job, and step in Azure DevOps.
  - Run a script on an available agent and locate its log.
  - Distinguish YAML stored in a repository from settings configured in Azure DevOps.
- **Concepts and deep dive:** Pipeline creation, YAML path, queue/run, agent allocation, task logs, hosted versus self-hosted pool overview. Deep dive follows a run from YAML selection through agent execution at a high level.
- **Walkthrough files:** `modules/00-first-run/README.md`, `modules/00-first-run/azure-pipelines.yml`, `docs/setup.md`.
- **Labs:** Run a one-step pipeline; break it by changing the YAML path or introducing an indentation error. Observe that validation/queueing fails before the script runs and compare the reported location.
- **Challenge:** Add a second named step and explain which log entries identify each step. Answer: `solutions/00-first-run-solution.yml`.
- **Common errors and fixes:** `No hosted parallelism has been purchased or granted` (or the current equivalent): check the organization’s parallel-job grant and pool; YAML validation errors: inspect indentation and the line reported by the editor/service.
- **Checkpoint:** Trigger a successful run, identify the agent and step logs, and explain where the YAML came from.
- **Estimate:** 1–1.5 hours.
- **Depends on:** Azure DevOps organization/project access.

### Module 01 — YAML syntax and pipeline anatomy

- **Objectives**
  - Identify `trigger`, `pr`, `schedules`, `pool`, `parameters`, `variables`, `resources`, and `stages`.
  - Convert between single-stage/single-job shorthand and explicit stage/job/step hierarchy.
  - Correct common YAML scalar, list, map, quote, and indentation mistakes.
- **Concepts and deep dive:** Pipeline root keys, defaults, shorthand expansion, lists versus mappings, quoted strings and boolean-looking values, literal `|` versus folded `>` multiline strings, comments, trigger branch filters, and schedule syntax. Explain which structure is YAML syntax and which is Azure Pipelines schema.
- **Walkthrough files:** `modules/01-yaml-and-pipeline-anatomy/README.md`, `azure-pipelines.yml`, `shorthand.yml`, `explicit.yml`.
- **Labs:** Run equivalent shorthand and explicit pipelines; break indentation and replace a quoted string with a boolean-looking scalar. Observe validation/schema differences and fix them.
- **Challenge:** Add a schedule and a branch-filtered trigger without changing the job graph. Answer: `solutions/01-anatomy-solution.yml`.
- **Common errors and fixes:** `Unexpected value ...` commonly means a key is at the wrong schema level or misspelled; a YAML parser message about a mapping/sequence usually points to indentation or a missing quote/colon. Use the reported line and validate the Azure Pipelines schema, not just generic YAML.
- **Checkpoint:** Given a short pipeline, label each root key, describe its expanded hierarchy, and repair a deliberately invalid snippet.
- **Estimate:** 1.5–2 hours.
- **Depends on:** Module 00.

### Module 02 — Expressions and processing order

- **Objectives**
  - Distinguish `${{ }}`, `$(var)`, and `$[ ]` by syntax and evaluation time.
  - Predict which values are available during template expansion versus runtime.
  - Write basic `eq`, `ne`, `and`, `or`, and `contains` expressions and use `${{ if }}` / `${{ each }}`.
  - Select the right syntax for a concrete pipeline requirement.
- **Concepts and deep dive:** Azure Pipelines template expansion, compile-time plan construction, runtime condition evaluation, macro substitution immediately before a task, and logging-command updates after a task begins. Explain that these phases are not interchangeable and that a variable unavailable during template expansion cannot be made available by changing punctuation. Runtime expressions can access more runtime variables than template expressions, but cannot access runtime parameters.
- **Walkthrough files:** `modules/02-expressions-and-processing-order/README.md`, `azure-pipelines.yml`, `expression-timing.yml`.
- **Labs:** Print the same variable through template, macro, and runtime-expression contexts; use a runtime parameter to include/exclude a step; break it by using `$(...)` where a template-time decision is required. Observe an empty/unavailable value or a step that remains in the compiled plan, then correct it.
- **Challenge:** Choose expression syntax for “select a template at queue time,” “use a variable set by an earlier step,” and “gate a job based on a dependency.” Answer: `solutions/02-expressions-solution.yml`.
- **Common errors and fixes:** A template expression cannot read many runtime-only variables; move the decision to a runtime `condition`, or pass a parameter/compile-time value. A runtime expression used as a condition/value must occupy the supported expression position and return the expected type.
- **Checkpoint:** For a sample run, draw the evaluation timeline and correctly predict the expanded plan, substituted task input, and condition result.
- **Estimate:** 2–2.5 hours.
- **Depends on:** Module 01.

### Module 03 — Predefined variables, user variables, and environment

- **Objectives**
  - Find representative `Agent.*`, `Build.*`, `System.*`, `Pipeline.*`, `Environment.*`, `Resources.*`, and deployment/strategy variables.
  - Describe variable scope (pipeline, stage, job, variable group) and syntax-time availability.
  - Map variables to script environment names and explicitly map secrets and `System.AccessToken`.
  - Set and pass output variables across steps, jobs, and stages.
- **Concepts and deep dive:** Explore `Build.SourcesDirectory`, `Build.ArtifactStagingDirectory`, `Build.BuildId`, `Build.Reason`, `Build.SourceBranch`, `Agent.TempDirectory`, `Pipeline.Workspace`, `System.DefaultWorkingDirectory`, `System.AccessToken`, `System.Debug`, and `System.PullRequest.*`. Compare predefined variables, queue-time/user variables, variable groups, secrets, and output variables. Explain name-to-environment mapping (uppercase, dots to underscores), case behavior on Windows versus Unix, and why secret variables and the job token are not automatically exposed as ordinary environment variables. Cover `##vso[task.setvariable ...]`, `isOutput=true`, `dependencies.<job>.outputs[...]`, and `stageDependencies.<stage>.<job>.outputs[...]`.
- **Walkthrough files:** `modules/03-variables-and-environment/README.md`, `azure-pipelines.yml`, `dump-env.sh`, `dump-env.ps1`.
- **Labs:** Dump `env | sort` in Bash and `Get-ChildItem env:` in PowerShell at pipeline, job, and later-step points; compare outputs and scopes. Set a same-job value, then an output across jobs and stages. Break it by omitting `name` or `isOutput=true`; observe that the consumer reference is empty/unavailable.
- **Challenge:** Pass a generated version between stages without printing a secret. Answer: `solutions/03-variables-solution.yml`.
- **Common errors and fixes:** A missing macro commonly expands to an empty string; check spelling, scope, and evaluation time. Output-variable references require a named producer step and the correct dependency syntax. A secret is masked in logs but masking is not a substitute for avoiding output; map it only to the task’s `env:` input.
- **Checkpoint:** Explain the origin, scope, environment name, and earliest usable syntax for a selected set of variables; successfully pass one non-secret output across stages.
- **Estimate:** 2–2.5 hours.
- **Depends on:** Modules 01–02.

### Module 04 — Stages, jobs, steps, and dependencies

- **Objectives**
  - Build a multi-stage pipeline with explicit stage/job/step structure.
  - Model linear dependencies and fan-out/fan-in with `dependsOn`.
  - Select a step type and set appropriate step-level properties.
  - Explain where ordinary agent jobs, server jobs, and deployment jobs fit.
- **Concepts and deep dive:** Stages as major workflow boundaries; jobs as scheduling/isolation units; steps as sequential operations in a job. Cover agent jobs, agentless/server jobs, and an introduction to `deployment` jobs. Cover `script`, `bash`, `pwsh`, `powershell`, `task`, `checkout`, `download`, `publish`, and template steps. Explain `displayName`, `name`, `condition`, `continueOnError`, `enabled`, `env`, `retryCountOnTaskFailure`, and `workingDirectory`, including which properties apply to which step/task types.
- **Walkthrough files:** `modules/04-stages-jobs-steps-and-dependencies/README.md`, `azure-pipelines.yml`, `fan-out-fan-in.yml`.
- **Labs:** Split build and test into stages, fan out platform-independent jobs, then fan in to a summary job. Break a dependency or set a step `enabled: false`; observe skipped jobs/steps and dependency results.
- **Challenge:** Add a lint job that runs in parallel with tests and a final job that needs both. Answer: `solutions/04-dependencies-solution.yml`.
- **Common errors and fixes:** A job that names `dependsOn` must reference a valid job/stage at the correct level. A job with no matching pool may inherit an unintended pool; make pool selection explicit in the lab.
- **Checkpoint:** Draw the dependency graph and predict which jobs can run concurrently and which are skipped after one dependency fails.
- **Estimate:** 2 hours.
- **Depends on:** Modules 01–03.

### Module 05 — Runtime parameters, templates, and reuse

- **Objectives**
  - Distinguish immutable-at-queue-time runtime parameters from runtime variables.
  - Create and call step, job, stage, and variable templates.
  - Use typed template parameters with defaults and `${{ if }}` / `${{ each }}`.
  - Reuse a template from the shared-templates repository at a fixed ref.
- **Concepts and deep dive:** Template expansion, parameter types and validation, `extends`, template inclusion, required parameters, defaults, repository resources, and why a template is expanded before runtime output variables exist. Explain that templates shape the plan; they are not runtime scripts. Runtime expressions can access more runtime variables than template expressions, but cannot access runtime parameters.
- **Walkthrough files:** `modules/05-parameters-templates-and-reuse/README.md`, `azure-pipelines.yml`, `templates/steps/build.yml`, `templates/jobs/test.yml`; companion repo `azure-pipelines-shared-templates`.
- **Labs:** Toggle a step via a boolean parameter and iterate a matrix of named steps via `${{ each }}`. Break the template path/ref or pass a wrong parameter type; observe template-resolution/validation failure before agent execution.
- **Challenge:** Refactor duplicated build steps into a template and add an `extends` guardrail. Answer: `solutions/05-templates-solution.yml`.
- **Common errors and fixes:** `File ... not found in repository ...` indicates a path/ref/repository alias mismatch; template parameters are compile-time values, so do not expect a value generated by a task to control `${{ if }}`.
- **Checkpoint:** Explain which parts of a template are expanded before the run and demonstrate the same template used by two pipelines with different parameter values.
- **Estimate:** 2–2.5 hours.
- **Depends on:** Modules 02 and 04; companion repositories configured for cross-repo exercise.

### Module 06 — Pools, workspaces, containers, and job strategies

- **Objectives**
  - Select a Microsoft-hosted image or a self-hosted pool and explain the trade-offs.
  - Locate source, artifacts, binaries, temporary files, and test results in the agent workspace.
  - Use a container job and distinguish it from a deployment environment.
  - Configure `matrix`, `parallel`, and `maxParallel` and interpret their run graph.
- **Concepts and deep dive:** Agent lifecycle and per-job isolation; `$(Pipeline.Workspace)` / `Agent.BuildDirectory` and common `s/`, `a/`, `b/`, and `TestResults/` locations; `workspace: clean` options and their effect. Explain image drift and pinning/maintenance considerations, container-job requirements, and how matrix variables become job inputs. A local self-hosted agent is optional and must be isolated from personal credentials.
- **Walkthrough files:** `modules/06-agents-workspaces-and-job-strategies/README.md`, `azure-pipelines.yml`, `container-job.yml`, `matrix.yml`.
- **Labs:** Print known workspace variables and list their contents before/after checkout and artifact staging. Break the image/container name or exceed available parallel capacity; observe job queueing or agent/container setup failure. Compare `matrix` expansion to `parallel`.
- **Challenge:** Run tests on two SDK/runtime values with a bounded `maxParallel`. Answer: `solutions/06-strategies-solution.yml`.
- **Common errors and fixes:** A requested image or container may be unavailable/incompatible; use a supported image and inspect the job initialization log. A job waiting in queue can mean no matching agent or no free parallel job, not a YAML syntax failure.
- **Checkpoint:** Sketch the workspace layout for the run, identify which values persist across jobs only when published, and predict matrix job count.
- **Estimate:** 2 hours.
- **Depends on:** Modules 03–04.

### Module 07 — Conditions, timeouts, and cancellation

- **Objectives**
  - Write job/stage/step `condition` expressions with `succeeded()`, `failed()`, `always()`, `eq`, `ne`, `and`, `or`, and `contains`.
  - Set job `timeoutInMinutes` and `cancelTimeoutInMinutes` and step `timeoutInMinutes`.
  - Describe default timeout/cancel behavior and how cancellation affects cleanup steps.
  - Diagnose a deliberately timed-out job without confusing it with a failed assertion.
- **Concepts and deep dive:** Default success conditions, dependency results, condition replacement of the default condition, cancellation propagation, step/job timeout boundaries, and cleanup windows. `always()` does not guarantee that an agent remains available after cancellation or timeout. Compare Microsoft-hosted/self-hosted agent and private/public project entitlements.
- **Walkthrough files:** `modules/07-conditions-timeouts-and-cancellation/README.md`, `azure-pipelines.yml`, `timeout-lab.yml`.
- **Labs:** Run a short sleep with a tiny timeout; compare `succeeded()`, `failed()`, and `always()` cleanup steps. Cancel a run manually and inspect which steps continue. Keep timeout experiments short and clearly labeled to avoid consuming hosted minutes.
- **Challenge:** Add a cleanup step that runs after a failed test but does not mask the original failure. Answer: `solutions/07-timeouts-solution.yml`.
- **Common errors and fixes:** A custom condition such as `eq(...)` can accidentally remove the implicit success requirement; combine it with `succeeded()` when that is intended. A task killed at its timeout is not evidence that its script's own timeout handling worked.
- **Checkpoint:** Given success/failure/cancel/timeout outcomes, predict which steps run and identify the timeout setting that limits each operation.
- **Estimate:** 1.5–2 hours.
- **Depends on:** Modules 02 and 04.

> **Timeouts—verify before coding:** Plan to test job-level `timeoutInMinutes`, `cancelTimeoutInMinutes`, and step-level `timeoutInMinutes`. For Azure DevOps Services, the current planning baseline is a 60-minute job default; Microsoft-hosted jobs are limited to 360 minutes for public projects with public repositories and 60 minutes when the project or repository is private unless additional capacity is paid for. Self-hosted agent jobs can run indefinitely; server jobs default to 60 minutes and have a 30-day maximum. The cancellation grace period is commonly documented as 5 minutes. These are verification targets, not promises: exact defaults and caps depend on job type, agent type, service/server, project visibility, and parallel-job grant. Check **Job timeouts** and **Configure and pay for parallel jobs** immediately before implementation; record exact values, applicability, and source in `docs/reference/timeouts.md`. Do not promise that a long-running test is allowed merely because the YAML accepts its value.

### Module 08 — Checkout, source paths, and workspace behavior

- **Objectives**
  - Configure `checkout: self` and `checkout: none` and explain when an implicit checkout occurs.
  - Use `path`, `fetchDepth`, `fetchTags`, `clean`, `persistCredentials`, `submodules`, and `lfs` deliberately.
  - Predict source-directory paths for one versus multiple repositories.
  - Compare `Build.SourcesDirectory`, `Build.Repository.LocalPath`, and `System.DefaultWorkingDirectory`.
- **Concepts and deep dive:** Default checkout under `$(Pipeline.Workspace)/s`, checkout path behavior, repository-specific path overrides, Git fetch depth/tags, credential persistence risks, submodules/LFS prerequisites, and the relationship between checkout, working directory, and task path. Include a table for one checkout, multiple default checkouts, and custom self checkout; verify each cell against the current multi-repo documentation and observed run.
- **Walkthrough files:** `modules/08-checkout-and-workspace-paths/README.md`, `azure-pipelines.yml`, `paths.yml`.
- **Labs:** Print checkout-related variables and list directories for self-only, no checkout, and two-repository runs. Break a script path by assuming the old `s/` layout after adding another repo; observe “file not found,” then use explicit checkout `path` and inspect the variables.
- **Challenge:** Check out self into a stable custom path and make scripts work before/after adding a second repository. Answer: `solutions/08-checkout-solution.yml`.
- **Common errors and fixes:** A script file not found after adding another checkout is often a path assumption; inspect actual directories and `Build.Repository.LocalPath`. An unexpected shallow-history result can come from `fetchDepth`, not missing commits on the server.
- **Checkpoint:** Given a checkout configuration, state the expected path of each repo and explain which path variable is stable or changes in that case.
- **Estimate:** 1.5–2 hours.
- **Depends on:** Modules 03 and 06.

### Module 09 — Cross-repository checkout and authorization

- **Objectives**
  - Declare Azure Repos and GitHub repositories under `resources.repositories`.
  - Check out multiple repos with correct `name`, `ref`, `endpoint`, and aliases.
  - Configure a repository-resource trigger where supported and distinguish it from the primary repository trigger.
  - Diagnose permission failures using authorization scope and build-service identity settings.
- **Concepts and deep dive:** Same-project versus cross-project Azure Repos, GitHub service connections, resource refs, multiple `checkout` steps, repository triggers, project-scoped versus collection-scoped job authorization, “Protect access to repositories in YAML pipelines,” explicit pipeline resource permissions, and least privilege. Explain likely causes of `TF401019`: missing repo, wrong project/name/ref, or an identity lacking access.
- **Walkthrough files:** `modules/09-cross-repository-checkout-and-permissions/README.md`, `azure-pipelines.yml`, `resources.yml`, companion `azure-pipelines-shared-templates`.
- **Labs:** Check out the shared-templates repo with a declared resource and pinned branch/tag. Break its permissions by withholding read access; observe the checkout authorization error, then grant the project build service identity the minimum required repository permission. Test a resource trigger only after confirming service support and settings.
- **Challenge:** Replace a broad collection-level permission with a project/repository-scoped grant and document the trade-off. Answer: `solutions/09-cross-repo-solution.yml`.
- **Common errors and fixes:** `TF401019: The Git repository with name or identifier ... does not exist or you do not have permissions for the operation you are attempting.` Check the exact project/repo, pipeline authorization, and build service identity read permission. “Repository not found” from GitHub may instead mean a missing/incorrect service connection or endpoint authorization.
- **Checkpoint:** Trace the identity used to access each repository and diagnose a controlled `TF401019` from logs and project settings.
- **Estimate:** 2–3 hours.
- **Depends on:** Modules 05 and 08.

### Module 10 — Pipeline artifacts within a pipeline

- **Objectives**
  - Publish and download a named artifact with `PublishPipelineArtifact@1` / `DownloadPipelineArtifact@2`.
  - Use `publish:` and `download:` shortcuts when appropriate.
  - Pass outputs between jobs/stages and find their download paths.
  - Control automatic artifact downloads in deployment jobs.
- **Concepts and deep dive:** Artifact staging versus pipeline workspace, artifact names/paths, same-run job/stage transfer, retention association, and automatic artifact download in the `deploy` lifecycle hook of deployment jobs; use `download: none` to disable it. Explain why an agent workspace is not a cross-job transport mechanism.
- **Walkthrough files:** `modules/10-pipeline-artifacts/README.md`, `azure-pipelines.yml`, `publish-download.yml`.
- **Labs:** Publish the console output from one job and consume it in another job and a later stage. Break it with a mismatched artifact name; observe the download error/no files and correct it. Compare explicit task syntax with shortcut syntax.
- **Challenge:** Publish test results and app output as separate artifacts and download only the app output in a deployment job. Answer: `solutions/10-artifacts-solution.yml`.
- **Common errors and fixes:** A missing artifact commonly indicates a name mismatch, an earlier publish step that did not run, or a wrong pipeline/run selection; inspect the publish log and the exact download path.
- **Checkpoint:** Locate the published artifact in the run UI and on the consumer agent, and explain why a file written only to one job's workspace is unavailable to another.
- **Estimate:** 1.5–2 hours.
- **Depends on:** Modules 04 and 06.

### Module 11 — Pipeline resources and artifact choices

- **Objectives**
  - Declare a producer pipeline under `resources.pipelines`.
  - Select a source pipeline, branch, and version and understand resource-trigger behavior.
  - Download by resource alias under `$(Pipeline.Workspace)`.
  - Choose between Pipeline Artifacts, legacy Build Artifacts, and Azure Artifacts feeds.
- **Concepts and deep dive:** Pipeline resources (`source`, optional `branch`/`version`, alias, trigger), resource variables, `download: <alias>`, same-organization project permissions, and provenance. Compare Pipeline Artifacts with legacy `PublishBuildArtifacts@1` / `DownloadBuildArtifacts@1` and package feeds: intended use, retention, portability, and size/performance considerations. Explain `.artifactignore` behavior, including implementation-time verification of edge cases and defaults.
- **Walkthrough files:** `modules/11-pipeline-resources-and-artifact-comparison/README.md`, `azure-pipelines.yml`, `consumer.yml`, `artifactignore-example`.
- **Labs:** Have a consumer pipeline download the latest successful artifact from the tutorial producer; then pin a branch/version and compare results. Break the resource alias or deny pipeline access; observe the resource/download failure and correct it.
- **Challenge:** Recommend the right storage mechanism for a same-run build output, a versioned NuGet package, and a legacy pipeline needing Build Artifacts. Answer: `solutions/11-resources-solution.md`.
- **Common errors and fixes:** A resource alias mismatch causes an invalid `download` reference; a missing producer artifact requires checking its publish step and selected run. Cross-project access may require authorizing the consumer pipeline on the producer.
- **Checkpoint:** Explain exactly which producer run was selected and justify artifact/feed choice for three scenarios.
- **Estimate:** 2 hours.
- **Depends on:** Module 10.

### Module 12 — Consuming Azure Artifacts feeds

- **Objectives**
  - Distinguish project-scoped and organization-scoped feeds and understand views/upstream sources.
  - Restore a package from a feed using a supported authentication task.
  - Grant the pipeline build identity the minimum Feed Reader access needed.
  - Diagnose authentication, URL, and upstream-source failures.
- **Concepts and deep dive:** Feed scope, package views such as `@local`, `@prerelease`, and `@release`, upstream sources, NuGet authentication and feed URLs, and package-manager-specific authentication. Cover current task majors to verify: `NuGetAuthenticate@1`, `npmAuthenticate@0`, `PipAuthenticate@1`, `MavenAuthenticate@0`, and Universal Packages via `UniversalPackages@0` and/or `az artifacts universal`. Explain service connections only where required; do not store credentials in repository files.
- **Walkthrough files:** `modules/12-consuming-azure-artifacts-feeds/README.md`, `azure-pipelines.yml`, package-manager config examples without credentials.
- **Labs:** Restore the published sample NuGet package using `NuGetAuthenticate@1`; optionally compare npm/Python/Maven/Universal Packages authentication using a small package-specific example or annotated variant. Break the feed URL or withhold Feed Reader permission; observe a 401/403 or package-not-found/upstream error and identify the correct layer to fix.
- **Challenge:** Add a package restore that can use an upstream source while keeping private-feed credentials out of source control. Answer: `solutions/12-feed-consumer-solution.yml`.
- **Common errors and fixes:** HTTP `401` usually means missing/incorrect authentication; `403` generally indicates the identity authenticated but lacks permission. A wrong feed URL/project scope can look like a missing package. An upstream source that is disabled or not configured cannot supply the package.
- **Checkpoint:** Restore a private package in a clean job and name the feed identity, role, endpoint, and authentication task involved.
- **Estimate:** 2–3 hours.
- **Depends on:** Modules 03, 09, and 11; feed and package permissions configured.

### Module 13 — Publishing packages and cross-project access

- **Objectives**
  - Pack and publish the sample library as a versioned NuGet package.
  - Grant the producer pipeline Feed Publisher/Contributor access and the consumer Feed Reader access.
  - Consume a published version from the separate consumer repository.
  - Explain why package feeds and pipeline artifacts have different lifecycles and permissions.
- **Concepts and deep dive:** Versioning, package identity, publish task/CLI choice, feed views/promotion, project build service identity access to a cross-project feed, and producer/consumer separation. Keep feed ACLs and package source configuration explicit and secret-free.
- **Walkthrough files:** `modules/13-publishing-packages-and-cross-project-access/README.md`, `publish-package.yml`, consumer pipeline in `azure-pipelines-feed-consumer`.
- **Labs:** Publish a prerelease package to `@local`, promote or select an appropriate view, then restore it from the other repo. Break the version or remove Contributor/Reader permission; observe duplicate/version-resolution or 401/403 behavior and correct the relevant configuration.
- **Challenge:** Publish a new package version without mutating the prior version and make the consumer select it reproducibly. Answer: `solutions/13-publish-feed-solution.yml`.
- **Common errors and fixes:** 401/403 errors require checking which project-scoped build service identity runs the job and its feed role. “Package not found” requires checking package name/version/view, feed URL, and whether publish succeeded.
- **Checkpoint:** Follow one package from source commit through feed publication to a clean consumer restore, identifying both pipeline identities and their least-privilege roles.
- **Estimate:** 2–3 hours.
- **Depends on:** Module 12 and the sample app.

### Module 14 — Deployment jobs, environments, strategies, and approvals

- **Objectives**
  - Define a deployment job targeting an Azure DevOps `environment`.
  - Compare `runOnce`, `rolling`, and `canary` deployment strategies and their supported target/resource contexts.
  - Configure an environment check/approval outside YAML and observe its effect on a run.
  - Control deployment artifact download behavior.
- **Concepts and deep dive:** Deployment job lifecycle, environment history/resources, strategy hooks, deployment-specific variables (`Deployment.*` and `Strategy.*` where available), server jobs with `ManualValidation@1` versus environment approvals/checks, approver permission boundaries, and `download: none`. Verify the manual-validation task version and supported job context when implementing. Clarify that sensitive approval/check configuration is managed by authorized project/environment administrators, not made safe by putting a condition in YAML.
- **Walkthrough files:** `modules/14-deployment-jobs-environments-and-approvals/README.md`, `azure-pipelines.yml`, `strategies.yml`.
- **Labs:** Deploy a harmless console output to a test environment using `runOnce`; add a manual approval check in project settings and observe the pending stage. Break the target environment name or use a normal job where a deployment job is intended; inspect the validation/target error and fix it.
- **Challenge:** Add a canary-shaped YAML example and document which environment/resource setup it requires; do not deploy to production. Answer: `solutions/14-deployment-solution.yml`.
- **Common errors and fixes:** An environment not found may need to be created or explicitly authorized. A stage waiting for approval is gated, not necessarily stuck; inspect checks/approver configuration. Strategy syntax and supported target types vary; validate against the current deployment-job docs.
- **Checkpoint:** Explain who approves, what resource is protected, which artifact is downloaded, and which strategy hook ran.
- **Estimate:** 2–3 hours.
- **Depends on:** Modules 04, 10, and 13.

### Module 15 — Debugging and pipeline security

- **Objectives**
  - Enable `System.Debug` for a targeted run and read task/agent logs systematically.
  - Diagnose whether a failure occurred in parsing, template expansion, scheduling, checkout, task execution, or artifact/feed access.
  - Protect secrets, access tokens, service connections, and build identities with least privilege.
  - Produce a useful troubleshooting report without exposing secret values.
- **Concepts and deep dive:** Azure DevOps “Download logs,” debug logging, expression/expanded-plan diagnosis, `System.AccessToken` explicit mapping via `env:`, secret variable groups, masking limitations, endpoint authorization, repository protection settings, and avoiding broad permissions. Discuss that debug logs can reveal more operational detail and should be handled accordingly.
- **Walkthrough files:** `modules/15-debugging-and-pipeline-security/README.md`, `azure-pipelines.yml`, `diagnostic-checklist.md`.
- **Labs:** Diagnose three seeded failures (compile-time expression, repository authorization, package-feed permission) using logs and settings. Break a secret mapping intentionally with a dummy value only; observe the missing environment variable without putting a real credential in YAML.
- **Challenge:** Write a redacted incident note with the failing pipeline phase, evidence, root cause, fix, and least-privilege permission change. Answer: `solutions/15-debugging-solution.md`.
- **Common errors and fixes:** A masked value in logs does not prove a secret is safe to print; never echo secrets. `System.AccessToken` must be explicitly made available to the task when needed and repository/job authorization must still allow the requested operation.
- **Checkpoint:** Diagnose a seeded run from evidence alone and show the fix without exposing a credential or granting unnecessary access.
- **Estimate:** 1.5–2 hours.
- **Depends on:** Modules 03, 09, 12, and 14.

## Capstone project

Build a realistic but harmless pipeline across the tutorial and companion repositories:

1. **Build and test:** Compile `Tutorial.Library` and `Tutorial.Console`; run tests in a stage with clearly named jobs and a test matrix where useful.
2. **Package:** Create a versioned NuGet package and publish it to the project- or organization-scoped tutorial feed using the configured producer identity.
3. **Pipeline artifact:** Publish the console output and test results using Pipeline Artifacts.
4. **Cross-pipeline/repository consumption:** Trigger or manually run the separate consumer pipeline; use a pipeline resource to fetch the selected producer artifact and authenticate to the feed to restore the package.
5. **Deploy:** Use a deployment job targeting a non-production environment; require an environment approval/check before the deployment proceeds. Demonstrate `runOnce`; document how rolling/canary would differ without making an unnecessary production deployment.
6. **Reuse and observability:** Use a template from `azure-pipelines-shared-templates`, output a non-secret value across a stage boundary, and include a short debug/troubleshooting record.

### Acceptance criteria

- A clean run shows the intended stage/job dependency graph, and independent build/test jobs can run concurrently when capacity permits.
- Tests run against the expected source revision; failures prevent package promotion/deployment unless a deliberately documented condition allows it.
- The package appears in the intended feed/view, and the consumer restores the intended version from a separate repo.
- The consumer downloads the artifact from the intended producer pipeline run and can locate it under the documented workspace path.
- The deployment job targets the configured environment and pauses for its approval/check; no production resource is required.
- Repository, feed, environment, and service-connection permissions are narrowly scoped to the pipeline identities that need them.
- No secret appears in the repository, artifact, or log; all task/environment mappings and output-variable references are documented.
- The README/runbook records setup, troubleshooting evidence, and the source pages/date used to verify volatile Azure DevOps limits and task versions.

## Reference appendices

Create these as `docs/reference/*.md` after the relevant modules:

1. **Variable cheat sheet (`variables.md`)** — category, example, scope, environment mapping, and availability table. Mark typical availability as follows, then verify each selected variable in the current documentation and a live run:

   | Value category | `${{ }}` template expression | `$(...)` macro | `$[ ]` runtime expression |
   |---|---|---|---|
   | Parameters | Yes, during template expansion | Not a variable macro | No; runtime expressions cannot access parameters |
   | Statically declared variables | Generally available after their declaration is processed | Available before a task runs | Available at runtime |
   | Predefined runtime/build variables | Many are not available during template expansion; check each variable’s documented availability | Commonly available at task runtime | Available where runtime expressions are supported |
   | Output variables | Not available to shape already-expanded templates | Available only after correct dependency mapping and producer completion | Commonly used in dependency/condition mappings |
   | Secrets and `System.AccessToken` | Do not treat as template-time values | Secret macros are masked, but secrets should be explicitly mapped to a task `env:`; token availability is opt-in/configuration-dependent | Do not use to bypass scope or secret rules |

   Include the Module 03 variable list and clearly label unsupported/variable-specific cases rather than implying every variable is available in every syntax.
2. **Expression syntax (`expressions.md`)** — `${{ }}` versus `$(var)` versus `$[ ]`, compile-time/runtime decision guide, common condition functions, and examples of `${{ if }}` / `${{ each }}`.
3. **Checkout paths (`checkout-paths.md`)** — single checkout, multiple default checkouts, custom `path`, and the observed values of `Pipeline.Workspace`, `Build.SourcesDirectory`, `Build.Repository.LocalPath`, and `System.DefaultWorkingDirectory`. Verify all rows against the current **Multi-repo checkout** page and a live run.
4. **Artifact comparison (`artifacts.md`)** — Pipeline Artifacts, legacy Build Artifacts, and Azure Artifacts feeds by use case, retention, access boundary, transfer behavior, and approximate size/performance considerations.
5. **Timeouts (`timeouts.md`)** — job/step/cancel settings and defaults/limits for Microsoft-hosted versus self-hosted and public versus private projects. Include the exact checked values, service context, verification date, and links to the current **Job timeouts** and **Configure and pay for parallel jobs** pages.
6. **Permissions and troubleshooting (`permissions-and-troubleshooting.md`)** — repository job-authorization settings, build service identity naming/scope, feed roles, service connections, environment approvals, and diagnostic paths for `TF401019`, repository-not-found, 401, and 403.

## Azure DevOps setup checklist

Complete in this order; record organization/project names in a private local note, not in committed sample YAML.

1. Create or select an Azure DevOps organization and a tutorial project. Confirm project visibility and current parallel-job grant/runtime limits.
2. Create the tutorial repo and the two companion repos: shared templates and feed consumer. Add the learner and build identities with least-privilege project/repository access.
3. Decide Azure Repos-only or add GitHub. If GitHub is used, create a narrowly scoped service connection and authorize only the required pipeline.
4. Create an Azure Artifacts feed. Choose project-scoped or organization-scoped based on the lab goal; configure upstream sources and views only where needed.
5. Grant the tutorial producer build service identity the required package-publishing role. Grant the consumer identity Feed Reader on the feed, including cross-project access if applicable.
6. Configure repository authorization: review **Limit job authorization scope to current project** and **Protect access to repositories in YAML pipelines**; explicitly authorize each pipeline to access the companion repository/resource.
7. Create a variable group only for non-public settings needed by examples. Add any real secrets through the Azure DevOps UI, restrict group/pipeline authorization, and never commit secret values.
8. Create a non-production Azure DevOps environment. Add an approval/check with a separate approver identity; verify learners cannot self-approve if the exercise is intended to teach separation of duties.
9. Select the hosted image or create an isolated self-hosted agent only if needed. Confirm tool availability and concurrency; do not put personal credentials on a shared lab agent.
10. Create pipelines from the module YAML paths, grant access explicitly, and run the first-run smoke test before progressing.
11. Verify task major versions, hosted-image support, timeout defaults/limits, repository triggers, and artifact behavior against current Microsoft Learn pages before running the corresponding modules.

## Implementation roadmap

Later implementation sessions should follow this sequence, keeping each change focused and runnable:

1. Create the root README, setup guide, sample .NET library/console/tests, and Module 00 first-run pipeline.
2. Add Module 01 YAML anatomy and Module 02 expressions, with concise broken examples and solution files.
3. Add Module 03 variable dumps/output-variable labs and the variable/expression appendices.
4. Add Module 04 dependency graph, then Module 05 templates/parameters and the shared-templates companion repo contents.
5. Add Module 06 pools/workspaces/strategies and Module 07 conditions/timeouts; verify timeouts and concurrency entitlements before publishing examples.
6. Add Module 08 checkout paths and Module 09 cross-repo permissions; capture actual path observations and authorization settings.
7. Add Module 10 same-run artifacts and Module 11 pipeline resources/comparison; test artifact names, paths, and selected producer runs.
8. Create the feed and implement Module 12 consume, then Module 13 publish/consume with the separate feed-consumer repo.
9. Add Module 14 environment deployment/approval and Module 15 debugging/security with dummy-only secret experiments.
10. Assemble the capstone, finish all reference appendices, check every documentation claim/task version against current Learn pages, and run the complete tutorial from setup through consumer deployment.

At each step, keep examples minimal, label intentionally failing pipelines, avoid credentials in source, and record Azure DevOps settings that cannot be represented in YAML.

## Summary and open questions

The planned sequence has **16 modules**: first run; YAML anatomy; expressions; variables; stages/jobs/steps; templates and parameters; agents/workspaces/strategies; conditions/timeouts; checkout paths; cross-repo checkout/permissions; Pipeline Artifacts; pipeline resources/artifact choices; feed consumption; package publishing; deployment jobs/approvals; and debugging/security.

### Assumptions to confirm before implementation

- The learner can create or administer an Azure DevOps organization/project, feeds, environments, and repositories—or can ask an administrator to complete the setup checklist.
- The sample will use a small .NET/NuGet library and console app unless another ecosystem is preferred.
- Azure Repos is the primary repository host; GitHub checkout is an optional service-connection extension.
- Labs target Azure DevOps Services, not an on-premises Azure DevOps Server, unless a Server compatibility track is requested.
- The feed and deployment labs use a non-production project feed and environment; no cloud application deployment or paid external service is required.
- Free-tier parallel-job availability, public/private timeout limits, and any organization-specific grants will be checked in the learner’s organization rather than assumed.

### Open questions

1. Is the learner using Azure DevOps Services, or must the plan also account for a specific Azure DevOps Server version?
2. Should the primary examples remain .NET/NuGet, or would Node/npm, Python/PyPI, or Maven better match the learner’s goals?
3. Will the learner have administrator access to configure feeds, repository permissions, and environment approvals?
4. Is GitHub cross-repository checkout required, or is Azure Repos sufficient for the core tutorial?

## Microsoft Learn documentation to verify

Use the current Azure DevOps view of these pages when implementing and record the verification date in the affected module:

- [Azure Pipelines YAML schema](https://learn.microsoft.com/azure/devops/pipelines/yaml-schema/?view=azure-pipelines)
- [Expressions](https://learn.microsoft.com/azure/devops/pipelines/process/expressions?view=azure-devops)
- [Variables](https://learn.microsoft.com/azure/devops/pipelines/process/variables?view=azure-devops)
- [Pipeline runs and processing order](https://learn.microsoft.com/azure/devops/pipelines/process/runs?view=azure-devops)
- [Stages](https://learn.microsoft.com/azure/devops/pipelines/process/stages?view=azure-devops)
- [Jobs and phases](https://learn.microsoft.com/azure/devops/pipelines/process/phases?view=azure-devops)
- [Deployment jobs](https://learn.microsoft.com/azure/devops/pipelines/process/deployment-jobs?view=azure-devops)
- [Multi-repo checkout](https://learn.microsoft.com/azure/devops/pipelines/repos/multi-repo-checkout?view=azure-devops)
- [Job timeouts](https://learn.microsoft.com/azure/devops/pipelines/process/phases?view=azure-devops#timeouts)
- [Configure and pay for parallel jobs](https://learn.microsoft.com/azure/devops/pipelines/licensing/concurrent-jobs?view=azure-devops)
- [Pipeline artifacts](https://learn.microsoft.com/azure/devops/pipelines/artifacts/pipeline-artifacts?view=azure-devops)
- [Pipeline resources](https://learn.microsoft.com/azure/devops/pipelines/process/resources?view=azure-devops)
- [Azure Artifacts documentation](https://learn.microsoft.com/azure/devops/artifacts/?view=azure-devops)
- [Secure access to repositories](https://learn.microsoft.com/azure/devops/pipelines/security/secure-access-to-repos?view=azure-devops)
- [Approvals and checks](https://learn.microsoft.com/azure/devops/pipelines/process/approvals?view=azure-devops)
