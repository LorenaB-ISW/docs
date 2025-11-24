# Data Connection Management

Data Connection Management is your gateway to connecting Simba Intelligence with your existing databases and data warehouses. Through this interface, you can establish secure connections to SQL Server, PostgreSQL, Snowflake, and other supported data sources, making them available for AI-powered querying and analysis.

## Overview

### What are Data Connections?

Think of Data Connections as specialized translators that speak both Simba Intelligence's language and your database's language. Each connector is designed to understand the specific connection requirements, query syntax, and data types of different database systems, enabling seamless integration without requiring data migration.

### How Data Connections Work

```
Your Applications & Users
           ↓
   Simba Intelligence
           ↓
   Query Engine Service
           ↓
   Data Connectors
           ↓
  Your Databases & Data Warehouses
```

**The connection flow:**
1. **You configure** a data connection with your database credentials
2. **Simba Intelligence registers** the connection with the Query Engine service  
3. **External Data Connections** handle database-specific communication
4. **Users query** data through natural language and translated to the Query Engine
5. **Queries are translated** and executed against your actual databases by a governed, secure query engine

### Supported Database Connectors

The platform currently supports the following database connectors with many more on the way:

- Postgres
- MS SQL
- Snowflake
- Oracle
- Google BigQuery

---

## Prerequisites

### Required Permissions

**To manage data connectors, you need:**
- **ROLE_MANAGE_CONNECTIONS** permission or higher
- **Administrative access** to Simba Intelligence
- **Network connectivity** from Simba Intelligence to target databases

### Database Requirements

**For each database you want to connect:**
- **Database server** accessible from Simba Intelligence network
- **User account** with appropriate read permissions
- **Network firewall rules** allowing connections on database ports
- **Connection credentials** (username/password or key-based authentication)

### Infrastructure Requirements

**Network connectivity:**
- Simba Intelligence can reach your database servers
- Appropriate ports open (1433 for SQL Server, 5432 for PostgreSQL, 443 for Snowflake)
- DNS resolution working for database hostnames
- No blocking proxies or firewalls between services

---

## Accessing Data Connection Management

### Navigation
1. **Log into Simba Intelligence** with appropriate permissions
2. **Click "Data Connections"** in the main navigation menu
3. **Or visit directly**: `http://your-domain/data-connectors`

### Interface Overview

The Data Connection Management interface provides:
- **Connection browser** showing all configured data sources
- **Connection status** indicators (active, inactive, error)
- **Add connection** functionality for new data sources
- **Test and validate** capabilities for existing connections
- **Management tools** for editing and removing connections

> **💡 Interface Note**: The Data Connection Management uses an embedded Query Engine interface, providing enterprise-grade data connection capabilities within the Simba Intelligence experience.

---

## Security Best Practices

### Network Security

**Connection security measures:**
- **Use encrypted connections** (SSL/TLS) whenever possible
- **Restrict network access** using firewalls and security groups
- **Monitor connection attempts** and log access patterns
- **Use private networks** for database connections when available

### Credential Management

**Secure credential handling:**
- **Use dedicated service accounts** for Simba Intelligence connections
- **Implement regular password rotation** following security policies
- **Apply principle of least privilege** - minimum necessary permissions
- **Store credentials securely** - never in plain text or configuration files

### Access Control

**Database permission best practices:**
- **Create read-only accounts** specifically for analytics
- **Grant schema-level permissions** rather than database-wide access
- **Use views and row-level security** to restrict data access where appropriate
- **Regular permission audits** to ensure access remains appropriate

---

## Troubleshooting Common Issues

### Network Connectivity Problems

**"Cannot reach database server"**

**Possible causes and solutions:**
- **Network connectivity**: Test using `telnet hostname port` or equivalent
- **Firewall rules**: Ensure ports are open between Simba Intelligence and database
- **DNS resolution**: Verify hostname resolves correctly
- **VPN/proxy issues**: Check if connections go through network security appliances

**Resolution steps:**
1. Verify basic network connectivity from Simba Intelligence server
2. Check firewall logs for blocked connections
3. Test connectivity during different times to identify intermittent issues
4. Contact network administrators if infrastructure changes are needed

### Authentication Failures

**"Login failed for user"**

**Common authentication issues:**
- **Incorrect credentials**: Verify username and password are correct
- **Account locked**: Check if database account is locked due to failed attempts
- **Password expired**: Ensure database password hasn't expired
- **Permission issues**: Verify account has login permissions

**Resolution approaches:**
1. Test credentials directly against database using database client tools
2. Check database logs for specific authentication error messages
3. Verify account status in database user management interface
4. Reset credentials if necessary and update connection configuration

### Permission and Access Issues

**"Access denied to database/table"**

**Permission troubleshooting:**
- **Database access**: Ensure user has `CONNECT` or equivalent permission to database
- **Schema access**: Verify user can access required schemas  
- **Table permissions**: Confirm `SELECT` permissions on target tables
- **Row-level security**: Check if row-level security policies are blocking access

**Resolution process:**
1. Review database permissions for the connector user account
2. Test access using database client with same credentials
3. Work with database administrators to grant appropriate permissions
4. Document required permissions for future reference

### Performance Issues

**"Connection timeout" or "Query timeout"**

**Performance optimization:**
- **Connection timeouts**: Increase connection timeout values if network is slow
- **Query timeouts**: Adjust query timeout for long-running operations
- **Connection pooling**: Ensure connection pooling is properly configured
- **Database performance**: Monitor database server performance during queries

**Optimization strategies:**
1. Monitor database performance during Simba Intelligence queries
2. Identify slow queries and optimize them or the underlying data structure
3. Consider using read replicas for analytics workloads
4. Adjust connection pool settings based on usage patterns

---

*Ready to connect your first database? Start with the Data Connections interface and follow the step-by-step process for your specific database type. Remember: secure, well-configured connections are the foundation for reliable AI-powered data analysis.*