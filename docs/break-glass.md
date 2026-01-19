# Table of Contents

- [Document Purpose and Break-Glass Philosophy](#document-purpose-and-break-glass-philosophy)
- [Break-Glass as a Governance Failure Path](#break-glass-as-a-governance-failure-path)
- [Conditions for Applying Break-Glass](#conditions-for-applying-break-glass)
  - [Classes of Acceptable Incidents](#classes-of-acceptable-incidents)
  - [What Does Not Constitute a Basis for Break-Glass](#what-does-not-constitute-a-basis-for-break-glass)
- [Emergency Access Subjects](#emergency-access-subjects)
  - [Who Can Initiate Break-Glass](#who-can-initiate-break-glass)
  - [Who Can Approve Break-Glass](#who-can-approve-break-glass)
  - [Role Separation](#role-separation)
- [Scope and Limitations of Emergency Access](#scope-and-limitations-of-emergency-access)
  - [Minimizing Blast Radius](#minimizing-blast-radius)
  - [Resource Restrictions](#resource-restrictions)
- [TTL and Time-Bound Override](#ttl-and-time-bound-override)
  - [Time Constraints](#time-constraints)
  - [Automatic Expiration of Privileges](#automatic-expiration-of-privileges)
  - [Extension](#extension)
- [Audit and Traceability](#audit-and-traceability)
  - [Mandatory Audit Events](#mandatory-audit-events)
  - [Audit Trail Storage](#audit-trail-storage)
  - [Immutability](#immutability)
- [Integration into Lifecycle](#integration-into-lifecycle)
  - [Link with Workflows](#link-with-workflows)
  - [Link with State Backend](#link-with-state-backend)
  - [Link with Security Model](#link-with-security-model)
- [Post-Incident Normalization](#post-incident-normalization)
  - [Revocation of Privileges](#revocation-of-privileges)
  - [Return to Standard Model](#return-to-standard-model)
  - [Post-Mortem](#post-mortem)
- [Explicit Non-Goals](#explicit-non-goals)
- [Relationship with Other Documents](#relationship-with-other-documents)

---

## Document Purpose and Break-Glass Philosophy

`break-glass.md` — break-glass access and emergency scenarios.

Break-glass defines a strictly controlled emergency access mechanism, applied **only** when the standard access management model and infrastructure governance processes fail.

Break-glass is not intended for:
- accelerating work,
- bypassing governance,
- circumventing RBAC/IAM inconveniences,
- compensating for poor process design.

Break-glass is a deliberate, documented, and auditable path **under governance failure conditions**, not an operational norm.

---

## Break-Glass as a Governance Failure Path

Break-glass is interpreted as:
- an indicator of a critical incident,
- a signal of a breach or insufficiency in the current governance model,
- an event requiring subsequent analysis and adjustment of architecture, policies, or processes.

Any use of break-glass:
- is considered a security incident,
- automatically triggers post-incident procedures,
- is treated as an exception, not an accepted practice.

---

## Conditions for Applying Break-Glass

### Classes of Acceptable Incidents

Break-glass is allowed **only** in the following cases:

- Loss of access to critical infrastructure that blocks service recovery.
- Compromise or failure of IAM/RBAC preventing restoration actions.
- Critical security incident requiring immediate intervention.
- Complete failure of CI/CD, GitOps, or control-plane with business-critical downtime.
- Threat of data loss or state integrity violation.

Criterion:
no alternative, standard, approved recovery path exists.

### What Does Not Constitute a Basis for Break-Glass

Break-glass is **prohibited** if:

- deployment or hotfix acceleration is required,
- access is lost due to user error,
- the process is perceived as "too slow",
- roles or policies are not configured,
- a standard workflow exists to resolve the issue.

---

## Emergency Access Subjects

### Who Can Initiate Break-Glass

Initiators can be:
- on-call lead,
- security lead,
- platform owner.

Responsibilities of the initiator:
- record the incident,
- specify the purpose and scope,
- trigger the approval procedure.

### Who Can Approve Break-Glass

Approvers are:
- security owner,
- head of platform/infrastructure,
- delegated authority according to security policy.

Principle:
initiator ≠ approver.

### Role Separation

Mandatory:
- at least two people,
- different roles,
- different domains of responsibility.

---

## Scope and Limitations of Emergency Access

### Minimizing Blast Radius

Emergency access:
- is granted strictly for the task,
- is limited to specific resources,
- is not expanded without justification.

Prohibited:
- blanket-admin,
- "just-in-case" access.

### Resource Restrictions

Scope is defined in advance:
- accounts,
- regions,
- services,
- allowed operations.

---

## TTL and Time-Bound Override

### Time Constraints

Each break-glass access:
- has a predefined TTL,
- measured in hours,
- minimally sufficient.

### Automatic Expiration of Privileges

Privileges:
- are revoked automatically,
- do not require manual cleanup,
- cannot remain active.

### Extension

Extension:
- requires a new approval,
- is recorded as a separate incident.

---

## Audit and Traceability

### Mandatory Audit Events

The following are logged:
- initiator,
- approver,
- activation time,
- scope,
- all actions,
- revocation moment.

### Audit Trail Storage

Audit trail:
- centralized,
- immutable,
- accessible to security and audit roles.

### Immutability

Requirements:
- append-only,
- integrity verification,
- no deletion or modification allowed.

---

## Integration into Lifecycle

### Link with Workflows

Break-glass:
- is an exception to workflows,
- serves as an emergency fallback,
- triggers post-mortem.

### Link with State Backend

Emergency access to state:
- carries maximum risk,
- requires separate approval,
- has minimal TTL and scope.

State is a security asset.

### Link with Security Model

Break-glass:
- does not expand standard roles,
- is not used continuously,
- exists outside the main RBAC model.

---

## Post-Incident Normalization

### Revocation of Privileges

After the incident:
- access revocation is confirmed,
- absence of residual access is verified,
- IAM/RBAC is validated.

### Return to Standard Model

Mandatory:
- return to GitOps and CI-driven management,
- prohibition of manual changes,
- drift verification.

### Post-Mortem

Conducted:
- analysis of the break-glass cause,
- identification of governance defects,
- recording of corrective actions.

Goal:
minimize repeat use of break-glass.

---

## Explicit Non-Goals

Break-glass:
- is not admin convenience,
- is not a shortcut,
- is not a workaround for poor architecture.

Use without an incident is a policy violation.

---

## Relationship with Other Documents

- security-model.md — roles, policies, access control  
- workflows.md — lifecycle of changes and incidents  
- state-backend.md — protection of state as a security asset
