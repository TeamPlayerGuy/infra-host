# General workflow

Reference the flowchart [here](https://www.figma.com/board/EVGGDR4SPpf5EzIeJXKw6C/Infra-Workflow?node-id=0-1&p=f&t=Xs3vXWvRKiQpO9hF-0)

Prior to any of these bootstrap files you must manually configure a few things...
1. Static IP (eth0)
2. mDNS (Avahi) Will likely already be installed
3. SSH Keys for the Ansible service account or users
    a. Key must be present in authorized_keys on node
    b. Node must be known by host
4. Sudoers must be configured to allow NOPASSWD:ALL for ansible accounts

## OS BootStrap
Responsible for preparing the Pi for Kubernetes.  Many things are required, including disabling swap memory, enabling kernel modules, and overwriting the stock bootloader to enable cgroup memory control, and installing required packages.

## K8S BootStrap
Responsible for preparing the system for a Kubernetes install, enabling HTTPS apt, adding keyrings and sources, install containerd.  Also responsible for installing Kubernetes itself, the network CNI (Calico), and then initializing/joining/configuring.

## Cluster BootStrap
Responsible for preparing Kubernetes to properly function within the cluster, installing Helm, Metrics Server, and many more to come.
