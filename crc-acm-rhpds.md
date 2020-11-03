# Deploy CRC and ACM on CRC

This document assumes you have access to RHPDS, have requested a DEV GPTE Sandbox lab and have used Agnosticd and the kni-osp plugin to provision 4 nodes: provision node and 3 master nodes.   The provisioning node should use the OSP flavor 16c64gb100d and the master nodes should use the OSP flavor 16c30gb100d.  Once that is provisioned you can then use the steps below to deploy an CRC environment on the provisioning node with which you can install ACM onto it and then deploy a 3 master baremetal IPI clutser.

Go to the following to obtain a link for crc that we can use to curl down the tar file:

https://cloud.redhat.com/openshift/install/crc/installer-provisioned

curl -o crc-linux-amd64.tar.xz https://mirror.openshift.com/pub/openshift-v4/clients/crc/latest/crc-linux-amd64.tar.xz

tar -xf crc-linux-amd64.tar.xz

cd crc-linux-1.17.0-amd64

crc setup 

crc config set cpus 10

crc config set memory 32000

crc start

Paste in pull secret obtained from Code Ready Container site

We need to resize the qcow2 it pulls down.  Once the environment finishes coming up stop it again

crc stop

Now resize 

qemu-img resize ~/.crc/machines/crc/crc.qcow2 +20G

crc start

crc oc-env

eval $(crc oc-env)

oc login -u kubeadmin -p dpDFV-xamBW-kKAk3-Fi6Lg https://api.crc.testing:6443

Login to crc node to finish resize:

ssh -i /home/cloud-user/.crc/machines/crc/id_rsa core@192.168.130.11 

sudo xfs_growfs /sysroot

Now we can move onto installing ACM.  The templates used to deploy ACM from the cli are in this repository.

oc create namespace acm

oc project acm

Make sure acm-pull-secret file exists in home directory with the require pull secret (same ones used to deploy OCP).

oc create secret generic acm-secret -n acm --from-file=.dockerconfigjson=/home/cloud-user/acm-pull-secret --type=kubernetes.io/dockerconfigjson

oc apply -f acm-operator.yaml

oc apply -f acm-subscription.yaml

Make sure all pods are up before proceeding to install hub:

oc get pods -n acm 

oc apply -f acm-hub.yaml

All pods should be up before proceeding:

oc get pods -n acm

If using ACM 2.0 or less then we have to expose baremetal in the UI:

oc patch deploy console-header -n acm -p '{"spec":{"template":{"spec":{"containers":[{"name":"console-header","env": [{"name": "featureFlags_baremetal","value":"true"}]}]}}}}'

oc patch -n acm $(oc get deploy -o name | grep consoleui) -p '{"spec":{"template":{"spec":{"containers":[{"name":"hcm-ui","env": [{"name": "featureFlags_baremetal","value":"true"}]}]}}}}'

To access the UI for ACM/CRC OCP Cluster I chose tigerVNC.  Configure vnc on provisioning node and run vnc-server.  Establish a vnc tunnel over SSH and then using the VNC client from your local laptop/workstation connect.  I set my display on provisioner and ran Firefox which then showed up on my vnc session.

Create the baremetal assets (ie the baremetal hosts that ACM will deploy OCP on):

https://access.redhat.com/documentation/en-us/red_hat_advanced_cluster_management_for_kubernetes/2.0/html/manage_cluster/creating-and-modifying-bare-metal-assets

Use the following to create a provider connection:

https://access.redhat.com/documentation/en-us/red_hat_advanced_cluster_management_for_kubernetes/2.0/html/manage_cluster/creating-a-provider-connection#creating-a-provider-connection-for-bare-metal

Then use the following for a baremetal cluster deploy:

https://access.redhat.com/documentation/en-us/red_hat_advanced_cluster_management_for_kubernetes/2.0/html/manage_cluster/creating-a-cluster-with-red-hat-advanced-cluster-management-for-kubernetes#creating-a-cluster-on-bare-metal

In order for the cluster to import properly to ACM since its running on CRC you need to add static route on each master node in the cluster:

ssh core@master-0
ip route add 192.168.130.11 via 10.20.0.2 dev ens4

We also need to add a dns zone on the provisioning hosts dns server for api.crc.testing:

[root@provision named]# more crc.testing.zone 
$TTL 1D
@   IN SOA  dns.crc.testing   root.dns.crc.testing. (
                                       2017031330      ; serial
                                       1D              ; refresh
                                       1H              ; retry
                                       1W              ; expire
                                       3H )            ; minimum

$ORIGIN         crc.testing.
crc.testing.            IN      NS      dns.crc.testing.
dns                     IN      A       10.20.0.2

api		        IN	A	192.168.130.11


