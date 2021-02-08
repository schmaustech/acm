# How to Obtain RHACM Pre-Release Software

Customers that are part of the RHACM Early Access program can obtain pre-release software versions as part of the program to test out new features and functionality and report back the experience to their designated Red Hat Field Product Manager who can deliver that feedback to Engineering and Product Management.  The following document guides the customer through the process of getting access and also deploying those pre-release versions into an OpenShift cluster.

1) Go to [Quay.io](https://quay.io) and register for an account.
2) Once a username has been created provide that username to the Red Hat Field Product Manager who will get it associated to the private pre-release registries.  Do not proceed until this step has been completed and it is confirmed the username has access rights.
3) In the Quay.io interface in the upper right hand corner if you click on the username a drop down with *Account Settings* will appear.  Go into *Account Settings*.
4) 
) Obtain the upstream ACM deploy repository at github on a machine that also has a working oc client and kubeconfig with access to the OpenShift cluster that will become the hub:
~~~bash
git clone https://github.com/open-cluster-management/deploy.git
~~~
) 
