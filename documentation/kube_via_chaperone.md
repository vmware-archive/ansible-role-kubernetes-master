#How to provision a Kubernetes environment using Chaperone

This procedure describes the steps required to provision a Kubernetes master and set of minions. 

**NOTE - Chaperone is under almost constant development. If this document doesn't do what it's intended to do please let us know**

And on with the show...

##Pre-requisites

 - Working Chaperone Deployment Server
 - One Photon 1.0 VM to be used as Kubernetes master
 - A number of Photon 1.0 VMs to be used as Kubernetes minions
 - Static IP addresses or DHCP reserved IP addresses for each Photon VM
 - Resolvable DNS A records for each Photon VM

##Procedure

###Configure Photon VMs
 1. Log into the console of Photon VM designated to be the Kubernetes Master. The username is *root* with a password of *changeme*.
 2. When prompted to change the password, update the password to *vmwarevmware*.
 3. Change the VM hostname to match the hostname portion of the FQDN.

 ```shell
 hostnamectl set-hostname REPLACE_ME
 ```

 4. Update the /etc/hosts file to reflect the new name and reboot.
 5. Execute steps 1 to 4 for each Photon VM.

###Prepare Chaperone Deployment Portal

 1. Open up a web browser to the Chaperone Administration portal and log in.
 2. Select the _Prepare_ menu on the left-hand side.
 3. Complete the steps outlined in the _Global Constants_, _Apps & Services_, and _Cloud Native Infrastructure_ section below prior to continuing. The _vSphere_ and _NSX_ sections are not required to deploy Kubernetes.

####Global Constants
 1. Under _Chaperone Configuration_, select _Global Constants_.
 2. Populate the **Primary NTP Server**, **Primary DNS Server**, **Syslog Server**, and **DNS Domain Name** fields.
 3. Click the _Save_ button when complete.

####Apps & Services
 1. Under _Chaperone Configuration_, select _Apps & Services_.
 2. Locate the _Cloud Management Platforms_ section and click the checkbox next to _Kubernetes_.
 3. Click the _Save_ button when complete.
 
####Cloud Native Infrastructure
 1. Under _Chaperone Configuration_, select _Cloud Native Infrastructure_.
 2. If Flannel should be installed from the source repository using a specific version click the checkbox next to _Install Flannel from source package_ and add the desired tag string into the _Flannel source package version_.
 3. If Kubernetes should be installed from the source repository using a specific version click the checkbox next to _Install kubernetes version from source package_ and add the desired tag string into the _Kubernetes source package version_.
 4. Add the **total** number of Photon VM instances that will be used to the _Number of Photon OS instances_ text box - this includes both the VM designated to be the Kubernetes Master and all of the Kubernetes Minions.
 5. Click the _Save_ button when complete.

###Install Chaperone Deployment Portal

1. Use an existing web browser session to the Chaperone Administration portal or open a web browser to the Chaperone Administration portal and log in.
2. Select the _Install_ menu on the left-hand side, then select _Chaperone Configuration_.
3. Click the _Save Chaperone Config_ button.
4. Monitor the Ansible command output in the textbox below. Look for the _PLAY RECAP_ section and verify that the output looks similar to below.

 ```shell
PLAY RECAP *********************************************************************
localhost                  : ok=5    changed=4    unreachable=0    failed=0   
```

###Prepare Photon and Kubernetes Configurations
1. Open a web browser to the Chaperone Deployment portal and log in.
2. Select the _Prepare_ menu on the left-hand side, the _Cloud Native Infrastructure_.
3. Complete the steps outlined in the _Photon OS_ and *Kubernetes Configuration* sections.

####Photon OS
1. Under the _Targets_ form section add the Photon VM IP address or FQDN to the _Photon OS Target_ text box and the password for the Photon VM's root account in the _Root Password_ text box. Note that the _Photon OS Target_ and _Root Password_ text boxes are grouped so that a single set of the _Photon OS Target_ and _Root Password_ text boxes are applicable to a single Photon VM.
2. Once all text boxes have been populated click the _Save_ button.

####Kubernetes Configuration

Populate the items in each sections as needed prior to continuing to the next steps.

#####General
1. Enter the desired DNS domain name for **intra-cluster** name resolution in the _Intra-cluster DNS domain name_. Note, this is the DNS domain name that will be assigned to each container provisioned by Kubernetes.

#####Management Topology
1. Add the IP address or FQDN name of the Photon OS instance that will act as the Kubernetes master to the _Master node IP or DNS A record_ text box.
2. For the remaining text boxes, add the IP address or FQDN name of each Photon OS instance that will act as the Kubernetes minion to the _Master node IP or DNS A record_ text box.

#####Network Design
1. Select the Flannel backend network driver from the _Flannel backend network type_ dropdown.
2. In the _Subnet for use by flannel_ text box add the subnet that should be used by Flannel for the routing fabric.
3. In the _Subnet mask for individual per-host networks_ text box add the VLSM that should be assigned to each Photon VM configured as a Kubernetes minion.
4. Add a beginning and end subnet to the _Beginning IP range for subnet allocation_ and _Ending IP range for subnet allocation_ text boxes. These two subnets limit the available networks that Flannel assigns to the Kubernetes minions.
5. Add the firewall port to be used for communication in the _Firewall port used to forward UDP or VXLAN traffic_ text box. This item is only required when the _Flannel backend network type_ is set to UDP or VXLAN.
6. If the _Flannel backend network type_ has been selected add the virtual network identifier value to the _VXLAN VNI_ text box.

Click the _Save_ button to save the configuration.

###Install the Photon and Kubernetes Configuration
1. Use an existing web browser session to the Chaperone Deployment portal or open a web browser to the Chaperone Deployment portal and log in.
2. Select the _Install_ menu on the left-hand side, then select _Cloud Native Infrastructure - Kubernetes_.
3. On the right-hand side of the portal several buttons are displayed. Read over their functions below.
    
    * Deploy Kubernetes Platform - this button combines all of the actions taken by the other _Cloud Native Infrastructure - Kubernetes_ buttons. If this button is selected no other buttons on this page are required.
    * Configure Photon OS only - this button applies a configuration to each Photon VM providing support for the other buttons.
    * Reset & install Kubernetes master and minions only - this button configures the Photon VMs as Kubernetes master and minions, and provisions the addons. Any existing Kubernetes deployments will be reset.
    * Reset & install Kubernetes master only - this button configures only the Photon VM designated as the Kubernetes master and provisions the addons. Any existing Kubernetes deployments will be reset.
    * (Re)Install Kubernetes minions only - this button configures only the Photon VM(s) designated as the Kubernetes minions. A Kubernetes master must be up and running. Any existing Kubernetes minion configuration will be reset.
    * (Re)Install Kubernetes addons only - this button removes any previously provisioned addons and deploys a new set of addons (see below for the list of addons). This button does not directly affect any existing payload containers but can indirectly affect communication due to loss of name resolution.
    * Reset Kubernetes to default state only - this button removes any existing addons and removes Kubernetes 

 A few things to note:
 * Buttons should be selected using the order specified below. These are the standard deployment patterns for a full Kubernetes environment.
	 * _Deploy Kubernetes Platform_
	 * _Configure Photon OS only_, then _Reset & install Kubernetes master and minions only_
	 * _Configure Photon OS only_, then _Reset & install Kubernetes master only_, then _(Re)Install Kubernetes minions only_
 * Selecting any of the buttons will reset a brownfield environment in some way. Any existing payload containers will not be removed but will be effectively orphaned from any ongoing control via Kubernetes.
 * At the time of this document's writing the only addons provisioned are:
	 * Kubernetes Dashboard
	 * DNS via SkyDNS
	 * Cluster logging via ElasticSearch, Fluentd, and Kibana
	 * Cluster monitoring via InfluxDB, Grafana, and Heapster.
4. When ready, click a button. The text box below the buttons displays the stdout from the Ansible playbooks. Depending on the button selected, a single Ansible play is executed or multiple Ansible plays are executed. 

 NOTE - When multiple Ansible plays are executed, a failure in one play does not stop subsequent plays. An example is when selecting the _Reset & install Kubernetes master and minions only_ button. This button actually calls four Ansible plays, starting with a reset, then the master configuration, addons provisioning, and finally minion configuration. If the Kubernetes master configuration fails this failure does not prevent the addons play from executing nor the minion configuration play. This logic can lead to scenarios where troubleshooting will require parsing the Ansible stdout. A quick method to find all failures is copying the entire stdout to a text editor and searching for the phrase _failed=1_.

###Validation
1. SSH into the Kubernetes master VM using the root account.
2. Verify that the Kubernetes nodes have successfully registered with the master. In this example below three Photon VMs were configured as minions (the master and two other VMs).

 ```
 kubectl get no
 NAME                         STATUS    AGE
 photon-1-master.corp.local   Ready     4s
 photon-1-node-1.corp.local   Ready     4s
 photon-1-node-2.corp.local   Ready     4s
 ```
3. Verify that all of the addons have been succesfully provisioned.

 ```
 kubectl get deployments,svc,rc,pods --all-namespaces
 NAMESPACE     NAME                                               DESIRED          CURRENT             UP-TO-DATE          AVAILABLE   AGE
kube-system   heapster-v1.2.0-beta.2                             1                1                   1                   0           3m
NAMESPACE     NAME                                               CLUSTER-IP       EXTERNAL-IP         PORT(S)             AGE
default       kubernetes                                         10.254.0.1       <none>              443/TCP             4m
kube-system   elasticsearch-logging                              10.254.32.69     <none>              9200/TCP            3m
kube-system   kibana-logging                                     10.254.6.23      <none>              5601/TCP            3m
kube-system   kube-dns                                           10.254.0.10      <none>              53/UDP,53/TCP       3m
kube-system   kubedash                                           10.254.75.31                         80/TCP              3m
kube-system   kubernetes-dashboard                               10.254.249.12    <none>              80/TCP              3m
kube-system   monitoring-grafana                                 10.254.252.206   <none>              80/TCP              3m
kube-system   monitoring-heapster                                10.254.70.7      <none>              8082/TCP            3m
kube-system   monitoring-influxdb                                10.254.37.92     <none>              8083/TCP,8086/TCP   3m
NAMESPACE     NAME                                               DESIRED          CURRENT             AGE
kube-system   elasticsearch-logging-v1                           1                1                   3m
kube-system   kibana-logging-v1                                  1                1                   3m
kube-system   kube-dns-v11                                       1                1                   3m
kube-system   kubedash                                           1                1                   3m
kube-system   kubernetes-dashboard-v1.1.0                        1                1                   3m
kube-system   monitoring-influxdb-grafana-v4                     1                1                   3m
NAMESPACE     NAME                                               READY            STATUS              RESTARTS   AGE
kube-system   elasticsearch-logging-v1-zav5y                     1/1              Running             0          3m
kube-system   fluentd-elasticsearch-photon-1-master.corp.local   1/1              Running             0          4m
kube-system   fluentd-elasticsearch-photon-1-node-1.corp.local   0/1              Running             0          1m
kube-system   fluentd-elasticsearch-photon-1-node-2.corp.local   0/1              Running   0          1m
kube-system   heapster-v1.2.0-beta.2-4211046505-znjkt            0/1              Running   0          3m
kube-system   kibana-logging-v1-yv9qk                            0/1              Running   0          3m
kube-system   kube-dns-v11-v0pgn                                 0/3              Running   0          3m
kube-system   kubedash-naww8                                     0/1              Running   0          3m
kube-system   kubernetes-dashboard-v1.1.0-tfj3k                  1/1              Running             0          3m
kube-system   monitoring-influxdb-grafana-v4-5y8ck               0/2              Running   0          3m

 ```
4. If any of the addons do not have a status of _running_ the error could be one of several places:
* Check the Ansible stdout for any errors during the import of the addons.
* The manifests for the addons are maintained in the /etc/kubernetes/addons directory. It is possible to remove and redeploy an existing addon using this command where "manifest.yml" represents the file name of the addon to remove. Look for any errors that the create command reports.
 
 ```
 kubectl delete -f manifest.yml
 kubectl create -f manifest.yml
 ```
* In some cases the addon containers the manifests point to may have issues. If an addon needs to use an updated container URL modify the specific manifest file and commit the change. We will thank you!