
#How to provision a demo LAMP stack to Kubernetes using Chaperone

This procedure describes the steps required to provision a two-tier LAMP stack to an existing Kubernetes environment using Chaperone.

**NOTE - Chaperone is under almost constant development. If this document doesn't do what it's intended to do please let us know**

##Background
As of the writing of this document the UI and Ansible flexibility is limited and requires additional work to successfully provision the LAMP stack. For simplistic reasons it is required to build both the Apache and MySQL containers with the data files included. The Ansible logic is configured to build the Kubernetes manifests from Jinja2 templates that only replace the target container URI, the replica count, and the NodePort for the Apache service.

The applications included are split into four Kubernetes manifests:

 * Apache replication controller
 * Apache service
 * MySQL replication controller
 * MySQL service

Although the Chaperone UI provides the ability to change the number of replicas this functionality is a work in progress.

##Prerequisites
* Working Chaperone Deployment Server
* Working Kubernetes environment deployed by Chaperone
* A Docker container running Apache with the PHP data files baked in
* A Docker container running MySQL with the data files baked in
* An accessible Docker registry hosting both of the Docker containers

##Procedure
1. If the Chaperone Deployment portal has not yet been configured then follow the steps outlined in [Prepare Chaperone Deployment Portal](https://github.com/vmware/ansible-role-kubernetes-master/blob/master/documentation/kube_via_chaperone.md#prepare-chaperone-deployment-portal) _Global Constants_ and _Apps & Services_ sections.
2. Open a web browser to the Chaperone Deployment portal. and log in.

###Prepare Application Provisioning
1. Select _Prepare_ from the left-hand side navigation menu, then expand _Cloud Native Applications_, and select the _Application Provisioning_ item.

####Remote Docker Registry

**NOTE - This step is only required if the target Apache and/or MySQL containers are hosted at Docker Hub.**

1. Add the Docker Registry information to following fields:
* Name of Kubernetes secret that will house this information
* Docker Registry URL
* Docker Registry User
* Docker Registry Password
* Docker Registry Email Address
2. Click the _Save_ button.

####Demo App
1. Select the _Demo App_ menu item located under the _Prepare/Cloud Native Applications_ menu on the left-hand side. Complete the forms required for each section below. Once completed click the _Save_ button,

###Apache Controller Configuration
1. Add the URL where the Apache Docker container is maintained in the _Container Image URI_ text box. This address needs to be resolvable by the Kubernetes master.
2. The default of _Image Pull Policy_ is _Always_, if the Apache container is already cached locally and there is no update then _Never_ can be selected in the dropdown.
3. For demo purposes leave the _Number of Apache replicas_ at the default of 1.

###Apache Service Configuration
1. Enter a TCP port from 30000 to 32767 to host the LAMP web site on in the _External Node Port_ text box. The port supplied must not already be in-use within this Kubernetes environment.

###MySQL Controller Configuration
1. Add the URL where the MySQL Docker container is maintained in the _Container Image URI_ text box. This address needs to be resolvable by the Kubernetes master.
2. The default of _Image Pull Policy_ is _Always_, if the MySQL container is already cached locally and there is no update then _Never_ can be selected in the dropdown.
3. For demo purposes leave the _Number of MySQL replicas_ at the default of 1.

###Install Cloud Native Applications

1. Select _Install_ from the left-hand side navigation menu, then select _Cloud Native Applications_.
2. Click the _Deploy Apps in Kubernetes_ button. This executes a single Ansible play that creates the Kubernetes manifests from pre-formed templates, then issues four _kubectl create -f \<manifest_file_name>_ commands to provision the containers.

###Validatation

 1. The validity of the Ansible play execution can be verified by reviewing the stdout response displayed in the UI below the task buttons. A successful run should report the following, where _photon-1-master.corp.local_ is the hostname of the Kubernetes master. Note that this result only describes the validity of the Ansible run, not whether the containers have been deployed successfully.

 ```shell
 PLAY RECAP *********************************************************************
 photon-1-master.corp.local : ok=5    changed=4    unreachable=0    failed=0
 ```
 2. It may take several of minutes to deploy the containers. The container provisioning process can be monitored from the Kubernetes master by executing the following command at the console:

 ```shell
 kubectl get po --watch
 ```
 3. Once the containers have finished provisioning the website should be accessible by via the Kubernetes master using the port defined in _Apache Service Configuration_. The kubectl command executed in step two should return a status of _Running_ as show below.

 ```shell
 kubectl get po --watch
 NAME            READY     STATUS    RESTARTS   AGE
 apache-rx0cu    1/1       Running   0          2d
 mysqldb-kcsfr   1/1       Running   0          2d
 ```
 
 4. If the container status is not _Running_ then troubleshooting is required. The most straightforward method is to run the _kubectl create -f \<manifest file>_ from the Kubernetes master. The location of these files is normally the _/root_ directory. The error message returned will indicate what the next steps should be.
