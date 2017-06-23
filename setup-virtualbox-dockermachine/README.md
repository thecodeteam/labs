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
    volumePath: /Users/$USER/VirtualBox/Volumes
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
    volumePath: /Users/$USER/VirtualBox/Volumes
    controllerName: SATA"
  $ docker-machine ssh vbox01 "sudo rexray start" && docker-machine ssh vbox02 "sudo rexray start"
  ```

5. Set your local Docker client to point to your host using the following
commands

  ```
  $ docker-machine env vbox01
  $ eval $(docker-machine env vbox01)
  ```


---

## Create a Swarm Service Using Quick Commands

1. Create 3 Machines
```
$ docker-machine create -d virtualbox vbox01 && docker-machine create -d
virtualbox vbox02 && docker-machine create -d virtualbox vbox03
```

2. Install REX-Ray As a Server Service
```
$ docker-machine ssh vbox01 "curl -sSL
https://dl.bintray.com/emccode/rexray/install | sh" && docker-machine ssh vbox02 "curl -sSL https://dl.bintray.com/emccode/rexray/install | sh" && docker-machine ssh vbox03 "curl -sSL https://dl.bintray.com/emccode/rexray/install | sh"
```

3. Add the REX-Ray Configuration Files
```
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
  volumePath: /Users/$USER/VirtualBox/Volumes
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
  volumePath: /Users/$USER/VirtualBox/Volumes
  controllerName: SATA"

$ docker-machine ssh vbox03 "sudo tee -a /etc/rexray/config.yml << EOF
libstorage:
  service: virtualbox
  integration:
    volume:
      operations:
        mount:
          preempt: true
virtualbox:
  endpoint: http://192.168.99.1:18083
  volumePath: /Users/$USER/VirtualBox/Volumes
  controllerName: SATA"
```

4. Start the REX-Ray Service
```
$ docker-machine ssh vbox01 "sudo rexray start" && docker-machine ssh vbox02
"sudo rexray start" && docker-machine ssh vbox03 "sudo rexray start"
```

5. Get the `vbox01` IP address
```
$ VBOX01IP=$(docker-machine ssh vbox01 "ifconfig | sed -En
's/127.0.0.1//;s/.*inet (addr:)?(([0-9]*\.){3}[0-9]*).*/\2/p' | grep 99")
```

6. Initialize the Swarm Cluster
```
$ docker-machine ssh vbox01 "docker swarm init --advertise-addr $VBOX01IP
--listen-addr $VBOX01IP"
```

7. Get the Swarm Token to add Worker nodes
```
$ VBOXSWARMTOKEN=$(docker-machine ssh vbox01 "docker swarm join-token -q
worker")
```

8. Add `vbox2` and `vbox3` as worker nodes to the swarm
```
$ docker-machine ssh vbox02 "docker swarm join --token $VBOXSWARMTOKEN
$VBOX01IP:2377" && docker-machine ssh vbox03 "docker swarm join --token $VBOXSWARMTOKEN $VBOX01IP:2377"
```

9. Optionally pre-download images on nodes
```
$ docker-machine ssh vbox01 "docker pull redis" && docker-machine ssh vbox02
"docker pull redis && docker pull postgres"&&
docker-machine ssh vbox03 "docker pull redis && docker pull postgres"
```

10. Create Some Services and Volumes
```
$ docker service create --replicas 1 --name red --mount type=volume,target=/data
--constraint 'node.role == worker' redis

$ docker service create --replicas 3 --name red redis

$ docker volume create -d rexray --name pgdata --opt=size=8

$ docker service create --replicas 1 --name pg -e
POSTGRES_PASSWORD=mysecretpassword \
--mount type=volume,target=/var/lib/postgresql/data,source=pgdata,volume-driver=rexray \
--constraint 'node.role == worker' postgres
```

11. Clean up.
```
$ docker-machine rm vbox01 && docker-machine rm vbox02 && docker-machine rm
vbox03
```
