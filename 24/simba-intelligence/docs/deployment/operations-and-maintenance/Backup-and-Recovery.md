# Backup and Recovery

This guide covers what you need to back up for Simba Intelligence and basic recovery planning.

> **Production Note:** For production deployments, you should use an external database service (such as AWS RDS, Azure Database, or Google Cloud SQL) managed outside the Helm chart rather than the bundled PostgreSQL instance.

## What to Back Up

**🎯 Critical Data:**
- **Simba Intelligence Database**
- **Discovery/Composer Databases** - Multiple databases including zoomdata, zoomdata-upload, zoomdata-keyset, zoomdata-user-auditing, and zoomdata-qe
- **Persistent Storage**

---

## Database Backups

Multiple databases require regular backups including the main simbaintelligence database and the Discovery/Composer databases (zoomdata, zoomdata-upload, zoomdata-keyset, zoomdata-user-auditing, and zoomdata-qe).

**Recommended schedule:** Daily automated backups with 30-day retention

---

## Persistent Storage Backups

Back up any Persistent Volume Claims created by the Helm chart. To find out what those are, run a Helm dry run to see what PVCs will be created.

**Recommended schedule:** Weekly backups

---

## Recovery Process

When you need to recover:

1. Stop the application by scaling down the StatefulSets and Deployments (set replicas to 0)
2. Restore from your most recent backup
3. Test the restoration
4. Reapply the Helm chart

---

## Best Practices

**Schedule:**
- Daily database backups
- Weekly storage backups  
- Monthly recovery testing

**Security:**
- Store backups separately from your main system
- Encrypt sensitive database backups
- Limit access to backup files

**Testing:**
- Regularly test your restore process
- Verify backup integrity
- Practice recovery during maintenance windows

**Retention:**
- Keep 30 days of daily backups
- Keep 12 weeks of weekly backups
- Store critical backups offsite
