# Kubernetes, Docker Swarm, and Mesos with Marathon Using Vagrant with ScaleIO

You've been looking for a simple way to test any container orchestrator and
now you finally made it to the right place!

The Vagrant file provided will use VirtualBox to create three (3) hosts. Each
host will have ScaleIO (a software that turns DAS storage into shared and
scale-out block storage) installed and configured. But before getting started,
make sure to read the instructions for the type of environment you want to
deploy.

The following environment variables can be used to install and configure
different components. Each one is given a further description below in their
respective sections.

| Env Variable        | Use           | On By Default?  |
| ------------- |:-------------:| :-----:|
| `SCALEIO_RAM`  | Set the RAM size for node01 and node02 machines | 1024 |
| `SCALEIO_DOCKER_INSTALL` | Install the latest Docker CE release | True |
| `SCALEIO_REXRAY_INSTALL` | Install the latest REX-Ray release   | True |
| `SCALEIO_SWARM_INSTALL`  | Configure Docker SwarmKit for the cluster | False |
| `SCALEIO_MESOS_INSTALL` | Install/Configure Apache Mesos with Marathon|False|
| `SCALEIO_K8S_INSTALL` | Install/Configure Kubernetes for the cluster|False|

  - [Cloning and Use](#cloning-and-use)
  - [Docker and REX-Ray](#docker-and-rexray)
  - [Docker Swarm](#docker-swarm)
  - [Apache Mesos and Marathon](#apache-mesos-and-marathon)
  - [Kubernetes](#kubernetes)
  - [Helpful Tips](#helpful-tips)

### Cloning and Use

Are you new to Vagrant? No worries. It's simple. The only step you need to
complete is installing [Vagrant](https://www.vagrantup.com/docs/installation/)
and [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git).
After you've installed Vagrant and Git, clone the repo.

```
$ git clone https://github.com/codedellemc/vagrant
$ cd vagrant/scaleio/
```

When it comes time to use the machines you can open 3 terminal sessions to
master, node01, and node02. In this lab, master functions as the API gateway,
the Master/Manager/Controller role for container orchestrators, and REX-Ray
utilizes it to access the storage platform. node01 and node02 are configured as
worker nodes with no management functionality.

```
...window 1...
$ vagrant ssh master
...window 2...
$ vagrant ssh node01
...window 3...
$ vagrant ssh node02
```

If you're interested in using the ScaleIO GUI, it is automatically extracted and
put into the `vagrant/scaleio/gui` directory. Run `./run.sh` to start the GUI.
Connect to your instance at 192.168.50.11 with the credentials `admin` and
`Scaleio123`.

![alt text](https://raw.githubusercontent.com/codedellemc/vagrant/master/scaleio/docs/images/scaleio-docker-rexray.png)

### Docker and REX-Ray

By default, the latest stable versions of Docker and REX-Ray are installed on
all three nodes but can be overridden using the Environment Variables below.
Each ScaleIO node will configure REX-Ray to use libStorage to manage ScaleIO
volumes for persistent applications in containers. Use `export` with environment
variables.

 - `SCALEIO_DOCKER_INSTALL` - Default is `true`. Set to `false` to not
 automatically install and opt to manually install later
 - `SCALEIO_REXRAY_INSTALL` - Default is `true`. Set to `false` to not
 automatically install and opt to manually install later.

To run a container with persistent data stored on ScaleIO, from any of the cluster nodes you can run the following examples:

Run Busybox with a volume mounted at `/data`:
```
docker -it --volume-driver=rexray -v data:/data busybox
```

Run Redis with a volume mounted at `/data`:
```
docker run -d --volume-driver=rexray -v redis-data:/data redis
```

Run MySQL with a volume mounted at `/var/lib/mysql`:
````
docker run -d --volume-driver=rexray -v mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw mysql
````

#### Docker High Availability

Since the nodes all have access to the ScaleIO environment, fail over
capabilities
with REX-Ray are available by starting a container with a persistent volume on
one host, and starting it again on another. Docker's integration with REX-Ray
will forcefully unmount the volume from the original host and mount the volume
to the new host automatically. This creates high availability for your
application to continue working as intended.

### Docker Swarm

[Docker Swarm](https://docs.docker.com/engine/swarm/#feature-highlights)
natively manages a cluster of Docker Engines called a swarm. Use the Docker CLI
to create a swarm, deploy application services to a swarm, and manage swarm
behavior. Use `export` with environment variables.

 - `SCALEIO_SWARM_INSTALL` - Default is `false`. Set to `true` to 
 automatically configure the Docker Swarm cluster. Docker and REX-Ray will
 automatically be installed during this process.
   - `export SCALEIO_SWARM_INSTALL=true`

The `docker service` command is used to create a service that is scheduled on
nodes and can be rescheduled on a node failure. As a quick demonstration, go to
master and run a postgres service and pin it to the worker nodes:

```
$ docker service create --replicas 1 --name pg -e POSTGRES_PASSWORD=mysecretpassword \
--mount type=volume,target=/var/lib/postgresql/data,source=postgres,volume-driver=rexray \
--constraint 'node.role == worker' postgres
```

Use `docker service ps pg` to see which ScaleIO node it was scheduled on. Go to
that node and stop the docker service with `sudo systemctl stop docker`. On MDM1, a `docker service ps pg` will show the container being rescheduled on a different worker.

If it doesn't work, restart the service on the node, go to the other and download the image using `docker pull postgres` and start again.

### Apache Mesos with Marathon

[Apache Mesos](http://mesos.apache.org/) abstracts CPU, memory, storage, and
other compute resources enabling fault-tolerant and elastic distributed systems to easily be built and run effectively. [Marathon by Mesosphere](https://github.com/mesosphere/marathon) is a production-proven Apache Mesos framework for container orchestration.

 - `SCALEIO_MESOS_INSTALL` - Default is `false`. Set to `true` to 
 automatically install and configure the Mesos cluster and Marathon framework.
 Docker and REX-Ray will automatically be installed during this process.
   - `export SCALEIO_MESOS_INSTALL=true`

Mesos and Marathon Web GUIs will be accessible from `http://192.168.50.11:5050`
and `http://192.168.50.11:8080` after installation is complete. 

For Instructions for deploying containers, visit the [{code} Labs Application
Demo](https://github.com/codedellemc/labs) section and try [Storage Persistence
with
Postgres using Mesos, Marathon, Docker, and REX-Ray](https://github.com/codedellemc/labs/tree/master/demo-persistence-with-postgres-marathon-docker).

```
$ curl -O https://raw.githubusercontent.com/codedellemc/labs/master/demo-persistence-with-postgres-marathon-docker/postgres.json
$ curl -k -XPOST -d @postgres.json -H "Content-Type: application/json" http://192.168.50.12:8080/v2/apps
```

### Kubernetes

[Kubernetes](https://kubernetes.io/) is an open-source system for automating
deployment, scaling, and management of containerized applications. ScaleIO has a
native [Kubernetes](https://kubernetes.io/) integration. This means it doesn't
rely on a tool like REX-Ray to function. Functionality such as Dynamic
Provisioning with Persistent Volume Claims, Deployments, and StatefulSets work
out of the box.

 - `SCALEIO_K8S_INSTALL` - Default is `false`. Set to `true` to 
 automatically install and configure the Kubernetes.
 Docker will automatically be installed during this process.
   - `export SCALEIO_K8S_INSTALL=true`

On `master` there is a folder called `k8s_examples` that can be used to create
the
secret, a standard pod, deployment, storage class, and more. Follow the 
[offical ScaleIO Kuberentes Documentation](https://github.com/kubernetes/kubernetes/tree/master/examples/volumes/scaleio) and learn how to use each component.

##### Using a Kubernetes Deployment

REX-Ray is installed on all nodes for ease of volume management. If storage
classes and dynamic provisioning is not used, Kubernetes expects the volumes to be available. REX-Ray is an easy tool to quickly create the volumes. 

1. Create a new volume that is a dependency of `deployment.yaml`:
  - `sudo rexray create pgdata-k8-01 --size=16`
2. Create the [Kubernetes Secret](https://kubernetes.io/docs/concepts/configuration/secret/). The `secret.yaml` contains all the information
to communicate with the ScaleIO Gateway and sensitive information is a base-64
encoded:
  - `kubectl create -f secret.yaml`
3. Create the [Kubernetes Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) which will deploy a [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod/) with a postgres container.- `kubectl create -f deployment.yaml`
4. Get the pod list:
  - `kubectl get pods`
5. View the pod details
  - `kubectl describe pod <pod id from last step>`
6. A Deployment will maintain state to make sure this container is always
running. Kill the container and watch
  - `kubectl delete pod <pod id from last step>`
7. Get the pod list:
  - `kubectl get pods`
8. View the pod details and notice that the container is restarting on a
different host but is performing all the functions necessary to detach it
from the previous host and move it to the new worker. 
  - `kubectl describe pod <pod id from last step>`
  - **NOTE:** you will see an error that says the volume was already mapped.
  This is a false positive because the container and volume follow along. Try it
  yourself by adding data to the postgres container following the steps at [Storage Persistence with Postgres using REX-Ray](https://github.com/codedellemc/labs/tree/master/demo-persistence-with-postgres-docker)


### Helpful Tips

Depending on the docker images being used, RAM may need to be increased on the
worker nodes. Usually 1.5GB or 2GB for MDM2 and TB will suffice. MDM1 will
always be configured with 3GB.
 - `SCALEIO_RAM` - Default is `1024`.
   - `export SCALEIO_RAM=1536`
