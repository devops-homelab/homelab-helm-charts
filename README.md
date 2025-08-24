# Homelab Helm Charts

This repository contains enterprise-grade Helm charts for managing and deploying cloud-native applications in your homelab environment. The charts support both traditional Ingress and modern Gateway API implementations, with advanced features like blue-green deployments, automatic TLS, and preview environments.

## Repository Structure

```
charts/
└── common-application-charts/
    ├── Chart.yaml                    # Chart metadata (v1.0.12)
    ├── values.yaml                   # Default configuration values
    └── templates/
        ├── _helpers.tpl              # Template helpers and functions
        ├── certificate.yaml          # TLS certificates for production
        ├── certificate-preview.yaml  # TLS certificates for preview
        ├── gateway.yaml              # Gateway API gateway resource
        ├── httproute.yaml            # HTTPRoute for production traffic
        ├── httproute-preview.yaml    # HTTPRoute for preview traffic
        ├── ingress.yaml              # Traditional ingress support
        ├── rollout.yaml              # Argo Rollouts deployment
        └── service.yaml              # Kubernetes service
```

## Chart Features

### Dual Network Implementation Support
- **Traditional Ingress**: Compatible with existing Kong/NGINX ingress controllers
- **Gateway API v1.0.0**: Modern Kubernetes networking with advanced traffic management
- **Seamless Migration**: Run both implementations side-by-side

### Advanced Deployment Strategies
- **Blue-Green Deployments**: Zero-downtime deployments with Argo Rollouts
- **Preview Environments**: Test new versions before promotion
- **Traffic Splitting**: Gradual rollouts with percentage-based traffic control
- **Automatic Rollbacks**: Health-based deployment validation

### TLS & Security
- **Automatic HTTPS**: cert-manager integration with Let's Encrypt
- **Multi-domain Support**: Production and preview domain certificates
- **Kong Integration**: Advanced security policies and rate limiting
- **Network Policies**: Micro-segmentation support

### Production-Ready Features
- **Health Checks**: Kubernetes liveness and readiness probes
- **Resource Management**: CPU/memory limits and requests
- **Environment Variables**: Secrets and ConfigMap injection
- **Monitoring**: Prometheus metrics and ServiceMonitor support

## Chart Values Structure

### Traditional Ingress Configuration
```yaml
ingress:
  enabled: true
  ingressClassName: kong
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
  hostname: myapp.domain.com
  tls:
    enabled: true
```

### Gateway API Configuration
```yaml
gateway:
  enabled: true
  gatewayClassName: kong
  listeners:
    - name: https
      port: 443
      protocol: HTTPS
      hostname: myapp.domain.com

httpRoute:
  enabled: true
  preview:
    enabled: true
    hostnames:
      - preview.myapp.domain.com

certificate:
  enabled: true
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
```

### Blue-Green Deployment
```yaml
strategy:
  type: blueGreen
  blueGreen:
    activeService: myapp-active
    previewService: myapp-preview
    autoPromotionEnabled: false
```

## Usage

### Prerequisites
- [Helm v3.8+](https://helm.sh/) installed
- Kubernetes cluster with Gateway API CRDs (if using Gateway API)
- Kong ingress controller with Gateway API support
- cert-manager for automatic TLS certificates
- Argo Rollouts controller (for blue-green deployments)

### Installation Methods

#### From Helm Repository
```bash
# Add the repository
helm repo add homelab-charts https://raw.githubusercontent.com/devops-homelab/homelab-helm-charts/gh-pages

# Install with traditional ingress
helm install myapp homelab-charts/common-application-charts \
  --set ingress.enabled=true \
  --set ingress.hostname=myapp.domain.com

# Install with Gateway API
helm install myapp homelab-charts/common-application-charts \
  --set gateway.enabled=true \
  --set httpRoute.enabled=true \
  --set gateway.listeners[0].hostname=myapp.domain.com
```

#### From Source
```bash
# Clone the repository
git clone https://github.com/devops-homelab/homelab-helm-charts.git
cd homelab-helm-charts/charts/common-application-charts

# Install with custom values
helm install myapp . -f custom-values.yaml
```

### Configuration Examples

#### Basic Web Application
```yaml
app:
  name: web-app
  version: v1.0.0

image:
  repository: nginx
  tag: latest

service:
  port: 80

ingress:
  enabled: true
  hostname: web.domain.com
```

#### Microservice with Gateway API
```yaml
app:
  name: api-service
  version: v2.1.0

gateway:
  enabled: true
  gatewayClassName: kong

httpRoute:
  enabled: true
  hostnames:
    - api.domain.com
  preview:
    enabled: true
    hostnames:
      - preview.api.domain.com

strategy:
  type: blueGreen
  blueGreen:
    autoPromotionEnabled: false
    scaleDownDelaySeconds: 30
```

### Advanced Operations

#### Upgrade with Preview Testing
```bash
# Deploy new version to preview
helm upgrade myapp . \
  --set image.tag=v2.0.0 \
  --set strategy.blueGreen.autoPromotionEnabled=false

# Test preview environment
curl https://preview.myapp.domain.com

# Promote to production
kubectl argo rollouts promote myapp
```

#### Traffic Management
```bash
# Split traffic 90/10 between versions
kubectl argo rollouts set image myapp app=myimage:v2.0.0
kubectl argo rollouts promote myapp --weight=10
```

## Continuous Integration

### Chart Release Pipeline
- **Automated Versioning**: Semantic versioning with Git tags
- **Quality Gates**: Helm lint, chart testing, and security scans
- **Registry Publishing**: Charts published to GitHub Pages
- **Change Detection**: Only modified charts are released

### Testing Strategy
- **Unit Tests**: Template rendering validation
- **Integration Tests**: Deployment validation on real clusters
- **Security Scanning**: Chart and container vulnerability analysis
- **Compatibility Testing**: Multi-version Kubernetes support

## Migration Guide

### From Traditional Ingress to Gateway API

1. **Phase 1: Parallel Deployment**
```bash
# Deploy Gateway API alongside existing Ingress
helm upgrade myapp . \
  --set gateway.enabled=true \
  --set httpRoute.enabled=true \
  --reuse-values
```

2. **Phase 2: Traffic Validation**
```bash
# Test Gateway API endpoints
curl -H "Host: myapp.domain.com" https://gateway-ip
```

3. **Phase 3: DNS Cutover**
```bash
# Update DNS to point to Gateway API
# Disable traditional ingress
helm upgrade myapp . \
  --set ingress.enabled=false \
  --reuse-values
```

## Contributing

We welcome contributions! Please follow these guidelines:

1. **Chart Changes**: Update chart version in `Chart.yaml`
2. **Testing**: Ensure all tests pass locally
3. **Documentation**: Update README for significant changes
4. **Security**: Follow security best practices
5. **Compatibility**: Maintain backward compatibility when possible

### Development Workflow
```bash
# Lint charts
helm lint charts/common-application-charts

# Test locally
helm template charts/common-application-charts

# Validate with real cluster
helm install test-release charts/common-application-charts --dry-run
```

## Chart Versions

| Version | Features | Compatibility |
|---------|----------|---------------|
| 1.0.12  | Gateway API, Preview routing, Separate certificates | K8s 1.25+ |
| 1.0.11  | Gateway API support, Blue-green deployments | K8s 1.24+ |
| 1.0.10  | TLS automation, Kong integration | K8s 1.23+ |

## License

This repository is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

---

**Gateway API Ready • GitOps Enabled • Production Tested**