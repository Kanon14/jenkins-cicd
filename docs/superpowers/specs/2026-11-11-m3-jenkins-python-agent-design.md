# M3 Jenkins Python Agent — Design

## Intent

Run the already passing FastAPI pytest check in Jenkins on a dedicated Python agent so the controller coordinates the build without executing project code itself.

## Goals

- Add a small Docker image for a dedicated Python 3.11 Jenkins inbound agent with Java and pinned project test dependencies.
- Connect the agent to the localhost-only Jenkins controller over a private Docker network, using WebSocket inbound connection over the existing HTTP endpoint.
- Register the agent in Jenkins with one executor and a dedicated label, then target the Test stage at that label.
- Keep the agent workspace separate from Jenkins home and avoid Docker socket mounts, privileged containers, cloud access, and public port exposure.

## Non-goals

- No Docker image builds, registry publishing, deployment, webhook, or cloud access.
- No change to the controller image or Jenkins home volume.
- No test execution on the built-in controller node.

## Selected approach

Use an inbound agent container on a dedicated private Docker network shared with Jenkins. Jenkins agents support WebSocket transport through the existing HTTP(S) endpoint, so the agent does not need an additional published TCP agent port. Store the agent secret in the Jenkins node configuration and provide it to the local container at runtime without committing it to Git or printing it in logs.

## Constraints and setup order

- Inspect the existing `jenkins-lab` network attachments before changing them; preserve its current localhost-only host binding.
- Build the Python agent from a pinned inbound-agent base and the repository's pinned dependencies.
- Create a dedicated permanent node labeled `python-test`, with one executor and an isolated remote root.
- Configure Jenkins' built-in node to zero executors only after the agent is online and the Pipeline can be scheduled to its label.
- Configure agent secret handling without putting its value in shell history, Git, or chat.
- User-run Docker commands may be needed because the current agent environment cannot access the Docker named pipe.

## Approval

The user selected the dedicated-agent approach on 2026-11-11. This short design is presented for review before implementation.
