# M4 Application Container Design

## Intent

Package the existing FastAPI health app as a small local Docker image so the learner can see how application code, pinned dependencies, a startup command, and port mapping produce a runnable container.

## Goals

- Add a pinned Uvicorn dependency to `requirements.txt` and `pyproject.toml`, then refresh `uv.lock`.
- Keep the Jenkins Python agent consistent with the shared requirements by rebuilding its image after the dependency update.
- Add a root application `Dockerfile` based on the same pinned Python 3.11 slim Bookworm release used by the agent.
- Run the app as a non-root user and start it with `uvicorn app.main:app` listening on container port 8000.
- Build and run the image locally with a localhost-only host port, then verify `GET /health` returns HTTP 200 and `{"status":"ok"}`.
- Update the learning tracker only after the user supplies the build and endpoint results.

## Non-goals

- Do not add a Jenkins image-build stage in this checkpoint. The current agent has no Docker CLI or Docker socket; a later stage needs its own reviewed build strategy.
- Do not publish the image to a registry, connect to Huawei Cloud, add secrets, or alter the FastAPI endpoint.

## Constraints and anti-goals

- Preserve the existing test behavior and exact dependency pinning.
- Keep `.venv`, caches, `.env` files, keys, and Jenkins data out of the Docker build context using the existing `.dockerignore`.
- Do not run the container as root, expose a host port beyond `127.0.0.1`, or mount the Docker socket.
- Keep the exercise to one checkpoint at a time; do not mark M4 complete until the local container endpoint is verified.

## Approaches considered

1. **Use the shared pinned dependencies (selected).** Add Uvicorn to the existing requirements and project metadata, refresh the lock, and rebuild the test agent. This keeps app runtime and CI dependency declarations aligned; the extra agent rebuild is the tradeoff.
2. **Add a separate runtime requirements file.** This avoids rebuilding the test agent now, but creates a second dependency list that can drift from the test environment.

## Decision

Use approach 1. Build a single-stage Python image first because the app is small and this checkpoint is meant to teach the basic Dockerfile, build context, startup command, and port mapping. Revisit a multi-stage or lock-driven production image after the first local run works.

## Remaining work

After approval, implement the dependency and Dockerfile changes, refresh the lock file, and give the learner the local image build command. Wait for their output before moving to container startup and `/health` verification.
