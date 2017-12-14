# mdb-cs-easy-sshd-ubuntu-1604
## Overview

This project builds uses the WonkyWumpus/easy-sshd-ubuntu-1604 project as a starting point for building a MariaDB AX (MariaDB Columnstore) cluster running on Kubernetes.  It includes a Dockerfile that takes the networking and sshd capabilities built into the easy-sshd project and then adds OS prerequisites and stages the MariaDB AX tarfile into the images.  It also includes yaml files for creating the containers and the service that will be used to run the cluster.

**Important:** This project has been developed to assist rapid **testing** of more complex MariaDB Clusters.  **This is not intended as a template for building a production-ready, durable MariaDB AX Cluster.**

## Instructions

Make sure that you have followed the instructions at WonkyWumpus/easy-sshd-ubuntu-1604 to create the easy network service and the key and secret.  Those items are required before you can proceed.

Simply run kubectl to create the containers are service required for MariaDB AX.

`kubectl create -f mdb-stateful-um-pm-client.yaml`

The above referenced yaml file create two perfmance modules (pm-0 and pm-1), two user modules (um-0 and um-1), the um-service, and an extra "client" pod (client-1).  If you desire a different topology, you can either run an edited version of the yaml file, or you can scale the stateful sets used to manage the pods after creation.

As of the current version of this project, the pods created only have the software staged. There is no automatic build/configuation of the MariaDB AX -- you will need to run the standard installation process.  Full details of the installation process can be found in the [MariaDB ColumnStore Documentation](https://mariadb.com/kb/en/library/mariadb-columnstore/).

However, abbreviated instructions for quickly completing the install and configuation are included here.  These instructions assume your are kubernetes on a single host via minikube.

Determine the IP address of the pm-0 container, and logon to that container as root.

`kubectl describe pod pm-0|grep IP`

`minikube ssh`

`ssh -i .ssh/easy-key root@<IP from first step>`

At this point you are on the container from which you will drive the entire installation.  Next step is to install the .deb packages.

`tar -xzf mariadb-columnstore-*-xenial.x86_64.deb.tar.gz`

`root@ubuntu-01:~# dpkg -i mariadb-columnstore-*-x86_64*.deb`

Now the software is installed on one node, and it is time time install on the other nodes and stiach them together as a single MariaDB AX Cluster.  This is done by executing the postConfigure script.

```
root@pm-0:~# /usr/local/mariadb/columnstore/bin/postConfigure

This is the MariaDB ColumnStore System Configuration and Installation tool.
It will Configure the MariaDB ColumnStore System and will perform a Package
Installation of all of the Servers within the System that is being configured.

IMPORTANT: This tool should only be run on the Parent OAM Module
           which is a Performance Module, preferred Module #1

Prompting instructions:

	Press 'enter' to accept a value in (), if available or
	Enter one of the options within [], if available, or
	Enter a new value


===== Setup System Server Type Configuration =====

There are 2 options when configuring the System Server Type: single and multi

  'single'  - Single-Server install is used when there will only be 1 server configured
              on the system. It can also be used for production systems, if the plan is
              to stay single-server.

  'multi'   - Multi-Server install is used when you want to configure multiple servers now or
              in the future. With Multi-Server install, you can still configure just 1 server
              now and add on addition servers/modules in the future.

Select the type of System Server install [1=single, 2=multi] (2) > 2


===== Setup System Module Type Configuration =====

There are 2 options when configuring the System Module Type: separate and combined

  'separate' - User and Performance functionality on separate servers.

  'combined' - User and Performance functionality on the same server

Select the type of System Module Install [1=separate, 2=combined] (2) > 1

Seperate Server Installation will be performed.

NOTE: Local Query Feature allows the ability to query data from a single Performance
      Module. Check MariaDB ColumnStore Admin Guide for additional information.

Enable Local Query feature? [y,n] (n) > y


NOTE: Local Query Feature is enabled

NOTE: MariaDB ColumnStore Replication Feature is enabled

Enter System Name (columnstore-1) > columnstore-1


===== Setup Storage Configuration =====


----- Setup Performance Module DBRoot Data Storage Mount Configuration -----

There are 2 options when configuring the storage: internal or external

  'internal' -    This is specified when a local disk is used for the DBRoot storage.
                  High Availability Server Failover is not Supported in this mode

  'external' -    This is specified when the DBRoot directories are mounted.
                  High Availability Server Failover is Supported in this mode.

Select the type of Data Storage [1=internal, 2=external] (1) > 1

===== Setup Memory Configuration =====

NOTE: Setting 'NumBlocksPct' to 70%
      Setting 'TotalUmMemory' to 50%

===== Setup the Module Configuration =====


----- User Module Configuration -----

Enter number of User Modules [1,1024] (1) > 2

*** User Module #1 Configuration ***

Enter Nic Interface #1 Host Name (unassigned) > um-0
Enter Nic Interface #1 IP Address of um-0 (172.17.0.6) >
Enter Nic Interface #2 Host Name (unassigned) >

*** User Module #2 Configuration ***

Enter Nic Interface #1 Host Name (unassigned) > um-1
Enter Nic Interface #1 IP Address of um-1 (172.17.0.12) >
Enter Nic Interface #2 Host Name (unassigned) >

----- Performance Module Configuration -----

Enter number of Performance Modules [1,1024] (1) > 2

*** Parent OAM Module Performance Module #1 Configuration ***

Enter Nic Interface #1 Host Name (pm-0) > pm-0
Enter Nic Interface #1 IP Address of pm-0 (172.17.0.5) >
Enter Nic Interface #2 Host Name (unassigned) >
Enter the list (Nx,Ny,Nz) or range (Nx-Nz) of DBRoot IDs assigned to module 'pm1' (1) > 1-6

*** Performance Module #2 Configuration ***

Enter Nic Interface #1 Host Name (unassigned) > pm-1
Enter Nic Interface #1 IP Address of pm-1 (172.17.0.13) >
Enter Nic Interface #2 Host Name (unassigned) >
Enter the list (Nx,Ny,Nz) or range (Nx-Nz) of DBRoot IDs assigned to module 'pm2' () > 7-12

===== System Installation =====

System Configuration is complete.
Performing System Installation.

Performing a MariaDB ColumnStore System install using using DEB packages
located in the /root directory.

Next step is to enter the password to access the other Servers.
This is either your password or you can default to using a ssh key
If using a password, the password needs to be the same on all Servers.

Enter password, hit 'enter' to default to using a ssh key, or 'exit' >


===== Running the MariaDB ColumnStore MariaDB ColumnStore setup scripts =====

post-mysqld-install Successfully Completed
post-mysql-install Successfully Completed

----- Performing Install on 'um1 / um-0' -----

Install log file is located here: /tmp/um1_deb_install.log


----- Performing Install on 'um2 / um-1' -----

Install log file is located here: /tmp/um2_deb_install.log


----- Performing Install on 'pm2 / pm-1' -----

Install log file is located here: /tmp/pm2_deb_install.log


MariaDB ColumnStore Package being installed, please wait ...  DONE

===== Checking MariaDB ColumnStore System Logging Functionality =====

The MariaDB ColumnStore system logging is setup and working on local server

===== MariaDB ColumnStore System Startup =====

System Configuration is complete.
Performing System Installation.

----- Starting MariaDB ColumnStore on local server -----

MariaDB ColumnStore successfully started

MariaDB ColumnStore Database Platform Starting, please wait .......... DONE

System Catalog Successfully Created

Run MariaDB ColumnStore Replication Setup..  DONE

MariaDB ColumnStore Install Successfully Completed, System is Active

Enter the following command to define MariaDB ColumnStore Alias Commands

. /usr/local/mariadb/columnstore/bin/columnstoreAlias

Enter 'mcsmysql' to access the MariaDB ColumnStore SQL console
Enter 'mcsadmin' to access the MariaDB ColumnStore Admin console

NOTE: The MariaDB ColumnStore Alias Commands are in /etc/profile.d/columnstoreAlias

root@pm-0:~# . /usr/local/mariadb/columnstore/bin/columnstoreAlias
```

One of your first tasks is likely to be to create a user that can connect from your client01 pod or from your host machine. You will create your user by connecting through the um-0 machine.

`root@pm-0:~# ssh root@um-0`

`root@um-0:~# mcsmysql`

```
MariaDB [(none)]> create user dba@'%' identified by 'm18Maria';`
Query OK, 0 rows affected (0.00 sec)
```

```
MariaDB [(none)]> grant all on *.* to dba@'%';
Query OK, 0 rows affected (0.00 sec)
```

You can test your connection into MariaDB AX from your client01 pod

`root@um-0:~# ssh root@client01`

`root@client01:~# /usr/local/mariadb/columnstore/mysql/bin/mysql --user=dba --password=m18Maria --host=um-0`

You can also test the connection from your host laptop into the UM Service.

`$ kubectl describe service um-service|grep IP`

`$ kubectl describe service um-service|grep "NodePort:"`

`$ mysql --user=dba --password=m18Maria --host=<IP from above> --port=<port from above>`

## Issues, Comments and Suggestions

Please use the github issue feature to provide any and all feedback.

## Errata and Future Enhancements

**Lots** of work (refactoring included) to do to make this closer to "Kubernetes native".  Some of the work is gated by hoped for changes to core install and config process of MariaDB AX.
