# hyperswitch-istio

![Version: 0.2.0](https://img.shields.io/badge/Version-0.2.0-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 1.16.0](https://img.shields.io/badge/AppVersion-1.16.0-informational?style=flat-square)

A Helm chart for Hyperswitch Istio configuration with integrated Istio components (istio-base, istiod, and istio-gateway).

## Overview

This Helm chart packages:
- Istio base CRDs (istio-base)
- Istio control plane (istiod)
- Istio ingress gateway
- Hyperswitch-specific Istio configurations (Gateway, VirtualService, DestinationRule)
- ALB Ingress configuration for AWS environments

## Prerequisites

- Kubernetes 1.19+
- Helm 3.2.0+
- kubectl configured to access your cluster

## Installation

### Local Development

For local development with default Docker Hub images:

```bash
# Add Istio repository (if not already added)
helm repo add istio https://istio-release.storage.googleapis.com/charts
helm repo update

# Update chart dependencies
helm dependency update ./hyperswitch-istio

# Install the chart
helm install hyperswitch-istio ./hyperswitch-istio \
  --namespace istio-system \
  --create-namespace
```

### Production Deployment (with Private ECR)

For production deployments using private ECR repositories:

```bash
# Update dependencies
helm dependency update ./hyperswitch-istio

# Install with custom values
helm install hyperswitch-istio ./hyperswitch-istio \
  --namespace istio-system \
  --create-namespace \
  --set istiod.global.hub="123456789.dkr.ecr.us-east-1.amazonaws.com/istio" \
  --set istiod.global.tag="1.25.0" \
  --set istiod.pilot.nodeSelector.node-type=memory-optimized \
  --set istio-gateway.global.hub="123456789.dkr.ecr.us-east-1.amazonaws.com/istio" \
  --set istio-gateway.global.tag="1.25.0" \
  --set istio-gateway.nodeSelector.node-type=memory-optimized
```

**Note**: Make sure your ECR repository contains the required Istio images:
- `istio/pilot:1.25.0` (for istiod)
- `istio/proxyv2:1.25.0` (for gateway and sidecars)

### Using a Custom Values File

Create a `custom-values.yaml` file:

```yaml
# Production overrides
istiod:
  global:
    hub: "123456789.dkr.ecr.us-east-1.amazonaws.com/istio"
    tag: "1.25.0"
  pilot:
    nodeSelector:
      node-type: memory-optimized

istio-gateway:
  global:
    hub: "123456789.dkr.ecr.us-east-1.amazonaws.com/istio"
    tag: "1.25.0"
  nodeSelector:
    node-type: memory-optimized

# ALB Ingress annotations
ingress:
  annotations:
    alb.ingress.kubernetes.io/scheme: internal
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/security-groups: sg-xxxxxxxxx
    alb.ingress.kubernetes.io/subnets: subnet-xxx,subnet-yyy
```

Then install:

```bash
helm install hyperswitch-istio ./hyperswitch-istio \
  --namespace istio-system \
  --create-namespace \
  -f custom-values.yaml
```

## Uninstallation

To uninstall the chart:

```bash
helm uninstall hyperswitch-istio -n istio-system
```

Note: This will remove all Istio components. If you have other applications using Istio, consider carefully before uninstalling.

## Configuration

The following table lists the configurable parameters and their default values.

### Hyperswitch Configuration

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| hyperswitchServer.version | string | `"v1o114o0"` | Hyperswitch server (router) version |
| hyperswitchControlCenter.version | string | `"v1o37o1"` | Hyperswitch control center version |
| image.version | string | `"v1o107o0"` | Legacy image version (deprecated) |
| ingress.annotations | object | `{}` | ALB Ingress annotations |
| ingress.className | string | `"alb"` | Ingress class name |
| ingress.enabled | bool | `true` | Enable ingress |
| ingress.hosts.paths[0].name | string | `"istio-ingressgateway"` | Service name for root path |
| ingress.hosts.paths[0].path | string | `"/"` | Root path |
| ingress.hosts.paths[0].pathType | string | `"Prefix"` | Path type |
| ingress.hosts.paths[0].port | int | `80` | Service port |
| ingress.hosts.paths[1].name | string | `"istio-ingressgateway"` | Service name for health check |
| ingress.hosts.paths[1].path | string | `"/healthz/ready"` | Health check path |
| ingress.hosts.paths[1].pathType | string | `"Prefix"` | Path type |
| ingress.hosts.paths[1].port | int | `15021` | Health check port |
| ingress.tls | list | `[]` | TLS configuration |
| livenessProbe.httpGet.path | string | `"/"` | Liveness probe path |
| livenessProbe.httpGet.port | string | `"http"` | Liveness probe port |
| readinessProbe.httpGet.path | string | `"/"` | Readiness probe path |
| readinessProbe.httpGet.port | string | `"http"` | Readiness probe port |
| replicaCount | int | `1` | Number of replicas |
| service.port | int | `80` | Service port |
| service.type | string | `"ClusterIP"` | Service type |
| createNamespace | bool | `true` | Create istio-system namespace |

### Istio Components Configuration

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| istio-base.enabled | bool | `true` | Enable Istio base CRDs |
| istio-base.defaultRevision | string | `"default"` | Default revision |
| istiod.enabled | bool | `true` | Enable Istiod |
| istiod.pilot.nodeSelector | object | `{}` | Node selector for Istiod |
| istio-gateway.enabled | bool | `true` | Enable Istio gateway |
| istio-gateway.name | string | `"istio-ingressgateway"` | Gateway name |
| istio-gateway.service.type | string | `"ClusterIP"` | Gateway service type |
| istio-gateway.nodeSelector | object | `{}` | Node selector for gateway |

## Troubleshooting

### Check Istio Installation

```bash
# Check if all Istio components are running
kubectl get pods -n istio-system

# Check CRDs
kubectl get crd | grep istio

# Check services
kubectl get svc -n istio-system
```

### Verify Gateway Configuration

```bash
# Check gateway
kubectl get gateway -n istio-system

# Check virtual services
kubectl get virtualservice -A

# Check destination rules
kubectl get destinationrule -A
```

### Debug Ingress Issues

```bash
# Check ingress
kubectl get ingress -n istio-system

# Describe ingress for events
kubectl describe ingress hyperswitch-istio-app-alb-ingress -n istio-system
```

----------------------------------------------
Autogenerated from chart metadata using [helm-docs v1.14.2](https://github.com/norwoodj/helm-docs/releases/v1.14.2)
