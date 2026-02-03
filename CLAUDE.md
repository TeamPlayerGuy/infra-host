# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Ansible infrastructure codebase for provisioning and managing Kubernetes clusters on Raspberry Pi, ARM, and virtualized hardware (Proxmox, VMware). Uses a layered role approach from base OS configuration through Kubernetes setup to feature enablement.

## Common Commands

```bash
# Syntax check a playbook
ansible-playbook --syntax-check playbooks/day1/1-bootstrap-os.yml

# Gather facts on target hosts
ansible-playbook playbooks/testing/facts.yml -e 'target_hosts=k8s_new_workers'

# Run bootstrap on specific host group
ansible-playbook playbooks/day1/1-bootstrap-os.yml -l k8s_new_workers

# Run with specific tags
ansible-playbook playbooks/day2/add-user.yml --tags service

# Dry run (check mode)
ansible-playbook playbooks/day1/1-bootstrap-os.yml --check

# Test a role against staging hosts
ansible-playbook playbooks/testing/role_testing.yml

# Tear down Kubernetes on a node (kubeadm reset)
ansible-playbook playbooks/testing/teardown-kube.yml
```

## Architecture

### Role Layering (Applied in Order)

1. **base_unix_os** - Base OS config for all nodes (swap, packages, kernel modules, SSH hardening, oh-my-bash)
2. **os_ubuntu / os_rhel** - OS family-specific configurations (apt repos for Docker and Kubernetes)
3. **platform_pi_5b / platform_generic_arm / platform_iota / platform_proxmox / platform_vmware** - Hardware/platform-specific configurations (bootloader, fan control, cgroups)
4. **k8s_base** - Common Kubernetes prerequisites for all nodes (containerd config, k8s packages, kubelet)
5. **k8s_master / k8s_worker** - Kubernetes node role setup (kubeadm init/join, node labeling/tagging)
6. **feature_*** - Optional features (storage_longhorn, compute_transcode)

### Inventory Structure

- `inventory/hosts.yml` - Host groups: `k8s_live_*` (production), `k8s_new_*` (staging), parent groups `k8s_live_cluster` and `k8s_new_cluster`
- `inventory/group_vars/all/default.yml` - Global defaults (ansible_user, python interpreter, SSH key)
- `inventory/group_vars/[group]/` - Group-specific overrides (e.g., `k8s_new_workers/packages.yml` for package lists)
- `inventory/host_vars/[hostname]/` - Host-specific overrides

### Playbook Organization

- `playbooks/day1/` - Bootstrap sequence: OS (1) → K8s (2) → Cluster services (3)
- `playbooks/day2/` - Operational tasks (user management, maintenance)
- `playbooks/testing/` - Validation, debugging, and teardown playbooks (role_testing, facts, teardown-kube)

## Code Conventions

### Required Patterns

- **SPDX header**: All YAML files start with `#SPDX-License-Identifier: MIT-0`
- **Fully-qualified module names**: Always use `ansible.builtin.*` prefix (e.g., `ansible.builtin.command`, `ansible.builtin.import_tasks`, `ansible.builtin.package`, `ansible.posix.mount`)
- **Task imports**: Main task file (`tasks/main.yml`) imports subtasks via `ansible.builtin.import_tasks` with tags

### Role Structure

```
role_name/
├── tasks/main.yml      # Imports subtasks via ansible.builtin.import_tasks
├── tasks/*.yml         # Individual task files by function
├── defaults/main.yml   # Default variables
├── vars/main.yml       # Role variables (higher priority)
├── files/              # Static config files (e.g., config.toml, k8s-modules.conf)
├── templates/          # Jinja2 templates when needed
├── handlers/main.yml   # Event handlers (restart containerd, restart kubelet, restart ssh)
└── meta/main.yml       # Role metadata and dependencies
```

### Configuration Files

- Static configs go in `roles/[role]/files/` with component prefix
- Include comment header: `# Managed by Ansible`

### Idempotency

- Use `when` clauses for conditional execution (e.g., `when: ansible_swaptotal_mb > 0`, `when: node_check.rc != 0`)
- Use `register` with `changed_when` for proper change detection
- Use `failed_when: false` for optional checks that should not halt execution
