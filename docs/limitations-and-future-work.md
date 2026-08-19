# Limitations & Future Work
[← Back to README](../README.md) | [Architecture](architecture.md) | [Implementation](implementation.md) | [Validation](validation.md)

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/33d165d3-b539-4936-a5f0-4ac9b0067221" />

## 1. Purpose

This document defines the boundaries of the VProfile Multi-Tier Provisioning project and identifies logical areas for future evolution.

The purpose is to distinguish clearly between:

```text
What this project demonstrates
        ↓
What this project intentionally does not demonstrate
        ↓
What could be implemented in future iterations
```

This project is a foundational traditional application deployment exercise, not a production cloud platform.

## 2. Current Project Scope

The project demonstrates:

- Multi-VM Linux application deployment
- Service-per-VM architecture
- Vagrant-based VM orchestration
- Manual service provisioning
- Bash-based provisioning
- Nginx reverse-proxy configuration
- Tomcat application deployment
- MySQL/MariaDB configuration
- RabbitMQ configuration
- Memcached configuration
- Hostname-based service communication
- Vagrant provisioning
- Application-level validation
- Repeatable local environment creation

The project progresses from:

```text
Manual Deployment
       ↓
Understand the System
       ↓
Validate the System
       ↓
Destroy the Environment
       ↓
Automate the Deployment
       ↓
Recreate the Environment
```

## 3. Application Ownership Boundary

VProfile is the application workload used by the project.

This repository represents the DevOps work performed around the application, including:

- Environment provisioning
- Service installation
- Service configuration
- Application deployment
- Backend connectivity
- Reverse-proxy configuration
- Provisioning automation
- Validation

It does not claim ownership of the VProfile application's Java business logic or application development.

```text
VProfile Application
        ≠
DevOps Engineering Around the Application
```

## 4. Infrastructure Limitations

### 4.1 Local Virtual Machines

The environment runs locally using Vagrant and VirtualBox.

```text
Current
Local VMs
   ↓
Vagrant + VirtualBox

Not demonstrated
Cloud Infrastructure
```

This provides a useful multi-tier learning environment, but it is not equivalent to a cloud production environment.

### 4.2 No Cloud Infrastructure

This project does not implement:

- AWS infrastructure
- Cloud networking
- Cloud load balancing
- Cloud-managed databases
- Cloud-managed caching
- Cloud-managed messaging
- Cloud auto scaling

These capabilities belong to later stages of the broader DevOps roadmap.

## 5. Infrastructure as Code Limitation

Vagrant defines the local VM environment, but this project does not implement a dedicated cloud Infrastructure as Code solution such as Terraform.

Current model:

```text
Vagrantfile
     +
Bash Scripts
     ↓
Local VM Provisioning
```

Future model:

```text
Terraform
     ↓
Cloud Infrastructure
```

## 6. Configuration Management Limitation

The service configuration is performed through Bash provisioning scripts.

Current model:

```text
Vagrant
   ↓
Bash Script
   ↓
Install + Configure Service
```

The project does not implement a dedicated configuration-management framework.

A later stage can introduce tools such as Ansible.

## 7. CI/CD Limitation

The project does not implement a continuous integration or continuous delivery pipeline.

Current application flow:

```text
Source
  ↓
Maven Build
  ↓
WAR
  ↓
Tomcat
```

There is no dedicated CI/CD platform coordinating source changes, builds, testing, artifact management, and deployment.

Future work can introduce:

- Jenkins
- GitHub Actions
- GitLab CI/CD
- Automated testing
- Security scanning
- Docker build and publish workflows

## 8. Containerization Limitation

The current deployment model uses traditional virtual machines:

```text
VM
 └── Service
```

It does not containerize the VProfile services.

Current architecture:

```text
db01   → MySQL
mc01   → Memcached
rmq01  → RabbitMQ
app01  → Tomcat
web01  → Nginx
```

Future direction:

```text
Containers
    ↓
Containerized Services
    ↓
Containerized Application
```

## 9. Microservices Limitation

The project uses a multi-tier application architecture, but it does not demonstrate independently deployable microservices.

```text
Current
Multi-Tier Application
        ↓
Five infrastructure/service components

Future Direction
Containerized Application
        ↓
Microservice-oriented deployment
```

Microservices should not be claimed as implemented by this repository unless separately demonstrated.

## 10. High-Availability Limitation

The architecture uses a single instance of each major service:

```text
1 × Nginx
1 × Tomcat
1 × MySQL/MariaDB
1 × RabbitMQ
1 × Memcached
```

There is no demonstrated redundancy for individual service tiers.

The project does not establish:

- Multiple Tomcat instances
- Multiple Nginx instances
- Database replication
- Automatic failover
- Multi-node RabbitMQ
- Multi-node caching
- Automatic recovery across hosts

Therefore, this architecture should not be described as highly available or production-grade HA.

## 11. Scalability Limitation

The current architecture is limited by its single-instance design.

Current:

```text
              Nginx
                │
                ▼
             Tomcat
                │
        ┌───────┼───────┐
        ▼       ▼       ▼
       DB      MQ      Cache
```

Future scaling model:

```text
             Load Balancer
              /                      ▼           ▼
         Tomcat-1     Tomcat-2
```

There is no demonstrated mechanism for application-instance scaling or automatic scaling.

## 12. Security Limitations

The project is designed as a learning environment rather than a production security implementation.

The project uses simple lab credentials and configuration values.

It does not demonstrate:

- Secrets management
- Credential rotation
- Encryption of application secrets
- TLS between every service
- Production certificate management
- IAM-based service authentication
- Centralized identity management

A production implementation would require a substantially stronger security model.

## 13. Observability Limitation

The project validates services primarily through service checks and application behavior.

It does not implement a centralized observability stack.

There is no demonstrated:

- Centralized log aggregation
- Metrics collection
- Distributed tracing
- Application performance monitoring
- Alerting platform
- Dashboarding system

Current validation:

```text
Service Status
      +
Application Behavior
      +
Connectivity Tests
```

Production observability would extend this to:

```text
Metrics
 +
Logs
 +
Traces
 +
Alerts
 =
Production Observability
```

## 14. Deployment Limitation

The application is deployed as a WAR artifact directly into Tomcat.

```text
Maven
  ↓
vprofile-v2.war
  ↓
ROOT.war
  ↓
Tomcat
```

The application configuration is determined during the build/deployment process in this implementation.

A more mature deployment model could separate configuration from the application artifact and inject environment-specific configuration during deployment.

## 15. Resource Limitation

The local VMs are intentionally lightweight.

The application VM has limited memory and Maven may require a memory workaround to complete the build successfully.

This is appropriate for a local learning environment but is not representative of production resource sizing.

A production implementation would require:

- Capacity planning
- Resource sizing
- Performance testing
- Load testing
- Memory tuning
- CPU sizing
- Storage planning

## 16. Networking Limitation

The project demonstrates inter-VM networking and hostname-based service communication, but the network model is intentionally simple.

The project does not demonstrate production cloud networking concepts such as:

- VPC design
- Public/private subnets
- Route tables
- NAT gateways
- Internet gateways
- Network ACLs
- Cloud security groups
- Private DNS services

## 17. Testing Limitation

The validation strategy is primarily functional and integration-oriented.

It verifies:

```text
Nginx → Tomcat
Tomcat → MySQL
Tomcat → RabbitMQ
Tomcat → Memcached
```

It does not demonstrate:

- Automated unit-test pipelines
- Automated integration-test pipelines
- Load testing
- Stress testing
- Failure injection
- Chaos testing
- Security testing
- Performance benchmarking

The validation proves that the learning environment works, not that it meets production performance or resilience requirements.

## 18. Automation Limitations

The Bash provisioning scripts make the environment repeatable, but Bash-based provisioning has limitations:

- Scripts can become difficult to maintain as complexity increases.
- Idempotency must be deliberately designed.
- Error handling must be explicitly implemented.
- State management is limited.
- Configuration drift can still occur.
- Secrets require careful handling.
- Large environments become harder to manage.

The project intentionally uses Bash because the learning objective is to transform manual provisioning into automation.

## 19. Future Work

The logical evolution is:

```text
Current
Traditional Multi-Tier VMs
        ↓
Networking
        ↓
Docker
        ↓
Containerized Applications
        ↓
Microservice Deployment
        ↓
AWS Cloud Deployment
        ↓
Infrastructure Automation
        ↓
CI/CD
```

## 20. Future Work — Networking

Potential work includes:

- IP addressing
- Ports
- Protocols
- OSI model
- Networking commands
- Service-to-service network troubleshooting

The goal is to move from simply making services communicate to understanding why communication works and how to troubleshoot it when it fails.

## 21. Future Work — Docker

The current model:

```text
VM
 └── Service
```

can evolve toward:

```text
Container
 └── Service
```

Potential work includes:

- Creating Docker images
- Running VProfile components in containers
- Understanding container networking
- Managing container configuration
- Comparing VM-based and container-based deployment
- Building a containerized application environment

## 22. Future Work — Microservices

The learning progression becomes:

```text
Traditional Multi-Tier
        ↓
Containerized Multi-Tier
        ↓
Microservices
```

The objective is to understand how service boundaries, networking, configuration, and deployment change as applications move toward independently deployable services.

## 23. Future Work — Cloud Migration

A later evolution can move the application from the local Vagrant environment into AWS.

```text
Local VMs
   ↓
AWS Infrastructure
   ↓
Cloud Application Deployment
```

This provides an opportunity to compare:

```text
Vagrant / VirtualBox
        vs.
AWS Cloud Infrastructure
```

and understand how the same application architecture maps onto cloud infrastructure.

## 24. Future Work — Infrastructure as Code

A later implementation could move infrastructure provisioning from local Vagrant/Bash toward dedicated Infrastructure as Code.

```text
Vagrant + Bash
      ↓
Terraform
      ↓
Cloud Infrastructure
```

The objective would be to define infrastructure declaratively and manage infrastructure lifecycle through code.

## 25. Future Work — Configuration Management

Service configuration could later be separated from infrastructure creation.

```text
Infrastructure Provisioning
        ↓
Terraform
        ↓
Configuration Management
        ↓
Ansible
```

This separates:

```text
Where infrastructure exists
        from
How software is configured
```

## 26. Future Work — CI/CD

The current project proves that the application can be manually built and deployed and that provisioning can be automated.

A future pipeline could automate the complete software delivery flow:

```text
Git Push
   ↓
Build
   ↓
Test
   ↓
Package
   ↓
Artifact
   ↓
Deploy
   ↓
Validate
```

## 27. Future Work — Observability

A future production-oriented implementation could add:

```text
Application
     │
     ├── Logs
     ├── Metrics
     └── Traces
             │
             ▼
       Observability
             │
             ▼
          Alerts
```

This would extend the current functional validation model into operational monitoring.

## 28. Future Work — High Availability

A future architecture could introduce redundancy:

```text
             Load Balancer
              /                      ▼           ▼
         App Server   App Server
             │           │
             └─────┬─────┘
                   ▼
              HA Backend
```

Potential areas include:

- Multiple application instances
- Load balancing
- Database replication
- Backend service redundancy
- Failure recovery

## 29. Future Work — Security

A production-oriented evolution should introduce stronger security controls.

Potential areas include:

- Secrets management
- Secure credential handling
- TLS
- Service authentication
- Least-privilege access
- Network segmentation
- Secure configuration injection

## 30. Future Evolution Map

```text
┌─────────────────────────────┐
│ Traditional Multi-Tier VMs  │
│                             │
│ Vagrant + Bash              │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│ Networking                  │
│                             │
│ IPs + Ports + Protocols     │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│ Docker                      │
│                             │
│ Container Fundamentals      │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│ Containerized Applications  │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│ Microservices               │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│ AWS Cloud Deployment        │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│ Infrastructure Automation   │
│ Terraform + Ansible         │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│ CI/CD                       │
│ Jenkins / GitHub / GitLab   │
└─────────────────────────────┘
```

## 31. What Should Not Be Claimed

Until separately implemented and evidenced, this repository should not claim:

- Production-ready cloud architecture
- AWS infrastructure
- Terraform infrastructure
- Kubernetes orchestration
- CI/CD implementation
- Production-grade high availability
- Production observability
- Zero-downtime deployment
- Automated cloud scaling
- Microservice implementation
- Development of the VProfile application

The repository should represent the work that was actually performed rather than technologies that are merely planned.

## 32. Final Perspective

This project's value is not that it represents a complete production platform.

Its value is that it establishes the foundation for the next layers of DevOps engineering.

The learning progression is:

```text
Understand the Application
          ↓
Understand the Infrastructure
          ↓
Understand the Dependencies
          ↓
Deploy Manually
          ↓
Validate
          ↓
Automate
          ↓
Containerize
          ↓
Move to Cloud
          ↓
Automate Infrastructure
          ↓
Automate Delivery
```

The current project establishes the traditional multi-tier deployment and provisioning foundation from which those later capabilities can be learned and implemented.

### Core Principle

> **Do not hide the limitations of the current project. Use them to show exactly where the next stage of engineering begins.**

[← Back to README](../README.md) | [Architecture](architecture.md) | [Implementation](implementation.md) | [Validation](validation.md)
