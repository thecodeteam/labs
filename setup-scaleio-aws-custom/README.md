# Custom ScaleIO Framework Deployment

This tutorial will allow the resources in Mesos Agent compute nodes to be
provisioned and included into a ScaleIO cluster. This is accomplished by
leveraging [Mesos Agent attributes](http://mesos.apache.org/documentation/latest/attributes-resources/) to carve up resources and imprint the configuration that is desired.

These directions allows a ScaleIO user the freedom and flexibility to map out a
configuration without getting into the details of installing RPMs or DEBs. If
the deployment of a Mesos cluster is using configuration management tools such
as Puppet, it can be augmented to define Mesos Agent attributes so you can turn
over the procedural steps to ScaleIO deployment to the Framework. Since this is
a Mesos Framework, its interface is a simple, stable and well-known interface
that insulates users from any ScaleIO changes in packaging, APIs, etc.

### Deploy the environment via Amazon CloudFormation

1. Follow the directions from [3-Node ScaleIO + 3-Node Apache Mesos Cluster with
Marathon on AWS](http://scaleio-framework.readthedocs.io/en/latest/user-guide/demo/) but stop at  **Launch Framework**. Remember, this only works in **US-West-1 (aka N.California) region**.

2. Once the CloudFormation template has successfully been deployed and the
Primary MDM has been identified, begin setting configurations

    ```
    $ scli --login --username admin --password F00barbaz
    $ scli --rename_storage_pool --protection_domain_name default --storage_pool_name default --new_name mypool
    $ scli --rename_protection_domain --protection_domain_name default
    --new_name mydomain
    ```

3. Verify everything is working and no volumes are available with `scli
--query_all_volumes` and `scli --query_all_sds`
    ```
    [root@ip-10-0-0-12 ec2-user]# scli --query_all_volumes
    Query-all-volumes returned 0 volumes
    Protection Domain 5a20702900000000 Name: mydomain
    Storage Pool 96eb24f700000000 Name: mypool
     <No volumes defined>

    [root@ip-10-0-0-12 ec2-user]# scli --query_all_sds
    Query-all-SDS returned 3 SDS nodes.

    Protection Domain 5a20702900000000 Name: mydomain
    SDS ID: 454581a100000002 Name: SDS_[10.0.0.11] State: Connected, Joined IP    10.0.0.11 Port: 7072 Version: 2.0.5014
    SDS ID: 454581a000000001 Name: SDS_[10.0.0.12] State: Connected, Joined IP    10.0.0.12 Port: 7072 Version: 2.0.5014
    SDS ID: 4545819f00000000 Name: SDS_[10.0.0.13] State: Connected, Joined IP    10.0.0.13 Port: 7072 Version: 2.0.5014
    ```


### Attach Storage to Mesos Agents to be used with ScaleIO
1. Within the AWS Console, Go to `Elastic Block Store -> Volumes` and create
a 180GB disk for each Mesos Agent. Attach the 180GB disk to `/dev/xvdf` of each
Mesos Agent node (in the AWS console this is `/dev/sdf`). Verify it's in the
same Availability Zone as the deployed configuration. Give the volumes a name as
well for readability.

### Configure Mesos Agent Attributes and Launch Framework
1. The next steps will set configuration parameters for creating a Protection
Domain called `mydomain` and a Storage Pool called `mypool`. Log into **every** Mesos Agent Node and run the following commands:

    ```
    sudo su
    cd /etc/mesos-slave/
    mkdir attributes
    cd attributes
    echo mydomain | tee scaleio-sds-domains
    echo mypool | tee scaleio-sds-mydomain
    echo /dev/xvdf | tee scaleio-sds-mypool
    echo mydomain | tee scaleio-sdc-domains
    echo mypool | tee scaleio-sdc-mydomain
    rm -rf /var/lib/mesos/meta/slaves/latest
    service mesos-slave restart
    ```

2. Go back to the directions for a [3-Node ScaleIO + 3-Node Apache Mesos Cluster
with Marathon on AWS]
(http://scaleio-framework.readthedocs.io/en/latest/user-guide/demo/) and continue with directions to **Launch Framework**.

    ```
    curl -k -XPOST -d @scaleio.json -H "Content-Type: application/json" [MESOS MASTER PUBLIC DNS/IP ADDRESS]:8080/v2/apps
    ```

3. Track the install progress using the scheduler link in the Marathon
UI. Click the link in the scheduler task. Each agent will reboot when install is
finished. 

### Create New Volumes and Deploy Applications
1. Create a volume from any Mesos Agent node using REX-Ray or
deploy a Marathon tasking using an external volume.

    ```
    $ sudo rexray volume create --size=16 postgres
    ```

2. Query ScaleIO with `scli --query_all_sds` to see there are now additional
nodes a part of the ScaleIO cluster. Use `scli --query_all_volumes` to see the
newly created volume:

    ```
    [root@ip-10-0-0-12 tmp]# scli --query_all_sds
    Query-all-SDS returned 5 SDS nodes.

    Protection Domain 507dbb5100000000 Name: mydomain
    SDS ID: d3e831d600000004 Name: sds_10.0.0.22 State: Connected, Joined IP:
    10.0.0.22 Port: 7072 Version: 2.0.10000
    SDS ID: d3e831d500000003 Name: sds_10.0.0.23 State: Connected, Joined IP: 10.0.0.23 Port: 7072 Version: 2.0.10000
    SDS ID: d3e80ac800000002 Name: sds_[10.0.0.13] State: Connected, Joined IP: 10.0.0.13 Port: 7072 Version: 2.0.10000
    SDS ID: d3e80ac700000001 Name: sds_[10.0.0.12] State: Connected, Joined IP: 10.0.0.12 Port: 7072 Version: 2.0.10000
    SDS ID: d3e80ac600000000 Name: sds_[10.0.0.11] State: Connected, Joined IP: 10.0.0.11 Port: 7072 Version: 2.0.10000
    [root@ip-10-0-0-12 tmp]# scli --query_all_volumes
    Query-all-volumes returned 1 volumes
    Protection Domain 507dbb5100000000 Name: mydomain
    Storage Pool bcb20fc300000000 Name: mypool
     Volume ID: 806e108500000000 Name: postgres Size: 16.0 GB (16384 MB)Unmapped Thin-provisioned
    ```

3. Test out running a Postgres image with [Storage Persistence with Postgres using Mesos, Marathon, Docker, and REX-Ray](https://github.com/codedellemc/labs/tree/master/demo-persistence-with-postgres-marathon-docker)