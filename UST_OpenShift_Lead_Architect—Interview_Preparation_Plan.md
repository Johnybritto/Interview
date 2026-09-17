# UST OpenShift Lead / Architect — Interview Preparation Plan

**Interview:** UST — OpenShift Lead / Architect  
**Location:** Pune  
**Experience:** 8+ years  
**Preparation priority:** OpenShift platform architecture, lifecycle, troubleshooting, migration, networking, security, operators, storage and enterprise leadership scenarios.

---

## 1. Your Introduction + OpenShift Project Pitch — HIGH

Prepare a strong 2-minute introduction covering:

- OpenShift / Kubernetes platform experience
- VMware / vSphere based platform experience
- Platform ownership and architecture responsibilities
- Production troubleshooting
- Scale and enterprise environments
- Team / stakeholder leadership
- Migration and modernization work

**Rule:** Every claim in the introduction must be something we can defend with follow-up questions and real examples.

---

## 2. OpenShift Architecture & Cluster Lifecycle — HIGH

Revise:

- OpenShift control-plane architecture
- Master / worker responsibilities
- Cluster Operators
- IPI vs UPI
- OpenShift installation on VMware / vSphere
- Bootstrap process
- Node maintenance and replacement
- Cluster upgrade procedure
- MachineConfig and Machine Config Operator (MCO)
- Certificate management
- Cluster health validation before and after changes

We have already prepared UPI installation, OpenShift upgrades and several failure scenarios, so this section is mainly revision.

---

## 3. OpenShift Troubleshooting Scenarios — VERY HIGH

This may be one of the most important areas in the interview.

Prepare these scenarios:

- OpenShift API unavailable
- Worker node `NotReady`
- Pods stuck in `Pending`
- `CrashLoopBackOff`
- Application / Route inaccessible
- DNS failure
- Operator degraded
- Upgrade stuck
- Certificate issue
- PVC / storage issue
- Network connectivity issue
- Ingress failure
- Cluster capacity / scheduling issue

### Troubleshooting answer structure

For every scenario, answer using:

**Symptom → Initial checks → Commands → Diagnosis → Fix → Validation → RCA / Prevention**

The interviewer should see a structured production troubleshooting approach rather than random commands.

---

## 4. Networking + Security — HIGH

Revise:

### Networking

- OpenShift Route vs Kubernetes Ingress
- External request flow into OpenShift
- API endpoint
- Ingress controller / router
- DNS
- Load balancer
- OVN-Kubernetes
- NetworkPolicy
- Firewall and enterprise connectivity troubleshooting

### Security

- RBAC
- Role vs ClusterRole
- RoleBinding vs ClusterRoleBinding
- Service Accounts
- SCC
- `restricted`, `anyuid`, `privileged`
- `runAsUser`
- LDAP / Active Directory identity provider
- Secrets
- Certificates

SCC and Service Account security were already prepared and should be treated as revision.

---

## 5. Operators / OLM — HIGH

This is an area to strengthen before the interview.

Know clearly:

- What an Operator is
- Why Operators are used in OpenShift
- CRD
- Custom Resource
- OperatorHub
- Operator Lifecycle Manager (OLM)
- Subscription
- InstallPlan
- ClusterServiceVersion (CSV)
- Operator upgrade lifecycle
- Automatic vs manual approval
- Troubleshooting a failed / degraded Operator

### Likely scenario

> An Operator becomes degraded after an OpenShift upgrade. How would you troubleshoot it?

Be prepared to explain the complete investigation path rather than only giving commands.

---

## 6. OpenShift Virtualization + KubeVirt + MTV — VERY HIGH

This is one of the biggest preparation gaps relative to the UST JD and therefore deserves disproportionate attention.

Prepare:

- What OpenShift Virtualization is
- KubeVirt architecture basics
- How VMs run on Kubernetes / OpenShift
- KubeVirt CRDs
- `VirtualMachine`
- `VirtualMachineInstance`
- VM networking
- VM storage
- VMware → OpenShift Virtualization migration
- Migration Toolkit for Virtualization (MTV)
- Source and target providers
- Storage mapping
- Network mapping
- Migration Plan
- Cold migration
- Warm migration
- Pre-migration checks
- Migration execution
- Failed migration troubleshooting
- Retry analysis
- Rollback approach
- Post-migration validation

### Architecture-level preparation

Be able to explain how you would assess whether an existing VMware workload is ready to move into OpenShift Virtualization.

Consider:

- CPU / memory requirements
- Storage
- Network dependencies
- Application dependencies
- Security
- Downtime tolerance
- Unsupported devices / configurations
- Rollback strategy
- Business change window

---

## 7. Storage / CSI — MEDIUM-HIGH

Revise:

- StorageClass
- PersistentVolume
- PersistentVolumeClaim
- CSI architecture
- Dynamic provisioning
- Volume attachment
- Mount flow
- PVC stuck in `Pending`
- Volume attachment failures
- Mount failures
- Node/storage dependencies

### Important architect question

Know how to identify whether an issue belongs to:

- OpenShift
- CSI driver
- Storage platform
- Network
- Application

The expectation at Lead / Architect level is not necessarily to operate every storage platform but to isolate the failure domain correctly.

---

## 8. Architect / Lead Scenarios — VERY HIGH

Do not prepare only as an OpenShift administrator.

Expect architecture and leadership scenarios such as:

- Design an enterprise OpenShift platform
- Design OpenShift on VMware / vSphere
- Perform migration readiness assessment
- Capacity planning
- Node sizing
- Multi-cluster strategy
- Production change planning
- OpenShift upgrade with application dependencies
- Rollback planning
- Disaster recovery
- Handling a Sev-1 production incident
- Working with Red Hat Support
- Coordinating Network / Storage / Application / Security teams
- Explaining technical risks to leadership
- Maintaining documentation and audit evidence
- Operating in a regulated environment

---

# UST Interview Pattern — What to Expect

Based on reported UST DevOps / senior infrastructure interview experiences, preparation should assume:

- Multiple technical / techno-managerial discussions may occur
- Questions are often driven from the candidate's resume
- Scenario-based troubleshooting is important
- Candidates may be asked to describe major production incidents and exactly how they solved them
- Kubernetes / container / CI-CD fundamentals may still appear
- Senior candidates are expected to explain decisions, not only commands

For this particular role, however, the UST JD is much more OpenShift-specific than a generic DevOps role.

Therefore likely probing areas are:

- Cluster lifecycle
- OpenShift troubleshooting
- Networking
- OLM / Operators
- Security
- KubeVirt / MTV
- Storage / CSI
- Enterprise migration
- Production incident management

### Expected interview style

Instead of only asking:

> What is Kubernetes?

expect questions more like:

> An Operator is degraded after an upgrade. What do you check first?

or:

> A VMware workload migration using MTV failed halfway. How do you investigate and recover?

or:

> Users can access the OpenShift API, but application Routes are failing. How would you isolate the issue?

---

# Preparation Priority

## Tier 1 — MUST CRACK

1. OpenShift troubleshooting
2. Cluster upgrade / lifecycle
3. OpenShift networking
4. MTV / KubeVirt migration
5. Operators / OLM

## Tier 2 — MUST BE COMFORTABLE

1. SCC / RBAC / LDAP
2. Storage / CSI
3. OpenShift architecture / design
4. Production incident stories
5. Migration readiness and rollback planning

## Tier 3 — QUICK REVISION

1. Kubernetes fundamentals
2. Docker
3. CI/CD
4. YAML
5. General DevOps concepts

---

# Existing Strengths We Can Reuse

We have already prepared several important areas that directly match this interview:

- OpenShift installation on vSphere
- IPI / UPI concepts
- UPI build process
- OpenShift upgrade procedure
- Upgrade dependency handling
- SCC
- `anyuid`
- `privileged`
- Service Accounts
- Kubernetes / OpenShift troubleshooting scenarios
- Routes vs Ingress
- Networking fundamentals

These should be treated as revision instead of starting again.

---

# Biggest Gaps to Close Before the Interview

## Gap 1 — OpenShift Virtualization / KubeVirt / MTV

Need an interview-ready architecture and troubleshooting understanding.

## Gap 2 — Operators / OLM

Need to be able to explain OLM objects, lifecycle and degraded Operator troubleshooting cleanly.

## Gap 3 — Lead / Architect Storytelling

Need 3–4 strong real-world stories:

1. Major production incident
2. OpenShift / Kubernetes upgrade
3. Migration / modernization
4. Cross-team architecture or troubleshooting problem

For each story prepare:

**Situation → Scale → Problem → Your responsibility → Investigation / design → Decision → Outcome → Lesson / prevention**

---

# Interview Strategy

The goal is not to memorize 100 answers.

The goal is to be able to confidently handle follow-up questions around a smaller number of core areas:

**OpenShift Architecture → Lifecycle → Networking → Security → Operators → Storage → Virtualization / MTV → Troubleshooting → Enterprise Architecture**

For scenario questions, always think aloud using a structured diagnostic flow and clearly separate:

- What you would check
- Why you would check it
- What result you expect
- What the result would tell you
- What action you would take next

This is the level expected from an OpenShift Lead / Architect.
