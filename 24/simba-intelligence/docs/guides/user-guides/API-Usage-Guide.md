# API Usage Guide

This guide explains how to use the Simba Intelligence REST API for natural language querying, data source related operations, and administrative integration tasks. It keeps the same core details as the original reference while aligning with end‑user integration usage.

## Overview

### API Architecture

The Simba Intelligence API follows RESTful principles with JSON request/response formats and supports both synchronous and streaming (Server-Sent Events) interactions for real-time query processing.

**🔗 Base URL Structure:**
```
https://your-domain/api/v1/
```

**📡 Communication Patterns:**
- **Standard REST**: Request/response for configuration and management
- **Server-Sent Events (SSE)**: Real-time streaming for query execution
- **WebSocket**: Real-time updates and notifications (where supported)

**🔐 Authentication Methods:**
- **Bearer Tokens**: Short-lived tokens for web application sessions
- **API Keys**: Long-lived keys for programmatic access
- **Role-Based Access**: Different endpoints require different permission levels

### API Documentation

Interactive API documentation (Swagger) is available in your deployment and can also be reached from the API Keys page:
```
https://your-domain/apidocs/
```

---

## Authentication

### API Key Authentication

#### Obtaining API Keys

**Create personal API key:**
```bash
# Through web interface
1. Login to Simba Intelligence
2. Navigate to user menu → "API Keys"
3. Click "Create API Key"
4. Copy generated key
```

**Using API keys in requests:**
```bash
curl -X POST https://your-domain/api/v1/data/query \
  -H "Authorization: Bearer your-api-key-here" \
  -H "Content-Type: application/json"
```

---

## Best Practices

### API Integration Best Practices

**Authentication security:**
- **Secure storage**: Never hardcode API keys in source code
- **Environment variables**: Use environment variables for key storage
- **Key rotation**: Regularly rotate API keys according to security policies
- **Scope limitation**: Use keys with minimum necessary permissions

**Performance considerations:**
- **Pagination**: Use pagination for large result sets
- **Streaming**: Use streaming endpoints for real-time results
- **Compression**: Enable gzip compression for large responses
- **Connection pooling**: Reuse HTTP connections for multiple requests

### Development Workflow

**API development workflow:**
1. **Test in development**: Use development environment for initial integration
2. **Error handling**: Implement comprehensive error handling for all scenarios
3. **Logging**: Log API interactions for debugging and monitoring
4. **Validation**: Validate API responses and handle schema changes
5. **Monitoring**: Monitor API usage and performance in production

---

*This API Usage Guide provides essential documentation for integrating programmatically with Simba Intelligence. For additional examples, interactive testing, and detailed schemas, visit the built-in API documentation at `https://your-domain/api/docs` when Simba Intelligence is running.*
