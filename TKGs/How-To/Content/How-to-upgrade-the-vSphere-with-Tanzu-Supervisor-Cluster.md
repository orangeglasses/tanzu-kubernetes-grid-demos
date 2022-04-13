# How to upgrade the vSphere with Tanzu Supervisor Cluster

Generally speaking VMware provides Rolling Updates for the Supervisor Cluster and the Tanzu Kubernetes Clusters. 

Updating the Supervisor Cluster involves first updating your vCenter Server and ESXi Hosts. Afterwards you can trigger a Supervisor Cluster Update by doing a vSphere Namespace Update. In general updating a Supervisor Cluster will trigger a Rolling Update of the Tanzu Kubernetes Clusters.

In some cases you might need to check the Compatibility between your Tanzu Kubernetes Clusters and your Supervisor Cluster. Once done the outcome might be that you need to update your TKC first before updating your Supervisor Cluster.

In order to verify the Compatibility of the Tanzu Kubernetes Releases or your Tanzu Kubernetes Clusters with the Supervisor Cluster; you can run the following commands:

````
kubectl get tanzukubernetesreleases

NAME                                VERSION                          READY   COMPATIBLE   CREATED   UPDATES AVAILABLE
v1.16.12---vmware.1-tkg.1.da7afe7   1.16.12+vmware.1-tkg.1.da7afe7   True    True         11d       [1.17.17+vmware.1-tkg.1.d44d45a 1.16.14+vmware.1-tkg.1.ada4837]
v1.16.14---vmware.1-tkg.1.ada4837   1.16.14+vmware.1-tkg.1.ada4837   True    True         11d       [1.17.17+vmware.1-tkg.1.d44d45a]

(look for the COMPATIBLE field, this field indicates if a TKR is compatibile with your Supervisor Cluster)

OR

kubectl get tkc

NAME           CONTROL PLANE   WORKER   TKR NAME                           AGE   READY   TKR COMPATIBLE   UPDATES AVAILABLE
tkgs-v2-wl02   1               1        v1.21.6---vmware.1-tkg.1.b3d708a   10d   True    True             
tkgs-v2-wl03   1               2        v1.21.2---vmware.1-tkg.1.ee25d55   24h   True    True             [1.21.6+vmware.1-tkg.1.b3d708a]

(lookf or the 'TKR COMPATIBILE' field)
````

The Official VMware Documentation related to Updating Supervisor Clusters and Tanzu Kubernetes Clusters can be found here:
https://docs.vmware.com/en/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-E491159F-645F-4810-B9A0-8B19AF3E9219.html


