# Table of Contents

- [Document Purpose and Scope](#document-purpose-and-scope)
- [Security Mindset and Threat-Driven Approach](#security-mindset-and-threat-driven-approach)
- [Assets](#assets)
  - [Infrastructure Assets](#infrastructure-assets)
  - [Kubernetes and Runtime Assets](#kubernetes-and-runtime-assets)
  - [AI and Data](#ai-and-data)
  - [Management and Governance Assets](#management-and-governance-assets)
- [Actors](#actors)
  - [Human Actors](#human-actors)
  - [Non-Human Actors](#non-human-actors)
  - [External Systems](#external-systems)
- [Trust Boundaries and Responsibility Zones](#trust-boundaries-and-responsibility-zones)
- [Threat Landscape (Threat Model)](#threat-landscape-threat-model)
  - [Cloud and Network Level Threats](#cloud-and-network-level-threats)
  - [Terraform and CI Level Threats](#terraform-and-ci-level-threats)
  - [Kubernetes Runtime Level Threats](#kubernetes-runtime-level-threats)
  - [AI and Data Level Threats](#ai-and-data-level-threats)
  - [Governance and Process Threats](#governance-and-process-threats)
- [Security Controls](#security-controls)
  - [Preventive Controls](#preventive-controls)
  - [Detective Controls](#detective-controls)
  - [Corrective Controls](#corrective-controls)
- [Threat → Control Mapping](#threat--control-mapping)
- [Enforcement Levels](#enforcement-levels)
  - [Terraform-Level Enforcement](#terraform-level-enforcement)
  - [CI-Level Enforcement](#ci-level-enforcement)
  - [Kubernetes-Level Enforcement](#kubernetes-level-enforcement)
  - [AI Runtime Enforcement](#ai-runtime-enforcement)
  - [Governance Enforcement](#governance-enforcement)
- [Roles, Access, and Privileges](#roles-access-and-privileges)
  - [IAM Roles](#iam-roles)
  - [Kubernetes RBAC](#kubernetes-rbac)
  - [OIDC and Federated Access](#oidc-and-federated-access)
- [Break-Glass and Emergency Scenarios](#break-glass-and-emergency-scenarios)
- [Explicit Non-Goals and Model Limitations](#explicit-non-goals-and-model-limitations)
- [References and Related Documents](#references-and-related-documents)

---

# Document Purpose and Scope

`security-model.md` — the security model of the platform.

This document describes the **system-level security model** of the Sovereign AI Cloud Platform.  
It defines which assets are protected, against which threats, by which controls, and at what enforcement level.

The document:
- does not duplicate code or policies
- is not a tools checklist
- does not cover application-level security

Goal — to demonstrate **the mindset of a Senior DevSecOps engineer**, designing security as an architectural layer.

---

# Security Mindset and Threat-Driven Approach

Platform security is built **from threats**, not from tools.

Thinking sequence:
1. Identify assets  
2. Identify actors  
3. Define trust boundaries  
4. Model threats  
5. Map threats to controls  
6. Implement enforcement at the appropriate level  

Security is viewed as a **system of constraints**, not just a collection of best practices.

---

# Assets

## Infrastructure Assets

- Cloud accounts and subscriptions  
- Network segments and routing  
- Compute resources (CPU, GPU)  
- Storage and state backend  

Compromise of these assets results in systemic risk.

## Kubernetes and Runtime Assets

- Control plane  
- Worker nodes  
- Runtime configuration  
- Service accounts and secrets  

These are high blast-radius assets.

## AI and Data

- Training datasets  
- Models and artifacts  
- Registry and inference endpoints  

Data and models are the core sovereign assets.

## Management and Governance Assets

- Terraform state  
- CI pipelines  
- Policy-as-Code  
- Audit logs and decision logs  

Compromise of governance assets is equivalent to losing control.

---

# Actors

## Human Actors

- Platform engineers  
- Security engineers  
- Incident responders  

Human access is considered **the least reliable**.

## Non-Human Actors

- CI/CD systems  
- Terraform automation  
- Kubernetes workloads  
- AI services  

Non-human access is **preferred**.

## External Systems

- Identity providers  
- Audit and monitoring systems  
- External integrations  

All external interactions are treated as **untrusted**.

---

# Trust Boundaries and Responsibility Zones

The platform explicitly models trust boundaries:

- Between environments  
- Between control plane and runtime  
- Between Terraform and Kubernetes  
- Between AI training and inference  
- Between human and non-human access  

Each boundary is:
- documented  
- enforced by policies  
- auditable

---

# Threat Landscape (Threat Model)

## Cloud and Network Level Threats

- Lateral movement  
- Network misconfiguration  
- Overprivileged IAM

## Terraform and CI Level Threats

- Unauthorized apply  
- State tampering  
- Policy check bypass

## Kubernetes Runtime Level Threats

- Privilege escalation  
- Pod escape  
- Uncontrolled access to secrets  

## AI and Data Level Threats

- Unauthorized data access  
- Model tampering  
- GPU usage outside authorized context  

## Governance and Process Threats

- Process bypass  
- Non-auditable changes  
- Abuse of emergency access  

---

# Security Controls

## Preventive Controls

- IAM least privilege  
- Policy-as-Code  
- Network isolation  
- Runtime constraints

## Detective Controls

- Audit logs  
- Monitoring and alerts  
- Policy violations tracking  

## Corrective Controls

- Automated rollback  
- Access revocation  
- Incident workflows  

Controls are distributed across layers.

---

# Threat → Control Mapping

Each critical threat is:
- mapped to one or more controls  
- enforced at least at one enforcement level  
- auditable  

Missing mapping is considered an **architectural flaw**.

---

# Enforcement Levels

## Terraform-Level Enforcement

- Resource constraints  
- Security defaults  
- Prohibition of unsafe configurations  

## CI-Level Enforcement

- OPA  
- Checkov  
- tfsec  
- Mandatory reviews  

CI is the first line of defense.

## Kubernetes-Level Enforcement

- RBAC  
- Admission controls  
- Runtime policies  

## AI Runtime Enforcement

- GPU isolation  
- Namespace separation  
- Data access policies

## Governance Enforcement

- Prohibition of process bypass  
- Exception workflows  
- Decision logging  

Governance is the last line of defense.

---

# Roles, Access, and Privileges

## IAM Roles

- Least privilege  
- Function-based separation  
- No shared credentials  

## Kubernetes RBAC

- Service-account-first approach  
- Namespace scope limitation  

## OIDC and Federated Access

- No long-lived keys  
- Centralized identity management

---

# Break-Glass and Emergency Scenarios

Emergency access is:
- strictly time-bound  
- fully auditable  
- requires post-incident normalization  

Break-glass is **an exception, not a management tool**.

See `break-glass.md` for details.

---

# Explicit Non-Goals and Model Limitations

The security model intentionally **does not cover**:

- Application-level security  
- ML model robustness  
- Data science pipelines  
- User-facing API security  

This is an architectural decision, not an oversight.

---

# References and Related Documents

- `architecture.md` — architectural boundaries and layers  
- `workflows.md` — change lifecycle  
- `data-flows.md` — data movement and enforcement  
- `break-glass.md` — emergency scenarios  

The security model serves as the connecting layer between architecture and processes.
