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
Installation as per tutorial. This just works.
```
wget -O - https://raw.githubusercontent.com/nephio-project/test-infra/v1.0.1/e2e/provision/init.sh |  \
sudo NEPHIO_DEBUG=false   \
     NEPHIO_BRANCH=v1.0.1 \
     NEPHIO_USER=ubuntu   \
     bash
```
Add port redirection to access UI for nephio and gitea:
```
ssh ubuntu@10.65.94.183               -L *:7007:localhost:7007                 -L *:3000:172.18.0.200:3000                 kubectl port-forward --namespace=nephio-webui svc/nephio-webui 7007
```
For some reason, this does not work for nephio UI - this is ok for gitea.

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
```


## First look

[official link](https://github.com/nephio-project/docs/blob/main/user-guide/exercises.md)


 

