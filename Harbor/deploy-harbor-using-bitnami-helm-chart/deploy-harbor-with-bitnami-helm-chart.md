
This file describes the steps and way Harbor was installed as part of the ITQ customer vSphere with Tanzu POC


# References

https://github.com/goharbor/harbor-helm

# Carvel or Helm?

VMware is using Carvel to supply, deploy and lifecycle their own projects inside their Kubernetes distributions. 
The Harbor package is fully supported if used in TKGs and TKGm 

This guys was written at a time when the Carvel package was not very mature yet.  The Carvel package is recommended for TKGm and TKG 2.0. 
For TKGs, the Bitnami helm chart is still recommended, because of sketchy kapp-controller support in TKGs

For TKGm, see: (check for latest version)
https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/1.5/vmware-tanzu-kubernetes-grid-15/GUID-packages-harbor-registry.html

For TCE, see: (check for latest version)
https://tanzucommunityedition.io/docs/v0.12/package-readme-harbor-2.4.2/


# Which Helm chart to use?

Helm charts for applications can be provided by the company or project that makes the product you wish to install.
Or it can be made my a third party. 

In the case of Harbor, there are 2 Helm charts we can use.

The Harbor Project has provided their own helm chart here: https://github.com/goharbor/harbor-helm
But an alternative helm chart is managed by Bitnami, which is here: https://github.com/bitnami/charts/tree/master/bitnami/harbor/


VMware has bought Bitnami, and also provides most of the engineering behind Harbor.
So with a Tanzu License, either of these Helm charts is supported by VMware. 




# Prepare Helm


Harbor is deployed using a Helm chart. 
We installed the Helm (v3) tool on the Linux jump server

we provide a values override file (helm-override-values-customer.yml) for any customizations on the default Helm chart values

We only need to override these things:
- How the application is exposed (on what internal IP in the VIP range)
- What external URL will be used to access Harbor

helm-harbor-bitnami-override-values-customer.yml
```
service:
 loadBalancerIP: "10.1.1.20" #This is an address in the VIP range of the Load-Balancer you are using in kubernetes. 
 tls:
  commonName: "harborpoc.customerdomain.local"
externalURL: "harborpoc.customerdomain.local"
  ```


# Install Harbor

helm install -f helm-harbor-bitnami-override-values-customer.yml harbor bitnami/harbor

```
[DevOpsAdmin1@customerdomain.local@linuxjumpbox helm]$ helm install -f helm-harbor-bitnami-override-values-customer.yml harbor bitnami/harbor
NAME: harbor
LAST DEPLOYED: Fri May  7 15:55:20 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
** Please be patient while the chart is being deployed **

1. Get the Harbor URL:

  NOTE: It may take a few minutes for the LoadBalancer IP to be available.
        Watch the status with: 'kubectl get svc --namespace default -w harbor'
  export SERVICE_IP=$(kubectl get svc --namespace default harbor --template "{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}")
  echo "Harbor URL: http://$SERVICE_IP/"

2. Login with the following credentials to see your Harbor application

  echo Username: "admin"
  echo Password: $(kubectl get secret --namespace default harbor-core-envvars -o jsonpath="{.data.HARBOR_ADMIN_PASSWORD}" | base64 --decode)

```


## Harbor Default Admin Logon

IP Address: 10.0.1.20 - harborpoc.customerdomain.local
https://harborpoc.customerdomain.local


The Bitnami Helm chart generates a random password for the admin account when Harbor is installed the first time.

It stores this password in the following Kubernetes secret: harbor-core-envvars

You can retrieve the password using this command (you must be cluster admin, or otherwise have the permission to read secrets in the namespace that Harbor was deployed in)
```
kubectl get secret --namespace default harbor-core-envvars -o jsonpath="{.data.HARBOR_ADMIN_PASSWORD}" | base64 --decode
```

Under some circumstances, the default login can sometimes fail, if the internal routing between the Harbor components gets confused. See this for more details: 
https://github.com/goharbor/harbor-helm/issues/589


THe password is then rememberd and saved in PVC (probably the postgres database), so as long as you dont remove the PVC's, password stays the same. Otherwise, retrieve it from k8s secret again. 



## Persistant Storage (PVC)

The Helm chart creates several different Persistent Volume Claims (PVC) for the various components of Harbor. 

Make sure there is enough disk space in theNamespace quota, for the storage that you are planning to use (default in this poc is: storageclass-tanzu-persistantvolumes)

If you run out of disk space, the creation of the Storage Claims will hang on 'pending'.  In k8s describe , or Octant, you can see an error message is annotated to the storage claim that indicates the storage quota was exceeded. 





## Uninstall Harbor - Clean Up

`helm uninstall harbor bitnami/harbor`


If you also want to delete all persistent data: 

(warning: you will have to start from scratch, no config is saved at all)

`kubectl delete pvc --all`



# Prepare vSphere with Tanzu to trust the Harbor registry. 

In order to connect to an external registry, Tanzu clusters must be configured with the certificate of that registry. 

https://docs.vmware.com/en/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-376FCCD1-7743-4202-ACCA-56F214B6892F.html

Once you configure the Tanzu Kubernetes Grid Service for a private registry, any new cluster that is provisioned will support the private registry. For existing clusters to support the private registry, a rolling update is required to apply the TkgServiceConfiguration. See section: 'Update Tanzu Kubernetes Clusters'. In addition, the first time you create a custom TkgServiceConfiguration, the system will initiate a rolling update.


Before changes:

```
[DevOpsAdmin1@customerdomain.local@linuxjumpbox app-examples]$ kubectl describe TkgServiceConfiguration
Name:         tkg-service-configuration
Namespace:
Labels:       <none>
Annotations:  <none>
API Version:  run.tanzu.vmware.com/v1alpha1
Kind:         TkgServiceConfiguration
Metadata:
  Creation Timestamp:  2021-05-03T13:10:07Z
  Generation:          1
  Managed Fields:
    API Version:  run.tanzu.vmware.com/v1alpha1
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .:
          f:kubectl.kubernetes.io/last-applied-configuration:
      f:spec:
        .:
        f:defaultCNI:
    Manager:         kubectl-client-side-apply
    Operation:       Update
    Time:            2021-05-03T13:10:07Z
  Resource Version:  2414
  Self Link:         /apis/run.tanzu.vmware.com/v1alpha1/tkgserviceconfigurations/tkg-service-configuration
  UID:               271776ff-2130-402d-93d4-c6c949a982c0
Spec:
  Default CNI:  antrea
Events:         <none>
```

Apply changes:

[DevOpsAdmin1@customerdomain.local@linuxjumpbox harbor]$ kubectl apply -f add-harbor-cert-to-TkgServiceConfiguration.yml
tkgserviceconfiguration.run.tanzu.vmware.com/tkg-service-configuration configured


TkgServiceConfiguration is only re-applied to existing nodes under certain circumstances. The easiest way to force TkgServiceConfiguration and any new external registry certs to be pushed to nodes, is to force a cluster re-scale. 

Add or subtract a worker node using the total worker node count. This will force a config refresh on all nodes, including the contents of TkgServiceConfiguration



## Check of certs have been added to the nodes: 

Use the procedure to log into TKC nodes with SSH using the procedure here: 
https://docs.vmware.com/en/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-37DC1DF2-119B-4E9E-8CA6-C194F39DDEDA.html


When SSH-ed into the TKC node, check the directory

/etc/ssl/certs/

You should see a file called 'tkg-harborpoc-ca.pem'

This should mean that the harbor registry is now trusted to make TLS (SSL) connections to. 


# Using Harbor


## Set up Harbor Project

A test project has been created in Harbor called tanzupoc

The three test users have been added to Harbor with local accounts, and given rights on the tanzupoc project

## Set up docker on Linux Host

Docker is installed on the Linux jumpbox. We will use Docker to pull a public image, and then place it in Harbor. 



https://docs.docker.com/engine/install/linux-postinstall/


Enable Docker

```
sudo systemctl start docker
sudo systemctl enable docker 
```

Create Docker group and Add the logged on use to the Docker execution group so you can connect the Docker client ot the Docker Daemon


```
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker 
```

Test docker
```
docker run hello-world

[DevOpsAdmin1@customerdomain.local@linuxjumpbox ~]$ docker run hello-world
Unable to find image 'hello-world:latest' locally
Trying to pull repository docker.io/library/hello-world ...
latest: Pulling from docker.io/library/hello-world
b8dfde127a29: Pull complete
Digest: sha256:f2266cbfc127c960fd30e76b7c792dc23b588c0db76233517e1891a4e357d519
Status: Downloaded newer image for docker.io/hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

[DevOpsAdmin1@customerdomain.local@linuxjumpbox ~]$

```

## Retrieve the certificate from Harbor

We need to get the certificate from Harbor. 

We need to import this certificate into the Linux jumpbox cert list, so that Docker will trust Harbor

Also, we will need to import this certificate into 'vSphere with Tanzu', so that our Tanzu clusters will trust Harbor


```
echo -n | openssl s_client -connect harborpoc.customerdomain.local:443 \
    | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p'
```    

```[DevOpsAdmin1@customerdomain.local@linuxjumpbox ~]$ echo -n | openssl s_client -connect harborpoc.customerdomain.local:443 \
>     | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p'
depth=0 CN = harborpoc.customerdomain.local
verify error:num=20:unable to get local issuer certificate
verify return:1
depth=0 CN = harborpoc.customerdomain.local
verify error:num=21:unable to verify the first certificate
verify return:1
DONE
-----BEGIN CERTIFICATE-----
MIIDCAjCgAwIBAgIQHzSiohOeAE/E2AbQPjANBgkqhk9w0BAQsFADAU

<example certificate> 

Q9hLim1CiUUAqJ56fhz2n0StURETh0Vsi6+U+ZL4FFmNn0VSk6kPit
CKg+WPNcuK12A1sf0ime4LzoQ==
-----END CERTIFICATE-----
```

Save cert locally as a file harborpoc.crt: 


```
echo -n | openssl s_client -connect harborpoc.customerdomain.local:443 | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > harborpoc.crt
```

# Copy cert to docker config

Docker maintains its own cert store, where you can add your own private registry certificates

```
sudo mkdir -p /etc/docker/certs.d/harborpoc.customerdomain.local
sudo cp harborpoc.crt /etc/docker/certs.d/harborpoc.customerdomain.local
sudo systemctl restart docker
```




# Log in to Harbor using Docker

docker login harborpoc.customerdomain.local

```
[DevOpsAdmin1@customerdomain.local@linuxjumpbox helm]$ docker login harborpoc.customerdomain.local
Username: devopsadmin1.harbor
Password:
Login Succeeded
```


# Pull image from public registry, re-tag it, and push it to harbor

Pull example image from dockerhub

```docker pull nginx```

We are going to tag and rename our image for the 'tanzupoc' project that we made in Harbor. 

```
 docker tag docker.io/nginx harborpoc.customerdomain.local/tanzupoc/nginx-tanzupoc:v1

 [DevOpsAdmin1@customerdomain.local@linuxjumpbox app-examples]$ docker image list
REPOSITORY                                         TAG                 IMAGE ID            CREATED             SIZE
docker.io/nginx                                    latest              62d49f9bab67        3 weeks ago         133 MB
harborpoc.customerdomain.local/tanzupoc/nginx-tanzupoc   v1                  62d49f9bab67        3 weeks ago         133 MB
docker.io/hello-world                              latest              d1165f221234        2 months ago        13.3 kB
```

now that the image has been renamed and tagged with our private repo path, we can push it. 

```[DevOpsAdmin1@customerdomain.local@linuxjumpbox app-examples]$ docker push harborpoc.customerdomain.local/tanzupoc/nginx-tanzupoc:v1
The push refers to a repository [harborpoc.customerdomain.local/tanzupoc/nginx-tanzupoc]
64ee8c6d0de0: Pushed
974e9faf62f1: Pushed
15aac1be5f02: Pushed
23c959acc3d0: Pushed
4dc529e519c4: Pushed
7e718b9c0c8c: Pushed
v1: digest: sha256:42bba58a1c5a6e2039af02302ba06ee66c446e9547cbfb0da33f4267638cdb53 size: 1570
```

We can now view the image in the docker registry



## Create a pull secret in kubernetes using our docker login token

We will re-use the token that we where given from our Docker login to harbor, to make a secret in Kubernetes, that deployments can also use to pull the image from Harbor. 

https://vnuggets.com/2019/11/06/pks-getting-started-part8-harbor-registry/
https://kubernetes.io/docs/concepts/configuration/secret/#docker-config-secrets


Our Harbor auth for docker can be found here: 

```
[DevOpsAdmin1@customerdomain.local@linuxjumpbox app-examples]$ cat ~/.docker/config.json
{
        "auths": {
                "harborpoc.customerdomain.local": {
                        "auth": "cmtsb29zdGVyaHVpcy5oYXJib3I6Vm13YXJlMTIzIQ=="
                }
        }
}[DevOpsAdmin1@customerdomain.local@linuxjumpbox app-examples]$
```

kubectl create secret generic harborcred-devopsadmin1 --from-file=.dockerconfigjson=docker_config.json --type=kubernetes.io/dockerconfigjson


## Create a Harbor robot account for k8s deployments to use

https://goharbor.io/docs/2.2.0/working-with-projects/project-configuration/create-robot-accounts/

```
Created 'robot$tanzupoc+tazupoc-robot' successfully.
This is the only time to copy this secret.You won't have another opportunity
Name robot$tanzupoc+tazupoc-robot
Token xrRX9<exampletoken>BUZyb5kJv6
```

Check if you can login to harbor from docker, using this secret. 

## Create Kuberenetes Secret to pull images in deployments for a harbor robot account

(make sure to put the username is quotes)

```
kubectl create secret docker-registry harborcred-tazupoc-robot --docker-server=harborpoc.customerdomain.local --docker-username='robot$tanzupoc+tazupoc-robot' --docker-password=xrRX9<exampletoken>BUZyb5kJv6 --docker-email=test@test.com
```

Double-check the contents of the secret:

```
kubectl get secret harborcred-tazupoc-robot --output="jsonpath={.data.\.dockerconfigjson}" | base64 --decode
{"auths":{"harborpoc.customerdomain.local":{"username":"robot$tanzupoc+tazupoc-robot","password":"xrRX9<exampletoken>BUZyb5kJv6 ","email":"test@test.com","auth":"cm9ib3Q<exampleoutput>TQThKQlVaeWI1a0p2Ng=="}}}
```


## Reference the harbor robot secret in a deployment spec

Adding a reference to secret to a deloyment, would ook like this:

```
spec:
  containers:
    - image: harborpoc.customerdomain.local/tanzupoc/nginx-tanzupoc
      name: image-nginx-tanzupoc
      volumeMounts:
        - name: nginx-tanzupoc-webcontent
          mountPath: /usr/share/nginx/html
  imagePullSecrets:
  - name: harborcred-tazupoc-robot #the name of the specific secret we made for the harbor robot account log in
  volumes:
    - name: nginx-tanzupoc-webcontent
      persistentVolumeClaim:
        claimName: claim-nginx-tanzupoc-webcontent

```

## Test Deployment using a harbor image pull

```
kubectl apply -f nginx-demo-pvc-harbor.yml
pod/nginx-tanzupoc created
persistentvolumeclaim/claim-nginx-tanzupoc-webcontent created
service/lb-nginx-tanzupoc created
```


check the details of the pod to see if it successfully pulled the image and started. At the bottom, in 'events', you should see if it pulled the image successfully. In any case, the pod will be able to start if it pulled the image correctly. 


```
kubectl describe pod nginx-tanzupoc
Name:         nginx-tanzupoc
Namespace:    default
Priority:     0
Node:         tkgs-infra-services-workers-zfxbr-7c488875f7-2smph/10.6.169.131
Start Time:   Mon, 10 May 2021 00:31:44 +0200
Labels:       name=nginx-tanzupoc
Annotations:  kubernetes.io/psp: vmware-system-privileged
Status:       Running
IP:           192.168.8.7
IPs:
  IP:  192.168.8.7
Containers:
  image-nginx-tanzupoc:
    Container ID:   containerd://d60291b44a97d49c756f11cc05325b7aa45fe14ad861e69e9d8d40a83fe545c4
    Image:          harborpoc.customerdomain.local/tanzupoc/nginx-tanzupoc:v1
    Image ID:       harborpoc.customerdomain.local/tanzupoc/nginx-tanzupoc@sha256:42bba58a1c5a6e2039af02302ba06ee66c446e9547cbfb0da33f4267638cdb53
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Mon, 10 May 2021 00:32:31 +0200
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /usr/share/nginx/html from nginx-tanzupoc-webcontent (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-jlf8n (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  nginx-tanzupoc-webcontent:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  claim-nginx-tanzupoc-webcontent
    ReadOnly:   false
  default-token-jlf8n:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-jlf8n
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason                  Age   From                                                         Message
  ----     ------                  ----  ----                                                         -------
  Warning  FailedScheduling        59s   default-scheduler                                            persistentvolumeclaim "claim-nginx-tanzupoc-webcontent" not found
  Warning  FailedScheduling        59s   default-scheduler                                            0/7 nodes are available: 3 node(s) had taint {node-role.kubernetes.io/master: }, that the pod didn't tolerate, 4 pod has unbound immediate PersistentVolumeClaims.
  Warning  FailedScheduling        59s   default-scheduler                                            0/7 nodes are available: 3 node(s) had taint {node-role.kubernetes.io/master: }, that the pod didn't tolerate, 4 pod has unbound immediate PersistentVolumeClaims.
  Normal   Scheduled               56s   default-scheduler                                            Successfully assigned default/nginx-tanzupoc to tkgs-infra-services-workers-zfxbr-7c488875f7-2smph
  Normal   SuccessfulAttachVolume  55s   attachdetach-controller                                      AttachVolume.Attach succeeded for volume "pvc-b3942503-9196-44f2-8c34-12c2164b3704"
  Normal   Pulling                 48s   kubelet, tkgs-infra-services-workers-zfxbr-7c488875f7-2smph  Pulling image "harborpoc.customerdomain.local/tanzupoc/nginx-tanzupoc:v1"
  Normal   Pulled                  10s   kubelet, tkgs-infra-services-workers-zfxbr-7c488875f7-2smph  Successfully pulled image "harborpoc.customerdomain.local/tanzupoc/nginx-tanzupoc:v1"
  Normal   Created                 9s    kubelet, tkgs-infra-services-workers-zfxbr-7c488875f7-2smph  Created container image-nginx-tanzupoc
  Normal   Started                 9s    kubelet, tkgs-infra-services-workers-zfxbr-7c488875f7-2smph  Started container image-nginx-tanzupoc
```








# Issues Encountered


## Forgot ExternalURL entry in Helm Overrides


```
Error response from daemon: Get https://harborpoc.customerdomain.local/v2/: Get https://core.harbor.domain/service/token?account=devopsadmin1.harbor&client_id=docker&offline_                                                    token=true&service=harbor-registry: dial tcp: lookup core.harbor.domain: no such host
```


Forgot to replace entries in helm that still try core.harbor.domain  


## Harbor Robot username needs to be in quotes

The Harbor Robot account includes a '$' sign that is not parsed correct by k8s if you do not quote it out. 
