# Table of Contents

- [Document Purpose and Scope](#document-purpose-and-scope)
  - [Document Objective](#document-objective)
  - [What is Considered Data in the Platform Context](#what-is-considered-data-in-the-platform-context)
  - [What is Intentionally Out of Scope](#what-is-intentionally-out-of-scope)
- [Security-First View on Data Flows](#security-first-view-on-data-flows)
  - [Data Flows as Security Invariants](#data-flows-as-security-invariants)
  - [Why This is Not a Data Pipeline Tutorial](#why-this-is-not-a-data-pipeline-tutorial)
  - [Relation Between Data Flows and Trust Boundaries](#relation-between-data-flows-and-trust-boundaries)
- [High-Level End-to-End AI Data Flows](#high-level-end-to-end-ai-data-flows)
  - [Overall Chain](#overall-chain)
  - [Data Classes at Each Stage](#data-classes-at-each-stage)
  - [Stage Criticality and Blast Radius](#stage-criticality-and-blast-radius)
- [Trust Boundaries in Data Flows](#trust-boundaries-in-data-flows)
  - [External Ingress Boundary](#external-ingress-boundary)
  - [Training Boundary](#training-boundary)
  - [Promotion Boundary](#promotion-boundary)
  - [Inference Boundary](#inference-boundary)
  - [Serving / Exposure Boundary](#serving--exposure-boundary)
- [Ownership and Responsibility at Each Stage](#ownership-and-responsibility-at-each-stage)
  - [Data Ownership at Ingress](#data-ownership-at-ingress)
  - [Ownership During Training](#ownership-during-training)
  - [Model Ownership in Registry](#model-ownership-in-registry)
  - [Ownership During Inference and Serving](#ownership-during-inference-and-serving)
  - [Handover of Responsibility as an Architectural Event](#handover-of-responsibility-as-an-architectural-event)
- [Promotion Boundaries and Gating Mechanisms](#promotion-boundaries-and-gating-mechanisms)
  - [Training → Model Registry](#training--model-registry)
  - [Model Registry → Inference](#model-registry--inference)
  - [Promotion as a Security Decision](#promotion-as-a-security-decision)
  - [What Constitutes an Unauthorized Bypass](#what-constitutes-an-unauthorized-bypass)
- [Enforcement Points and Mechanisms](#enforcement-points-and-mechanisms)
  - [Terraform-Level Enforcement](#terraform-level-enforcement)
  - [CI / Pipeline Enforcement](#ci--pipeline-enforcement)
  - [Policy-as-Code Enforcement](#policy-as-code-enforcement)
  - [Kubernetes / Runtime Enforcement](#kubernetes--runtime-enforcement)
  - [AI-Specific Enforcement](#ai-specific-enforcement)
- [Mapping Data Flows to Security Controls](#mapping-data-flows-to-security-controls)
  - [Relation to security-model.md](#relation-to-security-modelmd)
  - [Which Threats Are Mitigated](#which-threats-are-mitigated)
  - [Mandatory Enforcement Points](#mandatory-enforcement-points)
- [Audit and Traceability of Data Flows](#audit-and-traceability-of-data-flows)
  - [What is Logged](#what-is-logged)
  - [Where the Audit Trail is Stored](#where-the-audit-trail-is-stored)
  - [Relation of Audit Data to Governance](#relation-of-audit-data-to-governance)
- [Diagrams and Visual Model](#diagrams-and-visual-model)
  - [High-Level Data Flow Diagram](#high-level-data-flow-diagram)
  - [Trust Boundaries Overlay](#trust-boundaries-overlay)
  - [Promotion Boundary Schematic](#promotion-boundary-schematic)
- [Relation of Data Flows to Policy-as-Code Implementation](#relation-of-data-flows-to-policy-as-code-implementation)
  - [Ingress → Training](#ingress--training)
  - [Training (Internal AI Loop)](#training-internal-ai-loop)
  - [Training → Model Registry (Promotion Boundary)](#training--model-registry-promotion-boundary)
  - [Model Registry (Trusted AI Artifacts)](#model-registry-trusted-ai-artifacts)
  - [Model Registry → Inference](#model-registry--inference)
  - [Inference (Runtime Execution)](#inference-runtime-execution)
  - [Inference → Serving](#inference--serving)
  - [Architectural Invariant](#architectural-invariant)
- [Explicit Non-Goals and Model Limitations](#explicit-non-goals-and-model-limitations)
  - [What the Platform Consciously Does Not Solve](#what-the-platform-consciously-does-not-solve)
  - [Platform Responsibility Boundaries](#platform-responsibility-boundaries)
- [Relation to Other Documents](#relation-to-other-documents)
  - [security-model.md](#security-modelmd)
  - [workflows.md](#workflowsmd)
  - [architecture.md](#architecturemd)

---

# Document Purpose and Scope

`data-flows.md` – Data flows and AI loops of the platform

## Document Objective
To establish the **architectural model of data and model flows** within the AI platform, where data and AI artifacts are treated as critical assets.  
The document describes **trust boundaries, handover points, and enforcement**, not the implementation of data pipelines.

## What is Considered Data in the Platform Context
– Source datasets (raw / curated / sensitive)  
– Derived training data  
– Models (artifacts, weights, metadata)  
– Inference outputs  
– Telemetry and audit data related to AI processes  

All of the above are treated as **security-relevant assets**.

## What is Intentionally Out of Scope
– ML algorithms and model quality  
– Choice of training frameworks  
– Inference optimization  
– ETL and feature engineering implementation  
– Business logic of model usage

---

# Security-First View on Data Flows

## Data Flows as Security Invariants
Each data flow:  
– Crosses a trust boundary  
– Changes ownership  
– Requires an explicit policy decision  

If a flow is not defined — it is **denied by default**.

## Why This is Not a Data Pipeline Tutorial
The goal is not to show "how to train a model,"  
but to define:  
– **Who is allowed to move data**  
– **Where promotion occurs**  
– **Which loops are intentionally isolated**  

## Relation Between Data Flows and Trust Boundaries
A data flow = an authorized transition between trust boundaries.  
Any unauthorized transition is considered an incident.

---

# High-Level End-to-End AI Data Flows

## Overall Chain
ingress → training → model registry → inference → serving

## Data Classes at Each Stage
**Ingress**  
– External data  
– Potentially untrusted sources  

**Training**  
– Sensitive datasets  
– Temporary derived data  

**Model Registry**  
– Models as managed artifacts  
– Metadata and provenance  

**Inference**  
– Models in the runtime loop  
– Input requests  

**Serving**  
– Inference outputs  
– Public or semi-public interfaces

## Stage Criticality and Blast Radius
– Ingress: risk of contamination  
– Training: risk of data leakage  
– Registry: risk of supply chain compromise  
– Inference: risk of unauthorized access  
– Serving: risk of external exposure  

---

# Trust Boundaries in Data Flows

## External Ingress Boundary
Boundary between the external world and the platform.  
Untrusted zone.

Context:  
– No implicit trust  
– All data is considered potentially malicious  

## Training Boundary
Isolated compute loop.  
Access is strictly restricted.

Context:  
– Minimal human access  
– No direct egress without authorization

## Promotion Boundary
Key trust boundary.  
Between training and operational use.

Context:  
– Only controlled transitions allowed  
– Mandatory verification and audit  

## Inference Boundary
Runtime loop for model execution.

Context:  
– Models are considered trusted artifacts  
– Input data is again treated as untrusted  

## Serving / Exposure Boundary
Boundary for exposing results.

Context:  
– Protection against data leakage  
– Control over the scope and nature of responses

---

# Ownership and Responsibility at Each Stage

## Data Ownership at Ingress
Owner: external source / data provider.  
The platform **does not trust** the data.

Responsibilities:  
– Validation  
– Classification  
– Acceptance or rejection  

## Ownership During Training
Ownership transfers to the platform.

Responsibilities:  
– Data protection  
– Compute isolation  
– Leak prevention  

## Model Ownership in Registry
The model becomes a **managed artifact**.

Responsibilities:  
– Version control  
– Provenance  
– Prevention of unauthorized modifications

## Ownership During Inference and Serving
Model ownership remains with the platform.  
Responsibility for outputs is shared:  
– Platform  
– Consuming system  

## Handover of Responsibility as an Architectural Event
Any change of ownership:  
– Is recorded  
– Requires a policy decision  
– Leaves an audit trail  

---

# Promotion Boundaries and Gating Mechanisms

## Training → Model Registry
Critical point.

Allowed only if:  
– Model has passed validation  
– Provenance is verified  
– Policy checks succeed

## Model Registry → Inference
Promotion into the runtime loop.

Allowed only if:  
– Model is signed  
– Version is recorded  
– Environment complies with policy  

## Promotion as a Security Decision
Promotion ≠ file copy.  
It is a **security event**.

## What Constitutes an Unauthorized Bypass
– Direct access from training → inference  
– Manual model substitution  
– Bypassing the registry  

---

# Enforcement Points and Mechanisms

## Terraform-Level Enforcement
– Network isolation  
– Separation of accounts / projects  
– Prohibition of unauthorized connections

## CI / Pipeline Enforcement
– Checks before promotion  
– Deployment prohibited without review  
– Policy-as-Code  

## Policy-as-Code Enforcement
– OPA  
– Checkov  
– tfsec  

Used as a gate, not advisory.

## Kubernetes / Runtime Enforcement
– Namespace isolation  
– RBAC  
– Network policies  

## AI-Specific Enforcement
– GPU isolation  
– Data zones  
– Prohibition of shared runtime

---

# Mapping Data Flows to Security Controls

## Relation to security-model.md
All threats and controls are defined there.  
Here — **application by stage**.

## Which Threats Are Mitigated
– Data poisoning  
– Model tampering  
– Privilege escalation  
– Data exfiltration  

## Mandatory Enforcement Points
– Ingress  
– Promotion boundaries  
– Runtime access

---

# Audit and Traceability of Data Flows

## What is Logged
– Transition event  
– Subject  
– Object  
– Policy decision  

## Where the Audit Trail is Stored
– Centrally  
– Immutable  

## Relation of Audit Data to Governance
Audit is part of governance, not “check-the-box” logging.

---

# Diagrams and Visual Model

## High-Level Data Flow Diagram
Shows allowed directions of movement.

## Trust Boundaries Overlay
Overlays trust boundaries on the flow.

## Promotion Boundary Schematic
Records decision points.

---

# Relation of Data Flows to Policy-as-Code Implementation

Each **AI data flow** below is linked to **actual policy files**.  
The format is intentionally flat: flow → threats → enforcement.  
This is not a file catalog, but a **decision graph** tied to the repository.

## Ingress → Training

– Purpose: intake of external data into the training loop (untrusted zone → isolated loop).  
– Key threats:  
  – Data poisoning  
  – Public access to AI loops  
  – Leakage of sensitive data  

– Enforcement:  
  – OPA Terraform:  
    – `policies/opa/terraform/encryption.rego` — mandatory encryption of data-at-rest  
    – `policies/opa/terraform/regions.rego` — prohibition of unauthorized storage regions  
    – `policies/opa/terraform/tagging.rego` — data classification and ownership  
  – OPA AI:  
    – `policies/opa/ai/no-public-ai.rego` — prohibition of public AI endpoints  
    – `policies/opa/ai/ai-data-isolation.rego` — isolation of AI data zones  
  – tfsec:  
    – `policies/tfsec/ai/ai-storage.toml` — unsafe storage configurations  
  – Checkov:  
    – `policies/checkov/ai/ai_network.yaml` — public ingress / network paths

## Training (Internal AI Loop)

– Purpose: processing sensitive data and training models.  
– Key threats:  
  – Lateral movement  
  – Unauthorized egress  
  – Shared GPU usage  

– Enforcement:  
  – OPA AI:  
    – `policies/opa/ai/ai-data-isolation.rego` — strict isolation of training data  
    – `policies/opa/ai/ai-gpu-restrictions.rego` — prohibition of shared GPU between loops  
  – OPA Terraform:  
    – `policies/opa/terraform/tagging.rego` — separation of training / inference resources  
  – tfsec:  
    – `policies/tfsec/ai/ai-storage.toml` — control of temporary data volumes

## Training → Model Registry (Promotion Boundary)

– Purpose: transfer of a model from the untrusted training loop to the managed registry.  
– Architectural fact: promotion = security decision.  
– Key threats:  
  – Model tampering  
  – Non-auditable upload  
  – Bypassing the registry  

– Enforcement:  
  – OPA Terraform:  
    – `policies/opa/terraform/encryption.rego` — protection of model artifacts  
    – `policies/opa/terraform/tagging.rego` — recording provenance  
  – Checkov:  
    – `policies/checkov/ai/ai_encryption.yaml` — mandatory model encryption  
  – OPA AI:  
    – `policies/opa/ai/ai-data-isolation.rego` — prohibition of direct training → inference access

## Model Registry (Trusted AI Artifacts)

– Purpose: centralized storage of trusted models.  
– Key threats:  
  – Tampering  
  – Rollback attacks  
  – Manual modifications  

– Enforcement:  
  – OPA Terraform:  
    – `policies/opa/terraform/encryption.rego` — protection of artifacts  
    – `policies/opa/terraform/tagging.rego` — versioning, owner, lifecycle  
  – Checkov:  
    – `policies/checkov/ai/ai_encryption.yaml` — immutable storage assumptions  

## Model Registry → Inference

– Purpose: admission of the model into the runtime loop.  
– Key threats:  
  – Execution of unsigned models  
  – Mixing training and inference resources  

– Enforcement:  
  – OPA AI:  
    – `policies/opa/ai/ai-gpu-restrictions.rego` — dedicated inference GPUs  
    – `policies/opa/ai/ai-data-isolation.rego` — isolation of inference data  
  – OPA Terraform:  
    – `policies/opa/terraform/tagging.rego` — environment-aware promotion

## Inference (Runtime Execution)

– Purpose: execution of trusted models on untrusted input data.  
– Key threats:  
  – Privilege escalation  
  – Data leakage  
  – Shared runtime  

– Enforcement:  
  – OPA AI:  
    – `policies/opa/ai/ai-gpu-restrictions.rego` — prohibition of shared GPUs  
    – `policies/opa/ai/ai-data-isolation.rego` — runtime data boundaries  
  – OPA Terraform:  
    – `policies/opa/terraform/naming.rego` — unambiguous separation of loops  

## Inference → Serving

– Purpose: exposing inference outputs outside the platform.  
– Key threats:  
  – Leakage of sensitive data  
  – Public AI access  

– Enforcement:  
  – OPA AI:  
    – `policies/opa/ai/no-public-ai.rego` — prohibition of public serving without explicit allow  
  – Checkov:  
    – `policies/checkov/ai/ai_network.yaml` — control of exposure paths  
  – OPA Terraform:  
    – `policies/opa/terraform/regions.rego` — geographic serving restrictions

## Architectural Invariant

– No data flow is allowed without a policy.  
– Promotion always passes through AI-specific enforcement.  
– AI, data, and models are treated as a **single security loop**.  
– Policies define the **allowed architecture**, not style checks.

---

# Explicit Non-Goals and Model Limitations

## What the Platform Consciously Does Not Solve
– ML correctness  
– Model bias  
– Interpretability  

## Platform Responsibility Boundaries
The platform guarantees **control and isolation**,  
but not the business outcome of models.

---

# Relation to Other Documents

## security-model.md
Threats → controls → enforcement.

## workflows.md
When and under what conditions transitions occur.

## architecture.md
Overall platform loop and layers.
