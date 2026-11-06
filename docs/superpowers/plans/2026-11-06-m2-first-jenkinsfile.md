# M2 First Jenkinsfile Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add and run a minimal SCM-backed Declarative Jenkins Pipeline with explicit stages and post-build reporting.

**Architecture:** A root `Jenkinsfile` holds all Pipeline behavior. Jenkins will later load it from the `main` branch of the existing repository; the pipeline runs on the local Linux controller and executes only harmless workspace checks and echo commands.

**Tech Stack:** Jenkins Declarative Pipeline, Groovy-based Jenkinsfile syntax, GitHub SCM, Docker-hosted local Jenkins controller.

**Spec:** `docs/superpowers/specs/2026-11-06-m2-first-jenkinsfile-design.md`

## Global Constraints

- Use Declarative Pipeline syntax and `agent any`.
- Use explicit `Checkout`, `Validate`, and `Test` stages.
- Use `skipDefaultCheckout(true)` and `checkout scm` exactly once.
- Use Linux `sh` steps only; do not introduce applications, Docker builds, credentials, registries, cloud access, or webhooks.
- Keep success, failure, and cleanup reporting in `post` blocks.
- If the GitHub repository is private, stop at the SCM access blocker; do not add credentials without explicit approval.

## Review Focus

- The implicit default checkout must be disabled so the Checkout stage does not clone twice.
- The Validate stage must prove the workspace contains `Jenkinsfile` after SCM checkout.
- The Test stage must succeed without application code or dependencies.
- All post blocks must be syntactically valid and avoid secrets.
- The Jenkins job must use the `main` branch and repository-root `Jenkinsfile` when configured from SCM.

---

### Task 1: Add the repository Pipeline definition

**Files:**
- Create: `Jenkinsfile`
- Modify: `docs/progress.md`
- Modify: `docs/learning-path.md`

**Interfaces:**
- Consumes: Jenkins Declarative Pipeline and `checkout scm` supplied by the later SCM-configured Jenkins job.
- Produces: a root-level Pipeline definition consumed by the Jenkins Pipeline-from-SCM job.

- [ ] **Step 1: Add the desired Pipeline validation checklist**

The pipeline must contain `agent any`, `skipDefaultCheckout(true)`, stages named `Checkout`, `Validate`, and `Test`, `checkout scm`, `sh 'test -f Jenkinsfile'`, a successful placeholder test echo, and `post` sections for success, failure, and cleanup.

- [ ] **Step 2: Confirm the checklist fails before a `Jenkinsfile` exists**

Run: `Test-Path Jenkinsfile`

Expected: `False`.

- [ ] **Step 3: Create the root-level Declarative `Jenkinsfile`**

Use the exact stages and constraints from Step 1. The Test stage prints `M2 placeholder test passed`; no external command or secrets are allowed.

- [ ] **Step 4: Confirm the Pipeline definition has the required structure**

Run PowerShell checks that locate the required Pipeline directives and stage names in `Jenkinsfile`.

Expected: every required directive is present exactly once where applicable.

- [ ] **Step 5: Commit the repository Pipeline definition**

```powershell
git add Jenkinsfile docs/progress.md docs/learning-path.md
git commit -m "ci: add first declarative pipeline"
```

### Task 2: Run the Pipeline from SCM in local Jenkins

**Files:**
- No local file changes before user-run verification.
- Modify after verification: `docs/progress.md`, `docs/learning-path.md`

**Interfaces:**
- Consumes: committed root-level `Jenkinsfile` from Task 1 and local `jenkins-lab` controller.
- Produces: stage-level Jenkins logs demonstrating checkout, validation, test, and post-build behavior.

- [ ] **Step 1: Configure a Jenkins Pipeline job from SCM**

In Jenkins, create a Pipeline job named `m2-first-pipeline`; select Pipeline script from SCM, Git, the existing repository URL, branch `*/main`, and script path `Jenkinsfile`. Do not configure a webhook or credentials. If SCM access is denied because the repository is private, stop and record the access blocker.

- [ ] **Step 2: Run and inspect the successful Pipeline**

Use Build Now. Verify the three named stages are green and the console shows the placeholder test message plus success and cleanup post messages.

- [ ] **Step 3: Run the intentional failure drill**

On a temporary practice branch, change only the Test stage command to `exit 1`, commit it, configure or run the job against that branch, and verify the failure post message and failed stage. Then return the job to `main`; do not merge the intentional-failure commit into `main`.

- [ ] **Step 4: Record observed results and next checkpoint**

Record the actual build URLs or build numbers, branch used, stage results, failure diagnosis, and the next M3 action in the learning tracker.
