# Installation Guide

This guide provides a straightforward path to get Simba Intelligence up and running. For detailed technical configuration options, see the [Helm Chart Reference](../deployment/installation-and-setup/Helm-Chart-Reference).

## Prerequisites

Before you begin, ensure you have:

- A Kubernetes cluster (1.24+)
- [Helm](https://helm.sh/) (3.17+) installed
- `kubectl` configured for your cluster
- An AI/LLM provider account (Google Vertex AI, OpenAI, Azure OpenAI, or AWS Bedrock)

## Quick Installation

### 1. Add the Helm Repository and Install

For a basic installation with ingress enabled, save the following YAML configuration to a file named `simba-values.yaml`:

**Basic setup (accessible via localhost or cluster IP):**

```yaml
ingress:
  appendToPath: ""
  trimTrailingSlash: true
  enabled: true
  className: "traefik"
  annotations: {}
  hosts:
    - paths:
        - path: /
          pathType: ImplementationSpecific
```

**With hostname (requires proper DNS setup):**

```yaml
ingress:
  appendToPath: ""
  trimTrailingSlash: true
  enabled: true
  className: "traefik"
  annotations: {}
  hosts:
    - host: "simba.yourdomain.com"
      paths:
        - path: /
          pathType: ImplementationSpecific
```

> **🌐 DNS Setup:** If using a hostname, ensure your DNS is configured to point to your cluster's ingress controller. See the documentation for your specific ingress controller: [NGINX Ingress](https://kubernetes.github.io/ingress-nginx/deploy/), [Traefik](https://doc.traefik.io/traefik/providers/kubernetes-ingress/), or [AWS Load Balancer Controller](https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.4/guide/ingress/ingress_class/).

```shell
# Install Simba Intelligence using Helm with the values file
helm install simba-intelligence oci://docker.io/insightsoftware/simba-intelligence-chart --version <VERSION> -f simba-values.yaml
```

> **📋 Note:** Replace `<VERSION>` with the latest version. Find available versions on the [Docker Hub tags page](https://hub.docker.com/r/insightsoftware/simba-intelligence-chart/tags).

> **🔧 Ingress Controller:** This example uses Traefik as the ingress class. Adjust the `className` to match your cluster's ingress controller (e.g., "nginx", "alb", etc.).

### 2. Access the Application

After installation, check your ingress status:

```shell
kubectl get ingress
```

**If using the basic setup (no hostname):**

- Access via your cluster's external IP or localhost (if running locally)
- The ingress will route traffic based on the path only

**If using a hostname:**

- Access at your configured hostname (e.g., `http://simba.yourdomain.com`)
- Ensure DNS is properly configured to point to your ingress controller

> **💡 Production Tip:** For production deployments, always use proper hostnames with TLS certificates and DNS. See the [Helm Chart Reference](../deployment/installation-and-setup/Helm-Chart-Reference) for complete ingress configuration options including SSL/TLS setup.

### 3. Initial Configuration

1. Log in with the default administrator credentials (provided in the installation output)
2. Configure your LLM provider in the UI at `/llm-configuration`
3. Set up your first data source

## Need More Control?

For advanced configuration options, custom values, scaling parameters, security settings, and detailed deployment scenarios, see:

➡️ [**Helm Chart Reference**](../deployment/installation-and-setup/Helm-Chart-Reference) - Complete configuration guide

The Helm Chart Reference provides:

- All available configuration values
- Environment-specific deployment examples
- Security and scaling best practices
- Troubleshooting common issues

## Next Steps

Once installed, continue with the [Quick Start Guide](./Quick-Start-Guide) to:

- Configure your LLM provider
- Set up data sources
- Enable user access

## Need Help?

- **Configuration Details**: [Helm Chart Reference](../deployment/installation-and-setup/Helm-Chart-Reference)
- **System Requirements**: [System Requirements](./System-Requirements)
- **Operational Guidance**: [Administrator Guide](../guides/admin-guides/Administrator-Guide)