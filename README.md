# AWS Route 53 — DNS Routing and Failover

## 1. What is Amazon Route 53?

**Amazon Route 53** is an AWS managed DNS service.

DNS stands for:

**Domain Name System**

DNS converts domain names into destinations that users' applications can connect to.

Example:

```text
User
  |
  | https://example.com
  v
Route 53
  |
  v
Application
```

---

# 2. Project Overview

This project demonstrates how **Route 53 DNS routing and failover** can be used with:

* Amazon Route 53
* Elastic Load Balancer (ELB)
* Amazon EC2
* Amazon S3
* Health Checks
* VPC
* Security Groups

The practice architecture uses the **Mumbai AWS Region**.

Example domain used in the practice:

```text
sai.com
```

---

# 3. Architecture

```text
                         USER
                           |
                           v
                    +-------------+
                    |  Route 53   |
                    |     DNS     |
                    +------+------+
                           |
                     Health Check
                           |
              +------------+------------+
              |                         |
           HEALTHY                    UNHEALTHY
              |                         |
              v                         v
       +-------------+           +-------------+
       |     ELB     |           |     S3      |
       |   Primary   |           |  Secondary  |
       +------+------+           +------+------+
              |                         |
       +------+------+            maintenance
       |             |              .html
       v             v
     EC2-1         EC2-2
       |             |
       +------+------+
              |
        Web Application
```

---

# 4. Components Used

## Route 53

Used as the DNS service.

Responsibilities:

* DNS resolution
* Traffic routing
* Health checks
* Failover routing

---

## Elastic Load Balancer

The ELB distributes incoming requests across multiple EC2 instances.

Example:

```text
Route 53
    |
    v
   ELB
  /   \
 v     v
EC2-1 EC2-2
```

This prevents all application traffic from going to a single EC2 instance.

---

## EC2 Instances

EC2 instances host the web application.

Example:

```text
ELB
 |
 +---- EC2-1
 |
 +---- EC2-2
```

Multiple EC2 instances can provide application redundancy.

---

## Amazon S3

An S3 bucket can be used to host a static maintenance page.

Example:

```text
S3 Bucket
    |
    +-- maintenance.html
```

The maintenance page can inform users that the primary application is temporarily unavailable.

---

## Health Checks

Route 53 health checks can monitor the health of configured endpoints.

Example:

```text
Route 53
    |
    v
Health Check
    |
    +---- Healthy
    |
    +---- Unhealthy
```

The health-check result can be used with routing policies.

---

# 5. Failover Routing Policy

Route 53 **Failover Routing** allows DNS records to be configured as:

```text
PRIMARY
SECONDARY
```

Example:

```text
Primary
   |
   v
ELB
   |
   +---- EC2-1
   +---- EC2-2

Secondary
   |
   v
S3
   |
   +---- maintenance.html
```

---

# 6. Primary Server

The primary destination in this practice is the ELB.

```text
Route 53
   |
   v
Primary Record
   |
   v
ELB
   |
   v
EC2 Instances
```

When the primary endpoint is healthy, users are directed toward the application.

---

# 7. Secondary Server

The secondary destination is the S3 static website.

```text
Route 53
   |
   v
Secondary Record
   |
   v
S3 Bucket
   |
   v
maintenance.html
```

The secondary destination provides a maintenance page when the configured failover condition occurs.

---

# 8. Complete Traffic Flow

## Normal Condition

When the primary application is healthy:

```text
User
 |
 v
sai.com
 |
 v
Route 53
 |
 | Health Check = Healthy
 v
ELB
 |
 +---- EC2-1
 |
 +---- EC2-2
 |
 v
Application
```

---

# 9. Failover Condition

If the configured primary health check becomes unhealthy:

```text
User
 |
 v
sai.com
 |
 v
Route 53
 |
 | Primary = Unhealthy
 v
Secondary
 |
 v
S3 Bucket
 |
 v
maintenance.html
```

Users can then receive the maintenance page instead of the unavailable primary application.

---

# 10. VPC Architecture

The EC2 instances and load balancer are deployed inside a VPC.

Example:

```text
AWS Region
    |
    v
   VPC
    |
    +-------------------------+
    |                         |
    v                         v
Public Subnet             Public Subnet
    |                         |
    v                         v
   ELB                       ELB
    |                         |
    +------------+------------+
                 |
          Private/Public EC2
                 |
        +--------+--------+
        |                 |
        v                 v
      EC2-1             EC2-2
```

Security Groups are used to control allowed traffic.

---

# 11. Security Groups

Example security architecture:

```text
Internet
   |
   v
ELB Security Group
   |
   | Application Traffic
   v
EC2 Security Group
```

Example rules:

### ELB Security Group

```text
HTTP   → 80
HTTPS  → 443
```

### EC2 Security Group

Allow application traffic from the ELB security group.

This avoids unnecessarily exposing the application servers directly to the internet.

---

# 12. Route 53 Hosted Zone

First, create a hosted zone for the domain.

Example:

```text
Route 53
   |
   v
Hosted Zones
   |
   v
sai.com
```

A hosted zone contains DNS records for the domain.

---

# 13. DNS Records

Example records:

```text
sai.com
   |
   +---- Primary
   |       |
   |       +---- ELB
   |
   +---- Secondary
           |
           +---- S3
```

The exact record configuration depends on the type of endpoint and AWS routing setup being used.

---

# 14. Deployment Steps

## Step 1 — Create VPC

Create the required:

* VPC
* Subnets
* Route Tables
* Internet Gateway
* Security Groups

---

## Step 2 — Launch EC2 Instances

Launch multiple EC2 instances for the web application.

Example:

```text
EC2-1
EC2-2
```

Install and configure the web application.

---

## Step 3 — Create Elastic Load Balancer

Create an ELB and configure:

* Listener
* Target Group
* Health Check
* EC2 targets

Example:

```text
ELB
 |
 +---- EC2-1
 |
 +---- EC2-2
```

---

## Step 4 — Test the ELB

Access the ELB DNS name and verify that the application is working.

---

## Step 5 — Create S3 Bucket

Create an S3 bucket for the maintenance page.

Example:

```text
maintenance.html
```

Configure the appropriate static website/access setup required for the chosen Route 53 configuration.

---

## Step 6 — Create Route 53 Hosted Zone

Create a hosted zone for:

```text
sai.com
```

---

## Step 7 — Configure Primary Record

Configure the primary Route 53 record to route traffic to the ELB.

```text
Primary
   |
   v
ELB
```

---

## Step 8 — Configure Secondary Record

Configure the secondary destination for the failover scenario.

```text
Secondary
   |
   v
S3
```

---

## Step 9 — Configure Health Check

Configure a Route 53 health check for the primary application endpoint as appropriate.

Example:

```text
Route 53
    |
    v
Health Check
    |
    v
Primary Application
```

---

# 15. Failover Test

After completing the configuration, test both conditions.

### Test 1 — Application Healthy

```text
Route 53
    |
    v
ELB
    |
    v
EC2
    |
    v
Application
```

Expected result:

```text
Application Page
```

### Test 2 — Primary Unhealthy

Simulate the configured primary failure condition.

```text
Route 53
    |
    | Primary unhealthy
    v
Secondary
    |
    v
S3
    |
    v
maintenance.html
```

Expected result:

```text
Maintenance Page
```

---

# 16. Route 53 Failover Concept

The basic concept is:

```text
                  Route 53
                     |
              Health Check
                     |
              +------+------+
              |             |
           Healthy       Unhealthy
              |             |
              v             v
             ELB            S3
              |             |
             EC2       maintenance.html
```

---

# 17. Technologies Used

| Technology             | Purpose                        |
| ---------------------- | ------------------------------ |
| Amazon Route 53        | DNS and traffic routing        |
| Elastic Load Balancer  | Distribute application traffic |
| Amazon EC2             | Host application               |
| Amazon S3              | Static maintenance page        |
| VPC                    | Network isolation              |
| Security Groups        | Network access control         |
| Route 53 Health Checks | Monitor endpoint health        |

---

# 18. Key Concepts Learned

Through this project, I practiced:

* DNS
* Route 53
* Hosted Zones
* DNS Records
* Failover Routing
* Health Checks
* Elastic Load Balancer
* EC2
* S3 Static Website Hosting
* VPC
* Security Groups
* High Availability concepts
* Application failover

---

# 19. Key Takeaway

The main concept I learned is how DNS can be used as part of a failover architecture.

```text
                USER
                  |
                  v
             Route 53
                  |
           Health Check
                  |
          +-------+-------+
          |               |
       Healthy         Unhealthy
          |               |
          v               v
         ELB              S3
          |               |
        EC2s       maintenance.html
          |
     Application
```

The project helped me understand how **Route 53, ELB, EC2, S3, health checks, and VPC networking** can work together to build a more resilient AWS application architecture.

---

## My Learning Outcome

I learned how Route 53 can provide DNS resolution and traffic management, how ELB distributes traffic across EC2 instances, and how a secondary destination can be configured for a failover scenario.

This project also helped me understand the relationship between:

```text
DNS
 ↓
Load Balancing
 ↓
Health Checks
 ↓
Failover
 ↓
High Availability
```
