# Jenkins CI/CD Learning Lab

Personal, sandbox-only learning repository for building a secure, reproducible Jenkins CI/CD workflow for a small FastAPI service. The later lab will simulate immutable image promotion through VSIT → SIT → UAT → PROD, then map the pattern to Huawei Cloud SWR and CCE when authorized sandbox access is available.

## Current status

Milestone M3 (FastAPI CI on a dedicated Jenkins agent) is in final verification. A `main` build and the isolated failure drill passed their expected outcomes, and the Jenkins controller now has zero built-in executors. Push the latest M3 follow-up and confirm the offline dependency preflight in one more `main` build before beginning M4; see the progress journal for evidence.

## Start here

- [Learning path](docs/learning-path.md)
- [Progress journal](docs/progress.md)

## Safety boundary

Do not use this repository to access customer, employer, production, or Huawei Cloud infrastructure without explicit authorization. Never commit credentials, tokens, kubeconfigs, or real environment files.
