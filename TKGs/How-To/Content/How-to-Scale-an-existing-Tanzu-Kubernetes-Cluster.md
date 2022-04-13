# How to Scale an existing Tanzu Kubernetes Cluster

Scaling an Existing Tanzu Kubernetes Cluster is very straight forwards, it follows the same concept as scaling a Replicaset for your Application Deployments for example. 

Official VMware Documentation:
* v1alpha2 Clusters: https://docs.vmware.com/en/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-7992B7F6-9174-44F4-99BE-C0B5C45FA2EC.html
* v1alpha1 Clusters: https://docs.vmware.com/en/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-29DA638D-23B5-4A53-9152-7BD5D5F85BFE.html
## Table of Contents

1. [Prerequisites](#prerequisites)
1. [Scale a TKC with kubectl edit](#scale-a-tkc-with-kubectl-edit)
1. [Scale a TKC with kubectl apply](#scale-a-tkc-with-kubectl-apply)
1. [Monitor Scale Request](#monitor-scale-request)

## Prerequisites
* [Make sure you are logged in](How-to-Login.md) and connected to the vSphere Namespace where you would like to Scale the Tanzu Kubernetes Cluster.
* Once logged in, obtain the **name of your TKC** that you would like to scale by running:
    ````
    kubectl get tkc
    ````

## Scale a TKC with kubectl edit

A Tanzu Kubernetes Cluster (TKC) is just another Object in your environment. This means you can use the 'kubectl edit' command to modify your Tanzu Kubernetes Cluster.

Let's increase the number of Worker Nodes from 1 to 2 in our example:


````
kubectl edit tkc <TKC-NAME>

apiVersion: run.tanzu.vmware.com/v1alpha2
kind: TanzuKubernetesCluster
........
spec:
  topology:
    nodePools:
    - name: worker-nodepool-b1
    replicas: 2 # --> Increased this number to 2
........

tanzukubernetescluster.run.tanzu.vmware.com/tkgs-v2-wl02 edited
````

Once you save the updated Cluster Specification, a new Worker Node will be added to your TKC. 

## Scale a TKC with kubectl apply
A Tanzu Kubernetes Cluster (TKC) is just another Object in your environment. This means you can use the 'kubectl apply' command to modify your Tanzu Kubernetes Cluster.

You can achieve this by editing the Cluster Specification YAML file and later apply the changes. 

Let's increase the number of Worker Nodes from 2 to 1 in our example:

````
vi <CLUSTER-SPEC-YAML>

apiVersion: run.tanzu.vmware.com/v1alpha2
kind: TanzuKubernetesCluster
........
spec:
  nodePools:
      - name: worker-nodepool-b1
        replicas: 1 # Changed this value to 1

........

kubectl apply -f <CLUSTER-SPEC-YAML>

tanzukubernetescluster.run.tanzu.vmware.com/tkgs-v2-wl02 configured
````

Now your TKC's Worker Nodes are scaled down to 1. 

## Monitor Scale Request

You can monitor your Scale Request by running the following command:

````
kubectl get tkc -w

NAME           CONTROL PLANE   WORKER   TKR NAME                           AGE   READY   TKR COMPATIBLE   UPDATES AVAILABLE
tkgs-v2-wl02   1               2        v1.21.2---vmware.1-tkg.1.ee25d55   8d    False   True             [1.21.6+vmware.1-tkg.1.b3d708a]
````


Wait until the status of the 'Ready' field is 'True'. 


