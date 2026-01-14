# About the Project

This repository is a reference Terraform architecture for enterprise-level environments.  
It demonstrates how to design a cloud platform for Kubernetes (3 masters / 50+ workers) from scratch, focusing on security, scalability, and manageability.

It implements best practice principles:

- Strict layer separation: `global` → `modules` → `environments`.
- One module = one responsibility.
- Remote state isolated per environment.
- Immutable infrastructure: compute resources are recreated without side-effects.
- Predictable scaling: control over autoscaling and planned growth.
- No snowflake nodes: all master and worker nodes are standardized.
- Blast radius control: errors are isolated by responsibility zones.
- Clear master/worker separation for security and stability.
- Policy-as-code and DevSecOps: OPA, tfsec, Checkov, auditing.
- GitOps-ready: CI/CD, multi-environment support, controlled rollout.
- Secure management of secrets, keys, state, and sensitive files.
- Growth-ready: supports hundreds of nodes without architecture refactoring.

# Structure

### README.md
Entry point.  
How to work with the repository, deployment order, and guidelines.

### docs/
Documentation as part of the infrastructure.
- architecture.md — overall architecture and module boundaries
- security-model.md — threat model, IAM, networks, trust boundaries
- state-backend.md — state storage, locking, encryption
- workflows.md — CI/CD scenarios, plan/apply

### global/
Organization-wide resources. Created once.
- backend/ — S3 + DynamoDB + KMS for state
- iam/ — Terraform roles and policies
- org-policies/ — company-level guardrails and quotas

### modules/
Reusable business logic.
- network/ — VPC, subnets, NAT, routing
- security/ — SG, NSG, firewall
- compute/ — master, worker, autoscaling
- kubernetes/ — control-plane, node-groups, CNI, bootstrap
- storage/ — block, object, backups
- observability/ — logging, monitoring, tracing
- access/ — IAM, OIDC, RBAC
- shared/ — naming, labels, tags, locals

### environments/
Specific environments. Only wiring.
- dev/
- stage/
- prod/

Important:
- No direct resources.
- Only module calls.

### policies/
DevSecOps controls.
- opa/ — restrictions and guardrails
- tfsec/ — security scanning
- checkov/ — compliance

### ci/
CI/CD pipelines.
- validate — fmt, validate
- plan — plan + artifacts
- apply — apply from protected branch
- security-scan — tfsec, checkov, opa

### scripts/
Local ergonomics.
- init.sh — terraform init
- plan.sh — standard plan
- apply.sh — controlled apply
- architecture_bootstrap.sh — project folder structure creation

### .terraform-version
Terraform version pinning. Ensures reproducibility.

---

### Summary
Complete enterprise Terraform picture:
- global — foundation
- modules — logic
- environments — configuration
- policies + ci — DevSecOps protection
