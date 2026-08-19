# VProfile Architecture
[← Back to README](../README.md)

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/e4895072-8071-450b-9092-a3279112f065" />

## 1. Architecture Overview

VProfile is deployed as a **traditional multi-tier application** distributed across five Linux virtual machines.

The architecture separates the frontend, application server, database, messaging, and caching responsibilities into independent VMs.

```text
                         User
                           │
                           │ HTTP :80
                           ▼
                  ┌─────────────────┐
                  │   Nginx / web01 │
                  │   Web Tier      │
                  │   Port 80       │
                  └────────┬────────┘
                           │
                     proxy_pass
                           │
                           ▼
                  ┌─────────────────┐
                  │ Tomcat / app01  │
                  │ Application Tier│
                  │   Port 8080     │
                  └───────┬─────────┘
                          │
             ┌────────────┼────────────┐
             │            │            │
             ▼            ▼            ▼
      ┌────────────┐ ┌────────────┐ ┌────────────┐
      │ MySQL /    │ │ RabbitMQ / │ │ Memcached /│
      │ db01       │ │ rmq01      │ │ mc01       │
      │ Port 3306  │ │ Port 5672  │ │ Port 11211 │
      │ Database   │ │ Messaging  │ │ Cache      │
      └────────────┘ └────────────┘ └────────────┘
```

The application workload itself is supplied as part of the course/project material. The engineering work represented by this project is the infrastructure environment, service configuration, application deployment, connectivity, provisioning, and validation around that workload.

## 2. Component Model

| VM | Hostname | Operating System | Service | Port | Responsibility |
|---|---|---|---|---:|---|
| Database VM | `db01` | CentOS | MySQL/MariaDB | `3306` | Persistent application data |
| Cache VM | `mc01` | CentOS | Memcached | `11211` | Database/application caching |
| Messaging VM | `rmq01` | CentOS | RabbitMQ | `5672` | Message queuing |
| Application VM | `app01` | CentOS | Tomcat | `8080` | Runs the VProfile Java application |
| Web VM | `web01` | Ubuntu | Nginx | `80` | Frontend and reverse proxy |

## 3. Request Flow

```text
Browser
   │
   │ HTTP request
   │ Port 80
   ▼
Nginx / web01
   │
   │ proxy_pass
   ▼
Tomcat / app01
   │
   ├──────────────► MySQL / db01
   ├──────────────► RabbitMQ / rmq01
   └──────────────► Memcached / mc01
```

### Browser → Nginx

Nginx listens on HTTP port `80` and provides the user-facing entry point.

### Nginx → Tomcat

Nginx operates as a reverse proxy and forwards requests to `app01:8080`.

Conceptually:

```nginx
upstream vproapp {
    server app01:8080;
}

server {
    listen 80;

    location / {
        proxy_pass http://vproapp;
    }
}
```

### Tomcat → Backend Services

The VProfile application running inside Tomcat communicates with:

```text
Tomcat / app01
       │
       ├──► MySQL / db01
       ├──► Memcached / mc01
       └──► RabbitMQ / rmq01
```

The application configuration uses service hostnames rather than hardcoded backend IP addresses.

## 4. Backend Service Responsibilities

### MySQL / MariaDB — `db01`

MySQL/MariaDB provides the persistent relational data store.

```text
Tomcat
   │
   │ database connection
   ▼
db01 :3306
```

Database connectivity is exercised during application login validation.

### Memcached — `mc01`

Memcached provides the caching layer.

```text
Tomcat
   │
   │ cache access
   ▼
mc01 :11211
```

Memcached must be reachable from the application VM rather than listening only on localhost.

### RabbitMQ — `rmq01`

RabbitMQ provides message-queue functionality.

```text
Tomcat
   │
   │ messaging
   ▼
rmq01 :5672
```

The application includes a validation function that exercises the RabbitMQ connection.

## 5. Service Discovery

The VMs communicate using hostnames:

```text
web01
app01
db01
mc01
rmq01
```

The Vagrant environment uses the Vagrant Hostmanager plugin to populate `/etc/hosts` entries.

Conceptually:

```text
Vagrant Hostmanager
        │
        ▼
/etc/hosts
        │
        ├── db01
        ├── mc01
        ├── rmq01
        ├── app01
        └── web01
```

This allows configurations to reference:

```text
app01:8080
db01:3306
mc01:11211
rmq01:5672
```

instead of embedding specific VM IP addresses.

## 6. Configuration Relationships

### Nginx Configuration

```text
Nginx / web01
      │
      │ upstream
      ▼
app01:8080
```

### Application Configuration

The application's `application.properties` configuration establishes the connections from Tomcat to the backend services.

```text
application.properties
        │
        ├── db host     → db01
        ├── cache host  → mc01
        └── MQ host     → rmq01
```

## 7. Dependency Model

```text
User
 │
 ▼
Nginx
 │
 ▼
Tomcat
 │
 ├──► MySQL
 ├──► Memcached
 └──► RabbitMQ
```

### Provisioning Order

The documented provisioning order follows the dependency structure:

```text
MySQL
   ↓
Memcached
   ↓
RabbitMQ
   ↓
Tomcat
   ↓
Nginx
```

The automated workflow therefore provisions:

```text
db01
  ↓
mc01
  ↓
rmq01
  ↓
app01
  ↓
web01
```

Backend dependencies are prepared before the services that depend on them.

## 8. Network Boundaries

```text
web01
  └── TCP 80
       │
       ▼
app01
  └── TCP 8080
       │
       ├──► db01 :3306
       ├──► mc01 :11211
       └──► rmq01 :5672
```

Service-specific firewall configuration allows the required traffic between the VMs.

## 9. Nginx Reverse-Proxy Boundary

```text
External/User-facing
        │
        ▼
┌──────────────────┐
│ Nginx / web01    │
│ HTTP :80         │
└────────┬─────────┘
         │
         │ internal proxy
         ▼
┌──────────────────┐
│ Tomcat / app01   │
│ HTTP :8080       │
└──────────────────┘
```

The user interacts with Nginx rather than directly accessing Tomcat.

## 10. End-to-End Architecture Validation

```text
Browser
   │
   ▼
Nginx
   │
   ▼
Tomcat
   │
   ├──► Login
   │      └──► MySQL validation
   │
   ├──► RabbitMQ test
   │      └──► RabbitMQ validation
   │
   └──► User/cache test
          └──► Memcached validation
```

| Validation | Architecture Area Exercised |
|---|---|
| Application page loads | Browser → Nginx → Tomcat |
| Application login | Tomcat → MySQL |
| RabbitMQ test | Tomcat → RabbitMQ |
| User/cache test | Application → MySQL/Memcached |

## 11. Architecture and Provisioning Relationship

The architecture determines the provisioning sequence:

```text
Architecture Dependency
        │
        ▼
MySQL / Memcached / RabbitMQ
        │
        ▼
Tomcat
        │
        ▼
Nginx
```

The Vagrantfile encodes this provisioning order:

```text
Vagrantfile
    │
    ├── db01
    ├── mc01
    ├── rmq01
    ├── app01
    └── web01
```

The architecture defines the service dependencies; the Vagrantfile encodes how those components are provisioned.

## 12. Architectural Decisions

### Separate services into separate VMs

Each major service is isolated into its own VM, making service boundaries and inter-service communication explicit.

### Use Nginx as a reverse proxy

Nginx provides the user-facing HTTP endpoint and forwards requests to Tomcat.

### Use hostname-based service discovery

Services reference hostnames such as `app01`, `db01`, `mc01`, and `rmq01`.

### Provision dependencies before dependents

The provisioning sequence follows:

```text
Backend services
      ↓
Application server
      ↓
Frontend proxy
```

### Use separate Linux distributions where specified

The documented environment uses CentOS for the backend/application VMs and Ubuntu for the Nginx VM.

## 13. Architecture Boundary

This architecture demonstrates a **traditional local multi-VM application environment**.

It should not be interpreted as a production cloud architecture.

It does not establish:

- Cloud networking
- AWS infrastructure
- Terraform-managed infrastructure
- Kubernetes orchestration
- Production-grade high availability
- Production load balancing
- Production observability
- Production database replication
- CI/CD
- GitOps

## 14. Evolution Path

```text
Traditional Multi-Tier VMs
          │
          ▼
Containerized Application
          │
          ▼
Cloud Deployment
          │
          ▼
Infrastructure as Code
          │
          ▼
Configuration Management
          │
          ▼
Container Orchestration
          │
          ▼
CI/CD
          │
          ▼
GitOps
```

## 15. Architecture Summary

```text
                         USER
                           │
                           ▼
                    ┌─────────────┐
                    │    NGINX    │
                    │   web01:80  │
                    └──────┬──────┘
                           │
                       proxy_pass
                           │
                           ▼
                    ┌─────────────┐
                    │    TOMCAT   │
                    │  app01:8080 │
                    └──────┬──────┘
                           │
              ┌────────────┼────────────┐
              │            │            │
              ▼            ▼            ▼
         ┌─────────┐  ┌─────────┐  ┌──────────┐
         │  MYSQL  │  │ RABBITMQ│  │ MEMCACHE │
         │  db01   │  │  rmq01  │  │   mc01   │
         │  :3306  │  │  :5672  │  │  :11211  │
         └─────────┘  └─────────┘  └──────────┘
```

### Core Mental Model

> **Nginx is the entry point → Tomcat runs the application → MySQL stores persistent data → Memcached provides caching → RabbitMQ provides messaging → Vagrant provides the multi-VM environment in which these components are provisioned and connected.**

[← Back to README](../README.md)
