# Jenkins CI/CD Learning Lab

Personal, sandbox-only learning repository for building a secure, reproducible Jenkins CI/CD workflow for a small FastAPI service. The later lab will simulate immutable image promotion through VSIT → SIT → UAT → PROD, then map the pattern to Huawei Cloud SWR and CCE when authorized sandbox access is available.

## Current status

Milestone M3 (FastAPI CI on a dedicated Jenkins agent) is complete. M4 is in progress: the FastAPI app now builds and runs locally in a non-root container, bound to localhost, and `/health` returns `{"status":"ok"}`. The next M4 checkpoint is to review a safe strategy for building the image from Jenkins; see the progress journal for evidence.

## Start here

- [Learning path](docs/learning-path.md)
- [Progress journal](docs/progress.md)

## Safety boundary

Do not use this repository to access customer, employer, production, or Huawei Cloud infrastructure without explicit authorization. Never commit credentials, tokens, kubeconfigs, or real environment files.
