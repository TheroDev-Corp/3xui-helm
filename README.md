# 3xui Helm Chart

A Helm chart for deploying the 3x-ui panel ([mhsanaei/3x-ui](https://github.com/mhsanaei/3x-ui)) on Kubernetes. This chart is designed to support `hostNetwork` and `cert-manager` for easily setting up Xray VPN protocols with automatic Let's Encrypt certificates.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.0+
- PV provisioner support in the underlying infrastructure
- [cert-manager](https://cert-manager.io/) (if ingress and TLS are enabled)
- An Ingress Controller (like Traefik or NGINX)

## Installation

### 1. Install the Chart

```bash
helm install my-3xui ./3xui-helm -n xui-namespace --create-namespace
```

## Configuration

The following table lists the configurable parameters of the 3x-ui chart and their default values.

| Parameter | Description | Default |
|-----------|-------------|---------|
| `nameOverride` | Override the name of the chart | `"xui"` |
| `fullnameOverride` | Override the full name of the release | `"xui-panel"` |
| `image.repository` | 3x-ui image repository | `ghcr.io/mhsanaei/3x-ui` |
| `image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `image.tag` | 3x-ui image tag | `"latest"` |
| `hostNetwork` | Enable hostNetwork (required for most VPN protocols) | `true` |
| `env.XRAY_VMESS_AEAD_FORCED` | Environment variable passed to 3x-ui container | `"false"` |
| `env.XUI_ENABLE_FAIL2BAN` | Environment variable passed to 3x-ui container | `"true"` |
| `persistence.enabled` | Enable persistent volume | `true` |
| `persistence.storageClassName` | Storage class name (leave empty for default) | `""` |
| `persistence.accessMode` | PVC access mode | `ReadWriteOnce` |
| `persistence.size` | PVC storage size | `1Gi` |
| `service.type` | Kubernetes service type | `ClusterIP` |
| `service.panelPort` | Panel web interface port | `2053` |
| `service.subPort` | Subscription port | `2096` |
| `ingress.enabled` | Enable ingress resource | `true` |
| `ingress.className` | Ingress class name | `"traefik"` |
| `ingress.annotations` | Annotations for Ingress resource | `cert-manager.io/cluster-issuer: "letsencrypt-prod"` |
| `ingress.host` | Hostname for the ingress routing | `admin-panel.example.com` |
| `ingress.tlsSecretName` | Secret name to store the generated TLS certificate | `3xui-tls-secret` |
| `serviceAccount.create` | Specifies whether a service account should be created | `true` |
| `serviceAccount.name` | Name of the service account | `""` |
| `resources.requests.cpu` | CPU request | `500m` |
| `resources.requests.memory` | Memory request | `512Mi` |
| `resources.limits.cpu` | CPU limit | `1000m` |
| `resources.limits.memory` | Memory limit | `1Gi` |
| `nodeSelector` | Node selector string | `{}` |
| `tolerations` | Tolerations for pods | `[]` |

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`. For example:

```bash
helm install my-3xui ./3xui-helm \
  --set ingress.host=panel.mydomain.com \
  --set service.panelPort=2053
```

Alternatively, a YAML file that specifies the values for the above parameters can be provided while installing the chart. For example:

```bash
helm install my-3xui ./3xui-helm -f my-values.yaml
```

## Special Notes

- **hostNetwork:** By default `hostNetwork` is enabled (`true`). This is typically required so the Xray node can listen on the host's real IP and ports for various VPN protocols (VLESS, VMess, Trojan, etc.).
- **cert-manager:** The default ingress configuration includes annotations to issue Let's Encrypt certificates automatically via `cert-manager`. Make sure you have a `ClusterIssuer` named `letsencrypt-prod` (or update `ingress.annotations` accordingly).
- **Storage:** The container maps `/etc/x-ui/` to a persistent volume so that the SQLite database (and x-ui config) survives pod restarts.