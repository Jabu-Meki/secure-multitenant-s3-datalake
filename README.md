Secure Multi-Tenant S3 Data Lake
Overview

This repository demonstrates the design and implementation of a secure, multi-tenant data lake on Amazon S3, with a strong emphasis on tenant isolation, defense-in-depth security, and operational automation.

The project is intentionally built from first principles, focusing on how Amazon S3 enforces access control at the object-key (string) level and how IAM policies and bucket policies can be combined to create hard security boundaries in a shared storage environment.

Rather than relying on application-level trust, this design ensures that misconfigurations, overly permissive IAM policies, or key-guessing attempts cannot result in cross-tenant data access.

Problem Statement

Organizations often need to store data for multiple tenants (customers, teams, or workloads) in a single S3 bucket for cost efficiency and operational simplicity. However, S3 does not provide native directory permissions—object access is governed entirely by policy evaluation on object key strings.

This creates several challenges:

Preventing tenants from accessing or listing other tenants’ data

Enforcing least-privilege access without manual policy maintenance

Guarding against accidental IAM misconfiguration

Managing data lifecycle policies at scale

Auditing and detecting unauthorized access attempts

This project addresses these challenges through a layered security model.

Design Goals

Strong tenant isolation using prefix-based access control

IAM roles per tenant with least-privilege permissions

Bucket-level guardrails to prevent privilege escalation

No metadata or object name leakage between tenants

Encryption enforced at rest and in transit

Lifecycle automation for cost and data retention management

Fully reproducible infrastructure suitable for automation

Core Concepts
Prefix-Based Isolation

S3 does not have real directories. Each “folder” is a string prefix within an object key. Tenant boundaries are therefore enforced by restricting which key prefixes an IAM principal may access.

Example structure:

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


Access control is implemented by allowing or denying operations based on these prefixes.

IAM Roles per Tenant

Each tenant is assigned a dedicated IAM role that:

Can read and write only within its own prefix

Can list objects only within its own namespace

Has no permissions outside its assigned boundary

This mirrors a traditional Unix permission model where users own and access only their own directories.

Bucket Policy Guardrails

In addition to IAM role policies, the S3 bucket enforces global guardrails, including:

Denying unencrypted object uploads

Denying non-TLS requests

Denying access from unapproved principals

Preventing cross-tenant access even if an IAM policy is misconfigured

These guardrails ensure that security does not depend on perfect IAM policy hygiene.

Encryption and Auditability

Server-side encryption with AWS KMS (SSE-KMS)

Explicit key usage permissions scoped to S3

CloudTrail data events enabled for object-level auditing

Monitoring for unauthorized access attempts

Lifecycle Automation

Data lifecycle policies are applied using object tags rather than hard-coded paths, allowing:

Automated transition to lower-cost storage tiers

Automated expiration of stale data

Retention policies that can evolve without restructuring the bucket

Repository Structure
.
├── policies/          # IAM and bucket policy definitions
├── terraform/         # Infrastructure as code modules
├── tests/             # Access validation and isolation tests
├── docs/              # Design notes and threat modeling
└── README.md

Threat Model (High Level)

This project explicitly defends against:

Object key guessing across tenants

Over-permissive IAM role policies

Accidental wildcard resource grants

Metadata leakage through bucket listing

Unencrypted data ingestion

Unauthorized access attempts by compromised applications

Intended Audience

This repository is intended for:

Cloud engineers designing shared data platforms

Security-conscious AWS practitioners

Candidates preparing for solutions architect or platform engineering interviews

Anyone interested in understanding how S3 access control actually works under the hood

Status

This project is built incrementally. Each stage introduces additional controls and automation while preserving clarity and verifiability of the security model.v