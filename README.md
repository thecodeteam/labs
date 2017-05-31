# {code} Labs

![{code} Labs](labs_header.jpg "{code} Labs")

**stateful and persistent applications in containers... that work!**

## Welcome to the {code} Labs! 

The {code} Labs were created to get quickly moving with our projects through
automation tools like Vagrant. Our focus is containers, plain and simple. Our
mission is creating the glue between your application and storage while making
it as seamless and easy as possible.

The {code} Labs is structured in two parts. Before testing an application
that requires persistence (like a database, analytic tools, content management
platforms, service discovery tools, and more) you need an environment. Choose
your environment and container orchestrator and then explore the options for
deploying different types of applications.

## Environment QuickStart

Quickly deploy an environment to begin application testing. Each 
has key pieces of technology that make them unique.

1. [Kubernetes, Docker Swarm, and Mesos with Marathon Using
Vagrant with ScaleIO as storage](https://github.com/codedellemc/labs/tree/master/setup-scaleio-vagrant)
    - This is the trifecta. Use the Vagrant file provided to create
    three (3) hosts with VirtualBox. Each host will have ScaleIO (a software that
    turns DAS storage into shared and scale-out block storage) installed and
    configured. Using a combination of defaults and environment variables,
    choose to install [REX-Ray](https://rexray.codedellemc.com/), Kubernetes,
    Docker Swarm, or Mesos with Marathon. This creates a fully configured
    environment ready to test stateful applications using the ScaleIO driver
    with multiple container orchestrators. If you've been looking for a simple
    way to test any container orchestrator, this is the lab for you.
2. [Deploy VirtualBox with Docker Machine and Install REX-Ray](https://github.com/codedellemc/labs/tree/master/setup-virtualbox-dockermachine)
    - Use [Docker Machine](https://github.com/docker/machine) to deploy a VirtualBox host that is installed and
    configured with the latest and stable Docker Engine. Follow the directions
    to install REX-Ray using `curl | sh` and learn how to write a properly
    formatted configuration file. This environment will use the VirtualBox
    driver for REX-Ray to allow stateful applications to persist data. This is
    the perfect lab for any beginner looking to get started using REX-Ray and
    container persistence on your local machine.
3. [3-Node ScaleIO + 3-Node Apache Mesos Cluster with Marathon on AWS](http://scaleio-framework.readthedocs.io/en/latest/user-guide/demo/)
    - Use an [AWS Cloudformtion](https://aws.amazon.com/cloudformation/)
    template to deploy two (2) clusters in AWS. The first cluster is a three (3)
    node ScaleIO environment which will be used by REX-Ray as the storage
    platform. A second cluster is a three (3) node Apache Mesos cluster fully
    configured with Marathon ready to accept requests for scheduling containers.
    Follow this with [Application Demo #3](https://github.com/codedellemc/labs/tree/master/demo-persistence-with-postgres-marathon-docker)
    - **GO ADVANCED**
        + Take it to the next level by exploring features of performing a custom
        ScaleIO configuration and installation. This process will take the
        existing Mesos Agent Nodes, add additional storage, and install all
        the SDS components to add more storage to the existing ScaleIO cluster
        based on your pool and domain configuration settings. Try it at the 
        [Custom ScaleIO Framework Deployment](https://github.com/codedellemc/labs/tree/master/setup-scaleio-aws-custom)
4. [Deploy a 3-Node Ceph Environment Using Vagrant](https://github.com/codedellemc/vagrant/tree/master/ceph)
    - This Vagrant environment uses VirtualBox and Virtual Media as storage for
    Ceph to be consumed by REX-Ray along with Docker as the container runtime. This can be used as a quick way to get started working with Ceph and
    REX-Ray. The hosts will automatically install and configure Docker Engine
    with REX-Ray to provide a fully configured environment ready to test
    stateful applications.
5. [Kubernetes with FlexREX on ScaleIO](https://github.com/codedellemc/labs/tree/master/demo-persistence-with-scaleio-kubernetes)
    - FlexREX is an implementation of a FlexVolume driver for Kubernetes that
    enables the entire library of REX-Ray/libStorage supported storage
    platforms. This lab will explore the centralized architecture where
    REX-Ray is deployed as a central controller within a Pod and FlexREX will be
    installed on the Kubernetes minion nodes.
6. [Use REX-Ray as a Docker Plugin with AWS EBS Volumes](https://github.com/codedellemc/labs/tree/master/setup-awsec2-docker-plugin)
    - Use [Docker Machine](https://github.com/docker/machine) to deploy an AWS
    EC2 host that is installed and configured with the latest and stable Docker
    Engine. Follow the directions to install REX-Ray using the Docker Managed
    Plugin System. This environment will use AWS EC2 along with EBS driver for
    REX-Ray to allow stateful applications to persist data.
7. [Install REX-Ray as a Plugin on Docker for AWS (Cloudformation)](https://github.com/codedellemc/labs/tree/master/setup-dockerforaws)
**EXPERIMENTAL** 
    - Bring persistent volume functionality to [Docker for AWS](https://docs.docker.com/docker-for-aws/) with REX-Ray. Customizing the Cloudformation template allows automated installations of REX-Ray to AWS AutoScaling Groups and access to EBS volumes.

## Application Demo

#### Standalone Docker Container

1. [Storage Persistence with Postgres using REX-Ray](https://github.com/codedellemc/labs/tree/master/demo-persistence-with-postgres-docker)
    - Learn how to read a Dockerfile to know which paths need persistent data.
    Manually deploy a Docker Postgres container and create a few tables to write
    data. Destroy the container and start a new container on a different host to
    see the data persist.

#### Kubernetes

1. [Create Secrets, Deployments, and Storage Classes to use Dynamic
Provisioning with Kubernetes](https://github.com/kubernetes/kubernetes/tree/master/examples/volumes/scaleio)
    - Use the native integration of ScaleIO with Kubernetes to do everything
    from creating secrets and using StorageClasses to provision Dynamic
    Volumes with Deployments.
2. [Storage Persistence with MySQL Persistent Volumes and Persistent Volume
Claims with Kubernetes Pods and REX-Ray (Flex-REX)](https://github.com/codedellemc/labs/tree/master/demo-persistence-with-scaleio-kubernetes#step-5-quick-and-simple-create-a-persistent-volume-and-utilize-it-in-a-pod)
    - Learn how to use MySQL with a Persistent Volume in a Kubernetes Pod. Go
    through all the steps for manually creating the volume, creating the
    persistent volume claim, and attaching it to the pod. This will also
    demonstrate how to do a migration of the MySQL Pod from one host to another.Learn more about [FlexREX](https://rexray.readthedocs.io/en/stable/user-guide/schedulers/#kubernetes) on the [{code} blog](https://blog.codedellemc.com/2017/01/24/flexrex-adds-storage-options-kubernetes/).

#### Docker Swarm

1. [Storage Persistence and Failover with Minecraft using REX-Ray and Docker
Swarm Mode](https://github.com/codedellemc/labs/tree/master/demo-persistence-with-minecraft-docker)
    - Take a set of nodes and cluster them together using [Docker Swarm Mode](https://docs.docker.com/engine/swarm/)
    which allows distributed computing, reconciliation of failed hosts, and
    extended networking functionality. Play a game of Minecraft to create an
    inventory of data to persist. Turn off the Docker service to watch Docker
    Swarm Mode along with REX-Ray redeploy the container on a new host to keep
    inventory intact.

#### Mesos

1. [Storage Persistence with Postgres using Mesos, Marathon, Docker, and
REX-Ray](https://github.com/codedellemc/labs/tree/master/demo-persistence-with-postgres-marathon-docker)
    - Use the supplied application spec for Marathon to deploy a Postgres
    service to Mesos. Use the restart button to redeploy the Postgres service on
    a new host and see the data persist.

## Video

1. [REX-Ray with Docker 1.13 Managed Plugins](https://www.youtube.com/watch?v=Vwtyer-oiq8&index=13&list=PLbssOJyyvHuWiBQAg9EFWH570timj2fxt)
2. [Finally! An Open Solution to Solve for Persistent Storage - @DockerCon
2017](https://www.youtube.com/watch?v=AXWfdu9f8sU&index=1&list=PLbssOJyyvHuWiBQAg9EFWH570timj2fxt)
3. [Make Stateful Applications Highly Available w/ Docker Swarm Mode (and Docker
plugins!) - @DockerCon 2017](https://www.youtube.com/watch?v=U8Dsi5V-XG0&index=3&list=PLbssOJyyvHuWiBQAg9EFWH570timj2fxt&t)
4. [REX-Ray and Modern Data Persistence - Q3 2016](https://www.youtube.com/watch?v=EnMsUKSsK0s&list=PLbssOJyyvHuWiBQAg9EFWH570timj2fxt&index=2)
5. [ScaleIO Framework for Apache Mesos](https://www.youtube.com/watch?v=tt6qhEkeVOQ&index=16&list=PLbssOJyyvHuWiBQAg9EFWH570timj2fxt&)

## Guidelines for Contributing a Demo

1. Create a new folder with the title of your demo
2. Add all relevant code and step-by-step instructions for completing the demo
3. Remember that these are quick "demos" and not "tutorials"
4. Create a README.md file for each demo to display on GitHub that lays out the instructions for completing your demo from start to finish.
5. Screenshots are encouraged. 

### Contribution Rules

Create a fork of the project into your own repository. Make all your necessary changes and create a pull request with a description on what was added or removed and details explaining the changes in lines of code. If approved, project owners will merge it.


### Support

Please file bugs and issues on the GitHub issues page for this project. This is to help keep track and document everything related to this repo. For general discussions and further support you can join the [{code} by Dell EMC Community slack channel](http://community.codedellemc.com/). The code and documentation are released with no warranties or SLAs and are intended to be supported through a community driven process.