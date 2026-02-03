# Logging Configuration Guide

## Overview
Horaion uses a structured logging approach with Logback for log management, enhanced by the `LogAspect` for cross-cutting concerns.

## Configuration Files

### 1. Logback Configuration (`logback-spring.xml`)

#### Appenders
**Console Appender**:
- Pattern includes MDC fields: `[%X{requestId}] [%X{userId}]`
- Used in all environments

**File Appender**:
- Logs to `logs/horaion.log`
- Time-based rolling (daily)
- 30-day retention
- 1GB total size cap
- Used in SIT, PROD environments

#### Environment Profiles
```xml
<!-- Local/Dev: Console only -->
<springProfile name="local | dev | development">

<!-- SIT: Console + File -->
<springProfile name="sit">

<!-- PROD: Console + File -->
<springProfile name="prod | production">
```

### 2. Application Configuration (`application.yaml`)

#### Logging Aspect Settings
```yaml
app:
  logging:
    aspect:
      enabled: true  # Enable/disable the logging aspect
      slow-execution-threshold-ms: 1000  # Threshold for WARN logs
```

#### Test Configuration (`application-test.yaml`)
```yaml
app:
  logging:
    aspect:
      enabled: false  # Disabled in tests to avoid interference
```

## Dependencies

### Runtime Dependencies
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-logging</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aspectj</artifactId>
</dependency>

<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

### Test Dependencies
```xml
<dependency>
    <groupId>com.github.valfirst</groupId>
    <artifactId>slf4j-test</artifactId>
    <version>${slf4j.test.version}</version>
    <scope>test</scope>
</dependency>
```

## Build Configuration

### Maven Surefire Plugin
Excludes Logback from test classpath to use SLF4J Test:
```xml
<classpathDependencyExcludes>
    <classpathDependencyExclude>ch.qos.logback:logback-classic</classpathDependencyExclude>
</classpathDependencyExcludes>
```

### JaCoCo Configuration
Excludes constants classes from coverage:
```xml
<excludes>
    <exclude>**/*Application.class</exclude>
    <exclude>**/*Constant.class</exclude>
</excludes>
```

## Environment-Specific Settings

### Development
- Console logging only
- INFO level
- Aspect enabled

### SIT (System Integration Testing)
- Console + File logging
- INFO level
- Aspect enabled
- File rotation enabled

### Production
- Console + File logging
- INFO level
- Aspect enabled
- File rotation with retention

## Monitoring and Metrics

### Prometheus Integration
Metrics are exposed via Micrometer Prometheus registry:
- Metric: `app.method.execution`
- Type: Timer
- Tags: `method`, `status`

### Log Analysis
Logs include structured fields for analysis:
- Request correlation via `requestId`
- User context via `userId`
- Method context via `method`
- HTTP context via `httpMethod` and `endpoint`

## Troubleshooting

### Aspect Not Logging
1. Check `app.logging.aspect.enabled` property
2. Verify method matches pointcut patterns
3. Check Spring AOP is properly configured

### Missing MDC Fields
1. Verify SecurityContext is available for user ID
2. Check RequestContextHolder for HTTP context
3. Ensure MDC cleanup isn't removing needed fields

### Performance Issues
1. Adjust `slow-execution-threshold-ms` if needed
2. Consider disabling aspect in high-throughput scenarios
3. Monitor metrics for abnormal patterns

## Testing Configuration

### Test Setup
```java
@TestConfiguration
public class MetricsConfigurationTest {
    @Bean
    public MeterRegistry meterRegistry() {
        return new SimpleMeterRegistry();
    }
    
    @Bean
    public LogAspectProperties logAspectProperties() {
        return new LogAspectProperties(true, 1000L);
    }
}
```

### Test Profiles
Use `@ActiveProfiles("test")` to activate test configuration which disables the aspect to prevent interference with unit tests.