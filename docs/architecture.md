# Architecture v1 — Resilient Cloud Backbone

> **Status:** Baseline (v1)
>
> This document captures the initial architecture decisions for a highly available, cost-conscious, and evolvable cloud foundation. It is intentionally *not final* and will evolve based on testing, traffic patterns, and operational learnings.

---

## Goals

* Achieve **99.9% availability** for the web tier
* Follow **least-privilege** and **defense-in-depth** principles
* Enable **repeatable deployments** via IaC (AWS CDK)
* Keep **costs predictable** at low-to-moderate traffic
* Provide a foundation suitable for future AI/ML workloads

## Non-Goals (v1)

* Global multi-region active/active
* Zero-downtime schema migrations
* Complex service mesh or multi-account setup

---

## High-Level Design

* **Region:** Single AWS region
* **Availability:** Multi-AZ within the region
* **Topology:** 3-tier (ALB → App → Database)

**Rationale:**
A single-region, Multi-AZ design provides a strong availability baseline while minimizing operational complexity and cost. Regional isolation simplifies networking, IAM, and observability in v1.

---

## Networking

### VPC

* **CIDR:** `/16`
* **Subnets:**

  * 2× Public subnets (one per AZ)
  * 2× Private app subnets (one per AZ)
  * 2× Private DB subnets (one per AZ)

**Rationale:**

* `/16` allows future subnet expansion without readdressing
* Separation of public/private tiers limits blast radius
* DB subnets isolated with no direct internet access

---

## Traffic Ingress

### Application Load Balancer (ALB)

* Deployed across **2 AZs**
* Terminates TLS
* Health checks at the application layer

**Rationale:**

* ALB provides native Multi-AZ redundancy
* Centralized TLS simplifies certificate management
* Health checks enable automatic failover at the AZ level

---

## Application Tier

* Stateless compute (EC2 or ECS — decision deferred)
* Deployed across **2 AZs**
* Auto Scaling enabled

**Rationale:**

* Stateless design enables horizontal scaling
* AZ distribution ensures resilience to single-AZ failure
* Compute choice deferred to allow cost/performance testing

---

## Data Tier

### Amazon RDS (Multi-AZ)

* Single primary with synchronous standby
* Automated backups enabled

**Rationale:**

* Multi-AZ RDS provides automatic failover
* Simpler than self-managed replication in v1
* Sufficient for 99.9% availability target

---

## DNS & Failover

### Route 53

* Primary record pointing to ALB
* Health checks enabled (v2 enhancement)

**Rationale:**

* DNS-level failover provides an additional safety net
* Enables future multi-region expansion

---

## Security (v1)

* Security groups per tier
* No public access to app or DB subnets
* IAM roles for compute resources

**Deferred:**

* WAF
* Fine-grained outbound egress controls
* Secrets rotation automation

---

## Deployment Strategy

* **Infrastructure as Code:** AWS CDK
* **Deployment Model:** Single-command deploy

**Rationale:**

* CDK enables abstraction while retaining AWS-native constructs
* Single-command deploy reduces configuration drift

---

## Known Tradeoffs & Risks

* Single-region outage would cause downtime
* RDS failover introduces brief write unavailability
* ALB is a shared regional dependency

These are accepted risks in v1 and will be revisited.

---

## Planned Evolution

* v2: WAF, Route 53 health-based routing
* v3: Read replicas, async cross-region backup
* v4: Active/active multi-region

---

> **Note:** This architecture is intentionally conservative. The goal of v1 is operational correctness and clarity, not maximal sophistication.
