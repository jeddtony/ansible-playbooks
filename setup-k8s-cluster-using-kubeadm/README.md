# Kubernetes Cluster Setup with Ansible

This repository contains modular Ansible playbooks for setting up a Kubernetes cluster using kubeadm.

## Structure

```
.
├── roles/
│   ├── common/          # Common setup for all nodes
│   ├── cri-o/           # CRI-O container runtime installation
│   ├── kubernetes/      # Kubernetes components installation
│   ├── control-plane/   # Control plane initialization
│   ├── calico/          # Calico network plugin
│   ├── addons/          # Additional components (metrics server, sample apps)
│   ├── worker/          # Worker node joining
│   └── validation/      # Cluster validation and testing
├── group_vars/
│   └── all.yml          # Global variables
├── 01-setup-all-nodes.yml
├── 02-initialize-control-plane.yml
├── 03-join-worker-nodes.yml
├── 04-validate-cluster.yml
├── site.yml             # Main playbook (runs all)
└── inventory.ini        # Your inventory file
```

## Usage

### Run the complete setup:
```bash
ansible-playbook -i inventory.ini site.yml
```

### Run individual steps:
```bash
# Step 1: Setup all nodes
ansible-playbook -i inventory.ini 01-setup-all-nodes.yml

# Step 2: Initialize control plane
ansible-playbook -i inventory.ini 02-initialize-control-plane.yml

# Step 3: Join worker nodes
ansible-playbook -i inventory.ini 03-join-worker-nodes.yml

# Step 4: Validate cluster
ansible-playbook -i inventory.ini 04-validate-cluster.yml
```

## Inventory Example

```ini
[all:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/id_rsa
ansible_ssh_common_args='-o StrictHostKeyChecking=no'

[masters]
master ansible_host=192.xx.xx.xx

[workers]
worker1 ansible_host=192.xx.xx.xx
worker2 ansible_host=192.xx.xx.xx
```

## Configuration

Edit `group_vars/all.yml` to customize:
- Kubernetes version
- CRI-O version
- Pod CIDR
- Service CIDR

## Key Features & Fixes

The modular implementation includes all the same fixes as the single file version:

1. **Kubelet Timing Fix**: Kubelet is stopped initially and only started after cluster initialization
2. **Ubuntu User Kubeconfig**: Proper kubeconfig setup for both root and ubuntu users
3. **Worker Node Kubelet**: Kubelet is started before worker nodes join the cluster
4. **Error Handling**: Proper error checking and retries throughout
5. **Idempotent Operations**: All tasks can be run multiple times safely

## Benefits of Modular Structure

1. **Maintainability**: Each role has a single responsibility
2. **Reusability**: Roles can be used independently
3. **Debugging**: Easier to identify and fix issues
4. **Flexibility**: Run only the steps you need
5. **Testing**: Test individual components separately
6. **Consistency**: Same fixes applied across all implementations

## Prerequisites

- Ubuntu 20.04+ nodes
- SSH access to all nodes
- Ansible 2.9+
- Multipass (for local development) 

## Successful Setup
When you visit the ip of one of the worker nodes port 32000, it should display the "Welcome to Nginx page"