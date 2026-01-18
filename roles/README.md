# ToDo

## Role Structure
Roles should be as simple as possible, however, we've got a lot of things to consider
Differences in OS, Architecture, Master versus Worker, Storage versus Transcode

Simplest approach would be to start at the top and work down in specificity

1.  Base Role - role_base_os
    Any host that will run Kube
    Disable swap for instance, required on any Unix host
    Common sysctl (bridge netfilter, IP Forwarding)

2.  OS Family Specific Roles - role_os_debian:role_os_windows:role_os_rhel
    What does this specific OS require
    Package manager differences, service names (httpd:rhel versus apache2:debian)
    Kernel module enablement if OS specific

3.  Architecture/Platform Roles - role_platform_pi:role_platform_generic_arm:role_platform_x86
    Things are only true for this hardware platform
    Bootloader overwrite (Pi Only)
    cgroup quirks

4.  Kube Function Roles - role_k8s_master:role_k8s_worker
    kubeadm init verus kubeadm join
    Control-Plane taints

5.  Feature Roles - role_cni_calico:role_storage_longhorn
    What does this worker node do?
    Does it have 500gb of storage, then it's used for longhorn
    Does it lack an SSD all together, but has x86 architecture, then it's transcoding
