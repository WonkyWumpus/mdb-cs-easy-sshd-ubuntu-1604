# mdb-cs-ubuntu-1604
## Overview

This project builds uses the WonkyWumpus/easy-sshd-ubuntu-1604 project as a starting point for building a MariaDB AX (MariaDB Columnstore) cluster running on Kubernetes.  It includes a Dockerfile that takes the networking and sshd capabilities built into the easy-sshd project and then adds OS prerequisites and stages the MariaDB AX tarfile into the images.  It also includes yaml files for creating the containers and the service that will be used to run the cluster.

**Important:** This project has been developed to assist rapid **testing** of more complex MariaDB Clusters.  **This is not intended as a template for building a production-ready, durable MariaDB AX Cluster.**

## Instructions

Make sure that you have followed the instructions at WonkyWumpus/easy-sshd-ubuntu-1604 to create the easy network service and the key and secret.  Those items are required before you can proceed.

Simply run kubectl to create the containers are service required for MariaDB AX.

`kubectl create -f mdb-stateful-um-pm-client.yaml`

The above referenced yaml file create two perfmance modules (pm-0 and pm-1), two user modules (um-0 and um-1), the um-service, and an extra "client" pod (client-1).  If you desire a different topology, you can either run an edited version of the yaml file, or you can scale the stateful sets used to manage the pods after creation.

As of the current version of this project, the pods created only have the software staged. There is no automatic build/configuation of the MariaDB AX -- you will need to run the standard installation process.  Full details of the installation process can be found in the [MariaDB ColumnStore Documentation](https://mariadb.com/kb/en/library/mariadb-columnstore/)

## Errata and Future Enhancements

Lot of work to due to make this closer to "Kubernetes native"
