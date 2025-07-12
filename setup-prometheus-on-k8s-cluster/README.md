# Prometheus Monitoring Stack for Kubernetes

This Ansible playbook deploys a complete Prometheus monitoring stack on a Kubernetes cluster, including Prometheus server and kube-state-metrics for comprehensive cluster monitoring.

## Overview

The monitoring stack includes:
- **Prometheus Server**: Core monitoring and alerting system
- **kube-state-metrics**: Kubernetes metrics exporter for cluster state monitoring
- **RBAC Configuration**: Proper permissions for Prometheus to access cluster resources
- **NodePort Service**: External access to Prometheus web interface

## Prerequisites

- A running Kubernetes cluster (see `../setup-k8s-cluster-using-kubeadm/` for cluster setup)
- Ansible installed on the control machine
- SSH access to the Kubernetes master node
- `kubectl` configured and working on the master node

## Architecture

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Prometheus    │    │ kube-state-metrics│    │   Kubernetes    │
│   Server        │◄──►│                  │◄──►│   Cluster       │
│   (Port 30000)  │    │   (kube-system)  │    │   Resources     │
└─────────────────┘    └──────────────────┘    └─────────────────┘
```

## Components

### 1. Prometheus Server
- **Image**: `prom/prometheus:latest`
- **Namespace**: `monitoring`
- **Storage**: 200h retention, 512MB size limit
- **Access**: NodePort 30000

### 2. kube-state-metrics
- **Purpose**: Exports Kubernetes cluster state metrics
- **Namespace**: `kube-system`
- **Source**: Official devopscube configuration

### 3. RBAC Configuration
- **ClusterRole**: Permissions to read nodes, services, endpoints, pods
- **ClusterRoleBinding**: Binds permissions to monitoring namespace

## Installation

### 1. Update Inventory

Edit `inventory.ini` with your cluster details:

```ini
[all:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/id_rsa
ansible_ssh_common_args='-o StrictHostKeyChecking=no'

[masters]
master ansible_host=YOUR_MASTER_IP

[workers]
worker1 ansible_host=YOUR_WORKER1_IP
worker2 ansible_host=YOUR_WORKER2_IP
```

### 2. Run the Playbook

```bash
# Deploy Prometheus monitoring stack
ansible-playbook -i inventory.ini deploy-prometheus.yml
```

### 3. Verify Installation

```bash
# Check Prometheus deployment
kubectl get pods -n monitoring

# Check kube-state-metrics
kubectl get pods -n kube-system | grep kube-state-metrics

# Check services
kubectl get svc -n monitoring
```

## Accessing Prometheus

### Method 1: NodePort (External Access)
```bash
# Access via master node IP
http://MASTER_NODE_IP:30000
```

### Method 2: Port Forwarding (Local Access)
```bash
# Forward Prometheus service to local machine
kubectl port-forward svc/prometheus-service 9090:9090 -n monitoring

# Access at http://localhost:9090
```

## Configuration Details

### Prometheus Configuration
- **Config Source**: External ConfigMap from GitHub
- **Storage**: EmptyDir volume (ephemeral)
- **Retention**: 200 hours
- **Size Limit**: 512MB

### Service Configuration
- **Type**: NodePort
- **External Port**: 30000
- **Internal Port**: 9090
- **Annotations**: Prometheus scrape configuration

### RBAC Permissions
- Read access to nodes, services, endpoints, pods
- Access to ingress resources
- Metrics endpoint access

## Monitoring Features

### Default Metrics Collected
- **Kubernetes Cluster Metrics**:
  - Node status and resources
  - Pod lifecycle and resource usage
  - Service and endpoint information
  - Ingress configurations

- **System Metrics**:
  - CPU and memory usage
  - Network statistics
  - Storage metrics

### Pre-configured Targets
- Kubernetes API server
- kube-state-metrics
- Node exporters (if available)
- Custom service endpoints

## Troubleshooting

### Common Issues

1. **Prometheus Pod Not Starting**
   ```bash
   kubectl describe pod prometheus-deployment-xxx -n monitoring
   kubectl logs prometheus-deployment-xxx -n monitoring
   ```

2. **Service Not Accessible**
   ```bash
   # Check service configuration
   kubectl get svc prometheus-service -n monitoring -o yaml
   
   # Check endpoints
   kubectl get endpoints prometheus-service -n monitoring
   ```

3. **RBAC Issues**
   ```bash
   # Check ClusterRole and ClusterRoleBinding
   kubectl get clusterrole prometheus
   kubectl get clusterrolebinding prometheus
   ```

### Verification Commands

```bash
# Check all components
kubectl get all -n monitoring
kubectl get all -n kube-system | grep kube-state-metrics

# Check Prometheus targets
# Access Prometheus UI and go to Status > Targets

# Check metrics endpoint
curl http://MASTER_NODE_IP:30000/metrics
```

## Customization

### Modify Prometheus Configuration
1. Edit the ConfigMap referenced in the playbook
2. Update retention settings in `prometheus-deployment.yaml`
3. Add custom scrape configurations

### Add Additional Components
- **Grafana**: For visualization (separate playbook recommended)
- **AlertManager**: For alerting (separate playbook recommended)
- **Node Exporter**: For detailed node metrics

### Storage Configuration
To use persistent storage, modify the volume configuration in `prometheus-deployment.yaml`:

```yaml
volumes:
- name: prometheus-storage-volume
  persistentVolumeClaim:
    claimName: prometheus-pvc
```

## Security Considerations

- **Network Access**: NodePort exposes Prometheus externally
- **RBAC**: Minimal required permissions configured
- **Authentication**: No authentication configured by default
- **TLS**: No TLS termination configured

## Maintenance

### Updating Prometheus
```bash
# Update to latest version
kubectl set image deployment/prometheus-deployment prometheus=prom/prometheus:latest -n monitoring
```

### Backup Configuration
```bash
# Export current configuration
kubectl get configmap prometheus-server-conf -n monitoring -o yaml > prometheus-config-backup.yaml
```

### Scaling
```bash
# Scale Prometheus replicas (not recommended for single instance)
kubectl scale deployment prometheus-deployment --replicas=2 -n monitoring
```

## Support

For issues and questions:
1. Check the troubleshooting section above
2. Review Prometheus logs: `kubectl logs -n monitoring -l app=prometheus`
3. Verify cluster connectivity and permissions
4. Check Kubernetes events: `kubectl get events -n monitoring`

## Related Documentation

- [Prometheus Official Documentation](https://prometheus.io/docs/)
- [kube-state-metrics Documentation](https://github.com/kubernetes/kube-state-metrics)
- [Kubernetes Monitoring Best Practices](https://kubernetes.io/docs/tasks/debug-application-cluster/resource-usage-monitoring/)


## Successful Installation
When you visit one of the worker node IP on port 30000, it will display the prometheus set up.