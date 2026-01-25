# Cool things to note

## Packages
Using ansible.builtin.apt to update available package index files

## Kernel Modules
Using ansible.builtin.modprobe to enable required kernel modules\
You can view currently active modules using lsmod | grep <module>\
Using ansible.builtin.sysctl to modify the Kube sysctl file within sysctl.d\
Unfortunately, Ansible lacks a similar builtin to modify modules-load.d

## SSH Hardening
Using some logic to set ssh service name depending on os_family fact
