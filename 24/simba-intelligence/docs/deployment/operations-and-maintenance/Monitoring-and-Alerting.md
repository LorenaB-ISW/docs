# Monitoring and Alerting

This guide covers monitoring and alerting strategies for Simba Intelligence deployments in Kubernetes environments.

> **Audience Note:** This guide assumes familiarity with Kubernetes concepts, monitoring principles, and basic observability practices. For foundational Kubernetes knowledge, refer to the [official Kubernetes documentation](https://kubernetes.io/docs/home/).

> **Production Note:** For production deployments, implement comprehensive monitoring using your organization's existing Kubernetes monitoring stack and follow established monitoring best practices.

## Overview

Effective monitoring requires observing key metrics across your Simba Intelligence deployment to ensure system health, performance, and availability. This includes monitoring the application, underlying Kubernetes resources, and data flows.

---

## What to Monitor

### 🎯 Application Health
- **Service Availability** - Core application endpoints and API responsiveness
- **Database Connectivity** - Connection health to PostgreSQL databases
- **Application Performance** - Response times and throughput metrics
- **Business Metrics** - Custom metrics relevant to your use case

### 🎯 Kubernetes Resources
- **Pod Health** - Pod status, restarts, and resource utilization
- **Service Communication** - Inter-service connectivity and response times
- **Storage** - Persistent volume usage and performance
- **Ingress** - Traffic patterns and response times

### 🎯 Infrastructure Metrics
- **Resource Usage** - CPU, memory, disk, and network utilization
- **Scaling Metrics** - Horizontal Pod Autoscaler (HPA) behavior
- **Node Health** - Kubernetes node status and capacity

---

## Monitoring Strategy

### Application-Level Monitoring

**Health Checks:**
- The Helm chart automatically configures readiness and liveness probes for all Simba Intelligence containers:
  - **Simba Intelligence:** `/api/v1/healthz`
  - **Discovery/Composer:** `/discovery/actuator/health/liveness` and `/discovery/actuator/health/readiness`

**Application Metrics:**
- **Composer/Discovery Services:** Expose extensive custom metrics through actuator endpoints including JVM metrics, session counts, query performance, and WebSocket connections. See the [Composer System Metrics Guide](https://docs-composer.zendesk.com/hc/en-us/articles/25438714821261-Composer-System-Metrics) for detailed endpoint documentation.
- **Simba Intelligence:** Provides standard Kubernetes resource metrics (CPU, memory, disk) through the Kubernetes metrics API
- Track business metrics like query execution times and data processing rates

**Database Monitoring:**
- Monitor database connection pools and query performance
- Track database resource utilization and growth
- Set up alerts for connection failures or performance degradation

### Kubernetes-Level Monitoring

**Resource Monitoring:**
- Implement cluster-wide monitoring for nodes, pods, and services
- Track resource quotas and limits across namespaces
- Monitor persistent volume claims and storage usage

**Event Monitoring:**
- Capture Kubernetes events for deployment, scaling, and error conditions
- Monitor pod lifecycle events and container restarts
- Track ingress traffic patterns and SSL certificate status

---

## Alerting Strategy

### Critical Alerts
- **Service Downtime** - Application pods not responding or repeatedly restarting
- **Database Issues** - Connection failures or high query latency
- **Resource Exhaustion** - CPU, memory, or storage approaching limits
- **Security Events** - Authentication failures or suspicious access patterns

### Warning Alerts
- **Performance Degradation** - Response times above acceptable thresholds
- **Resource Pressure** - Resource usage trending toward limits
- **Configuration Changes** - Unexpected configuration modifications
- **Scaling Events** - Frequent autoscaling activities

### Alert Configuration Best Practices
- Set appropriate thresholds to avoid alert fatigue
- Implement escalation procedures for critical issues
- Use runbook automation where possible
- Group related alerts to reduce noise

---

## Implementation Approaches

### Using Existing Kubernetes Monitoring

**Cluster Monitoring:**

Most Kubernetes environments provide built-in monitoring capabilities. Leverage your existing cluster monitoring infrastructure:

- **Metrics Server** - For basic resource metrics and HPA functionality
- **Node Monitoring** - Built into most managed Kubernetes services
- **Event Collection** - Standard Kubernetes event monitoring
- **Log Aggregation** - Centralized logging from all containers

### Custom Monitoring Integration

**Application Metrics:**
Simba Intelligence services expose different levels of metrics depending on the component:

- **Discovery/Composer Services:** Comprehensive custom metrics via actuator endpoints (`/composer/actuator/prometheus`, `/actuator/metrics`) including JVM metrics, session statistics, query performance, and WebSocket connections
- **Simba Intelligence Core:** Standard Kubernetes resource metrics (CPU, memory, disk) and health endpoints
- **Database Integration:** Connection pool metrics and query performance data
- Integration with Prometheus, Grafana, or existing monitoring infrastructure

For detailed Composer metrics documentation, see: [Composer System Metrics Guide](https://docs-composer.zendesk.com/hc/en-us/articles/25438714821261-Composer-System-Metrics)

**Example Business Metrics:**
```
# Active user sessions
zoomdata_sessions_count{service_instance="simbaintelligence1-discovery-web-0"} 1.0
zoomdata_sessions_count_last_minute{service_instance="simbaintelligence1-discovery-web-0"} 1.0

# Concurrent users across all instances  
zoomdata_web_concurrent_user{service_instance="simbaintelligence1-discovery-web-0"} 0.0

# License utilization
zoomdata_web_used_concurrent_session_licenses{service_instance="simbaintelligence1-discovery-web-0"} 0.0

# Database connection health
hikaricp_connections_active{pool="HikariPool-1"} 0.0
hikaricp_connections_idle{pool="HikariPool-1"} 2.0
hikaricp_connections_max{pool="HikariPool-1"} 20.0

# HTTP request performance by endpoint
http_server_requests_seconds_count{method="GET",uri="/actuator/health",status="200"} 203.0
http_server_requests_seconds_sum{method="GET",uri="/actuator/health",status="200"} 4.47136307
```

**Monitoring Context:**
These metrics provide insight into application health and usage patterns. Establish baseline values during normal operation to identify unusual patterns that may require attention. Your monitoring team can configure appropriate thresholds based on your specific deployment requirements and business needs.

**Log Integration:**
Simba Intelligence does not integrate automatically with specific logging or monitoring services, but provides flexibility to work with any logging solution. All containers write logs to standard output (stdout) and standard error (stderr), making them compatible with most logging frameworks.

**Popular logging integration options:**
- **ELK Stack** (Elasticsearch, Logstash, Kibana) - Comprehensive log analysis and visualization
- **Fluentd/Fluent Bit** - Lightweight log forwarding and processing
- **Splunk** - Enterprise log management and analytics
- **Cloud Provider Solutions** - AWS CloudWatch Logs, Azure Monitor Logs, Google Cloud Logging
- **Grafana Loki** - Log aggregation with Prometheus-style queries

---

## Best Practices

For comprehensive monitoring and alerting best practices, refer to these authoritative resources:

**Kubernetes Monitoring:**
- [Kubernetes Monitoring Best Practices](https://kubernetes.io/docs/concepts/cluster-administration/monitoring/) - Official Kubernetes monitoring guidance
- [CNCF Observability Landscape](https://landscape.cncf.io/guide#observability-and-analysis--observability) - Industry-standard monitoring tools and patterns

**Cloud Provider Resources:**
- **AWS:** [EKS Best Practices for Monitoring](https://aws.github.io/aws-eks-best-practices/reliability/docs/dataplane/#monitoring-and-logging)
- **Azure:** [AKS Monitoring Best Practices](https://docs.microsoft.com/en-us/azure/aks/monitor-aks)
- **GCP:** [GKE Observability Best Practices](https://cloud.google.com/kubernetes-engine/docs/concepts/observability)

**Industry Standards:**
- [Site Reliability Engineering (SRE) Practices](https://sre.google/books/) - Google's approach to monitoring and alerting
- [Prometheus Best Practices](https://prometheus.io/docs/practices/naming/) - If using Prometheus-based monitoring

---

## Common Monitoring Patterns

### Health Check Implementation
The Helm chart automatically configures Kubernetes probes for application health monitoring using these endpoints:

**Simba Intelligence Health Endpoints:**
- **Main Application:** `/api/v1/healthz` - Configured for readiness and liveness probes
- **Discovery/Composer:** 
  - `/discovery/actuator/health/liveness` - Configured for liveness probes
  - `/discovery/actuator/health/readiness` - Configured for readiness probes

### Resource Monitoring
Implement comprehensive resource monitoring:
- **Request/Limit Monitoring** - Track resource requests vs actual usage
- **Quality of Service** - Monitor QoS classes and their impact
- **Cluster Capacity** - Track overall cluster resource utilization

### Service Monitoring
Monitor service-to-service communication:
- **Service Connectivity** - Monitor communication between application components
- **DNS Resolution** - Monitor internal DNS performance
- **Network Performance** - Track network latency and errors

---

## Getting Started

### Quick Implementation Steps

1. **Assess Current Monitoring** - Review existing Kubernetes monitoring capabilities
2. **Define Key Metrics** - Identify critical metrics for your deployment
3. **Configure Basic Alerts** - Set up essential health and performance alerts
4. **Implement Gradual Enhancement** - Add monitoring complexity over time

### Integration with Existing Tools

Rather than implementing specific monitoring solutions, integrate Simba Intelligence monitoring with your existing organizational standards:

- Leverage existing monitoring infrastructure and expertise
- Follow established alert management and escalation procedures  
- Use familiar dashboards and reporting tools
- Maintain consistency with other application monitoring approaches

---

This strategic approach ensures your monitoring implementation aligns with organizational standards while providing comprehensive visibility into your Simba Intelligence deployment.
