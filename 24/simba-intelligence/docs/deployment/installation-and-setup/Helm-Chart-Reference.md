# Helm Chart Reference Guide

This guide teaches you how to discover and understand the Simba Intelligence Helm chart configuration options, dependencies, and deployment patterns by exploring the chart files directly. This approach ensures you always have the most up-to-date and accurate information.

## Prerequisites

### Install Helm

Before you can explore the chart files, you need to have Helm installed on your system. 

**📖 Installation Guide:** Follow the official [Helm Installation Guide](https://helm.sh/docs/intro/install/) for step-by-step instructions on your platform.

**Verify installation:**
```bash
helm version
```

### Pull and Extract the Chart

Once Helm is installed, you have two options for exploring the chart:

**First, find available versions:**

To find available chart versions, visit the [Docker Hub repository tags page](https://hub.docker.com/r/insightsoftware/simba-intelligence-chart/tags) for the Simba Intelligence chart.

> **🏷️ Choosing the Right Tag:** Look for tags that follow semantic versioning patterns like `25.3.0`, `26.1.0`, etc. These are chart versions. Avoid tags like `latest`, `main`, or `dev` which may be unstable. Sort by "Last updated" to find the most recent stable releases.

**Option 1: View chart information directly (no download needed)**
```bash
# View values without downloading (replace <VERSION> with actual version)
helm show values oci://docker.io/insightsoftware/simba-intelligence-chart --version <VERSION>

# View README without downloading  
helm show readme oci://docker.io/insightsoftware/simba-intelligence-chart --version <VERSION>

# View all chart information
helm show all oci://docker.io/insightsoftware/simba-intelligence-chart --version <VERSION>
```

**Option 2: Download and extract for detailed file exploration**

**Linux/macOS/WSL:**
```bash
# Pull the chart from the OCI registry (replace <VERSION> with actual version)
helm pull oci://docker.io/insightsoftware/simba-intelligence-chart --version <VERSION>

# Extract the chart files for exploration
tar -xzf simba-intelligence-chart-<VERSION>.tgz

# Navigate into the chart directory
cd simba-intelligence-chart/
```

**Windows PowerShell:**
```powershell
# Pull the chart from the OCI registry (replace <VERSION> with actual version)
helm pull oci://docker.io/insightsoftware/simba-intelligence-chart --version <VERSION>

# Extract the chart files for exploration
Expand-Archive -Path "simba-intelligence-chart-<VERSION>.tgz" -DestinationPath "."

# Navigate into the chart directory
cd simba-intelligence-chart\
```

> **📝 Note:** Replace `<VERSION>` with the specific version you want to explore (e.g., 25.3.0, 26.1.0, etc.). Use the version discovery commands above to find available versions.
> 
> **Quick Start:** Use Option 1 for fast access to values and documentation. Use Option 2 if you need to examine template files or prefer working with local files.

## Overview

### Understanding the Helm Chart Structure

Rather than maintaining static documentation, this guide shows you how to explore the Simba Intelligence Helm chart to understand its capabilities and configuration options.

**📋 What You'll Learn:**
- How to inspect chart metadata and dependencies
- How to discover available configuration values
- How to understand component relationships
- How to find installation and upgrade guidance

### Chart Files You'll Explore

The key files that contain all the information you need:

| File | Contains | How to Use |
|------|----------|------------|
| `Chart.yaml` | Chart metadata, version, dependencies | Shows chart name, version, and subchart dependencies |
| `values.yaml` | Default configuration values | Complete configuration schema with defaults |
| `README.md` | Installation and usage guidance | Specific instructions for this chart |
| `templates/` | Kubernetes resource definitions | Shows what components are deployed |

---

## Discovering Chart Information

### Step 1: Examine Chart Metadata

First, look at the `Chart.yaml` file to understand the chart basics:

**Linux/macOS/WSL:**
```bash
# Ensure you're in the extracted chart directory
cd simba-intelligence-chart/

# View chart metadata
cat Chart.yaml
```

**Windows PowerShell:**
```powershell
# Ensure you're in the extracted chart directory
cd simba-intelligence-chart\

# View chart metadata
Get-Content Chart.yaml
```

**Key information you'll find:**
- Chart name and version
- Application version
- Dependencies and their versions
- Chart description and type

**Example output:**
```yaml
name: simba-intelligence-chart
version: 26.1.0  # This will vary by release
appVersion: "1.17.0"  # This will vary by release
dependencies:
  - name: postgresql
    version: 11.6.3  # Dependency versions will vary by chart release
    repository: https://charts.bitnami.com/bitnami
  - name: redis
    version: 21.2.14  # Check your actual Chart.yaml for current versions
    repository: https://charts.bitnami.com/bitnami
  # ... other dependencies shown in your actual file
```

> **🔍 Version Discovery:** The exact dependency versions depend on your chart version. Always check your actual `Chart.yaml` file for the current dependency versions used in your specific chart release.

### Step 2: Explore Dependencies

Understanding what subcharts are included:

**Linux/macOS/WSL:**
```bash
# View dependency lock file
cat Chart.lock

# List downloaded dependencies
ls charts/
```

**Windows PowerShell:**
```powershell
# View dependency lock file
Get-Content Chart.lock

# List downloaded dependencies
Get-ChildItem charts\
```

This shows you exactly which versions of PostgreSQL, Redis, Composer, and other components are bundled.

### Step 3: Discover Available Components

Examine the templates directory to see what Kubernetes resources will be created:

**Linux/macOS/WSL:**
```bash
# List all template files
ls templates/
```

**Windows PowerShell:**
```powershell
# List all template files
Get-ChildItem templates\
```

**Example output shows:**
```
# simba-intelligence-deployment.yaml      - Main web application
# simba-intelligence-worker-deployment.yaml - Celery workers  
# simba-intelligence-beat-statefulset.yaml  - Celery scheduler
# simba-intelligence-db-migrate-job.yaml    - Database migrations
# simba-intelligence-hpa.yaml               - Auto-scaling
# ingress.yaml                               - External access
# ... and more
```

---

## Installation Discovery

### How to Find Installation Instructions

The chart's `README.md` file contains the most current installation instructions:

**Linux/macOS/WSL:**
```bash
# Read the chart's installation guide (from extracted chart directory)
cat README.md
```

**Windows PowerShell:**
```powershell
# Read the chart's installation guide (from extracted chart directory)
Get-Content README.md
```

**What you'll typically find:**
- Prerequisites and system requirements
- Step-by-step installation commands
- Database setup strategies
- Configuration examples specific to this chart version

### Using Standard Helm Commands

Once you have the chart, you can install it using either approach:

**Option 1: Install directly from OCI registry**
```bash
# Install directly from OCI registry (replace <VERSION> with actual version)
helm install simba-intelligence oci://docker.io/insightsoftware/simba-intelligence-chart \
  --version <VERSION> \
  --namespace simba-intelligence \
  --create-namespace
```

**Option 2: Install from extracted local chart**
```bash
# Get help on installation options
helm install --help

# Validate your configuration before installing (using extracted chart)
helm install simba-intelligence ./simba-intelligence-chart \
  --dry-run --debug

# Install with default values (using extracted chart)
helm install simba-intelligence ./simba-intelligence-chart \
  --namespace simba-intelligence \
  --create-namespace
```

> **💡 Pro Tip:** Always use `--dry-run --debug` first to validate your configuration and see what resources will be created.

For more detailed Helm installation guidance, see the [official Helm documentation](https://helm.sh/docs/helm/helm_install/).

---

## Configuration Discovery

Now that you know how to install the chart, you'll likely want to customize it for your environment. The Simba Intelligence chart offers extensive configuration options for different deployment scenarios, resource requirements, and integration needs.

This section teaches you how to discover and understand all available configuration options by exploring the chart's documentation and default values.

### Step 1: Explore the Values Schema

The `values.yaml` file is your complete configuration reference:

**Linux/macOS/WSL:**
```bash
# View all available configuration options (from extracted chart)
cat values.yaml

# Or use Helm to show values directly from OCI registry (replace <VERSION>)
helm show values oci://docker.io/insightsoftware/simba-intelligence-chart --version <VERSION>

# Or from extracted chart directory
helm show values ./simba-intelligence-chart
```

**Windows PowerShell:**
```powershell
# View all available configuration options (from extracted chart)
Get-Content values.yaml

# Or use Helm to show values directly from OCI registry (replace <VERSION>)
helm show values oci://docker.io/insightsoftware/simba-intelligence-chart --version <VERSION>

# Or from extracted chart directory (use forward slashes for helm)
helm show values ./simba-intelligence-chart
```

**Understanding the values structure:**
```yaml
# Top-level sections you'll find:
global:           # Settings shared across subcharts
simba:            # Simba Intelligence specific configuration
postgresql:       # Database configuration (if enabled)
redis:            # Cache configuration (if enabled)
discovery:        # Composer/Discovery configuration (if enabled)
```

> **📋 Configuration Documentation:** The chart's `README.md` contains comprehensive tables that explain what each configuration key does, including descriptions, data types, and default values. This is your best resource for understanding available configuration options.

### Step 2: Find Configuration Examples

The chart's `README.md` contains configuration examples and usage scenarios. Simply read through the README to find:

- Common configuration patterns
- Example values files for different deployment scenarios  
- Production vs. development configuration differences
- Integration examples with external services

**Linux/macOS/WSL:**
```bash
# Read the complete README for configuration guidance
cat README.md
```

**Windows PowerShell:**
```powershell
# Read the complete README for configuration guidance
Get-Content README.md
```

### Step 3: Discover Dependencies Configuration

Understanding how subchart dependencies are configured involves examining both the subchart defaults and how they're customized for Simba Intelligence.

**First, check what dependencies are included** by reading your Chart.yaml (as shown in Step 1 above).

**Then, explore dependency configuration:**

**Linux/macOS/WSL:**
```bash
# Read your main values.yaml to see dependency overrides
cat values.yaml

# Read the chart's README for dependency configuration guidance
cat README.md
```

**Windows PowerShell:**
```powershell
# Read your main values.yaml to see dependency overrides  
Get-Content values.yaml

# Read the chart's README for dependency configuration guidance
Get-Content README.md
```

**What to look for in values.yaml:**
- `postgresql:` section - Database configuration overrides
- `redis:` section - Cache configuration overrides  
- `discovery:` section - Composer/Discovery configuration overrides
- `global:` section - Settings shared across all subcharts

**What to look for in README.md:**
- Dependency configuration tables explaining available options
- Examples of enabling/disabling bundled services
- External service connection examples

> **⚠️ Important:** Not all configuration values available in the child charts (PostgreSQL, Redis, etc.) will be compatible with the Simba Intelligence chart. For heavy customization of dependencies, it is recommended to disable the bundled subcharts and use external service options instead. Check the chart's `README.md` for details on configuring external services.

---

## Chart Management

For comprehensive guidance on managing Helm charts, refer to the official Helm documentation:

- **[Helm User Guide](https://helm.sh/docs/intro/using_helm/)** - Complete guide to using Helm
- **[Managing Charts](https://helm.sh/docs/helm/)** - Chart installation, upgrade, and rollback commands
- **[Values Files](https://helm.sh/docs/chart_best_practices/values/)** - Best practices for configuration
- **[Troubleshooting](https://helm.sh/docs/topics/troubleshooting/)** - Common issues and solutions

### Quick Reference Commands

```bash
# Chart management basics
helm list                                    # List installed releases
helm status <release-name>                   # Check release status
helm history <release-name>                  # View release history
helm upgrade <release-name> <chart>          # Upgrade release
helm rollback <release-name> <revision>      # Rollback to previous version
helm uninstall <release-name>                # Remove release

# Configuration and validation
helm show values <chart>                     # View default values
helm get values <release-name>               # Get current release values
helm diff upgrade <release> <chart>          # Preview changes (requires helm-diff plugin)
```

---

## Helpful Resources

### Chart-Specific Resources

Always check these files in your chart directory for the most current information:
- `README.md` - Installation and configuration guide
- `values.yaml` - Complete configuration schema
- `Chart.yaml` - Dependencies and metadata
- `NOTES.txt` (if present) - Post-installation instructions

### Basic Troubleshooting

For chart-specific issues, use this discovery approach:

```bash
# Check current chart status
helm status simba-intelligence

# Find deployed pods for your release
kubectl get pods -n simba-intelligence

# Describe a specific pod (replace <pod-name> with actual name from above)
kubectl describe pod <pod-name> -n simba-intelligence

# Get logs from a pod (replace <pod-name> with actual name)
kubectl logs <pod-name> -n simba-intelligence

# See all resources created by the chart
kubectl get all -n simba-intelligence
```

> **🔍 Discovery Tip:** Use `kubectl get pods` first to see the actual pod names, then use those names in the `describe` and `logs` commands. Pod names typically follow patterns like `simba-intelligence-<component>-<random-id>`.

For comprehensive troubleshooting guidance, see the [official Helm troubleshooting guide](https://helm.sh/docs/topics/troubleshooting/).

---

## Conclusion

This approach ensures you always have accurate, up-to-date information about the Simba Intelligence Helm chart by exploring the source files directly. The chart maintainers update these files with each release, so you'll never work with outdated configuration examples or installation instructions.

**Remember:** When in doubt, the files in the chart directory (`Chart.yaml`, `values.yaml`, `README.md`) are the authoritative source of truth.