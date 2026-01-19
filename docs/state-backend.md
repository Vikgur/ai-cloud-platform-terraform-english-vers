# Table of Contents

- [Document Purpose and Scope](#document-purpose-and-scope)
- [Terraform State as a Security Asset](#terraform-state-as-a-security-asset)
  - [Why the State is a Critical Asset](#why-the-state-is-a-critical-asset)
  - [Blast Radius in Case of State Compromise](#blast-radius-in-case-of-state-compromise)
- [Threat Model for the State](#threat-model-for-the-state)
  - [Confidentiality Threats](#confidentiality-threats)
  - [Integrity Threats](#integrity-threats)
  - [Availability Threats](#availability-threats)
- [Backend Architecture for State Storage](#backend-architecture-for-state-storage)
  - [Backend Type and State Format](#backend-type-and-state-format)
  - [Environment Isolation](#environment-isolation)
  - [State Encryption](#state-encryption)
- [Access Model for the State](#access-model-for-the-state)
  - [IAM and Access Roles](#iam-and-access-roles)
  - [KMS and Key Management](#kms-and-key-management)
  - [Non-Human Access to the Backend](#non-human-access-to-the-backend)
- [Locking and Race Condition Protection](#locking-and-race-condition-protection)
  - [Why Locking is Needed](#why-locking-is-needed)
  - [Consequences of Missing Locking](#consequences-of-missing-locking)
- [Versioning and Immutability](#versioning-and-immutability)
  - [State Change History](#state-change-history)
  - [Protection Against Rollback Attacks](#protection-against-rollback-attacks)
- [Backup and Recovery](#backup-and-recovery)
  - [Backup Policy](#backup-policy)
  - [Disaster Recovery Scenarios](#disaster-recovery-scenarios)
  - [Recovery Responsibility Boundaries](#recovery-responsibility-boundaries)
- [Role of the State in the Change Workflow](#role-of-the-state-in-the-change-workflow)
  - [Integration with CI/CD](#integration-with-cicd)
  - [Integration with Promotion](#integration-with-promotion)
  - [Drift Detection](#drift-detection)
- [Break-Glass Access to the State](#break-glass-access-to-the-state)
  - [Conditions for Use](#conditions-for-use)
  - [Constraints and TTL](#constraints-and-ttl)
  - [Audit and Post-Incident Actions](#audit-and-post-incident-actions)
- [Explicit Non-Goals and Limitations](#explicit-non-goals-and-limitations)
- [Relationship with Other Documents](#relationship-with-other-documents)

---

# Document Purpose and Scope

`state-backend.md` – Terraform state backend and security model.

This document describes the **architectural model for storing, protecting, and using Terraform state**.  
State is treated not as a Terraform implementation detail, but as a **critical security asset**, whose compromise equals a compromise of the entire platform.

The document:  
– does not duplicate Terraform documentation  
– does not describe specific backend HCL files  
– captures security invariants and blast-radius thinking  

---

# Terraform State as a Security Asset

## Why the State is a Critical Asset

Terraform state contains:  
– a complete map of the infrastructure  
– actual resource identifiers  
– relationships between security domains  
– sensitive values (even if encrypted at rest)  

Compromising the state:  
– exposes the architecture  
– simplifies lateral movement  
– enables preparation of targeted attacks  

State = **the infrastructure control plane**.

## Blast Radius in Case of State Compromise

Without architectural safeguards:  
– one state = entire platform  
– one access = full control  

In this architecture, the blast radius is reduced through:  
– environment-specific state isolation  
– strict IAM model  
– no human access  
– mandatory locking  

---

# Threat Model for the State

## Confidentiality Threats

– leakage of sensitive outputs  
– exposure of network / IAM structure  
– analysis of security posture  

## Integrity Threats

– state tampering  
– rollback to a vulnerable state  
– race-condition corruption  

## Availability Threats

– loss of state  
– loss of locking  
– blocked apply  

---

# Backend Architecture for State Storage

## Backend Type and State Format

– Remote backend is used.  
– State is not stored locally.  
– Format — standard Terraform state with encryption-at-rest enabled.  

## Environment Isolation

– Dev / Stage / Prod have separate backends.  
– No shared bucket / storage.  
– No shared locking.  

Environment isolation = blast radius isolation.

## State Encryption

– At-rest encryption is mandatory.  
– KMS with restricted access is used.  
– Keys are not reused across environments.   

---

# Access Model for the State

## IAM and Access Roles

– Minimal set of roles:  
  - CI apply role  
  - CI plan role  

– No wildcard permissions.  
– No shared credentials.  

## KMS and Key Management

– KMS keys belong to the security domain.  
– Terraform does not manage key lifecycle.  
– Rotation is enabled.  

## Non-Human Access to the Backend

– Backend access is granted only to CI service accounts.  
– Human access is disabled by default.  
– OIDC + short-lived credentials.

---

# Locking and Race Condition Protection

## Why Locking is Needed

– Prevents parallel applies.  
– Protects state integrity.  
– Ensures predictable changes.  

## Consequences of Missing Locking

– Corrupted state.  
– Irreversible drift.  
– Platform-level incidents.  

Locking is considered a **mandatory security control**.

---

# Versioning and Immutability

## State Change History

– Versioning backend is enabled.  
– Every state change is traceable.  
– Forensic analysis is possible.  

## Protection Against Rollback Attacks

– Rollback is not possible without explicit intervention.  
– Reversion requires break-glass.  
– Rollback is recorded as a security event.  

---

# Backup and Recovery

## Backup Policy

– State backup is automatic.  
– Stored with restricted access.  
– Backup is not directly accessible by CI.  

## Disaster Recovery Scenarios

– Loss of CI  
– Loss of backend endpoint  
– Partial state loss  

For each scenario:  
– state restoration  
– CI re-initialization  

## Recovery Responsibility Boundaries

– State recovery ≠ AI data recovery.  
– State recovery ≠ workload restoration.  

---

# Role of the State in the Change Workflow

## Integration with CI/CD

– CI initializes the backend.  
– CI manages locking.  
– CI records state transitions.  

## Integration with Promotion

– Promotion = use of a different backend.  
– State is not transferred between environments.  
– Promotion does not copy state.  

## Drift Detection

– Drift = security incident.  
– Drift is detected by comparing state with reality.  
– Manual drift correction is prohibited.  

---

# Break-Glass Access to the State

## Conditions for Use

– Loss of CI control.  
– Corrupted state.  
– Platform-level incident.  

## Constraints and TTL

– Temporary access.  
– Minimal scope.  
– Automatic expiration of privileges.  

## Audit and Post-Incident Actions

– All actions are logged.  
– Mandatory post-mortem.  
– Return to CI-only access.  

---

# Explicit Non-Goals and Limitations

– The document does not describe a specific backend provider.  
– Does not describe Terraform commands.  
– Does not replace workflows.md.  

---

# Relationship with Other Documents

– workflows.md — how state participates in the change lifecycle.  
– security-model.md — threats and controls for the state.  
– break-glass.md — emergency access.  
– architecture.md — architectural placement of the state.  
