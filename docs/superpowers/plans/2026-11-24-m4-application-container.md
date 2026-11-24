# M4 Application Container Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build and run the FastAPI `/health` app in a small, non-root Docker container on the local machine.

**Architecture:** Add Uvicorn as a pinned project dependency shared by local development and the Jenkins Python agent. Build a single-stage image from `python:3.11.4-slim-bookworm`, copy only the app and dependency manifest, run as UID/GID 10001, and listen on container port 8000. Keep this checkpoint local; the Jenkins image-build stage needs a separate reviewed strategy.

**Tech Stack:** Python 3.11.4, FastAPI, Uvicorn 0.54.0, Docker Desktop, uv.

**Spec:** `docs/superpowers/specs/2026-11-24-m4-application-container-design.md`

## Global Constraints

- Pin Uvicorn to `0.54.0` in `pyproject.toml` and `requirements.txt`.
- Keep `uv.lock` in sync with `pyproject.toml`.
- Keep the Jenkins Python agent aligned with `requirements.txt`; rebuild its image and recreate its container using the existing network/workspace configuration and a securely supplied secret.
- Use `python:3.11.4-slim-bookworm` for the application image.
- Run the app as UID/GID `10001:10001` and listen on container port `8000`.
- Publish the app only at `127.0.0.1:8000`; do not mount the Docker socket or add secrets to the image.
- Use the existing `.dockerignore` to exclude `.venv`, caches, `.env` files, keys, and Jenkins data from the build context.
- Do not add a Jenkins image-build stage or publish an image to a registry in this checkpoint.
- Do not add or run automated tests; use the milestone's Docker build and manual `/health` check as verification.

## Review Focus

- **Dependency drift:** The rebuilt agent must include Uvicorn 0.54.0 so Jenkins' offline preflight passes; verify the installed version before replacing the running agent.
- **Build context:** `.venv`, caches, secrets, and Jenkins data must remain excluded at every directory depth by `.dockerignore`; verify with harmless nested marker files and inspect the built image.
- **ASGI target and port:** Startup must use `app.main:app` on `0.0.0.0:8000`; verify the container stays running and `/health` returns the expected JSON.
- **Privileges:** The application process must not run as root; inspect `id` inside the running container.
- **Host exposure:** The port mapping must bind to `127.0.0.1`; inspect the running container's published ports.

---

### Task 1: Add and lock the runtime server dependency

**Files:**
- Modify: `pyproject.toml`
- Modify: `requirements.txt`
- Modify: `uv.lock`

**Interfaces:**
- Consumes: current FastAPI project and pinned dependency manifest.
- Produces: `uvicorn==0.54.0` as a project dependency and in the shared requirements used by the app and Jenkins agent images.

- [x] **Step 1: Add the exact Uvicorn pin and refresh the project lock**

Run: `uv add --no-sync "uvicorn==0.54.0"`

Expected: `pyproject.toml` declares the exact version and `uv.lock` contains a matching resolved package entry; the virtual environment is not synchronized by this command.

- [x] **Step 2: Add the same pin to the pip requirements file**

Add one line, `uvicorn==0.54.0`, to root `requirements.txt`.

Expected: The Uvicorn pin matches in `requirements.txt` and `pyproject.toml`.

- [x] **Step 3: Check lock consistency**

Run: `uv lock --check`

Expected: The lock file is up to date with `pyproject.toml`.

### Task 2: Rebuild and refresh the Jenkins test agent

**Files:**
- No source changes; uses `infra/jenkins-agent/Dockerfile` and updated root `requirements.txt`.

**Interfaces:**
- Consumes: updated shared dependencies from Task 1.
- Produces: a `jenkins-python-agent:3.11` image and online `python-test` node with the new dependency installed.

- [x] **Step 1: Rebuild the agent image** *(user-reported complete)*

Run: `docker build -t jenkins-python-agent:3.11 -f infra/jenkins-agent/Dockerfile .`

Expected: The build completes with the updated shared requirements.

- [x] **Step 2: Verify Uvicorn in the rebuilt image** *(user output: 0.54.0)*

Run: `docker run --rm --entrypoint python jenkins-python-agent:3.11 -m pip show uvicorn`

Expected: `Version: 0.54.0`.

- [x] **Step 3: Recreate the connected agent container with the rebuilt image** *(user-run; Jenkins status verified connected)*

After confirming no Jenkins build is running, replace `jenkins-python-test-agent` using its existing settings: node name `python-test`, WebSocket mode, private network `jenkins-agent-network`, named volume `jenkins-python-test-workspace`, and the current controller URL. Supply the secret through the masked PowerShell prompt; do not place it literally in the command, history, repository, or chat. Keep the workspace volume, publish no ports, and do not mount the Docker socket.

Expected: Jenkins reports `python-test` online on the rebuilt image. Confirm it reconnects before proceeding.

### Task 3: Add the local application Dockerfile

**Files:**
- Create: `Dockerfile`

**Interfaces:**
- Consumes: root `requirements.txt`, `app/main.py`, and `.dockerignore`.
- Produces: an image that starts `app.main:app` under Uvicorn on container port 8000 as UID/GID 10001.

- [x] **Step 1: Add a single-stage Python image**

Use `python:3.11.4-slim-bookworm`; set `/app` as the working directory; install `requirements.txt`; copy `app/` with ownership `10001:10001`; set `USER 10001:10001`; and use the JSON-form command `python -m uvicorn app.main:app --host 0.0.0.0 --port 8000`. Do not copy the repository wholesale or add a healthcheck in this learning checkpoint.

Expected: Only the application package and dependency manifest are copied; the image runs the app as the non-root UID.

### Task 4: Build and verify the local app image

**Files:**
- No source changes; local Docker image/container only.

**Interfaces:**
- Consumes: root `Dockerfile` and updated requirements from Tasks 1 and 3.
- Produces: local image `jenkins-cicd-app:0.1.0`; its `/health` endpoint returns `{"status":"ok"}`.

- [x] **Step 1: Build the tagged image** *(user-reported complete)*

Run: `docker build -t jenkins-cicd-app:0.1.0 .`

Expected: Docker reports a successful image build.

- [x] **Step 2: Start the app on localhost only** *(user output: Up; 127.0.0.1:8000->8000/tcp)*

Run: `docker run --rm --detach --name jenkins-cicd-app --publish 127.0.0.1:8000:8000 jenkins-cicd-app:0.1.0`

Expected: Docker returns a container ID; `docker ps --filter "name=jenkins-cicd-app"` shows the port bound to `127.0.0.1:8000`.

- [x] **Step 3: Verify the process identity and health endpoint** *(user output: UID/GID 10001; `/health` returned `{"status":"ok"}`)*

Run: `docker exec jenkins-cicd-app id` and `Invoke-RestMethod http://127.0.0.1:8000/health`

Expected: The process reports UID 10001, and PowerShell prints `status : ok`.

- [x] **Step 4: Stop the temporary app container** *(user output: `jenkins-cicd-app`)*

Run: `docker stop jenkins-cicd-app`

Expected: The app container stops and is removed by `--rm`; the image remains available locally.

- [x] **Step 5: Record verified M4 progress and commit**

Update the learning tracker and progress journal only with observed build, port mapping, UID, endpoint, and agent reconnection results. Commit the M4 changes after the user's local verification.

## Scope boundary

The learning path also asks for a Jenkins image-build stage. This plan stops after local container verification. Before adding that stage, review safe Docker build options for an agent that has no Docker CLI, host socket, or privileged mode.
