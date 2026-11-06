# Jenkins → Huawei Cloud CI/CD — Learning Path & Codex Project Guide

> **Status:** Not started
> **Created:** 2026-11-05
> **Primary goal:** Learn Jenkins by building a reproducible, secure CI/CD workflow for a Python FastAPI service, then progress to Docker image promotion through **VSIT → SIT → UAT → PROD** using Huawei Cloud SWR and CCE where access is available.
> **Learning environment:** Windows, VS Code, Git/GitHub, Docker Desktop (WSL2), Python; Codex as a coding tutor and pair programmer.
> **Scope:** Personal sandbox and simulated enterprise environments only. **Do not connect to customer, employer, or production infrastructure without explicit authorization.**

---

## 1. Why this project exists

A cloud deployment discussion raised the question: *How does Jenkins move or promote container images from VSIT to SIT, UAT, and production on Huawei Cloud?*

The learning outcome is to explain and demonstrate this flow:

```text
Developer -> Git repository -> Jenkins (test/build) -> container registry
                                                 |
                                                 v
                          VSIT -> SIT -> UAT -> PROD
                           |      |      |      |
                          gates, tests, approvals, release evidence

Huawei example: SWR (image registry) + CCE (Kubernetes runtime)
```

**Important distinctions**
- Jenkins orchestrates automated CI/CD workflows; it is not a Huawei Cloud service.
- Docker images are built once and ideally promoted by **immutable digest**; environment-specific configuration is supplied at deployment time.
- A `kubectl` rollout changes the workload in an environment; the application image does **not** necessarily get copied from one environment to another. Depending on architecture, each environment can pull from shared SWR or images can be replicated between isolated registries.
- **VSIT** is project-specific terminology; verify its exact meaning with the delivery team. For practice, interpret it as *vendor-side integration/testing*.
- Passing a technical deployment does not equal business/UAT approval.

## 2. End-state capabilities

After completing the roadmap, I should be able to:

1. Explain CI, continuous delivery versus continuous deployment, Jenkins controllers/agents, jobs, pipelines, and webhooks.
2. Start and maintain a local Jenkins instance, with persistent data and secured admin access.
3. Create a version-controlled Declarative `Jenkinsfile` and diagnose pipeline failures from logs.
4. Automatically lint/test a small FastAPI application on commits or pull requests.
5. Build, tag, scan (when tooling is available), and publish a Docker image to a practice registry.
6. Deploy an exact image digest across isolated *simulated* VSIT/SIT/UAT/PROD environments.
7. Gate promotions with automated tests and manual approval; demonstrate rollback.
8. Explain Huawei Cloud IAM, SWR, CCE, VPC, LTS, credentials and access dependencies at a conceptual level.
9. Push a practice image to **Huawei Cloud SWR** and deploy to **CCE** **only when an authorized sandbox account is available**; otherwise document the equivalent commands and keep using local Kubernetes.
10. Present a release runbook and deployment diagram to a technical team.

## 3. Ground rules for Codex

> **Paste this section into `AGENTS.md` in the repository** or instruct Codex to read this markdown file before each session.

### Role and approach
- Act as an **instructor + pair programmer**, not an autopilot that completes the full roadmap in one session.
- Start each session by reading the **Progress Tracker** and the **Current Milestone**.
- Teach concepts briefly, then propose the **smallest working exercise**. Explain *why* each command and configuration is needed.
- Work on **one milestone and one checkpoint at a time**. Ask me to run commands when credentials, external websites, or cloud billing are involved.
- Prefer reproducible, version-controlled configuration over clicking through UIs; document necessary UI clicks.
- Target **Windows PowerShell**, **WSL2** and **Docker Desktop**. Explicitly identify commands that belong in PowerShell vs. Linux shell vs. Jenkins `sh` steps.
- Use Python for the demo app, `pytest` for tests, and Declarative Jenkins Pipeline syntax before introducing Scripted Pipeline.
- Do not assume proprietary infrastructure topology, VSIT definition, actual Huawei tenancy, or Cloudpoint/Farben access.

### Security and change-control requirements
- **Never** put passwords, access keys, registry tokens, kubeconfigs, cloud secrets, or client data in Git or Jenkins console output.
- Use Jenkins Credentials and minimum permissions. Store sample values in `.env.example`, not `.env` committed to Git.
- Avoid mounting the host Docker socket into a general-purpose Jenkins controller: it is highly privileged. If experimenting locally, explicitly explain risks and use a dedicated/isolated build agent or an appropriate safer builder where practical.
- Do not expose Jenkins to the public Internet for a practice project; use localhost or a controlled tunnel and understand webhook security before enabling external access.
- **No automatic production releases.** PROD is a *local simulation* until separately approved; include human approval and rollback evidence.
- For cloud work: show expected cost-causing resources and ask for explicit consent before creating them. Do not execute actions against real Huacomm/Farben/Huawei/customer environments without permission.
- Pin or document tested dependency/tool versions; avoid using unreviewed `latest` tags for deliverables.

### Definition of done for each task
For every task, record:
1. Concept learned (2–5 sentences).
2. Commands or files changed (paths included).
3. Verification commands and observed output (not just expected output).
4. One intentional failure tested and how to troubleshoot it.
5. Updates to the tracker and next smallest action.

If an exercise fails, **debug before proceeding**; don't declare the milestone complete because a file exists.

## 4. Prerequisites & recommended repository structure

### Prerequisite checklist
- [ ] Windows with VS Code and Git configured.
- [ ] Docker Desktop running Linux containers through WSL2.
- [ ] Python 3.11+ (or another supported pinned version) and a project virtual environment.
- [ ] GitHub repository created (private is fine); no credentials committed.
- [ ] Basic understanding of Docker image vs. container, Dockerfile, ports and volumes.
- [ ] Optional later: `kubectl`, local Kubernetes (e.g., kind), Helm basics.
- [ ] Optional later: **authorized** Huawei Cloud sandbox access to SWR and CCE.

### Suggested directory tree (create incrementally, not all at once)

```text
jenkins-huawei-cicd-lab/
├── AGENTS.md                     # Codex operating rules / learning behavior
├── README.md                     # Setup, architecture and demo walkthrough
├── docs/
│   ├── learning-path.md          # Copy of this roadmap (single source of truth)
│   ├── concepts.md               # Personal notes / glossary
│   ├── progress.md               # Evidence and session journal
│   ├── architecture.md           # CI/CD and environment diagrams
│   ├── runbook.md                # Promotion, validation, rollback
│   └── troubleshooting.md        # Real errors and fixes
├── app/
│   ├── main.py                   # Small FastAPI service with /health
│   └── __init__.py
├── tests/
│   └── test_health.py
├── requirements.txt              # Or pyproject.toml with locked deps
├── Dockerfile
├── .dockerignore
├── .gitignore
├── .env.example
├── Jenkinsfile
├── infra/
│   ├── jenkins/                  # Local Jenkins setup (when created)
│   ├── kubernetes/               # Generic deployment templates
│   └── environments/             # VSIT/SIT/UAT/PROD sample configs
└── scripts/                      # Health checks and release validation
```

## 5. Roadmap at a glance

| Milestone | Focus | Practical deliverable | Exit criterion |
|---|---|---|---|
| **M0** | Tools & foundations | Running Docker, Git repo, environment audit | Baseline checks passed |
| **M1** | CI/CD and Jenkins concepts | Concept notes + basic local Jenkins | Can explain CI/CD and access Jenkins |
| **M2** | First Jenkinsfile | Simple checkout + echo + test pipeline | Green and deliberately red builds observed |
| **M3** | GitHub + FastAPI CI | Lint/test on branch/PR or push | Code change triggers reliable CI |
| **M4** | Docker packaging | Tested application image | Build and run app from image locally |
| **M5** | Jenkins + image registry | Image build + push to *practice* registry | Exact tag/digest available for pull |
| **M6** | Local environment promotion | VSIT→SIT→UAT→PROD simulation | Same digest deployed with gates |
| **M7** | Huawei Cloud fundamentals | SWR/CCE/IAM deployment mapping | Explain authorization and network flows |
| **M8** | Optional Huawei Cloud lab | Image in SWR + workload in CCE | Verified deployment or documented access blocker |
| **M9** | Release engineering | Rollback, audit trail, runbook, demo | Reproduce full workflow and explain failures |

**Pace:** Learn by verified milestones rather than fixed weekly deadlines. M0–M6 can be completed without spending on Huawei Cloud.

---

## 6. Detailed milestones

### M0 — Establish local development baseline

**Learn:** What runs on Windows, WSL2, Docker Desktop, and inside Jenkins agents.

**Tasks**
- [ ] Confirm `git --version`, `docker version`, `docker compose version`, `python --version`.
- [ ] Run `docker run --rm hello-world` successfully.
- [ ] Create the GitHub repository and `.gitignore`.
- [ ] Record the starting versions and OS/shell details in `docs/progress.md`.
- [ ] Draw a basic diagram: developer → GitHub → Jenkins → image → runtime.

**Verify:** Docker works and the repo has an initial commit. **Failure drill:** intentionally stop Docker Desktop, observe the error, and explain how to recover.

### M1 — Understand Jenkins and run it locally

**Learn:** CI/CD; controller vs. agent; freestyle job vs. pipeline; stages, steps, workspaces, plugins, credentials; polling vs. webhooks.

**Tasks**
- [ ] Read the Jenkins Pipeline and Docker installation guides.
- [ ] Launch Jenkins locally with a persistent named volume, following the current official Docker installation documentation.
- [ ] Complete secure first-run setup: admin user, minimum plugins, localhost-only exposure.
- [ ] Identify the Jenkins Home volume and learn restart vs. data deletion.
- [ ] Create a simple throwaway job in the UI and inspect console logs.

**Verify:** Log in to Jenkins; restart its container and confirm jobs persist. **Failure drill:** run a failing shell command and inspect build status.

### M2 — Jenkins Pipeline-as-Code

**Learn:** Declarative `pipeline`, `agent`, `stages`, `steps`, `environment`, `post`, parameters, `when`, and return codes.

**Tasks**
- [ ] Commit a basic `Jenkinsfile` to Git.
- [ ] Configure a Jenkins Pipeline job to read the `Jenkinsfile` from SCM.
- [ ] Add stages: Checkout → Validate → Test (initial placeholders are okay).
- [ ] Add `post` behavior for success, failure and cleanup.
- [ ] Trigger a pipeline manually and inspect stage-level logs.

**Verify:** At least one successful and one intentionally failed run, with logs and cause explained. **Failure drill:** deliberately use an incorrect command and fix it.

### M3 — FastAPI application and automated CI

**Learn:** unit tests, test reports, linting, dependency management, build failures and branch workflows.

**Tasks**
- [ ] Create minimal FastAPI app with `/health` returning a predictable response.
- [ ] Add `pytest` tests; optionally use `ruff` to lint.
- [ ] Run the same checks locally and from Jenkins.
- [ ] Configure Git integration: webhook **only if securely reachable**, otherwise SCM polling or manual triggers first.
- [ ] Produce clear test output and keep commits small.

**Verify:** A passing change yields green CI and a deliberately broken test blocks success. **Failure drill:** commit a failing test on a practice branch and show rejection.

### M4 — Containerize the application

**Learn:** image layers, Dockerfile, build context, health endpoints, ports, container lifecycle, `.dockerignore`.

**Tasks**
- [ ] Write a minimal, reasonably secure Dockerfile for FastAPI.
- [ ] Build a tagged image locally.
- [ ] Run the container and call `/health`.
- [ ] Add a Jenkins stage to build the image using an explicitly reviewed build-agent strategy.
- [ ] Ensure credentials and unnecessary files do not enter image layers.

**Verify:** Rebuild from a clean checkout, run the container, and reach the endpoint. **Failure drill:** break the configured port or startup command, diagnose and fix.

### M5 — Registry publishing and immutable artifacts

**Learn:** image name/tag/digest, registry authentication, image push/pull, supply-chain basics.

**Tasks**
- [ ] Set up an **isolated practice registry** locally or use a permitted personal registry.
- [ ] Authenticate through Jenkins Credentials; never echo secrets.
- [ ] Publish the Docker image with a unique version or commit-based tag.
- [ ] Capture and record the **image digest** from the registry.
- [ ] Pull and run the pushed image from a fresh container instance.
- [ ] Explain difference between promoting an image by digest and rebuilding code for each environment.

**Verify:** Pulled image digest matches published artifact. **Failure drill:** attempt a pull with a wrong tag or unauthorized credentials and explain the error.

### M6 — Simulate VSIT → SIT → UAT → PROD locally

**Learn:** environment isolation, deployment manifests, Kubernetes namespaces/contexts, promotion gates, business vs. technical sign-off.

**Tasks**
- [ ] Choose a local target: Docker Compose (simpler) or kind Kubernetes (closer to CCE). Record why.
- [ ] Create separate **simulated** VSIT/SIT/UAT/PROD environments; for Kubernetes practice use separate namespaces and configurations.
- [ ] Deploy **the same immutable image digest** to VSIT, then SIT, UAT, and PROD simulation.
- [ ] Validate `/health` and an application-version endpoint after each deployment.
- [ ] Add automated verification and manual approval before UAT and PROD simulation.
- [ ] Use distinct dummy configuration for each environment; no real data.
- [ ] Demonstrate a failed rollout and rollback to the prior version.

**Verify:** A release record shows exact digest, promotion history, checks, approvals, and rollback result. **Failure drill:** introduce a broken image or bad config and restore last known good state.

### M7 — Map the local workflow to Huawei Cloud

**Learn:** Huawei SWR (container registry), CCE (managed Kubernetes), IAM (access), VPC (networking), ECS (possible Jenkins host), LTS (logs), and optionally CodeArts as an alternative.

**Tasks**
- [ ] Draw how Jenkins authenticates to SWR, pushes an image, and deploys to CCE.
- [ ] Distinguish shared-registry promotion from cross-registry image replication.
- [ ] Identify where Jenkins runs, which identity it uses, and what network connectivity it needs.
- [ ] Document which parts would require IAM permission and secret storage.
- [ ] Document where UAT sign-off, change approval, and deployment ownership sit.
- [ ] List questions for the delivery/cloud team about *actual* VSIT/SIT/UAT/PROD topology.

**Verify:** Explain the architecture to a colleague without confusing image storage, Jenkins automation and Kubernetes runtime.

### M8 — Optional: Authorized Huawei Cloud sandbox implementation

**Prerequisite:** A non-production Huawei Cloud account/region, permission to use SWR and CCE, network access, and explicit agreement on costs.

**Tasks**
- [ ] Record selected region, registry, cluster, namespace, account boundary, IAM role and expected costs **without storing secrets**.
- [ ] Create or use an approved SWR organization/repository.
- [ ] Configure Jenkins access with least-privilege credentials.
- [ ] Push a practice Docker image to SWR and verify digest.
- [ ] Deploy to an authorized CCE namespace using `kubectl` or a reviewed deployment workflow.
- [ ] Verify rollout, logs and endpoint health; perform a controlled rollback.
- [ ] Clean up billable resources after the lab, per authorization.

**If cloud access is unavailable:** mark **Blocked — external dependency**, not failed; keep local simulation as completed evidence and add the intended cloud command mapping to `docs/architecture.md`.

**Verify:** A sandbox CCE workload runs the exact image digest from SWR and has logged rollout/rollback evidence (or blockers are documented).

### M9 — Production-style readiness and knowledge transfer

**Learn:** release traceability, separation of duties, approval gates, rollback criteria, operational monitoring, incident escalation.

**Tasks**
- [ ] Produce a concise deployment architecture diagram and release runbook.
- [ ] Record tested build ID, commit SHA, image digest, environment, approval, deployment result.
- [ ] Document checks before/after deploy, log locations and rollback steps.
- [ ] Run a 10-minute demo: push change → test → build → publish → promote → validate → rollback.
- [ ] Record what Jenkins automates vs. what engineers/customer approvers decide.
- [ ] Prepare a list of questions to compare this lab against real project meeting minutes.

**Verify:** Another developer can follow the README and reproduce the local lab without access to my machine or secrets.

---

## 7. Progress tracker (update after each session)

| Milestone | Status | Evidence / link | Blockers | Last updated |
|---|---|---|---|---|
| M0 | Completed | See `progress.md` and `architecture.md` | Mermaid visual preview remains unobserved | 2026-11-06 |
| M1 | Not started | — | — | 2026-11-05 |
| M2 | Not started | — | — | 2026-11-05 |
| M3 | Not started | — | — | 2026-11-05 |
| M4 | Not started | — | — | 2026-11-05 |
| M5 | Not started | — | — | 2026-11-05 |
| M6 | Not started | — | — | 2026-11-05 |
| M7 | Not started | — | — | 2026-11-05 |
| M8 | Not started | — | Requires authorized cloud access | 2026-11-05 |
| M9 | Not started | — | — | 2026-11-05 |

**Allowed statuses:** Not started / In progress / Completed / Blocked.
**Current milestone:** **M1 — Understand Jenkins and run it locally**.
**Next action:** Read the official Jenkins Pipeline and Docker installation guides; record key concepts and questions before starting Jenkins.

### Session journal template

```markdown
## YYYY-MM-DD — Milestone M# / checkpoint
- Goal:
- Concepts learned:
- Commands run:
- Files changed:
- Verification results (actual):
- Failure tested and diagnosis:
- Blockers / questions:
- Next checkpoint:
```

## 8. How to use this file with Codex

1. Save this file in your repo as `docs/learning-path.md`.
2. Create `AGENTS.md` containing or referencing the **Ground rules for Codex** above.
3. Start with the prompt below; do not ask Codex to generate all stages at once.
4. At the end of each session, ask Codex to update the progress tracker and journal based on **verified results**, commit changes, and identify the next exercise.

### Codex starter prompt

```text
You are my Jenkins/Huawei Cloud CI/CD learning coach and pair programmer.
Read docs/learning-path.md fully before starting. Follow section 3 (Ground rules
for Codex) strictly. I am developing on Windows with VS Code, Docker Desktop
and GitHub. We are learning for a personal sandbox, NOT accessing company or
customer infrastructure.

Begin with milestone M0 only. First check what files and tools already exist,
then explain the minimum next exercise and provide PowerShell-friendly
commands. Do not implement later milestones. After I show execution results,
help me verify them, troubleshoot if necessary, and record evidence in
progress.md and the learning-path tracker. Avoid cloud resources, secrets,
and billable changes without my explicit approval.
```

### Useful follow-up prompt

```text
Review the current milestone acceptance criteria in docs/learning-path.md.
Check existing code, configuration, tests and logs. Tell me which criteria
are verified, which are unverified, and the single next exercise. Do not mark
a task complete without observable evidence.
```

## 9. Questions to resolve from the real deployment discussion

- What does **VSIT** mean in this particular project?
- Is Jenkins on Huawei ECS, inside CCE, or hosted elsewhere?
- Is the container registry Huawei **SWR**, another registry, or both?
- Is there one shared registry or separate registries/accounts per environment?
- Does “pull images from VSIT to SIT” mean *deploy the same SWR artifact* or *copy/replicate an image between registries*?
- Are environments different CCE clusters, namespaces, VPCs, regions, or tenants?
- Who manages Jenkins pipelines, SWR, CCE, credentials, networking, release approvals and rollback?
- What are the exit criteria for SIT and UAT, and who authorizes PROD rollout?
- What logs, health checks, and audit evidence are expected after deployment?

## 10. Official references

- Jenkins Handbook: https://www.jenkins.io/doc/book/
- Jenkins Pipeline: https://www.jenkins.io/doc/book/pipeline/
- Jenkinsfile reference: https://www.jenkins.io/doc/book/pipeline/jenkinsfile/
- Jenkins with Docker: https://www.jenkins.io/doc/book/pipeline/docker/
- Jenkins installation with Docker: https://www.jenkins.io/doc/book/installing/docker/
- Huawei Cloud SWR docs: https://support.huaweicloud.com/intl/en-us/swr/
- Huawei Cloud CCE docs: https://support.huaweicloud.com/intl/en-us/cce/
- Huawei Cloud documentation home: https://support.huaweicloud.com/intl/en-us/

> **Version note:** Commands, supported cloud regions, service availability, and billing can change. Check current official documentation when performing cloud or tool installation steps.

---

## 11. Completion rubric

- **Foundation:** I can explain Jenkins, agents, stages and CI/CD without notes.
- **Functional CI:** A commit produces a visible pass/fail result from real tests.
- **Artifact integrity:** One tagged and digest-identified image is promoted, not rebuilt, between stages.
- **Deployment control:** Environment-specific config, verification, approvals, rollback and audit trail work in the local simulation.
- **Cloud fluency:** I can clearly diagram SWR ↔ CCE ↔ Jenkins access and can distinguish orchestration, registry and runtime responsibilities.
- **Operational readiness:** A new developer can reproduce the lab from README and troubleshoot a failed release.

**Learning philosophy:** Build the smallest working pipeline first, prove it works, break it intentionally, fix it, then add the next layer.
