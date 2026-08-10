# VProfile Multi-Tier Application — Automated Provisioning

A hands-on DevOps project demonstrating the deployment, configuration, validation, and automated provisioning of a multi-tier VProfile application across multiple Linux virtual machines using **Vagrant and Bash**.

---

## Overview

This project deploys the VProfile application as a traditional multi-tier environment consisting of:

- **Nginx** — reverse proxy / frontend
- **Tomcat** — application server
- **MySQL/MariaDB** — relational database
- **RabbitMQ** — message broker
- **Memcache** — caching layer

Each service runs on a dedicated virtual machine.

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
Automated Stack Deployment

---

Application Ownership Boundary

The VProfile application was used as the workload for this project.

The application itself was not developed as part of this project.

The engineering work represented here focuses on the environment around the application, including:

- provisioning the virtual machines
- configuring the required services
- deploying the application
- configuring service connectivity
- configuring Nginx as the frontend reverse proxy
- validating application-to-backend connectivity
- converting the manual provisioning process into an automated Vagrant/Bash workflow

Course-provided or third-party application/source artifacts are not presented as original application development work.

---

Architecture

The environment consists of five virtual machines:

                         User
                          │
                          ▼
                    ┌───────────┐
                    │  Nginx    │
                    │  web01    │
                    │   :80     │
                    └─────┬─────┘
                          │
                     Reverse Proxy
                          │
                          ▼
                    ┌───────────┐
                    │  Tomcat   │
                    │  app01    │
                    │   :8080   │
                    └─────┬─────┘
                          │
              ┌───────────┼───────────┐
              │           │           │
              ▼           ▼           ▼
        ┌──────────┐ ┌──────────┐ ┌──────────┐
        │  MySQL   │ │ RabbitMQ │ │ Memcache │
        │  db01    │ │  rmq01   │ │  mc01    │
        │  :3306   │ │  :5672   │ │  :11211  │
        └──────────┘ └──────────┘ └──────────┘

Request Flow

Browser
   │
   │ HTTP :80
   ▼
Nginx
   │
   │ proxy_pass
   ▼
Tomcat :8080
   │
   ├──► MySQL
   │
   ├──► RabbitMQ
   │
   └──► Memcache

See the detailed architecture:

"Architecture" (docs/architecture.md)

---

My Engineering Contribution

The practical work covered the following engineering activities:

Infrastructure

- Created and managed a multi-VM Vagrant environment.
- Worked with separate VMs for each application tier/service.
- Configured hostname-based communication between services.

Linux Service Administration

- Installed required packages.
- Started and enabled services using "systemctl".
- Configured service files and service settings.
- Configured network-accessible service endpoints where required.
- Worked with Linux configuration files and firewall settings.

Application Deployment

- Configured the Tomcat application server.
- Built and deployed the application artifact.
- Configured the application to communicate with backend services.

Reverse Proxy

Configured Nginx on "web01" to:

- listen on HTTP port "80"
- receive requests from users
- forward requests to Tomcat on "app01:8080"

Automation

Converted the manual provisioning workflow into automated Bash scripts and connected those scripts to Vagrant provisioning.

The automated workflow uses:

Vagrantfile
     │
     ├── db01      → mysql.sh
     ├── mc01      → memcache.sh
     ├── rmq01     → rabbitmq.sh
     ├── app01     → tomcat_ubuntu.sh
     └── web01     → nginx.sh

The goal is to make the environment reproducible through:

vagrant up

---

Manual → Automated Provisioning

The automation stage does not introduce a completely different deployment process.

Instead, the manual process is encoded into executable scripts.

Manual

SSH into VM
    ↓
Run commands manually
    ↓
Edit configuration files
    ↓
Start services
    ↓
Deploy application
    ↓
Repeat for every VM

Automated

vagrant up
    ↓
Vagrant creates VMs
    ↓
VMs become SSH-accessible
    ↓
Vagrant executes provisioning scripts
    ↓
Services are configured automatically
    ↓
Complete stack becomes available

---

Key Automation Techniques

Bash Shebang

Each provisioning script uses:

#!/bin/bash

This specifies Bash as the script interpreter.

Variables

Variables are used to avoid repeatedly hardcoding values within scripts.

Non-Interactive SQL

SQL commands can be executed directly from Bash using:

mysql -u username -p password -e "SQL QUERY"

This removes the need for an interactive MySQL shell during provisioning.

Heredoc Configuration

Multi-line configuration files can be generated without an interactive editor:

cat > /path/to/file <<EOT
configuration content
goes here
EOT

This approach is used for configuration files that previously required manual editing.

---

Automated Provisioning Flow

The automated environment is provisioned sequentially:

vagrant up
    │
    ▼
db01
mysql.sh
    │
    ▼
memcache
memcache.sh
    │
    ▼
rmq01
rabbitmq.sh
    │
    ▼
app01
tomcat_ubuntu.sh
    │
    ▼
web01
nginx.sh
    │
    ▼
Complete Stack

The order follows the service dependency model used by the project.

---

Validation

The completed environment is validated through the application rather than by checking individual services in isolation.

Application Access

The user accesses the Nginx frontend.

Browser
   ↓
Nginx
   ↓
Tomcat
   ↓
VProfile Application

Database Validation

The application login is used to verify database connectivity.

admin_vp / admin_vp

A successful login demonstrates that the application can communicate with the database and authenticate against the provisioned data.

RabbitMQ Validation

The application's RabbitMQ validation functionality is used to confirm connectivity to the message broker.

Memcache Validation

The application is used to verify the caching workflow.

The first request retrieves data from the database and inserts it into the cache.

A subsequent request for the same data demonstrates that the cached result can be retrieved.

Validation Model

Page loads
    ↓
Nginx + Tomcat + Application
    ↓
Login succeeds
    ↓
MySQL validated
    ↓
RabbitMQ test succeeds
    ↓
RabbitMQ validated
    ↓
Cache test succeeds
    ↓
Memcache validated

See the detailed validation process:

"Validation" (docs/validation.md)

---

Vagrant Lifecycle

The automated environment supports the following lifecycle:

vagrant up
    ↓
Create + provision
    ↓
Running
    │
    ├── vagrant halt
    │        ↓
    │    Powered Off
    │        ↓
    │    vagrant up
    │        ↓
    │    Running
    │
    └── vagrant destroy -f
             ↓
        Environment Removed
             ↓
          vagrant up
             ↓
        Fresh Provisioning

For an existing VM, "vagrant up" powers the VM back on without automatically repeating the original provisioning.

Provisioning can be explicitly triggered with:

vagrant provision

or:

vagrant up --provision

---

Key Commands

Create and provision the environment

vagrant up

Check VM status

vagrant status

Stop the environment

vagrant halt

Re-run provisioning

vagrant provision

Start and force provisioning

vagrant up --provision

Destroy the environment

vagrant destroy -f

---

Repository Structure

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

Documentation

- "Architecture" (docs/architecture.md) — system architecture, component relationships, traffic flow, and service dependencies.
- "Implementation" (docs/implementation.md) — provisioning, configuration, deployment, and automation implementation.
- "Validation" (docs/validation.md) — validation strategy and results.
- "Limitations & Future Work" (docs/limitations-and-future-work.md) — project boundaries, limitations, and possible future evolution.

---

Technologies

Area| Technology
Virtualization| Vagrant / VirtualBox
Operating Systems| Linux
Web / Reverse Proxy| Nginx
Application Server| Apache Tomcat
Database| MySQL / MariaDB
Message Broker| RabbitMQ
Cache| Memcached
Automation| Bash
Provisioning| Vagrant Shell Provisioner
Application Build| Maven
Source Control| Git

---

Project Boundaries

This project demonstrates traditional multi-tier application deployment and local infrastructure provisioning.

It does not demonstrate:

- production-grade cloud infrastructure
- Terraform-based cloud provisioning
- Ansible configuration management
- Kubernetes orchestration
- CI/CD pipelines
- GitOps
- production-grade high availability
- production observability architecture
- development of the VProfile application itself

These capabilities belong to later stages of the DevOps learning and project progression.

---

What This Project Demonstrates

The strongest engineering capability demonstrated by this project is the progression from:

Manual Operations
       ↓
Understanding the Deployment Process
       ↓
Encoding the Process in Bash
       ↓
Connecting Scripts to Vagrant
       ↓
Automated Provisioning
       ↓
Repeatable Environment
       ↓
End-to-End Validation

The central engineering lesson is:

«Understand the manual process first, then automate it into a repeatable workflow.»

---

Evidence

Evidence included in this repository should represent the completed environment and personally performed work.

High-signal evidence includes:

- architecture diagram
- successful automated provisioning
- successful application deployment
- end-to-end application validation

Course screenshots, lecture material, and copied course documentation are not used as evidence of personal execution.

---

Future Evolution

The concepts demonstrated here provide a foundation for progressively more advanced DevOps implementations:

Vagrant + Bash
      ↓
Containerization
      ↓
AWS Deployment
      ↓
Terraform
      ↓
Ansible
      ↓
Kubernetes
      ↓
CI/CD
      ↓
GitOps

Future capabilities listed above are not part of this implementation.

---

License / Source Attribution

The VProfile application and any course-provided artifacts used during the practical are not represented as original application-development work.

Before redistributing any supplied source code or provisioning artifacts, verify the applicable ownership and redistribution rights.