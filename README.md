# Secure Multi-Tenant S3 Data Lake

A production-grade implementation of a secure, multi-tenant data lake on Amazon S3, demonstrating defense-in-depth security and tenant isolation through prefix-based access control.

---

## Overview

This project demonstrates how to build a **secure shared data lake** where multiple tenants store data in a single S3 bucket without risking cross-tenant data access. Rather than relying on application-level trust, this design ensures that **IAM misconfigurations, overly permissive policies, or key-guessing attempts cannot breach tenant boundaries**.

The architecture is built from first principles, focusing on how S3 enforces access control at the object-key level and how IAM policies and bucket policies combine to create hard security boundaries.

---

## Problem Statement

Organizations need to store data for multiple tenants (customers, teams, workloads) in shared S3 infrastructure for cost efficiency and operational simplicity. However, S3 doesn't provide native directory permissions access is governed entirely by policy evaluation on object key strings.

**Key Challenges:**
- Preventing tenants from accessing or listing other tenants' data
- Enforcing least-privilege access without manual policy maintenance
- Guarding against accidental IAM misconfiguration
- Managing data lifecycle policies at scale
- Auditing and detecting unauthorized access attempts

---

## Design Principles

### 1. Prefix-Based Isolation

S3 object keys are flat strings "folders" are prefix conventions. Tenant boundaries are enforced by restricting which key prefixes an IAM principal may access.

```
s3://datalake-bucket/
└── tenants/
    ├── tenant-a/
    │   ├── raw/
    │   ├── staged/
    │   └── curated/
    └── tenant-b/
        ├── raw/
        ├── staged/
        └── curated/
```

### 2. IAM Roles per Tenant

Each tenant receives a dedicated IAM role that:
- Can read/write only within its own prefix
- Can list objects only within its own namespace
- Has no permissions outside its assigned boundary

### 3. Bucket Policy Guardrails

The S3 bucket enforces global security controls:
- Deny unencrypted object uploads
- Deny non-TLS requests
- Deny access from unapproved principals
- Prevent cross-tenant access even if IAM policies are misconfigured

**Security does not depend on perfect IAM policy hygiene.**

### 4. Encryption and Auditability

- Server-side encryption with AWS KMS (SSE-KMS)
- Key usage permissions scoped to S3 operations
- CloudTrail data events for object-level auditing
- Monitoring for unauthorized access attempts

### 5. Lifecycle Automation

Data lifecycle policies use object tags rather than hard-coded paths:
- Automated transition to lower-cost storage tiers
- Automated expiration of stale data
- Retention policies that evolve without bucket restructuring

---

## Repository Structure

```
.
├── policies/          # IAM and bucket policy definitions
├── terraform/         # Infrastructure as code modules
├── tests/             # Access validation and isolation tests
├── docs/              # Design notes and threat modeling
└── README.md
```

---

## Threat Model

This design explicitly defends against:

| Threat | Mitigation |
|--------|-----------|
| Object key guessing across tenants | Prefix-based IAM conditions + bucket policy deny rules |
| Over-permissive IAM role policies | Bucket policy enforces maximum permissions boundary |
| Accidental wildcard resource grants | Explicit prefix matching in all policies |
| Metadata leakage through bucket listing | IAM policies restrict `ListBucket` to specific prefixes |
| Unencrypted data ingestion | Bucket policy denies uploads without encryption headers |
| Compromised application credentials | Least-privilege roles limit blast radius |

---

## Security Model

### Layered Defense

```
┌─────────────────────────────────────┐
│   Application Layer (Untrusted)    │
├─────────────────────────────────────┤
│   IAM Role Policies (Tenant-scoped)│  ← Least privilege
├─────────────────────────────────────┤
│   S3 Bucket Policies (Global rules)│  ← Hard boundaries
├─────────────────────────────────────┤
│   KMS Encryption (Data at rest)    │  ← Defense in depth
├─────────────────────────────────────┤
│   CloudTrail Auditing              │  ← Detect anomalies
└─────────────────────────────────────┘
```

### Access Evaluation Flow

1. **Principal** assumes tenant-specific IAM role
2. **IAM policy** allows access only to tenant prefix
3. **Bucket policy** validates request doesn't violate global rules
4. **KMS policy** authorizes encryption key usage
5. **Request succeeds** only if all layers permit

---

## Getting Started

### Prerequisites

- AWS CLI configured with appropriate credentials
- Terraform >= 1.0
- Basic understanding of IAM policy evaluation

### Quick Start

```bash
# Clone the repository
git clone https://github.com/yourusername/secure-s3-datalake
cd secure-s3-datalake

# Review and customize configuration
cd terraform
cp terraform.tfvars.example terraform.tfvars

# Deploy infrastructure
terraform init
terraform plan
terraform apply

# Run isolation tests
cd ../tests
./run_isolation_tests.sh
```

---

## Use Cases

- **Multi-customer SaaS platforms** requiring strict data isolation
- **Enterprise data lakes** with multiple business units
- **Regulated industries** (healthcare, finance) with compliance requirements
- **Development/staging/production** environment separation

---

## Intended Audience

- Cloud engineers designing shared data platforms
- Security-conscious AWS practitioners
- Solutions Architect or Platform Engineering interview candidates
- Anyone interested in understanding S3 access control internals

---

## Project Status

This project is built incrementally. Each stage introduces additional controls and automation while preserving clarity and verifiability of the security model.

| Phase | Status | Description |
|-------|--------|-------------|
| Core isolation | Complete | Prefix-based access control and IAM roles |
| Encryption | Complete | KMS integration and bucket policies |
| Lifecycle automation | In Progress | Tag-based lifecycle policies |
| Monitoring | Planned | CloudWatch alarms and anomaly detection |

---

## Contributing

Contributions welcome! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

---

## License

MIT License - see [LICENSE](LICENSE) for details

---

## Acknowledgments

Built on AWS security best practices and real-world production experience securing multi-tenant data platforms.

---

> **Note:** This is a demonstration project. Always conduct thorough security reviews and penetration testing before deploying multi-tenant infrastructure in production.
