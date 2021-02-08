# How to Obtain RHACM Pre-Release Software

Customers that are part of the RHACM Early Access program can obtain pre-release software versions as part of the program to test out new features and functionality and report back the experience to their designated Red Hat Field Product Manager who can deliver that feedback to Engineering and Product Management.  The following document guides the customer through the process of getting access and also deploying those pre-release versions into an OpenShift cluster.

1) Go to [Quay.io](https://quay.io) and register for an account.
2) Once a username has been created provide that username to the Red Hat Field Product Manager who will get it associated to the private pre-release registries.
3) In the Quay.io interface in the upper right hand corner if you click on the username a drop down with *Account Settings* will appear.  Go into *Account Settings*.
4) On the left hand side of *Account Settings* select the *Gears* icon for user settings.
5) Near the top left of the settings page will be *CLI Password*.  Select this and type in the password for the username.
6) This will generate a *Username Credentials* box that will have various credentials for the user.  Select the *Kubernetes Secret* on the left hand side list.
7) Download the username-secret.yaml from the box as we will need this for later steps.  
8) Do not proceed with the next steps until the username has been confirmed by Red Hat that it has been given access to the private registry.
9) Obtain the upstream ACM deploy repository at Github on a machine that also has a working oc client and kubeconfig with access to the OpenShift cluster that will become the hub.  Also make a softlink for kubectl to the oc client if kubectl does not exist on host. :
~~~bash
sudo ln -s /usr/local/bin/oc /usr/local/bin/kubectl
git clone https://github.com/open-cluster-management/deploy.git
~~~
10) Create a pull-secret.yaml file from the contents of username-secret.yaml file obtained from Quay.io.  The metadata name should be updated to: multiclusterhub-operator-pull-secret.  Place the file inside the deploy/prereqs directory.  The finished file and location should look similar to the below but note the pull-secret itself has been redacted in this example:
~~~bash
$ cat ~/deploy/prereqs/pull-secret.yaml 
apiVersion: v1
kind: Secret
metadata:
  name: multiclusterhub-operator-pull-secret
data:
  .dockerconfigjson: <PULL-SECRET REDACTED>
type: kubernetes.io/dockerconfigjson
~~~
11) Update the snapshot file to a known good pre-release snapshot.  If not know ask the Red Hat Field Product Manager and they can provide that.  The updated file will look similar to the one below:
~~~bash
$ cat ~/deploy/snapshot.ver 
2.2.0-DOWNSTREAM-2021-01-28-15-20-59
~~~
12) Create an ImageContentSourcePolicy and apply it to the cluster.  The contents of that policy should look like the one below:
~~~bash
$ cat ~/icsp.yaml
apiVersion: operator.openshift.io/v1alpha1
kind: ImageContentSourcePolicy
metadata:
  name: rhacm-repo
spec:
  repositoryDigestMirrors:
  - mirrors:
    - quay.io:443/acm-d
    source: registry.redhat.io/rhacm2
  - mirrors:
    - registry.redhat.io/openshift4/ose-oauth-proxy
    source: registry.access.redhat.com/openshift4/ose-oauth-proxy
$ oc apply -f ~/icsp.yaml
~~~
13) Wait for nodes to reboot as the ImageContentSourcePolicy is applied before proceeding.
14) 
