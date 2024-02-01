# Background

This readme describes testings with nephio. The setup is based on tutorial available on nephio.org [Learning with Nephio R1 - Episode 4 - Building a Demo Environment](https://wiki.nephio.org/display/HOME/Learning+with+Nephio+R1+-+Episode+4+-+Building+a+Demo+Environment).

## Preparation

Create an UBUNTU VM with the requirements - that's very straightforward with multipass -
-    Linux Flavour: Ubuntu-20.04-focal
-    8 cores
-    32 GB memory
-    200 GB disk size
-    Default user with sudo passwordless permissions

```
multipass launch --name ubuntu-vm --mem 32G --disk 200G --cpus 8 focal
```
```
root@fiveg-host-24-node4:~# multipass list
Name                    State             IPv4             Image
ubuntu-vm               Running           10.65.94.183     Ubuntu 20.04 LTS
```

Installation as per tutorial. This just works.
```
wget -O - https://raw.githubusercontent.com/nephio-project/test-infra/v1.0.1/e2e/provision/init.sh |  \
sudo NEPHIO_DEBUG=false   \
     NEPHIO_BRANCH=v1.0.1 \
     NEPHIO_USER=ubuntu   \
     bash
```
Add port redirection on the host that host the VM (10.65.94.183 is the VM IP) to access UI for nephio and gitea:
```
ssh ubuntu@10.65.94.183               -L *:7007:localhost:7007                 -L *:3000:172.18.0.200:3000                 kubectl port-forward --namespace=nephio-webui svc/nephio-webui 7007
```
For some reason, the nephio UI does not work (port 7007). 
But, this is ok for gitea (port 3000).
After creating an account in gitea, we can access to the gitea repos and see the imported packages. The below capture has been taken during the deployment, hence we already can see a copy of repos (regional, edge etc.) which control the deployment of workloads running in each cluster. 

![image](https://github.com/robric/nephio-testings/assets/21667569/1e7502f6-b986-45ce-a715-560154f1da7d)


The declarative definition of the deployment (infra, pods, NF config) is a a  cornerstone of the nephio vision. It is all exposed via  KRM (Kubernetes Resource Model). Here gitea is a local repo, which stores the data related to the deployment.

## What you get from there

After installation you get a cluster working with a set of tooling that are part of nephio's core install:

*    backend-system: This  includes backend services for Nephio, such as the resource-backend-controller, which would manage backend resources specific to the Nephio platform.
*    capd-system, capi-kubeadm-bootstrap-system, capi-kubeadm-control-plane-system, capi-system: These namespaces contain components of the Cluster API (CAPI), which Nephio uses to create, configure, and manage Kubernetes clusters. CAPD is specifically for Docker-based local development environments.
*   cert-manager: This is a Kubernetes add-on that issues and manages TLS certificates, ensuring encrypted communication within the cluster. It's essential for securing communications in a Nephio-managed cluster.
*   config-management-monitoring and config-management-system: These namespaces are likely part of a configuration management and monitoring solution integrated with Nephio to maintain desired state and monitor the health of the system.
*    gitea: This lightweight Git server is probably used for version control of configuration files and code within the Nephio environment.
*    kube-system: This namespace contains the core components of Kubernetes, which are fundamental to any Kubernetes cluster, including a Nephio-managed one.
*    local-path-storage: Provides storage solutions for the cluster, which Nephio would use to store persistent data.
*   metallb-system: MetalLB provides load balancing services, which are crucial for exposing services in a Kubernetes cluster managed by Nephio, especially in environments without a cloud provider's load balancer.
*   nephio-system: This namespace contains Nephio-specific controllers like the nephio-controller and token-controller, which are central to the Nephio platform's functionality.
*    nephio-webui: Hosts the web-based user interface for Nephio, allowing users to interact with the Nephio platform through a web browser.
*    network-config: Manages network configurations, which is particularly important in telecom environments where Nephio is targeted.
*    porch-system: package management in  Nephio (kpt)
*    resource-group-system: Manages logical groupings of resources within the Nephio platform, allowing for better organization and management of cluster resources.

Overall, the software installed in this Nephio test environment includes a mix of core Kubernetes components, Cluster API for cluster lifecycle management, security and configuration management tools, storage solutions, and Nephio-specific services for managing Kubernetes in telecom environments.

```
ubuntu@ubuntu-vm:~$ kubectl get pods -A
NAMESPACE                           NAME                                                            READY   STATUS    RESTARTS      AGE
backend-system                      resource-backend-controller-b7c895858-kd4fr                     2/2     Running   0             15h
capd-system                         capd-controller-manager-c479754b7-slc7x                         1/1     Running   0             15h
capi-kubeadm-bootstrap-system       capi-kubeadm-bootstrap-controller-manager-bcdfbf4c5-h4pjj       1/1     Running   0             15h
capi-kubeadm-control-plane-system   capi-kubeadm-control-plane-controller-manager-b9485b857-vmnvk   1/1     Running   0             15h
capi-system                         capi-controller-manager-9d9548dc8-rqqlt                         1/1     Running   0             15h
cert-manager                        cert-manager-7476c8fcf4-sm4nj                                   1/1     Running   0             15h
cert-manager                        cert-manager-cainjector-bdd866bd4-xpqlq                         1/1     Running   0             15h
cert-manager                        cert-manager-webhook-5655dcfb4b-2v7jl                           1/1     Running   0             15h
config-management-monitoring        otel-collector-798c8784bd-s7rwm                                 1/1     Running   0             15h
config-management-system            config-management-operator-6946b77565-s6p7r                     1/1     Running   0             15h
config-management-system            reconciler-manager-5b5d8557-hxh8r                               2/2     Running   0             15h
config-management-system            root-reconciler-mgmt-6fdf94dfd4-xz85d                           4/4     Running   0             15h
gitea                               gitea-0                                                         1/1     Running   0             15h
gitea                               gitea-memcached-6777864fbd-zdwkd                                1/1     Running   0             15h
gitea                               gitea-postgresql-0                                              1/1     Running   0             15h
kube-system                         coredns-5d78c9869d-kfkn4                                        1/1     Running   0             15h
kube-system                         coredns-5d78c9869d-prcml                                        1/1     Running   0             15h
kube-system                         etcd-kind-control-plane                                         1/1     Running   0             15h
kube-system                         kindnet-9qtgk                                                   1/1     Running   0             15h
kube-system                         kube-apiserver-kind-control-plane                               1/1     Running   0             15h
kube-system                         kube-controller-manager-kind-control-plane                      1/1     Running   0             15h
kube-system                         kube-proxy-lg52d                                                1/1     Running   0             15h
kube-system                         kube-scheduler-kind-control-plane                               1/1     Running   0             15h
local-path-storage                  local-path-provisioner-6bc4bddd6b-hm9d7                         1/1     Running   0             15h
metallb-system                      controller-7948676b95-kgh9v                                     1/1     Running   0             15h
metallb-system                      speaker-wq57f                                                   1/1     Running   0             15h
nephio-system                       nephio-controller-58769ddfc8-nb8cj                              2/2     Running   0             15h
nephio-system                       token-controller-798d6f6d8-tdtqm                                2/2     Running   0             15h
nephio-webui                        nephio-webui-5464c8dc64-ztkfn                                   1/1     Running   0             15h
network-config                      network-config-controller-5b5b45cb76-lnhdk                      2/2     Running   0             15h
porch-system                        function-runner-7c5766c7d6-6f7zc                                1/1     Running   0             15h
porch-system                        function-runner-7c5766c7d6-pxf9z                                1/1     Running   0             15h
porch-system                        porch-controllers-c694cd64d-nv7qb                               1/1     Running   0             15h
porch-system                        porch-server-6b5f89996b-wjhmm                                   1/1     Running   9 (97m ago)   15h
resource-group-system               resource-group-controller-manager-6c9d56d88-6n7ns               3/3     Running   0             15h

And we have plenty of CRDs to manage the different softwares components:

ubuntu@ubuntu-vm:~$ kubectl get crds
NAME                                                         CREATED AT
addresspools.metallb.io                                      2024-01-08T18:45:17Z
amfdeployments.workload.nephio.org                           2024-01-08T18:48:33Z
bfdprofiles.metallb.io                                       2024-01-08T18:45:17Z
bgpadvertisements.metallb.io                                 2024-01-08T18:45:17Z
bgppeers.metallb.io                                          2024-01-08T18:45:17Z
capacities.req.nephio.org                                    2024-01-08T18:48:33Z
certificaterequests.cert-manager.io                          2024-01-08T18:45:12Z
certificates.cert-manager.io                                 2024-01-08T18:45:12Z
challenges.acme.cert-manager.io                              2024-01-08T18:45:12Z
clusterclasses.cluster.x-k8s.io                              2024-01-08T18:46:25Z
clustercontexts.infra.nephio.org                             2024-01-08T18:48:33Z
clusterissuers.cert-manager.io                               2024-01-08T18:45:12Z
clusterresourcesetbindings.addons.cluster.x-k8s.io           2024-01-08T18:46:25Z
clusterresourcesets.addons.cluster.x-k8s.io                  2024-01-08T18:46:25Z
clusters.cluster.x-k8s.io                                    2024-01-08T18:46:25Z
clusters.clusterregistry.k8s.io                              2024-01-08T18:49:05Z
clusterselectors.configmanagement.gke.io                     2024-01-08T18:49:05Z
communities.metallb.io                                       2024-01-08T18:45:17Z
configmanagements.configmanagement.gke.io                    2024-01-08T18:48:54Z
datanetworknames.req.nephio.org                              2024-01-08T18:48:33Z
datanetworks.req.nephio.org                                  2024-01-08T18:48:34Z
dockerclusters.infrastructure.cluster.x-k8s.io               2024-01-08T18:46:44Z
dockerclustertemplates.infrastructure.cluster.x-k8s.io       2024-01-08T18:46:44Z
dockermachinepools.infrastructure.cluster.x-k8s.io           2024-01-08T18:46:45Z
dockermachines.infrastructure.cluster.x-k8s.io               2024-01-08T18:46:45Z
dockermachinetemplates.infrastructure.cluster.x-k8s.io       2024-01-08T18:46:45Z
endpoints.inv.nephio.org                                     2024-01-08T18:45:11Z
extensionconfigs.runtime.cluster.x-k8s.io                    2024-01-08T18:46:25Z
hierarchyconfigs.configmanagement.gke.io                     2024-01-08T18:49:05Z
interfaces.req.nephio.org                                    2024-01-08T18:48:34Z
ipaddressclaims.ipam.cluster.x-k8s.io                        2024-01-08T18:46:25Z
ipaddresses.ipam.cluster.x-k8s.io                            2024-01-08T18:46:25Z
ipaddresspools.metallb.io                                    2024-01-08T18:45:17Z
ipclaims.ipam.resource.nephio.org                            2024-01-08T18:45:11Z
ipprefixes.ipam.resource.nephio.org                          2024-01-08T18:45:11Z
issuers.cert-manager.io                                      2024-01-08T18:45:12Z
kubeadmconfigs.bootstrap.cluster.x-k8s.io                    2024-01-08T18:46:25Z
kubeadmconfigtemplates.bootstrap.cluster.x-k8s.io            2024-01-08T18:46:25Z
kubeadmcontrolplanes.controlplane.cluster.x-k8s.io           2024-01-08T18:46:25Z
kubeadmcontrolplanetemplates.controlplane.cluster.x-k8s.io   2024-01-08T18:46:26Z
l2advertisements.metallb.io                                  2024-01-08T18:45:17Z
links.inv.nephio.org                                         2024-01-08T18:45:11Z
machinedeployments.cluster.x-k8s.io                          2024-01-08T18:46:26Z
machinehealthchecks.cluster.x-k8s.io                         2024-01-08T18:46:26Z
machinepools.cluster.x-k8s.io                                2024-01-08T18:46:26Z
machines.cluster.x-k8s.io                                    2024-01-08T18:46:26Z
machinesets.cluster.x-k8s.io                                 2024-01-08T18:46:26Z
namespaceselectors.configmanagement.gke.io                   2024-01-08T18:49:05Z
networkconfigs.infra.nephio.org                              2024-01-08T18:48:34Z
networkinstances.ipam.resource.nephio.org                    2024-01-08T18:45:12Z
networks.config.nephio.org                                   2024-01-08T18:48:34Z
networks.infra.nephio.org                                    2024-01-08T18:48:34Z
nodes.inv.nephio.org                                         2024-01-08T18:45:12Z
orders.acme.cert-manager.io                                  2024-01-08T18:45:12Z
packagerevs.config.porch.kpt.dev                             2024-01-08T18:47:16Z
packagevariants.config.porch.kpt.dev                         2024-01-08T18:47:16Z
packagevariantsets.config.porch.kpt.dev                      2024-01-08T18:47:16Z
rawtopologies.topo.nephio.org                                2024-01-08T18:45:12Z
repositories.config.porch.kpt.dev                            2024-01-08T18:47:16Z
repositories.infra.nephio.org                                2024-01-08T18:48:34Z
reposyncs.configsync.gke.io                                  2024-01-08T18:49:05Z
resourcegroups.kpt.dev                                       2024-01-08T18:45:01Z
rootsyncs.configsync.gke.io                                  2024-01-08T18:48:54Z
smfdeployments.workload.nephio.org                           2024-01-08T18:48:34Z
targets.inv.nephio.org                                       2024-01-08T18:45:12Z
tokens.infra.nephio.org                                      2024-01-08T18:48:34Z
upfdeployments.workload.nephio.org                           2024-01-08T18:48:34Z
vlanclaims.vlan.resource.nephio.org                          2024-01-08T18:45:12Z
vlanindices.vlan.resource.nephio.org                         2024-01-08T18:45:12Z
vlans.vlan.resource.nephio.org                               2024-01-08T18:45:12Z
workloadclusters.infra.nephio.org                            2024-01-08T18:48:34Z
ubuntu@ubuntu-vm:~$ 
```

The setup is based on kind so we can have multiple clusters. At first there is a single node (control plane).

```
ubuntu@ubuntu-vm:~$ kind get nodes
kind-control-plane
ubuntu@ubuntu-vm:~$

ubuntu@ubuntu-vm:~$ kubectl get nodes
NAME                 STATUS   ROLES           AGE   VERSION
kind-control-plane   Ready    control-plane   22h   v1.27.1

ubuntu@ubuntu-vm:~$ kubectl get clusters
No resources found in default namespace.
ubuntu@ubuntu-vm:~$
```

## First time use

### get packages (regional) via kpt

Again this is taken from: [official link](https://github.com/nephio-project/docs/blob/main/user-guide/exercises.md)

The idea to have multiple clusters with 
- 1 regional cluster
- 2 edge clusters


There are few packages already installed. 

```
ubuntu@ubuntu-vm:~$ kubectl get repositories.config.porch.kpt.dev
NAME                      TYPE   CONTENT   DEPLOYMENT   READY   ADDRESS
free5gc-packages          git    Package   false        True    https://github.com/nephio-project/free5gc-packages.git
mgmt                      git    Package   true         True    http://172.18.0.200:3000/nephio/mgmt.git
mgmt-staging              git    Package   false        True    http://172.18.0.200:3000/nephio/mgmt-staging.git
nephio-example-packages   git    Package   false        True    https://github.com/nephio-project/nephio-example-packages.git

This is actually the same output as the one that gets kpt repos

ubuntu@ubuntu-vm:~$ kpt  alpha repo  get
NAME                      TYPE   CONTENT   DEPLOYMENT   READY   ADDRESS
free5gc-packages          git    Package   false        True    https://github.com/nephio-project/free5gc-packages.git
mgmt                      git    Package   true         True    http://172.18.0.200:3000/nephio/mgmt.git
mgmt-staging              git    Package   false        True    http://172.18.0.200:3000/nephio/mgmt-staging.git
nephio-example-packages   git    Package   false        True    https://github.com/nephio-project/nephio-example-packages.git

```

We're having tons of packages there...  These are remote packages (see rpkg in the command) that we want to clone locally (gitea).

```
ubuntu@ubuntu-vm:~$ kpt alpha rpkg get
NAME                                                               PACKAGE                              WORKSPACENAME   REVISION   LATEST   LIFECYCLE   REPOSITORY
free5gc-packages-148a8446856cfcba9ba53f9c5786bacb835f3733          free5gc-cp                           main            main       false    Published   free5gc-packages
free5gc-packages-daba5a241a23af3ce777b256c39dcedc3c5e3917          free5gc-cp                           v1              v1         true     Published   free5gc-packages
free5gc-packages-0173ec955eeb21c2bebdf53699d969295bbd17a6          free5gc-operator                     main            main       false    Published   free5gc-packages
free5gc-packages-9f723b4dd4dd0d1078da499ab71f11f54b6f4bea          free5gc-operator                     v1              v1         false    Published   free5gc-packages
[...]
ubuntu@ubuntu-vm:~$ kpt alpha rpkg get --name nephio-workload-cluster
NAME                                                               PACKAGE                   WORKSPACENAME   REVISION   LATEST   LIFECYCLE   REPOSITORY
nephio-example-packages-05707c7acfb59988daaefd85e3f5c299504c2da1   nephio-workload-cluster   main            main       false    Published   nephio-example-packages
[...]
nephio-example-packages-48cea934a3bd876b775099ab59e7c12456888ffd   nephio-workload-cluster   v9              v9         true     Published   nephio-example-packages
...
```
We can clone and pull the remote package:
-  kpt alpha rpkg clone: clone of a package from a repo to a target repository
-  kpt alpha rpkg pull: makes a local copy on the package locally 
```
ubuntu@ubuntu-vm:~$ kpt alpha rpkg clone -n default nephio-example-packages-48cea934a3bd876b775099ab59e7c12456888ffd --repository mgmt regional
mgmt-08c26219f9879acdefed3469f8c3cf89d5db3868 created
ubuntu@ubuntu-vm:~$ kpt alpha rpkg pull -n default mgmt-08c26219f9879acdefed3469f8c3cf89d5db3868 regional
```
So we have a new local  directory of the remote package (this is the effect of the kpt alpha rpkg pull  command)

```
ubuntu@ubuntu-vm:~$ ls
nephio.yaml  regional  test-infra
ubuntu@ubuntu-vm:~$ cd regional/
ubuntu@ubuntu-vm:~/regional$ ls -l
total 52
-rw-r--r-- 1 ubuntu ubuntu 880 Jan  9 10:10 Kptfile
-rw-r--r-- 1 ubuntu ubuntu 782 Jan  9 10:10 README.md
-rw-r--r-- 1 ubuntu ubuntu 848 Jan  9 10:10 apply-replacements.yaml
-rw-r--r-- 1 ubuntu ubuntu 283 Jan  9 10:10 package-context.yaml
-rw-r--r-- 1 ubuntu ubuntu 654 Jan  9 10:10 pv-cluster.yaml
-rw-r--r-- 1 ubuntu ubuntu 609 Jan  9 10:10 pv-configsync.yaml
-rw-r--r-- 1 ubuntu ubuntu 594 Jan  9 10:10 pv-kindnet.yaml
-rw-r--r-- 1 ubuntu ubuntu 669 Jan  9 10:10 pv-local-path-provisioner.yaml
-rw-r--r-- 1 ubuntu ubuntu 589 Jan  9 10:10 pv-multus.yaml
-rw-r--r-- 1 ubuntu ubuntu 635 Jan  9 10:10 pv-repo.yaml
-rw-r--r-- 1 ubuntu ubuntu 657 Jan  9 10:10 pv-rootsync.yaml
-rw-r--r-- 1 ubuntu ubuntu 654 Jan  9 10:10 pv-vlanindex.yaml
-rw-r--r-- 1 ubuntu ubuntu 405 Jan  9 10:10 workload-cluster.yaml
ubuntu@ubuntu-vm:~/regional$
```

Use the set-label function against the package.
```
ubuntu@ubuntu-vm:~$ kpt fn eval --image gcr.io/kpt-fn/set-labels:v0.2.0 regional -- "nephio.org/site-type=regional" "nephio.org/region=us-west1"
[RUNNING] "gcr.io/kpt-fn/set-labels:v0.2.0"
[PASS] "gcr.io/kpt-fn/set-labels:v0.2.0" in 2.3s
  Results:
    [info]: set 18 labels in total
ubuntu@ubuntu-vm:~$
```
This kpt fn eval launches a docker "gcr.io/kpt-fn/set-labels"
```
ubuntu@ubuntu-vm:~/regional$ docker image ls
REPOSITORY                 TAG       IMAGE ID       CREATED         SIZE
...
gcr.io/kpt-fn/set-labels   v0.2.0    85f5cf1ba0f4   15 months ago   20.5MB
``` 
Now if we look at any of the files in the /regional folder, they have the new labels  "nephio.org/site-type=regional" "nephio.org/region=us-west1"
```
ubuntu@ubuntu-vm:~/regional$ cat workload-cluster.yaml 
apiVersion: infra.nephio.org/v1alpha1
kind: WorkloadCluster
metadata: # kpt-merge: /example
  name: regional
  annotations:
    internal.kpt.dev/upstream-identifier: 'infra.nephio.org|WorkloadCluster|default|example'
  labels:
    nephio.org/region: us-west1
    nephio.org/site-type: regional
[...]
ubuntu@ubuntu-vm:~/regional$
```
Propose the package for being published
```
ubuntu@ubuntu-vm:~$ kpt alpha rpkg propose -n default mgmt-08c26219f9879acdefed3469f8c3cf89d5db3868
mgmt-08c26219f9879acdefed3469f8c3cf89d5db3868 proposed
ubuntu@ubuntu-vm:~$ 
```
### deploy the regional cluster (via kpt)
Just approve the package, this starts the deployment (kind node creation etc.) 
```
ubuntu@ubuntu-vm:~$ kpt alpha rpkg approve -n default mgmt-08c26219f9879acdefed3469f8c3cf89d5db3868
mgmt-08c26219f9879acdefed3469f8c3cf89d5db3868 approved
ubuntu@ubuntu-vm:~$ 
```
After some time we can see a new 'regional' kind cluster created with a set of nodes. 
```
ubuntu@ubuntu-vm:~$ kind get clusters
kind
regional

ubuntu@ubuntu-vm:~$ kind get nodes -n regional
regional-md-0-72czx-79d7f67568xds4wb-m96hq
regional-v6gkt-br7nq
regional-lb
ubuntu@ubuntu-vm:~$

### A  cluster ressource is created.

ubuntu@ubuntu-vm:~$ kubectl get clusters.cluster.x-k8s.io 
NAME       PHASE         AGE   VERSION
regional   Provisioned   13m   v1.26.3
ubuntu@ubuntu-vm:~$

### The deployment is tracked in "machine" CRDs. This maps with kind nodes (fortunately :-) )

ubuntu@ubuntu-vm:~$ kubectl get machinedeployments.cluster.x-k8s.io 
NAME                  CLUSTER    REPLICAS   READY   UPDATED   UNAVAILABLE   PHASE     AGE   VERSION
regional-md-0-72czx   regional   1          1       1         0             Running   33m   v1.26.3

ubuntu@ubuntu-vm:~$ kubectl get machinesets.cluster.x-k8s.io 
NAME                                   CLUSTER    REPLICAS   READY   AVAILABLE   AGE   VERSION
regional-md-0-72czx-79d7f67568xds4wb   regional   1          1       1           32m   v1.26.3

ubuntu@ubuntu-vm:~$ kubectl get machines
NAME                                         CLUSTER    NODENAME                                     PROVIDERID                                              PHASE     AGE   VERSION
regional-md-0-72czx-79d7f67568xds4wb-m96hq   regional   regional-md-0-72czx-79d7f67568xds4wb-m96hq   docker:////regional-md-0-72czx-79d7f67568xds4wb-m96hq   Running   33m   v1.26.3
regional-v6gkt-br7nq                         regional   regional-v6gkt-br7nq                         docker:////regional-v6gkt-br7nq                         Running   32m   v1.26.3       

ubuntu@ubuntu-vm:~$ 
```
Access the nested cluster via kubectl.
```
ubuntu@ubuntu-vm:~$ kubectl get secret regional-kubeconfig -o jsonpath='{.data.value}' | base64 -d > $HOME/.kube/regional-kubeconfig
ubuntu@ubuntu-vm:~$ export KUBECONFIG=$HOME/.kube/config:$HOME/.kube/regional-kubeconfig

ubuntu@ubuntu-vm:~$ kubectl config get-contexts 
CURRENT   NAME                      CLUSTER     AUTHINFO         NAMESPACE
*         kind-kind                 kind-kind   kind-kind        
          regional-admin@regional   regional    regional-admin   
ubuntu@ubuntu-vm:~$
ubuntu@ubuntu-vm:~$ kubectl config use-context regional-admin@regional 
Switched to context "regional-admin@regional".

ubuntu@ubuntu-vm:~$ kubectl get ns
NAME                           STATUS   AGE
config-management-monitoring   Active   27m
config-management-system       Active   27m
default                        Active   27m
kube-node-lease                Active   27m
kube-public                    Active   27m
kube-system                    Active   27m
local-path-storage             Active   27m
resource-group-system          Active   26m
ubuntu@ubuntu-vm:~$

ubuntu@ubuntu-vm:~$ kubectl get nodes
NAME                                         STATUS   ROLES           AGE   VERSION
regional-md-0-72czx-79d7f67568xds4wb-m96hq   Ready    <none>          25m   v1.26.3
regional-v6gkt-br7nq                         Ready    control-plane   26m   v1.26.3
ubuntu@ubuntu-vm:~$

ubuntu@ubuntu-vm:~$ kubectl get pods -A
NAMESPACE                      NAME                                                 READY   STATUS    RESTARTS   AGE
config-management-monitoring   otel-collector-859586c54-gj5fm                       1/1     Running   0          24m
config-management-system       config-management-operator-684584c5df-crgl8          1/1     Running   0          25m
config-management-system       reconciler-manager-86c5f8cc79-7swvd                  2/2     Running   0          24m
config-management-system       root-reconciler-regional-55dd9c795b-zw9np            4/4     Running   0          24m
kube-system                    coredns-787d4945fb-bz6sk                             1/1     Running   0          25m
kube-system                    coredns-787d4945fb-szpnd                             1/1     Running   0          25m
kube-system                    etcd-regional-v6gkt-br7nq                            1/1     Running   0          25m
kube-system                    kindnet-96qp7                                        1/1     Running   0          25m
kube-system                    kindnet-g7jkl                                        1/1     Running   0          25m
kube-system                    kube-apiserver-regional-v6gkt-br7nq                  1/1     Running   0          25m
kube-system                    kube-controller-manager-regional-v6gkt-br7nq         1/1     Running   0          25m
kube-system                    kube-multus-ds-9ttqz                                 1/1     Running   0          25m
kube-system                    kube-multus-ds-thtrx                                 1/1     Running   0          25m
kube-system                    kube-proxy-czz58                                     1/1     Running   0          25m
kube-system                    kube-proxy-rf5qs                                     1/1     Running   0          25m
kube-system                    kube-scheduler-regional-v6gkt-br7nq                  1/1     Running   0          25m
local-path-storage             local-path-provisioner-5d76447cf7-jpbl8              1/1     Running   0          25m
resource-group-system          resource-group-controller-manager-6cdfc6486d-hrj77   3/3     Running   0          24m

```

### Deploy Edge Clusters

Unlike for the regional clusters, the deployment of edge clusters is all controlled via kubernetes (packagevariant). Let's have a closer look at the deployment.

```
### Apply package (porch) via kubernetes. It will be equivalent to the kpt commands previously seen but managed via kubernetes manifests. The content of the file is below.

ubuntu@ubuntu-vm:~$ kubectl apply -f test-infra/e2e/tests/002-edge-clusters.yaml
packagevariantset.config.porch.kpt.dev/edge-clusters unchanged

ubuntu@ubuntu-vm:~$ cat test-infra/e2e/tests/002-edge-clusters.yaml 
apiVersion: config.porch.kpt.dev/v1alpha2
kind: PackageVariantSet
metadata:
  name: edge-clusters
spec:
  upstream:
    repo: nephio-example-packages
    package: nephio-workload-cluster
    revision: v9
  targets:
  - repositories:
    - name: mgmt
      packageNames:
      - edge01
      - edge02
    template:
      annotations:
        approval.nephio.org/policy: initial
      pipeline:
        mutators:
        - image: gcr.io/kpt-fn/set-labels:v0.2.0
          configMap:
            nephio.org/site-type: edge
            nephio.org/region: us-west1

### We see new source 

ubuntu@ubuntu-vm:~$ kubectl get repositories.
NAME                      TYPE   CONTENT   DEPLOYMENT   READY   ADDRESS
edge01                    git    Package   true         True    http://172.18.0.200:3000/nephio/edge01.git
edge02                    git    Package   true         True    http://172.18.0.200:3000/nephio/edge02.git 
free5gc-packages          git    Package   false        True    https://github.com/nephio-project/free5gc-packages.git
mgmt                      git    Package   true         True    http://172.18.0.200:3000/nephio/mgmt.git
mgmt-staging              git    Package   false        True    http://172.18.0.200:3000/nephio/mgmt-staging.git
nephio-example-packages   git    Package   false        True    https://github.com/nephio-project/nephio-example-packages.git
regional                  git    Package   true         True    http://172.18.0.200:3000/nephio/regional.git
ubuntu@ubuntu-vm:~$

### After some time new clusters are created...

ubuntu@ubuntu-vm:~$ kind get clusters
edge01
edge02
kind
regional

ubuntu@ubuntu-vm:~$ kubectl get machinesets.cluster.x-k8s.io 
NAME                                   CLUSTER    REPLICAS   READY   AVAILABLE   AGE    VERSION
edge01-md-0-5cz45-64b9665598x9b9q9     edge01     1          1       1           2m4s   v1.26.3
edge02-md-0-sq944-54f5949dd4xgkjcf     edge02     1          1       1           75s    v1.26.3
regional-md-0-72czx-79d7f67568xds4wb   regional   1          1       1           46m    v1.26.3
ubuntu@ubuntu-vm:~$
```

Same as previously, we add the edge kubeconfig files for the new clusters and we make sure to have this permanent via .bashrc. 
That allows us to enter any cluster: regional, edge01 and edge 02.

```
kubectl get secret edge01-kubeconfig -o jsonpath='{.data.value}' | base64 -d > $HOME/.kube/edge01-kubeconfig
kubectl get secret edge02-kubeconfig -o jsonpath='{.data.value}' | base64 -d > $HOME/.kube/edge02-kubeconfig
export KUBECONFIG=$HOME/.kube/config:$HOME/.kube/regional-kubeconfig:$HOME/.kube/edge01-kubeconfig:$HOME/.kube/edge02-kubeconfig
echo "export KUBECONFIG=$HOME/.kube/config:$HOME/.kube/regional-kubeconfig:$HOME/.kube/edge01-kubeconfig:$HOME/.kube/edge02-kubeconfig" >> ~/.bashrc
```
Then configure networking to interconnect regional and edges via 3 vlans (2,3,4).
```
./test-infra/e2e/provision/hacks/inter-connect_workers.sh
./test-infra/e2e/provision/hacks/vlan-interfaces.sh
```
Network is managed by containerlab (a new config folder ~/clab-free5gc-net is created). Next, new vlans are created in each of the kind nodes.
```
ubuntu@ubuntu-vm:~$ docker exec -it edge01-md-0-5cz45-64b9665598x9b9q9-8gcp4  ip link
[...]
8: eth1.2@eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether aa:c1:ab:c4:73:75 brd ff:ff:ff:ff:ff:ff
9: eth1.3@eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether aa:c1:ab:c4:73:75 brd ff:ff:ff:ff:ff:ff
10: eth1.4@eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether aa:c1:ab:c4:73:75 brd ff:ff:ff:ff:ff:ff
```
Configure routing backend via nephio via package. Things start to become interesting here.
```
kubectl apply -f test-infra/e2e/tests/003-network.yaml
```
Let's have a closer look !
```
### A resource "packagevariant" is created from repo "nephio-example-packages" using package "network"

ubuntu@ubuntu-vm:~$ cat test-infra/e2e/tests/003-network.yaml
apiVersion: config.porch.kpt.dev/v1alpha1
kind: PackageVariant
metadata:
  name: network
spec:
  upstream:
    repo: nephio-example-packages
    package: network
    revision: v1
  downstream:
    repo: mgmt
    package: network
  annotations:
    approval.nephio.org/policy: initial

### We can quickly find the git source !

ubuntu@ubuntu-vm:~$ kubectl get repositories.
NAME                      TYPE   CONTENT   DEPLOYMENT   READY   ADDRESS
[...]
nephio-example-packages   git    Package   false        True    https://github.com/nephio-project/nephio-example-packages.git
###
```

This is what we have in the "network" package.
![image](https://github.com/robric/nephio-testings/assets/21667569/a98a38c8-798f-49a6-b158-041c85473b0b)
and if we double click on a a given network:
![image](https://github.com/robric/nephio-testings/assets/21667569/a0b61e41-1b9b-4ccf-bd01-9ac9a802d9cb)

As expected this all cloned to gitea following the downstream specs of the PackageVariant resource (repo: mgmt / package: network).

![image](https://github.com/robric/nephio-testings/assets/21667569/f767cca9-b767-4959-8495-8762d2897ead)

nice...  so we end up with the corresponding "networks" CRDs instantiated in our cluster. 

```
ubuntu@ubuntu-vm:~$ kubectl get networks.infra.nephio.org 
NAME           READY
vpc-internal   True
vpc-internet   True
vpc-ran        True
ubuntu@ubuntu-vm:~$

ubuntu@ubuntu-vm:~$ kubectl get networkinstances.ipam.resource.nephio.org
NAME           SYNC   STATUS   NETWORK-INSTANCE   PREFIX0      PREFIX1        PREFIX2      PREFIX3        PREFIX4   AGE
vpc-internal   True   True     vpc-internal       172:1::/32   172.1.0.0/16                                         4d19h
vpc-internet   True   True     vpc-internet       172::/32     172.0.0.0/16   1000::/32    10.0.0.0/8               4d19h
vpc-ran        True   True     vpc-ran            172:2::/32   172.2.0.0/16   172:3::/32   172.3.0.0/16             4d19h
ubuntu@ubuntu-vm:~$ 
```

Finally we execute a script that generates a yaml manifest (003-network-topo.yaml) for the rawtopologies.topo.nephio.org CRD. The script captures the IP address (172.18.0.12) that was  allocated to the leaf router (srlinux).
This actually does not seem to do anything. Maybe this information is provided to show the vision, if so this should be more explicitely mentionned.

```
ubuntu@ubuntu-vm:~$ ./test-infra/e2e/provision/hacks/network-topo.sh
ubuntu@ubuntu-vm:~$ kubectl apply -f test-infra/e2e/tests/003-network-topo.yaml

ubuntu@ubuntu-vm:~$ docker inspect  net-free5gc-net-leaf | grep IPA
[...]
                    "IPAddress": "172.18.0.12",

### This IP will be pushed the rawtopologies manifest/CRD

ubuntu@ubuntu-vm:~$  kubectl get rawtopologies.topo.nephio.org  -o yaml
apiVersion: v1
items:
- apiVersion: topo.nephio.org/v1alpha1
  kind: RawTopology
  metadata:
    annotations:
      kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"topo.nephio.org/v1alpha1","kind":"RawTopology","metadata":{"annotations":{},"name":"nephio","namespace":"default"},"spec":{"links":[{"endpoints":[{"interfaceName":"e1-1","nodeName":"srl"},{"interfaceName":"eth1","nodeName":"edge01"}]},{"endpoints":[{"interfaceName":"e1-2","nodeName":"srl"},{"interfaceName":"eth1","nodeName":"edge02"}]},{"endpoints":[{"interfaceName":"e1-3","nodeName":"srl"},{"interfaceName":"eth1","nodeName":"regional"}]}],"nodes":{"edge01":{"labels":{"nephio.org/cluster-name":"edge01"},"provider":"docker.io"},"edge02":{"labels":{"nephio.org/cluster-name":"edge02"},"provider":"docker.io"},"mgmt":{"provider":"docker.io"},"regional":{"labels":{"nephio.org/cluster-name":"regional"},"provider":"docker.io"},"srl":{"address":"172.18.0.12:57400","provider":"srl.nokia.com"}}}}
    creationTimestamp: "2024-01-26T16:54:18Z"
    finalizers:
    - vlan.nephio.org/finalizer
    generation: 1
    name: nephio
    namespace: default
    resourceVersion: "9497089"
    uid: 29219b5e-4fcf-483a-bc24-9d86ee24365e
  spec:
    links:
    - endpoints:
      - interfaceName: e1-1
        nodeName: srl
      - interfaceName: eth1
        nodeName: edge01
    - endpoints:
      - interfaceName: e1-2
        nodeName: srl
      - interfaceName: eth1
        nodeName: edge02
    - endpoints:
      - interfaceName: e1-3
        nodeName: srl
      - interfaceName: eth1
        nodeName: regional
    nodes:
      edge01:
        labels:
          nephio.org/cluster-name: edge01
        provider: docker.io
      edge02:
        labels:
          nephio.org/cluster-name: edge02
        provider: docker.io
      mgmt:
        provider: docker.io
      regional:
        labels:
          nephio.org/cluster-name: regional
        provider: docker.io
      srl:
        address: 172.18.0.12:57400
        provider: srl.nokia.com

```
Here is a recap of the topology as it is deployed:
- We have 3 kind clusters, each deployed with a dedicated kubernetes cluster. We have a single worker and this is what is of interest here.
- We have a centralized router (nokia containerized srlinux) called leaf
- The kube workers have  an additional interface connected to the router. The interface meshing corresponds with the description in the rawtopologies.topo.nephio.org CRD.

![image](https://github.com/robric/nephio-testings/assets/21667569/3bc25a45-98b8-41a5-8d01-ecc0b0ee0593)

As reflected in nephio CRDs, we have three distinct vlans for each interface

![image](https://github.com/robric/nephio-testings/assets/21667569/9704e5ab-36f9-476b-8c37-0569492e5c58)


## Deploy Free5GC on the regional site

Clone  the free5gc package to the regional repository and deploy:

```
ubuntu@ubuntu-vm:~/free5gc$ kpt alpha rpkg get | grep free5g
free5gc-packages-148a8446856cfcba9ba53f9c5786bacb835f3733          free5gc-cp                           main               main       false    Published   free5gc-packages
free5gc-packages-daba5a241a23af3ce777b256c39dcedc3c5e3917          free5gc-cp                           v1                 v1         true     Published   free5gc-packages
[...]

ntu@ubuntu-vm:~$    kpt alpha rpkg clone -n default  free5gc-packages-daba5a241a23af3ce777b256c39dcedc3c5e3917 free5gc --repository regional
regional-6d264f6ffe11138112e9f2b3b843c89646a0df42 created

ubuntu@ubuntu-vm:~$
ubuntu@ubuntu-vm:~$ kpt alpha rpkg propose -n default regional-6d264f6ffe11138112e9f2b3b843c89646a0df42
regional-6d264f6ffe11138112e9f2b3b843c89646a0df42 proposed
ubuntu@ubuntu-vm:~$ kpt alpha rpkg approve -n default regional-6d264f6ffe11138112e9f2b3b843c89646a0df42
regional-6d264f6ffe11138112e9f2b3b843c89646a0df42 approved
ubuntu@ubuntu-vm:~$ 
```
We have successfully cloned the free5gc in the local regional repo. We can check that via the local gitea webUI  (http://10.87.104.36:3000/nephio/regional/src/branch/main/free5gc)
![image](https://github.com/robric/nephio-testings/assets/21667569/d3ac963a-569e-4431-84af-d7e52e24353d)

Then we see all the corresponding resources created the free5gc namespace (free5gc is name of the target package for "kpt alpha clone" command). 

```
ubuntu@ubuntu-vm:~/regional$ kubectl get ns --context regional-admin@regional free5gc 
NAME      STATUS   AGE
free5gc   Active   2m11s
ubuntu@ubuntu-vm:~/regional$

ubuntu@ubuntu-vm:~$ kubectl get all -n free5gc    --context regional-admin@regional
NAME                                 READY   STATUS    RESTARTS   AGE
pod/free5gc-ausf-7d494d668d-p6pzw    1/1     Running   0          20m
pod/free5gc-nrf-66cc98cfc5-kjnfm     1/1     Running   0          20m
pod/free5gc-nssf-668db85d54-bxnlx    1/1     Running   0          20m
pod/free5gc-pcf-55d4bfd648-cqsq8     1/1     Running   0          20m
pod/free5gc-udm-845db6c9c8-ntmgv     1/1     Running   0          20m
pod/free5gc-udr-79466f7f86-rs556     1/1     Running   0          20m
pod/free5gc-webui-84ff8c456c-v84ft   1/1     Running   0          20m
pod/mongodb-0                        1/1     Running   0          20m

NAME                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/ausf-nausf      ClusterIP   10.136.73.176    <none>        80/TCP           20m
service/mongodb         ClusterIP   10.131.217.111   <none>        27017/TCP        20m
service/nrf-nnrf        ClusterIP   10.139.184.118   <none>        8000/TCP         20m
service/nssf-nnssf      ClusterIP   10.143.248.107   <none>        80/TCP           20m
service/pcf-npcf        ClusterIP   10.143.164.18    <none>        80/TCP           20m
service/udm-nudm        ClusterIP   10.131.73.175    <none>        80/TCP           20m
service/udr-nudr        ClusterIP   10.140.155.175   <none>        80/TCP           20m
service/webui-service   NodePort    10.129.87.12     <none>        5000:30500/TCP   20m

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/free5gc-ausf    1/1     1            1           20m
deployment.apps/free5gc-nrf     1/1     1            1           20m
deployment.apps/free5gc-nssf    1/1     1            1           20m
deployment.apps/free5gc-pcf     1/1     1            1           20m
deployment.apps/free5gc-udm     1/1     1            1           20m
deployment.apps/free5gc-udr     1/1     1            1           20m
deployment.apps/free5gc-webui   1/1     1            1           20m

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/free5gc-ausf-7d494d668d    1         1         1       20m
replicaset.apps/free5gc-nrf-66cc98cfc5     1         1         1       20m
replicaset.apps/free5gc-nssf-668db85d54    1         1         1       20m
replicaset.apps/free5gc-pcf-55d4bfd648     1         1         1       20m
replicaset.apps/free5gc-udm-845db6c9c8     1         1         1       20m
replicaset.apps/free5gc-udr-79466f7f86     1         1         1       20m
replicaset.apps/free5gc-webui-84ff8c456c   1         1         1       20m

NAME                       READY   AGE
statefulset.apps/mongodb   1/1     20m
```
Now we deploy the free5gc operator to finalize the deployment.
```

### This is controlled via a package variange. 

ubuntu@ubuntu-vm:~$ cat test-infra/e2e/tests/004-free5gc-operator.yaml 
apiVersion: config.porch.kpt.dev/v1alpha2
kind: PackageVariantSet
metadata:
  name: free5gc-operator
spec:
  upstream:
    repo: free5gc-packages
    package: free5gc-operator
    revision: v5
  targets:
  - objectSelector:
      apiVersion: infra.nephio.org/v1alpha1
      kind: WorkloadCluster
    template:
      annotations:
        approval.nephio.org/policy: initial
ubuntu@ubuntu-vm:~$

### Here the target for operator is the workloadclusters

ubuntu@ubuntu-vm:~$ kubectl get workloadclusters.infra.nephio.org 
NAME       AGE
edge01     6d17h
edge02     6d17h
regional   6d17h
```

This copies the package to the "regional" and "edge0x" repositories:
![image](https://github.com/robric/nephio-testings/assets/21667569/2299be3e-9660-4b14-86f3-281bc04c93e0)
![image](https://github.com/robric/nephio-testings/assets/21667569/e6e2e1bb-d03d-4ecc-b4ec-34e78d8b01a8)

The operator is deployed in edge clusters (but actually not in the regional - I couldn't not find out why -).

```
ubuntu@ubuntu-vm:~$ kubectl -n free5gc get all --context edge01-admin@edge01 
NAME                                    READY   STATUS    RESTARTS   AGE
pod/free5gc-operator-79bd5b6f7d-2qrsv   2/2     Running   0          23m

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/free5gc-operator   1/1     1            1           23m

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/free5gc-operator-79bd5b6f7d   1         1         1       23m
ubuntu@ubuntu-vm:~$
```

Then deploy:
- UPF at Edge
- SMF and AMF in regional

Cluster selection is based on labels that were defined during workloadcluster creation

```
ubuntu@ubuntu-vm:~$ kubectl apply -f test-infra/e2e/tests/005-edge-free5gc-upf.yaml
ubuntu@ubuntu-vm:~$ kubectl apply -f test-infra/e2e/tests/006-regional-free5gc-amf.yaml
ubuntu@ubuntu-vm:~$ kubectl apply -f test-infra/e2e/tests/006-regional-free5gc-smf.yaml

ubuntu@ubuntu-vm:~$ cat test-infra/e2e/tests/006-regional-free5gc-amf.yaml
apiVersion: config.porch.kpt.dev/v1alpha2
kind: PackageVariantSet
[....]
  targets:
  - objectSelector:
      apiVersion: infra.nephio.org/v1alpha1
      kind: WorkloadCluster
      matchLabels:
        nephio.org/site-type: regional  <------------------------------------  LABEL 

ubuntu@ubuntu-vm:~$ cat regional/workload-cluster.yaml 
apiVersion: infra.nephio.org/v1alpha1
kind: WorkloadCluster
[...]
  labels:
    nephio.org/site-type: regional   <------------------------------------  LABEL 


ubuntu@ubuntu-vm:~$ kubectl get pods -n free5gc-upf -l name=upf-edge01 --context edge01-admin@edge01 
NAME                          READY   STATUS    RESTARTS   AGE
upf-edge01-6dcbb7b77d-mbzsr   1/1     Running   0          13m
ubuntu@ubuntu-vm:~$

```

#  APPENDIX

## SR LINUX CLI and outputs

```
### Find the srlinux router container

ubuntu@ubuntu-vm:~$ docker ps | grep srl
fa84deab86bd   ghcr.io/nokia/srlinux:22.11.2-116   "/tini -- fixuid -q …"   5 days ago    Up 5 days                                           net-free5gc-net-leaf

### Connect to it via docker

ubuntu@ubuntu-vm:~$ sudo docker exec -it      net-free5gc-net-leaf  sh
sh-4.4# sr_cli
Using configuration file(s): []
Welcome to the srlinux CLI.
Type 'help' (and press <ENTER>) if you need any help using this.
--{ + running }--[  ]--
A:leaf#

### Run commands

A:leaf# show network-instance route-table  
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
IPv4 unicast route table of network instance mgmt
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
+-----------------------------+-------+------------+----------------------+----------------------+----------+---------+------------------+------------------+
|           Prefix            |  ID   | Route Type |     Route Owner      |        Active        |  Metric  |  Pref   | Next-hop (Type)  |     Next-hop     |
|                             |       |            |                      |                      |          |         |                  |    Interface     |
+=============================+=======+============+======================+======================+==========+=========+==================+==================+
| 0.0.0.0/0                   | 1     | dhcp       | dhcp_client_mgr      | True                 | 0        | 5       | 172.18.0.1       | mgmt0.0          |
|                             |       |            |                      |                      |          |         | (direct)         |                  |
| 172.18.0.0/16               | 0     | linux      | linux_mgr            | False                | 0        | 5       | 172.18.0.0       | mgmt0.0          |
|                             |       |            |                      |                      |          |         | (direct)         |                  |
| 172.18.0.0/16               | 1     | local      | net_inst_mgr         | True                 | 0        | 0       | 172.18.0.12      | mgmt0.0          |
|                             |       |            |                      |                      |          |         | (direct)         |                  |
| 172.18.0.12/32              | 1     | host       | net_inst_mgr         | True                 | 0        | 0       | None (extract)   | None             |
| 172.18.255.255/32           | 1     | host       | net_inst_mgr         | True                 | 0        | 0       | None (broadcast) |                  |
+-----------------------------+-------+------------+----------------------+----------------------+----------+---------+------------------+------------------+
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
IPv4 routes total                    : 5
IPv4 prefixes with active routes     : 4
IPv4 prefixes with active ECMP routes: 0
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
IPv4 unicast route table of network instance vpc-internal
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
+-----------------------------+-------+------------+----------------------+----------------------+----------+---------+------------------+------------------+
|           Prefix            |  ID   | Route Type |     Route Owner      |        Active        |  Metric  |  Pref   | Next-hop (Type)  |     Next-hop     |
|                             |       |            |                      |                      |          |         |                  |    Interface     |
+=============================+=======+============+======================+======================+==========+=========+==================+==================+
| 172.1.0.0/24                | 3     | local      | net_inst_mgr         | True                 | 0        | 0       | 172.1.0.1        | irb0.2372        |
|                             |       |            |                      |                      |          |         | (direct)         |                  |
| 172.1.0.1/32                | 3     | host       | net_inst_mgr         | True                 | 0        | 0       | None (extract)   | None             |
| 172.1.0.255/32              | 3     | host       | net_inst_mgr         | True                 | 0        | 0       | None (broadcast) |                  |
| 172.1.1.0/24                | 17    | local      | net_inst_mgr         | True                 | 0        | 0       | 172.1.1.1        | irb0.2026        |
|                             |       |            |                      |                      |          |         | (direct)         |                  |
| 172.1.1.1/32                | 17    | host       | net_inst_mgr         | True                 | 0        | 0       | None (extract)   | None             |
| 172.1.1.255/32              | 17    | host       | net_inst_mgr         | True                 | 0        | 0       | None (broadcast) |                  |
| 172.1.2.0/24                | 16    | local      | net_inst_mgr         | True                 | 0        | 0       | 172.1.2.1        | irb0.2025        |
|                             |       |            |                      |                      |          |         | (direct)         |                  |
| 172.1.2.1/32                | 16    | host       | net_inst_mgr         | True                 | 0        | 0       | None (extract)   | None             |
| 172.1.2.255/32              | 16    | host       | net_inst_mgr         | True                 | 0        | 0       | None (broadcast) |                  |
+-----------------------------+-------+------------+----------------------+----------------------+----------+---------+------------------+------------------+
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
IPv4 routes total                    : 9
IPv4 prefixes with active routes     : 9
IPv4 prefixes with active ECMP routes: 0
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
IPv4 unicast route table of network instance vpc-internet
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
+-----------------------------+-------+------------+----------------------+----------------------+----------+---------+------------------+------------------+
|           Prefix            |  ID   | Route Type |     Route Owner      |        Active        |  Metric  |  Pref   | Next-hop (Type)  |     Next-hop     |
|                             |       |            |                      |                      |          |         |                  |    Interface     |
+=============================+=======+============+======================+======================+==========+=========+==================+==================+
| 172.0.0.0/24                | 7     | local      | net_inst_mgr         | True                 | 0        | 0       | 172.0.0.1        | irb0.2384        |
|                             |       |            |                      |                      |          |         | (direct)         |                  |
| 172.0.0.1/32                | 7     | host       | net_inst_mgr         | True                 | 0        | 0       | None (extract)   | None             |
| 172.0.0.255/32              | 7     | host       | net_inst_mgr         | True                 | 0        | 0       | None (broadcast) |                  |
| 172.0.1.0/24                | 6     | local      | net_inst_mgr         | True                 | 0        | 0       | 172.0.1.1        | irb0.2038        |
|                             |       |            |                      |                      |          |         | (direct)         |                  |
| 172.0.1.1/32                | 6     | host       | net_inst_mgr         | True                 | 0        | 0       | None (extract)   | None             |
| 172.0.1.255/32              | 6     | host       | net_inst_mgr         | True                 | 0        | 0       | None (broadcast) |                  |
| 172.0.2.0/24                | 19    | local      | net_inst_mgr         | True                 | 0        | 0       | 172.0.2.1        | irb0.2037        |
|                             |       |            |                      |                      |          |         | (direct)         |                  |
| 172.0.2.1/32                | 19    | host       | net_inst_mgr         | True                 | 0        | 0       | None (extract)   | None             |
| 172.0.2.255/32              | 19    | host       | net_inst_mgr         | True                 | 0        | 0       | None (broadcast) |                  |
+-----------------------------+-------+------------+----------------------+----------------------+----------+---------+------------------+------------------+
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
IPv4 routes total                    : 9
IPv4 prefixes with active routes     : 9
IPv4 prefixes with active ECMP routes: 0
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
IPv4 unicast route table of network instance vpc-ran
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
+-----------------------------+-------+------------+----------------------+----------------------+----------+---------+------------------+------------------+
|           Prefix            |  ID   | Route Type |     Route Owner      |        Active        |  Metric  |  Pref   | Next-hop (Type)  |     Next-hop     |
|                             |       |            |                      |                      |          |         |                  |    Interface     |
+=============================+=======+============+======================+======================+==========+=========+==================+==================+
| 172.2.0.0/24                | 12    | local      | net_inst_mgr         | True                 | 0        | 0       | 172.2.0.1        | irb0.1486        |
|                             |       |            |                      |                      |          |         | (direct)         |                  |
| 172.2.0.1/32                | 12    | host       | net_inst_mgr         | True                 | 0        | 0       | None (extract)   | None             |
| 172.2.0.255/32              | 12    | host       | net_inst_mgr         | True                 | 0        | 0       | None (broadcast) |                  |
| 172.2.1.0/24                | 11    | local      | net_inst_mgr         | True                 | 0        | 0       | 172.2.1.1        | irb0.1485        |
|                             |       |            |                      |                      |          |         | (direct)         |                  |
| 172.2.1.1/32                | 11    | host       | net_inst_mgr         | True                 | 0        | 0       | None (extract)   | None             |
| 172.2.1.255/32              | 11    | host       | net_inst_mgr         | True                 | 0        | 0       | None (broadcast) |                  |
| 172.2.2.0/24                | 13    | local      | net_inst_mgr         | True                 | 0        | 0       | 172.2.2.1        | irb0.1832        |
|                             |       |            |                      |                      |          |         | (direct)         |                  |
| 172.2.2.1/32                | 13    | host       | net_inst_mgr         | True                 | 0        | 0       | None (extract)   | None             |
| 172.2.2.255/32              | 13    | host       | net_inst_mgr         | True                 | 0        | 0       | None (broadcast) |                  |
| 172.3.0.0/24                | 12    | local      | net_inst_mgr         | True                 | 0        | 0       | 172.3.0.1        | irb0.1486        |
|                             |       |            |                      |                      |          |         | (direct)         |                  |
| 172.3.0.1/32                | 12    | host       | net_inst_mgr         | True                 | 0        | 0       | None (extract)   | None             |
| 172.3.0.255/32              | 12    | host       | net_inst_mgr         | True                 | 0        | 0       | None (broadcast) |                  |
| 172.3.1.0/24                | 11    | local      | net_inst_mgr         | True                 | 0        | 0       | 172.3.1.1        | irb0.1485        |
|                             |       |            |                      |                      |          |         | (direct)         |                  |
| 172.3.1.1/32                | 11    | host       | net_inst_mgr         | True                 | 0        | 0       | None (extract)   | None             |
| 172.3.1.255/32              | 11    | host       | net_inst_mgr         | True                 | 0        | 0       | None (broadcast) |                  |
| 172.3.2.0/24                | 13    | local      | net_inst_mgr         | True                 | 0        | 0       | 172.3.2.1        | irb0.1832        |
|                             |       |            |                      |                      |          |         | (direct)         |                  |
| 172.3.2.1/32                | 13    | host       | net_inst_mgr         | True                 | 0        | 0       | None (extract)   | None             |
| 172.3.2.255/32              | 13    | host       | net_inst_mgr         | True                 | 0        | 0       | None (broadcast) |                  |
+-----------------------------+-------+------------+----------------------+----------------------+----------+---------+------------------+------------------+
--------------------------------------------------------------------------------------------------------
[....]


A:leaf# show interface  
==================================================================================================================================================================================
ethernet-1/1 is up, speed 10G, type None
  ethernet-1/1.2 is up
    Network-instance: vpc-ran-edge01-bd
    Encapsulation   : vlan-id 2
    Type            : bridged
  ethernet-1/1.3 is up
    Network-instance: vpc-internal-edge01-bd
    Encapsulation   : vlan-id 3
    Type            : bridged
  ethernet-1/1.4 is up
    Network-instance: vpc-internet-edge01-bd
    Encapsulation   : vlan-id 4
    Type            : bridged
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
ethernet-1/2 is up, speed 10G, type None
  ethernet-1/2.2 is up
    Network-instance: vpc-internet-edge02-bd
    Encapsulation   : vlan-id 2
    Type            : bridged
  ethernet-1/2.3 is up
    Network-instance: vpc-ran-edge02-bd
    Encapsulation   : vlan-id 3
    Type            : bridged
  ethernet-1/2.4 is up
    Network-instance: vpc-internal-edge02-bd
    Encapsulation   : vlan-id 4
    Type            : bridged
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
ethernet-1/3 is up, speed 100G, type None
  ethernet-1/3.2 is up
    Network-instance: vpc-internal-regional-bd
    Encapsulation   : vlan-id 2
    Type            : bridged
  ethernet-1/3.3 is up
    Network-instance: vpc-internet-regional-bd
    Encapsulation   : vlan-id 3
    Type            : bridged
  ethernet-1/3.4 is up
    Network-instance: vpc-ran-regional-bd
    Encapsulation   : vlan-id 4
    Type            : bridged
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
irb0 is up, speed None, type None
  irb0.1485 is up
    Network-instance: vpc-ran, vpc-ran-edge01-bd
    Encapsulation   : null
    Type            : None
    IPv4 addr    : 172.2.1.1/24 (static, preferred, primary, anycast)
    IPv4 addr    : 172.3.1.1/24 (static, preferred, anycast)
    IPv6 addr    : 172:2:0:1::1/64 (static, preferred, primary, anycast)
    IPv6 addr    : 172:3:0:1::1/64 (static, preferred, anycast)
    IPv6 addr    : fe80::200:5eff:fe00:101/64 (link-layer, preferred, anycast)
    IPv6 addr    : fe80::1868:2ff:feff:41/64 (link-layer, preferred)
  irb0.1486 is up
    Network-instance: vpc-ran, vpc-ran-edge02-bd
    Encapsulation   : null
    Type            : None
    IPv4 addr    : 172.2.0.1/24 (static, preferred, primary, anycast)
    IPv4 addr    : 172.3.0.1/24 (static, preferred, anycast)
    IPv6 addr    : 172:2::1/64 (static, preferred, primary, anycast)
    IPv6 addr    : 172:3::1/64 (static, preferred, anycast)
    IPv6 addr    : fe80::200:5eff:fe00:101/64 (link-layer, preferred, anycast)
    IPv6 addr    : fe80::1868:2ff:feff:41/64 (link-layer, preferred)
  irb0.1832 is up
    Network-instance: vpc-ran, vpc-ran-regional-bd
    Encapsulation   : null
    Type            : None
    IPv4 addr    : 172.2.2.1/24 (static, preferred, primary, anycast)
    IPv4 addr    : 172.3.2.1/24 (static, preferred, anycast)
    IPv6 addr    : 172:2:0:2::1/64 (static, preferred, primary, anycast)
    IPv6 addr    : 172:3:0:2::1/64 (static, preferred, anycast)
    IPv6 addr    : fe80::200:5eff:fe00:101/64 (link-layer, preferred, anycast)
    IPv6 addr    : fe80::1868:2ff:feff:41/64 (link-layer, preferred)
  irb0.2025 is up
    Network-instance: vpc-internal, vpc-internal-edge01-bd
    Encapsulation   : null
    Type            : None
    IPv4 addr    : 172.1.2.1/24 (static, preferred, primary, anycast)
    IPv6 addr    : 172:1:0:2::1/64 (static, preferred, primary, anycast)
    IPv6 addr    : fe80::200:5eff:fe00:101/64 (link-layer, preferred, anycast)
    IPv6 addr    : fe80::1868:2ff:feff:41/64 (link-layer, preferred)
  irb0.2026 is up
    Network-instance: vpc-internal, vpc-internal-edge02-bd
    Encapsulation   : null
    Type            : None
    IPv4 addr    : 172.1.1.1/24 (static, preferred, primary, anycast)
    IPv6 addr    : 172:1:0:1::1/64 (static, preferred, primary, anycast)
    IPv6 addr    : fe80::200:5eff:fe00:101/64 (link-layer, preferred, anycast)
    IPv6 addr    : fe80::1868:2ff:feff:41/64 (link-layer, preferred)
  irb0.2037 is up
    Network-instance: vpc-internet, vpc-internet-edge01-bd
    Encapsulation   : null
    Type            : None
    IPv4 addr    : 172.0.2.1/24 (static, preferred, primary, anycast)
    IPv6 addr    : 172:0:0:2::1/64 (static, preferred, primary, anycast)
    IPv6 addr    : fe80::200:5eff:fe00:101/64 (link-layer, preferred, anycast)
    IPv6 addr    : fe80::1868:2ff:feff:41/64 (link-layer, preferred)
  irb0.2038 is up
    Network-instance: vpc-internet, vpc-internet-edge02-bd
    Encapsulation   : null
    Type            : None
    IPv4 addr    : 172.0.1.1/24 (static, preferred, primary, anycast)
    IPv6 addr    : 172:0:0:1::1/64 (static, preferred, primary, anycast)
    IPv6 addr    : fe80::200:5eff:fe00:101/64 (link-layer, preferred, anycast)
    IPv6 addr    : fe80::1868:2ff:feff:41/64 (link-layer, preferred)
  irb0.2372 is up
    Network-instance: vpc-internal, vpc-internal-regional-bd
    Encapsulation   : null
    Type            : None
    IPv4 addr    : 172.1.0.1/24 (static, preferred, primary, anycast)
    IPv6 addr    : 172:1::1/64 (static, preferred, primary, anycast)
    IPv6 addr    : fe80::200:5eff:fe00:101/64 (link-layer, preferred, anycast)
    IPv6 addr    : fe80::1868:2ff:feff:41/64 (link-layer, preferred)
  irb0.2384 is up
    Network-instance: vpc-internet, vpc-internet-regional-bd
    Encapsulation   : null
    Type            : None
    IPv4 addr    : 172.0.0.1/24 (static, preferred, primary, anycast)
    IPv6 addr    : 172::1/64 (static, preferred, primary, anycast)
    IPv6 addr    : fe80::200:5eff:fe00:101/64 (link-layer, preferred, anycast)
    IPv6 addr    : fe80::1868:2ff:feff:41/64 (link-layer, preferred)
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
mgmt0 is up, speed 1G, type None
  mgmt0.0 is up
    Network-instance: mgmt
    Encapsulation   : null
    Type            : None
    IPv4 addr    : 172.18.0.12/16 (dhcp, preferred)
    IPv6 addr    : fc00:f853:ccd:e793::c/64 (dhcp, preferred)
    IPv6 addr    : fe80::42:acff:fe12:c/64 (link-layer, preferred)
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
==================================================================================================================================================================================
Summary
  0 loopback interfaces configured
  4 ethernet interfaces are up
  1 management interfaces are up
  19 subinterfaces are up
==================================================================================================================================================================================
--{ + running }--[  ]--
A:leaf#
```





