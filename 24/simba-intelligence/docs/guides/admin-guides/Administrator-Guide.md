# Administrator Guide

This guide provides system administrators with comprehensive information for configuring, managing, and maintaining Simba Intelligence in production environments. It covers user management, system configuration, security settings, and operational best practices.

## Overview

### Administrator Responsibilities

As a Simba Intelligence administrator, you are responsible for:

**🔐 User Access Management**
- Creating and managing user accounts
- Assigning appropriate roles and permissions
- Monitoring user activity and access patterns
- Implementing security policies and controls

**⚙️ System Configuration**
- Configuring AI/LLM provider integrations
- Managing data connectors
- Setting up system-wide policies and rules
- Monitoring system performance and health

**🛡️ Security and Compliance**
- Implementing authentication and authorization policies
- Managing API keys and credentials
- Ensuring data privacy and access controls
- Maintaining audit logs and compliance reporting

**📊 Operational Monitoring**
- Monitoring system performance and availability
- Managing resource allocation and scaling
- Troubleshooting system issues
- Planning capacity and upgrades

---

## User Management

### User Roles and Permissions

Simba Intelligence implements a hierarchical role-based access control (RBAC) system with specific roles that grant different levels of system access:

#### Permission Hierarchy

```
supervisor (Full Admin)
    ├── All system administration capabilities
    ├── User management and role assignment
    ├── LLM configuration and system settings
    └── Access to all data sources and features
    
ROLE_CREATE_SOURCES (Data Creator)
    ├── Data Agent access for creating data sources
    ├── Data Sources management and configuration
    ├── Playground access for querying
    └── Basic user capabilities
    
ROLE_MANAGE_CONNECTIONS (Connection Manager) 
    ├── Data Connections configuration and management
    ├── Database connection setup and maintenance
    ├── Playground access for querying
    └── Basic user capabilities
    
Basic User (Query Only)
    └── Playground interface for data querying
```

#### Detailed Role Capabilities

**Supervisor Role:**
- **Full administrative access** to all system functions
- **User management**: Create, modify, and delete user accounts
- **LLM configuration**: Set up and manage AI provider integrations
- **System settings**: Configure global system policies and rules
- **License management**: Monitor and manage system licensing
- **Access to admin overlay**: Advanced system configuration interfaces

**ROLE_CREATE_SOURCES:**
- **Data Agent access**: Use AI-powered data source creation
- **Data Sources management**: Create, configure, and manage data sources
- **Playground access**: Natural language querying capabilities
- **API key management**: Manage personal API keys for integrations

**ROLE_MANAGE_CONNECTIONS:**
- **Data Connections**: Configure and manage database connections
- **Connection testing**: Validate and troubleshoot database connections
- **Playground access**: Query data through connected sources
- **Infrastructure integration**: Set up external data source integrations

**Basic User:**
- **Playground interface**: Natural language querying of available data sources
- **Query results**: View, analyze, and export query results
- **Personal settings**: Manage personal preferences and API keys

### User Account Management

#### Creating User Accounts

**Access user management:**
1. **Log in as supervisor** or user with administrative privileges
2. **Navigate to admin functions**:
   - Click your **user menu** (top-right corner)
   - Select **"Users & Groups"**
   - This opens the Admin Overlay with user management interface

**Add new users:**
1. **Click "Add User"** or equivalent option in the user management interface
2. **Enter user details**:
   ```
   Username: user@company.com
   Full Name: John Smith
   Email: john.smith@company.com
   Initial Password: [Generate secure password]
   ```
3. **Assign roles**: Select appropriate roles based on user responsibilities
4. **Set permissions**: Configure specific data source access if needed
5. **Save and notify**: Create account and send credentials to user

#### Modifying User Permissions

**Role assignment process:**
1. **Access user management** through Admin Overlay
2. **Select user account** to modify
3. **Update role assignments**:
   - Add roles by selecting from available options
   - Remove roles by deselecting or using removal options
   - Multiple roles can be assigned to a single user
4. **Apply changes** and verify new permissions take effect

**Permission verification:**
- **Test access**: Have user log in and verify available features
- **Check navigation**: Ensure menu items appear correctly for assigned roles
- **Validate functionality**: Confirm user can perform expected actions

#### User Account Maintenance

**Regular maintenance tasks:**
- **Password policies**: Enforce regular password changes
- **Access reviews**: Periodically review user permissions and remove unnecessary access
- **Inactive account cleanup**: Disable or remove accounts for users who no longer need access
- **Role optimization**: Ensure users have minimum necessary permissions for their work

### Authentication Configuration

#### Local Authentication Setup

**Default authentication method:**
- Username/password authentication managed within Simba Intelligence
- User credentials stored securely with encrypted passwords
- Set password expiration policies

## LLM Provider Configuration

### AI Provider Management

The LLM Configuration interface allows administrators to set up and manage AI providers that power Simba Intelligence's natural language capabilities.

#### Accessing LLM Configuration

1. **Navigate to configuration**: http://your-domain/llm-configuration
2. **Requires supervisor role** for access to configuration interface
3. **Main interface**: Tabbed layout showing available providers and their configurations

#### Google Vertex AI Configuration

**Prerequisites:**
- Google Cloud Platform account with Vertex AI API enabled
- Service account with appropriate permissions
- Project with billing configured

**Configuration process:**
1. **Select "Vertex AI" tab** in LLM configuration interface
2. **Enter service account JSON**:
   ```json
   {
     "type": "service_account",
     "project_id": "your-gcp-project-id",
     "private_key_id": "your-key-id",
     "private_key": "-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----\n",
     "client_email": "service-account@project.iam.gserviceaccount.com",
     "client_id": "your-client-id",
     "auth_uri": "https://accounts.google.com/o/oauth2/auth",
     "token_uri": "https://oauth2.googleapis.com/token"
   }
   ```
3. **Configure capabilities**:
   - **Chat capability**: Enable with model `gemini-2.0-flash` and location `us-central1`
   - **Embeddings capability**: Enable with model `text-embedding-004`
   - **Vision capability**: Enable for image analysis features
4. **Test configuration** and save when successful

**Security considerations:**
- Store service account JSON securely
- Use dedicated service accounts for Simba Intelligence
- Apply principle of least privilege for API permissions
- Monitor usage and costs in Google Cloud Console

#### OpenAI/Azure OpenAI Configuration

**OpenAI setup:**
1. **Select "OpenAI" tab** in configuration interface
2. **Enter API credentials**:
   ```
   API Key: sk-your-openai-api-key
   Organization ID: org-your-organization-id (optional)
   ```
3. **Configure chat capability**:
   ```
   Model: gpt-4o or gpt-3.5-turbo
   Temperature: 0.7 (adjustable)
   Max Tokens: 4000 (adjustable)
   ```

**Azure OpenAI setup:**
1. **Select "Azure OpenAI" tab**
2. **Enter Azure-specific details**:
   ```
   API Key: your-azure-openai-key
   Azure Endpoint: https://your-instance.openai.azure.com/
   API Version: 2023-05-15
   ```
3. **Configure deployments**:
   ```
   Chat Deployment: your-gpt4-deployment-name
   Embeddings Deployment: your-embedding-deployment-name
   ```

#### AWS Bedrock Configuration

**Prerequisites:**
- AWS account with Bedrock access enabled
- IAM credentials with Bedrock permissions
- Model access granted for required models (Claude, etc.)

**Configuration steps:**
1. **Select "AWS Bedrock" tab**
2. **Enter IAM credentials**:
   ```
   Access Key ID: AKIA...
   Secret Access Key: your-secret-key
   Region: us-east-1
   Session Token: optional-session-token
   ```
3. **Configure model settings**:
   ```
   Chat Model: anthropic.claude-3-sonnet-20240229-v1:0
   Temperature: 0.7
   Max Tokens: 4000
   ```

### Provider Management Best Practices

**Security management:**
- **Rotate API keys** regularly according to security policies
- **Monitor usage and costs** for all configured providers
- **Set up alerts** for unusual usage patterns or cost spikes
- **Use separate credentials** for development and production environments

**Performance optimization:**
- **Configure multiple providers** for redundancy and load distribution
- **Monitor response times** and adjust provider priorities as needed
- **Set appropriate timeout values** to balance performance and reliability
- **Cache frequently used responses** to reduce API calls and costs

**Cost management:**
- **Monitor token usage** across all providers to understand costs
- **Set up usage alerts** to prevent unexpected charges
- **Optimize model selection** based on use case requirements
- **Implement usage quotas** for different user groups if needed

---

## System Configuration

### API and Integration Management

#### API Key Management

**System API keys:**
- **Administrative API keys**: Keys with full system access for automation
- **Service account keys**: Keys for system-to-system integration
- **User API keys**: Keys for individual user programmatic access
- **Integration keys**: Keys for specific third-party integrations

**API key lifecycle:**
1. **Creation**: Generate keys
2. **Distribution**: Securely provide keys to authorized users or systems
3. **Rotation**: Regularly rotate keys according to security policies
4. **Revocation**: Immediately revoke compromised or unused keys

#### External Integration Configuration

**Third-party integrations:**
- **Business intelligence tools**: Configure connections to Tableau, Power BI, etc.
- **Data catalogs**: Integrate with enterprise data catalog solutions
- **Monitoring systems**: Connect to APM and infrastructure monitoring tools
- **Identity providers**: Configure SAML, OIDC, or LDAP integrations

---

## Troubleshooting and Support

### Common Administrative Issues

#### User Access Issues

**"User cannot access expected features"**

**Diagnosis steps:**
1. **Check user roles**: Verify correct roles are assigned in user management
2. **Permission inheritance**: Ensure roles provide expected permissions
3. **Session refresh**: Have user log out and back in to refresh permissions
4. **Cache clearing**: Clear application cache if permission changes don't take effect

**Resolution process:**
1. **Verify role assignment** in admin interface
2. **Check navigation menu** appearance for user
3. **Test specific functionality** user should be able to access
4. **Monitor audit logs** for permission-related errors

#### LLM Configuration Problems

**"AI provider not responding or returning errors"**

**Troubleshooting checklist:**
- **Credential validation**: Verify API keys or service account credentials are correct
- **Quota and billing**: Check AI provider account for quota limits or billing issues
- **Network connectivity**: Ensure Simba Intelligence can reach AI provider endpoints
- **Rate limiting**: Check if requests are being rate-limited by provider

**Resolution approaches:**
1. **Test credentials** directly with provider's tools or APIs
2. **Monitor usage** in provider's dashboard or console
3. **Check error logs** for specific error messages
4. **Implement retry logic** and backoff strategies for temporary issues

#### Performance Issues

**"System responding slowly or timing out"**

**Performance diagnosis:**
1. **Check system resources**: Monitor CPU, memory, and disk usage
2. **Database performance**: Analyze slow query logs and connection counts
3. **Network latency**: Test connectivity to AI providers and databases
4. **Cache effectiveness**: Monitor cache hit rates and memory usage

**Optimization strategies:**
- **Scale resources**: Increase CPU, memory, or replica count as needed
- **Optimize queries**: Work with database teams to improve query performance
- **Implement caching**: Enable semantic caching for frequently used queries
- **Load balancing**: Distribute load across multiple instances

### System Diagnostics

#### Diagnostic Information Collection

**For troubleshooting, collect:**
```bash
# System status
kubectl get pods -n simba-intelligence
kubectl describe pod <pod-name> -n simba-intelligence

# Application logs
kubectl logs deployment/simba-intelligence-website -n simba-intelligence

# Resource usage
kubectl top pods -n simba-intelligence
kubectl top nodes

# Database status
psql -h localhost -U simba_user -c "SELECT version();"
psql -h localhost -U simba_user -c "SELECT count(*) FROM pg_stat_activity;"

# Cache status
redis-cli info memory
redis-cli info clients
```

#### Support Escalation Procedures

**When to escalate issues:**
- **System-wide outages** affecting multiple users
- **Security incidents** or suspected breaches
- **Data corruption** or loss scenarios
- **Performance degradation** that cannot be resolved through standard troubleshooting

**Information to provide when escalating:**
- **System configuration**: Environment details and deployment information
- **Error messages**: Complete error logs and stack traces
- **Reproduction steps**: Clear steps to reproduce the issue
- **Impact assessment**: Number of affected users and business impact
- **Mitigation attempts**: What troubleshooting steps have already been tried

---

## Best Practices

### Security Best Practices

**Access management:**
- **Principle of least privilege**: Grant minimum necessary permissions
- **Regular access reviews**: Quarterly review of user permissions and roles
- **Strong authentication**: Enforce strong password policies and consider MFA
- **Audit trail maintenance**: Maintain comprehensive logs for security compliance

**System security:**
- **Regular updates**: Keep system components and dependencies updated
- **Network security**: Use firewalls, VPNs, and encrypted connections
- **Credential management**: Rotate credentials regularly and store securely
- **Incident response**: Maintain procedures for security incident response

### Operational Excellence

**Proactive monitoring:**
- **Automated alerts**: Set up alerts for system health and performance issues
- **Capacity planning**: Monitor trends and plan for future resource needs
- **Performance baselines**: Establish performance baselines and monitor deviations
- **User feedback**: Regularly collect and analyze user feedback for improvements

**Change management:**
- **Testing procedures**: Test all changes in non-production environments first
- **Rollback plans**: Maintain ability to quickly rollback problematic changes
- **Documentation**: Keep system documentation current and accessible
- **Communication**: Notify users of planned maintenance and system changes

### User Experience Optimization

**Performance optimization:**
- **Query optimization**: Work with users to optimize frequently-used queries
- **Caching strategies**: Implement intelligent caching for common use cases
- **Resource allocation**: Ensure adequate resources for peak usage periods
- **User training**: Provide training on effective use of the system

**Feature enablement:**
- **Progressive rollout**: Gradually enable new features and gather feedback
- **Usage monitoring**: Track feature adoption and user satisfaction
- **Support documentation**: Maintain current user guides and help documentation
- **Feedback loops**: Establish channels for user feedback and feature requests

---

*This administrator guide provides the foundation for effective Simba Intelligence administration. For specific technical procedures, refer to the [Configuration Reference](configuration-reference.md) and [Operations Guide](operations-guide.md). Remember: successful administration balances security, performance, and user experience to maximize the value of AI-powered data analysis.*
