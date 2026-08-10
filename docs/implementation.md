# VProfile Implementation

[← Back to README](../README.md) | [Architecture](architecture.md)

## 1. Implementation Overview

The VProfile environment was implemented in two stages:

```text
Stage 1
Manual Provisioning
       ↓
Stage 2
Automated Provisioning
```

The first stage established an understanding of how each component works and how the services communicate.

The second stage converted those manual procedures into Bash provisioning scripts and connected those scripts to Vagrant.

The resulting implementation can be summarized as:

```text
Manual Commands
      ↓
Bash Scripts
      ↓
Vagrant Provisioners
      ↓
vagrant up
      ↓
Complete Multi-VM Environment
```

The architecture itself remains unchanged between the two stages. The deployment mechanism changes from interactive manual operations to automated provisioning.

## 2. Implementation Architecture

The environment consists of five virtual machines:

```text
Vagrantfile
    │
    ├── db01   → MySQL/MariaDB
    ├── mc01   → Memcached
    ├── rmq01  → RabbitMQ
    ├── app01  → Tomcat + VProfile
    └── web01  → Nginx
```

The provisioning relationship is:

```text
Vagrantfile
    │
    ├── db01   → mysql.sh
    ├── mc01   → memcache.sh
    ├── rmq01  → rabbitmq.sh
    ├── app01  → tomcat_ubuntu.sh
    └── web01  → nginx.sh
```

Each VM receives a service-specific shell script.

## 3. Manual Provisioning Phase

Before automation, each VM was configured manually.

The general process was:

```text
vagrant up
      ↓
vagrant ssh <vm>
      ↓
sudo -i
      ↓
Update operating system
      ↓
Install packages
      ↓
Configure service
      ↓
Start / enable service
      ↓
Configure firewall
      ↓
Validate
```

The manual process established:

- required packages
- service configuration
- required ports
- service communication
- important configuration files
- validation requirements
- automation requirements

The manual phase provided the operational knowledge that was later encoded into automation.

## 4. Database Implementation — `db01`

The database VM provides the persistent data layer.

```text
db01
 │
 └── MariaDB
       │
       └── accounts database
```

The implementation follows:

```text
OS preparation
      ↓
Install MariaDB
      ↓
Start service
      ↓
Enable service
      ↓
Secure installation
      ↓
Create database/user
      ↓
Grant remote access
      ↓
Import schema
      ↓
Configure firewall
      ↓
Validate
```

The database is configured to accept remote connections because Tomcat runs on a separate VM.

The application therefore connects through:

```text
app01
   │
   │ db01:3306
   ▼
MariaDB
```

The database schema is deployed from the supplied VProfile project data.

## 5. Memcached Implementation — `mc01`

The cache VM provides the application's caching layer.

```text
mc01
 │
 └── Memcached
       │
       └── TCP 11211
```

The implementation includes:

```text
Install Memcached
      ↓
Start service
      ↓
Enable service
      ↓
Configure bind address
      ↓
Restart service
      ↓
Configure firewall
```

Memcached must be reachable from the application VM rather than listening only on localhost.

Conceptually:

```text
127.0.0.1
    ↓
0.0.0.0
```

This allows:

```text
app01
  │
  │ mc01:11211
  ▼
Memcached
```

## 6. RabbitMQ Implementation — `rmq01`

RabbitMQ provides the messaging layer.

```text
rmq01
 │
 └── RabbitMQ
       │
       └── TCP 5672
```

The implementation includes:

```text
Install RabbitMQ
      ↓
Configure RabbitMQ
      ↓
Configure remote access
      ↓
Create application/test user
      ↓
Set permissions
      ↓
Start service
      ↓
Enable service
      ↓
Configure firewall
```

The application server communicates with:

```text
app01
  │
  │ rmq01:5672
  ▼
RabbitMQ
```

## 7. Application Server Implementation — `app01`

The application server contains:

1. the Tomcat runtime
2. the VProfile application artifact

The implementation therefore has two distinct phases:

```text
Tomcat Service
      +
VProfile Application
```

### 7.1 Operating System Preparation

The application VM is prepared with:

```text
System update
      ↓
EPEL repository
      ↓
Java 17
      ↓
Git
      ↓
wget
```

Java is required for Tomcat and Maven.

Git is required to obtain the VProfile source.

`wget` is used for downloading binary packages.

### 7.2 Tomcat Installation

Tomcat is installed from its binary distribution.

The general pattern is:

```text
Download Tomcat archive
      ↓
Extract
      ↓
Install under /usr/local/tomcat
      ↓
Create tomcat user
      ↓
Set ownership
      ↓
Create systemd service
      ↓
daemon-reload
      ↓
Start Tomcat
      ↓
Enable Tomcat
```

### 7.3 Tomcat Systemd Integration

Tomcat is converted into a managed Linux service through:

```text
/etc/systemd/system/tomcat.service
```

The implementation then performs:

```bash
systemctl daemon-reload
systemctl start tomcat
systemctl enable tomcat
```

The operational model is:

```text
systemctl
    │
    ▼
tomcat.service
    │
    ▼
Tomcat
```

## 8. Application Build and Deployment

Once Tomcat is prepared, the VProfile application is built and deployed.

```text
VProfile source
      ↓
application.properties
      ↓
Maven build
      ↓
WAR artifact
      ↓
Tomcat webapps
      ↓
VProfile application
```

### 8.1 Source Code

The VProfile project is cloned from the appropriate project branch.

The source material identifies the `local` branch as the branch used for this environment.

### 8.2 Application Configuration

The important backend relationships are:

```text
application.properties
        │
        ├── Database → db01
        ├── Cache    → mc01
        └── RabbitMQ → rmq01
```

The values must correspond to the service hostnames and ports created by the multi-VM environment.

### 8.3 Maven Build

The project is built using Maven:

```bash
/usr/local/maven3.9/bin/mvn install
```

The build produces a WAR artifact.

```text
Java Source
     ↓
Maven
     ↓
vprofile-v2.war
```

### 8.4 WAR Deployment

The deployment process is:

```text
Stop Tomcat
      ↓
Remove existing ROOT application
      ↓
Copy WAR
      ↓
Rename to ROOT.war
      ↓
Set ownership
      ↓
Start / restart Tomcat
```

The artifact is placed under:

```text
/usr/local/tomcat/webapps/
```

Using:

```text
ROOT.war
```

causes the application to be served at the root URL path.

## 9. Nginx Implementation — `web01`

The frontend VM runs Ubuntu and therefore uses `apt`.

The implementation is:

```text
Install Nginx
      ↓
Create VProfile site configuration
      ↓
Configure upstream → app01:8080
      ↓
Configure proxy_pass
      ↓
Disable default site
      ↓
Enable VProfile site
      ↓
Restart Nginx
```

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

The configuration is stored under:

```text
/etc/nginx/sites-available/vproapp
```

and activated through a symbolic link under:

```text
/etc/nginx/sites-enabled/
```

The default Nginx site is removed so that the VProfile configuration becomes the active site.

## 10. Service Connectivity

The environment uses hostnames rather than hardcoded service IPs.

```text
Nginx
 │
 └──► app01:8080

Tomcat
 ├──► db01:3306
 ├──► mc01:11211
 └──► rmq01:5672
```

The Vagrant Hostmanager plugin populates hostname mappings so these names resolve between the VMs.

The application configuration can therefore reference:

```text
db01
mc01
rmq01
```

and Nginx can reference:

```text
app01
```

## 11. Firewall Configuration

The service VMs expose the ports required for their respective roles.

```text
Nginx       → 80/tcp
Tomcat      → 8080/tcp
MySQL       → 3306/tcp
RabbitMQ    → 5672/tcp
Memcached   → 11211/tcp
```

The CentOS services use `firewalld`.

The general pattern is:

```text
Start firewalld
      ↓
Enable firewalld
      ↓
Add service port
      ↓
Make rule persistent
      ↓
Reload / apply configuration
```

## 12. Manual Validation Before Automation

After all five VMs were configured, the complete stack was validated.

```text
Browser
   ↓
Nginx
   ↓
Tomcat
   ↓
VProfile
   │
   ├──► MySQL
   ├──► RabbitMQ
   └──► Memcached
```

The validation confirmed:

```text
Nginx       → page loads
Tomcat      → application renders
MySQL       → login succeeds
RabbitMQ    → connection test succeeds
Memcached   → cache test succeeds
```

This validation established that the manual implementation worked before converting it into automation.

## 13. Destroy Before Automation

Once the manually provisioned environment was validated, the VMs were destroyed.

The purpose was to create a clean starting point for testing whether the complete environment could be recreated automatically.

```text
Manual Setup
      ↓
Validation
      ↓
Destroy
      ↓
Automation
      ↓
Fresh Provisioning
```

## 14. Automation Strategy

The automation phase does not introduce a different architecture.

Instead, it captures the existing manual procedures as executable scripts.

```text
Manual Commands
      ↓
Capture in Bash
      ↓
Make Non-Interactive
      ↓
Attach to Vagrant VM
      ↓
Execute Automatically
```

The automation preserves the knowledge gained during manual setup.

## 15. Script Structure

Each provisioning script follows the general pattern:

```bash
#!/bin/bash

VARIABLE=value

dnf install <package> -y

systemctl start <service>
systemctl enable <service>

sed -i 's/old/new/' <file>

cat > <file> <<EOT
...
EOT

mysql -u username -p password -e "SQL QUERY"
```

The major automation adaptations are:

| Manual Approach | Automated Approach |
|---|---|
| Hardcoded values | Variables |
| Interactive SQL | `mysql -e` |
| Edit files with `vim` | Heredoc |
| Human executes commands | Vagrant provisioner |

## 16. Variables

Manual commands often contain repeated hardcoded values.

Automation replaces these with variables.

Conceptually:

```bash
DB_USER=admin
DB_PASS=admin123
DB_NAME=accounts
```

The script can then reference the variables throughout its execution.

## 17. Non-Interactive Package Installation

Interactive commands are unsuitable for unattended provisioning.

Package installation therefore uses non-interactive flags such as:

```bash
dnf install <package> -y
```

The `-y` option automatically confirms package installation prompts.

## 18. Non-Interactive SQL

Manual database configuration can involve entering the MySQL shell:

```text
mysql
   ↓
type SQL
   ↓
quit
```

Automation replaces this with:

```bash
mysql -u username -p password -e "SQL QUERY"
```

The `-e` option tells the MySQL client to execute the supplied SQL without entering an interactive shell.

## 19. Heredoc File Generation

Manual provisioning frequently uses an editor:

```text
vim
   ↓
insert configuration
   ↓
save
   ↓
exit
```

Automation replaces this with a heredoc:

```bash
cat > /path/to/file <<EOT
configuration content
configuration content
configuration content
EOT
```

This is useful for generating:

- systemd unit files
- Nginx configurations
- service configuration files
- other multi-line configuration files

## 20. Provisioning Script Responsibilities

### `mysql.sh`

```text
Variables
   ↓
Install MariaDB
   ↓
Start / enable service
   ↓
Obtain required project data
   ↓
Execute SQL non-interactively
   ↓
Configure database access
   ↓
Configure firewall
```

### `memcache.sh`

```text
Install EPEL
   ↓
Install Memcached
   ↓
Start / enable service
   ↓
Change bind address
   ↓
Restart service
   ↓
Configure firewall
```

### `rabbitmq.sh`

```text
Install RabbitMQ
   ↓
Configure RabbitMQ
   ↓
Configure users/access
   ↓
Start / enable service
   ↓
Configure firewall
```

### `tomcat_ubuntu.sh`

Responsibilities include:

```text
Install dependencies
      ↓
Download Tomcat
      ↓
Create Tomcat user
      ↓
Install Tomcat
      ↓
Create systemd unit
      ↓
daemon-reload
      ↓
Start / enable Tomcat
      ↓
Obtain VProfile source
      ↓
Configure application
      ↓
Build with Maven
      ↓
Deploy WAR
```

### `nginx.sh`

```text
Install Nginx
      ↓
Generate configuration
      ↓
Disable default site
      ↓
Enable VProfile site
      ↓
Start / enable Nginx
```

## 21. Vagrant Provisioner Wiring

The Vagrantfile connects each VM to its provisioning script:

```text
Vagrantfile
    │
    ├── db01   → mysql.sh
    ├── mc01   → memcache.sh
    ├── rmq01  → rabbitmq.sh
    ├── app01  → tomcat_ubuntu.sh
    └── web01  → nginx.sh
```

The Vagrantfile becomes the orchestration point for the complete environment.

## 22. Automated Execution

The operational entry point is:

```bash
vagrant up
```

The high-level execution sequence is:

```text
vagrant up
      ↓
Read Vagrantfile
      ↓
Create VM
      ↓
Boot VM
      ↓
Wait for SSH readiness
      ↓
Execute provisioner
      ↓
Service configured
      ↓
Move to next VM
      ↓
Complete stack
```

No manual SSH session is required during the automated provisioning flow.

## 23. Automated Provisioning Order

The provisioning sequence follows the service dependency structure:

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

The reasoning is:

```text
Backend services
      ↓
Application server
      ↓
Frontend reverse proxy
```

## 24. Vagrant Lifecycle

### Check VM state

```bash
vagrant status
```

### Start environment

```bash
vagrant up
```

### Stop environment

```bash
vagrant halt
```

### Re-run provisioning

```bash
vagrant provision
```

### Start and explicitly provision

```bash
vagrant up --provision
```

### Destroy environment

```bash
vagrant destroy -f
```

The lifecycle model is:

```text
Create
  ↓
Provision
  ↓
Run
  ↓
Halt
  ↓
Resume
  ↓
Reprovision when required
  ↓
Destroy
  ↓
Rebuild
```

## 25. Reprovisioning vs Rebuilding

### Reprovision

```bash
vagrant provision
```

Runs the provisioning scripts again against the existing VMs.

### Rebuild

```bash
vagrant destroy -f
vagrant up
```

Removes the existing VMs and creates a fresh environment.

This tests whether the provisioning definition is sufficient to recreate the environment from scratch.

## 26. Implementation Validation Points

### Database

```text
MariaDB service
      ↓
Database exists
      ↓
Users/grants exist
      ↓
Schema imported
```

### Memcached

```text
Service active
      ↓
Port 11211 available
      ↓
Remote binding configured
```

### RabbitMQ

```text
Service active
      ↓
User/configuration exists
      ↓
Port 5672 available
```

### Tomcat

```text
Tomcat service active
      ↓
Tomcat files owned correctly
      ↓
WAR artifact exists
      ↓
ROOT.war deployed
```

### Nginx

```text
Nginx active
      ↓
vproapp configuration exists
      ↓
Default site disabled
      ↓
app01:8080 configured as upstream
```

## 27. Implementation Flow

```text
Create VMs
    ↓
Configure backend services
    ↓
Configure application server
    ↓
Build + deploy application
    ↓
Configure Nginx
    ↓
Validate end-to-end
    ↓
Destroy environment
    ↓
Encode manual procedures as Bash
    ↓
Wire scripts into Vagrant
    ↓
vagrant up
    ↓
Automated provisioning
    ↓
Validate again
```

## 28. Manual vs Automated Implementation

| Area | Manual | Automated |
|---|---|---|
| VM creation | Vagrant | Vagrant |
| SSH | Manual | Vagrant provisioner |
| Package installation | Typed commands | Bash |
| Service configuration | Typed commands/editor | Bash |
| SQL operations | Interactive | `mysql -e` |
| Configuration files | `vim` | Heredoc |
| Service startup | Manual | Script |
| Firewall configuration | Manual | Script |
| Application build | Manual | Script |
| Application deployment | Manual | Script |
| Full environment setup | Multiple steps | `vagrant up` |

The important point is that automation encodes the underlying implementation knowledge into executable procedures.

## 29. Engineering Patterns

| Pattern | Implementation |
|---|---|
| Manual → automation | Existing commands converted into scripts |
| Parameterization | Variables replace repeated hardcoded values |
| Non-interactive execution | `-y`, `mysql -e`, heredocs |
| Service management | `systemctl start/enable` |
| Configuration generation | Heredoc |
| Dependency ordering | Backend → app → frontend |
| Service discovery | Hostnames through `/etc/hosts` |
| Application deployment | Maven → WAR → Tomcat |
| Reverse proxy | Nginx → Tomcat |
| Repeatable environment | Vagrant + provisioning scripts |
| Lifecycle management | up / halt / provision / destroy |

## 30. Implementation Boundary

This implementation demonstrates:

- Local VM provisioning
- Linux service configuration
- Bash automation
- Vagrant shell provisioners
- Application deployment
- Multi-service connectivity
- Repeatable environment creation

It does not establish:

- Terraform-based cloud IaC
- AWS infrastructure
- Kubernetes
- CI/CD
- GitOps
- Production-grade HA
- Production observability

The Vagrant/Bash implementation should therefore be understood as a **foundational automation implementation**, not as a production cloud platform.

## 31. Final Implementation Mental Model

```text
MANUAL
──────
SSH → Install → Configure → Start → Validate
                    │
                    ▼
              Understand Flow
                    │
                    ▼
AUTOMATE
────────
Bash Script → Vagrant Provisioner → vagrant up
                    │
                    ▼
              Repeatable Stack
```

The four key automation transformations are:

```text
Hardcoded values
      ↓
Variables

Interactive SQL
      ↓
mysql -e

Interactive file editing
      ↓
Heredoc

Manual execution
      ↓
Vagrant provisioner
```

The final operational model is:

```text
Vagrantfile
     │
     ├── mysql.sh
     ├── memcache.sh
     ├── rabbitmq.sh
     ├── tomcat_ubuntu.sh
     └── nginx.sh
             │
             ▼
         vagrant up
             │
             ▼
     Complete VProfile Stack
```

> **Understand the manual deployment flow first, encode that flow into non-interactive scripts, connect those scripts to the infrastructure definition, and use a single entry point to recreate the environment.**

[← Back to README](../README.md) | [Architecture](architecture.md) | [Validation](validation.md)
