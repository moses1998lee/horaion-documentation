# - Performance Requirements

> **Horaion Workforce Management Platform - Performance Requirements**



---

## 10.1 Response Time {#id-10.1-response-time}

### Target Response Times (Established)

| Operation Type | Target (P95) | Target (P99) | Notes |
|----------------|--------------|--------------|-------|
| Simple GET | < 100ms | < 200ms | Single entity lookup |
| List GET | < 200ms | < 400ms | Paginated list (20 items) |
| POST/PUT | < 300ms | < 500ms | Create/update with validation |
| DELETE | < 150ms | < 300ms | Soft delete operation |
| Schedule Generation | 15-30 min | 45 min | External engine dependent |

### Known Performance Characteristics

#### Schedule Generation
- **Duration**: 15-30 minutes (typical)
- **Max Timeout**: 45 minutes (Feign timeout)
- **Async**: Yes (non-blocking)
- **Dependency**: External optimization engine

#### Cognito Operations
- **Account Creation**: 1-5 minutes (bulk async)
- **Authentication**: < 1 second (AWS Cognito)
- **Token Validation**: < 50ms (local JWKS validation)

### Performance Testing Framework

The following tools and scenarios are recommended for future validated performance benchmarking:

**Recommended Tools**:
- Apache JMeter
- Gatling
- k6
- Artillery

**Test Scenarios**:
1. **Baseline**: Single user, all endpoints
2. **Concurrent Users**: 10, 50, 100, 500 users
3. **Sustained Load**: 1 hour at target load
4. **Spike Test**: Sudden traffic increase
5. **Stress Test**: Find breaking point

---

## 10.2 Scalability {#id-10.2-scalability}

### Current Configuration

#### Database Connection Pool (HikariCP)
- **Max Connections**: 10
- **Min Idle**: 10
- **Connection Timeout**: 30 seconds

#### Thread Pools

**Schedule Task Executor**:
- **Core Threads**: 5
- **Max Threads**: 20
- **Queue Capacity**: 100 tasks

**General Task Executor**:
- **Core Threads**: 3
- **Max Threads**: 10
- **Queue Capacity**: 50 tasks

#### Tomcat Connector
- **Max Connections**: 8,192
- **Max Threads**: 200
- **Accept Queue**: 100

### Scalability Considerations

**Future Definition Areas**:
- Expected concurrent users
- Expected requests per second
- Expected data growth rate
- Scaling strategy (vertical vs. horizontal)

### Horizontal Scaling Readiness

**Stateless Design**: ✅ Yes
- No server-side sessions
- JWT tokens for authentication
- Database for persistent state

**Load Balancer Ready**: ✅ Yes
- Health check endpoint: `/actuator/health`
- Graceful shutdown support
- No sticky sessions required

**Database Scaling**:
- Read replicas configuration
- Connection pool sizing for multiple instances
- Database partitioning strategy

---

## 10.3 Monitoring & Logging {#id-10.3-monitoring-logging}

### Current Monitoring

#### Spring Boot Actuator
- **Endpoint**: `/actuator/health`
- **Metrics**: `/actuator/metrics`
- **Info**: `/actuator/info`

#### HikariCP Metrics
- `hikaricp.connections.active`
- `hikaricp.connections.idle`
- `hikaricp.connections.pending`
- `hikaricp.connections.timeout`

#### Application Logging
- **Framework**: SLF4J + Logback
- **MDC Fields**: `requestId`, `userId`
- **Log Levels**: INFO, WARN, ERROR
- **Aspect Logging**: All controller methods logged

### External Monitoring Strategy

**Future Configuration Areas**:
- Prometheus (metrics collection)
- Grafana (dashboards)
- ELK Stack (log aggregation)
- APM tools (e.g., New Relic, Datadog)
- Alert configuration

### Logging Strategy

#### What to Log
- ✅ All API requests (via LogAspect)
- ✅ Authentication attempts
- ✅ Schedule generation start/end
- ✅ Cognito account creation results
- ✅ External API calls (Feign)
- ⚠️ Database query performance?
- ⚠️ Business events?

#### Log Retention
#### Log Retention Policy
- Retention period to be defined (e.g., 30 days hot, 1 year cold)
- Log storage location (e.g., S3, CloudWatch Logs)
- Log rotation strategy

---

## 10.4 Performance Optimization Checklist {#id-10.4-performance-optimization-checklist}

### Database ✅ Implemented
- [x] HikariCP connection pooling
- [x] PostgreSQL optimizations (prepared statement caching)
- [x] Batch insert rewriting
- [ ] Database indexes (verify coverage)
- [ ] Query performance analysis

### Application ✅ Implemented
- [x] Async processing for long operations
- [x] Thread pool configuration
- [x] Unlimited timeouts for long operations
- [ ] Response caching (not implemented)
- [ ] Query result caching (not implemented)

### Infrastructure (Planned)
- [ ] CDN for static assets
- [ ] Load balancer configuration
- [ ] Auto-scaling rules
- [ ] Database read replicas

---

## 10.5 Performance Testing Plan {#id-10.5-performance-testing-plan}

### Phase 1: Baseline Testing
1. **Setup**: Single instance, no load
2. **Measure**: Response times for all endpoints
3. **Document**: Baseline metrics

### Phase 2: Load Testing
1. **Ramp-up**: 10 → 50 → 100 → 500 users
2. **Duration**: 30 minutes per level
3. **Measure**: Response times, error rates, resource usage

### Phase 3: Stress Testing
1. **Increase load**: Until system degrades
2. **Identify**: Bottlenecks and breaking points
3. **Document**: Maximum capacity

### Phase 4: Endurance Testing
1. **Sustained load**: Target load for 4-8 hours
2. **Monitor**: Memory leaks, connection leaks
3. **Verify**: System stability

---

## Next Steps

1. **Run performance tests** using JMeter/Gatling/k6
2. **Document actual metrics** in this file
3. **Configure monitoring** (Prometheus/Grafana)
4. **Set up alerting** for critical metrics
5. **Define SLAs** based on test results

---


