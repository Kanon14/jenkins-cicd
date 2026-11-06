# Learning Progress Journal

## 2026-11-06 — Milestone M0 / local environment baseline

- Goal: Verify the local Git, Docker, Docker Compose, and Python tools; perform a Docker daemon failure and recovery drill.
- Concepts learned: Docker Desktop provides the local Linux container engine used by Docker commands. A successful container run requires the client to reach that daemon. When the daemon is stopped, the missing named-pipe error identifies the local Docker engine as the immediate dependency to restore.
- Commands run:
  - `git --version`
  - `docker version --format 'Client: {{.Client.Version}}\`nServer: {{.Server.Version}}'`
  - `docker compose version`
  - `python --version`
  - `docker run --rm hello-world`
  - `docker desktop stop`
  - `docker desktop start`
  - `docker desktop status`
- Files changed: None during the environment audit. This journal was added after the evidence was collected.
- Verification results (actual):
  - Git: `git version 2.43.0.windows.1`.
  - Docker Client and Server: `29.8.1`.
  - Docker Compose: `v5.5.1`.
  - Python: `3.11.4`.
  - `docker run --rm hello-world` printed Docker's successful installation message.
  - After Docker Desktop was stopped, Docker reported: `failed to connect to the docker API at npipe:////./pipe/dockerDesktopLinuxEngine; check if the path is correct and if the daemon is running: open //./pipe/dockerDesktopLinuxEngine: The system cannot find the file specified.`
  - Docker Desktop was restarted and the user reported a verified successful recovery. The final command output was not captured in this journal.
- Failure tested and diagnosis: Stopping Docker Desktop made the Linux engine named pipe unavailable; restarting Docker Desktop restored the daemon.
- Blockers / questions: None.
- Next checkpoint: Draw the basic developer → GitHub → Jenkins → image → runtime learning diagram in `docs/architecture.md`.
