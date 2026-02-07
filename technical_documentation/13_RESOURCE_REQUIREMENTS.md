# - Resource Requirements

> **Horaion Workforce Management Platform - Resource Requirements**
 
 ## 6.1 Hardware Requirements
 
 ### Server Specifications
 - **Minimum Server**: 2 vCPU, 2 GB RAM (Bespoke / Development)
 - **Recommended Server**: 2 vCPU, 4 GB RAM (Production Standard)
 - **Database**: 2 vCPU, 4 GB RAM, 60GB SSD (Managed PostgreSQL)
 - **Network**: 1 Gbps Virtual Network Interface
 
 ### Environment Specifics
 **Production (AWS Lightsail)**:
 - **Instance Size**: 2 vCPU, 4 GB RAM
 - **Storage**: 60 GB SSD
 - **OS**: Amazon Linux 2023 / Ubuntu 22.04 LTS
 
 **Staging (Digital Ocean)**:
 - **Droplet Size**: 2 vCPU, 2 GB RAM
 - **Storage**: 60 GB SSD
 - **OS**: Ubuntu 22.04 LTS
 
 ---
 
 ## 6.2 Software Requirements
 
 - **Operating System**: Docker-compatible Linux distribution (Ubuntu 22.04 LTS Recommended)
 - **Database**: PostgreSQL 15+ (Relational Database)
 - **Runtime**: Java 21 LTS (Spring Boot 3.4 Runtime)
 - **Web Server**: Nginx (Reverse Proxy & SSL Termination)
 - **SSL Certificate**: Valid SSL certificate for HTTPS (AWS ACM or Let's Encrypt)
 - **Container Runtime**: Docker Engine 24+
 
 ---
 
 ## 6.3 Capacity Planning
 
 ### Growth Projections
 - **Storage Growth**: 20% annual growth in database size
 - **User Growth**: Support for 50% annual increase in user base
 - **Backup Storage**: 3x production database size for backups (Daily Snapshots + WAL)
 - **Log Storage**: 6 months of application logs retention (S3-IA / CloudWatch Logs)
 
 ### Transaction Volume Estimates
 - **Average Load**: ~5-10 concurrent requests/second
 - **Peak Load**: ~50 concurrent requests during schedule generation window
 - **Data Volume**: ~100MB initial dataset with projected 20% YoY growth
 
 ---
 
 ## 6.4 Scaling Strategy
 
 ### Vertical Scaling (Phase 1)
 - **Trigger**: Memory usage > 80% or CPU load > 70% sustained for 15 minutes.
 - **Action**: Upgrade server instance size (e.g., 4GB RAM → 8GB RAM).
 
 ### Horizontal Scaling (Phase 2 - Future)
 - **Trigger**: >500 concurrent users.
 - **Action**: Deploy Load Balancer and multiple application instances.
