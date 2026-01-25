# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Ansible infrastructure codebase for provisioning and managing Kubernetes clusters on Raspberry Pi and ARM hardware. Uses a layered role approach from base OS configuration through Kubernetes setup to feature enablement.

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
```

## Architecture

### Role Layering (Applied in Order)

1. **base_unix_os** - Base OS config for all K8s nodes (swap, packages, kernel modules, SSH hardening)
2. **os_ubuntu / os_rhel** - OS family-specific configurations
3. **platform_pi / platform_generic_arm** - Hardware-specific configurations (bootloader, cgroups)
4. **k8s_master / k8s_worker** - Kubernetes node role setup (kubeadm init vs join)
5. **feature_*** - Optional features (storage_longhorn, storage_nas, compute_ai, compute_transcode)

### Inventory Structure

- `inventory/hosts.yml` - Host groups: `k8s_live_*` (production), `k8s_new_*` (staging)
- `inventory/group_vars/all/` - Global defaults (ansible_user, python interpreter, SSH key)
- `inventory/group_vars/[group]/` - Group-specific overrides
- `inventory/host_vars/[hostname]/` - Host-specific overrides

### Playbook Organization

- `playbooks/day1/` - Bootstrap sequence: OS (1) → K8s (2) → Cluster services (3)
- `playbooks/day2/` - Operational tasks (user management, maintenance)
- `playbooks/testing/` - Validation and debugging playbooks

## Code Conventions

### Required Patterns

- **SPDX header**: All YAML files start with `#SPDX-License-Identifier: MIT-0`
- **Fully-qualified module names**: Use `ansible.builtin.command`, `ansible.posix.mount`, etc.
- **Task imports**: Main task file (`tasks/main.yml`) imports subtasks from separate files

### Role Structure

```
role_name/
├── tasks/main.yml      # Imports subtasks
├── tasks/*.yml         # Individual task files by function
├── defaults/main.yml   # Default variables
├── vars/main.yml       # Role variables (higher priority)
├── files/              # Static config files (prefixed: k8s-modules.conf)
├── handlers/main.yml   # Event handlers
└── meta/main.yml       # Role metadata
```

### Configuration Files

- Static configs go in `roles/[role]/files/` with component prefix
- Include comment header: `# Managed by Ansible`

### Idempotency

- Use `when` clauses for conditional execution (e.g., `when: ansible_swaptotal_mb > 0`)
- Use `register` with `changed_when` for proper change detection
