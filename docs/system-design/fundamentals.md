# System Design Fundamentals

## Overview
System design is the process of defining the architecture, components, modules, interfaces, and data for a system to satisfy specified requirements. This guide covers fundamental concepts for designing scalable, reliable, and maintainable systems.

## Table of Contents
1. [Scalability](#scalability)
2. [Availability](#availability)
3. [Performance](#performance)
4. [Reliability](#reliability)
5. [Security](#security)
6. [Data Management](#data-management)

## Scalability

### Definition
The ability of a system to handle growing amounts of work by adding resources.

### Horizontal Scaling (Scale Out)
- Add more machines to the pool
- Distribute load across multiple servers
- Better fault tolerance
- Potentially unlimited scaling
- More complex to implement

**Example**: Adding more web servers behind a load balancer

### Vertical Scaling (Scale Up)
- Add more resources to existing machine
- Increase CPU, RAM, disk
- Simpler to implement
- Limited by hardware capacity
- Single point of failure

**Example**: Upgrading server from 8GB to 32GB RAM

### Load Balancing

#### Algorithms
1. **Round Robin**: Distribute requests evenly in rotation
2. **Least Connections**: Send to server with fewest active connections
3. **IP Hash**: Use client IP to determine server
4. **Weighted**: Assign different weights to servers based on capacity

#### Types
- **Layer 4 (Transport)**: Route based on IP and port
- **Layer 7 (Application)**: Route based on content (URL, headers)

#### Popular Solutions
- HAProxy
- NGINX
- AWS Elastic Load Balancer
- Google Cloud Load Balancing

### Caching

#### Cache Strategies

**1. Cache-Aside (Lazy Loading)**
```
Application checks cache → Miss → Query DB → Update cache
```
- Simple to implement
- Cache populated on demand
- Stale data possible

**2. Write-Through**
```
Write to cache → Write to DB simultaneously
```
- Always consistent
- Write latency increases
- Unnecessary cache writes

**3. Write-Back (Write-Behind)**
```
Write to cache → Async write to DB later
```
- Fast writes
- Risk of data loss
- Complex implementation

**4. Refresh-Ahead**
```
Automatically refresh cache before expiration
```
- Reduced latency
- Requires prediction of what to refresh

#### Cache Eviction Policies
- **LRU (Least Recently Used)**: Remove least recently accessed items
- **LFU (Least Frequently Used)**: Remove least frequently accessed items
- **FIFO (First In First Out)**: Remove oldest items
- **TTL (Time To Live)**: Remove items after time expires

#### Caching Layers
1. **Client-Side**: Browser cache, mobile app cache
2. **CDN**: Content Delivery Network for static assets
3. **Application**: In-memory cache (Redis, Memcached)
4. **Database**: Query cache, index cache

## Availability

### Definition
The proportion of time a system is operational and accessible.

### Measuring Availability
```
Availability = Uptime / (Uptime + Downtime)
```

#### Common SLA Targets
- **99% (Two Nines)**: ~3.65 days downtime/year
- **99.9% (Three Nines)**: ~8.76 hours downtime/year
- **99.99% (Four Nines)**: ~52.6 minutes downtime/year
- **99.999% (Five Nines)**: ~5.26 minutes downtime/year

### High Availability Patterns

#### Redundancy
- **Active-Active**: All nodes handle traffic
- **Active-Passive**: Backup nodes on standby
- **N+1 Redundancy**: One extra node for failover

#### Replication
- **Master-Slave**: One master, multiple read replicas
- **Master-Master**: Multiple masters, bidirectional replication
- **Quorum-Based**: Majority consensus required

#### Failover
- **Automatic Failover**: System detects failure and switches
- **Manual Failover**: Human intervention required
- **Health Checks**: Regular monitoring of component health

### CAP Theorem
A distributed system can provide only two of three guarantees:

1. **Consistency**: All nodes see the same data at the same time
2. **Availability**: Every request receives a response
3. **Partition Tolerance**: System continues despite network partitions

**Trade-offs**:
- **CP Systems**: Consistency + Partition Tolerance (e.g., MongoDB, HBase)
- **AP Systems**: Availability + Partition Tolerance (e.g., Cassandra, DynamoDB)
- **CA Systems**: Consistency + Availability (only possible without partitions)

### PACELC Theorem
Extension of CAP:
- **If Partition (P)**: Choose between Availability (A) and Consistency (C)
- **Else (E)**: Choose between Latency (L) and Consistency (C)

## Performance

### Metrics

#### Latency
- Time to process a single request
- Measured in milliseconds
- Target: < 100ms for interactive applications

#### Throughput
- Number of requests processed per unit time
- Measured in requests/second (RPS)
- Target: Depends on application scale

#### Response Time Percentiles
- **P50 (Median)**: 50% of requests faster
- **P95**: 95% of requests faster
- **P99**: 99% of requests faster
- **P99.9**: 99.9% of requests faster

### Optimization Techniques

#### Database Optimization
1. **Indexing**: Speed up queries
2. **Query Optimization**: Efficient SQL queries
3. **Connection Pooling**: Reuse database connections
4. **Denormalization**: Trade storage for query speed
5. **Partitioning**: Split data across multiple databases

#### Application Optimization
1. **Asynchronous Processing**: Non-blocking operations
2. **Batch Processing**: Process multiple items together
3. **Lazy Loading**: Load data only when needed
4. **Compression**: Reduce data transfer size
5. **Parallelization**: Use multiple threads/processes

#### Network Optimization
1. **CDN**: Serve static content from edge locations
2. **HTTP/2**: Multiplexing, header compression
3. **Compression**: Gzip, Brotli
4. **Keep-Alive**: Reuse TCP connections
5. **DNS Optimization**: Reduce DNS lookup time

## Reliability

### Definition
The probability that a system will perform correctly over a specific period.

### Patterns for Reliability

#### Circuit Breaker
Prevents cascading failures by stopping requests to failing services.

**States**:
1. **Closed**: Normal operation, requests pass through
2. **Open**: Too many failures, requests fail immediately
3. **Half-Open**: Test if service recovered

#### Retry Pattern
Automatically retry failed operations.

**Best Practices**:
- Exponential backoff
- Maximum retry limit
- Idempotent operations only
- Jitter to avoid thundering herd

#### Bulkhead Pattern
Isolate resources to prevent total failure.

**Example**: Separate thread pools for different operations

#### Timeout Pattern
Set maximum time to wait for operation.

**Best Practices**:
- Set realistic timeouts
- Different timeouts for different operations
- Consider cascading timeouts

### Monitoring and Alerting

#### Key Metrics
1. **System Metrics**: CPU, memory, disk, network
2. **Application Metrics**: Request rate, error rate, latency
3. **Business Metrics**: User signups, transactions, revenue

#### Monitoring Tools
- **Prometheus**: Time-series database and monitoring
- **Grafana**: Visualization and dashboards
- **ELK Stack**: Logging (Elasticsearch, Logstash, Kibana)
- **Datadog**: Full-stack monitoring platform

#### Alerting Best Practices
- Alert on symptoms, not causes
- Actionable alerts only
- Clear escalation paths
- Avoid alert fatigue

## Security

### Authentication
- Verify identity of users/systems
- Methods: Passwords, MFA, OAuth, SAML

### Authorization
- Determine what authenticated users can do
- RBAC (Role-Based Access Control)
- ABAC (Attribute-Based Access Control)

### Encryption
- **At Rest**: Encrypt stored data
- **In Transit**: Use TLS/SSL
- Key management and rotation

### Common Vulnerabilities
1. **SQL Injection**: Sanitize inputs, use parameterized queries
2. **XSS**: Escape output, Content Security Policy
3. **CSRF**: Use tokens, SameSite cookies
4. **DDoS**: Rate limiting, CDN, WAF

### Security Best Practices
- Principle of least privilege
- Defense in depth
- Regular security audits
- Keep dependencies updated
- Input validation
- Secure coding practices

## Data Management

### Database Types

#### Relational (SQL)
- Structured data with relationships
- ACID transactions
- Examples: PostgreSQL, MySQL, Oracle

#### NoSQL

**Document Stores**
- JSON-like documents
- Examples: MongoDB, CouchDB
- Use case: Flexible schemas, hierarchical data

**Key-Value Stores**
- Simple key-value pairs
- Examples: Redis, DynamoDB
- Use case: Caching, session storage

**Column-Family Stores**
- Wide-column storage
- Examples: Cassandra, HBase
- Use case: Time-series data, high write throughput

**Graph Databases**
- Nodes and relationships
- Examples: Neo4j, Amazon Neptune
- Use case: Social networks, recommendation engines

### Data Partitioning (Sharding)

#### Strategies
1. **Range-Based**: Partition by range of values
2. **Hash-Based**: Hash function determines partition
3. **Directory-Based**: Lookup table for routing

#### Considerations
- Balanced distribution
- Avoiding hot spots
- Rebalancing strategy
- Cross-shard queries

### Data Replication

#### Purposes
- High availability
- Read scaling
- Disaster recovery
- Geographic distribution

#### Methods
- **Synchronous**: Wait for replicas to confirm
- **Asynchronous**: Don't wait for replicas
- **Semi-Synchronous**: Wait for some replicas

### Consistency Models

1. **Strong Consistency**: All reads see latest write
2. **Eventual Consistency**: Reads eventually see latest write
3. **Causal Consistency**: Related operations ordered
4. **Read-Your-Writes**: Users see their own writes

## Design Process

### 1. Requirements Gathering
- Functional requirements
- Non-functional requirements (scale, performance)
- Constraints (time, budget, technology)

### 2. Estimation
- Traffic estimates
- Storage estimates
- Bandwidth estimates

### 3. High-Level Design
- System components
- Communication between components
- Data flow

### 4. Detailed Design
- Database schema
- API design
- Algorithm details
- Technology choices

### 5. Bottlenecks and Trade-offs
- Identify potential issues
- Discuss trade-offs
- Plan for scale

## References
- [Designing Data-Intensive Applications by Martin Kleppmann](https://dataintensive.net/)
- [System Design Primer](https://github.com/donnemartin/system-design-primer)
- [High Scalability Blog](http://highscalability.com/)
