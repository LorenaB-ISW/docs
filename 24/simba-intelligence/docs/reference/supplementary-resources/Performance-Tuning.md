# Performance Tuning

This document provides optimization recommendations for improving the performance of Simba Intelligence across its various components including AI processing, caching, database operations, and infrastructure deployment.

## Overview

Simba Intelligence is a multi-layered AI-powered data engineering platform that processes natural language queries, manages multiple data sources, and provides real-time responses. Performance optimization focuses on several key areas:

- **AI/LLM Response Times** - Reducing latency through semantic caching
- **Database Performance** - Optimizing PostgreSQL and vector operations  
- **Task Processing** - Efficient background job handling with Celery
- **Caching Strategy** - Multi-level caching with Redis
- **Infrastructure Scaling** - Container resource management and Kubernetes optimization

## AI and LLM Performance Optimization

### Semantic Caching

The most impactful performance optimization is the Semantic Cache System, which dramatically reduces LLM API calls and response times.

**Key Benefits:**
- Reduces response times from seconds to milliseconds for similar queries
- Significantly decreases LLM API costs 
- Improves user experience with near-instant responses for cached content

**Configuration Recommendations:**

```python
# Optimize cache hit rates by configuring appropriate similarity thresholds
SEMANTIC_CACHE_SIMILARITY_THRESHOLD = 0.8  # Adjust based on use case
SEMANTIC_CACHE_MAX_ENTRIES = 10000  # Per user namespace
SEMANTIC_CACHE_TTL = 3600  # 1 hour default expiration
```

**Cache Isolation Strategies:**
- Use item-specific caching for data source queries: `cache.check_get_cache(query, item_id=source_id)`
- Implement global user caching for general queries: `cache.check_get_cache(query)`
- Monitor cache hit rates to optimize similarity thresholds

### LLM Provider Selection

Choose the optimal LLM provider based on performance characteristics:

**For Low Latency:**
- **Google Vertex AI**: Best for embedding generation and fast response times
- Configure location-specific deployments: `location: us-central1` for US users

**For Cost Optimization:**  
- Monitor token usage across providers
- Use smaller models for simple queries
- Implement request batching where possible

**Provider-Specific Optimizations:**

```yaml
# Vertex AI Configuration
vertex_ai:
  location: "us-central1"  # Closest to your users
  model_name: "gemini-2.0-flash"  # Optimized for speed
  temperature: 0.3  # Lower for consistent responses

# Azure OpenAI Configuration  
azure_openai:
  api_version: "2023-05-15"  # Latest stable version
  max_tokens: 1000  # Limit to reduce latency
  temperature: 0.5
```

## Caching Strategy

### Redis Configuration

Redis serves as both the semantic cache backend and general application cache. Optimize Redis for your workload:

**Memory Optimization:**
```conf
# redis.conf optimizations
maxmemory 2gb
maxmemory-policy allkeys-lru
tcp-keepalive 60
timeout 300
```

**Connection Pooling:**
```python
# Configure connection pooling to handle concurrent requests
REDIS_CONNECTION_POOL_SIZE = 20
REDIS_CONNECTION_TIMEOUT = 10
REDIS_SOCKET_KEEPALIVE = True
```

### Multi-Level Caching

Implement caching at multiple application layers:

1. **Semantic Cache** - AI/LLM responses
2. **Query Results Cache** - Database query results  
3. **Session Cache** - User authentication and permissions
4. **Metadata Cache** - Data source schemas and configurations

**Cache Warming Strategy:**
```python
# Pre-populate cache with common queries during low-traffic periods
def warm_cache():
    common_queries = get_popular_queries()
    for query in common_queries:
        if not cache.check_get_cache(query):
            response = generate_response(query)
            cache.update_cache(query, response)
```

## Database Performance

### PostgreSQL Optimization

Simba Intelligence uses PostgreSQL with the pgvector extension for vector similarity search.

**Connection and Memory Settings:**
```conf
# postgresql.conf optimizations
shared_buffers = 256MB          # 25% of RAM
effective_cache_size = 1GB      # 75% of RAM  
work_mem = 4MB                  # Per connection
maintenance_work_mem = 64MB     # For maintenance operations
max_connections = 100           # Adjust based on load
```

**Vector Search Optimization:**
```sql
-- Create appropriate indexes for vector operations
CREATE INDEX CONCURRENTLY idx_embeddings_vector 
ON embeddings USING ivfflat (vector_column vector_cosine_ops) 
WITH (lists = 100);

-- Monitor and optimize vector queries
EXPLAIN (ANALYZE, BUFFERS) 
SELECT * FROM embeddings 
ORDER BY vector_column <=> query_vector 
LIMIT 10;
```

**Query Performance Monitoring:**
```sql
-- Enable slow query logging
log_statement = 'all'
log_min_duration_statement = 1000  -- Log queries > 1 second
log_line_prefix = '%t [%p]: [%l-1] user=%u,db=%d,app=%a,client=%h'
```

### Database Connection Pooling

Use SQLAlchemy connection pooling for optimal database performance:

```python
# Database configuration
SQLALCHEMY_ENGINE_OPTIONS = {
    'pool_size': 10,
    'max_overflow': 20,
    'pool_pre_ping': True,
    'pool_recycle': 3600,
    'connect_args': {
        'connect_timeout': 10,
        'application_name': 'simba-intelligence'
    }
}
```

## Background Task Processing

### Celery Optimization

Celery handles background AI processing and data operations. Optimize for your workload:

**Worker Configuration:**
```python
# celeryconfig.py
broker_url = 'redis://redis:6379/1'
result_backend = 'redis://redis:6379/2'

# Performance settings
task_acks_late = True
worker_prefetch_multiplier = 1  # For memory-intensive tasks
task_compression = 'gzip'
result_compression = 'gzip'

# Concurrency settings
worker_concurrency = 4  # CPU cores
worker_max_tasks_per_child = 1000
worker_disable_rate_limits = False
```

**Task Routing:**
```python
# Route different task types to specialized workers
task_routes = {
    'simba_intelligence.tasks.llm_tasks': {'queue': 'llm_queue'},
    'simba_intelligence.tasks.data_processing': {'queue': 'data_queue'},
    'simba_intelligence.tasks.vector_tasks': {'queue': 'vector_queue'},
}
```

**Task Optimization:**
```python
# Use task batching for efficiency
@celery_app.task(bind=True)
def process_batch_embeddings(self, texts_batch):
    """Process multiple texts in a single task to reduce overhead"""
    embeddings = []
    for text in texts_batch:
        embedding = generate_embedding(text)
        embeddings.append(embedding)
    return embeddings
```

## Infrastructure and Deployment Optimization

### Container Resource Management

**Memory Allocation:**
```yaml
# Docker Compose resource limits
services:
  main-app:
    mem_limit: 2g
    memswap_limit: 2g
    
  celery-worker:
    mem_limit: 1g
    deploy:
      replicas: 3
      
  redis:
    mem_limit: 512m
    
  postgres-main:
    mem_limit: 1g
    shm_size: 256m  # Important for PostgreSQL
```

**CPU Optimization:**
```yaml
# CPU limits and requests
resources:
  requests:
    cpu: 100m
    memory: 256Mi
  limits:
    cpu: 500m
    memory: 1Gi
```

### Kubernetes Scaling

**Horizontal Pod Autoscaler (HPA):**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: simba-intelligence-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: simba-intelligence-website
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

**Pod Disruption Budget:**
```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: simba-intelligence-pdb
spec:
  minAvailable: 50%
  selector:
    matchLabels:
      app: simba-intelligence
```

### Load Balancer Configuration

**GKE Backend Configuration:**
```yaml
apiVersion: cloud.google.com/v1
kind: BackendConfig
metadata:
  name: simba-intelligence-backend-config
spec:
  timeoutSec: 1800
  connectionDraining:
    drainingTimeoutSec: 1800
  healthCheck:
    checkIntervalSec: 10
    timeoutSec: 5
    healthyThreshold: 1
    unhealthyThreshold: 3
```

## Monitoring and Performance Metrics

### Key Performance Indicators

Monitor these critical metrics for optimal performance:

**Application Metrics:**
- AI/LLM response times and cache hit rates
- Database query execution times
- Celery task queue lengths and processing times
- Memory and CPU utilization per container

**Business Metrics:**
- Average query resolution time
- User satisfaction scores from rating system
- Data source connection success rates
- Query success rates

### Health Checks and Monitoring

**Application Health Endpoints:**
```python
@app.route('/api/v1/healthz')
def health_check():
    checks = {
        'database': check_database_connection(),
        'redis': check_redis_connection(),
        'llm_providers': check_llm_providers(),
        'celery': check_celery_workers()
    }
    
    if all(checks.values()):
        return {'status': 'healthy', 'checks': checks}, 200
    else:
        return {'status': 'unhealthy', 'checks': checks}, 503
```

**Kubernetes Health Checks:**
```yaml
livenessProbe:
  httpGet:
    path: /api/v1/healthz
    port: 5050
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 5

readinessProbe:
  httpGet:
    path: /api/v1/ready
    port: 5050
  initialDelaySeconds: 5
  periodSeconds: 5
  timeoutSeconds: 3
```

## Troubleshooting Common Performance Issues

### High Memory Usage

**Symptoms:**
- Container restarts due to OOM kills
- Slow response times
- High swap usage

**Solutions:**
```bash
# Check memory usage patterns
docker stats --no-stream
kubectl top pods

# Analyze Python memory usage
python -m memory_profiler your_script.py

# Optimize garbage collection
export PYTHONMALLOC=malloc
export MALLOC_ARENA_MAX=2
```

### Slow Query Performance

**Symptoms:**
- Database connection pool exhaustion
- High query execution times
- User timeout errors

**Diagnosis:**
```sql
-- Check for slow queries
SELECT query, mean_exec_time, calls, total_exec_time 
FROM pg_stat_statements 
ORDER BY mean_exec_time DESC 
LIMIT 10;

-- Check for missing indexes
SELECT schemaname, tablename, attname 
FROM pg_stats 
WHERE schemaname NOT IN ('information_schema', 'pg_catalog') 
  AND n_distinct > 100 
  AND correlation < 0.1;
```

### Cache Performance Issues

**Low Cache Hit Rates:**
```python
# Monitor cache statistics
def get_cache_stats():
    info = redis_client.info('memory')
    keyspace = redis_client.info('keyspace')
    
    return {
        'memory_usage': info['used_memory_human'],
        'hit_rate': calculate_hit_rate(),
        'key_count': keyspace.get('db0', {}).get('keys', 0)
    }
```

**Cache Eviction Problems:**
```python
# Implement intelligent cache eviction
def smart_cache_cleanup():
    # Remove expired entries first
    expired_keys = redis_client.scan_iter(match="cache:*")
    for key in expired_keys:
        if redis_client.ttl(key) <= 0:
            redis_client.delete(key)
    
    # Remove least recently used entries if memory pressure
    if get_memory_usage() > MEMORY_THRESHOLD:
        lru_keys = get_lru_keys(limit=100)
        redis_client.delete(*lru_keys)
```

## Best Practices Summary

1. **Enable Semantic Caching** - Implement for all LLM interactions to achieve 10x performance improvements
2. **Monitor Resource Usage** - Set up comprehensive monitoring for proactive optimization
3. **Scale Horizontally** - Use Kubernetes HPA to handle variable workloads
4. **Optimize Database Queries** - Regular query analysis and index optimization
5. **Implement Circuit Breakers** - Prevent cascade failures during high load
6. **Use Connection Pooling** - For all external service connections
7. **Regular Performance Testing** - Load test major releases and configuration changes
8. **Cache Warm-up** - Pre-populate caches during deployment
9. **Graceful Degradation** - Implement fallbacks for service unavailability
10. **Capacity Planning** - Monitor growth trends and scale infrastructure proactively

By following these performance optimization strategies, you can significantly improve Simba Intelligence's response times, reduce infrastructure costs, and provide a better user experience for data engineers and analysts using the platform.