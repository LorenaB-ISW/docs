# Upgrade Procedures

This guide covers strategic upgrade procedures for Simba Intelligence deployments using Helm charts in Kubernetes environments.

> **Audience Note:** This guide assumes familiarity with Kubernetes, Helm, and basic upgrade planning principles. For foundational knowledge, refer to the [official Helm documentation](https://helm.sh/docs/).

> **Production Note:** Always test upgrades in non-production environments before applying to production systems. Follow your organization's change management and approval processes.

## Overview

Successful upgrades require careful planning, appropriate backup strategies, and understanding the scope of changes in each version type. This guide focuses on Helm-based upgrade strategies that integrate with your existing operational practices.

---

## Pre-Upgrade Planning

### Backup Strategy

Before any upgrade, ensure comprehensive backup coverage:

**Critical Backup Components:**
- **Database Backups** - Complete PostgreSQL database snapshots
- **Persistent Volume Snapshots** - All persistent data and configuration
- **Helm Configuration** - Current values and deployment state
- **Kubernetes Resources** - ConfigMaps, Secrets, and custom resources

**Backup Verification:**
Verify that your backup systems are functioning correctly and restoration procedures are tested. Your backup strategy should align with your organization's data protection and recovery requirements.

### Version Assessment

**Version Type Classification:**
- **Major.Minor Versions** (e.g., 25.2 → 25.3 or 26.0) - May include significant changes
- **Revision Updates** (e.g., 25.2.1 → 25.2.2) - Primarily bug fixes and patches

**Pre-Upgrade Review:**
- Review release notes, and helm readme for breaking changes
- Assess impact on existing integrations
- Verify compatibility with current infrastructure
- Plan for any required configuration updates

---

## Upgrade Strategies

### Major/Minor Version Upgrades

For major or minor version changes, use a conservative approach to minimize risk:

**Recommended Approach:**
1. **Scale Down Deployments and StatefulSets** - Temporarily scale deployments to zero replicas
2. **Update Application** - Deploy new version via Helm upgrade
3. **Verify Functionality** - Confirm system health and schema updates completed successfully

**Strategic Considerations:**
- **Planned Maintenance Window** - Schedule during low-usage periods
- **Extended Downtime Planning** - Communicate expected downtime to users
- **Rollback Readiness** - Ensure quick rollback capability if issues arise

### Revision Upgrades

For revision updates (patch releases), normal rolling updates are typically appropriate:

**Recommended Approach:**
1. **Rolling Update** - Use Helm's default rolling update strategy
2. **Monitor Deployment** - Watch for any deployment issues
3. **Verify Health** - Confirm system continues operating normally

**Strategic Considerations:**
- **Configuration-Dependent Risk** - Risk level depends on StatefulSet configuration and replica counts
- **Continuous Monitoring** - Watch application metrics during update
- **Quick Detection** - Be prepared to identify any unexpected issues

---

## Upgrade Execution

### Helm Upgrade Process

**Basic Upgrade Command Structure:**
```bash
helm upgrade [RELEASE_NAME] oci://docker.io/insightsoftware/simba-intelligence-chart --namespace [NAMESPACE] --version [VERSION]
```

**Key Upgrade Parameters:**
- **Version Specification** - Always specify exact chart version
- **Values Preservation** - Use `--reuse-values` or provide updated values
- **Timeout Configuration** - Set appropriate timeout for your environment
- **Rollback on Failure** - Configure automatic rollback behavior

### Upgrade Monitoring

**During Upgrade:**
- Monitor pod rollout status
- Watch for any error conditions
- Track resource utilization
- Verify service connectivity

**Health Verification:**
- Application health endpoints respond correctly
- Database connectivity is maintained
- All expected services are running
- Integration points continue functioning

---

## Post-Upgrade Verification

**Essential Verification Steps:**
- Confirm all services are running and healthy
- Verify core application functionality works as expected
- Validate performance meets baseline requirements
- Test critical integration points
- Document upgrade completion and any issues encountered

**Communication:**
Follow your organization's established procedures for communicating upgrade completion and system status updates.

---

## Rollback Procedures

### When to Rollback

Consider rollback when:
- Critical functionality is impaired
- Performance degradation exceeds acceptable thresholds
- Integration failures prevent normal operation
- Data integrity concerns arise

### Rollback Execution

**Rollback Procedure:**
1. **Scale Down Services** - Temporarily scale deployments and StatefulSets to zero replicas
2. **Restore Backups** - Restore database and persistent data from pre-upgrade backups
3. **Helm Rollback** - Rollback to previous chart release
4. **Verify Functionality** - Confirm system health after rollback

**Helm Rollback:**
```bash
helm rollback [RELEASE_NAME] [REVISION] --namespace [NAMESPACE]
```

---

## Best Practices

For comprehensive upgrade best practices, refer to these authoritative resources:

**Simba Intelligence Specific:**
- [Helm Deployment Guide](../installation-and-setup/Helm-Chart-Reference.md) - Product-specific deployment and configuration guidance
- **Helm Chart README** - Detailed chart documentation and upgrade notes (included with chart package)

**Helm Upgrade Best Practices:**
- [Helm Best Practices - Managing Releases](https://helm.sh/docs/chart_best_practices/) - Official Helm upgrade and release management guidance
- [Helm Upgrade Documentation](https://helm.sh/docs/helm/helm_upgrade/) - Complete reference for Helm upgrade command and options

**Kubernetes Deployment Best Practices:**
- [Kubernetes Deployment Strategies](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#strategy) - Official guidance on rolling updates and deployment strategies
- [Managing Kubernetes Upgrades](https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/) - Best practices for application upgrades in Kubernetes

**Support Resources:**
For assistance with upgrade planning or troubleshooting, contact InsightSoftware Support through your standard support channels for expert guidance and best practices tailored to your specific deployment.

---

This strategic approach ensures your Simba Intelligence upgrades align with organizational standards while maintaining system stability and minimizing business disruption.
