# Table of Contents

- [Document Purpose and Scope](#document-purpose-and-scope)
- [Fundamental Change Management Principles](#fundamental-change-management-principles)
  - [Immutability as an Architectural Requirement](#immutability-as-an-architectural-requirement)
  - [Non-human First Access](#non-human-first-access)
  - [Prohibition of Manual Changes](#prohibition-of-manual-changes)
- [Infrastructure Lifecycle](#infrastructure-lifecycle)
  - [Change Initiation](#change-initiation)
  - [Planning and Validation](#planning-and-validation)
  - [Applying Changes](#applying-changes)
  - [Post-apply Control](#post-apply-control)
- [CI/CD as the Single Point of Application](#cicd-as-the-single-point-of-application)
  - [Terraform Workflow](#terraform-workflow)
  - [Policy-as-code Gates](#policy-as-code-gates)
  - [Separation of Plan / Apply](#separation-of-plan--apply)
  - [Environment Promotion](#environment-promotion)
- [AI Infrastructure Lifecycle](#ai-infrastructure-lifecycle)
  - [Training Environment](#training-environment)
  - [Model Registry Environment](#model-registry-environment)
  - [Inference Environment](#inference-environment)
  - [Promotion as a Security Decision](#promotion-as-a-security-decision)
- [Promotion Boundaries and Gates](#promotion-boundaries-and-gates)
  - [Infrastructure Promotion](#infrastructure-promotion)
  - [AI Promotion](#ai-promotion)
  - [What Constitutes a Workflow Violation](#what-constitutes-a-workflow-violation)
- [Human vs Non-human Actions](#human-vs-non-human-actions)
  - [Permissible Human Actions](#permissible-human-actions)
  - [Prohibited Human Actions](#prohibited-human-actions)
  - [Role of Service Accounts and OIDC](#role-of-service-accounts-and-oidc)
- [Break-glass as a Workflow Exception](#break-glass-as-a-workflow-exception)
  - [When Break-glass is Allowed](#when-break-glass-is-allowed)
  - [How It Integrates into the Lifecycle](#how-it-integrates-into-the-lifecycle)
- [Audit and Change Traceability](#audit-and-change-traceability)
  - [What is Logged](#what-is-logged)
  - [Where the Audit Trail is Stored](#where-the-audit-trail-is-stored)
- [Explicit Non-goals and Constraints](#explicit-non-goals-and-constraints)
- [Relation to Other Documents](#relation-to-other-documents)

---

# Document Purpose and Scope

`workflows.md` – change management and platform workflows.

This document describes **how changes are allowed, validated, applied, and promoted** within the platform.  
The focus is not on tools, but on the **architectural model of change management**.

The document intentionally:  
– does not duplicate `security-model.md`  
– does not describe specific YAML / HCL implementations  
– records invariants and decision points  

---

# Fundamental Change Management Principles

## Immutability as an Architectural Requirement

– Infrastructure is not “healed”; it is recreated.  
– Compute, Kubernetes nodes, and AI workloads are considered disposable.  
– Any change = a new version of the infrastructure.  

Consequences:  
– no snowflake nodes  
– rollback = return to the previous state via state management  
– drift is treated as an incident 

## Non-human First Access

– All changes are executed by **service accounts**.  
– Human access is not part of the normal workflow.  
– OIDC + short-lived credentials are used instead of static keys.  

Humans:  
– design  
– approve  
– analyze  
but **do not apply changes directly**.

## Prohibition of Manual Changes

– Direct `terraform apply` outside CI is architecturally prohibited.  
– Changes via the cloud console are considered a violation of the model.  
– The only exception is break-glass (see below).  

---

# Infrastructure Lifecycle

## Change Initiation

– Changes start with a Git commit.  
– Git is the single source of truth.  
– Every change must be reproducible.

## Planning and Validation

– Terraform plan is executed automatically.  
– Before apply, the following checks are performed:  
  - OPA policies  
  - tfsec  
  - Checkov  

Invalid changes:  
– do not reach apply  
– require no manual intervention  

## Applying Changes

– Apply is executed only via CI.  
– Plan / apply separation is strict.  
– Apply is allowed only after review.  

## Post-apply Control

– Drift detection  
– Policy compliance verification  
– Audit of applied changes

---

# CI/CD as the Single Point of Application

## Terraform Workflow

– CI provisions an isolated environment.  
– Remote backend with locking is used.  
– State is isolated per environment.  

## Policy-as-Code Gates

– Policies are blocking, not advisory.  
– Policy failure = pipeline failure.  
– No override possible without break-glass.  

## Separation of Plan / Apply

– Plan is available for review.  
– Apply is a separate step with elevated privileges.  
– Apply is not triggered automatically without conditions.  

## Environment Promotion

– Dev → Stage → Prod.  
– No direct jumps.  
– Promotion = re-application of a validated change.

---

# AI Infrastructure Lifecycle

## Training Environment

– Frequently changing.  
– High risk.  
– Isolated from inference.  

Changes are allowed frequently but strictly limited by blast radius.

## Model Registry Environment

– Stable.  
– Trusted artifacts only.  
– Promotion is strictly controlled.  

## Inference Environment

– Production-grade.  
– Minimal changes.  
– Maximum control.  

## Promotion as a Security Decision

– A model is not “moved”; it is **approved**.  
– Promotion = change in trust level.  
– Any bypass is treated as an incident.

---

# Promotion Boundaries and Gates

## Infrastructure Promotion

– Promotion is environment-based.  
– State and backend are separate.  
– No shared resources between environments.  

## AI Promotion

– Training → Registry → Inference.  
– Each boundary is enforced by policy-as-code.  

## What Constitutes a Workflow Violation

– Manual apply.  
– Direct access to prod.  
– State modification outside CI.  

---

# Human vs Non-human Actions

## Permissible Human Actions

– Architecture design.  
– Plan review.  
– Change approval.  
– Incident analysis.

## Prohibited Human Actions

– Direct access to infrastructure APIs.  
– Manual correction of production resources.  
– Direct access to AI runtime.  

## Role of Service Accounts and OIDC

– All applies are executed by service accounts.  
– Credentials are short-lived.  
– Tied to identity, not to a human.  

---

# Break-glass as a Workflow Exception

## When Break-glass is Allowed

– Incident.  
– Loss of control.  
– Security emergency.  

## How It Integrates into the Lifecycle

– Limited scope.  
– TTL enforced.  
– Mandatory audit.  
– Mandatory post-mortem.  

Break-glass is **not an alternative workflow**, but a controlled failure mode.

---

# Audit and Change Traceability

## What is Logged

– Who initiated the change.  
– Which commit was applied.  
– Which policies passed.  
– Which resources were modified.  

## Where the Audit Trail is Stored

– CI logs.  
– Cloud audit logs.  
– Governance decision logs.  

---

# Explicit Non-goals and Constraints

– The document does not describe specific CI implementations.  
– Does not describe developer UX.  
– Does not replace `security-model.md`.  

---

# Relation to Other Documents

– `security-model.md` — why these constraints exist.  
– `data-flows.md` — which workflows affect data and models.  
– `break-glass.md` — what a permissible exception looks like.  
– `architecture.md` — where these workflows are architecturally enforced.
