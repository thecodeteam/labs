# Exploring Kubernetes with libStorage Persistent Volumes on AWS - multinode version

This VM will be used to build, deploy, and manage your Kubernetes cluster. It is possible to use a local VM, or physical machine, for this role. However, an AWS hosted VM was chosen for this lab because it minimizes variations, and will have a high bandwidth, low latency network path during the cluster deployment stage. 


sudo service docker start
sudo usermod -a -G docker ubuntu

Logout and reconnect to allow user of docker by the ubuntu user account.


The libstorage volume plugin for Kubernetes has been submitted as a PR, but it is not delivered in the current release (1.4). We will build Kubernetes from a source "fork" which is the basis of the libstorage PR.

make quick-release

Warning: If you use a host other than the recommended AWS large instance, build failures on a host with inadequate memory can result in errors not obviously related to memory. 

### Step 5: install AWS CLI and verify operation

The Kubernetes deployment script we will be using automatically creates and configures the AWS objects required for a hosted Kubernetes cluster. It required installation an configuration of the standard AWS CLI on the build host.

Login to the build host and perform these steps. On the configure step, submit:

  1. AWS Access Key
  2. AWS secret Key
  3. AWS region, e.g. us-west-2

```
curl -O https://bootstrap.pypa.io/get-pip.py
sudo python2.7 get-pip.py
sudo pip install awscli
aws help
aws configure
aws describe-instances
```

On the build host, in `/home/ubuntu/work/src/k8s.io/kubernetes`, create a cluster deployment script named bootstrap-cluster.sh with this content:

This script sets some environment variables, before calling the standard Kubernetes kube-up script.

Be sure to specify zone complete with with a subregion letter to be used in the deployment. 

```
#!/bin/bash
export KUBE_LOGGING_DESTINATION=elasticsearch
export KUBE_ENABLE_NODE_LOGGING=true
export KUBE_ENABLE_INSECURE_REGISTRY=true
export KUBERNETES_PROVIDER=aws
export KUBE_AWS_ZONE=us-west-2a
export MASTER_SIZE=t2.large
export NODE_SIZE=t2.small
export NUM_NODES=2
# optional: specifiy  an existing SSH key instead of generating
# export AWS_SSH_KEY=$HOME/.ssh/kube_aws_rsa

function cluster_up {
        ./cluster/kube-up.sh
}

function cluster_down {
       ./cluster/kube-down.sh
}

if [[ "${1:-up}" == "down" ]]; then
        cluster_down
else
        cluster_up
fi
```

Use the script to deploy the cluster. After deployment, exercise kubectl to verify the cluster is operational. Note: some the commands are expected to return "No resources found." since this is a new cluster.


ssh into the kubenetes master instance, using the generated SSH key, and test operation with these commands.

If you generated the default ssh key for access to the Kubernetes cluster, ssh from linux would look somewhat like this:

`ssh -i ~/.ssh/kube_aws_rsa admin@ec2-203-0-113-4.us-west-2.compute.amazonaws.com`

```
kubectl cluster-info dump
```

### Step 7: Install REX-Ray on the Kubernetes master node

#### Install REX-Ray.

Continue using the ssh session to the kubernetes master instance from the previous step.

```

As sudo, create the file /etc/rexray/config.yml with this content using nano, vi or other editor, substituting your AWS details (two keys and region). :
  loglevel: warn
  modules:
    default-docker:
      disabled: true
libstorage:
  logging:
    level: warn
  host: tcp://172.20.0.9:7979
  embedded: true
  service:
    ebs
  server:
    endpoints:
      public:
        address: tcp://127.0.0.1:7979
      public2:
        address: tcp://172.20.0.9:7979
    services:
      ebs:
        driver: ebs

Start REX-Ray as a service.

`sudo rexray service start`

Examine the log in /var/log/rexray/rexray.log to verify that it started without errors.


`rexray volume create mysql-1 --size=1 -h tcp://127.0.0.1:7979`

#### Create a pod containing a stateful application with will use the volume,

Continue using the ssh session to the kubernetes master. In the home directory, create the file pod.yaml with this content. Observe that using a volume mount in this pod definition *decouples the database's data from the container*.:

Use the file to create the pod, and verify.

```


Show the log, which should indicate that MySQL is ready for connections:

`kubectl  logs mysql`

Persistent volumes are intended for "network volumes" like GCE persistent disks, and AWS elastic block stores, but can also represent volumes from "overlay" storage providers like ScaleIO. A Persistent Volume (PV) in Kubernetes represents a real piece of underlying storage capacity in the infrastructure. 

A user (application) sees and consumes claims, which hide storage implementation details. This allows an application's configuration to be portable across platforms and providers.



#### Create a Kubernetes PersistentVolume and PersistentVolumeClaim

kind: Service
metadata:
  labels:
    name: postgres-libstorage
  name: postgres-libstorage
spec:
  type: NodePort
  ports:
    - port: 5432
      targetPort: postgres-client
      nodePort: 30032
  selector:
    app: postgres-libstorage
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: postgres-libstorage
  labels:
    app: postgres-libstorage
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: postgres-libstorage
    spec:
      containers:
      - image: postgres
        name: postgres
        env:
        - name: POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        - name: POSTGRES_PASSWORD
          # CHANGE THIS PASSWORD!
          value: mysecretpassword
        - name: PGDATA
          value: /var/lib/postgresql/data/pgdata
        ports:
        - name: postgres-client
          containerPort: 5432
        volumeMounts:
        - name: pg-data
          mountPath: /var/lib/postgresql/data
      volumes:
        - name: pg-data
          persistentVolumeClaim:
            claimName: pg-data-claim

Now use the file to create the service and deployment.


A deploment generates a unique name when it creates a pod. This will lookup the name by label and place it in an environment variable, for convenient reuse later.

```
export PGPOD=$(kubectl get pods -l app=postgres-libstorage --no-headers | awk '{print $1}')
```

Examine logs to vefiy postgres started. Using an interactive session into the pod's container, create a new database named libstoragedemo.

```
kubectl logs $PGPOD
kubectl exec -ti $PGPOD -- bash
su - postgres
psql
CREATE DATABASE libstoragedemo;
\l
\q
exit
exit
```

### Step 10: Engage in Node maintenance to demonstate migration of a statefull pod to a different cluster host.

#### Determine the hosting node

`kubectl get nodes`

should return a list of all nodes like this:

```
NAME                                         STATUS    AGE
ip-172-20-0-239.us-west-2.compute.internal   Ready     15h
ip-172-20-0-240.us-west-2.compute.internal   Ready     15h
```

We can look up the Postgres pod using its label and determine the pod's host node. We will do this and save the node in an environment variable.

Determine which of the hosts is running the Postgres database pod now:

```
export DB_NODE=$(kubectl describe pod -l app=postgres-libstorage | grep Node: | awk -F'[ \t//]+' '{print $2}')
echo $DB_NODE
```

#### Prepare the hosting node for maintenance

We will simulate a planned node outage. (In this example we won't evacuate all pods, only the Postgres pod, but this will be adequate to demonstate stateful application failover.) 

Tell the Kubernetes scheduler to cease directing pods to the node hosting Postgres.

`kubectl cordon $DB_NODE`

Delete the existing pod. The Deployment will automatically bring up a new replacement pod. It will be deployed on a different node because we cordoned off the existing node.

`kubectl delete pod -l app=postgres-libstorage`

Verify that the pod has migrated succesfully.

`kubectl describe pods -l app=postgres-libstorage`

It may take a brief time before the replacement pod shows a "running" status. The AWS console can be used to verify that the volume has been remounted to the alternate minion node.

Use an interactive session to verify that the "libstoragedemo" database we created is present:

```
export PGPOD=$(kubectl get pods -l app=postgres-libstorage --no-headers | awk '{print $1}')
kubectl logs $PGPOD
kubectl exec -ti $PGPOD -- bash
su - postgres
psql
\l
\q
exit
exit
```

Finally, tell the scheduler to return the node to service 

`kubectl uncordon $DB_NODE`

Dynamic provisioning is a mechanism that allows storage volumes to be created on demand, rather than pre-provisioned by an administrator. This is a Beta feature of Kubernetes 1.4.

#### Create a storage class:

Use the file to create the storage class.


Create a a persistent volume claim. This step will result in creation of an unattached AWS EBS volume, with a name in the form of "kubernetes-dynamic-pvc-GUID" .


Save the volume's ID in an environment variable that can be used to match against the ID in the AWS console.

`export EBS_DYN_VOL_ID=kubernetes-dynamic-$(kubectl describe pvc pvc-0002 | grep Volume: | awk '{print $2}')`

  labels:
    app: webserv-libstorage

Use the file to create the pod.

`kubectl create -f pod-sc-pvc.yaml`

Verify that the pod has been created and has entered the running state
`kubectl describe pods -l app=webserv-libstorage`

The AWS console can be used to verify that a volume has been created, and attached to the node running this pod.

### Step 12: Cleanup 

Delete the MySQL server. Deleting the mount automatically unmounts the volume. Note that by design, volumes persist in AWS until explicitly removed. Volumes can be reused in another pod - even in a different instance of a Kubernetes cluster. A volume could even be migrated to a different container scheduler platform such as Mesos.

Delete MySQL server pod.

`kubectl delete pod mysql`

Delete the Postgres server service and deployement, which will automatically delete the pod.

```
kubectl delete deployment -l app=postgres-libstorage
kubectl delete service postgres-libstorage
```

Delete the webserver pod. 

`kubectl delete pod -l app=webserv-libstorage`

Delete the persistent volume claims. But before we delete the dynamically provisioned claim we need to save the AWS EBS volume's ID so we can delete the EBS volume later.

```
export EBS_DYN_VOL_ID=kubernetes-dynamic-$(kubectl describe pvc pvc-0002 | grep Volume: | awk '{print $2}')
kubectl delete pvc pvc-0002 pg-data-claim
```

Delete the storage class.

`kubectl delete storageclass bronze`

Delete the persistent volume.

`kubectl delete pv small-pv-pool-01`

Delete the AWS EBS volumes.

```
rexray volume rm $EBS_DYN_VOL_ID mysql-1 admin-managed-1g-01 -h tcp://127.0.0.1:7979
```

To permanently delete the Kubernetes cluster VM instances in AWS, switch back to the build VM instance and run:

`./bootstrap-cluster.sh down` 
