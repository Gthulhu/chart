# Gthulhu Helm Chart

This Helm chart deploys Gthulhu BPF Scheduler and BSS Metrics API Server to Kubernetes.

## Overview

Gthulhu optimizes cloud-native workloads using the Linux Scheduler Extension (sched-ext) for different application scenarios. This chart deploys:

1. **Gthulhu Scheduler**: A BPF-based scheduler that runs as a DaemonSet on all nodes
2. **BSS Metrics API Server**: A REST API service for collecting and processing scheduler metrics

## Prerequisites

- Kubernetes 1.19+
- Helm 3.0+
- Linux kernel 6.12+ with sched-ext support on worker nodes
- Container runtime with BPF capabilities

## Installation

### Add the Helm repository (if applicable)

```bash
helm repo add gthulhu https://your-helm-repo.com
helm repo update
```

### Install the chart

```bash
# Install with default values
helm install gthulhu gthulhu/gthulhu

# Install with custom values
helm install gthulhu gthulhu/gthulhu -f custom-values.yaml

# Install from local chart
helm install gthulhu ./gthulhu
```

## Configuration

The following table lists the configurable parameters and their default values:

### Global Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `global.imagePullSecrets` | Image pull secrets | `[]` |
| `global.nameOverride` | Override the name | `""` |
| `global.fullnameOverride` | Override the full name | `""` |

### Scheduler Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `scheduler.enabled` | Enable the BPF scheduler | `true` |
| `scheduler.image.repository` | Scheduler image repository | `gthulhu` |
| `scheduler.image.tag` | Scheduler image tag | `latest` |
| `scheduler.image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `scheduler.hostPID` | Use host PID namespace | `true` |
| `scheduler.resources.limits.cpu` | CPU limit | `500m` |
| `scheduler.resources.limits.memory` | Memory limit | `512Mi` |
| `scheduler.resources.requests.cpu` | CPU request | `100m` |
| `scheduler.resources.requests.memory` | Memory request | `128Mi` |

### API Server Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `api.enabled` | Enable the metrics API server | `true` |
| `api.replicaCount` | Number of API server replicas | `1` |
| `api.image.repository` | API server image repository | `gthulhu-api` |
| `api.image.tag` | API server image tag | `latest` |
| `api.port` | API server port | `8080` |
| `api.service.type` | Service type | `ClusterIP` |
| `api.service.port` | Service port | `80` |
| `api.ingress.enabled` | Enable ingress | `false` |
| `api.healthCheck.enabled` | Enable health checks | `true` |
| `api.autoscaling.enabled` | Enable HPA | `false` |

### Monitoring Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `monitoring.enabled` | Enable monitoring | `false` |
| `monitoring.serviceMonitor.enabled` | Enable ServiceMonitor for Prometheus | `false` |

## Examples

### Basic Installation

```bash
helm install gthulhu ./gthulhu
```

### Production Installation with Custom Values

```yaml
# production-values.yaml
scheduler:
  resources:
    limits:
      cpu: 1000m
      memory: 1Gi
    requests:
      cpu: 200m
      memory: 256Mi

api:
  replicaCount: 3
  ingress:
    enabled: true
    className: nginx
    hosts:
      - host: gthulhu-api.example.com
        paths:
          - path: /
            pathType: Prefix
  autoscaling:
    enabled: true
    minReplicas: 2
    maxReplicas: 10
    targetCPUUtilizationPercentage: 70

monitoring:
  enabled: true
  serviceMonitor:
    enabled: true
```

```bash
helm install gthulhu ./gthulhu -f production-values.yaml
```

### Development Installation (API only)

```yaml
# dev-values.yaml
scheduler:
  enabled: false

api:
  enabled: true
  service:
    type: NodePort
```

```bash
helm install gthulhu-dev ./gthulhu -f dev-values.yaml
```

## Accessing the API

### Using port-forward (ClusterIP)

```bash
kubectl port-forward svc/gthulhu-api 8080:80
curl http://localhost:8080/health
```

### Using NodePort

```bash
export NODE_PORT=$(kubectl get svc gthulhu-api -o jsonpath='{.spec.ports[0].nodePort}')
export NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[0].address}')
curl http://$NODE_IP:$NODE_PORT/health
```

### Using Ingress

```bash
curl http://gthulhu-api.example.com/health
```

## API Endpoints

- `POST /api/v1/metrics` - Submit BSS metrics data
- `GET /health` - Health check endpoint
- `GET /` - API information

## Monitoring

When monitoring is enabled, the chart creates a ServiceMonitor that can be picked up by Prometheus Operator to scrape metrics from the API server.

## Troubleshooting

### Scheduler Issues

Check if the scheduler is running:

```bash
kubectl get daemonset gthulhu-scheduler
kubectl logs -l app.kubernetes.io/component=scheduler
```

### API Server Issues

Check API server logs:

```bash
kubectl logs -l app.kubernetes.io/component=api
```

### Common Issues

1. **Kernel Compatibility**: Ensure your nodes have Linux kernel 6.12+ with sched-ext support
2. **BPF Capabilities**: Verify that your container runtime supports BPF operations
3. **Privileges**: The scheduler requires privileged access and host PID namespace

## Uninstallation

```bash
helm uninstall gthulhu
```

## Development

### Testing the Chart

```bash
# Lint the chart
helm lint ./gthulhu

# Dry run installation
helm install gthulhu ./gthulhu --dry-run --debug

# Template rendering
helm template gthulhu ./gthulhu
```

## Contributing

Please read the main project [README](https://github.com/Gthulhu/Gthulhu) for contribution guidelines.

## License

This software is distributed under the terms of the Apache License 2.0.
