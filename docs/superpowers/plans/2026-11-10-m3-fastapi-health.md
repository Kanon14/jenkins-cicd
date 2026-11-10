# M3 FastAPI Health Service Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build and locally verify a minimal FastAPI `/health` endpoint with an exact pytest contract.

**Architecture:** `app/main.py` exposes the FastAPI instance and health route; `tests/test_health.py` exercises it in-process through `TestClient`, so no local server is required. `requirements.txt` pins the direct runtime and test dependencies verified against PyPI on 2026-11-10.

**Tech Stack:** Python 3.11.4, FastAPI 0.162.2, HTTPX 0.28.1, pytest 9.2.0.

**Spec:** `docs/superpowers/specs/2026-11-10-m3-fastapi-health-design.md`

## Global Constraints

- `GET /health` returns HTTP 200 and exactly `{"status": "ok"}`.
- Use `app/main.py`, `app/__init__.py`, `tests/test_health.py`, and root `requirements.txt`.
- Pin FastAPI 0.162.2, HTTPX 0.28.1, and pytest 9.2.0.
- Use the M0 Python 3.11.4 environment.
- Do not add Docker, a running web server, credentials, configuration, webhooks, or Jenkinsfile changes in this plan.
- Record Jenkins execution as an architecture blocker until a reviewed, isolated Python build-agent strategy is selected.

## Review Focus

- `/health` must use GET and not accept inputs or call external services.
- The test must assert both status code and exact JSON body.
- Dependencies must be exact pins, not floating versions.
- Test execution must not require a running Uvicorn server.
- Jenkins must not be modified to mount the host Docker socket or install Python into the controller as a shortcut.

---

### Task 1: Implement and locally verify the health service

**Files:**
- Create: `requirements.txt`
- Create: `app/__init__.py`
- Create: `app/main.py`
- Create: `tests/test_health.py`
- Modify: `docs/progress.md`
- Modify: `docs/learning-path.md`

**Interfaces:**
- Produces: `app.main.app`, a FastAPI application with `GET /health`.
- Consumes: FastAPI `TestClient` in `tests/test_health.py`.

- [ ] **Step 1: Add exact direct dependency pins to `requirements.txt`**

Set `fastapi==0.162.2`, `httpx==0.28.1`, and `pytest==9.2.0`, one per line.

- [ ] **Step 2: Create a local `.venv` and install the pinned dependencies**

Run: `python -m venv .venv` and `.\.venv\Scripts\python -m pip install -r requirements.txt`.

Expected: installation completes without changing tracked dependency files.

- [ ] **Step 3: Write the failing endpoint contract test**

Create `tests/test_health.py` with:

```python
from fastapi.testclient import TestClient

from app.main import app


def test_health_returns_ok() -> None:
    response = TestClient(app).get("/health")

    assert response.status_code == 200
    assert response.json() == {"status": "ok"}
```

- [ ] **Step 4: Run the test to verify it fails because `app.main` is absent**

Run: `.\.venv\Scripts\python -m pytest tests/test_health.py -v`

Expected: collection fails with `ModuleNotFoundError: No module named 'app'` or `app.main`.

- [ ] **Step 5: Implement `app.main.app` and `GET /health`**

Create the package marker and a FastAPI application whose health handler returns `{"status": "ok"}`.

- [ ] **Step 6: Run the endpoint test and full local suite**

Run: `.\.venv\Scripts\python -m pytest tests/test_health.py -v` then `.\.venv\Scripts\python -m pytest -v`.

Expected: both commands pass with one collected test.

- [ ] **Step 7: Record M3 local evidence and the Jenkins execution blocker**

Document actual test output and that the current Jenkins controller has no approved isolated Python build agent; do not modify `Jenkinsfile`.

- [ ] **Step 8: Commit the health service**

```powershell
git add requirements.txt app tests docs/progress.md docs/learning-path.md
git commit -m "feat: add FastAPI health endpoint"
```
