# System Requirements

This document outlines the infrastructure and software requirements for deploying Simba Intelligence in Kubernetes environments using Helm charts.

> **⚠️ Important Note:** Application resource requirements can vary widely depending on your deployment configuration. The Discovery/Composer instance, Redis, and PostgreSQL can all be external services. For production deployments, external PostgreSQL is recommended. These specifications serve as a general guide - expect to right-size resources based on your individual use case and external service configuration.

---

## Kubernetes Infrastructure Requirements

### Cluster Specifications

**Minimum Cluster Requirements:**
- **Kubernetes Version**: 1.24+
- **Node Count**: 2+ worker nodes for production deployments
- **Storage Support**: Persistent volume support with dynamic provisioning
- **Container Runtime**: Docker 20.10+ or containerd 1.6+

**Sizing Strategy:**
- **Initial Deployment**: Overprovision resources initially to establish baseline performance
- **Right-Sizing**: Analyze actual usage patterns and resource consumption after deployment
- **Scaling Approach**: Application usage varies dramatically based on use cases - monitoring and adjustment is essential
- **Performance Tuning**: Use observed metrics to optimize resource allocation over time

---

## Application Resource Requirements

### Core Components

**Simba Intelligence Application:**
- **Minimum**: 4 CPU cores, 8GB memory total
- **Recommended**: 8+ CPU cores, 16GB+ memory total
- **Scaling**: Supports horizontal scaling with multiple replicas
- **Components**: Includes web application, background workers (Celery), scheduler (Beat), and database migration jobs

### Optional Built-in Services

**Discovery/Composer Services:**
- **System Requirements**: See [Composer System Requirements](https://logi-composer-v25.insightsoftware.com/hc/en-us/articles/34933082999693-System-Requirements) for detailed specifications
- **Recommendation**: Use external services for production deployments

**PostgreSQL Database:**
- **Recommendation**: Use external managed database for production

**Redis Cache:**
- **Recommendation**: Use external Redis service for production

---

## Software Dependencies

### Required Components

**Kubernetes Platform:**
- **Kubernetes**: Version 1.24 or later
- **Helm**: Version 3.17+ for chart management
- **Container Runtime**: Docker 20.10+ or containerd 1.6+
- **Network Plugin**: CNI-compatible networking

**Storage Requirements:**
- **StorageClass**: Default storage class with dynamic provisioning
- **Persistent Volumes**: Support for ReadWriteOnce volumes; ReadWriteMany may be required for Composer built-in volume (driver deployment)
- **Backup**: Volume snapshot capability recommended

**Ingress and Networking:**
- **Ingress Controller**: Functional ingress controller for external access
- **DNS**: Functional cluster DNS

**Database**
- **Postgres**: Postgres is required, and an external managed database is recommended for production instead of the built-in service.

---

## Network Requirements

**External Access:**
- **Internet**: Outbound internet access for container image pulls
- **Load Balancer**: External access for users
- **SSL/TLS**: Certificate management for secure connections

**Internal Communication:**
- **Pod-to-Pod**: Standard Kubernetes cluster networking
- **Service Discovery**: Functional Kubernetes service discovery

**Security:**
- Implement appropriate network policies for production environments
