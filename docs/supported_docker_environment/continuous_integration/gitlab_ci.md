# GitLab CI

In order to use Testcontainers in a Gitlab CI pipeline, you need to run the job as a Docker container (see [Patterns for running inside Docker](dind_patterns.md)).
So edit your `.gitlab-ci.yml` to include the [Docker-In-Docker service](https://docs.gitlab.com/ee/ci/docker/using_docker_build.html#use-docker-in-docker-workflow-with-docker-executor) (`docker:dind`) and set the `DOCKER_HOST` variable to `tcp://docker:2375` and `DOCKER_TLS_CERTDIR` to empty string. 

Caveat: Current docker releases (verified for 20.10.9) intentionally delay the startup, if the docker api is bound to a network address but not TLS protected. To avoid this delay, the docker process needs to be started with the argument `--tls=false`.  Otherwise jobs which access the docker api at the very beginning might fail.

Here is a sample `.gitlab-ci.yml` that executes test with gradle:

```yml
# DinD service is required for Testcontainers
services:
  - name: docker:dind
    # explicitly disable tls to avoid docker startup interruption
    command: ["--tls=false"]

variables:
  # Instruct Testcontainers to use the daemon of DinD.
  DOCKER_HOST: "tcp://docker:2375"
  # Instruct Docker not to start over TLS.
  DOCKER_TLS_CERTDIR: ""
  # Improve performance with overlayfs.
  DOCKER_DRIVER: overlay2

test:
 image: gradle:5.0
 stage: test
 script: ./gradlew test
```
