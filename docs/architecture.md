# Table of Contents

- [Document Purpose](#document-purpose)
- [Architectural Principles](#architectural-principles)
- [Trust Boundaries](#trust-boundaries)
- [Platform Layered Architecture](#platform-layered-architecture)
  - [Cloud Layer](#cloud-layer)
  - [Infrastructure Layer](#infrastructure-layer)
  - [Kubernetes Layer](#kubernetes-layer)
  - [AI Layer](#ai-layer)
  - [Governance Layer](#governance-layer)
- [Separation of Responsibilities](#separation-of-responsibilities)
  - [Provisioning](#provisioning)
  - [Enforcement](#enforcement)
  - [Runtime](#runtime)
  - [Governance](#governance)
- [Architectural Decisions and Rationale](#architectural-decisions-and-rationale)
- [High-Level Platform Architecture](#high-level-platform-architecture)
  - [Control Plane](#control-plane)
  - [Data Plane](#data-plane)
  - [AI Compute Plane](#ai-compute-plane)
- [Relationship with Other Documents](#relationship-with-other-documents)
- [Repository Structure](#repository-structure)
- [Final Architectural Overview](#final-architectural-overview)

---

# Document Purpose

`architecture.md` — platform architecture.

This document describes the architectural framework of the Sovereign AI Cloud Platform:  
design principles, trust boundaries, layered model, and architectural decisions.

The purpose of this document is to show **how and why** key architectural decisions were made.

This document is to be read together with:  
- `docs/security-model.md` — to understand threats and controls  
- `docs/workflows.md` — to understand the change lifecycle  
- `docs/data-flows.md` — to understand AI data flows  
- `docs/repository-structure.md` — to understand the purpose of each directory and file

---

# Architectural Principles

The platform architecture is based on the following principles:

- **Platform-first, not project-first**  
  The platform is designed to host dozens of services and AI workloads.

- **Security by design**  
  Security is built into the architecture, not added on top.

- **Policy-driven management**  
  Policies serve as the enforceable mechanism for approving changes.

- **Minimization of trust**  
  No implicit trust between layers, environments, and components.

- **Immutability**  
  Changes are achieved through recreation and promotion, not manual edits.

- **Auditability**  
  Every action leaves a verifiable trail.

---

# Trust Boundaries

The platform explicitly models trusted and untrusted zones.

Key trust boundaries:

- Between environments (`dev` / `stage` / `prod`)  
- Between provisioning and runtime  
- Between control plane and data plane  
- Between AI training and inference  
- Between human and non-human access  

Each boundary:  
- has formalized policies  
- is protected by enforcement mechanisms  
- is considered in the threat model  

Trust boundaries take precedence over specific technologies.

---

# Platform Layered Architecture

The architecture is organized as independent yet interconnected layers.

## Cloud Layer

The foundational cloud layer includes:  
- Networking  
- IAM  
- Compute  
- Storage  
- Observability  

This layer is responsible for:  
- Isolation  
- Scalability  
- Basic security  

It contains no business logic or AI-specific components.

## Infrastructure Layer

Terraform modules describing the architecture:  
- Resource aggregation  
- Reusability  
- Standardization  

The Infrastructure layer:  
- Is environment-agnostic  
- Is CI-agnostic  
- Contains no admission policies  

## Kubernetes Layer

Responsible for:  
- Control plane  
- Node groups  
- CNI  
- Runtime constraints  

Kubernetes is considered an **infrastructure primitive**,  
not an application platform.

## AI Layer

A separate trusted domain:  
- GPU node pools  
- Data zones  
- Training / registry / inference  

The AI layer:  
- Is isolated from general compute  
- Has its own trust boundaries  
- Is managed through model promotion  

## Governance Layer

The enforceable control layer:  
- Policy-as-Code  
- Audit trail  
- Exception workflows  
- Decision logs  

Governance does not observe — it **denies or allows** actions.

---

# Separation of Responsibilities

The architecture strictly separates areas of responsibility.

## Provisioning

- Terraform  
- Resource creation  
- No runtime access  

Provisioning does not manage changes after apply.

## Enforcement

- OPA  
- Checkov  
- tfsec  
- CI policy gates  

Enforcement makes decisions on the permissibility of changes.

## Runtime

- Kubernetes  
- AI workloads  
- Autoscaling  
- Execution environments  

Runtime does not have the right to modify infrastructure.

## Governance

- Process control  
- Auditing  
- Exceptions  
- Compliance enforcement  

Governance does not perform infrastructure operations.

---

# Architectural Decisions and Rationale

Key decisions:

- Avoid resources in root Terraform  
  → reduces blast radius

- Isolated state per environment  
  → prevents cross-environment errors

- CI as the sole apply point  
  → eliminates human error

- GPU as a separate compute domain  
  → controls access and cost

- Policy-as-Code  
  → ensures reproducible governance  

Each decision is made to reduce systemic risk.

---

# High-Level Platform Architecture

The architecture is divided into three planes:

## Control Plane

- Terraform  
- CI  
- Policies  
- Governance  

Manages changes but does not process data.

## Data Plane

- Kubernetes workloads  
- Services  
- Storage  

Has no access to the control plane.

## AI Compute Plane

- GPU nodes  
- Training jobs  
- Inference workloads  

Isolated from the general data plane.

---

# Relationship with Other Documents

- `security-model.md`  
  Describes threats and controls for each boundary.

- `workflows.md`  
  Describes the change lifecycle and promotion process.

- `data-flows.md`  
  Describes AI data movement across trust boundaries.

- `repository-structure.md`  
  Detailed description of repository structure, directories, and files.

---

# Repository Structure

The structure reflects architectural layers, not Terraform convenience:

- `global/` — organizational and foundational cloud layer  
- `modules/` — infrastructure primitives  
- `environments/` — environment-specific parameters  
- `policies/` — enforcement policies  
- `governance/` — control and audit  
- `ai/` — Sovereign AI domain  

The file structure follows the architecture, not the other way around.

For a detailed description of repository structure, directories, and files, see `docs/repository-structure.md`.

---

# Final Architectural Overview

The platform is a:  
- Managed  
- Scalable  
- Auditable  
- Security-first system  

The architecture is designed for growth, tightening requirements, and operation in sovereign environments without rebuilding the foundation.
