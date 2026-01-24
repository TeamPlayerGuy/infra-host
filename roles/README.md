# Role Structure
Roles should be as simple as possible, however, we've got a lot of things to consider\
OS Family: Debian or RHEL\
Platform:  Pi, Panda, or VMWare\
K8S Role:  Master or Worker\
Feature:   Storage versus Transcode

##Base Role\
    Example: base_unix_os\
    Any Unix host that will run Kube\
    Disable swap for instance, required on any Unix host\
    Common sysctl (bridge netfilter, IP Forwarding)\
    This role should also contain logic to account for various Unix OS\
    ssh_service_name: "{{ 'ssh' if ansible_facts.os_family == 'Debian' else 'sshd' }}"

2.  **OS Family Specific Roles** - Example: role_os_debian\
    What does this specific OS require?\
    HTTPS repo enablement, different Repo, GPG keys, package names\
    SELinux policies versus AppArmor profiles\
    Network configuration (netplan verus NetworkManager)

3.  **Architecture/Platform Roles** - Example: role_platform_pi\
    Things are only true for specific hardware platform\
    Bootloader overwrite (Pi Only)\
    cgroup quirks

4.  **Kube Base Role** - Example: k8s_base\
    Things that all Kubernetes nodes require\
    Packages like containerd.io, kubelet, kubeadm, kubectl

5.  **Kube Function Roles** - Example: role_k8s_worker\
    kubeadm init verus kubeadm join\
    Control-Plane taints

6.  **Feature Roles** - Example: role_cni_calico\
    What does this worker node do?\
    Does it have 500gb of storage, then it's used for longhorn\
    Does it lack an SSD all together, but has x86 architecture, then it's transcoding
