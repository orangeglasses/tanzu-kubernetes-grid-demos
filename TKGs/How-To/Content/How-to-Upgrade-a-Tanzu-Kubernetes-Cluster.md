# How to Upgrade a Tanzu Kubernetes Cluster

You cannot decrease the version. For example, you cannot downgrade from Kubernetes v1.18 to v1.17.

You can upgrade the minor version, but only incrementally. Skipping minor versions is not supported. For example, you can upgrade from Kubernetes v1.17 to v1.18, but you cannot upgrade from Kubernetes v1.16 to v1.18.

You cannot change the major version, such as upgrading from v1.18 to v2.0.

https://docs.vmware.com/en/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-DF2B3886-4BE0-4E88-B549-DC9C1C653FDB.html

Version updates of kubernetes can be updated by 1 or 2 methods:

Edit the manifest directly: 
https://docs.vmware.com/en/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-A7A0BC51-F49D-4E08-B4DD-782D84CB3762.html


Use Patch Method:
https://docs.vmware.com/en/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-C285CC64-93F6-4EB3-9CBA-7903DE74755E.html


You can also edit and re-apply the YAML file that was originally used, but this is NOT recommended. (see: https://docs.vmware.com/en/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-DF2B3886-4BE0-4E88-B549-DC9C1C653FDB.html#cluster-manifest-update-methods-2)

## Table of Contents

1. [Prerequisites](#prerequisites)
1. [Check the current Version of your Tanzu Kubernetes Cluster](#check-the-current-version-of-your-tanzu-kubernetes-cluster)
1. [List the possible Tanzu Kubernetes Releases](#list-the-possible-tanzu-kubernetes-releases ) 
1. [Upgrade the Tanzu Kubernetes Cluster using kubectl edit](#upgrade-the-tanzu-kubernetes-cluster-using-kubectl-edit)
1. [Upgrade the Tanzu Kubernetes Cluster using kubectl patch](#upgrade-the-tanzu-kubernetes-cluster-using-kubectl-patch)
1. [Monitor the TKC Upgrade](#monitor-the-tkc-upgrade)

## Prerequisites
* [Make sure you are logged in](How-to-Login.md) and connected to the vSphere Namespace where you would like to Scale the Tanzu Kubernetes Cluster.

## Check the current Version of your Tanzu Kubernetes Cluster

In order to check the current version of your Tanzu Kubernetes Cluster, you can use the following command:
````
kubectl get tanzukubernetesclusters

OR

kubectl get tkc

NAME           CONTROL PLANE   WORKER   TKR NAME                           AGE   READY   TKR COMPATIBLE   UPDATES AVAILABLE
tkgs-v2-wl02   1               1        v1.21.2---vmware.1-tkg.1.ee25d55   8d    True    True             [1.21.6+vmware.1-tkg.1.b3d708a]
````

The Current Version of your TKC is shown under 'TKR NAME'. If there are updates available this is shown under the field 'UPDATES AVAILABLE'. 

**Warning:** The version mentioned under 'UPDATES AVAILABLE' cannot directly be used in v1alpha2 Cluster Specifications since this the deprecated Distribution Format. Use the real TKR name ash shown below in the next step (List the possible Tanzu Kubernetes Releases).

In this example the TKC is currently at version 1.21.2 and an update is available of version 1.21.6. 

## List the possible Tanzu Kubernetes Releases


Let's show all the Tanzu Kubernetes Releases available in our vSphere with Tanzu environment:

````
kubectl get tanzukubernetesreleases

OR

kubectl get tkr

NAME                                VERSION                          READY   COMPATIBLE   CREATED   UPDATES AVAILABLE
v1.16.12---vmware.1-tkg.1.da7afe7   1.16.12+vmware.1-tkg.1.da7afe7   True    True         10d       [1.17.17+vmware.1-tkg.1.d44d45a 1.16.14+vmware.1-tkg.1.ada4837]
v1.16.14---vmware.1-tkg.1.ada4837   1.16.14+vmware.1-tkg.1.ada4837   True    True         10d       [1.17.17+vmware.1-tkg.1.d44d45a]
v1.16.8---vmware.1-tkg.3.60d2ffd    1.16.8+vmware.1-tkg.3.60d2ffd    False   False        10d       [1.17.17+vmware.1-tkg.1.d44d45a 1.16.14+vmware.1-tkg.1.ada4837]
v1.17.11---vmware.1-tkg.1.15f1e18   1.17.11+vmware.1-tkg.1.15f1e18   True    True         10d       [1.18.19+vmware.1-tkg.1.17af790 1.17.17+vmware.1-tkg.1.d44d45a]
v1.17.11---vmware.1-tkg.2.ad3d374   1.17.11+vmware.1-tkg.2.ad3d374   True    True         10d       [1.18.19+vmware.1-tkg.1.17af790 1.17.17+vmware.1-tkg.1.d44d45a]
v1.17.13---vmware.1-tkg.2.2c133ed   1.17.13+vmware.1-tkg.2.2c133ed   True    True         10d       [1.18.19+vmware.1-tkg.1.17af790 1.17.17+vmware.1-tkg.1.d44d45a]
v1.17.17---vmware.1-tkg.1.d44d45a   1.17.17+vmware.1-tkg.1.d44d45a   True    True         10d       [1.18.19+vmware.1-tkg.1.17af790]
v1.17.7---vmware.1-tkg.1.154236c    1.17.7+vmware.1-tkg.1.154236c    True    True         10d       [1.18.19+vmware.1-tkg.1.17af790 1.17.17+vmware.1-tkg.1.d44d45a]
v1.17.8---vmware.1-tkg.1.5417466    1.17.8+vmware.1-tkg.1.5417466    True    True         10d       [1.18.19+vmware.1-tkg.1.17af790 1.17.17+vmware.1-tkg.1.d44d45a]
v1.18.10---vmware.1-tkg.1.3a6cd48   1.18.10+vmware.1-tkg.1.3a6cd48   True    True         10d       [1.19.16+vmware.1-tkg.1.df910e2 1.18.19+vmware.1-tkg.1.17af790]
v1.18.15---vmware.1-tkg.1.600e412   1.18.15+vmware.1-tkg.1.600e412   True    True         10d       [1.19.16+vmware.1-tkg.1.df910e2 1.18.19+vmware.1-tkg.1.17af790]
v1.18.15---vmware.1-tkg.2.ebf6117   1.18.15+vmware.1-tkg.2.ebf6117   True    True         10d       [1.19.16+vmware.1-tkg.1.df910e2 1.18.19+vmware.1-tkg.1.17af790]
v1.18.19---vmware.1-tkg.1.17af790   1.18.19+vmware.1-tkg.1.17af790   True    True         10d       [1.19.16+vmware.1-tkg.1.df910e2]
v1.18.5---vmware.1-tkg.1.c40d30d    1.18.5+vmware.1-tkg.1.c40d30d    True    True         10d       [1.19.16+vmware.1-tkg.1.df910e2 1.18.19+vmware.1-tkg.1.17af790]
v1.19.11---vmware.1-tkg.1.9d9b236   1.19.11+vmware.1-tkg.1.9d9b236   True    True         10d       [1.20.12+vmware.1-tkg.1.b9a42f3 1.19.16+vmware.1-tkg.1.df910e2]
v1.19.14---vmware.1-tkg.1.8753786   1.19.14+vmware.1-tkg.1.8753786   True    True         10d       [1.20.12+vmware.1-tkg.1.b9a42f3 1.19.16+vmware.1-tkg.1.df910e2]
v1.19.16---vmware.1-tkg.1.df910e2   1.19.16+vmware.1-tkg.1.df910e2   True    True         10d       [1.20.12+vmware.1-tkg.1.b9a42f3]
v1.19.7---vmware.1-tkg.1.fc82c41    1.19.7+vmware.1-tkg.1.fc82c41    True    True         10d       [1.20.12+vmware.1-tkg.1.b9a42f3 1.19.16+vmware.1-tkg.1.df910e2]
v1.19.7---vmware.1-tkg.2.f52f85a    1.19.7+vmware.1-tkg.2.f52f85a    True    True         10d       [1.20.12+vmware.1-tkg.1.b9a42f3 1.19.16+vmware.1-tkg.1.df910e2]
v1.20.12---vmware.1-tkg.1.b9a42f3   1.20.12+vmware.1-tkg.1.b9a42f3   True    True         10d       [1.21.6+vmware.1-tkg.1.b3d708a]
v1.20.2---vmware.1-tkg.1.1d4f79a    1.20.2+vmware.1-tkg.1.1d4f79a    True    True         10d       [1.21.6+vmware.1-tkg.1.b3d708a 1.20.12+vmware.1-tkg.1.b9a42f3]
v1.20.2---vmware.1-tkg.2.3e10706    1.20.2+vmware.1-tkg.2.3e10706    True    True         10d       [1.21.6+vmware.1-tkg.1.b3d708a 1.20.12+vmware.1-tkg.1.b9a42f3]
v1.20.7---vmware.1-tkg.1.7fb9067    1.20.7+vmware.1-tkg.1.7fb9067    True    True         10d       [1.21.6+vmware.1-tkg.1.b3d708a 1.20.12+vmware.1-tkg.1.b9a42f3]
v1.20.8---vmware.1-tkg.2            1.20.8+vmware.1-tkg.2            True    True         10d       [1.21.6+vmware.1-tkg.1]
v1.20.9---vmware.1-tkg.1.a4cee5b    1.20.9+vmware.1-tkg.1.a4cee5b    True    True         10d       [1.21.6+vmware.1-tkg.1.b3d708a 1.20.12+vmware.1-tkg.1.b9a42f3]
v1.21.2---vmware.1-tkg.1.ee25d55    1.21.2+vmware.1-tkg.1.ee25d55    True    True         10d       [1.21.6+vmware.1-tkg.1.b3d708a]
v1.21.6---vmware.1-tkg.1            1.21.6+vmware.1-tkg.1            True    True         10d       
v1.21.6---vmware.1-tkg.1.b3d708a    1.21.6+vmware.1-tkg.1.b3d708a    True    True         10d    
````

Let's upgrade the cluster to v1.21.6---vmware.1-tkg.1.b3d708a below. 

## Upgrade the Tanzu Kubernetes Cluster using kubectl edit

As highlighted earlier: one of the ways to upgrade a running Tanzu Kubernetes Cluster is to use the 'kubectl edit' command. 

So in our example we know the following:
* TKC to upgrade: tkgs-v2-wl02
* Current Version: v1.21.2---vmware.1-tkg.1.ee25d55
* Version to upgrade to (TKR): v1.21.6---vmware.1-tkg.1.b3d708a

Let's get started!

**Note:** Currently, as of writing this document, it is not supported to have your Control Planes set to a different version than your Worker Nodes. Hence defining the new version on your Control Planes only is sufficient to trigger the upgrade of your entire TKC. However putting a TKR Name in both Control Plane & Worker Nodes makes you ready for future releases.  

````
kubectl edit tkc tkgs-v2-wl02

apiVersion: run.tanzu.vmware.com/v1alpha2
kind: TanzuKubernetesCluster
......
spec:
......
 topology:
  controlPlane:
    replicas: 1
    storageClass: tkg-storagepolicy
    tkr:
      reference:
        name: v1.21.6---vmware.1-tkg.1.b3d708a # --> Change this reference to your desired TKR
    vmClass: best-effort-xsmall
  nodePools:
  - name: worker-nodepool-b1
    replicas: 1
    storageClass: tkg-storagepolicy
    tkr:
      reference:
        name: v1.21.6---vmware.1-tkg.1.b3d708a # --> Change this reference to your desired TKR
   vmClass: best-effort-xsmall
......
tanzukubernetescluster.run.tanzu.vmware.com/tkgs-v2-wl02 edited
````

The Upgrade of your TKC should start. 

## Upgrade the Tanzu Kubernetes Cluster using kubectl patch

Another way to Upgrade your TKC is by using the 'kubectl patch' command. 

First you need to get the 'topology' specifications from your current cluster. This can be achieved by running the following command:

````
kubectl get TKG-CLUSTER-NAME -o YAML

...
topology:
    controlPlane:
      replicas: 1
      storageClass: tkg-storagepolicy
      tkr:
        reference:
          name: v1.21.2---vmware.1-tkg.1.ee25d55
      vmClass: best-effort-xsmall
    nodePools:
    - name: worker-nodepool-b1
      replicas: 1
      storageClass: tkg-storagepolicy
      tkr:
        reference:
          name: v1.21.2---vmware.1-tkg.1.ee25d55
      vmClass: best-effort-xsmall
... 
````

Make sure to copy the topology section. 

Then you need to define the name of the TKR you want to upgrade to in a $PATCH variable as shown below (do not copy directly from the example below, change it in your previously found topology section):
````
$ read -r -d '' PATCH <<'EOF'
spec:
  topology:
    controlPlane:
      replicas: 1
      storageClass: tkg-storagepolicy
      tkr:
        reference:
          name: v1.21.6---vmware.1-tkg.1.b3d708a # --> Change   this reference to your desired TKR
      vmClass: best-effort-xsmall
    nodePools:
    - name: worker-nodepool-b1
      replicas: 1
      storageClass: tkg-storagepolicy
      tkr:
        reference:
          name: v1.21.6---vmware.1-tkg.1.b3d708a # --> Change   this reference to your desired TKR
     vmClass: best-effort-xsmall
EOF
````
Now let's use this $PATCH variable to Upgrade our TKC:
````
kubectl patch --type=merge tanzukubernetescluster TKG-CLUSTER-NAME --patch "$PATCH"


tanzukubernetescluster.run.tanzu.vmware.com/TKG-CLUSTER-NAME patched
````

That's it!

## Monitor the TKC Upgrade

In order to actively monitor the TKC Upgrade, you can run the following command:
````
kubectl get tanzukubernetesclusters -w

OR

kubectl get tkc -w

NAME           CONTROL PLANE   WORKER   TKR NAME                           AGE   READY   TKR COMPATIBLE   UPDATES AVAILABLE
tkgs-v2-wl02   1               1        v1.21.6---vmware.1-tkg.1.b3d708a   9d    False   True     
````
Wait until the 'Ready' Status says 'True'. 



