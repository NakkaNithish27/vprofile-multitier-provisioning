# VProfile Validation

[← Back to README](../README.md) | [Architecture](architecture.md) | [Implementation](implementation.md)

## 1. Validation Overview

Validation proves that the complete VProfile multi-tier environment is not only running, but that the individual services are correctly connected and functioning together.

The validation is performed from the user-facing entry point downward:

```text
Browser
   ↓
Nginx
   ↓
Tomcat
   ↓
Backend Services
   ├── MySQL
   ├── RabbitMQ
   └── Memcached
```

The objective is not simply to check whether individual services are active.

```text
Service Running
      +
Service Reachable
      +
Service Correctly Configured
      +
Application Integration Working
      =
Validated Stack
```

The VProfile application provides built-in validation points that exercise each backend dependency.

## 2. Validation Strategy

The validation follows four major application-level tests:

```text
1. Application Page Loads
          ↓
2. Login Succeeds
          ↓
3. RabbitMQ Test Succeeds
          ↓
4. Memcached Two-Click Test Succeeds
```

| Test | Primary Validation |
|---|---|
| Application page loads | Nginx → Tomcat → Application |
| Application login | Tomcat → MySQL |
| RabbitMQ test | Tomcat → RabbitMQ |
| Memcached test | Tomcat → Memcached + MySQL |

## 3. Validation Environment

| VM | Service | Primary Validation |
|---|---|---|
| `web01` | Nginx | HTTP request and reverse proxy |
| `app01` | Tomcat | Application deployment and backend connectivity |
| `db01` | MariaDB | Application authentication/database access |
| `rmq01` | RabbitMQ | Message broker connectivity |
| `mc01` | Memcached | Cache read/write behavior |

The application is accessed through the web VM rather than directly through Tomcat.

## 4. Test 1 — Application Page Load

### Objective

Verify the complete frontend-to-application path:

```text
Browser
   ↓
web01 :80
   ↓
Nginx
   ↓
app01 :8080
   ↓
Tomcat
   ↓
VProfile
```

### Test Procedure

Obtain the IP address assigned to `web01` from the Vagrantfile.

For the documented environment:

```text
192.168.56.11
```

Open:

```text
http://192.168.56.11
```

Or explicitly:

```text
http://192.168.56.11:80
```

### Internal Request Flow

```text
Browser
   │
   │ HTTP :80
   ▼
Nginx / web01
   │
   │ proxy_pass
   ▼
Tomcat / app01 :8080
   │
   ▼
VProfile application
```

### Expected Result

The VProfile login page loads.

### What This Proves

A successful page load proves:

- Nginx is running.
- Nginx is listening on port `80`.
- Nginx can reach `app01:8080`.
- Tomcat is running.
- The VProfile application is deployed.
- The application can return a response through the configured reverse proxy.

```text
Page Loads
    =
Nginx ✓
Tomcat ✓
Application Deployment ✓
Nginx → Tomcat Connectivity ✓
```

## 5. Troubleshooting Test 1

### Check Nginx

```bash
vagrant ssh web01
sudo systemctl status nginx
```

Expected:

```text
active (running)
```

### Check Tomcat

```bash
vagrant ssh app01
sudo systemctl status tomcat
```

Expected:

```text
active (running)
```

### Check Hostname Resolution

From `web01`:

```bash
ping app01 -c 4
```

Also inspect:

```bash
cat /etc/hosts
```

### Check Nginx Configuration

```bash
cat /etc/nginx/sites-enabled/vproapp
```

The upstream should point to:

```text
app01:8080
```

## 6. Test 2 — Database Connectivity Through Login

### Objective

Verify that the application server can connect to MariaDB and retrieve user information from the `accounts` database.

```text
Browser
   ↓
Nginx
   ↓
Tomcat
   ↓
MySQL / MariaDB
```

### Test Procedure

On the VProfile login page, use:

```text
Username: admin_vp
Password: admin_vp
```

### Expected Result

Login succeeds and the application dashboard is displayed.

### What This Proves

Successful login proves:

- Tomcat can connect to `db01:3306`.
- The `accounts` database exists.
- Required user data exists.
- The database schema was successfully imported.
- The configured database credentials work.
- The application can execute the required database queries.

```text
Login Succeeds
      ↓
Tomcat → MySQL Connectivity ✓
      ↓
Database ✓
      ↓
Schema ✓
      ↓
Application Authentication ✓
```

## 7. Troubleshooting Test 2

### Check MariaDB

```bash
vagrant ssh db01
sudo systemctl status mariadb
```

Expected:

```text
active (running)
```

### Check Database Tables

```bash
mysql -u root -padmin123 accounts
```

Then:

```sql
show tables;
```

### Check Application Configuration

On `app01`, verify:

```text
src/main/resources/application.properties
```

The application configuration should reference:

```text
db01
```

and the appropriate database credentials.

### Check Firewall

On `db01`:

```bash
firewall-cmd --list-ports
```

Expected:

```text
3306/tcp
```

## 8. Test 3 — RabbitMQ Connectivity

### Objective

Verify that the application server can communicate with RabbitMQ.

```text
Browser
   ↓
Nginx
   ↓
Tomcat
   ↓
RabbitMQ / rmq01:5672
```

### Test Procedure

After successfully logging into the application as `admin_vp`, locate the **RabbitMQ** validation button on the application dashboard.

Click the button.

### Expected Result

The application displays a success message indicating that the queue connection was established.

### What This Proves

A successful RabbitMQ test proves:

- Tomcat can reach `rmq01`.
- RabbitMQ is running.
- Port `5672` is reachable.
- The configured RabbitMQ user exists.
- The RabbitMQ permissions are sufficient.
- The application can establish the required queue connection.

```text
RabbitMQ Test Succeeds
        ↓
Tomcat → rmq01:5672 ✓
        ↓
RabbitMQ Service ✓
        ↓
RabbitMQ Authentication/Permissions ✓
```

## 9. Troubleshooting Test 3

### Check RabbitMQ Service

```bash
vagrant ssh rmq01
sudo systemctl status rabbitmq-server
```

### Check RabbitMQ Users

```bash
sudo rabbitmqctl list_users
```

### Check Remote Access Configuration

Verify that RabbitMQ's configuration permits connections beyond its default loopback restrictions.

### Check Firewall

```bash
firewall-cmd --list-ports
```

Expected:

```text
5672/tcp
```

### Check Application Configuration

On `app01`, verify that the RabbitMQ hostname is:

```text
rmq01
```

and the configured port is:

```text
5672
```

## 10. Test 4 — Memcached Two-Click Test

### Objective

Verify that Memcached is not merely reachable, but that the application can actually use it for caching.

The test demonstrates cache-aside behavior:

```text
First request
    ↓
MySQL
    ↓
Return data
    ↓
Store data in Memcached

Second request
    ↓
Memcached
    ↓
Return cached data
```

## 11. Memcached Test — First Request

After logging into the application:

1. Click **All Users**.
2. Select a user, for example **Aejaaz Habeeb**.
3. Observe the resulting message.

Expected message:

```text
data is from db, and data is inserted in cache
```

### What This Proves

The first request proves:

```text
Application
     ↓
MySQL
     ↓
User data retrieved
     ↓
Memcached
     ↓
Data inserted into cache
```

This demonstrates that the application can retrieve data from the database and write the result to Memcached.

## 12. Memcached Test — Second Request

Repeat the same user request.

The application should now retrieve the previously cached data.

```text
First click
    ↓
data from DB
    ↓
insert into cache

Second click
    ↓
data from cache
```

### What This Proves

A successful second request proves:

- Tomcat can reach Memcached.
- Memcached accepts write operations.
- Cached data can be retrieved.
- The application is using the cache.
- Cache-aside behavior is functioning.

## 13. Troubleshooting Test 4

### Check Memcached

```bash
vagrant ssh mc01
sudo systemctl status memcached
```

### Check Bind Address

Inspect:

```text
/etc/sysconfig/memcached
```

The service should be configured to accept connections from the application VM rather than only:

```text
127.0.0.1
```

The documented setup changes the bind address to:

```text
0.0.0.0
```

### Check Firewall

```bash
firewall-cmd --list-ports
```

Expected:

```text
11211/tcp
```

### Check Application Configuration

On `app01`, verify that the application references:

```text
mc01
```

for the cache host.

## 14. Validation Matrix

| Layer | Test | Expected Result | What It Proves |
|---|---|---|---|
| Web | Open application URL | Login page loads | Nginx + Tomcat + application |
| Application → DB | Login | Dashboard loads | MySQL connectivity + schema |
| Application → MQ | RabbitMQ button | Success message | RabbitMQ connectivity |
| Application → Cache | First user request | Data from DB + inserted into cache | DB retrieval + cache write |
| Application → Cache | Second user request | Data from cache | Cache read |

## 15. Validation Dependency Map

```text
                 PAGE LOAD
                    │
                    ▼
          Nginx + Tomcat + App
                    │
                    ▼
                  LOGIN
                    │
                    ▼
              MySQL / DB
                    │
                    ▼
              RABBITMQ TEST
                    │
                    ▼
               RabbitMQ
                    │
                    ▼
            MEMCACHED TEST
                    │
                    ▼
              Memcached
```

This creates a progressive confidence model:

```text
Level 1
Frontend path works
        ↓
Level 2
Database integration works
        ↓
Level 3
Messaging integration works
        ↓
Level 4
Caching integration works
        ↓
Complete Stack Validated
```

## 16. Service-Level vs Application-Level Validation

### Service-Level Checks

```bash
systemctl status nginx
systemctl status tomcat
systemctl status mariadb
systemctl status rabbitmq-server
systemctl status memcached
```

These answer:

> Is the service running?

### Application-Level Checks

```text
Open application
      ↓
Login
      ↓
RabbitMQ test
      ↓
Memcached test
```

These answer:

> Can the application actually use the service?

A service being `active (running)` does not by itself prove that the application can reach and correctly use it.

## 17. End-to-End Validation Model

```text
                  USER
                    │
                    ▼
             ┌─────────────┐
             │    NGINX    │
             │   web01:80  │
             └──────┬──────┘
                    │
                    ▼
             ┌─────────────┐
             │   TOMCAT    │
             │ app01:8080  │
             └──────┬──────┘
                    │
          ┌─────────┼─────────┐
          │         │         │
          ▼         ▼         ▼
       MySQL    RabbitMQ   Memcached
       db01      rmq01       mc01
```

Validation travels from the user's perspective down through the dependency chain.

## 18. Failure Isolation Model

```text
Page fails
   ↓
Investigate Nginx / Tomcat / networking

Page works
   ↓
Login fails
   ↓
Investigate MySQL / application DB config

Login works
   ↓
RabbitMQ fails
   ↓
Investigate RabbitMQ / credentials / firewall

RabbitMQ works
   ↓
Cache test fails
   ↓
Investigate Memcached / bind address / firewall
```

This makes the validation process useful not only for proving success, but also for narrowing down failures.

## 19. Validation Evidence

Evidence should focus on high-signal proof rather than screenshots of every command.

Recommended evidence categories:

### Application Access

Screenshot showing the VProfile login page.

### Successful Login

Screenshot showing the application dashboard.

### RabbitMQ Validation

Screenshot showing the RabbitMQ success message.

### Memcached Validation

Screenshots showing the two states:

```text
First request → data from DB / inserted into cache
Second request → data from cache
```

### Environment Status

Optional evidence can show the five VMs running and the relevant services active.

The repository's `evidence/screenshots/` directory should contain only evidence actually captured from the completed environment.

## 20. Validation Sequence

The complete validation procedure can be remembered as:

```text
1. Find web01 IP
       ↓
2. Open VProfile
       ↓
3. Verify login page
       ↓
4. Login
       ↓
5. Verify RabbitMQ
       ↓
6. Run Memcached two-click test
       ↓
7. Confirm all services
```

## 21. Final Validation Result

The VProfile stack is considered validated when all four application-level tests succeed:

```text
┌─────────────────────────────────────┐
│  VPROFILE VALIDATION                │
├─────────────────────────────────────┤
│                                     │
│  ✓ Application page loads           │
│  ✓ Login succeeds                   │
│  ✓ RabbitMQ test succeeds           │
│  ✓ Memcached cache test succeeds    │
│                                     │
│  ✓ Complete stack validated         │
│                                     │
└─────────────────────────────────────┘
```

| Service | Validation Method | Result |
|---|---|---|
| Nginx | Application page loads | Passed |
| Tomcat | Application renders | Passed |
| MySQL | Login succeeds | Passed |
| RabbitMQ | Queue connection button | Passed |
| Memcached | Two-click cache test | Passed |

## 22. Validation → Automation Transition

Validation is the boundary between the manual and automated stages of the project.

```text
Manual Provisioning
       ↓
Manual Validation
       ↓
Prove Architecture Works
       ↓
Destroy Environment
       ↓
Automate Provisioning
       ↓
Recreate Environment
       ↓
Validate Again
```

The purpose of validating before automation is important:

> **Automation should reproduce a known-working deployment rather than automate an unverified configuration.**

## 23. Validation Mental Model

```text
PAGE
 ↓
Nginx → Tomcat → App
 ↓
LOGIN
 ↓
Tomcat → MySQL
 ↓
RABBITMQ BUTTON
 ↓
Tomcat → RabbitMQ
 ↓
CACHE TEST
 ↓
Tomcat → Memcached
 ↓
COMPLETE STACK
```

Or:

```text
Page → Login → RabbitMQ → Cache
  ↓       ↓         ↓        ↓
 Web     DB        MQ       Cache
  ↓       ↓         ↓        ↓
       Complete VProfile Stack
              ✓
```

### Core Validation Principle

> **Do not stop at “the services are running.” Prove that the application can actually traverse the architecture and use each dependency successfully.**

[← Back to README](../README.md) | [Architecture](architecture.md) | [Implementation](implementation.md)
