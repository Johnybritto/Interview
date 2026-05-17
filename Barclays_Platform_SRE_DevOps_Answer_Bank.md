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

## Advanced Production Troubleshooting

### Pods stuck in Pending

Common causes:
- insufficient resources
- taints/tolerations mismatch
- PVC binding failure
- topology spread constraints
- IP exhaustion

Debug commands:
```bash
kubectl describe pod <pod>
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl top nodes
```

AWS-specific issue:
EKS clusters often hit ENI/IP exhaustion.

Azure-specific issue:
AKS with Azure CNI may run out of subnet IPs.

---

### Node becomes NotReady

Root causes:
- kubelet failure
- memory pressure
- disk pressure
- network partition
- certificate expiry

Recovery steps:
1. cordon node
2. drain workloads
3. inspect kubelet logs
4. recover node
5. rejoin cluster

Commands:
```bash
kubectl describe node <node>
systemctl status kubelet
journalctl -u kubelet
```

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

AWS Example:
ALB Ingress Controller routes traffic to EKS services.

Azure Example:
Application Gateway Ingress Controller routes traffic to AKS.

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

### mTLS Troubleshooting

Common issue:
Services stop communicating after enabling strict mTLS.

Debug steps:
```bash
istioctl proxy-status
istioctl analyze
kubectl logs <envoy-sidecar>
```

Top interview follow-ups:
- permissive vs strict mTLS
- when not to use Istio
- sidecar injection internals

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

---

## OpenShift Upgrade Strategy

Enterprise flow:
1. upgrade non-prod first
2. validate operators
3. verify SCC/policies
4. monitor MachineConfigPools
5. validate ingress/storage

Commands:
```bash
oc get clusterversion
oc get mcp
oc adm upgrade
```

Common failure scenarios:
- incompatible operators
- pending MCP updates
- CRD mismatch
- certificate expiry

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

### ArgoCD Internals

Components:
- API server
- repo server
- application controller
- Redis cache

Scenario:
Manual kubectl changes are automatically reverted after reconciliation.

Commands:
```bash
argocd app sync
argocd app diff
```

---

# Observability

## Difference between monitoring and observability

Monitoring:
Known metrics and alerts.

Observability:
Ability to understand unknown failures using logs, metrics, and traces.

Enterprise Stack:
- Prometheus
- Grafana
- Loki
- Tempo/Jaeger
- OpenTelemetry

Common production issues:
- alert storms
- high-cardinality metrics
- expensive logging
- trace sampling gaps

---

# Terraform

## Terraform state management

Terraform state tracks infrastructure resources.

AWS Best Practice:
- S3 backend
- DynamoDB locking
- KMS encryption

Azure Best Practice:
- Azure Blob backend
- Key Vault integration
- RBAC-controlled access

### Failure Scenario
Terraform apply partially fails.

Recovery:
- inspect state
- import missing resources
- avoid manual drift
- taint/recreate selectively

---

# Python Automation

## Python automation for SRE

Common tasks:
- health checks
- Kubernetes automation
- incident remediation
- API polling
- metrics collection

### Auto-remediation Example
If pod restart count exceeds threshold:
1. gather diagnostics
2. capture logs
3. restart workload
4. notify Slack/Teams
5. create incident ticket

Important point:
Automation must be idempotent and observable.

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

Leadership expectation:
Remain calm and structured under pressure.

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

Requirements:
- multi-cluster architecture
- centralized observability
- GitOps
- self-service platform
- RBAC isolation
- quota management
- network policies

Suggested Stack:
- OpenShift/EKS/AKS
- ArgoCD
- Istio
- Prometheus + Loki + Tempo
- Backstage
- Crossplane

Scaling challenges:
- API server scalability
- etcd growth
- observability cost
- CI/CD bottlenecks

Top interview follow-ups:
- noisy tenant isolation
- disaster recovery design
- cost optimization strategy

---

# Final Advice

For Barclays-level interviews:

Always answer in this order:
1. Explain concept
2. Explain production impact
3. Explain troubleshooting approach
4. Explain AWS/Azure/OpenShift implementation
5. Explain trade-offs

Top candidates explain:
- production failures
- operational recovery
- scalability bottlenecks
- cloud implementation patterns
- architecture trade-offs

That is what differentiates Principal / VP Platform Engineers from standard DevOps engineers.
