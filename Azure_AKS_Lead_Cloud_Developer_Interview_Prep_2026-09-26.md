# Azure DevOps / AKS + Lead Cloud Developer — Formidable Interview Preparation Plan

**Interview date:** 26 September 2026  
**Interview window:** 9:30 AM–2:00 PM  
**Location:** Pune (Hinjawadi)  
**Primary role:** Azure DevOps / AKS Engineer  
**Secondary / possible related role:** Lead Cloud Developer  
**Experience band:** 8–15 years  
**Preparation window:** 22–25 September + interview-morning revision

---

# 0. Mission

This preparation must cover both possible interview directions without diluting the high-probability areas.

The primary assumption is that the interview will be driven by:

1. AKS / Kubernetes
2. Azure networking and identity
3. Azure DevOps / GitHub Actions CI/CD
4. Terraform / ARM
5. Kubernetes and cloud troubleshooting
6. Python / PowerShell automation
7. Ansible Tower
8. Production architecture / HA / DR / observability

The second layer protects against the **Lead Cloud Developer** JD:

1. Cloud-native application architecture
2. REST/API design
3. Event-driven and serverless systems
4. Containers vs serverless
5. Data modelling / integration
6. Application security
7. Testing strategy
8. AWS ↔ Azure service mapping
9. Software engineering / code quality / mentoring

**Rule:** Prepare for the role, but answer at **Lead / Senior Platform Engineer depth**.

---

# 1. Interview Positioning

## Core pitch

Position yourself as:

> A senior platform/cloud engineer who has built and operated enterprise Kubernetes platforms, automated infrastructure and delivery workflows, supported large developer populations, handled production incidents, and increasingly works at the boundary between platform engineering and cloud-native application architecture.

Do **not** pretend to be a traditional full-stack application developer.

Instead, connect your real experience to the JD through:

- Kubernetes / OpenShift / PCF platform engineering
- AWS / Azure cloud infrastructure
- Terraform / Ansible
- CI/CD and GitOps
- Python / shell automation
- developer self-service / IDP workflows
- security / RBAC / identity
- observability
- production troubleshooting
- architecture and migration work

### Defensible IDP example

Use the namespace/self-service workflow:

**Developer portal request → input validation → GitLab repo/files created → platform configuration generated → Argo CD sync → target Kubernetes cluster → validation / policy checks**

This is useful because it shows software workflow + platform automation + GitOps rather than only infrastructure administration.

---

# 2. Preparation Priority Matrix

| Priority | Area | Expected depth |
|---|---|---|
| P0 | AKS architecture, lifecycle, upgrades | Deep |
| P0 | Kubernetes troubleshooting | Deep |
| P0 | Azure networking | Deep |
| P0 | CI/CD — Azure DevOps + GitHub Actions | Deep |
| P0 | Terraform | Deep |
| P1 | Azure identity/security/Key Vault | Strong |
| P1 | Python automation | Strong |
| P1 | Containers / Docker | Strong |
| P1 | Observability / SRE | Strong |
| P1 | HA / DR / multi-region design | Strong |
| P1 | Git / branching / release strategy | Strong |
| P2 | Ansible Tower | Comfortable |
| P2 | ARM templates / Bicep concepts | Comfortable |
| P2 | Cloud-native APIs / microservices | Comfortable |
| P2 | Event-driven / serverless | Comfortable |
| P2 | AWS ↔ Azure mapping | Comfortable |
| P3 | Full-stack UI technologies | Awareness only unless panel pushes |

---

# 3. Four-Day Battle Plan

## Day 1 — 22 Sep — AKS + Kubernetes + Networking

### Goal

Be able to whiteboard AKS architecture and troubleshoot common failures without guessing.

### AKS architecture

Revise and explain:

- Managed control plane
- API server
- etcd responsibility
- scheduler / controllers
- system node pool vs user node pool
- VMSS-based worker nodes
- kubelet
- container runtime
- CoreDNS
- kube-proxy / service routing
- Azure CNI options
- ingress controller
- Azure Load Balancer / Application Gateway
- ACR integration
- managed identities
- Key Vault integration
- Azure Monitor / Container Insights

### AKS cluster build flow

Be able to explain:

**Subscription → Resource Group → VNet/Subnet → Identity → AKS control plane → node pools → CNI → ACR → ingress → observability → policy/security → application onboarding**

### AKS upgrades

Must explain end-to-end:

- supported version check
- control-plane upgrade
- node-pool Kubernetes version upgrade
- surge settings
- PodDisruptionBudget
- drain / cordon behavior
- workload validation
- rollback limitations
- node-image upgrade
- snapshot-based node pool strategy when version control / fallback is required

Know the difference between:

- Kubernetes version upgrade
- node image upgrade
- node pool replacement
- control-plane upgrade

### Kubernetes scheduling

Revise:

- requests / limits
- nodeSelector
- affinity / anti-affinity
- taints / tolerations
- topology spread
- PDB
- priorities
- HPA
- VPA concept
- Cluster Autoscaler

### Kubernetes troubleshooting — must answer using structure

**Symptom → scope → recent change → events/logs → dependency checks → root cause → remediation → validation → prevention**

Scenarios:

1. Pod Pending
2. CrashLoopBackOff
3. ImagePullBackOff
4. Node NotReady
5. Service reachable internally but not externally
6. DNS failure
7. Ingress 502/503
8. PVC Pending
9. Application cannot reach database
10. High CPU / memory
11. HPA not scaling
12. Cluster Autoscaler not adding nodes
13. Deployment rollout stuck
14. NetworkPolicy blocking traffic
15. Certificate/TLS issue

### Networking — critical whiteboard

Be able to draw and narrate:

```text
Client
  |
Azure DNS / Front Door (if global)
  |
Application Gateway / Azure Load Balancer
  |
Ingress Controller
  |
Kubernetes Service
  |
Endpoint / Pod
  |
Downstream service / database
```

For each hop explain:

- DNS resolution
- L4 vs L7
- frontend IP
- backend pool
- health probe
- ingress rule
- Service selector
- endpoint discovery
- kube-proxy / dataplane
- CNI routing
- NSG / UDR / firewall influence
- SNAT considerations

### Day 1 output

You must be able to answer, without notes:

- Explain AKS architecture.
- Explain packet flow from Internet to pod.
- Explain pod-to-pod traffic across nodes.
- Explain AKS upgrade.
- Troubleshoot Pending / CrashLoopBackOff / 502.
- Explain Azure CNI.
- Explain HPA vs Cluster Autoscaler.

---

## Day 2 — 23 Sep — CI/CD + Terraform + Git + Containers

### Goal

Design an enterprise deployment pipeline for many microservices and defend every decision.

## Azure DevOps pipeline

Whiteboard:

```text
Commit
  ↓
Pull Request
  ↓
Lint / Unit Test
  ↓
SAST / Dependency Scan
  ↓
Docker Build
  ↓
Image Scan
  ↓
Push to ACR
  ↓
Package Helm / Manifest
  ↓
Deploy Dev
  ↓
Integration / Smoke Tests
  ↓
Approval / Policy Gate
  ↓
Stage
  ↓
Production
  ↓
Post-deploy validation
  ↓
Rollback if required
```

Know:

- stages
- jobs
- steps
- agents
- Microsoft-hosted vs self-hosted agents
- variable groups
- templates
- environments
- approvals/checks
- service connections
- artifacts
- pipeline caching
- secrets
- parallel jobs
- deployment jobs
- conditions
- reusable YAML

## GitHub Actions

Know:

- workflow
- event/trigger
- job
- step
- action
- runner
- GitHub-hosted vs self-hosted
- environments
- secrets
- OIDC/workload identity
- artifacts
- cache
- matrix strategy
- reusable workflows

### Key comparison

Be ready to compare:

**Azure DevOps agent ↔ GitHub Actions runner ↔ GitLab runner**

Explain where the executor actually runs and how Kubernetes/self-hosted runners create workloads.

### Promotion strategy

Explain three approaches:

1. Build once, promote same immutable artifact
2. Helm values per environment
3. GitOps repository per environment / environment directories

Preferred principle:

> Build once, promote the same image digest through environments.

### Deployment strategies

Know:

- rolling
- recreate
- blue/green
- canary
- feature flags
- rollback
- progressive delivery

## Terraform

Must know deeply:

- provider
- resource
- data source
- variable
- output
- locals
- module
- state
- remote backend
- state locking
- plan
- apply
- destroy
- import
- refresh/drift concept
- lifecycle
- depends_on
- count
- for_each
- dynamic blocks concept
- workspaces
- environment separation
- secrets

### Enterprise Terraform answer

For a multi-engineer setup:

**Git PR → CI validation → terraform fmt/validate → security scan → plan → review → approval → controlled apply → remote backend → locking → RBAC → audit trail**

Azure backend:

**Storage Account + Blob Container + RBAC**, with state protected and environment separation.

### Terraform scenarios

Prepare:

- resource changed manually in Azure
- state file lost/corrupted
- resource exists but not in state
- same module for dev/stage/prod
- secret passed into Terraform
- two engineers apply simultaneously
- refactor resource/module without recreation
- provider upgrade
- failed partial apply

## Git

Know:

- merge vs rebase
- pull request
- branch protection
- trunk-based vs GitFlow
- tags/releases
- revert vs reset
- cherry-pick
- conflict resolution
- protected main branch

## Containers

Know:

- Dockerfile
- image layers
- ENTRYPOINT vs CMD
- multi-stage builds
- .dockerignore
- non-root containers
- image scanning
- immutable tags/digests
- ACR
- container startup troubleshooting

### Day 2 output

Whiteboard one complete solution:

> Design CI/CD for 50 microservices deployed to multiple AKS clusters across dev, stage and production.

You must cover:

- reusable templates
- parallel builds
- artifact/image management
- security
- environment promotion
- secrets
- approvals
- rollback
- observability
- GitOps option

---

## Day 3 — 24 Sep — Azure Security + Automation + Cloud-Native Development

### Goal

Close the gap between DevOps/platform engineering and Lead Cloud Developer expectations.

## Azure identity/security

Revise:

- Entra ID
- Azure RBAC
- managed identity
- service principal
- workload identity
- OIDC federation
- Key Vault
- secrets/certificates/keys
- Private Endpoint
- NSG
- least privilege
- pod/workload identity
- ACR authentication
- AKS API access
- Kubernetes RBAC vs Azure RBAC

Be ready for:

> How would an AKS workload securely access Key Vault without storing credentials?

Answer around:

**Workload Identity/OIDC → managed identity → Key Vault RBAC/policy → no static client secret.**

## Python

Do not prepare competitive programming.

Prepare practical platform automation:

- functions
- classes/basic OOP
- list/dict/set
- comprehensions
- exceptions
- context managers
- file handling
- JSON/YAML
- REST API using requests
- environment variables
- logging
- retry/backoff concept
- subprocess
- concurrency/async awareness
- unit tests / mocking basics

Hands-on drills:

1. Parse JSON API response.
2. Read YAML and validate required fields.
3. Call Azure/REST endpoint.
4. Retry failed request.
5. Filter Kubernetes objects.
6. Write structured logs.
7. Handle exceptions cleanly.

## Ansible Tower

Know:

- inventory
- playbook
- task
- role
- variable
- handler
- template
- vault
- idempotency
- dynamic inventory
- Tower/AWX job template
- credentials
- workflow
- RBAC
- survey
- scheduling

Be able to answer:

> Terraform vs Ansible?

Use:

- Terraform: desired-state infrastructure provisioning.
- Ansible: configuration/orchestration and procedural operational automation.
- They are complementary.

## ARM templates / Bicep

Need working awareness:

- declarative Azure-native IaC
- resource dependencies
- parameters
- outputs
- modules
- deployment scopes

Be able to compare ARM/Bicep and Terraform without dismissing either.

---

# 4. Lead Cloud Developer Protection Layer

## Cloud-native application architecture

Prepare:

- monolith vs microservices
- synchronous vs asynchronous
- stateless services
- horizontal scaling
- caching
- resilience
- retries
- timeout
- circuit breaker
- idempotency
- rate limiting
- API gateway
- distributed tracing
- eventual consistency
- dead-letter queue

## REST API design

Know:

- resources/nouns
- HTTP methods
- status codes
- versioning
- pagination
- filtering
- validation
- authentication
- authorization
- idempotency
- error model
- correlation ID
- rate limiting

### Example

For an order API:

```text
POST /orders
GET /orders/{id}
GET /orders?customerId=...
PATCH /orders/{id}
```

Discuss duplicate POST handling using an **idempotency key**.

## Event-driven architecture

Whiteboard:

```text
API / Producer
   |
Event / Queue
   |
Consumer
   |
Database / downstream service
   |
Retry → Dead Letter Queue
```

Know why asynchronous architecture helps:

- decoupling
- buffering
- resilience
- independent scaling

Know trade-offs:

- eventual consistency
- duplicate delivery
- ordering
- debugging complexity

## Serverless

Compare:

### AWS
- Lambda
- API Gateway
- EventBridge
- SQS/SNS

### Azure
- Azure Functions
- API Management
- Event Grid
- Service Bus

Prepare:

> Containers vs serverless — when would you choose each?

Consider:

- execution duration
- traffic pattern
- cold start
- runtime control
- portability
- operational overhead
- scaling
- cost model

---

# 5. AWS ↔ Azure Mapping — Must Memorize

| AWS | Azure |
|---|---|
| EKS | AKS |
| EC2 | Azure VM |
| Auto Scaling Group | VM Scale Sets |
| VPC | VNet |
| Security Group | NSG |
| Route 53 | Azure DNS |
| ALB | Application Gateway |
| NLB | Azure Load Balancer |
| CloudFront | Front Door / CDN |
| ECR | ACR |
| IAM | Entra ID + Azure RBAC |
| Secrets Manager | Key Vault |
| CloudWatch | Azure Monitor |
| Lambda | Azure Functions |
| API Gateway | API Management |
| SQS | Service Bus Queue |
| SNS | Service Bus Topic / Event Grid |
| EventBridge | Event Grid |
| RDS | Azure SQL / managed DB services |
| S3 | Blob Storage |
| CloudFormation | ARM/Bicep |
| Transit Gateway | Virtual WAN / hub-spoke networking concepts |

Do not claim services are always exact equivalents. Explain functional similarities and design differences where needed.

---

# 6. System Design Drills

Perform these as 15–20 minute whiteboard exercises.

## Design 1 — Enterprise AKS platform

Requirements:

- multiple application teams
- dev/stage/prod
- secure private networking
- ACR
- Key Vault
- CI/CD
- monitoring
- autoscaling
- DR

Cover:

**Identity → network → cluster → node pools → ingress → secrets → registry → policy → observability → pipeline → backup/DR**

## Design 2 — Multi-region application

Explain:

```text
Users
  |
Global DNS / Front Door
  |
+---------------------------+
|                           |
Region A                    Region B
App Gateway                 App Gateway
AKS                         AKS
Services                    Services
Data                        Data/Replica
```

Discuss:

- active-active vs active-passive
- health checks
- DNS/global routing
- data replication
- RPO/RTO
- session state
- failover
- deployment strategy
- observability

## Design 3 — Cloud-native order system

Components:

- API
- authentication
- database
- queue/event bus
- workers
- caching
- observability
- CI/CD
- secrets
- failure handling

Then map it to both AWS and Azure.

## Design 4 — Platform for thousands of developers

Use your real platform experience.

Discuss:

- namespaces/tenancy
- RBAC
- quotas
- network policies
- templates/golden paths
- developer portal
- GitOps
- observability
- guardrails
- self-service
- cluster strategy
- platform SLOs

---

# 7. Production Troubleshooting Drills

At senior level, do not fire random commands.

Use this structure:

1. Establish blast radius.
2. Check recent changes.
3. Determine infrastructure vs platform vs application.
4. Inspect metrics/events/logs.
5. Trace request path.
6. Validate dependencies.
7. Mitigate impact first where necessary.
8. Identify root cause.
9. Validate recovery.
10. Define prevention/RCA actions.

Practice these scenarios:

1. AKS cluster healthy but application unavailable.
2. Application Gateway returns 502.
3. Pods can resolve DNS but cannot reach external endpoint.
4. Only one node pool has failures.
5. Deployment worked in dev but fails in prod.
6. Terraform pipeline suddenly wants to recreate critical resources.
7. CI pipeline intermittently fails on self-hosted agent.
8. ACR image pull fails.
9. Key Vault access suddenly fails.
10. CPU normal but response latency high.
11. Production deployment causes partial outage.
12. One region fails in a multi-region setup.
13. Kubernetes upgrade causes an add-on incompatibility.
14. Certificate expires.
15. Network change breaks pod-to-database connectivity.

---

# 8. Senior / Lead Behavioural Stories

Prepare **five** defensible stories from real work.

## Story 1 — Major production incident

Must include:

- impact
- your role
- diagnostic method
- cross-team coordination
- mitigation
- root cause
- prevention

## Story 2 — Kubernetes / platform upgrade

Include:

- compatibility
- dependencies
- testing
- change planning
- rollback
- validation

## Story 3 — Migration / modernization

Good option:

**PCF/TAS → Kubernetes**

Explain:

- why migrate
- assessment
- architecture
- application dependencies
- CI/CD changes
- security
- observability
- phased migration
- risk controls

## Story 4 — Automation / developer productivity

Use IDP / namespace provisioning / GitOps workflow.

Explain:

- manual problem
- automation design
- validation/guardrails
- Git repository creation
- Argo CD sync
- developer experience
- measurable operational benefit if defensible

## Story 5 — Architecture disagreement / technical decision

Show:

- alternatives considered
- trade-offs
- stakeholders
- evidence
- decision
- outcome

**Rule:** Never fabricate metrics. If exact numbers are unavailable, describe outcome qualitatively.

---

# 9. High-Probability Question Bank

## AKS / Kubernetes

1. Explain AKS architecture.
2. System vs user node pools?
3. How do you upgrade AKS?
4. Kubernetes upgrade vs node-image upgrade?
5. How do you control disruption during upgrade?
6. How does HPA work?
7. HPA vs Cluster Autoscaler?
8. Pod Pending — troubleshooting?
9. CrashLoopBackOff — troubleshooting?
10. Node NotReady — troubleshooting?
11. How does ingress traffic reach a pod?
12. How do pods communicate across nodes?
13. Azure CNI vs alternative AKS networking modes?
14. How do you secure AKS?
15. How do you design multi-tenant AKS?
16. How do you isolate workloads?
17. How do you manage secrets?
18. How does Kubernetes service discovery work?
19. What happens during a rolling update?
20. How do PDBs affect maintenance?

## Azure

21. VNet/subnet/NSG/UDR relationship?
22. Application Gateway vs Load Balancer?
23. Public vs private AKS?
24. Private Endpoint vs Service Endpoint?
25. Managed identity vs service principal?
26. How does workload identity work?
27. Key Vault integration?
28. Hub-spoke architecture?
29. How do you troubleshoot Azure networking?
30. How would you design multi-region Azure architecture?

## CI/CD

31. Design pipeline for many microservices.
32. Azure DevOps stages/jobs/steps?
33. Self-hosted vs Microsoft-hosted agents?
34. GitHub-hosted vs self-hosted runners?
35. How do you reuse pipeline code?
36. How do you pass artifacts?
37. How do you cache dependencies?
38. How do you promote across environments?
39. How do you manage approvals?
40. How do you manage secrets?
41. Blue/green vs canary?
42. Rollback strategy?
43. GitOps vs pipeline-driven deployment?
44. What should happen when security scan fails?

## Terraform

45. Why state?
46. Remote state?
47. Locking?
48. Modules?
49. Workspaces?
50. Drift?
51. Import?
52. for_each vs count?
53. Lifecycle?
54. How do multiple engineers work safely?
55. How do you handle secrets?
56. Terraform vs ARM/Bicep?
57. Terraform vs Ansible?
58. How do you prevent accidental destruction?

## Python / automation

59. Parse API JSON.
60. Exception handling.
61. Logging.
62. Retry logic.
63. REST API call.
64. Read YAML.
65. Basic OOP.
66. Unit test / mocking.
67. subprocess use.
68. list/dict manipulation.

## Lead Cloud Developer

69. Microservices vs monolith?
70. REST API design?
71. Authentication vs authorization?
72. OAuth/OIDC concept?
73. Idempotency?
74. Sync vs async communication?
75. Queue vs pub/sub?
76. What is eventual consistency?
77. Containers vs serverless?
78. How would you make an API resilient?
79. How do you test cloud-native applications?
80. What observability would you build into an application?

---

# 10. Hands-On Minimum Set

Do not spend all preparation time reading.

Complete these small practical drills:

1. Write a Kubernetes Deployment + Service + Ingress YAML.
2. Add requests/limits, readiness/liveness probes.
3. Add ConfigMap + Secret reference.
4. Write HPA YAML.
5. Write a simple NetworkPolicy.
6. Write a minimal Azure DevOps YAML pipeline.
7. Write a minimal GitHub Actions workflow.
8. Build/push/deploy flow to ACR/AKS conceptually.
9. Write Terraform VNet/subnet skeleton.
10. Write Terraform AKS module invocation.
11. Configure remote backend conceptually.
12. Write Python script to call REST API and parse JSON.
13. Write Python retry/error-handling example.
14. Explain one Ansible role/playbook.
15. Draw Internet → Azure LB/App Gateway → ingress → service → pod.

**Interview standard:** You should be able to write the skeleton from memory. Exact syntax perfection is secondary to understanding.

---

# 11. Day 4 — 25 Sep — Mock Interview Day

Do not learn large new areas unless a critical gap remains.

## Round 1 — 45 min technical

Questions:

- AKS architecture
- networking
- upgrades
- troubleshooting
- Terraform
- CI/CD

Target: concise first answer, then depth on follow-up.

## Round 2 — 45 min architecture

Design:

> Secure enterprise application platform on Azure using AKS for multiple teams.

Then introduce:

- second region
- database failure
- zero-downtime releases
- security requirements
- cost pressure

## Round 3 — 30 min coding / automation

- Python REST API
- dictionary/list manipulation
- parsing YAML/JSON
- exception/retry
- simple test

## Round 4 — 30 min Lead Cloud Developer

- API design
- event-driven architecture
- serverless
- security
- AWS/Azure mapping

## Round 5 — 30 min behavioural / resume probing

Practice:

- tell me about yourself
- biggest production incident
- difficult migration
- architecture decision
- disagreement
- mentoring
- failure/lesson
- why this role

---

# 12. Interview Morning — 26 Sep

Keep this light.

### 45–60 minute revision only

Review:

- 2-minute introduction
- AKS architecture diagram
- packet flow
- AKS upgrade sequence
- CI/CD pipeline diagram
- Terraform state/backend
- Azure identity
- 5 production stories
- AWS↔Azure mapping
- top troubleshooting structure

Do **not** start a new technology topic on interview morning.

---

# 13. Answering Strategy

## For definition questions

Use:

**Definition → why it matters → where you used it → one example**

## For architecture questions

Use:

**Requirements → assumptions → design → traffic/data flow → security → resiliency → observability → deployment → trade-offs**

## For troubleshooting questions

Use:

**Blast radius → recent change → evidence → isolate layer → fix → validate → prevent**

## For experience questions

Use:

**Situation → scale → responsibility → decision/action → result → lesson**

---

# 14. Red Flags to Avoid

Do not:

- claim deep React/Angular/full-stack experience if it is not real
- pretend Azure DevOps syntax is identical to GitLab
- give only kubectl commands without diagnostic reasoning
- say Terraform is always better than ARM/Bicep
- say microservices are always better than monoliths
- say Kubernetes is always better than serverless
- rebuild artifacts separately for each environment
- put secrets directly in YAML/repository
- describe manual production deployments as acceptable standard practice
- answer architecture without security/observability/HA
- give a 10-minute answer when a 60-second answer is sufficient

---

# 15. What Not to Over-Prepare

Given the time available, avoid spending disproportionate time on:

- frontend frameworks
- obscure Azure services
- advanced algorithm problems
- Kubernetes internals at source-code level
- memorizing hundreds of CLI flags
- writing large ARM templates
- certification-style trivia

Prioritize **architecture + troubleshooting + delivery + automation + security**.

---

# 16. Readiness Gate

Before sleeping on 25 Sep, verify you can confidently do all of these:

- [ ] Give a 2-minute introduction aligned to the role.
- [ ] Draw AKS architecture.
- [ ] Explain Internet → pod packet flow.
- [ ] Explain AKS control-plane/node-pool/node-image upgrades.
- [ ] Troubleshoot Pending, CrashLoopBackOff and ingress 502.
- [ ] Explain Azure CNI/networking.
- [ ] Design CI/CD for many microservices.
- [ ] Explain Azure DevOps and GitHub Actions runners/agents.
- [ ] Explain build-once/promote strategy.
- [ ] Explain Terraform remote state/locking/modules/drift.
- [ ] Explain managed identity/workload identity/Key Vault.
- [ ] Write basic production-style Python automation.
- [ ] Explain Terraform vs Ansible.
- [ ] Explain ARM/Bicep vs Terraform.
- [ ] Design multi-region AKS application architecture.
- [ ] Explain REST/API and event-driven fundamentals.
- [ ] Compare containers vs serverless.
- [ ] Map major AWS services to Azure.
- [ ] Tell five strong real production stories.
- [ ] Handle one unknown question without panicking: clarify assumptions, reason from fundamentals, and state what you would verify.

---

# 17. Final Strategy

This interview should **not** be prepared as two unrelated jobs.

Treat it as one stack:

```text
Software / API
      ↓
Container / Serverless
      ↓
CI/CD
      ↓
IaC
      ↓
Azure / AWS
      ↓
AKS / Kubernetes
      ↓
Networking + Security
      ↓
Observability
      ↓
Operations / SRE / Troubleshooting
```

Your strongest zone is the lower and middle layers: **platform, Kubernetes, cloud infrastructure, automation, CI/CD, security and operations**.

The objective is to make those areas extremely strong, while being sufficiently fluent in the upper application layer to handle the Lead Cloud Developer variation.

**Interview goal:** demonstrate that you can design, automate, secure, deploy, operate and troubleshoot a cloud-native platform end-to-end — and explain your decisions like a lead engineer.
