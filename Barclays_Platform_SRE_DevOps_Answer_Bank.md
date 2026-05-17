# Barclays Platform / SRE / DevOps Interview Answer Bank

## Practical Senior-Level Answers

This document contains practical answers and production-oriented explanations for Barclays-style Platform Engineering, Kubernetes, OpenShift, SRE, DevOps, GitOps, Cloud, Python, Terraform, Leadership, and System Design interviews.

---

# Kubernetes

## Explain Kubernetes architecture end-to-end

Kubernetes consists of a control plane and worker nodes.

Control Plane:
- API Server
- etcd
- Scheduler
- Controller Manager

Worker Nodes:
- kubelet
- container runtime
- kube-proxy
- CNI plugins

Flow:
1. User applies manifest
2. API server validates request
3. etcd stores desired state
4. Scheduler selects node
5. kubelet creates pod
6. CNI configures networking
7. Controllers continuously reconcile state

Production Focus:
- API server latency directly impacts cluster responsiveness
- etcd health is critical
- Scheduler bottlenecks appear in large clusters

AWS Example:
EKS manages the control plane while EC2 worker nodes run workloads.

Azure Example:
AKS integrates with VMSS, Azure CNI, and Entra ID.

---

## What happens internally when a pod is created?

1. Request reaches API server
2. Authentication & RBAC checks happen
3. Admission controllers validate request
4. Object stored in etcd
5. Scheduler assigns node
6. kubelet receives assignment
7. Images pulled
8. CNI creates network
9. Pod starts running

Common Scenario:
Pod stuck in Pending:
- insufficient CPU/memory
- taints/tolerations mismatch
- PVC issue
- node selector mismatch

---

# Networking

## Explain packet flow from Ingress → Service → Pod

Traffic Flow:
Client → Load Balancer → Ingress Controller → Service → kube-proxy → Pod

Important Components:
- Ingress Controller
- kube-proxy
- CNI
- iptables/IPVS

Troubleshooting:
- verify endpoints
- check ingress rules
- validate DNS
- inspect Network Policies

AWS Example:
ALB Ingress Controller routes traffic to EKS services.

Azure Example:
Application Gateway Ingress Controller routes traffic to AKS.

---

# OpenShift

## Difference between OpenShift and Kubernetes

OpenShift is an enterprise Kubernetes platform with:
- built-in registry
- OAuth integration
- SCC security model
- Operator Lifecycle Manager
- integrated CI/CD
- enterprise support

Kubernetes is the upstream orchestration platform.

---

## What are SCCs?

Security Context Constraints control:
- privileged access
- UID ranges
- capabilities
- host mounts

They provide stronger multi-tenant isolation.

---

# GitOps

## Explain GitOps workflow

1. Developer commits change
2. Git becomes source of truth
3. ArgoCD/Flux detects drift
4. Cluster reconciles automatically

Benefits:
- auditability
- rollback
- consistency
- reduced manual changes

Scenario:
Manual kubectl change gets reverted automatically by reconciliation.

---

# Istio / Service Mesh

## Explain Istio architecture

Components:
- Istiod (control plane)
- Envoy sidecars (data plane)

Capabilities:
- mTLS
- traffic shifting
- retries
- circuit breaking
- observability

Production Example:
Canary deployment using VirtualService with 90/10 traffic split.

---

# SRE

## What are SLI/SLO/SLA?

SLI:
Measured metric (availability, latency)

SLO:
Target objective
Example: 99.9% availability

SLA:
Business/legal agreement

---

## Explain error budget

Error budget defines acceptable unreliability.

Example:
99.9% uptime allows ~43 minutes downtime/month.

If error budget exhausted:
- stop feature releases
- focus on reliability improvements

---

# Observability

## Difference between monitoring and observability

Monitoring:
Known metrics and alerts.

Observability:
Ability to understand unknown failures using logs, metrics, and traces.

Stack Example:
- Prometheus
- Grafana
- Loki
- Jaeger
- OpenTelemetry

---

# Terraform

## Terraform state management

Terraform state tracks infrastructure resources.

Best Practice:
- remote backend
- state locking
- versioning enabled

AWS:
S3 + DynamoDB locking

Azure:
Azure Blob backend + storage locking

---

# Python Automation

## Python automation for SRE

Common tasks:
- health checks
- Kubernetes automation
- incident remediation
- API polling
- metrics collection

Example:
Python script checks pod failures and restarts unhealthy workloads automatically.

---

# Leadership

## How do you handle Sev1 incidents?

Approach:
1. Stabilize service
2. Establish communication bridge
3. Assign clear ownership
4. Reduce blast radius
5. Restore service
6. Conduct RCA
7. Prevent recurrence

Key leadership trait:
Stay calm and structured.

---

# Platform Engineering

## What is Platform Engineering?

Platform Engineering builds internal platforms enabling developers to self-service infrastructure safely and consistently.

Key Concepts:
- golden paths
- self-service
- GitOps
- governance
- developer experience

---

# Barclays-Style System Design

## Design Kubernetes platform for 10,000 developers

Key Requirements:
- multi-cluster architecture
- strong RBAC
- GitOps
- centralized observability
- self-service platform
- quota management
- network isolation

Suggested Design:
- OpenShift/EKS/AKS multi-cluster
- ArgoCD for GitOps
- Istio for service mesh
- Prometheus + Loki + Tempo
- Backstage developer portal

---

# Final Advice

For Barclays-level interviews:

Always answer in this order:
1. Explain concept
2. Explain production impact
3. Explain troubleshooting approach
4. Explain AWS/Azure/OpenShift implementation
5. Explain trade-offs

The differentiator at Lead/VP level is:
- troubleshooting depth
- architecture thinking
- reliability mindset
- automation maturity
- leadership clarity
