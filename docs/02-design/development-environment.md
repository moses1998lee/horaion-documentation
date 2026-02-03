# Development Environment Setup

## Prerequisites
- **Java**: JDK 21 (Eclipse Temurin recommended)
- **Maven**: 3.9.11 (automatically managed via wrapper)
- **Git**: Version control system
- **Docker**: For containerized builds (optional)

## IDE Configuration
The project includes IDE-specific ignore patterns for:
- **IntelliJ IDEA** (`.idea/`, `*.iml`, `*.iws`, `*.ipr`)
- **Eclipse/STS** (`.project`, `.classpath`, `.settings/`)
- **VS Code** (`.vscode/`)
- **NetBeans** (`nbproject/`, `nbbuild/`, `nbdist/`)

## Build System
### Maven Wrapper
The project uses Maven wrapper to ensure consistent builds:
```bash
# Unix/Linux/Mac
./mvnw clean install

# Windows
mvnw.cmd clean install
```

### Maven Configuration
- **Wrapper Version**: 3.3.4
- **Distribution**: Apache Maven 3.9.11
- **Repository**: Maven Central

## Line Ending Configuration
- Unix line endings (LF) for shell scripts (`mvnw`)
- Windows line endings (CRLF) for batch files (`*.cmd`)