# Authentication and Security

This guide explains Simba Intelligence's security model, authentication systems, and access control mechanisms. It covers user management, role-based permissions, API security, and security best practices for administrators and users.

## Overview

### Security Architecture

Simba Intelligence implements a multi-layered security approach designed to protect data, control access, and ensure compliance with enterprise security requirements.

```
User Authentication Layer
         ↓
Role-Based Access Control (RBAC)
         ↓
Application Security Layer
         ↓
Data Source Security
         ↓
Network & Infrastructure Security
```

**Security principles:**
- **Zero Trust**: Every request is authenticated and authorized
- **Least Privilege**: Users receive minimum necessary permissions
- **Defense in Depth**: Multiple security layers protect against threats
- **Audit Trail**: Complete logging of security-relevant activities

### Security Components

**🔐 Authentication Systems**
- Token-based authentication with automatic renewal
- Session management with configurable timeouts

**👤 Authorization and Access Control**
- Hierarchical role-based permission system
- Granular access control for data sources and features
- API key management for programmatic access

**🛡️ Data Protection**
- Encrypted credential storage and transmission
- Data source access controls and query filtering
- Audit logging of all data access activities

**🌐 Network Security**
- HTTPS/TLS encryption for all communications
- Secure integration with external systems
- Network isolation and firewall protection

---

## User Authentication

### Authentication Flow

Simba Intelligence uses a token-based authentication system with automatic renewal to provide secure, seamless user access.

#### Login Process

1. **User Login**: User provides credentials (username/password)
2. **Credential Validation**: System validates against configured authentication provider
3. **Token Generation**: System generates secure access token with expiration
4. **Session Establishment**: User session created with appropriate permissions
5. **Token Renewal**: Automatic token refresh before expiration

#### Token Management

**Token lifecycle:**
```
Login → Token Issued (Valid 1 hour) → Auto-Renewal (if active) → Session Timeout/Logout
```

**Token characteristics:**
- **Expiration**: Default 1-hour lifetime with automatic renewal
- **Scope**: Contains user identity and role information
- **Storage**: Securely cached in browser with automatic cleanup

### Authentication Methods

#### Local Authentication

**Default authentication using Simba Intelligence user accounts:**
- Username and password stored securely in the system
- Password complexity requirements configurable
- Account lockout protection after failed attempts
- Password expiration and rotation policies

**User account creation:**
1. **Administrator creates account** through user management interface
2. **Initial credentials** provided securely to user
3. **First login** may require password change
4. **User profile** populated with role assignments and permissions

---

## Role-Based Access Control (RBAC)

### Permission Hierarchy

Simba Intelligence implements a hierarchical permission system where higher-level roles include all capabilities of lower-level roles.

#### Role Definitions

**Supervisor (Full Administrator)**
```
Capabilities:
├── All system administration functions
├── User management and role assignment
├── LLM configuration and system settings
├── Access to admin overlay and advanced features
├── Full data source and connector management
└── All query and analysis capabilities
```

**ROLE_CREATE_SOURCES (Data Creator)**
```
Capabilities:
├── Data Agent access for AI-powered data source creation
├── Data Sources management and configuration
├── Playground access for natural language querying
├── Personal API key management
└── Basic user query capabilities
```

**ROLE_MANAGE_CONNECTIONS (Connection Manager)**
```
Capabilities:
├── Data Connections configuration and management
├── Database connection setup and testing
├── Playground access for querying connected data
├── Infrastructure integration capabilities
└── Basic user query capabilities
```

**Basic User (Query Access)**
```
Capabilities:
├── Playground interface for natural language queries
├── Query results viewing, analysis, and export
├── Personal settings and preferences
└── Personal API key management
```

### Permission Enforcement

#### UI-Level Access Control

**Navigation menu adaptation:**
- Menu items appear only for users with appropriate permissions
- Disabled or hidden interface elements for restricted functions
- Role-based redirection to appropriate landing pages

**Example navigation for different roles:**
```javascript
// Supervisor sees all menu items
[Home, Data Agent, Data Sources, Data Connections, Playground, Admin]

// ROLE_CREATE_SOURCES sees
[Home, Data Agent, Data Sources, Playground]

// Basic User sees  
[Home, Playground]
```

#### API-Level Authorization

**Every API request validated:**
- **Token validation**: Verify user authentication token
- **Permission check**: Ensure user has required role for requested operation
- **Resource access**: Verify user can access specific data sources or resources
- **Action authorization**: Confirm user can perform the requested action

**Authorization flow:**
```
API Request → Token Validation → Role Check → Resource Access Check → Action Execution
```

### Dynamic Permission Evaluation

#### Context-Aware Permissions

**Permissions evaluated based on:**
- **User roles**: Assigned roles and their associated capabilities
- **Data source access**: Specific permissions for individual data sources  
- **Resource ownership**: Access to user-created resources
- **Time-based access**: Configurable time-based access restrictions

#### Permission Caching and Performance

**Efficient permission checking:**
- **Permission cache**: User permissions cached in session for performance
- **Cache invalidation**: Automatic cache refresh when roles change
- **Minimal latency**: Sub-millisecond permission checks for common operations

---

## API Security

### API Key Management

#### Personal API Keys

**Users can create and manage personal API keys:**
- **Self-service creation**: Users generate their own API keys through the interface
- **Permission inheritance**: API keys inherit the user's role-based permissions
- **Key rotation**: Users can regenerate keys as needed for security

**API key management interface:**
- **Key generation**: Create new API keys with custom names/descriptions
- **Key visibility**: Toggle between masked and visible key display
- **Key deletion**: Remove compromised or unused keys
- **Usage monitoring**: Track API key usage patterns

#### API Key Security Features

**Security measures:**
- **Key masking**: Keys displayed as `••••••••` by default with reveal option
- **Audit logging**: All API key operations logged for security monitoring

### API Authentication

#### Authentication Methods

**Bearer token authentication:**
```bash
# API request with authentication
curl -H "Authorization: Bearer your-api-key" \
     https://your-domain/api/v1/query
```

**Authentication flow:**
1. **Client includes API key** in Authorization header
2. **Server validates key** against stored keys
3. **User permissions retrieved** based on key owner
4. **Request processed** with user's permission context

### API Security Best Practices

#### For API Users

**Secure API key handling:**
- **Never commit keys to version control** or share in plain text
- **Use environment variables** to store keys in applications
- **Rotate keys regularly** according to security policies
- **Monitor key usage** for unexpected activity

**Request security:**
- **Always use HTTPS** for API requests
- **Validate responses** and handle errors appropriately
- **Implement retry logic** with exponential backoff
- **Log API usage** for debugging and monitoring

#### For Administrators

**API security monitoring:**
- **Monitor API usage patterns** across all users
- **Set up alerts** for unusual activity or potential abuse
- **Regular security audits** of API key usage and permissions
- **Enforce corporate policies** for API key management

---

## Data Security

### Data Source Access Control

#### Permission-Based Data Access

**Data source security layers:**
1. **Connection-level**: User must have appropriate role to use data connections
2. **Source-level**: Specific permissions required for individual data sources  
3. **Query-level**: Row and column-level security applied during queries
4. **Result-level**: Additional filtering applied to query results

### Credential Management

#### Secure Credential Storage

**Database connection credentials:**
- **Encryption at rest**: All credentials encrypted using strong encryption
- **Secure transmission**: Credentials transmitted only over encrypted connections
- **Access control**: Only authorized systems can access stored credentials

#### AI Provider Credential Security

**LLM provider API keys:**
- **Encrypted storage**: All AI provider credentials encrypted at rest
- **Scope limitation**: Credentials configured with minimal necessary permissions

### Data Privacy and Compliance

#### Data Processing Privacy

**Data sent to AI providers:**
- **Query text**: Natural language questions (may contain business context)
- **Schema information**: Table and column names for query generation
- **No raw data**: Actual data values never sent to external AI services

**Data retention and privacy:**
- **Local processing**: Data analysis performed within your infrastructure
- **Minimal external data**: Only metadata sent to AI providers
- **Data sovereignty**: Support for regional data processing requirements

---

## Troubleshooting Security Issues

### Common Authentication Problems

#### Login Failures

**"Invalid username or password"**
- **Verify credentials**: Ensure username and password are correct
- **Check account status**: Verify account is not locked or disabled
- **Password expiration**: Check if password has expired

**"Session expired"**
- **Token renewal**: Check if automatic token renewal is working
- **Session timeout**: Verify session timeout configuration
- **Network connectivity**: Ensure stable connection during session
- **Clear browser cache**: Try clearing cookies and browser cache

#### Permission Issues

**"Access denied" or "Insufficient permissions"**
- **Role verification**: Check user's assigned roles in user management
- **Permission propagation**: Wait for permission changes to take effect
- **Resource-specific access**: Verify user has access to specific data sources
- **Session refresh**: Log out and back in to refresh permissions

### API Authentication Issues

#### API Key Problems

**"Invalid API key"**
- **Key format**: Verify API key format and completeness
- **Key status**: Check if API key has been revoked or expired
- **User permissions**: Ensure API key owner has required permissions
- **Rate limiting**: Check if requests are being rate limited

**API key management issues:**
- **Key visibility**: Use reveal toggle to verify key is copied correctly
- **Key rotation**: Generate new key if current key is compromised
- **Permission inheritance**: Verify user's role provides necessary API permissions

---

## Security Best Practices

### For Users

#### Account Security

**Password management:**
- Use strong, unique passwords for Simba Intelligence accounts
- Enable password managers to generate and store secure passwords
- Change passwords immediately if compromise is suspected
- Report suspicious account activity to administrators

**Session security:**
- Always log out when finished, especially on shared computers
- Don't share login credentials with colleagues
- Use secure networks and avoid public WiFi for accessing sensitive data
- Keep browser and security software updated

#### API Security

**API key protection:**
- Never share API keys or commit them to version control
- Store API keys securely using environment variables or secret management
- Rotate API keys regularly according to security policies
- Monitor API key usage for unexpected activity

### For Administrators

#### System Hardening

**Security configuration:**
- Enforce strong password policies and multi-factor authentication
- Implement principle of least privilege for all user accounts
- Regular security audits and permission reviews
- Keep system components updated with latest security patches

**Monitoring and response:**
- Implement comprehensive security monitoring and alerting
- Establish incident response procedures and practice regularly  
- Regular backup and disaster recovery testing
- Maintain security documentation and runbooks

#### Compliance and Governance

**Data governance:**
- Implement appropriate data classification and handling procedures
- Ensure compliance with relevant regulations (GDPR, HIPAA, SOX, etc.)
- Regular compliance audits and documentation updates
- Data retention and destruction policies

**Security policies:**
- Develop and maintain comprehensive security policies
- Regular security training for users and administrators
- Vendor security assessments for third-party integrations
- Business continuity and disaster recovery planning

---

*This security guide provides the foundation for implementing and maintaining strong security in your Simba Intelligence deployment. Security is an ongoing process that requires regular attention, monitoring, and updates. For additional security questions or incident response, contact your organization's security team or Simba Intelligence support.*
