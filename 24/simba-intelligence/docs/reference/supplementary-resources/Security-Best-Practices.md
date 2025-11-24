# Security Best Practices

This guide provides comprehensive security hardening recommendations for Simba Intelligence deployments. It covers authentication, network security, data protection, and operational security practices to ensure your AI-powered data platform meets enterprise security requirements.

## Overview

### Security Framework

Think of security hardening like building a fortress with multiple defensive layers. Just as a medieval castle had outer walls, inner walls, guards, and secure vaults, Simba Intelligence requires multiple security layers working together to protect your data and systems.

**Defense in Depth Strategy:**
```
┌─────────────────────────────────────────────────────────────┐
│                External Perimeter Security                  │
│ Firewalls • DDoS Protection • Geographic Filtering         │
├─────────────────────────────────────────────────────────────┤
│                Network Security Layer                       │
│ TLS Encryption • VPN Access • Network Segmentation         │
├─────────────────────────────────────────────────────────────┤
│                Application Security                         │
│ Authentication • Authorization • Input Validation          │
├─────────────────────────────────────────────────────────────┤
│                  Data Security                              │
│ Encryption at Rest • Access Controls • Audit Logging       │
├─────────────────────────────────────────────────────────────┤
│                Infrastructure Security                      │
│ Container Security • Secrets Management • Monitoring       │
└─────────────────────────────────────────────────────────────┘
```

### Security Principles

**Core security principles for Simba Intelligence:**
- **Zero Trust**: Verify every request, never assume trust
- **Least Privilege**: Grant minimum necessary permissions
- **Defense in Depth**: Multiple security layers prevent single points of failure
- **Continuous Monitoring**: Real-time security monitoring and alerting
- **Incident Response**: Prepared procedures for security events

---

## Authentication and Authorization Security

### User Account Security

#### Strong Authentication Configuration

**Implement robust password policies:**
```bash
# Environment configuration for strong authentication
SESSION_TIMEOUT=7200                    # 2 hours maximum session
MAX_CONCURRENT_SESSIONS=2               # Limit concurrent sessions
PASSWORD_MIN_LENGTH=14                  # Strong password length
PASSWORD_COMPLEXITY=true                # Require complex passwords
ACCOUNT_LOCKOUT_THRESHOLD=3            # Lock after 3 failed attempts
LOCKOUT_DURATION=1800                  # 30 minute lockout period
PASSWORD_HISTORY_COUNT=5               # Remember last 5 passwords
MAX_PASSWORD_AGE=7776000               # 90 day password expiration
```

**Production authentication configuration:**
```yaml
# Kubernetes production authentication
simba:
  intelligence:
    security:
      authentication:
        sessionTimeout: 7200
        maxConcurrentSessions: 2
        forcePasswordChange: true
        passwordPolicy:
          minLength: 14
          requireUppercase: true
          requireLowercase: true
          requireNumbers: true
          requireSpecialChars: true
          historyCount: 5
          maxAge: 90
        accountLockout:
          enabled: true
          threshold: 3
          duration: 1800
```

#### Multi-Factor Authentication Integration

**Configure MFA for enhanced security:**
```yaml
# MFA configuration (if integrating with enterprise SSO)
authentication:
  mfa:
    enabled: true
    methods:
      - totp              # Time-based one-time passwords
      - sms               # SMS verification (if available)
      - hardware_tokens   # FIDO/U2F tokens
    enforcement:
      admin_users: required
      regular_users: optional
      api_access: recommended
```

### Role-Based Access Control Hardening

#### Principle of Least Privilege

**Optimize role assignments:**
```sql
-- Example role optimization for production
-- Create specialized roles instead of broad permissions

-- Read-only analyst role
CREATE ROLE readonly_analyst;
-- Grant only Playground access
GRANT PLAYGROUND_ACCESS TO readonly_analyst;

-- Data creator role (limited)
CREATE ROLE limited_data_creator; 
GRANT PLAYGROUND_ACCESS, DATA_AGENT_ACCESS TO limited_data_creator;
-- Restrict to specific data sources only

-- Connection manager role (infrastructure focused)
CREATE ROLE connection_manager;
GRANT PLAYGROUND_ACCESS, DATA_CONNECTOR_MANAGEMENT TO connection_manager;
-- No data source creation capabilities

-- Full supervisor access (limited to essential personnel)
CREATE ROLE system_supervisor;
GRANT ALL_PERMISSIONS TO system_supervisor;
-- Assign only to essential administrators
```

**Role audit and cleanup procedures:**
```bash
#!/bin/bash
# role-audit.sh - Regular role review

echo "Role-Based Access Control Audit - $(date)"
echo "========================================"

# Audit supervisor access (should be minimal)
echo "Users with supervisor access:"
curl -s -H "Authorization: Bearer admin-token" \
  https://your-domain/api/v1/admin/users | \
  jq '.[] | select(.roles[] | contains("supervisor")) | {username: .username, last_login: .last_login}'

# Identify inactive users
echo ""
echo "Users inactive for 90+ days:"
curl -s -H "Authorization: Bearer admin-token" \
  https://your-domain/api/v1/admin/users | \
  jq --arg cutoff "$(date -d '90 days ago' +%Y-%m-%d)" \
    '.[] | select(.last_login < $cutoff) | {username: .username, last_login: .last_login, roles: .roles}'

# API key audit
echo ""
echo "API keys requiring attention:"
curl -s -H "Authorization: Bearer admin-token" \
  https://your-domain/api/v1/admin/api-keys | \
  jq --arg cutoff "$(date -d '30 days ago' +%Y-%m-%d)" \
    '.[] | select(.expires_at < "'$(date -d '30 days from now' +%Y-%m-%d)'" or .last_used < $cutoff) | 
    {user: .user_id, expires: .expires_at, last_used: .last_used}'
```

### API Security Hardening

#### API Key Management Security

**Secure API key lifecycle:**
```bash
# API key security configuration
API_KEY_EXPIRATION=15552000              # 6 months maximum
API_KEY_ROTATION_WARNING=2592000         # 30 days advance warning
API_KEY_MAX_PER_USER=5                   # Limit keys per user
API_VALIDATE_USER_AGENT=true             # Validate client identification
API_REQUIRE_HTTPS=true                   # Force HTTPS for all API calls
API_ENABLE_CORS=false                    # Disable CORS for API-only access
```

**API rate limiting and protection:**
```bash
# Rate limiting configuration
API_RATE_LIMIT_ENABLED=true
API_REQUESTS_PER_MINUTE=200              # Increased for production
API_BURST_LIMIT=50                       # Higher burst capacity
API_RATE_LIMIT_WINDOW=60                 # 1 minute window
API_ABUSE_DETECTION=true                 # Enable abuse detection
API_SUSPICIOUS_ACTIVITY_THRESHOLD=1000   # Alert on high usage
```

---

## Network Security Configuration

### TLS/SSL Security

#### Certificate Management

**Production SSL configuration:**
```yaml
# Kubernetes TLS configuration
ingress:
  enabled: true
  className: "nginx"
  annotations:
    # Force HTTPS redirection
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    # Use only strong ciphers
    nginx.ingress.kubernetes.io/ssl-ciphers: "ECDHE-RSA-AES128-GCM-SHA256,ECDHE-RSA-AES256-GCM-SHA384"
    # Enable HSTS
    nginx.ingress.kubernetes.io/custom-http-errors: "400,401,403,404,405,407,408,409,410,411,412,413,414,431,500,501,502,503,504,505"
    nginx.ingress.kubernetes.io/configuration-snippet: |
      add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
      add_header X-Content-Type-Options "nosniff" always;
      add_header X-Frame-Options "DENY" always;
      add_header X-XSS-Protection "1; mode=block" always;
  
  tls:
    - secretName: simba-intelligence-tls
      hosts:
        - simba-intelligence.yourdomain.com
```

**Certificate security best practices:**
```bash
# Certificate validation script
#!/bin/bash
# check-certificate-security.sh

DOMAIN="simba-intelligence.yourdomain.com"

echo "Certificate Security Analysis for $DOMAIN"
echo "========================================"

# Check certificate validity
openssl s_client -connect $DOMAIN:443 -servername $DOMAIN < /dev/null 2>/dev/null | \
  openssl x509 -noout -dates

# Check certificate chain
echo ""
echo "Certificate Chain Validation:"
openssl s_client -connect $DOMAIN:443 -servername $DOMAIN < /dev/null 2>/dev/null | \
  openssl x509 -noout -issuer -subject

# Check TLS version support
echo ""
echo "TLS Version Support:"
for version in tls1 tls1_1 tls1_2 tls1_3; do
  if openssl s_client -connect $DOMAIN:443 -$version < /dev/null &>/dev/null; then
    echo "✅ $version: Supported"
  else
    echo "❌ $version: Not supported (good if deprecated)"
  fi
done

# Check cipher strength
echo ""
echo "Cipher Strength Analysis:"
nmap --script ssl-enum-ciphers -p 443 $DOMAIN
```

### Network Isolation and Firewalls

#### Network Security Configuration

**Kubernetes network policies:**
```yaml
# Network policy for production security
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: simba-intelligence-network-policy
  namespace: simba-intelligence
spec:
  podSelector:
    matchLabels:
      app.kubernetes.io/name: simba-intelligence
  
  policyTypes:
  - Ingress
  - Egress
  
  ingress:
  # Allow ingress from ingress controller only
  - from:
    - namespaceSelector:
        matchLabels:
          name: ingress-nginx
    ports:
    - protocol: TCP
      port: 5000
  
  # Allow monitoring access
  - from:
    - namespaceSelector:
        matchLabels:
          name: monitoring
    ports:
    - protocol: TCP
      port: 9090  # Metrics port
  
  egress:
  # Allow DNS resolution
  - to: []
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
  
  # Allow HTTPS to AI providers
  - to: []
    ports:
    - protocol: TCP
      port: 443
  
  # Allow database connections (specific to your setup)
  - to:
    - namespaceSelector:
        matchLabels:
          name: database
    ports:
    - protocol: TCP
      port: 5432
  
  # Allow Redis connections
  - to:
    - podSelector:
        matchLabels:
          app.kubernetes.io/name: redis
    ports:
    - protocol: TCP
      port: 6379
```

**Firewall rules for Docker deployment:**
```bash
#!/bin/bash
# configure-firewall.sh - Production firewall setup

echo "Configuring production firewall rules..."

# Default deny all
ufw default deny incoming
ufw default deny outgoing

# Allow SSH (restrict by source IP in production)
ufw allow 22/tcp

# Allow HTTP/HTTPS for web interface
ufw allow 80/tcp
ufw allow 443/tcp

# Allow outbound HTTPS for AI providers
ufw allow out 443/tcp

# Allow outbound database connections (adjust ports as needed)
ufw allow out 5432/tcp  # PostgreSQL
ufw allow out 1433/tcp  # SQL Server
ufw allow out 6379/tcp  # Redis

# Deny direct access to internal services
ufw deny 5050/tcp   # Flask application
ufw deny 8080/tcp   # Composer service
ufw deny 6379/tcp   # Redis (inbound)
ufw deny 5432/tcp   # PostgreSQL (inbound)

# Enable firewall
ufw enable

# Show current rules
ufw status numbered

echo "Firewall configuration completed"
```

---

## Container and Infrastructure Security

### Container Security Hardening

#### Pod Security Standards

**Implement restrictive pod security contexts:**
```yaml
# Secure pod security context
securityContext:
  # Run as non-root user
  runAsNonRoot: true
  runAsUser: 1000
  runAsGroup: 3000
  
  # Filesystem security
  readOnlyRootFilesystem: true
  allowPrivilegeEscalation: false
  
  # Linux capabilities
  capabilities:
    drop:
    - ALL
    add:
    - NET_BIND_SERVICE  # Only if binding to privileged ports
  
  # Security profiles
  seccompProfile:
    type: RuntimeDefault
  apparmorProfile:
    type: RuntimeDefault

# Pod-level security context
podSecurityContext:
  runAsNonRoot: true
  fsGroup: 2000
  fsGroupChangePolicy: "OnRootMismatch"
  supplementalGroups: [3000]
```

#### Container Image Security

**Secure container image practices:**
```dockerfile
# Security-hardened Dockerfile example
FROM python:3.11-slim

# Create non-root user early
RUN groupadd -r simba && useradd -r -g simba simba

# Install security updates
RUN apt-get update && apt-get upgrade -y && \
    apt-get install -y --no-install-recommends \
    # Only essential packages
    curl \
    && rm -rf /var/lib/apt/lists/* \
    && apt-get clean

# Install Python dependencies as root, then switch to user
COPY requirements.txt /tmp/
RUN pip install --no-cache-dir -r /tmp/requirements.txt && \
    rm /tmp/requirements.txt

# Copy application code
COPY --chown=simba:simba src/ /app/src/
COPY --chown=simba:simba config/ /app/config/

# Create necessary directories with proper ownership
RUN mkdir -p /app/logs /app/tmp && \
    chown -R simba:simba /app

# Switch to non-root user
USER simba
WORKDIR /app

# Use non-root port
EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1

# Secure startup
CMD ["python", "-m", "src.main.app"]
```

**Container scanning integration:**
```bash
#!/bin/bash
# container-security-scan.sh

IMAGE_NAME="your-registry/simba-intelligence:latest"

echo "Container Security Scanning: $IMAGE_NAME"
echo "========================================"

# Scan for vulnerabilities using Trivy
trivy image --severity HIGH,CRITICAL $IMAGE_NAME

# Check for common misconfigurations
trivy config .

# Verify image signature (if using signed images)
cosign verify $IMAGE_NAME

# Check for secrets in image
truffleHog docker --image $IMAGE_NAME

echo "Container security scan completed"
```

### Kubernetes Security Hardening

#### Cluster Security Configuration

**Pod Security Standards enforcement:**
```yaml
# Pod Security Standards configuration
apiVersion: v1
kind: Namespace
metadata:
  name: simba-intelligence
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

**Resource quotas and limits:**
```yaml
# Resource quota for security and stability
apiVersion: v1
kind: ResourceQuota
metadata:
  name: simba-intelligence-quota
  namespace: simba-intelligence
spec:
  hard:
    requests.cpu: "10"
    requests.memory: "20Gi"
    limits.cpu: "20"
    limits.memory: "40Gi"
    persistentvolumeclaims: "10"
    services: "5"
    secrets: "10"
    configmaps: "10"

---
# Limit range to enforce security policies
apiVersion: v1
kind: LimitRange
metadata:
  name: simba-intelligence-limits
  namespace: simba-intelligence
spec:
  limits:
  - type: Container
    default:
      cpu: 1000m
      memory: 2Gi
    defaultRequest:
      cpu: 100m
      memory: 256Mi
    max:
      cpu: 4000m
      memory: 8Gi
    min:
      cpu: 50m
      memory: 128Mi
  - type: Pod
    max:
      cpu: 8000m
      memory: 16Gi
```

#### RBAC Security Configuration

**Minimize service account permissions:**
```yaml
# Minimal service account for Simba Intelligence
apiVersion: v1
kind: ServiceAccount
metadata:
  name: simba-intelligence
  namespace: simba-intelligence
automountServiceAccountToken: false  # Disable automatic token mounting

---
# Minimal RBAC permissions
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: simba-intelligence-role
  namespace: simba-intelligence
rules:
# Only essential permissions for operation
- apiGroups: [""]
  resources: ["configmaps", "secrets"]
  verbs: ["get", "list"]
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
  
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: simba-intelligence-binding
  namespace: simba-intelligence
subjects:
- kind: ServiceAccount
  name: simba-intelligence
  namespace: simba-intelligence
roleRef:
  kind: Role
  name: simba-intelligence-role
  apiGroup: rbac.authorization.k8s.io
```

---

## Database Security Hardening

### PostgreSQL Security Configuration

#### Database Access Controls

**Implement restrictive database security:**
```sql
-- Create security-focused database configuration

-- 1. Create dedicated database user with minimal permissions
CREATE USER simba_analytics_user WITH 
    PASSWORD 'SecureComplexPassword123!'
    NOSUPERUSER
    NOCREATEDB
    NOCREATEROLE
    NOINHERIT
    LOGIN
    CONNECTION LIMIT 50;

-- 2. Create security schema for access control
CREATE SCHEMA security_controls;

-- 3. Create role for Simba Intelligence with specific permissions
CREATE ROLE simba_intelligence_role NOINHERIT;

-- 4. Grant only necessary schema access
GRANT USAGE ON SCHEMA public TO simba_intelligence_role;
GRANT USAGE ON SCHEMA analytics TO simba_intelligence_role;
GRANT USAGE ON SCHEMA security_controls TO simba_intelligence_role;

-- 5. Grant table-level permissions (not schema-wide)
-- Be explicit about what tables can be accessed
GRANT SELECT ON analytics.sales_summary TO simba_intelligence_role;
GRANT SELECT ON analytics.customer_metrics TO simba_intelligence_role;
GRANT SELECT ON public.product_catalog TO simba_intelligence_role;

-- 6. Create security policies for row-level access
ALTER TABLE analytics.sales_summary ENABLE ROW LEVEL SECURITY;

-- 7. Create security policy function
CREATE FUNCTION security_controls.simba_access_policy(user_region TEXT)
RETURNS BOOLEAN AS $$
BEGIN
    -- Allow access only to user's region data
    RETURN user_region = current_setting('app.user_region', true);
END;
$$ LANGUAGE plpgsql;

-- 8. Apply row-level security policy
CREATE POLICY simba_regional_access ON analytics.sales_summary
    FOR SELECT
    TO simba_intelligence_role
    USING (security_controls.simba_access_policy(region));

-- 9. Grant role to user
GRANT simba_intelligence_role TO simba_analytics_user;
```

#### Database Connection Security

**Configure secure database connections:**
```bash
# PostgreSQL configuration for security (postgresql.conf)
# Network and connection security
listen_addresses = 'localhost,10.0.0.0/24'    # Restrict listening interfaces
port = 5432
max_connections = 200
superuser_reserved_connections = 3

# SSL configuration
ssl = on
ssl_cert_file = '/var/lib/postgresql/data/server.crt'
ssl_key_file = '/var/lib/postgresql/data/server.key'
ssl_ca_file = '/var/lib/postgresql/data/ca.crt'
ssl_min_protocol_version = 'TLSv1.2'
ssl_max_protocol_version = 'TLSv1.3'

# Authentication security
password_encryption = scram-sha-256
krb_server_keyfile = '/var/lib/postgresql/data/postgresql.keytab'

# Logging for security monitoring
log_connections = on
log_disconnections = on
log_failed_authentication = on
log_statement = 'ddl'
log_min_duration_statement = 5000
```

**pg_hba.conf security configuration:**
```bash
# pg_hba.conf - Secure authentication configuration
# TYPE  DATABASE        USER                    ADDRESS                 METHOD

# Local connections (Unix sockets)
local   all             postgres                                        peer
local   all             all                                             md5

# Remote connections - be very specific about source IPs
hostssl simba_intelligence simba_analytics_user 10.0.1.0/24          scram-sha-256
hostssl simba_intelligence simba_analytics_user 192.168.1.100/32     scram-sha-256

# Reject all other connections
host    all             all                     0.0.0.0/0               reject
```

### External Database Security

#### Secure EDC Configuration

**Database-specific security settings:**
```yaml
# SQL Server EDC security configuration
sql_server_edc:
  connection_security:
    encrypt_connection: true
    trust_server_certificate: false
    certificate_validation: true
    connection_timeout: 30
    
  authentication:
    method: "sql_server_auth"  # Or windows_auth for domain environments
    username: "simba_sqlserver_user"
    # Password stored in Kubernetes secret
    
  query_restrictions:
    allowed_operations: ["SELECT"]
    blocked_keywords: ["DROP", "DELETE", "UPDATE", "INSERT", "TRUNCATE", "ALTER"]
    max_query_time: 300
    max_result_rows: 100000

# PostgreSQL EDC security configuration  
postgresql_edc:
  connection_security:
    ssl_mode: "require"
    ssl_cert_verification: true
    connection_timeout: 30
    
  access_controls:
    readonly_user: true
    schema_restrictions: ["analytics", "reporting"]
    table_whitelist: true  # Only allow explicitly granted tables
```

---

## Credential and Secrets Security

### Secrets Management

#### Kubernetes Secrets Security

**Secure secrets configuration:**
```yaml
# Encrypted secrets for AI providers
apiVersion: v1
kind: Secret
metadata:
  name: vertex-ai-credentials
  namespace: simba-intelligence
  annotations:
    # Encryption configuration
    encryption.kubernetes.io/provider: "aes-cbc"
type: Opaque
data:
  credentials.json: <base64-encoded-service-account-json>

---
# Database credentials with rotation metadata
apiVersion: v1
kind: Secret
metadata:
  name: database-credentials
  namespace: simba-intelligence
  labels:
    credential-type: "database"
    rotation-schedule: "quarterly"
type: Opaque
stringData:
  username: "simba_analytics_user"
  password: "SecureComplexPassword123!"
  connection-string: "postgresql://simba_analytics_user:SecureComplexPassword123!@postgres.company.com:5432/simba_intelligence?sslmode=require"
```

**External secrets management integration:**
```yaml
# External Secrets Operator configuration
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: vault-backend
  namespace: simba-intelligence
spec:
  provider:
    vault:
      server: "https://vault.yourdomain.com"
      path: "secret"
      version: "v2"
      auth:
        kubernetes:
          mountPath: "kubernetes"
          role: "simba-intelligence"
          secretRef:
            name: vault-auth-secret
            key: vault-token

---
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: ai-provider-credentials
  namespace: simba-intelligence
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: vault-backend
    kind: SecretStore
  target:
    name: ai-credentials
    creationPolicy: Owner
  data:
  - secretKey: vertex-ai-key
    remoteRef:
      key: ai-providers
      property: vertex-ai-service-account
  - secretKey: openai-key
    remoteRef:
      key: ai-providers  
      property: openai-api-key
```

#### Credential Rotation Procedures

**Automated credential rotation:**
```bash
#!/bin/bash
# credential-rotation.sh

ROTATION_LOG="/var/log/credential-rotation.log"

log_rotation_event() {
    echo "$(date): $1" >> "$ROTATION_LOG"
}

rotate_database_credentials() {
    log_rotation_event "Starting database credential rotation"
    
    # Generate new secure password
    NEW_PASSWORD=$(openssl rand -base64 32 | tr -d "=+/" | cut -c1-25)
    
    # Update password in database
    psql "$DATABASE_URL" -c "ALTER USER simba_analytics_user PASSWORD '$NEW_PASSWORD';"
    
    # Update Kubernetes secret
    kubectl patch secret database-credentials -n simba-intelligence \
      --type='json' \
      -p="[{\"op\": \"replace\", \"path\": \"/data/password\", \"value\":\"$(echo -n $NEW_PASSWORD | base64 -w 0)\"}]"
    
    # Test new credentials
    if psql "postgresql://simba_analytics_user:$NEW_PASSWORD@$DB_HOST:$DB_PORT/$DB_NAME" -c "SELECT 1;" &>/dev/null; then
        log_rotation_event "Database credential rotation successful"
    else
        log_rotation_event "ERROR: Database credential rotation failed"
        # Rollback procedures here
    fi
    
    # Restart application to pick up new credentials
    kubectl rollout restart deployment/simba-intelligence-website -n simba-intelligence
}

rotate_ai_credentials() {
    log_rotation_event "Starting AI provider credential rotation"
    
    # This would integrate with your AI provider credential management
    # For example, rotating service account keys in Google Cloud
    
    # Create new service account key
    gcloud iam service-accounts keys create new-key.json \
      --iam-account=simba-intelligence@project.iam.gserviceaccount.com
    
    # Update Kubernetes secret
    kubectl create secret generic vertex-ai-credentials-new \
      --from-file=credentials.json=new-key.json \
      -n simba-intelligence
    
    # Test new credentials
    # Switch to new secret in deployment
    # Delete old credentials after validation
}

# Schedule rotations
case "${1:-help}" in
    "database")
        rotate_database_credentials
        ;;
    "ai")
        rotate_ai_credentials
        ;;
    *)
        echo "Usage: $0 {database|ai}"
        echo "Schedule with cron:"
        echo "0 2 1 */3 * /path/to/credential-rotation.sh database  # Quarterly"
        echo "0 3 1 */6 * /path/to/credential-rotation.sh ai       # Semi-annually"
        ;;
esac
```

---

## Data Protection and Privacy

### Data Encryption

#### Encryption at Rest and in Transit

**Database encryption configuration:**
```sql
-- Enable database encryption features
-- Transparent Data Encryption (if available)
CREATE EXTENSION IF NOT EXISTS pgcrypto;

-- Encrypt sensitive columns
CREATE TABLE encrypted_customer_data (
    customer_id SERIAL PRIMARY KEY,
    customer_name TEXT,
    email TEXT,
    -- Encrypt sensitive personal information
    ssn_encrypted BYTEA,
    credit_card_encrypted BYTEA,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Create functions for encryption/decryption
CREATE OR REPLACE FUNCTION encrypt_sensitive_data(plaintext TEXT, key TEXT)
RETURNS BYTEA AS $$
    SELECT pgp_sym_encrypt(plaintext, key);
$$ LANGUAGE SQL;

CREATE OR REPLACE FUNCTION decrypt_sensitive_data(ciphertext BYTEA, key TEXT)  
RETURNS TEXT AS $$
    SELECT pgp_sym_decrypt(ciphertext, key);
$$ LANGUAGE SQL;

-- Grant minimal access to encryption functions
GRANT EXECUTE ON FUNCTION encrypt_sensitive_data(TEXT, TEXT) TO simba_intelligence_role;
-- Do NOT grant decrypt function to Simba Intelligence user
```

**Application-level encryption configuration:**
```bash
# Data encryption settings
ENABLE_DATA_ENCRYPTION=true
ENCRYPTION_KEY_ID=your-encryption-key-id
ENCRYPT_LOGS=true
ENCRYPT_CACHE=false                     # Performance consideration
ENCRYPT_BACKUPS=true
FIELD_LEVEL_ENCRYPTION=true             # Encrypt sensitive fields
```

#### Data Masking and Anonymization

**Implement data masking for development environments:**
```sql
-- Create data masking views for non-production environments
CREATE VIEW analytics.customers_masked AS
SELECT 
    customer_id,
    -- Mask email addresses
    REGEXP_REPLACE(email, '(.{2})(.*)', '\1****') as email_masked,
    -- Mask customer names  
    REGEXP_REPLACE(customer_name, '(.{3})(.*)', '\1****') as name_masked,
    region,
    customer_since,
    -- Keep non-sensitive analytics data
    total_orders,
    total_revenue,
    avg_order_value
FROM customers;

-- Grant access to masked view instead of real table
GRANT SELECT ON analytics.customers_masked TO simba_dev_role;
REVOKE SELECT ON customers FROM simba_dev_role;
```

### Data Access Controls

#### Implement Data Governance

**Row-level security for multi-tenant scenarios:**
```sql
-- Multi-tenant row-level security setup
CREATE TABLE tenant_access (
    user_name TEXT,
    tenant_id TEXT,
    access_level TEXT CHECK (access_level IN ('read', 'write', 'admin'))
);

-- Security policy for tenant data isolation
CREATE FUNCTION security_controls.tenant_isolation_policy(data_tenant_id TEXT)
RETURNS BOOLEAN AS $$
BEGIN
    -- Allow access only to user's assigned tenants
    RETURN data_tenant_id IN (
        SELECT tenant_id 
        FROM tenant_access 
        WHERE user_name = current_user
          AND access_level IN ('read', 'write', 'admin')
    );
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

-- Apply policy to sensitive tables
ALTER TABLE customer_data ENABLE ROW LEVEL SECURITY;
CREATE POLICY tenant_data_access ON customer_data
    FOR ALL
    TO simba_intelligence_role
    USING (security_controls.tenant_isolation_policy(tenant_id));
```

---

## Monitoring and Audit Security

### Security Monitoring

#### Comprehensive Security Logging

**Configure security event logging:**
```bash
# Security logging configuration
ENABLE_AUDIT_LOGGING=true
AUDIT_LOG_LEVEL=INFO
SECURITY_LOG_LEVEL=INFO
AUDIT_LOG_RETENTION_DAYS=365
LOG_SANITIZE_SENSITIVE=true
LOG_INCLUDE_USER_ID=true
LOG_INCLUDE_REQUEST_ID=true

# Security event types to log
AUDIT_LOGIN_EVENTS=true
AUDIT_PERMISSION_CHANGES=true
AUDIT_DATA_ACCESS=true
AUDIT_API_USAGE=true
AUDIT_ADMIN_ACTIONS=true
AUDIT_FAILED_AUTH=true
```

**Security monitoring script:**
```bash
#!/bin/bash
# security-monitoring.sh

SECURITY_LOG="/var/log/simba-intelligence/security.log"
ALERT_THRESHOLD_FILE="/tmp/security-thresholds"

# Create alert thresholds if not exist
cat > "$ALERT_THRESHOLD_FILE" << EOF
FAILED_LOGINS_PER_HOUR=50
API_ERRORS_PER_HOUR=100
PRIVILEGE_ESCALATION_ATTEMPTS=1
UNUSUAL_DATA_ACCESS_THRESHOLD=1000
EOF

source "$ALERT_THRESHOLD_FILE"

echo "Security Monitoring Report - $(date)"
echo "===================================="

# Check failed login attempts
FAILED_LOGINS=$(grep "authentication_failed" "$SECURITY_LOG" | \
  grep "$(date +%Y-%m-%d %H)" | wc -l)

if [[ $FAILED_LOGINS -gt $FAILED_LOGINS_PER_HOUR ]]; then
    echo "🚨 ALERT: High failed login attempts: $FAILED_LOGINS in last hour"
    # Send security alert
    curl -X POST "$SECURITY_WEBHOOK" \
      -d "{\"alert\":\"High failed login attempts\",\"count\":$FAILED_LOGINS}"
fi

# Check for privilege escalation attempts
PRIVILEGE_ATTEMPTS=$(grep "privilege_escalation" "$SECURITY_LOG" | \
  grep "$(date +%Y-%m-%d)" | wc -l)

if [[ $PRIVILEGE_ATTEMPTS -gt 0 ]]; then
    echo "🚨 CRITICAL: Privilege escalation attempts detected: $PRIVILEGE_ATTEMPTS"
    # Immediate security team notification
fi

# Check API abuse patterns
API_ERRORS=$(grep "api_error" "$SECURITY_LOG" | \
  grep "$(date +%Y-%m-%d %H)" | wc -l)

if [[ $API_ERRORS -gt $API_ERRORS_PER_HOUR ]]; then
    echo "⚠️ WARNING: High API error rate: $API_ERRORS in last hour"
fi

# Check for unusual data access patterns
LARGE_QUERIES=$(grep "large_result_set" "$SECURITY_LOG" | \
  grep "$(date +%Y-%m-%d)" | wc -l)

if [[ $LARGE_QUERIES -gt 10 ]]; then
    echo "⚠️ WARNING: Unusual data access pattern: $LARGE_QUERIES large queries today"
fi

# Generate security summary
echo ""
echo "Security Summary:"
echo "Failed logins (last hour): $FAILED_LOGINS"
echo "Privilege escalation attempts (today): $PRIVILEGE_ATTEMPTS"  
echo "API errors (last hour): $API_ERRORS"
echo "Large data access queries (today): $LARGE_QUERIES"
```

### Compliance and Audit

#### Audit Trail Configuration

**Comprehensive audit logging:**
```python
# audit_logging.py - Conceptual audit configuration
audit_configuration = {
    "events_to_audit": [
        "user_authentication",
        "user_authorization", 
        "data_access",
        "query_execution",
        "admin_actions",
        "configuration_changes",
        "api_key_usage",
        "export_operations"
    ],
    
    "audit_data_fields": [
        "timestamp",
        "user_id", 
        "source_ip",
        "user_agent",
        "action_type",
        "resource_accessed",
        "outcome",
        "error_details"
    ],
    
    "retention_policies": {
        "security_events": "7_years",
        "data_access": "3_years", 
        "administrative_actions": "7_years",
        "general_audit": "1_year"
    }
}
```

**Compliance reporting automation:**
```bash
#!/bin/bash
# compliance-report.sh

REPORT_PERIOD=${1:-"monthly"}
REPORT_DATE=$(date +%Y-%m-%d)
REPORT_FILE="/var/log/compliance/compliance-report-$REPORT_DATE.txt"

{
    echo "Simba Intelligence Compliance Report"
    echo "Report Period: $REPORT_PERIOD"
    echo "Generated: $(date)"
    echo "=================================="
    
    # User access summary
    echo "User Access Summary:"
    curl -s -H "Authorization: Bearer admin-token" \
      https://your-domain/api/v1/admin/users | \
      jq -r '.[] | "\(.username): \(.roles | join(", ")) - Last login: \(.last_login)"'
    
    # Data access patterns
    echo ""
    echo "Data Access Patterns:"
    grep "data_access" /var/log/simba-intelligence/audit.log | \
      grep "$(date -d '1 month ago' +%Y-%m)" | \
      awk '{print $3}' | sort | uniq -c | sort -nr | head -10
    
    # Security events
    echo ""
    echo "Security Events:"
    grep -E "(failed_auth|privilege_escalation|security_violation)" \
      /var/log/simba-intelligence/security.log | \
      grep "$(date -d '1 month ago' +%Y-%m)" | wc -l
    echo "security events in report period"
    
    # API usage summary
    echo ""
    echo "API Usage Summary:"
    grep "api_request" /var/log/simba-intelligence/api.log | \
      grep "$(date -d '1 month ago' +%Y-%m)" | \
      awk '{print $3}' | sort | uniq -c | sort -nr | head -5
      
} > "$REPORT_FILE"

# Send to compliance team
mail -s "Simba Intelligence Compliance Report - $REPORT_DATE" \
  compliance@yourdomain.com < "$REPORT_FILE"

echo "Compliance report generated: $REPORT_FILE"
```

---

## Incident Response Security

### Security Incident Procedures

#### Automated Security Response

**Security incident detection and response:**
```bash
#!/bin/bash
# security-incident-response.sh

INCIDENT_TYPE=$1
SEVERITY=${2:-"medium"}
INCIDENT_ID="SEC-$(date +%Y%m%d%H%M%S)"

echo "SECURITY INCIDENT RESPONSE ACTIVATED"
echo "Incident ID: $INCIDENT_ID"
echo "Type: $INCIDENT_TYPE"
echo "Severity: $SEVERITY"

case $INCIDENT_TYPE in
    "suspected_breach")
        echo "SUSPECTED SECURITY BREACH - IMMEDIATE ACTION"
        
        # 1. Isolate affected systems
        kubectl scale deployment simba-intelligence-website --replicas=0 -n simba-intelligence
        
        # 2. Preserve forensic evidence
        kubectl logs deployment/simba-intelligence-website -n simba-intelligence > \
          "/var/log/forensics/incident-$INCIDENT_ID-app-logs.txt"
        
        # 3. Notify security team immediately
        curl -X POST "$CRITICAL_SECURITY_WEBHOOK" \
          -d "{\"incident_id\":\"$INCIDENT_ID\",\"type\":\"suspected_breach\",\"severity\":\"critical\"}"
        
        # 4. Disable all user sessions
        redis-cli flushdb  # Clear all sessions
        
        # 5. Generate forensic report
        ./scripts/generate-forensic-report.sh $INCIDENT_ID
        ;;
        
    "credential_compromise")
        echo "CREDENTIAL COMPROMISE DETECTED"
        
        # 1. Revoke all API keys for affected user
        AFFECTED_USER=$3
        curl -X DELETE "https://your-domain/api/v1/admin/users/$AFFECTED_USER/api-keys" \
          -H "Authorization: Bearer admin-token"
        
        # 2. Force password reset
        curl -X POST "https://your-domain/api/v1/admin/users/$AFFECTED_USER/force-password-reset" \
          -H "Authorization: Bearer admin-token"
        
        # 3. Review recent activity
        grep "$AFFECTED_USER" /var/log/simba-intelligence/audit.log | tail -50 > \
          "/var/log/security/credential-compromise-$INCIDENT_ID.log"
        ;;
        
    "unauthorized_access")
        echo "UNAUTHORIZED ACCESS ATTEMPT"
        
        # 1. Block source IP if identified
        SUSPECT_IP=$3
        if [[ -n "$SUSPECT_IP" ]]; then
            # Add firewall rule to block IP
            ufw deny from $SUSPECT_IP
        fi
        
        # 2. Enhanced monitoring for next 24 hours
        ./scripts/enhanced-security-monitoring.sh $INCIDENT_ID
        ;;
esac

# Log incident response completion
echo "$(date): Security incident response completed for $INCIDENT_ID" >> \
  /var/log/security-incidents.log
```

#### Forensic Data Collection

**Security forensics collection:**
```bash
#!/bin/bash
# collect-security-forensics.sh

INCIDENT_ID=$1
FORENSICS_DIR="/var/log/forensics/incident-$INCIDENT_ID"

mkdir -p "$FORENSICS_DIR"

echo "Collecting forensic evidence for incident: $INCIDENT_ID"

# 1. System state snapshot
kubectl get all -n simba-intelligence -o yaml > "$FORENSICS_DIR/system-state.yaml"
kubectl get events -n simba-intelligence > "$FORENSICS_DIR/recent-events.txt"

# 2. Application logs (last 24 hours)
kubectl logs deployment/simba-intelligence-website -n simba-intelligence --since=24h > \
  "$FORENSICS_DIR/application-logs.txt"

# 3. Security events
grep -E "(authentication|authorization|security)" \
  /var/log/simba-intelligence/*.log > "$FORENSICS_DIR/security-events.txt"

# 4. Database activity (if accessible)
psql "$DATABASE_URL" -c "
SELECT 
    query_start,
    usename,
    application_name,
    client_addr,
    query
FROM pg_stat_activity 
WHERE query_start >= NOW() - INTERVAL '24 hours'
ORDER BY query_start DESC;
" > "$FORENSICS_DIR/database-activity.txt"

# 5. Network connections
ss -tulpn > "$FORENSICS_DIR/network-connections.txt"
iptables -L -n > "$FORENSICS_DIR/firewall-rules.txt"

# 6. User activity audit
curl -s -H "Authorization: Bearer admin-token" \
  https://your-domain/api/v1/admin/audit-log?since=24h > \
  "$FORENSICS_DIR/user-audit.json"

# 7. Create forensic archive
tar -czf "/var/log/forensics/incident-$INCIDENT_ID.tar.gz" \
  -C /var/log/forensics "incident-$INCIDENT_ID"

# 8. Generate forensic summary
cat > "$FORENSICS_DIR/forensic-summary.txt" << EOF
Security Incident Forensic Report
================================
Incident ID: $INCIDENT_ID
Collection Time: $(date)
Collected By: $(whoami)

Evidence Collected:
- System state snapshot
- Application logs (24 hours)
- Security event logs
- Database activity logs
- Network connection state
- User activity audit trail

Archive Location: /var/log/forensics/incident-$INCIDENT_ID.tar.gz
Archive Size: $(du -h /var/log/forensics/incident-$INCIDENT_ID.tar.gz | cut -f1)

Next Steps:
1. Secure archive for investigation
2. Analyze logs for indicators of compromise
3. Review affected user accounts and data access
4. Implement additional security controls if needed
EOF

echo "Forensic evidence collected: $FORENSICS_DIR"
```

---

## Security Operations

### Security Hardening Checklist

#### Production Deployment Security Checklist

**Pre-production security validation:**
```bash
#!/bin/bash
# security-hardening-checklist.sh

echo "Simba Intelligence Security Hardening Checklist"
echo "=============================================="

CHECKS_PASSED=0
TOTAL_CHECKS=0

check_security_item() {
    local check_name=$1
    local check_command=$2
    local success_message=$3
    local failure_message=$4
    
    ((TOTAL_CHECKS++))
    
    echo -n "Checking $check_name... "
    if eval "$check_command" &>/dev/null; then
        echo "✅ PASS - $success_message"
        ((CHECKS_PASSED++))
    else
        echo "❌ FAIL - $failure_message"
    fi
}

# Authentication security checks
check_security_item "HTTPS Enforcement" \
    "curl -I http://your-domain 2>/dev/null | grep -q 'HTTP/1.1 301'" \
    "HTTP redirects to HTTPS" \
    "HTTP redirection not configured"

check_security_item "TLS Certificate" \
    "echo | openssl s_client -connect your-domain:443 2>/dev/null | grep -q 'Verify return code: 0'" \
    "Valid SSL certificate" \
    "SSL certificate issues detected"

check_security_item "Session Security" \
    "curl -s https://your-domain/api/v1/healthz | grep -q healthy" \
    "Session management working" \
    "Session management issues"

# Database security checks
check_security_item "Database SSL" \
    "psql \"$DATABASE_URL\" -c 'SELECT ssl_is_used();' | grep -q 't'" \
    "Database connections encrypted" \
    "Database connections not encrypted"

# Container security checks  
check_security_item "Non-root Containers" \
    "kubectl get pods -n simba-intelligence -o jsonpath='{.items[*].spec.securityContext.runAsNonRoot}' | grep -q true" \
    "Containers run as non-root" \
    "Some containers may be running as root"

check_security_item "Read-only Filesystem" \
    "kubectl get pods -n simba-intelligence -o jsonpath='{.items[*].spec.containers[*].securityContext.readOnlyRootFilesystem}' | grep -q true" \
    "Read-only root filesystem enabled" \
    "Read-only filesystem not configured"

# Network security checks
check_security_item "Network Policies" \
    "kubectl get networkpolicy -n simba-intelligence | grep -q simba-intelligence" \
    "Network policies configured" \
    "Network policies not found"

# Secrets security checks
check_security_item "Secrets Encryption" \
    "kubectl get secrets -n simba-intelligence -o yaml | grep -q 'type: Opaque'" \
    "Secrets properly configured" \
    "Secrets configuration needs review"

# Summary
echo ""
echo "Security Hardening Summary:"
echo "Checks passed: $CHECKS_PASSED/$TOTAL_CHECKS"

if [[ $CHECKS_PASSED -eq $TOTAL_CHECKS ]]; then
    echo "✅ All security checks passed - deployment ready"
else
    echo "❌ Security hardening incomplete - address failures before production"
fi
```

### Regular Security Maintenance

#### Scheduled Security Tasks

**Weekly security maintenance:**
```bash
#!/bin/bash
# weekly-security-maintenance.sh

echo "Weekly Security Maintenance - $(date)"
echo "===================================="

# 1. Security log review
echo "Reviewing security logs..."
./scripts/analyze-security-logs.sh weekly

# 2. User access audit
echo "Auditing user access..."
./scripts/user-access-audit.sh

# 3. API key management
echo "Checking API key status..."
curl -s -H "Authorization: Bearer admin-token" \
  https://your-domain/api/v1/admin/api-keys | \
  jq '.[] | select(.expires_at < "'$(date -d '30 days from now' +%Y-%m-%d)'") | 
      {user: .user_id, expires: .expires_at}' | \
  while read line; do
    echo "⚠️ API key expiring soon: $line"
done

# 4. Certificate expiration check
echo "Checking certificate expiration..."
echo | openssl s_client -servername your-domain -connect your-domain:443 2>/dev/null | \
  openssl x509 -noout -dates

# 5. Security update check
echo "Checking for security updates..."
# Container image security scanning
trivy image simba-intelligence:latest --severity HIGH,CRITICAL

# 6. Backup security validation
echo "Validating backup security..."
ls -la /backups/simba-intelligence/ | grep "$(date +%Y-%m-%d)"

echo "Weekly security maintenance completed"
```

**Monthly security review:**
```bash
#!/bin/bash
# monthly-security-review.sh

echo "Monthly Security Review - $(date)"
echo "==============================="

# 1. Comprehensive security audit
./scripts/comprehensive-security-audit.sh

# 2. Penetration testing (if automated tools available)
echo "Running automated security tests..."
# This would integrate with your security testing tools

# 3. Access right review
echo "Reviewing access rights..."
./scripts/access-rights-review.sh

# 4. Security policy compliance check
echo "Checking security policy compliance..."
./scripts/policy-compliance-check.sh

# 5. Incident response drill (quarterly)
MONTH=$(date +%m)
if [[ $((MONTH % 3)) -eq 1 ]]; then
    echo "Quarterly incident response drill due"
    ./scripts/incident-response-drill.sh
fi

# 6. Generate security metrics report
./scripts/generate-security-metrics.sh monthly

echo "Monthly security review completed"
```

---

## Best Practices by Environment

### Development Environment Security

**Secure development practices:**
- **Separate credentials**: Never use production credentials in development
- **Data masking**: Use masked or synthetic data for development
- **Network isolation**: Isolate development environments from production
- **Secure defaults**: Use secure configuration defaults even in development
- **Regular updates**: Keep development environments updated

**Development security configuration:**
```bash
# Development security configuration (.secrets)
DEBUG_MODE=true
SECRET_KEY=development-secret-change-for-production
LOG_LEVEL=DEBUG

# Use separate development AI provider accounts
VERTEX_AI_PROJECT=dev-project-id
OPENAI_API_KEY=dev-api-key

# Development database with limited data
DATABASE_URL=postgresql://dev_user:dev_password@localhost:4432/simba_dev

# Security features enabled even in development
ENABLE_AUDIT_LOGGING=true
SESSION_COOKIE_SECURE=false  # Only because development uses HTTP
FORCE_HTTPS=false           # Only for development
```

### Production Environment Security

**Maximum security configuration:**
```yaml
# Production security configuration (values-production-secure.yaml)
simba:
  intelligence:
    security:
      # Maximum authentication security
      authentication:
        sessionTimeout: 3600      # 1 hour sessions
        maxConcurrentSessions: 2
        forceHttps: true
        sessionCookieSecure: true
        sessionCookieHttpOnly: true
        csrfProtection: true
        
      # Strict API security
      api:
        rateLimitEnabled: true
        requestsPerMinute: 100
        burstLimit: 20
        requireHttps: true
        validateUserAgent: true
        enableCors: false
        
      # Data protection
      dataProtection:
        enableEncryption: true
        encryptLogs: true
        encryptCache: false  # Performance vs security tradeoff
        sanitizeLogs: true
        
      # Audit and compliance
      audit:
        enableAuditLogging: true
        logLevel: INFO
        retentionDays: 365
        includeQueryDetails: true
        
    # Network security
    networkPolicy:
      enabled: true
      defaultDeny: true
      
    # Pod security
    podSecurityContext:
      runAsNonRoot: true
      runAsUser: 1000
      fsGroup: 2000
      
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop: ["ALL"]

# Ingress security
ingress:
  annotations:
    # Security headers
    nginx.ingress.kubernetes.io/configuration-snippet: |
      add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
      add_header X-Content-Type-Options "nosniff" always;
      add_header X-Frame-Options "DENY" always;
      add_header X-XSS-Protection "1; mode=block" always;
      add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    # Rate limiting
    nginx.ingress.kubernetes.io/rate-limit: "100"
    nginx.ingress.kubernetes.io/rate-limit-window: "1m"
```

---

## Security Testing and Validation

### Security Testing Procedures

#### Automated Security Testing

**Security validation test suite:**
```bash
#!/bin/bash
# security-validation-tests.sh

echo "Security Validation Test Suite"
echo "============================="

PASSED_TESTS=0
TOTAL_TESTS=0

run_security_test() {
    local test_name=$1
    local test_command=$2
    
    ((TOTAL_TESTS++))
    echo -n "Running $test_name... "
    
    if eval "$test_command" &>/dev/null; then
        echo "✅ PASS"
        ((PASSED_TESTS++))
    else
        echo "❌ FAIL"
    fi
}

# Authentication security tests
run_security_test "HTTPS Redirect" \
    "curl -s -I http://your-domain | grep -q '301 Moved Permanently'"

run_security_test "SSL Certificate Valid" \
    "echo | openssl s_client -connect your-domain:443 2>/dev/null | grep -q 'Verify return code: 0'"

run_security_test "Strong TLS Ciphers" \
    "nmap --script ssl-enum-ciphers -p 443 your-domain | grep -q 'TLSv1.2\|TLSv1.3'"

# API security tests
run_security_test "API Authentication Required" \
    "curl -s https://your-domain/api/v1/admin/users | grep -q 'authentication.*required'"

run_security_test "Rate Limiting Active" \
    "for i in {1..110}; do curl -s https://your-domain/api/v1/healthz > /dev/null; done; curl -s https://your-domain/api/v1/healthz | grep -q 'rate.limit'"

# Database security tests
run_security_test "Database SSL Connection" \
    "psql \"$DATABASE_URL\" -c 'SELECT ssl_is_used();' | grep -q 't'"

run_security_test "Database User Permissions" \
    "psql \"$DATABASE_URL\" -c \"SELECT has_database_privilege('simba_user', 'simba_intelligence', 'CREATE');\" | grep -q 'f'"

# Container security tests  
run_security_test "Non-root Containers" \
    "kubectl get pods -n simba-intelligence -o jsonpath='{.items[*].spec.securityContext.runAsNonRoot}' | grep -q 'true'"

run_security_test "Read-only Root Filesystem" \
    "kubectl get pods -n simba-intelligence -o jsonpath='{.items[*].spec.containers[*].securityContext.readOnlyRootFilesystem}' | grep -q 'true'"

# Network security tests
run_security_test "Network Policies Configured" \
    "kubectl get networkpolicy -n simba-intelligence | grep -q 'simba-intelligence'"

run_security_test "Internal Services Not Exposed" \
    "! curl -s http://your-domain:6379 &>/dev/null"

# Summary
echo ""
echo "Security Test Summary:"
echo "Passed: $PASSED_TESTS/$TOTAL_TESTS tests"

if [[ $PASSED_TESTS -eq $TOTAL_TESTS ]]; then
    echo "✅ All security tests passed"
else
    echo "❌ Security hardening incomplete"
    echo "Review failed tests and implement required security measures"
fi
```

### Vulnerability Management

#### Security Scanning and Updates

**Automated vulnerability scanning:**
```bash
#!/bin/bash
# vulnerability-scan.sh

echo "Vulnerability Scanning - $(date)"
echo "==============================="

# 1. Container image scanning
echo "Scanning container images..."
IMAGES=(
    "simba-intelligence:latest"
    "simba-intelligence-job-base:latest"
    "postgres:16-alpine"
    "redis:8.0-alpine"
)

for image in "${IMAGES[@]}"; do
    echo "Scanning $image..."
    trivy image --severity HIGH,CRITICAL --format table $image
done

# 2. Kubernetes configuration scanning
echo ""
echo "Scanning Kubernetes configurations..."
trivy k8s --report summary cluster

# 3. Dependency scanning
echo ""
echo "Scanning Python dependencies..."
safety check --json

# 4. Infrastructure scanning
echo ""
echo "Scanning infrastructure configurations..."
checkov -d . --framework kubernetes

# 5. Generate vulnerability report
VULN_REPORT="/var/log/security/vulnerability-report-$(date +%Y%m%d).txt"
{
    echo "Vulnerability Scan Report - $(date)"
    echo "=================================="
    echo "Scan completed by: $(whoami)"
    echo "Environment: ${ENVIRONMENT:-unknown}"
    echo ""
    
    echo "High/Critical vulnerabilities found:"
    trivy image --severity HIGH,CRITICAL --format json simba-intelligence:latest | \
        jq '.Results[]?.Vulnerabilities[]? | select(.Severity == "HIGH" or .Severity == "CRITICAL") | {ID: .VulnerabilityID, Severity: .Severity, Package: .PkgName}'
        
} > "$VULN_REPORT"

echo "Vulnerability scan completed: $VULN_REPORT"
```

---

## Compliance and Governance

### Regulatory Compliance

#### GDPR Compliance Configuration

**Data privacy and GDPR compliance:**
```bash
# GDPR compliance configuration
ENABLE_GDPR_COMPLIANCE=true
DATA_RETENTION_POLICY=enabled
PERSONAL_DATA_ENCRYPTION=true
RIGHT_TO_DELETION=enabled
DATA_PROCESSING_LOGGING=comprehensive

# Data classification
ENABLE_DATA_CLASSIFICATION=true
CLASSIFY_PERSONAL_DATA=true
CLASSIFY_SENSITIVE_DATA=true

# Consent management
TRACK_DATA_CONSENT=true
CONSENT_WITHDRAWAL_ENABLED=true
```

**GDPR data handling procedures:**
```sql
-- GDPR-compliant data handling
CREATE SCHEMA gdpr_compliance;

-- Data retention tracking
CREATE TABLE gdpr_compliance.data_retention_tracking (
    table_name TEXT,
    record_id TEXT,
    data_category TEXT,  -- personal, sensitive, public
    retention_period INTERVAL,
    created_at TIMESTAMP DEFAULT NOW(),
    expires_at TIMESTAMP,
    deletion_requested BOOLEAN DEFAULT FALSE
);

-- Data processing audit
CREATE TABLE gdpr_compliance.processing_audit (
    processing_id UUID DEFAULT gen_random_uuid(),
    user_id TEXT,
    data_subject_id TEXT,
    processing_purpose TEXT,
    legal_basis TEXT,
    data_accessed JSONB,
    processed_at TIMESTAMP DEFAULT NOW()
);

-- Right to deletion implementation
CREATE FUNCTION gdpr_compliance.request_data_deletion(subject_id TEXT)
RETURNS VOID AS $$
BEGIN
    -- Mark for deletion
    UPDATE gdpr_compliance.data_retention_tracking 
    SET deletion_requested = TRUE 
    WHERE data_subject_id = subject_id;
    
    -- Log deletion request
    INSERT INTO gdpr_compliance.deletion_requests (subject_id, requested_at)
    VALUES (subject_id, NOW());
END;
$$ LANGUAGE plpgsql;
```

#### SOC 2 Compliance

**SOC 2 security controls implementation:**
```yaml
# SOC 2 compliance configuration
soc2_compliance:
  security_controls:
    access_control:
      - principle_of_least_privilege: enforced
      - role_based_access: implemented
      - regular_access_reviews: quarterly
      
    system_monitoring:
      - continuous_monitoring: enabled
      - log_aggregation: centralized
      - anomaly_detection: automated
      - incident_response: documented
      
    data_protection:
      - encryption_at_rest: enabled
      - encryption_in_transit: enforced
      - backup_encryption: enabled
      - data_classification: implemented
      
    change_management:
      - configuration_management: version_controlled
      - change_approval: required
      - testing_procedures: mandatory
      - rollback_procedures: documented
      
    availability:
      - high_availability: configured
      - backup_procedures: automated
      - disaster_recovery: tested
      - monitoring_alerts: comprehensive
```

---

## Security Monitoring and Response

### Threat Detection

#### Security Event Monitoring

**Real-time security monitoring:**
```python
# security_monitor.py - Conceptual security monitoring
import logging
import time
from collections import defaultdict

class SecurityEventMonitor:
    def __init__(self):
        self.failed_auth_attempts = defaultdict(list)
        self.suspicious_activity = defaultdict(list)
        self.admin_actions = []
        
    def monitor_authentication_events(self, log_line):
        """Monitor authentication events for suspicious patterns"""
        if "authentication_failed" in log_line:
            # Extract IP address and timestamp
            ip_address = self.extract_ip(log_line)
            timestamp = self.extract_timestamp(log_line)
            
            self.failed_auth_attempts[ip_address].append(timestamp)
            
            # Check for brute force attacks
            recent_attempts = [
                t for t in self.failed_auth_attempts[ip_address] 
                if timestamp - t < 3600  # Last hour
            ]
            
            if len(recent_attempts) > 10:
                self.trigger_security_alert(
                    "BRUTE_FORCE_DETECTED",
                    f"IP {ip_address} has {len(recent_attempts)} failed auth attempts"
                )
    
    def monitor_api_usage(self, api_request_log):
        """Monitor API usage for abuse patterns"""
        user_id = api_request_log.get('user_id')
        endpoint = api_request_log.get('endpoint')
        timestamp = api_request_log.get('timestamp')
        
        # Track API usage per user
        self.suspicious_activity[user_id].append({
            'endpoint': endpoint,
            'timestamp': timestamp
        })
        
        # Check for unusual usage patterns
        recent_activity = [
            activity for activity in self.suspicious_activity[user_id]
            if timestamp - activity['timestamp'] < 3600  # Last hour
        ]
        
        if len(recent_activity) > 500:  # More than 500 requests/hour
            self.trigger_security_alert(
                "UNUSUAL_API_USAGE",
                f"User {user_id} has {len(recent_activity)} API requests in last hour"
            )
    
    def trigger_security_alert(self, alert_type, message):
        """Trigger security alert and response"""
        alert = {
            'type': alert_type,
            'message': message,
            'timestamp': time.time(),
            'severity': self.classify_severity(alert_type)
        }
        
        # Send to security team
        self.send_security_notification(alert)
        
        # Log security event
        logging.critical(f"SECURITY_ALERT: {alert_type} - {message}")
        
        # Implement automatic response if needed
        if alert_type == "BRUTE_FORCE_DETECTED":
            self.implement_ip_blocking(self.extract_ip_from_message(message))
```

### Security Incident Response

#### Incident Response Automation

**Automated incident response workflow:**
```bash
#!/bin/bash
# automated-incident-response.sh

INCIDENT_TYPE=$1
SEVERITY=${2:-"medium"}
INCIDENT_ID="INC-$(date +%Y%m%d%H%M%S)"

echo "Automated Security Incident Response"
echo "Incident ID: $INCIDENT_ID"
echo "Type: $INCIDENT_TYPE"
echo "Severity: $SEVERITY"

case $SEVERITY in
    "critical")
        # Critical incident - immediate containment
        echo "CRITICAL INCIDENT - IMPLEMENTING IMMEDIATE CONTAINMENT"
        
        # 1. Isolate affected systems
        kubectl scale deployment simba-intelligence-website --replicas=0 -n simba-intelligence
        
        # 2. Notify security team immediately
        curl -X POST "$CRITICAL_SECURITY_WEBHOOK" \
          -H "Content-Type: application/json" \
          -d "{
            \"incident_id\": \"$INCIDENT_ID\",
            \"type\": \"$INCIDENT_TYPE\", 
            \"severity\": \"critical\",
            \"message\": \"Critical security incident detected - system isolated\",
            \"actions_taken\": \"System isolation, forensic collection initiated\"
          }"
        
        # 3. Collect forensic evidence
        ./scripts/collect-forensic-evidence.sh $INCIDENT_ID
        
        # 4. Disable all user sessions
        redis-cli flushdb
        ;;
        
    "high")
        # High severity - enhanced monitoring and partial containment
        echo "HIGH SEVERITY INCIDENT - IMPLEMENTING ENHANCED MONITORING"
        
        # 1. Enable enhanced logging
        kubectl set env deployment/simba-intelligence-website \
          LOG_LEVEL=DEBUG -n simba-intelligence
        
        # 2. Implement temporary access restrictions
        # This could involve temporarily reducing API rate limits
        
        # 3. Notify operations team
        curl -X POST "$OPS_TEAM_WEBHOOK" \
          -d "{\"incident_id\":\"$INCIDENT_ID\",\"severity\":\"high\"}"
        
        # 4. Start enhanced monitoring
        ./scripts/enhanced-monitoring.sh $INCIDENT_ID
        ;;
esac

# Log incident response action
echo "$(date): Incident response completed for $INCIDENT_ID" >> \
  /var/log/security-incident-response.log
```

---

## Best Practices Summary

### Security Implementation Roadmap

#### Phase 1: Basic Security (Week 1)

**Essential security measures:**
- ✅ **HTTPS enforcement**: Configure SSL/TLS certificates
- ✅ **Basic authentication**: Implement strong password policies
- ✅ **Firewall rules**: Configure network access controls
- ✅ **Container security**: Non-root containers and read-only filesystems
- ✅ **Secret management**: Secure credential storage

#### Phase 2: Enhanced Security (Week 2-3)

**Advanced security controls:**
- ✅ **Network policies**: Kubernetes network segmentation
- ✅ **RBAC implementation**: Role-based access control
- ✅ **Audit logging**: Comprehensive activity logging
- ✅ **Database security**: Row-level security and access controls
- ✅ **Monitoring setup**: Security event detection and alerting

#### Phase 3: Operational Security (Week 4+)

**Ongoing security operations:**
- ✅ **Incident response**: Documented procedures and automation
- ✅ **Vulnerability management**: Regular scanning and updates
- ✅ **Compliance reporting**: Automated compliance monitoring
- ✅ **Security training**: Team training on security procedures
- ✅ **Regular audits**: Periodic security assessments

### Security Maintenance Schedule

#### Regular Security Tasks

**Daily security tasks:**
- Monitor security alerts and failed authentication attempts
- Review audit logs for unusual activity
- Check system health and security service status
- Verify backup completion and integrity

**Weekly security tasks:**
- Review user access and permissions
- Analyze security logs for patterns
- Check API key expiration dates
- Update security dashboards and metrics

**Monthly security tasks:**
- Comprehensive access review and cleanup
- Vulnerability scanning and assessment
- Security policy compliance review
- Incident response procedure testing

**Quarterly security tasks:**
- Credential rotation (passwords, API keys, certificates)
- Penetration testing and security assessment
- Security training and awareness programs
- Business continuity and disaster recovery testing

---

*This Security Best Practices guide provides comprehensive security hardening recommendations for Simba Intelligence. Security is an ongoing process that requires regular attention, monitoring, and updates. Implement these practices systematically, starting with basic security measures and progressively adding advanced controls based on your organization's security requirements and risk tolerance.*

*Remember: security is only as strong as its weakest link. Regular review, testing, and improvement of security measures ensures continued protection against evolving threats.*