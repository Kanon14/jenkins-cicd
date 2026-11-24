# M3 Jenkins Python Agent Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Run the passing FastAPI pytest check in Jenkins on a dedicated Python agent.

**Architecture:** Build a small, version-pinned inbound Jenkins agent image with Python 3.11 and the repository's pinned test dependencies. Connect it over WebSocket through Jenkins' existing localhost-published HTTP endpoint, with the agent and controller sharing a private Docker network; route the Pipeline Test stage to the `python-test` label.

**Tech Stack:** Docker Desktop, Jenkins inbound agent, Java 21, Python 3.11, FastAPI 0.162.2, HTTPX2 2.15.0, pytest 9.2.0, Declarative Pipeline.

**Spec:** `docs/superpowers/specs/2026-11-11-m3-jenkins-python-agent-design.md`

## Global Constraints

- Keep builds off the Jenkins built-in node and use the `python-test` agent label.
- Preserve the `127.0.0.1:8080` host binding; do not publish new host ports.
- Use a private Docker network and WebSocket inbound agent connection.
- Keep the agent workspace separate from `/var/jenkins_home`.
- Do not mount the host Docker socket or use privileged containers.
- Pin base-image and Python dependency versions; do not use unreviewed `latest` tags.
- Keep the inbound agent secret out of Git, command history, console logs, and chat.
- Do not perform Docker image builds, registry publishing, cloud access, deployment, or webhook configuration.

## Review Focus

- The agent must connect to Jenkins through a resolvable container-network address, not `127.0.0.1` from inside the agent container.
- The agent container must not receive the Jenkins home volume or host Docker socket.
- Jenkins must not schedule Python project code on the built-in controller node.
- The Test stage must execute the exact pinned test dependencies and fail on a deliberately broken health assertion.
- The agent secret must be supplied without appearing in source files, shell history, or logs.

---

### Task 1: Add the dedicated Python agent image

**Files:**
- Create: `infra/jenkins-agent/Dockerfile`
- Modify: `docs/progress.md`
- Modify: `docs/learning-path.md`

**Interfaces:**
- Produces: a locally buildable image tagged `jenkins-python-agent:3.11` with Java 21, Python 3.11, and the repository's pinned dependencies.
- Consumes: root `requirements.txt` and the official inbound agent base image.

- [x] **Step 1: Add the agent Dockerfile using an explicit official inbound-agent version tag**

Install Python 3.11 in the agent image, copy only `requirements.txt`, install the exact pins, and retain the official inbound-agent entrypoint. Do not install or configure a Docker CLI.

- [x] **Step 2: Build the image locally** *(user-run; verified by successful runs from the resulting image)*

Run: `docker build -t jenkins-python-agent:3.11 -f infra/jenkins-agent/Dockerfile .`

Expected: image builds successfully. If registry TLS blocks the base-image pull, stop and record the blocker; do not disable certificate verification.

- [x] **Step 3: Verify tools and dependency versions in the image**

Run the image's Python version and `pip show fastapi httpx2 pytest` checks.

Expected: Python 3.11 and exact project pins are displayed; no Jenkins secret is involved.

### Task 2: Connect a dedicated agent to local Jenkins

**Files:**
- No file changes before the node is online.
- Modify after successful connection: `docs/progress.md`, `docs/learning-path.md`

**Interfaces:**
- Consumes: image `jenkins-python-agent:3.11` from Task 1.
- Produces: online Jenkins node labeled `python-test`, one executor, private-network WebSocket connection.

- [x] **Step 1: Inspect the current Docker networks and Jenkins attachments**

Run read-only Docker commands to list networks and inspect `jenkins-lab` attachments. Do not disconnect it from any existing network.

- [x] **Step 2: Create a dedicated private network and attach Jenkins**

Create a named network for this lab and attach `jenkins-lab` to it. Verify the localhost host port remains `127.0.0.1:8080`.

- [x] **Step 3: Create permanent node `python-test` in Jenkins**

Set one executor, a separate remote root, label `python-test`, and inbound WebSocket connection. Obtain the generated agent secret only through Jenkins' local UI.

- [x] **Step 4: Start the agent without exposing the secret**

Use a PowerShell secure prompt or a local-only environment variable that is not echoed or saved in the repository. Start the container attached only to the dedicated network, with a named isolated agent workspace volume. Set `JENKINS_URL` to the controller's container-network URL (for example `http://jenkins-lab:8080/` if that name resolves on the shared network), `JENKINS_AGENT_NAME=python-test`, and WebSocket mode.

- [x] **Step 5: Verify the agent is online and isolated**

Expected: Jenkins node page reports the agent online; Docker inspect shows the agent only on the dedicated network, an isolated workspace volume, and no controller Jenkins home mount or host Docker socket; the official base image may also define its own `/home/jenkins/.jenkins` volume. The controller host port remains localhost-only.

### Task 3: Run the application test on the dedicated agent

**Files:**
- Modify: `Jenkinsfile`
- Modify: `docs/progress.md`
- Modify: `docs/learning-path.md`

**Interfaces:**
- Consumes: online agent label `python-test` from Task 2 and root `tests/test_health.py`.
- Produces: Jenkins build evidence that pytest runs on the Python agent.

- [x] **Step 1: Update the Jenkinsfile to use agent label `python-test`**

Keep Checkout and Validate behavior. In Test, run `python --version` followed by `python -m pytest -v -W error::DeprecationWarning`.

- [x] **Step 2: Run the Pipeline from `main`**

Expected: Jenkins confirms the Test stage runs on `python-test` and reports one passing health test.

- [x] **Step 3: Verify the intentional failing-test drill on a temporary branch**

Temporarily change the expected health body in the test, run the job on an isolated practice branch, and confirm pytest fails and Jenkins marks the Pipeline failed. Restore the expected body and point the job back to `main`; do not merge the failure branch.

- [x] **Step 4: Record actual results and commit repository changes** *(implementation commit `fa10855` was user-pushed; completion record is in a follow-up commit)*

Record actual image, node, success-build, and isolated failure-build evidence. Commit the Dockerfile, Jenkinsfile, and tracker updates only after user verification of the Jenkins UI steps.
