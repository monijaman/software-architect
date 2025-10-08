# 📈 Scalability Patterns

Master the patterns and techniques for building systems that can grow to serve millions of users.

## Table of Contents
- [Horizontal vs Vertical Scaling](#horizontal-vs-vertical-scaling)
- [Caching Strategies](#caching-strategies)
- [Load Balancing](#load-balancing)
- [Database Scaling](#database-scaling)
- [Content Delivery Networks (CDN)](#content-delivery-networks-cdn)
- [Asynchronous Processing](#asynchronous-processing)
- [Auto-Scaling](#auto-scaling)
- [Performance Optimization](#performance-optimization)

---

## Horizontal vs Vertical Scaling

### Vertical Scaling (Scale Up)
**Definition**: Adding more power to existing machines (CPU, RAM, storage)

**Advantages**:
- Simpler implementation - no code changes required
- No data partitioning complexity
- Strong consistency guarantees
- Lower operational complexity

**Disadvantages**:
- Hardware limits (maximum capacity per machine)
- Single point of failure
- Expensive (high-end hardware costs more per unit)
- Downtime required for upgrades

**When to Use**:
- Small to medium applications
- Relational databases with complex queries
- Applications requiring strong consistency
- Limited budget or time for architectural changes

**Example**:
```
Before: 4 CPU cores, 16GB RAM
After:  16 CPU cores, 64GB RAM
```

### Horizontal Scaling (Scale Out)
**Definition**: Adding more machines to the resource pool

**Advantages**:
- Nearly unlimited scaling potential
- Fault tolerance (multiple machines)
- Cost-effective (commodity hardware)
- Geographical distribution possible

**Disadvantages**:
- Complex implementation (data partitioning, consistency)
- Network latency and failures
- Load balancing complexity
- Operational overhead

**When to Use**:
- Large-scale applications
- High availability requirements
- Geographically distributed users
- Cost-sensitive applications

**Example**:
```
Before: 1 server handling 1000 RPS
After:  5 servers each handling 200 RPS
```

---

## Caching Strategies

### Cache Levels

#### 1. **Browser Cache**
- Client-side caching
- HTTP cache headers (Cache-Control, ETag)
- Reduces server load and improves user experience

#### 2. **CDN (Content Delivery Network)**
- Geographic distribution of content
- Cache static assets (images, CSS, JS)
- Reduce latency and bandwidth costs

#### 3. **Reverse Proxy Cache**
- Server-side caching (Varnish, NGINX)
- Cache full HTTP responses
- Reduce load on application servers

#### 4. **Application Cache**
- In-memory caching (Redis, Memcached)
- Cache database queries, computed results
- Sub-millisecond response times

#### 5. **Database Cache**
- Query result caching
- Buffer pools for frequently accessed pages
- Reduce disk I/O

### Caching Patterns

#### Cache-Aside (Lazy Loading)
```python
def get_user(user_id):
    # Check cache first
    user = cache.get(f"user:{user_id}")
    if user is None:
        # Cache miss - load from database
        user = database.get_user(user_id)
        # Store in cache
        cache.set(f"user:{user_id}", user, ttl=3600)
    return user
```

**Pros**: Only cache what's needed, cache failures don't affect application
**Cons**: Cache miss penalty, potential for stale data

#### Write-Through
```python
def update_user(user_id, data):
    # Write to database first
    database.update_user(user_id, data)
    # Then update cache
    cache.set(f"user:{user_id}", data, ttl=3600)
```

**Pros**: Cache always consistent, no cache miss penalty for writes
**Cons**: Higher write latency, cache may store unused data

#### Write-Behind (Write-Back)
```python
def update_user(user_id, data):
    # Write to cache immediately
    cache.set(f"user:{user_id}", data, ttl=3600)
    # Queue database write for later
    write_queue.enqueue(user_id, data)
```

**Pros**: Lower write latency, better write throughput
**Cons**: Risk of data loss, complexity in handling failures

#### Refresh-Ahead
```python
def get_user(user_id):
    user = cache.get(f"user:{user_id}")
    # Refresh cache proactively before expiration
    if cache.ttl(f"user:{user_id}") < REFRESH_THRESHOLD:
        background_task.refresh_cache(user_id)
    return user
```

**Pros**: No cache miss penalty, always fresh data
**Cons**: Complexity, may refresh unused data

### Cache Invalidation Strategies

#### Time-Based (TTL)
- Set expiration time for cached data
- Simple but may serve stale data

#### Event-Based
- Invalidate cache when source data changes
- More complex but ensures data freshness

#### Tag-Based
- Tag cache entries with related identifiers
- Invalidate all entries with specific tags

---

## Load Balancing

### Load Balancing Algorithms

#### Round Robin
```python
servers = ["server1", "server2", "server3"]
current = 0

def get_server():
    global current
    server = servers[current]
    current = (current + 1) % len(servers)
    return server
```

#### Weighted Round Robin
```python
servers = [
    ("server1", 3),  # Weight 3
    ("server2", 2),  # Weight 2
    ("server3", 1)   # Weight 1
]

def get_server():
    # Select server based on weights
    total_weight = sum(weight for _, weight in servers)
    random_value = random.randint(1, total_weight)
    # ... implementation
```

#### Least Connections
- Route to server with fewest active connections
- Good for long-lived connections
- Requires connection tracking

#### IP Hash
```python
def get_server(client_ip):
    hash_value = hash(client_ip)
    return servers[hash_value % len(servers)]
```

#### Geographic Proximity
- Route based on client location
- Minimize network latency
- Requires geolocation services

### Load Balancer Types

#### Layer 4 (Transport Layer)
- Routes based on IP address and port
- Fast and efficient
- Cannot inspect application data
- Examples: AWS Network Load Balancer, HAProxy

#### Layer 7 (Application Layer)
- Routes based on HTTP headers, URLs, cookies
- More flexible routing rules
- Higher overhead
- Examples: AWS Application Load Balancer, NGINX

---

## Database Scaling

### Read Replicas
```
                    Write
Client ───────────► Master DB
  │                     │
  │                     │ Replication
  │                     ▼
  └──── Read ──────► Slave DB 1
                    Slave DB 2
                    Slave DB 3
```

**Benefits**:
- Scale read operations
- Geographical distribution
- Backup and disaster recovery

**Considerations**:
- Replication lag (eventual consistency)
- Master failure handling
- Connection routing logic

### Database Sharding (Horizontal Partitioning)

#### Shard Key Selection
```python
# Hash-based sharding
def get_shard(user_id):
    return hash(user_id) % num_shards

# Range-based sharding
def get_shard(user_id):
    if user_id < 1000000:
        return "shard1"
    elif user_id < 2000000:
        return "shard2"
    # ... etc
```

**Strategies**:
1. **Hash-based**: Even distribution but no range queries
2. **Range-based**: Range queries possible but potential hotspots
3. **Directory-based**: Lookup service maps keys to shards

**Challenges**:
- Cross-shard queries
- Rebalancing shards
- Transaction management
- Joins across shards

### Database Federation (Functional Partitioning)
```
Users Service ─────► Users DB
Orders Service ────► Orders DB
Products Service ──► Products DB
```

**Benefits**:
- Clear service boundaries
- Independent scaling
- Technology diversity

**Challenges**:
- Cross-database transactions
- Data consistency
- Shared reference data

---

## Content Delivery Networks (CDN)

### CDN Architecture
```
User Request → Edge Server (Cache) → Origin Server
                    │
                    └─ Cache Hit: Serve from edge
                    └─ Cache Miss: Fetch from origin
```

### CDN Strategies

#### Push CDN
- Content uploaded to CDN in advance
- Good for sites with low traffic or infrequent updates
- Control over content timing

#### Pull CDN
- Content cached when first requested
- Good for sites with heavy traffic
- Less storage space used

### CDN Optimization
- **Cache Headers**: Set appropriate TTL values
- **Compression**: Gzip, Brotli compression
- **Image Optimization**: WebP format, responsive images
- **HTTP/2**: Multiplexing, server push

---

## Asynchronous Processing

### Message Queue Patterns

#### Work Queue
```
Producer → Queue → Worker 1
                → Worker 2
                → Worker 3
```

#### Publish/Subscribe
```
Publisher → Topic → Subscriber 1
                 → Subscriber 2
                 → Subscriber 3
```

### Task Processing
```python
# Celery example
from celery import Celery

app = Celery('tasks', broker='redis://localhost:6379')

@app.task
def process_image(image_id):
    # Heavy processing task
    image = load_image(image_id)
    processed = resize_and_optimize(image)
    save_processed_image(processed)
    return f"Processed image {image_id}"

# Async call
result = process_image.delay(image_id)
```

### Event-Driven Architecture
```python
# Event publishing
event_bus.publish("user.created", {
    "user_id": 123,
    "email": "user@example.com",
    "timestamp": datetime.now()
})

# Event handling
@event_handler("user.created")
def send_welcome_email(event):
    email_service.send_welcome_email(event.data["email"])

@event_handler("user.created")
def create_user_profile(event):
    profile_service.create_profile(event.data["user_id"])
```

---

## Auto-Scaling

### Metrics-Based Scaling
```yaml
# Kubernetes HPA example
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app
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

### Predictive Scaling
- Use historical data to predict load
- Scale proactively before demand spikes
- Machine learning-based forecasting

### Scheduled Scaling
- Scale based on known patterns
- Business hours vs off-hours
- Seasonal traffic variations

---

## Performance Optimization

### Application-Level Optimizations

#### Connection Pooling
```python
# Database connection pool
from sqlalchemy import create_engine
from sqlalchemy.pool import QueuePool

engine = create_engine(
    'postgresql://user:pass@localhost/db',
    poolclass=QueuePool,
    pool_size=20,
    max_overflow=30,
    pool_pre_ping=True
)
```

#### Batch Processing
```python
# Batch database operations
def bulk_insert_users(users):
    # Instead of inserting one by one
    # Insert in batches of 1000
    batch_size = 1000
    for i in range(0, len(users), batch_size):
        batch = users[i:i + batch_size]
        db.bulk_insert(batch)
```

#### Lazy Loading
```python
# Load data only when needed
class User:
    def __init__(self, user_id):
        self.user_id = user_id
        self._profile = None
    
    @property
    def profile(self):
        if self._profile is None:
            self._profile = load_user_profile(self.user_id)
        return self._profile
```

### Database Optimizations
- **Indexing**: Create indexes for frequently queried columns
- **Query Optimization**: Use EXPLAIN to analyze query plans
- **Denormalization**: Trade storage for read performance
- **Partitioning**: Split large tables into smaller chunks

### Network Optimizations
- **HTTP/2**: Multiplexing, header compression
- **Keep-Alive**: Reuse TCP connections
- **Compression**: Gzip responses
- **Minification**: Reduce payload sizes

---

## Scalability Testing

### Load Testing Tools
- **Apache JMeter**: GUI-based load testing
- **Artillery**: Lightweight, scriptable
- **k6**: Developer-centric performance testing
- **Gatling**: High-performance load testing

### Testing Strategies
1. **Baseline Testing**: Establish normal performance
2. **Load Testing**: Test under expected load
3. **Stress Testing**: Test beyond normal capacity
4. **Spike Testing**: Test sudden load increases
5. **Volume Testing**: Test with large amounts of data
6. **Endurance Testing**: Test over extended periods

### Monitoring and Metrics
- **Response Time**: 95th, 99th percentile latency
- **Throughput**: Requests per second
- **Error Rate**: Percentage of failed requests
- **Resource Utilization**: CPU, memory, disk, network
- **Saturation**: Queue lengths, connection pools

---

## Best Practices

### Design for Scalability
1. **Stateless Services**: Store state externally
2. **Horizontal Scaling**: Design for scale-out from day one
3. **Async Processing**: Use queues for heavy operations
4. **Cache Everything**: Cache at multiple levels
5. **Database Optimization**: Proper indexing and query optimization

### Monitoring and Observability
1. **Comprehensive Metrics**: Track all important KPIs
2. **Alerting**: Proactive notifications for issues
3. **Distributed Tracing**: Track requests across services
4. **Log Aggregation**: Centralized logging
5. **Performance Profiling**: Regular performance analysis

### Operational Excellence
1. **Capacity Planning**: Forecast growth and plan resources
2. **Disaster Recovery**: Test backup and recovery procedures
3. **Security**: Implement security at scale
4. **Cost Optimization**: Monitor and optimize cloud costs
5. **Documentation**: Keep architecture and procedures documented

---

## Real-World Case Studies

### Twitter's Scaling Journey
- **2006**: Ruby on Rails monolith
- **2008**: Message queue for timeline generation
- **2010**: Scala services, custom MySQL sharding
- **2012**: Manhattan (distributed database)
- **2015**: Microservices architecture

### Instagram's Scaling
- **2010**: Django + PostgreSQL
- **2011**: Moved photos to Amazon S3
- **2012**: Sharded PostgreSQL
- **2014**: Cassandra for user feeds
- **2016**: Machine learning for feed ranking

### Netflix's Architecture Evolution
- **2008**: Monolithic datacenter-based architecture
- **2009**: Started AWS migration
- **2012**: Microservices architecture
- **2016**: 700+ microservices
- **2020**: Global edge computing with Open Connect

---

*Return to [System Design Overview](./README.md) or [Core Architectural Skills](../README.md)*