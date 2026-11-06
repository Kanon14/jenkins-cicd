# Jenkins CI/CD Lab Foundation Pack — Design

## Intent

Establish a safe, minimal documentation and repository foundation for the Jenkins and Huawei Cloud CI/CD learning lab. The foundation preserves verified M0 evidence and makes the learning path and operating boundaries available in the repository before any pipeline, application, Docker, or cloud work begins.

## Goals

- Store the learning roadmap at `docs/learning-path.md`.
- Record the verified M0 environment audit and Docker recovery drill in `docs/progress.md`.
- Provide repository-local operating instructions in `AGENTS.md`.
- Prevent common local, secret-bearing, and generated files from being committed.
- Make the repository entry point clear in `README.md`.

## Non-goals

- No Jenkins, FastAPI, Dockerfile, image registry, Kubernetes, or cloud configuration.
- No external GitHub, cloud, or customer-system changes.
- No real credentials, tokens, kubeconfigs, or environment values.

## Constraints

- Work one milestone and checkpoint at a time.
- Keep M0 evidence factual and distinguish observed results from assumptions.
- Use Windows PowerShell-friendly instructions.
- Treat cloud deployment as an authorized sandbox-only later milestone.
- Keep sample configuration in `.env.example`; never commit `.env`.

## Selected approach

Create the complete approved foundation pack now: roadmap, operating instructions, progress journal, Git ignore rules, commented environment template, and a concise README. This is preferred over an evidence-only update because the user selected the foundation-pack option and the repository currently contains only an initial README.
