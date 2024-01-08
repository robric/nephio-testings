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
## 


 

