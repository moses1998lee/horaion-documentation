# Project Overview

## Introduction
This is a Spring Boot project bootstrapped with Maven, designed for modern CI/CD workflows with comprehensive quality gates and automated deployment pipelines.

## Technology Stack
- **Framework**: Spring Boot
- **Build Tool**: Apache Maven 3.9.11
- **Java Version**: Eclipse Temurin 21
- **CI/CD**: GitLab CI
- **Quality Analysis**: SonarCloud, Qodana
- **Containerization**: Docker
- **Deployment**: Docker Hub

## Project Structure
```
.
├── .gitattributes          # Git line ending configuration
├── .gitignore             # IDE and build artifacts exclusion
├── .gitlab-ci.yml         # CI/CD pipeline configuration
├── .mvn/                  # Maven wrapper configuration
├── mvnw                   # Unix Maven wrapper script
└── mvnw.cmd               # Windows Maven wrapper script
```

## Key Characteristics
- **Modern Java**: Uses Java 21 with Eclipse Temurin distribution
- **Quality-First**: Integrated SonarCloud and Qodana analysis
- **Container-Ready**: Docker build and publish pipeline
- **Cross-Platform**: Maven wrapper for consistent builds across environments
- **Enterprise CI/CD**: Multi-stage GitLab pipeline with quality gates