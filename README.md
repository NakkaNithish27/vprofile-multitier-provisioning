# VProfile Multi-Tier Provisioning
A hands-on DevOps project demonstrating the deployment, configuration, validation, and automated provisioning of a traditional multi-tier VProfile application across multiple Linux virtual machines using Vagrant and Bash.
<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/477c4803-b0fd-4d5e-93cd-7f366583b3f0" />

---

## Overview

This project deploys the **VProfile application** across a five-VM multi-tier environment.

The environment consists of:

- **Nginx** — frontend reverse proxy
- **Tomcat** — application server
- **MySQL/MariaDB** — relational database
- **RabbitMQ** — message broker
- **Memcached** — caching layer

The project follows a practical progression:

```text
Manual Provisioning
        ↓
Service Configuration
        ↓
Application Deployment
        ↓
End-to-End Validation
        ↓
Bash Automation
        ↓
Vagrant Provisioning
        ↓
Single-Command Environment Setup
```

The same application architecture is first understood and configured manually and then converted into a repeatable automated provisioning workflow.

---

## Application Ownership Boundary

The VProfile application is used as the **application workload** for this project; it was provided as part of the course/project material.

My engineering work focuses on the environment around the application:

- Provisioning the virtual machines
- Configuring the required Linux services
- Deploying the application
- Configuring communication between application and backend services
- Configuring Nginx as the frontend reverse proxy
- Validating the complete application stack
- Executing the automated provisioning workflow
- Managing the Vagrant environment lifecycle

This project does **not** claim ownership of the VProfile application's source code, business logic, or application development.

---

## Architecture

![VProfile Architecture](architecture.png)

The environment uses five virtual machines:

| VM | Service | Role |
|---|---|---|
| `web01` | Nginx | Frontend / Reverse Proxy |
| `app01` | Tomcat | Application Server |
| `db01` | MySQL/MariaDB | Database |
| `rmq01` | RabbitMQ | Message Broker |
| `mc01` | Memcached | Caching Layer |

### Request Flow

```text
User
  │
  │ HTTP :80
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

Nginx receives the user's HTTP request and forwards it to Tomcat. The application server communicates with the backend database, message broker, and caching service.

For the detailed architecture and service relationships:

**[Architecture →](docs/architecture.md)**

---

## My Engineering Contribution

The practical work covered the complete deployment lifecycle:

### Infrastructure

- Created and managed a multi-VM Vagrant environment
- Configured VM networking and hostname-based service communication
- Managed VM lifecycle using Vagrant

### Linux Service Configuration

- Installed and configured the required services
- Managed services using `systemctl`
- Configured service connectivity and required ports
- Troubleshot service and configuration issues

### Application Deployment

- Configured Tomcat as the application server
- Built and deployed the application artifact
- Configured the application to communicate with backend services

### Reverse Proxy

- Installed and configured Nginx
- Configured an upstream pointing to Tomcat
- Configured HTTP traffic on port 80
- Activated the VProfile Nginx configuration

### Automation

The manual provisioning process was converted into an automated workflow using:

- Bash shell scripts
- Vagrant shell provisioners
- Non-interactive package installation
- Shell variables
- Non-interactive MySQL commands
- Heredoc-based configuration file generation

The result is a provisioning workflow where:

```bash
vagrant up
```

creates and provisions the complete environment.

---

## Automated Provisioning

The automated environment separates the provisioning logic by service:

```text
Vagrantfile
    │
    ├── db01      → mysql.sh
    ├── mc01      → memcache.sh
    ├── rmq01     → rabbitmq.sh
    ├── app01     → tomcat_ubuntu.sh
    └── web01     → nginx.sh
```

The provisioning lifecycle is:

```text
vagrant up
    ↓
Create VM
    ↓
Boot VM
    ↓
Wait for SSH readiness
    ↓
Execute provisioning script
    ↓
Configure service
    ↓
Continue to next VM
    ↓
Complete stack
```

The provisioning order follows the dependency structure of the application environment.

For the implementation details:

**[Implementation →](docs/implementation.md)**

---

## Validation

The completed stack was validated through the application itself.

### Frontend / Application

Accessing the web tier validates the path:

```text
Browser
   ↓
Nginx
   ↓
Tomcat
   ↓
VProfile application
```

### Database

Successful application login validates the application's connectivity to MySQL/MariaDB.

### RabbitMQ

The application's RabbitMQ validation function confirms connectivity to the message broker.

### Memcached

The cache validation demonstrates that application data can be retrieved from the database and subsequently served from the cache.

The validation model therefore checks the stack through real application interactions rather than simply checking whether individual services are running.

**[Validation →](docs/validation.md)**

---

## Environment Lifecycle

The automated environment supports the following lifecycle:

```text
vagrant up
    ↓
Running + Provisioned
    │
    ├── vagrant halt
    │       ↓
    │   Powered Off
    │       ↓
    │   vagrant up
    │       ↓
    │   Running
    │
    └── vagrant destroy -f
            ↓
        Clean Environment
            ↓
        vagrant up
            ↓
        Fresh Provisioning
```

Important Vagrant behavior:

- Initial `vagrant up` creates and provisions the VMs.
- `vagrant halt` powers off the environment while preserving its state.
- `vagrant up` on existing VMs powers them back on without automatically reprovisioning them.
- `vagrant provision` or `vagrant up --provision` can explicitly rerun provisioning.
- `vagrant destroy -f` removes the environment so it can be rebuilt from scratch.

---

## Key Engineering Concepts Demonstrated

This project provided practical experience with:

- Multi-tier application architecture
- Linux system administration
- Nginx reverse proxying
- Tomcat application deployment
- MySQL/MariaDB
- RabbitMQ
- Memcached
- Vagrant
- Bash scripting
- Shell provisioners
- Non-interactive automation
- Heredoc configuration generation
- Service management with systemd
- Hostname-based service discovery
- End-to-end application validation
- Repeatable environment provisioning

The broader engineering pattern is:

```text
Understand Manually
        ↓
Automate the Procedure
        ↓
Provision Repeatedly
        ↓
Validate the Result
        ↓
Destroy and Rebuild When Required
```

---

## Project Boundaries

This project demonstrates **local multi-VM application deployment and automated provisioning**.

It does not demonstrate:

- Production cloud infrastructure
- Terraform-based infrastructure provisioning
- Kubernetes orchestration
- CI/CD pipelines
- GitOps
- Production-grade high availability
- Production observability architecture
- Production database high availability
- Development of the VProfile application itself

These capabilities belong to later stages of the broader DevOps learning progression.

For the complete boundaries and possible evolution of this project:

**[Limitations & Future Work →](docs/limitations-and-future-work.md)**

---

## Technologies

| Category | Technologies |
|---|---|
| Virtualization | VirtualBox |
| VM Management | Vagrant |
| Operating Systems | Linux |
| Web Tier | Nginx |
| Application Tier | Apache Tomcat |
| Database | MySQL / MariaDB |
| Messaging | RabbitMQ |
| Caching | Memcached |
| Automation | Bash / Shell Provisioning |
| Build | Maven |
| Source Control | Git / GitHub |

---

## Repository Structure

```text
vprofile-multitier-provisioning/
│
├── README.md
├── architecture.png
│
├── infrastructure/
│   ├── Vagrantfile
│   └── scripts/
│       ├── mysql.sh
│       ├── memcache.sh
│       ├── rabbitmq.sh
│       ├── tomcat_ubuntu.sh
│       └── nginx.sh
│
├── docs/
│   ├── architecture.md
│   ├── implementation.md
│   ├── validation.md
│   └── limitations-and-future-work.md
│
├── evidence/
│   └── screenshots/
│
└── .gitignore
```

---

## Documentation

| Document | Purpose |
|---|---|
| [Architecture](docs/architecture.md) | System architecture, service relationships, traffic flow, and design boundaries |
| [Implementation](docs/implementation.md) | Provisioning, configuration, deployment, and automation implementation |
| [Validation](docs/validation.md) | End-to-end validation strategy and results |
| [Limitations & Future Work](docs/limitations-and-future-work.md) | Current boundaries and potential future evolution |

---

## Evidence

Evidence is intentionally kept limited to high-signal artifacts demonstrating:

- Environment architecture
- Successful provisioning
- Successful application deployment
- End-to-end validation

Only evidence from the completed project environment should be used as proof of execution.

**[View Evidence →](evidence/screenshots/)**

---

## Key Takeaway

The main engineering lesson from this project is the transition from **manual operational procedures to repeatable provisioning**.

Instead of repeatedly executing:

```text
SSH
  ↓
Install
  ↓
Configure
  ↓
Start
  ↓
Validate
```

the process becomes:

```text
Vagrantfile
     ↓
Provisioning Scripts
     ↓
vagrant up
     ↓
Complete Environment
```

The infrastructure definition and provisioning logic become executable, repeatable project artifacts rather than a sequence of commands that must be manually remembered.

---

## Project Status

**Completed — local multi-tier deployment and automated provisioning workflow.**

The project serves as a foundation for progressing toward more advanced DevOps practices such as containerization, cloud deployment, Infrastructure as Code, configuration management, orchestration, and CI/CD.
