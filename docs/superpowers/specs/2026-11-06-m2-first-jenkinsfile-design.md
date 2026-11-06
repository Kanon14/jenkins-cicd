# M2 First Jenkinsfile — Design

## Intent

Create the lab's first version-controlled Jenkins Pipeline so the user can learn Declarative Pipeline structure, SCM checkout, stage-level logs, and `post` behavior in the existing local Jenkins controller.

## Goals

- Add one root-level `Jenkinsfile` using Declarative syntax.
- Define explicit `Checkout`, `Validate`, and `Test` stages.
- Use the existing local Linux-based Jenkins controller through `agent any`.
- Add success, failure, and cleanup `post` messages.
- Configure a Jenkins Pipeline job to read the file from the existing GitHub repository after the file is committed.

## Non-goals

- No FastAPI application, test framework, Docker build, registry push, cloud access, credentials, webhooks, or production deployment.
- No Scripted Pipeline or shared libraries.
- No repository secrets or Jenkins credentials.

## Selected approach

Use a Declarative Pipeline from SCM. `Checkout` explicitly performs `checkout scm`; `Validate` verifies that the checked-out workspace contains `Jenkinsfile`; `Test` is an intentionally harmless placeholder shell command. The pipeline disables the implicit default checkout to avoid checking out the repository twice.

## Constraints

- Run only local shell commands inside the Jenkins container.
- Keep the first successful run and one intentional failure as separate exercises.
- Do not print or store credentials.
- The user performs browser configuration and reports only non-secret results.

## Approval

The user selected the SCM-based Declarative Pipeline approach on 2026-11-06. This design requires explicit approval before implementation.
