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

## 2026-11-06 — Milestone M0 / conceptual CI/CD flow

- Goal: Capture the minimal developer → GitHub → Jenkins → image → runtime relationship before installing or configuring Jenkins.
- Concepts learned: GitHub stores version-controlled application and pipeline definitions. Jenkins will later orchestrate testing and image builds; the Docker image is the deployable artifact, while the local runtime executes it.
- Commands run:
  - `git remote -v`
- Files changed:
  - `docs/architecture.md`
- Verification results (actual):
  - `docs/architecture.md` did not exist before this checkpoint and was then created.
  - The configured `origin` remote is `https://github.com/Kanon14/jenkins-cicd.git` for fetch and push.
- Failure tested and diagnosis: Not applicable to this static conceptual Markdown artifact. Visual Mermaid rendering has not yet been observed and remains a later manual preview check.
- Blockers / questions: None.
- Next checkpoint: Record the actual Windows and PowerShell version details, then assess M0 completion criteria.

## 2026-11-06 — Milestone M0 / completion assessment

- Goal: Confirm the host operating system and PowerShell details, then assess the M0 exit criterion.
- Concepts learned: Windows PowerShell 5.1 is the shell used for the local lab commands. The M0 baseline establishes the developer machine, Git repository, Docker engine, and conceptual delivery flow before Jenkins is introduced.
- Commands run:
  - `$PSVersionTable.PSVersion`
  - `(Get-CimInstance Win32_OperatingSystem).Caption`
- Files changed:
  - `docs/progress.md`
  - `docs/learning-path.md`
- Verification results (actual):
  - PowerShell: Major `5`, Minor `1`, Build `26100`, Revision `9711`.
  - Operating system: `Microsoft Windows 11 Home Single Language`.
  - M0 exit criterion is met: Docker completed `hello-world`, the Docker stopped-daemon failure/recovery was observed, Git and Python versions were recorded, the GitHub remote and initial commit exist, and the conceptual CI/CD diagram is documented.
- Failure tested and diagnosis: The M0 Docker daemon failure drill was previously completed and documented above; no additional failure was needed for this evidence-only completion assessment.
- Blockers / questions: Mermaid visual rendering remains unobserved, but the diagram source is present and does not block M0's Docker-and-repository exit criterion.
- Next checkpoint: Begin M1 by reading the official Jenkins Pipeline and Docker installation guides; do not start or expose Jenkins yet.

## 2026-11-06 — Milestone M1 / official documentation study

- Goal: Establish the Pipeline and Docker concepts needed before running a local Jenkins controller.
- Concepts learned: Jenkins Pipeline models delivery work as code in a `Jenkinsfile`; Declarative Pipeline is the starting syntax for this lab. On Windows, the Jenkins Docker image requires Linux containers, and a persistent Jenkins home must be planned before the controller is started.
- Commands run: None. This was a read-only documentation checkpoint.
- Files changed:
  - `docs/concepts.md`
  - `docs/progress.md`
  - `docs/learning-path.md`
- Verification results (actual): Official Jenkins Pipeline and Docker installation guidance were reviewed on 2026-11-06; source links and takeaways are recorded in `docs/concepts.md`.
- Failure tested and diagnosis: Not applicable to this read-only study checkpoint; no Jenkins service or local infrastructure was started.
- Blockers / questions: Docker Desktop Linux-container mode and local resource availability are not yet verified.
- Next checkpoint: Inspect Docker's current operating system/type and available local resources without creating Jenkins resources.

## 2026-11-06 — Milestone M1 / Docker preflight

- Goal: Confirm that Docker Desktop can run the Linux-based Jenkins image and that the local machine has reasonable capacity before creating any Jenkins resources.
- Concepts learned: Docker Desktop can expose a Linux engine on Windows. Existing images, stopped containers, volumes, and build cache are local state; this lab must not delete or reclaim them unless explicitly requested.
- Commands run:
  - `docker version --format 'Server OS: {{.Server.Os}}/{{.Server.Arch}}'`
  - `docker info --format 'OSType: {{.OSType}} | Docker: {{.OperatingSystem}} | CPUs: {{.NCPU}} | Memory bytes: {{.MemTotal}}'`
  - `docker system df`
  - `Get-PSDrive -PSProvider FileSystem | Select-Object Name, Used, Free`
- Files changed:
  - `docs/progress.md`
  - `docs/learning-path.md`
- Verification results (actual):
  - Docker server: `linux/amd64`; Docker type: `linux`; platform: `Docker Desktop`.
  - Docker resources: `20` CPUs and `16,572,067,840` bytes of memory (approximately 15.4 GiB).
  - Docker currently stores 17 images (5.419 GB), 10 inactive containers, 17 local volumes, and 53 build-cache entries. These pre-existing resources were not changed.
  - The `C:` drive has `137,360,441,344` bytes free (approximately 127.9 GiB).
- Failure tested and diagnosis: Not applicable to this read-only preflight. Docker's Linux engine was already verified in the M0 daemon failure/recovery drill.
- Blockers / questions: None for a local, controller-only Jenkins exercise. The existing Docker resources must be preserved.
- Next checkpoint: Start one localhost-only Jenkins controller with a dedicated persistent volume; do not configure Docker-in-Docker, host-socket mounting, credentials, or cloud access.

## 2026-11-06 — Milestone M1 / local Jenkins controller started

- Goal: Run one local Jenkins controller with persistent home data and a localhost-only web binding.
- Concepts learned: A Jenkins controller is the service that coordinates jobs and Pipeline execution. The Docker port mapping controls host exposure, while a Docker volume preserves Jenkins home data across container restarts.
- Commands run:
  - `docker volume create jenkins-lab-home`
  - `docker run --name jenkins-lab --detach --publish 127.0.0.1:8080:8080 --volume jenkins-lab-home:/var/jenkins_home jenkins/jenkins:2.580.2-jdk21`
  - `docker ps --filter "name=jenkins-lab"`
  - Filtered Jenkins log readiness check for `Jenkins is fully up and running`
- Files changed:
  - `docs/progress.md`
  - `docs/learning-path.md`
- Verification results (actual):
  - Container `jenkins-lab` (`c325c6c2908a`) is `Up` with image `jenkins/jenkins:2.580.2-jdk21`.
  - The only published port shown is `127.0.0.1:8080->8080/tcp`.
  - Jenkins logged `hudson.lifecycle.Lifecycle#onReady: Jenkins is fully up and running`.
- Failure tested and diagnosis: A local agent-side `docker inspect` attempt was denied access to Docker's Windows named pipe. This did not affect the user-run Docker commands or the healthy container evidence. No Jenkins build failure drill has been performed yet.
- Blockers / questions: Initial administrator setup and persistence-through-restart are unverified. The initial administrator password is a secret and must not be pasted into chat or committed.
- Next checkpoint: Complete the local Jenkins first-run wizard at `http://127.0.0.1:8080` using the initial password only on the local machine, create a local administrator account, and select the minimum required plugins.

## 2026-11-06 — Milestone M1 / local first-run setup

- Goal: Secure the local controller with a non-default administrator account and install the initial plugin set.
- Concepts learned: Jenkins creates an initial administrator account during setup. Plugins extend Jenkins functionality, so the installed set should be known and reviewed rather than silently treated as part of Jenkins core.
- Commands run: None reported. Setup was completed in the local Jenkins browser wizard.
- Files changed:
  - `docs/progress.md`
  - `docs/learning-path.md`
- Verification results (user-reported):
  - The local Jenkins first-run wizard was completed.
  - A non-default local administrator account was created; its credentials were not shared or recorded.
  - The user chose **Install suggested plugins**, rather than the previously proposed minimum-plugin selection.
- Failure tested and diagnosis: Not applicable to the first-run browser setup. A Jenkins job failure drill remains required later in M1.
- Blockers / questions: Persistence across restart and login after restart are unverified.
- Next checkpoint: Restart `jenkins-lab`, wait for the readiness message, and confirm that the local administrator login still works.
