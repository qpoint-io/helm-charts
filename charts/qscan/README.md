# QScan Helm Chart

A Helm chart for deploying QScan PII detection worker on Kubernetes.

## Overview

QScan is a PII (Personal Identifiable Information) detection worker service that processes HTTP transactions to identify and flag sensitive data. It polls for scanning jobs from the Pulse scheduler, retrieves artifacts from S3-compatible storage, runs ML-based PII detection models, and uploads scan reports back to S3.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.0+
- Access to Pulse API endpoint
- AWS S3 or S3-compatible storage credentials
- (Optional) GPU-enabled nodes for accelerated processing

## Installation

### Add the Qpoint Helm repository

```bash
helm repo add qpoint https://helm.qpoint.io/
helm repo update
```

### Install the chart with external secrets (recommended for production)

```bash
# Create a Kubernetes secret with credentials
kubectl create secret generic qscan-credentials \
  --from-literal=pulse-api-key=<YOUR_PULSE_API_KEY> \
  --from-literal=AWS_ACCESS_KEY_ID=<YOUR_AWS_ACCESS_KEY_ID> \
  --from-literal=AWS_SECRET_ACCESS_KEY=<YOUR_AWS_SECRET_ACCESS_KEY>

# Install the chart referencing the external secret
helm install qscan qpoint/qscan \
  --set pulse.apiKeySecretRefName=qscan-credentials \
  --set aws.credentials.secretRefName=qscan-credentials
```

### Install with inline credentials (testing only)

```bash
helm install qscan qpoint/qscan \
  --set pulse.apiKey=<YOUR_PULSE_API_KEY> \
  --set aws.credentials.accessKeyId=<YOUR_AWS_ACCESS_KEY_ID> \
  --set aws.credentials.secretAccessKey=<YOUR_AWS_SECRET_ACCESS_KEY>
```

## Configuration

### Key Configuration Parameters

| Parameter                       | Description                              | Default                                       |
| ------------------------------- | ---------------------------------------- | --------------------------------------------- |
| `image.repository`              | QScan worker image repository            | `us-docker.pkg.dev/qpoint-edge/private/qscan` |
| `image.tag`                     | Image tag (defaults to chart appVersion) | `""`                                          |
| `replicaCount`                  | Number of worker replicas                | `1`                                           |
| `pulse.endpoint`                | Pulse API endpoint URL                   | `https://pulse.qpoint.io`                     |
| `pulse.apiKeySecretRefName`     | External secret name for Pulse API key   | `""`                                          |
| `pulse.apiKey`                  | Inline Pulse API key (testing only)      | `""`                                          |
| `aws.s3.bucketName`             | S3 bucket name for artifacts             | `assets`                                      |
| `aws.s3.regionName`             | AWS region                               | `us-east-1`                                   |
| `aws.s3.endpointUrl`            | S3 endpoint URL                          | `https://s3.warehouse.qpoint.io`              |
| `aws.credentials.secretRefName` | External secret name for AWS credentials | `""`                                          |
| `qscan.numPollers`              | Number of async job pollers              | `2`                                           |
| `qscan.numScanners`             | Number of ML scanner workers             | `2`                                           |
| `qscan.logLevel`                | Logging level                            | `info`                                        |
| `resources.limits.cpu`          | CPU limit                                | `4000m`                                       |
| `resources.limits.memory`       | Memory limit                             | `16Gi`                                        |
| `gpu.enabled`                   | Enable GPU support                       | `false`                                       |
| `autoscaling.enabled`           | Enable horizontal pod autoscaling        | `false`                                       |

### Examples

#### Enable autoscaling

```bash
helm install qscan qpoint/qscan \
  --set autoscaling.enabled=true \
  --set autoscaling.minReplicas=2 \
  --set autoscaling.maxReplicas=10 \
  --set pulse.apiKeySecretRefName=qscan-credentials \
  --set aws.credentials.secretRefName=qscan-credentials
```

#### Enable GPU support

```bash
helm install qscan qpoint/qscan \
  --set gpu.enabled=true \
  --set gpu.count=1 \
  --set pulse.apiKeySecretRefName=qscan-credentials \
  --set aws.credentials.secretRefName=qscan-credentials
```

For GPU support, ensure your cluster has GPU-enabled nodes with the NVIDIA device plugin installed. You may also need to add node selectors or tolerations:

```yaml
gpu:
  enabled: true
  count: 1

nodeSelector:
  cloud.google.com/gke-accelerator: nvidia-tesla-t4

tolerations:
  - key: nvidia.com/gpu
    operator: Exists
    effect: NoSchedule
```

#### Custom worker configuration

```bash
helm install qscan qpoint/qscan \
  --set qscan.numPollers=4 \
  --set qscan.numScanners=4 \
  --set resources.limits.cpu=8000m \
  --set resources.limits.memory=32Gi \
  --set pulse.apiKeySecretRefName=qscan-credentials \
  --set aws.credentials.secretRefName=qscan-credentials
```

#### Custom model configuration

Create a values file with custom config:

```yaml
config: |
  global:
    device: cuda
    log_level: DEBUG

  models:
    piiranha:
      enabled: true
      model_name: iiiorg/piiranha-v1-detect-personal-information

    presidio:
      enabled: false

    flair:
      enabled: true
      model_name: flair/ner-english-large

pulse:
  apiKeySecretRefName: qscan-credentials

aws:
  credentials:
    secretRefName: qscan-credentials
```

Install with the custom values:

```bash
helm install qscan qpoint/qscan -f custom-values.yaml
```

## Monitoring

QScan exposes Prometheus metrics on port 9090 at the `/metrics` endpoint. A Kubernetes Service is automatically created for metrics scraping.

### Prometheus ServiceMonitor example

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: qscan
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: qscan
  endpoints:
    - port: metrics
      interval: 30s
```

### Manual scraping configuration

If not using the Prometheus Operator, configure scraping via annotations:

```yaml
service:
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "9090"
    prometheus.io/path: "/"
```

## Security Best Practices

1. **Use external secrets**: Always prefer `secretRefName` over inline credentials in production
2. **Use a secrets manager**: Consider integrating with External Secrets Operator, Sealed Secrets, or cloud-native secrets managers
3. **Limit permissions**: Use RBAC to restrict service account permissions
4. **Network policies**: Implement Kubernetes Network Policies to control traffic
5. **Run as non-root**: The default security context runs as non-root user (UID 1000)

## Troubleshooting

### Pods not starting

Check pod events and logs:

```bash
kubectl describe pod -l app.kubernetes.io/name=qscan
kubectl logs -l app.kubernetes.io/name=qscan
```

### Common issues

1. **Missing credentials**: Ensure secrets are created and referenced correctly
2. **Model loading timeout**: Increase `startupProbe.failureThreshold` if models take longer to load
3. **Out of memory**: Increase `resources.limits.memory` or reduce `qscan.numScanners`
4. **Connection to Pulse fails**: Check network policies and firewall rules

### Health check endpoints

- Metrics: `http://<pod-ip>:9090/metrics`
- Health: `http://<pod-ip>:9090/`

### Debugging

Enable debug logging:

```bash
helm upgrade qscan qpoint/qscan --set qscan.logLevel=debug
```

View logs:

```bash
kubectl logs -f deployment/qscan
```

## Upgrading

### Upgrade to a new chart version

```bash
helm repo update
helm upgrade qscan qpoint/qscan
```

### Upgrade with custom values

```bash
helm upgrade qscan qpoint/qscan -f custom-values.yaml
```

## Uninstalling

```bash
helm uninstall qscan
```

This removes all Kubernetes resources associated with the chart. Secrets created manually must be deleted separately:

```bash
kubectl delete secret qscan-credentials
```

## Development

### Lint the chart

```bash
helm lint charts/qscan
```

### Test template rendering

```bash
helm template qscan charts/qscan
```

### Install from local chart

```bash
helm install qscan ./charts/qscan -f values.yaml
```

## Support

For issues, questions, or contributions, please visit the [Qpoint Helm Charts repository](https://github.com/qpoint/helm-charts).
