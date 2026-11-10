# M3 FastAPI Health Service — Design

## Intent

Add the smallest real Python service that gives the Jenkins learning lab an executable, testable application artifact before CI integration.

## Goals

- Provide `GET /health` through a FastAPI app.
- Return the exact JSON body `{"status": "ok"}` with HTTP 200.
- Add one pytest test using FastAPI's `TestClient`.
- Use a root `requirements.txt` with explicit direct dependency versions.
- Verify the check locally before a later Jenkins Pipeline update.

## Non-goals

- No Dockerfile, registry, cloud, database, authentication, webhook, Jenkinsfile changes, or deployment.
- No application configuration, environment variables, or secrets.
- No additional routes beyond `/health`.

## Selected approach

Use `app/main.py` for the FastAPI application, `app/__init__.py` to make the package explicit, `tests/test_health.py` for the endpoint contract, and `requirements.txt` for the direct runtime/test dependencies. Pin exact compatible versions after checking their official release metadata.

## Constraints

- Keep the response predictable: HTTP 200 and exactly `{"status": "ok"}`.
- Keep the test independent of a running server and external services.
- Use Python 3.11.4, already verified in M0.
- Keep Jenkins integration as the next M3 checkpoint only after local tests pass.

## Approval

The user selected the `requirements.txt` plus FastAPI `TestClient` approach on 2026-11-10. This design requires explicit approval before implementation.
