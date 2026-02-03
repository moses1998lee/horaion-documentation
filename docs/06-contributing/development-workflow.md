# Development Workflow

## Branch Strategy
```mermaid
gitGraph
    commit id: "initial"
    branch dev
    checkout dev
    commit id: "feature-1"
    commit id: "feature-2"
    checkout main
    merge dev id: "release"
    tag id: "v1.0.0"
    checkout dev
    commit id: "feature-3"
```

## Workflow Steps

### 1. Feature Development
```bash
# Create feature branch
git checkout -b feature/your-feature-name

# Make changes and commit
git add .
git commit -m "feat: add new feature"

# Push to remote
git push origin feature/your-feature-name
```

### 2. Create Merge Request
- Source: `feature/your-feature-name`
- Target: `dev`
- Pipeline automatically runs all checks
- Wait for quality gates to pass

### 3. Code Review
- Reviewers check the code
- Qodana provides automated code review comments
- SonarCloud provides quality metrics
- All checks must pass before merge

### 4. Merge to Dev
- Merge when all checks pass
- Pipeline runs on dev branch
- Docker image built as `dev-{pipeline_id}`

### 5. Release to Main
- Create merge request from `dev` to `main`
- All quality gates re-evaluated
- On merge, Docker image built as `rc-{pipeline_id}`

### 6. Production Release
```bash
# Create version tag
git tag -a v1.2.3 -m "Release version 1.2.3"
git push origin v1.2.3
```
- Pipeline builds production Docker image
- Tagged as `1.2.3` and `latest`

## Commit Message Convention
Use conventional commits:
- `feat:` New feature
- `fix:` Bug fix
- `docs:` Documentation changes
- `style:` Code style changes
- `refactor:` Code refactoring
- `test:` Test changes
- `chore:` Maintenance tasks

## Quality Requirements
Before merging, ensure:
1. ✅ All tests pass
2. ✅ Coverage ≥80%
3. ✅ SonarCloud quality gate passes
4. ✅ Qodana analysis passes
5. ✅ No security vulnerabilities
6. ✅ Code style compliance

## Local Development Tips
```bash
# Run tests with coverage
./mvnw verify

# Check code quality locally
# (Consider installing SonarScanner and Qodana CLI)

# Build Docker image locally
docker build -f deployments/Dockerfile -t local/horaion:latest .
```

## Troubleshooting
### Common Issues
1. **Coverage too low**: Add more tests or refactor complex code
2. **Quality gate failures**: Address SonarCloud issues
3. **Build failures**: Check Java version compatibility
4. **Docker build failures**: Verify Dockerfile exists in deployments/

### Pipeline Debugging
- Check GitLab CI logs for detailed error messages
- Verify environment variables are set
- Ensure Docker daemon is running for publish stage
- Check network connectivity for external services