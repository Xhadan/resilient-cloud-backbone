# Architecture Diagram – v1 (Conceptual)

## Purpose

This document describes the **initial architecture diagram** for the Resilient Cloud Backbone project. The goal is to capture *intent and reasoning*, not produce final visuals.

This diagram is intentionally **conceptual** and will evolve as implementation details (CDK, security hardening, cost optimizations) are added.

---

## Diagram Scope (v1)

The v1 diagram focuses on:

* Network isolation and traffic flow
* High availability across Availability Zones
* Clear separation between public and private resources

Out of scope for v1:

* Detailed IAM policies
* CI/CD pipelines
* Observability tooling (CloudWatch, X-Ray)

---

## High-Level Components

### 1. Route 53

* Provides DNS resolution for the application
* Configured for health checks and future failover routing

### 2. Application Load Balancer (ALB)

* Internet-facing
* Deployed across multiple AZs
* Terminates TLS and forwards traffic to application targets

### 3. VPC

* Single VPC per environment
* CIDR sized to allow future expansion

#### Subnets

* **Public Subnets (per AZ)**

  * Host ALB and NAT Gateway
* **Private App Subnets (per AZ)**

  * Host application compute (ECS/EKS/EC2 – TBD)
* **Private DB Subnets (per AZ)**

  * Isolated network tier for RDS

### 4. Application Tier

* Stateless services
* Designed to scale horizontally
* No direct internet access

### 5. Database Tier (RDS Multi-AZ)

* Synchronous replication across AZs
* Automatic failover
* No public endpoint

---

## Traffic Flow (Request Path)

1. Client resolves domain via Route 53
2. Request reaches ALB in public subnet
3. ALB forwards traffic to application targets in private subnets
4. Application queries RDS via private networking
5. Response flows back through ALB to client

---

## Availability Considerations

* No single AZ failure should take down the service
* ALB and RDS are Multi-AZ by default
* Stateless app tier enables rapid replacement

---

## Known Gaps / Future Iterations

* Add WAF in front of ALB
* Add VPC endpoints to reduce NAT dependency
* Introduce blue/green or canary deployment strategy
* Expand diagram to include CI/CD and observability

---

## Next Step

Create a **visual diagram (v1)** based on this document using draw.io or Lucidchart, then version it alongside this file.
