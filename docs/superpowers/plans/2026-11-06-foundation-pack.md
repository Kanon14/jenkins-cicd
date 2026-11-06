# Jenkins CI/CD Lab Foundation Pack Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create the safe repository foundation and M0 evidence record for the Jenkins CI/CD learning lab.

**Architecture:** Documentation under `docs/` holds the roadmap and evidence journal; repository-root files provide operating rules, onboarding, and safeguards. No executable CI/CD assets are introduced at this stage.

**Tech Stack:** Markdown, Git, Windows PowerShell, Docker Desktop.

**Spec:** `docs/superpowers/specs/2026-11-06-foundation-pack-design.md`

## Global Constraints

- Work one milestone and checkpoint at a time.
- Preserve actual M0 evidence; do not invent unobserved output.
- Do not add secrets, real environment values, cloud credentials, or customer data.
- Do not create Jenkins, FastAPI, Docker, Kubernetes, registry, or cloud configuration.
- Use Windows PowerShell-friendly wording.

## Review Focus

- `.gitignore` must ignore `.env` while explicitly retaining `.env.example`.
- The progress journal must distinguish the observed Docker failure from the successful recovery.
- The roadmap must remain complete after copying into `docs/learning-path.md`.
- `AGENTS.md` must prohibit unauthorized cloud or production actions and secret handling.
- README links must resolve to the roadmap and progress journal from the repository root.

---

### Task 1: Establish documentation and safety controls

**Files:**
- Create: `AGENTS.md`
- Create: `.gitignore`
- Create: `.env.example`
- Create: `docs/learning-path.md`
- Create: `docs/progress.md`
- Modify: `README.md`

**Interfaces:**
- Consumes: the approved design and `Jenkins_Huawei_Cloud_CICD_Learning_Path.md`.
- Produces: the repository-local learning instructions, roadmap, M0 evidence journal, and onboarding links used by later milestones.

- [ ] **Step 1: Copy the complete approved roadmap to `docs/learning-path.md`**

Preserve its status, milestone sequence, safety requirements, progress tracker, and official-reference links.

- [ ] **Step 2: Add `AGENTS.md` with the repository learning boundaries**

Require an instructor/pair-programmer approach, verified evidence before completion, one milestone/checkpoint at a time, no secret commits, and explicit authorization for cloud, customer, production, or billable actions.

- [ ] **Step 3: Add safe local-file controls**

Create `.gitignore` for Python caches, virtual environments, local `.env` files, test/build artifacts, editor settings, and Jenkins runtime data, while retaining `.env.example`. Create `.env.example` containing comments only.

- [ ] **Step 4: Record M0 evidence in `docs/progress.md`**

Use the session-journal template. Record the supplied version values, the successful `hello-world` run, the observed stopped-daemon error, and the user-reported verified restart. Leave unobserved command output unquoted.

- [ ] **Step 5: Update `README.md`**

Describe the lab’s learning purpose, safe sandbox boundary, current M0 status, and relative links to the roadmap and progress journal.

- [ ] **Step 6: Verify content and repository state**

Run: `git status --short`, `git diff --check`, and `git diff -- . ':!docs/learning-path.md'`.

Expected: only the foundation-pack files are changed; no whitespace errors; no secrets or later-milestone configurations appear.

- [ ] **Step 7: Commit after user review**

Run: `git add AGENTS.md .gitignore .env.example README.md docs && git commit -m "docs: establish Jenkins learning lab foundation"`.

Expected: one focused documentation-and-safety commit. Do not commit until the user approves the changes.
