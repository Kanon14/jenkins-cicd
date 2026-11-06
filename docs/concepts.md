# Concepts and Reference Notes

## M1 — Jenkins Pipeline and Docker foundations

### Sources reviewed on 2026-11-06

- [Jenkins Pipeline documentation](https://www.jenkins.io/doc/book/pipeline/)
- [Jenkins Docker installation documentation](https://www.jenkins.io/doc/book/installing/docker/)

### Concepts learned

- A Jenkins Pipeline is a plugin-based, automated model of the delivery process. It can define build, test, and later deployment stages as code in a version-controlled `Jenkinsfile`.
- The lab will use Declarative Pipeline syntax first. Its `pipeline`, `agent`, `stages`, and `steps` structure makes the execution model explicit before introducing Scripted Pipeline.
- A controller coordinates Jenkins work; an agent/node executes the pipeline in a workspace. A stage is a visible logical grouping such as Build or Test, while a step is an individual action in that stage.
- Pipeline can pause for human input, which supports the later simulated UAT and PROD approval gates. Technical deployment success remains distinct from business approval.
- On Windows, Jenkins supplies a Linux container image. Docker Desktop must therefore be configured for Linux containers, not Windows containers.
- Prefer an explicit LTS-based Jenkins image reference over an unreviewed `latest` tag. The standard Jenkins image does not include the Docker CLI, and Docker-in-Docker uses privileged access; neither concern is being configured in this checkpoint.

### Questions to carry into setup

- Is Docker Desktop currently using Linux containers and able to run the Jenkins image?
- Does the host have sufficient disk and memory for a local Jenkins controller and its persistent data volume?
- Which minimal plugins are needed for the first local Pipeline, without adding Docker build privileges to the controller?

### Deliberate non-actions

No Jenkins image was pulled, no container/network/volume was created, no port was exposed, and no credentials or cloud resources were used during this study checkpoint.
