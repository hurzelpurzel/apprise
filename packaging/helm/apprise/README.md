# Apprise Helm Chart

A Helm chart for deploying Apprise API - Push Notifications That Work with Everything.

## Introduction

Apprise allows you to send notifications to a wide variety of services including Slack, Discord, Telegram, Microsoft Teams, email, and many more. This Helm chart deploys the Apprise API server on Kubernetes.

## Prerequisites

- Kubernetes 1.16+
- Helm 3.0+
- PV provisioner support in the underlying infrastructure (for persistent storage)

## Installing the Chart

To install the chart with the release name `my-apprise`:

```bash
helm install my-apprise ./apprise
```

To install with custom values:

```bash
helm install my-apprise ./apprise -f my-values.yaml
```

## Uninstalling the Chart

To uninstall/delete the `my-apprise` deployment:

```bash
helm delete my-apprise
```

## Configuration

The following table lists the configurable parameters and their default values.

### Global Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of replicas | `1` |
| `image.repository` | Container image repository | `caronc/apprise` |
| `image.tag` | Container image tag | `""` (uses Chart.appVersion) |
| `image.pullPolicy` | Container image pull policy | `IfNotPresent` |

### Apprise Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `apprise.config.statefulMode` | Enable stateful mode for persistent configurations | `simple` |
| `apprise.config.workerCount` | Number of worker processes | `1` |
| `apprise.config.admin` | Enable admin interface | `true` |
| `apprise.config.baseUrl` | Base URL for reverse proxy scenarios | `""` |

### Service Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `service.type` | Kubernetes service type | `ClusterIP` |
| `service.port` | Service port | `80` |
| `service.targetPort` | Container port | `8000` |

### Ingress Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `ingress.enabled` | Enable ingress | `false` |
| `ingress.className` | Ingress class name | `""` |
| `ingress.annotations` | Ingress annotations | `{}` |
| `ingress.hosts` | Ingress hosts configuration | See values.yaml |
| `ingress.tls` | Ingress TLS configuration | `[]` |

### Persistence Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `persistence.config.enabled` | Enable persistent storage for configurations | `true` |
| `persistence.config.size` | Size of the config volume | `1Gi` |
| `persistence.config.storageClass` | Storage class for config volume | `""` |
| `persistence.attach.enabled` | Enable persistent storage for attachments | `true` |
| `persistence.attach.size` | Size of the attachment volume | `5Gi` |
| `persistence.plugin.enabled` | Enable persistent storage for plugins | `false` |

### Security Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `podSecurityContext.runAsNonRoot` | Run as non-root user | `true` |
| `podSecurityContext.runAsUser` | User ID to run as | `1000` |
| `podSecurityContext.readOnlyRootFilesystem` | Use read-only root filesystem | `true` |
| `basicAuth.enabled` | Enable basic authentication | `false` |
| `basicAuth.username` | Basic auth username | `admin` |

### Resource Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `resources.limits.cpu` | CPU limit | `500m` |
| `resources.limits.memory` | Memory limit | `512Mi` |
| `resources.requests.cpu` | CPU request | `250m` |
| `resources.requests.memory` | Memory request | `128Mi` |

### Health Checks

| Parameter | Description | Default |
|-----------|-------------|---------|
| `healthcheck.enabled` | Enable health checks | `true` |
| `healthcheck.livenessProbe.initialDelaySeconds` | Liveness probe initial delay | `20` |
| `healthcheck.readinessProbe.initialDelaySeconds` | Readiness probe initial delay | `5` |

## Usage Examples

### Basic Installation

```bash
# Install with default values
helm install apprise ./apprise

# Install with ingress enabled
helm install apprise ./apprise \
  --set ingress.enabled=true \
  --set ingress.hosts[0].host=apprise.example.com \
  --set ingress.hosts[0].paths[0].path=/ \
  --set ingress.hosts[0].paths[0].pathType=Prefix
```

### Hardened Security Installation

```bash
helm install apprise ./apprise \
  --set podSecurityContext.readOnlyRootFilesystem=true \
  --set securityContext.runAsNonRoot=true \
  --set securityContext.allowPrivilegeEscalation=false
```

### With Basic Authentication

```bash
helm install apprise ./apprise \
  --set basicAuth.enabled=true \
  --set basicAuth.username=admin \
  --set basicAuth.password=mySecurePassword
```

### API Usage

Once deployed, you can use the Apprise API:

```bash
# Health check
curl http://apprise.example.com/status

# Send notification (stateless)
curl -X POST http://apprise.example.com/notify \
  -H "Content-Type: application/json" \
  -d '{
    "urls": ["mailto://user:pass@gmail.com"],
    "body": "Test notification",
    "title": "Hello from Apprise"
  }'

# Save configuration (with admin enabled)
curl -X POST http://apprise.example.com/add/mykey \
  -H "Content-Type: application/json" \
  -d '{"urls": ["mailto://user:pass@gmail.com"]}'

# Send notification with saved key
curl -X POST http://apprise.example.com/notify/mykey \
  -H "Content-Type: application/json" \
  -d '{
    "body": "Test with saved config",
    "title": "Hello again"
  }'
```

## Security Considerations

- By default, the chart runs with a hardened security context (non-root, read-only filesystem)
- Enable basic authentication for production deployments
- Use TLS/HTTPS with proper ingress configuration
- Consider network policies to restrict access
- Store sensitive configuration in Kubernetes secrets

## Troubleshooting

### Pod fails to start

Check the pod logs:
```bash
kubectl logs deployment/apprise
```

Common issues:
- Storage permissions (ensure the storage class supports the access mode)
- Resource constraints (increase CPU/memory limits)
- Security context conflicts

### Service not accessible

1. Check service and endpoints:
```bash
kubectl get svc,endpoints
```

2. Verify ingress configuration:
```bash
kubectl get ingress
```

3. Test internal connectivity:
```bash
kubectl port-forward deployment/apprise 8000:8000
```

## Support

- [Apprise Documentation](https://github.com/caronc/apprise)
- [Apprise API Documentation](https://github.com/caronc/apprise-api)
- [Official Apprise Website](https://appriseit.com/)