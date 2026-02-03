# Configuration and Deployment

## Build Configuration

### Maven Settings
The CI pipeline generates a custom `settings.xml` for GitHub authentication:
```xml
<settings>
  <servers>
    <server>
      <id>github</id>
      <username>${GITHUB_USERNAME}</username>
      <password>${GITHUB_TOKEN}</password>
    </server>
  </servers>
</settings>
```

### Maven Options
```bash
MAVEN_OPTS="
  -Dhttps.protocols=TLSv1.2
  -Dmaven.repo.local=$CI_PROJECT_DIR/.m2/repository
  -Dorg.slf4j.simpleLogger.showDateTime=true
  -Djava.awt.headless=true
"

MAVEN_CLI_OPTS="
  --batch-mode
  --errors
  --fail-at-end
  --show-version
  --no-transfer-progress
"
```

## Docker Configuration

### Image Tagging Strategy
| Branch/Tag | Version Tag | Latest Tag | Description |
|------------|-------------|------------|-------------|
| `v1.2.3` | `1.2.3` | `latest` | Production release |
| `main` | `rc-{pipeline_id}` | `rc-latest` | Release candidate |
| `dev` | `dev-{pipeline_id}` | `dev-latest` | Development build |

### Docker Build Command
```bash
docker build \
  --file deployments/Dockerfile \
  --tag "$DOCKERHUB_USERNAME/$IMAGE_NAME:$VERSION" \
  --tag "$DOCKERHUB_USERNAME/$IMAGE_NAME:$LATEST_TAG" \
  .
```

## Cache Configuration
```yaml
cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - .m2/repository
    - target/
```

## Branch Protection Rules
The pipeline runs on:
1. **Merge Request Events**: All analysis and quality gates
2. **Dev Branch**: Full pipeline including Docker publish
3. **Main Branch**: Full pipeline including Docker publish
4. **Version Tags**: Production Docker image publishing

## Artifact Retention
- **Build Artifacts**: 1 hour
- **Test Artifacts**: 1 week
- **Coverage Reports**: 1 week