VProfile Multi-Tier Application — Automated Provisioning

A hands-on DevOps project demonstrating the deployment, configuration, validation, and automated provisioning of a traditional multi-tier VProfile application across multiple Linux virtual machines using Vagrant and Bash.

«Application ownership: VProfile was used as the application workload for this project. The engineering work documented here focuses on the infrastructure environment, service configuration, application deployment, automation, and validation—not development of the VProfile application itself.»

---

Project Overview

The project implements a multi-tier application environment consisting of five virtual machines:

                         User
                          │
                       Browser
                          │
                          ▼
                  ┌──────────────┐
                  │    Nginx     │
                  │    web01     │
                  │    Port 80   │
                  └──────┬───────┘
                         │
                    Reverse Proxy
                         │
                         ▼
                  ┌──────────────┐
                  │    Tomcat    │
                  │    app01     │
                  │   Port 8080  │
                  └──────┬───────┘
                         │
              ┌──────────┼──────────┐
              │          │          │
              ▼          ▼          ▼
        ┌──────────┐ ┌──────────┐ ┌────────────┐
        │  MySQL   │ │ RabbitMQ │ │  Memcache  │
        │  db01    │ │  rmq01   │ │   mc01     │
        │  :3306   │ │  :5672   │ │  :11211    │
        └──────────┘ └──────────┘ └────────────┘

The environment was first understood and configured through manual provisioning and then converted into an automated provisioning workflow using Bash scripts and Vagrant provisioners.

---

Project Objective

The objective was to understand how a traditional multi-tier application is assembled from individual infrastructure and application services, and then convert the manual setup process into a repeatable automated provisioning workflow.

The project follows this progression:

Manual Provisioning
        ↓
Service Configuration
        ↓
Application Deployment
        ↓
End-to-End Validation
        ↓
Manual Environment Destroyed
        ↓
Provisioning Commands Encoded as Bash
        ↓
Vagrant Provisioners Added
        ↓
Automated Provisioning
        ↓
End-to-End Validation

The final workflow allows the environment to be created and provisioned through:

vagrant up

---

My Engineering Contribution

The project involved hands-on work across the infrastructure and deployment layers.

Infrastructure & Systems

- Created and managed a multi-VM Vagrant environment.
- Worked with Linux-based virtual machines.
- Installed and configured the required services.
- Managed services using "systemctl".
- Configured service connectivity between VMs.
- Worked with host-based service discovery.

Application Deployment

- Configured the Tomcat application server.
- Built and deployed the application artifact.
- Configured Nginx as the frontend reverse proxy.
- Connected the application tier with MySQL, RabbitMQ, and Memcache.

Automation

- Converted manual provisioning procedures into Bash-based provisioning scripts.
- Used Vagrant shell provisioners to execute the scripts automatically.
- Used shell variables for parameterization.
- Used non-interactive MySQL commands.
- Used Bash heredocs to generate configuration files without an interactive editor.
- Managed the provisioning lifecycle through Vagrant.

Validation

Validated the completed application stack through the application itself:

- Nginx/Tomcat application access
- MySQL-backed authentication
- RabbitMQ connectivity
- Memcache functionality
- End-to-end application flow

---

Architecture

The environment contains five primary service tiers:

VM| Service| Role
"web01"| Nginx| Frontend reverse proxy
"app01"| Tomcat| Application server
"db01"| MySQL/MariaDB| Persistent database
"rmq01"| RabbitMQ| Message broker
"mc01"| Memcache| In-memory cache

Request Flow

Browser
   │
   │ HTTP :80
   ▼
Nginx (web01)
   │
   │ proxy_pass
   ▼
Tomcat (app01)
   │
   ├──────────────► MySQL (db01)
   │
   ├──────────────► RabbitMQ (rmq01)
   │
   └──────────────► Memcache (mc01)

"Architecture" (architecture.png)

For the detailed architecture and service relationships:

"→ Architecture Documentation" (docs/architecture.md)

---

Manual → Automated Provisioning

A major engineering objective of the project was converting a manually executed deployment process into an automated provisioning workflow.

Manual Approach

vagrant up
    ↓
SSH into VM
    ↓
Run commands manually
    ↓
Configure files manually
    ↓
Start services
    ↓
Repeat for next VM

Automated Approach

vagrant up
    ↓
Vagrant reads Vagrantfile
    ↓
Creates VM
    ↓
Waits for VM to become accessible
    ↓
Runs associated provisioning script
    ↓
Moves to next VM
    ↓
Complete stack

Each VM has an associated provisioning script:

VM| Provisioning Script
"db01"| "mysql.sh"
"mc01"| "memcache.sh"
"rmq01"| "rabbitmq.sh"
"app01"| "tomcat_ubuntu.sh"
"web01"| "nginx.sh"

---

Key Automation Techniques

Bash Shebang

Each provisioning script uses:

#!/bin/bash

This identifies Bash as the script interpreter.

Parameterization

Shell variables are used where values need to be referenced multiple times.

Non-Interactive SQL

Instead of entering the MySQL shell interactively, SQL can be executed from Bash:

mysql -u username -p password -e "SQL QUERY"

Heredoc Configuration

Configuration files can be created directly from a script:

cat > /path/to/file <<EOT
configuration content
goes here
EOT

This removes the need for an interactive editor such as "vim" during automated provisioning.

Vagrant Provisioners

The Vagrantfile associates each VM with its provisioning script so that Vagrant can execute the required configuration automatically.

Detailed implementation information is available here:

"→ Implementation Documentation" (docs/implementation.md)

---

Provisioning Order

The automated environment is provisioned sequentially:

1. db01
   ↓
2. mc01
   ↓
3. rmq01
   ↓
4. app01
   ↓
5. web01

The ordering follows the dependency structure of the application environment.

The application tier depends on the backend services, while Nginx provides the frontend entry point to the Tomcat application tier.

---

Validation

The completed environment was validated through the application.

Application Access

The browser accesses the Nginx frontend:

Browser
   ↓
http://web01
   ↓
Nginx
   ↓
Tomcat
   ↓
VProfile application

Database Validation

The application login uses:

Username: admin_vp
Password: admin_vp

A successful login validates the application's connectivity to the database and the availability of the required database data.

RabbitMQ Validation

The application provides a RabbitMQ validation operation.

A successful result confirms that the application can communicate with RabbitMQ.

Memcache Validation

The application performs a cache test by retrieving user information.

The first request retrieves data from the database and inserts it into the cache.

A subsequent request for the same data demonstrates retrieval from the cache.

First request
    ↓
MySQL
    ↓
Application
    ↓
Memcache populated

Second request
    ↓
Memcache
    ↓
Application

Detailed validation methodology is documented here:

"→ Validation Documentation" (docs/validation.md)

---

Vagrant Environment Lifecycle

The automated environment supports the following lifecycle:

vagrant up
    │
    ▼
Create + Provision
    │
    ▼
Running
    │
    ├── vagrant halt
    │       ↓
    │   Powered Off
    │       │
    │       └── vagrant up
    │               ↓
    │            Running
    │
    └── vagrant destroy -f
            ↓
       Environment Removed
            │
            └── vagrant up
                    ↓
             Fresh Rebuild

Common Commands

# Create and provision the environment
vagrant up

# Check VM state
vagrant status

# Stop the VMs while preserving their state
vagrant halt

# Start existing VMs
vagrant up

# Re-run provisioning
vagrant provision

# Force provisioning during startup
vagrant up --provision

# Destroy the environment
vagrant destroy -f

A normal "vagrant up" on already-created VMs starts the existing machines without automatically re-running the provisioning scripts.

---

Engineering Lessons

The most important engineering lessons from this project are:

1. Understand before automating

The manual deployment process established how each service works and how the components depend on one another.

2. Automation should encode an understood process

The automated scripts capture the previously manual operations rather than introducing an entirely different deployment process.

3. Infrastructure configuration can be treated as code

The Vagrantfile and provisioning scripts describe how the environment should be assembled and configured.

4. Automation must be non-interactive

Commands that require human interaction must be adapted before they can reliably participate in automated provisioning.

5. Validation should test the complete data path

A service being "running" does not necessarily prove that the application can use it.

Application-level validation provides stronger evidence that:

Service
   +
Configuration
   +
Credentials
   +
Network Connectivity
   +
Application Integration

are working together.

---

Project Boundaries

This project demonstrates local multi-VM application deployment and automated provisioning.

It does not establish:

- production-grade cloud infrastructure
- AWS deployment
- Terraform-based infrastructure provisioning
- Ansible configuration management
- Kubernetes orchestration
- CI/CD implementation
- GitOps
- production-grade high availability
- production observability
- development of the VProfile application itself

The VProfile application was used as the workload for the infrastructure and deployment exercise.

---

Technologies

Infrastructure

- Vagrant
- VirtualBox
- Linux Virtual Machines

Web & Application

- Nginx
- Apache Tomcat
- Maven

Backend Services

- MySQL/MariaDB
- RabbitMQ
- Memcache

Automation

- Bash
- Vagrant Shell Provisioners
- systemd

Validation

- Browser-based application validation
- Service-level validation
- End-to-end integration validation

---

Documentation

Document| Purpose
"Architecture" (docs/architecture.md)| System architecture, service relationships, traffic flow, and boundaries
"Implementation" (docs/implementation.md)| Provisioning, configuration, deployment, automation, and lifecycle
"Validation" (docs/validation.md)| Validation strategy, checks, results, and evidence
"Limitations & Future Work" (docs/limitations-and-future-work.md)| Current boundaries and potential future evolution

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

---

Ownership & Attribution

This repository represents the infrastructure, deployment, configuration, automation, and validation work performed as part of the project.

The VProfile application itself was used as the project workload and should not be interpreted as an application developed by the repository owner.

Course-provided or third-party artifacts should only be redistributed when their ownership and redistribution rights permit it. Where applicable, such artifacts should be clearly attributed rather than represented as original work.

---

Project Outcome

The project demonstrates the progression from:

Manual Infrastructure Operations
              ↓
Understanding Service Dependencies
              ↓
Application Deployment
              ↓
End-to-End Validation
              ↓
Bash Automation
              ↓
Vagrant Provisioning
              ↓
Repeatable Environment Creation

The key engineering capability demonstrated is the ability to understand a traditional multi-tier application environment, configure its infrastructure and services, validate the complete application path, and encode the provisioning process into an automated workflow.

---

Next Evolution

The concepts established here provide a foundation for progressively more advanced DevOps implementations:

Vagrant + Bash
      ↓
Containerization
      ↓
Cloud Deployment
      ↓
Infrastructure as Code
      ↓
Configuration Management
      ↓
Container Orchestration
      ↓
CI/CD
      ↓
GitOps

Future implementations should be treated as separate engineering iterations rather than retroactively claiming capabilities that were not demonstrated by this project.

---

"← Back to Repository Root" (./)