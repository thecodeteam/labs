# AWS EC2 with Docker Machine and REX-Ray with Docker 1.13 Managed Plugin

Use [Docker Machine](https://github.com/docker/machine) to deploy an AWS EC2
host that is installed and configured with the latest and stable Docker Engine.
Follow the directions to install REX-Ray using the Docker 1.13 Managed Plugin
System. This environment will use AWS EC2 along with EBS driver for REX-Ray
to allow stateful applications to persist data. This is currently considered
**EXPERIMENTAL** and not for production use.

---

1. Install [Docker for Mac](https://docs.docker.com/docker-for-mac/) or [Docker
for Windows](https://docs.docker.com/docker-for-windows/) so the laptop has the
local Docker client installed. Install [Docker Machine]
(https://github.com/docker/machine/releases) to deploy hosts. Check out the instructions on the `releases` tab

2. Deploy a machine into EC2 with Docker Machine

  ```
  $ docker-machine create --driver amazonec2 --amazonec2-access-key <my access
  key> --amazonec2-secret-key <my secret key> --amazonec2-vpc-id <my vpc>
  plugin-ebs
  ```

3. Set your local Docker client to point to your host using the following
commands

  ```
  $ docker-machine env plugin-ebs
  $ eval $(docker-machine env plugin-ebs)
  ```

4. Install the REX-Ray EBS Plugin from Docker Hub

  ```
  $ docker version
  ... verify 1.13+ is installed ...
  $ docker plugin install rexray/ebs REXRAY_LOGLEVEL=debug
  REXRAY_PREEMPT=true EBS_ACCESSKEY=<my access key> EBS_SECRETKEY=<my secret
  key>
  $ docker plugin ls
  ... verify rexray/ebs is enabled
  ```

Look at all the environment variables available at 
[cduchesne/rexray-plugin-test](https://github.com/cduchesne/rexray-plugin-test/tree/wip/initial_commit/ebs)

Create a new persistent application from the [Application Demo](https://github.com/codedellemc/labs#application-demo) section