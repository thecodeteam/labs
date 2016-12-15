# Deploy VirtualBox with Docker Machine and Install REX-Ray

Use [Docker Machine](https://github.com/docker/machine) to deploy two VirtualBox
hosts that are installed and configured with the latest and stable Docker
Engine. Follow the directions to install REX-Ray using `curl | sh` and learn how
to write a properly formatted configuration file. This environment will use the
VirtualBox driver for REX-Ray to allow stateful applications to persist data.

---

1. Install [Docker for Mac](https://docs.docker.com/docker-for-mac/) or [Docker
for Windows](https://docs.docker.com/docker-for-windows/) so the laptop has the
local Docker client installed. Install [Docker Machine]
(https://github.com/docker/machine/releases) to deploy hosts. Check out the instructions on the `releases` tab

2. When using the VirtualBox SOAP API service for the first time, disable
authentication:

  ```
  $ VBoxManage setproperty websrvauthlibrary null
  ```

3. Start the VirtualBox SOAP API to accept API requests from the REX-Ray
service in a new terminal window:

  ```
  $ vboxwebsrv -H 0.0.0.0 -v
  ```

4. Open a second terminal window and follow the command prompts. **Change
`<username>`** with your actual username or specify a new path to place VirtualBox
Volumes. Terminal Window 2:

  ```
  $ docker-machine create -d virtualbox vbox01
  $ docker-machine create -d virtualbox vbox02
  $ docker-machine ssh vbox01 "curl -sSL https://dl.bintray.com/emccode/rexray/install | sh" && docker-machine ssh vbox02 "curl -sSL https://dl.bintray.com/emccode/rexray/install | sh"
  $ docker-machine ssh vbox01 "sudo tee -a /etc/rexray/config.yml << EOF
  libstorage:
    service: virtualbox
    integration:
      volume:
        operations:
          mount:
            preempt: true
  virtualbox:
    endpoint: http://192.168.99.1:18083
    volumePath: /Users/<username>/VirtualBox/Volumes
    controllerName: SATA"
  $ docker-machine ssh vbox02 "sudo tee -a /etc/rexray/config.yml << EOF
  libstorage:
    service: virtualbox
    integration:
      volume:
        operations:
          mount:
            preempt: true
  virtualbox:
    endpoint: http://192.168.99.1:18083
    volumePath: /Users/<username>/VirtualBox/Volumes
    controllerName: SATA"
  $ docker-machine ssh vbox01 "sudo rexray start" && docker-machine ssh vbox02 "sudo rexray start"
  ```

5. Set your local Docker client to point to your host using the following
commands

  ```
  $ docker-machine env vbox01
  $ eval $(docker-machine env vbox01)
  ```
