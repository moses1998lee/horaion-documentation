# Environment Variables Configuration

This document outlines the environment variables used to configure the Horaion application across different deployment environments.

## Overview

The application uses environment variables for configuration to support the 12-factor app methodology. Configuration is separated into different files for different deployment scenarios.

## Variable Categories

### 1. Docker Image Configuration
| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `HORAION_APP_IMAGE` | Docker image name and tag | `horaion-app:latest` | Yes |
| `IMAGE_NAME` | Base image name for Docker Hub | `horaion-app` | Yes |
| `VERSION` | Image version tag | `1.0.0` | No |
| `DOCKERFILE_PATH` | Path to Dockerfile | `Dockerfile` | No |
| `BUILD_CONTEXT` | Docker build context | `.` | No |
| `PLATFORMS` | Target platforms for multi-arch build | `linux/amd64,linux/arm64` | No |

### 2. Application Configuration
| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `SPRING_PROFILE` | Spring Boot active profile | `sit` | No |
| `APP_PORT` | Application port (host mapping) | `8100` | No |
| `SERVER_PORT` | Application port (container internal) | `8100` | No |
| `TZ` | Timezone | `Asia/Singapore` | No |

### 3. Database Configuration
| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `POSTGRES_DB_HOST` | PostgreSQL hostname | `postgres` | Yes |
| `POSTGRES_DB_NAME` | Database name | `horaion` | No |
| `POSTGRES_DB_USER` | Database username | `horaion` | No |
| `POSTGRES_DB_PASS` | Database password | `horaion123` | No |
| `POSTGRES_DB_PORT` | Database port (host mapping) | `54321` | No |

### 4. Database Connection Pool Configuration
| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `DB_POOL_SIZE` | Maximum connection pool size | `10` | No |
| `DB_POOL_MIN_IDLE` | Minimum idle connections | `2` | No |
| `DB_IDLE_TIMEOUT` | Idle connection timeout (ms) | `300000` | No |
| `DB_MAX_LIFETIME` | Maximum connection lifetime (ms) | `1800000` | No |
| `DB_CONNECTION_TIMEOUT` | Connection timeout (ms) | `30000` | No |
| `DB_LEAK_DETECTION` | Leak detection threshold (ms) | `60000` | No |

### 5. Authentication Configuration (Auth0)
| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `AUTH0_DOMAIN` | Auth0 domain URL | - | Yes |
| `AUTH0_AUDIENCE` | Auth0 API audience | - | Yes |

### 6. CORS Configuration
| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `CORS_ALLOWED_ORIGINS` | Allowed origins | `http://localhost:3000` | No |
| `CORS_ALLOWED_METHODS` | Allowed HTTP methods | `GET,POST,PUT,PATCH,DELETE,OPTIONS` | No |
| `CORS_ALLOWED_HEADERS` | Allowed HTTP headers | `Authorization,Content-Type,Accept,X-Request-Id` | No |
| `CORS_EXPOSED_HEADERS` | Exposed HTTP headers | `X-Request-Id` | No |
| `CORS_ALLOW_CREDENTIALS` | Allow credentials | `true` | No |
| `CORS_MAX_AGE` | Preflight cache duration (seconds) | `3600` | No |

### 7. Logging Configuration
| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `SHOW_SQL_FLAG` | Enable SQL query logging | `false` | No |

### 8. Docker Hub Credentials
| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `DOCKERHUB_USERNAME` | Docker Hub username | - | Yes |
| `DOCKERHUB_PASSWORD` | Docker Hub password/token | - | No |

## Configuration Files

### `.env.compose.example`
Used for Docker Compose deployments. Contains runtime configuration for the application and database.

**Important**: Copy to `.env.compose` and fill in actual values before use.

### `.env.docker.example`
Used for Docker image build and push operations. Contains Docker Hub credentials and build configuration.

**Important**: Copy to `.env.docker` and fill in actual values before use.

## Environment-Specific Configuration

### Development (Local)
```bash
SPRING_PROFILE=local
CORS_ALLOWED_ORIGINS=http://localhost:3000
SHOW_SQL_FLAG=true
```

### System Integration Testing (SIT)
```bash
SPRING_PROFILE=sit
CORS_ALLOWED_ORIGINS=http://sit-frontend.example.com
SHOW_SQL_FLAG=false
```

### Production
```bash
SPRING_PROFILE=prod
CORS_ALLOWED_ORIGINS=https://production-frontend.example.com
SHOW_SQL_FLAG=false
DB_POOL_SIZE=20
DB_POOL_MIN_IDLE=5
```

## Security Considerations

1. **Never commit sensitive data**: Environment files with real credentials should be excluded from version control (already in `.gitignore`)

2. **Use secrets management**: For production, consider using:
   - Docker Secrets
   - Kubernetes Secrets
   - AWS Secrets Manager
   - HashiCorp Vault

3. **Principle of least privilege**: Database credentials should have minimal required permissions

4. **Regular rotation**: Schedule regular credential rotation for production environments

## Validation

The `docker.sh` script includes validation for required variables:

```bash
# Validate required variables
if [ -z "$DOCKERHUB_USERNAME" ]; then
    echo "Error: DOCKERHUB_USERNAME not set in .env.docker"
    exit 1
fi

if [ -z "$IMAGE_NAME" ]; then
    echo "Error: IMAGE_NAME not set in .env.docker"
    exit 1
fi
```

## Best Practices

1. **Use descriptive defaults**: Provide sensible defaults for optional variables
2. **Document all variables**: Keep this documentation updated as new variables are added
3. **Environment segregation**: Use different variable sets for different environments
4. **Validation**: Implement validation for required variables at startup
5. **Sensitive data**: Never log or expose sensitive environment variables

## Variable Resolution Order

The application resolves variables in this order (highest priority first):

1. Docker Compose environment variables
2. `.env.compose` file
3. Dockerfile environment variables
4. System environment variables
5. Application defaults

## Troubleshooting

### Common Issues

1. **Missing required variables**: Check error messages and verify all required variables are set
2. **Port conflicts**: Ensure `APP_PORT` and `POSTGRES_DB_PORT` don't conflict with existing services
3. **Database connectivity**: Verify `POSTGRES_DB_HOST` matches the Docker Compose service name
4. **CORS issues**: Ensure `CORS_ALLOWED_ORIGINS` matches the frontend application URL

### Debugging
```bash
# Check environment variables in running container
docker exec horaion-app env

# View Docker Compose resolved variables
docker-compose config
```

## Related Documentation

- [Docker Deployment Guide](../deployment/docker.md)
- [Database Configuration](../database/postgresql.md)
- [Security Configuration](../security/auth0-integration.md)