# Barclays Platform / SRE / DevOps Interview Answer Bank

## Deep Practical Handbook for Principal Engineer / VP / Lead Platform Roles

This file is written for Barclays-style Platform Engineering, SRE, OpenShift, Kubernetes, DevOps, Python Automation, Terraform, Cloud, and Leadership interviews.

Use every answer in this format:

1. **Concept** — what it is.
2. **Production impact** — why it matters in a banking/enterprise platform.
3. **Failure scenario** — what can go wrong.
4. **Troubleshooting approach** — how to debug it.
5. **Implementation example** — AWS, Azure, OpenShift, or Kubernetes example.
6. **Trade-off** — what decision needs balancing.

---

# 1. Kubernetes Core

## 1.1 Explain Kubernetes architecture end-to-end

### Answer
Kubernetes is a declarative orchestration platform. Users define desired state through the API server, Kubernetes stores that desired state in etcd, and controllers continuously reconcile the actual state of the cluster to match the desired state.

The control plane contains API Server, etcd, Scheduler, and Controller Manager. Worker nodes run kubelet, container runtime, kube-proxy, and CNI plugins.

### Production Impact
In a financial services platform, Kubernetes architecture matters because platform outages can impact customer-facing APIs, internal trading systems, batch jobs, risk systems, or developer deployment pipelines. A weak control plane design can result in slow deployments, failed rollouts, delayed incident recovery, or compliance gaps.

### Flow
```text
User / CI/CD / GitOps
        ↓
API Server → AuthN/AuthZ → Admission Controllers
        ↓
etcd stores desired state
        ↓
Scheduler assigns pod to node
        ↓
Kubelet starts workload
        ↓
CNI + kube-proxy provide networking
        ↓
Controllers continuously reconcile
```

### AWS Implementation
In EKS, AWS manages the control plane. Worker nodes run as EC2 managed node groups, self-managed node groups, or Fargate profiles. Platform teams still own networking, IAM integration, node scaling, add-ons, observability, and application reliability.

### Azure Implementation
In AKS, Azure manages the control plane and integrates with VM Scale Sets, Azure CNI, Azure Monitor, Azure Policy, Key Vault, and Entra ID. Platform teams must design subnet sizing, network policies, private cluster access, node pools, and upgrade strategy.

### Trade-off
Managed Kubernetes reduces control plane operational burden, but it also limits low-level access to etcd and control plane tuning. Self-managed OpenShift gives more control but requires stronger operational discipline.

### Interview Follow-ups
- What happens if the API server is slow?
- What happens if etcd latency increases?
- How would you design HA for the control plane?
- What parts are managed in EKS/AKS and what parts remain your responsibility?

---

## 1.2 How does kube-apiserver validate requests?

### Answer
The API server validates every request through authentication, authorization, admission control, schema validation, quota checks, and persistence to etcd.

### Request Flow
```text
kubectl/app/CI tool
  → API Server
  → Authentication: who are you?
  → Authorization: are you allowed?
  → Admission: should this request be changed or blocked?
  → Validation: does object match schema?
  → etcd write
```

### Production Impact
This is critical in banking because every cluster action must be auditable, policy-controlled, and least-privilege. A weak API server policy model can allow privileged pods, unapproved images, secrets exposure, or unauthorized production changes.

### Failure Scenario
A deployment fails with an admission webhook error.

### Troubleshooting
```bash
kubectl describe deployment <name>
kubectl get events -n <namespace>
kubectl get validatingwebhookconfiguration
kubectl get mutatingwebhookconfiguration
kubectl logs -n <policy-namespace> <webhook-pod>
```

### Fix
Identify whether the request is blocked by RBAC, quota, Pod Security Admission, OPA Gatekeeper, Kyverno, or a custom webhook. Fix the YAML or update the policy if it is incorrectly blocking valid workloads.

### Best Practice
Admission policies should be tested in audit/warn mode before enforce mode so that production deployments are not unexpectedly blocked.

### Trade-off
Strict admission control improves security and compliance, but overly strict policies can slow developer velocity and create deployment friction.

---

## 1.3 What happens internally when a pod is created?

### Answer
When a pod is created, Kubernetes first stores the pod in etcd as desired state. The scheduler then assigns it to a suitable node. The kubelet on that node pulls the image, mounts volumes, configures networking through CNI, and starts containers.

### Step-by-Step
```text
1. YAML submitted
2. API server validates
3. Pod object stored in etcd
4. Scheduler finds a suitable node
5. Binding object created
6. Kubelet sees assigned pod
7. Container runtime pulls image
8. CNI assigns IP and network route
9. Volumes are mounted
10. Containers start
```

### Failure Scenario: Pod stuck in Pending

#### Common Causes
- No node has enough CPU or memory.
- Node selector or affinity does not match available nodes.
- Taints exist but pod has no toleration.
- PVC is not bound.
- Cluster autoscaler cannot scale due to cloud quota or subnet capacity.
- In EKS/AKS, subnet IP exhaustion prevents pod networking.

#### Troubleshooting
```bash
kubectl describe pod <pod> -n <ns>
kubectl get events -n <ns> --sort-by=.metadata.creationTimestamp
kubectl describe nodes | egrep "Taints|Allocatable|Allocated"
kubectl get pvc -n <ns>
kubectl get pods -n kube-system | grep autoscaler
```

#### AWS Example
On EKS using VPC CNI, each pod gets an IP from the VPC subnet. If the subnet or ENI capacity is exhausted, pods may remain Pending or fail networking. The fix may involve increasing subnet size, enabling prefix delegation, adding node groups, or redesigning IP allocation.

#### Azure Example
On AKS with Azure CNI, pod IPs come from the subnet. If the subnet was undersized, new pods cannot get IPs. The fix is often to create a new node pool with a larger subnet or use Azure CNI Overlay for better IP scale.

#### Prevention
- Define resource requests properly.
- Use cluster autoscaler.
- Size subnets based on pod density.
- Use quota and capacity planning.
- Monitor pending pods and IP utilization.

### Interview Follow-ups
- How do you troubleshoot a pod stuck in ContainerCreating?
- How do you handle ImagePullBackOff?
- How do you debug PVC binding issues?
- How do you prevent IP exhaustion in EKS/AKS?

---

## 1.4 Difference between Deployment and StatefulSet

### Answer
A Deployment is used for stateless applications where pods are interchangeable. A StatefulSet is used when pods need stable identity, ordered deployment, ordered termination, and persistent storage mapping.

### Example
Use Deployment for REST APIs, frontend services, and stateless microservices. Use StatefulSet for Kafka brokers, Elasticsearch nodes, PostgreSQL replicas, or ZooKeeper members.

### Production Impact
Choosing the wrong workload type can cause data loss, unstable cluster identity, or broken quorum-based applications.

### Trade-off
Deployments are easier to scale and replace. StatefulSets provide stability but require more careful storage, backup, and upgrade planning.

---

## 1.5 etcd, Raft, quorum, and recovery

### Answer
etcd is the strongly consistent key-value store for Kubernetes cluster state. It uses Raft consensus, where one leader accepts writes and replicates them to followers. A write is committed only after quorum acknowledgement.

### Production Impact
If etcd is slow, the entire cluster feels slow. Pod scheduling, config changes, deployments, leader elections, and controller operations all depend on etcd health.

### Failure Scenario: etcd quorum lost

#### What happens
Existing workloads continue running, but the control plane cannot reliably accept writes. New pods may not schedule, rollouts may fail, and cluster operations become unavailable.

#### Recovery
```bash
ETCDCTL_API=3 etcdctl endpoint health
ETCDCTL_API=3 etcdctl member list
ETCDCTL_API=3 etcdctl snapshot restore snapshot.db
```

#### Practical Recovery Steps
1. Confirm how many etcd members are healthy.
2. Restore failed member if possible.
3. If quorum cannot be restored, recover from a valid snapshot.
4. Rebuild the cluster membership carefully.
5. Validate API server and controllers after recovery.

### Best Practice
Take regular encrypted etcd backups and periodically test restore. A backup that has never been restored is only a hope, not a recovery plan.

### Trade-off
A 3-member etcd cluster tolerates one failure. A 5-member cluster tolerates two failures but adds replication overhead and latency sensitivity.

### Interview Follow-ups
- What is etcd compaction?
- Why does etcd disk latency matter?
- How do large CRDs affect etcd?
- What is the difference between snapshot restore and member recovery?

---

# 2. Kubernetes Networking

## 2.1 Explain packet flow from Ingress to Service to Pod

### Answer
External traffic usually reaches a cloud load balancer, then an ingress controller, then a Kubernetes Service, and finally one of the backend pods selected by endpoints.

### Flow
```text
Client
  → DNS
  → Cloud Load Balancer
  → Ingress Controller Pod
  → Kubernetes Service
  → EndpointSlice
  → Pod IP
```

### Production Impact
Networking is often the hardest area to troubleshoot because failures can occur at DNS, load balancer, ingress rules, service selectors, kube-proxy, CNI, NetworkPolicy, security groups, NACLs, firewall, MTU, or application readiness.

### Failure Scenario: Ingress returns 502/503

#### Possible Causes
- Ingress rule points to wrong service.
- Service has no endpoints.
- Pod readiness probe is failing.
- Backend port mismatch.
- NetworkPolicy blocks ingress controller.
- Cloud load balancer health checks fail.

#### Troubleshooting
```bash
kubectl get ingress -A
kubectl describe ingress <name> -n <ns>
kubectl get svc -n <ns>
kubectl get endpoints -n <ns>
kubectl get endpointslice -n <ns>
kubectl describe pod <pod> -n <ns>
kubectl logs <ingress-controller-pod> -n <ingress-ns>
```

#### AWS Example
For EKS with AWS Load Balancer Controller, check ALB target group health, security groups, listener rules, subnet tags, and ingress annotations.

#### Azure Example
For AKS with Application Gateway Ingress Controller, check backend pool health, NSG rules, route tables, and AGIC logs.

### Trade-off
Ingress centralizes routing and TLS management, but it can become a shared dependency. For high-criticality platforms, use HA ingress controllers and clear tenant isolation.

---

## 2.2 ClusterIP vs NodePort vs LoadBalancer

### Answer
ClusterIP exposes a service internally inside the cluster. NodePort exposes a service on every node IP at a static port. LoadBalancer provisions an external cloud load balancer.

### Production Impact
Using NodePort directly in enterprise production is usually discouraged because it exposes node-level ports and complicates security. LoadBalancer or Ingress is more manageable.

### Trade-off
ClusterIP is secure but internal only. NodePort is simple but less controlled. LoadBalancer is production-friendly but creates cloud cost and resource sprawl if not governed.

---

## 2.3 CoreDNS and service discovery

### Answer
CoreDNS provides DNS-based service discovery inside Kubernetes. Services get DNS names such as `service.namespace.svc.cluster.local`.

### Failure Scenario: DNS failures

#### Symptoms
- Pods cannot resolve service names.
- Application logs show `no such host`.
- Direct IP works but service name fails.

#### Troubleshooting
```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system deploy/coredns
kubectl exec -it <pod> -- nslookup kubernetes.default
kubectl exec -it <pod> -- cat /etc/resolv.conf
```

#### Causes
- CoreDNS pods down.
- kube-dns service unreachable.
- NetworkPolicy blocking DNS UDP/TCP 53.
- NodeLocal DNS cache issue.
- Upstream DNS failure.

### Best Practice
Run multiple CoreDNS replicas, monitor DNS latency/error rate, and explicitly allow DNS in NetworkPolicies.

---

## 2.4 CNI plugins: Calico, Flannel, Cilium

### Answer
CNI plugins provide pod networking. Flannel provides simple overlay networking. Calico provides routing and network policy. Cilium uses eBPF for high-performance networking, security, and observability.

### Trade-off
Flannel is simple but limited for enterprise policy. Calico is mature for network policy. Cilium provides advanced eBPF capabilities but requires stronger operational expertise.

### Interview Follow-ups
- How do NetworkPolicies get enforced?
- Why is eBPF becoming popular?
- How would you debug MTU mismatch?

---

# 3. OpenShift

## 3.1 Difference between OpenShift and Kubernetes

### Answer
OpenShift is an enterprise Kubernetes distribution. It adds opinionated security, integrated authentication, SCCs, Routes, Operators, built-in registry, Machine API, monitoring, logging integrations, and lifecycle management.

### Production Impact
In a bank, OpenShift is often preferred because it provides stronger enterprise controls, integrated auditability, standardized operations, and supportability.

### Trade-off
OpenShift provides an enterprise-ready platform but is more opinionated and heavier than vanilla Kubernetes. Teams must understand SCCs, Operators, OLM, MachineConfigPools, and upgrade channels.

---

## 3.2 What are SCCs in OpenShift?

### Answer
Security Context Constraints define what a pod is allowed to do in OpenShift, such as running privileged, using host networking, mounting host paths, or running as a specific UID.

### Production Impact
SCCs prevent unsafe workloads from running with excessive privileges. This is critical in multi-tenant clusters where one team’s workload should not compromise the node or other namespaces.

### Failure Scenario
A pod fails with permission denied or forbidden SCC error.

#### Troubleshooting
```bash
oc describe pod <pod> -n <ns>
oc get scc
oc adm policy who-can use scc privileged
```

#### Fix
Use the least privileged SCC that allows the workload to run. Avoid assigning `privileged` SCC unless absolutely required and approved.

### Trade-off
Strict SCCs improve security but may require application teams to modify container images to run as non-root.

---

## 3.3 OpenShift upgrade strategy

### Answer
OpenShift upgrades should be treated as controlled platform changes. The correct approach is to upgrade lower environments first, validate all operators, confirm MachineConfigPools are healthy, check deprecated APIs, and then proceed gradually in production.

### Production Impact
A failed OpenShift upgrade can impact deployment pipelines, ingress, storage, monitoring, and all hosted applications.

### Upgrade Flow
```text
Non-prod upgrade
  → operator compatibility check
  → backup validation
  → MachineConfigPool health check
  → production maintenance/change window
  → upgrade control plane
  → upgrade workers gradually
  → post-upgrade validation
```

### Commands
```bash
oc get clusterversion
oc adm upgrade
oc get co
oc get mcp
oc get nodes
```

### Failure Scenario: MCP stuck updating

#### Causes
- Node failed to drain.
- DaemonSet blocks eviction.
- PDB too strict.
- MachineConfig failed to apply.

#### Troubleshooting
```bash
oc describe mcp <pool>
oc get nodes
oc adm drain <node> --ignore-daemonsets --delete-emptydir-data
oc logs -n openshift-machine-config-operator <pod>
```

### Barclays-Style Best Practice
For a regulated banking environment, upgrades should be backed by change approval, rollback plan, business impact assessment, stakeholder communication, validation checklist, and audit evidence.

### Interview Follow-ups
- How do you plan zero-downtime OpenShift upgrades?
- What if an operator is incompatible with the target version?
- How do PDBs affect node draining?

---

# 4. Istio / Service Mesh

## 4.1 What is service mesh?

### Answer
A service mesh manages service-to-service communication using sidecar proxies or ambient data plane models. It provides mTLS, traffic routing, retries, timeouts, circuit breaking, telemetry, and policy enforcement without requiring application code changes.

### Production Impact
In enterprise microservices, service mesh helps enforce zero-trust communication and improves control over east-west traffic.

### Trade-off
Service mesh improves control and observability, but it also adds operational complexity, sidecar resource overhead, debugging complexity, and upgrade risk.

---

## 4.2 Istio architecture

### Answer
Istio has a control plane called Istiod and a data plane usually made of Envoy proxies. Istiod distributes configuration and certificates. Envoy sidecars intercept inbound and outbound traffic.

### Flow
```text
Service A container
  → Envoy sidecar
  → mTLS connection
  → Envoy sidecar
  → Service B container
```

### Failure Scenario: Service fails after sidecar injection

#### Causes
- App does not handle proxy startup timing.
- mTLS policy mismatch.
- AuthorizationPolicy blocks traffic.
- Port naming is incorrect.
- Readiness probes are intercepted incorrectly.

#### Troubleshooting
```bash
istioctl analyze
istioctl proxy-status
istioctl proxy-config routes <pod>
istioctl proxy-config clusters <pod>
kubectl logs <pod> -c istio-proxy
```

### Fix
Validate PeerAuthentication, DestinationRule, AuthorizationPolicy, and service port names. Use permissive mode during migration and move to strict mode gradually.

### Best Practice
Adopt Istio namespace by namespace. Do not enable strict mTLS globally without testing service dependencies.

---

## 4.3 Canary deployment using Istio

### Example
```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: payments
spec:
  hosts:
  - payments.example.com
  http:
  - route:
    - destination:
        host: payments
        subset: v1
      weight: 90
    - destination:
        host: payments
        subset: v2
      weight: 10
```

### Production Use
Use this to send 10% traffic to a new payments API version and monitor error rate, latency, and business transaction success before increasing traffic.

### Trade-off
Canary reduces blast radius but requires reliable observability and rollback automation. Without metrics, a canary is just a risky partial rollout.

---

# 5. GitOps / ArgoCD / Flux

## 5.1 Explain GitOps workflow

### Answer
GitOps means Git is the source of truth for desired infrastructure and application state. A controller such as ArgoCD or Flux continuously compares Git state with cluster state and reconciles drift.

### Production Impact
GitOps improves auditability, rollback, consistency, and compliance because every platform change is traceable to a commit.

### Failure Scenario: ArgoCD sync fails

#### Causes
- Invalid YAML.
- Missing CRD.
- RBAC permission issue.
- Helm rendering issue.
- Admission policy blocked deployment.

#### Troubleshooting
```bash
argocd app get <app>
argocd app diff <app>
argocd app sync <app>
kubectl describe application <app> -n argocd
```

#### Fix
Resolve manifest errors, install required CRDs before dependent resources, check ArgoCD service account permissions, and validate Helm values.

### Best Practice
Use sync waves for dependency ordering. For example, install CRDs first, then controllers, then application resources.

### Trade-off
GitOps improves consistency but can create friction when emergency manual fixes are needed. The best practice is to make emergency fixes through Git or immediately back-port them to Git.

---

# 6. Observability / SRE

## 6.1 Monitoring vs observability

### Answer
Monitoring tells you whether known conditions are healthy. Observability helps you understand unknown failures using metrics, logs, traces, and events.

### Production Impact
In banking, observability must support fast incident detection, customer impact analysis, audit trails, and RCA evidence.

### Best Practice
Dashboards should be designed around user journeys and SLIs, not just infrastructure metrics.

### Trade-off
More telemetry improves visibility but increases cost, storage, and noise. Mature teams focus on actionable signals.

---

## 6.2 SLI, SLO, SLA, and error budget

### Answer
An SLI is a measured reliability indicator, such as request success rate or p95 latency. An SLO is the target, such as 99.9% successful requests. An SLA is a formal agreement with business/legal consequences. Error budget is the allowed unreliability within the SLO.

### Example
If the payments API has 99.9% monthly availability SLO, the error budget is roughly 43 minutes of allowed downtime per month.

### Production Impact
If the error budget is exhausted, feature releases should slow down and reliability work should take priority.

### Trade-off
Very high SLOs increase engineering cost. Not every internal service needs 99.99%; criticality should drive reliability target.

---

## 6.3 Latency spike scenario

### Scenario
Users report transaction latency increased from 200ms to 3 seconds.

### Troubleshooting Flow
1. Confirm scope: one service, one region, all users, or one tenant.
2. Check golden signals: latency, traffic, errors, saturation.
3. Check recent changes: deployment, config, infra, DB, network.
4. Compare app metrics with infrastructure metrics.
5. Trace request path using distributed tracing.
6. Roll back if issue correlates with deployment.

### Commands / Checks
```bash
kubectl top pods -n <ns>
kubectl describe pod <pod>
kubectl logs <pod> --since=30m
kubectl get events -n <ns>
```

### Possible Root Causes
- CPU throttling due to low limits.
- DB connection pool exhaustion.
- downstream dependency latency.
- DNS latency.
- service mesh retry storm.
- node-level resource saturation.

### Prevention
Use SLO-based alerts, load tests, canary releases, tracing, and proper capacity planning.

---

# 7. Terraform / Cloud

## 7.1 Terraform state management

### Answer
Terraform state maps Terraform configuration to real infrastructure. Without state, Terraform cannot safely know what it owns.

### Production Impact
State corruption or accidental state deletion can cause resource recreation, downtime, or loss of infrastructure control.

### AWS Best Practice
Use S3 backend with versioning, DynamoDB locking, KMS encryption, restricted IAM policies, and separate state files per environment/component.

### Azure Best Practice
Use Azure Blob Storage backend with storage account encryption, RBAC, private endpoint access, and versioning where available.

### Failure Scenario: Terraform apply partially fails

#### Troubleshooting
1. Read the error carefully.
2. Check which resources were created.
3. Compare Terraform state and cloud console.
4. Import missing resources if needed.
5. Re-run plan before apply.

#### Commands
```bash
terraform state list
terraform state show <resource>
terraform import <resource> <id>
terraform plan
```

### Trade-off
One large state file is easier initially but creates blast radius and locking contention. Smaller state files improve isolation but require better dependency management.

---

# 8. AWS / Azure Cloud Architecture

## 8.1 Design highly available architecture

### Answer
A highly available architecture spreads workloads across multiple availability zones, removes single points of failure, uses autoscaling, load balancing, health checks, and resilient data stores.

### AWS Example
Route53 → ALB → EKS node groups across 3 AZs → RDS Multi-AZ / DynamoDB → CloudWatch / X-Ray.

### Azure Example
Azure Front Door → Application Gateway → AKS across Availability Zones → Azure SQL zone redundant / Cosmos DB → Azure Monitor.

### Trade-off
Multi-AZ improves availability but increases cost, network complexity, and data replication considerations.

---

## 8.2 Multi-region failover strategy

### Answer
Multi-region failover means the platform can recover from a full region outage by shifting traffic to another region.

### Patterns
- Active-passive: cheaper and simpler, but slower failover.
- Active-active: faster and more resilient, but complex data consistency and routing.

### Banking Consideration
Financial systems must consider RTO, RPO, regulatory data residency, transaction consistency, and audit requirements.

### Trade-off
Active-active reduces downtime but requires strong data replication, conflict handling, and operational maturity.

---

# 9. Python Automation for SRE

## 9.1 Monitor CPU and memory using Python

### Sample Program
```python
import psutil

CPU_THRESHOLD = 80
MEM_THRESHOLD = 80

cpu = psutil.cpu_percent(interval=1)
mem = psutil.virtual_memory().percent

if cpu > CPU_THRESHOLD or mem > MEM_THRESHOLD:
    print(f"ALERT: CPU={cpu}% MEMORY={mem}%")
else:
    print(f"OK: CPU={cpu}% MEMORY={mem}%")
```

### Interview Explanation
This is a basic node-level health check. In production, I would export this as Prometheus metrics or send alerts to a monitoring system rather than just printing output.

---

## 9.2 Parse logs and count errors

### Sample Program
```python
from collections import Counter

log_file = "application.log"
error_counter = Counter()

with open(log_file, "r", encoding="utf-8") as f:
    for line in f:
        if "ERROR" in line:
            if "timeout" in line.lower():
                error_counter["timeout"] += 1
            elif "connection" in line.lower():
                error_counter["connection"] += 1
            else:
                error_counter["other"] += 1

print(error_counter)
```

### Production Improvement
For large files, stream logs instead of loading them into memory. For enterprise platforms, send parsed logs to Splunk, OpenSearch, or Loki.

---

## 9.3 Kubernetes pod health checker using Python

### Sample Program
```python
from kubernetes import client, config

config.load_kube_config()
v1 = client.CoreV1Api()

namespace = "default"
pods = v1.list_namespaced_pod(namespace=namespace)

for pod in pods.items:
    phase = pod.status.phase
    restarts = 0
    if pod.status.container_statuses:
        restarts = sum(cs.restart_count for cs in pod.status.container_statuses)

    if phase != "Running" or restarts > 3:
        print(f"POD ISSUE: {pod.metadata.name}, phase={phase}, restarts={restarts}")
```

### Interview Explanation
This script checks pod status and restart count. In production, I would add namespace filtering, label selectors, exception handling, metrics export, Slack/Teams notification, and avoid unsafe auto-remediation without guardrails.

---

## 9.4 API retry logic

### Sample Program
```python
import time
import requests

URL = "https://example.com/health"

for attempt in range(1, 4):
    try:
        response = requests.get(URL, timeout=3)
        if response.status_code == 200:
            print("Service healthy")
            break
        print(f"Attempt {attempt}: status={response.status_code}")
    except requests.RequestException as exc:
        print(f"Attempt {attempt}: error={exc}")

    time.sleep(2 ** attempt)
else:
    print("Service unhealthy after retries")
```

### Best Practice
Retries should use timeout, exponential backoff, jitter, and max retry limit. Without limits, retries can amplify outages.

---

# 10. Sev1 Incident Handling — Barclays-Style

## 10.1 How do you handle a Sev1 incident?

### Strong Interview Answer
For a Sev1 incident, my first priority is service stabilization and customer/business impact reduction. I do not start with a deep RCA during the outage. I establish command structure, communicate clearly, reduce blast radius, restore service, and only then perform detailed root cause analysis.

### Barclays / Banking Context
In a bank, a Sev1 may affect payments, trading, customer login, fraud systems, regulatory reporting, or internal critical platforms. The response must consider business impact, regulatory sensitivity, audit trail, communication discipline, and change-control obligations.

### Incident Flow
```text
Detection
  → Severity confirmation
  → Incident commander assigned
  → War room opened
  → Business/technology stakeholders notified
  → Impact assessment
  → Mitigation or rollback
  → Service restoration
  → Monitoring validation
  → RCA
  → Corrective actions
```

### Roles
- **Incident Commander:** owns coordination and decisions.
- **Technical Lead:** drives diagnosis and remediation.
- **Communications Lead:** sends updates to stakeholders.
- **Scribe:** records timeline, actions, and decisions.
- **Business Liaison:** confirms business impact and priority.

### Example Scenario
A production OpenShift cluster hosting internal payment APIs experiences API latency and pod restarts after a new deployment.

### Correct Actions
1. Declare Sev1 if customer/business impact is confirmed.
2. Freeze unrelated deployments.
3. Open bridge/war room.
4. Check latest changes and deployment timeline.
5. Roll back if strong correlation exists.
6. Scale stable version if needed.
7. Check infra health: nodes, API server, DNS, ingress, DB.
8. Communicate every fixed interval with facts, impact, and next action.
9. Confirm recovery using business-level metrics, not only pod status.
10. Start RCA after stabilization.

### What Not To Do
- Do not blame teams during the incident.
- Do not allow multiple people to make uncoordinated production changes.
- Do not chase deep root cause while users are still impacted.
- Do not declare recovery based only on Kubernetes green status.

### RCA Structure
1. Summary
2. Timeline
3. Customer/business impact
4. Technical root cause
5. Contributing factors
6. Detection gaps
7. What went well
8. What went wrong
9. Corrective actions
10. Owners and due dates

### Sample Interview Statement
“In a Sev1, I separate mitigation from root cause. During the incident I focus on restoring service safely, reducing blast radius, and communicating clearly. After recovery, I drive a blameless RCA with concrete preventive actions such as better canary checks, SLO alerts, safer rollout gates, capacity thresholds, or automation improvements.”

### Trade-off
A fast rollback may restore service quickly but can hide root cause temporarily. The right approach is to restore first, preserve evidence, and then investigate in RCA.

---

# 11. Platform Engineering

## 11.1 What is Platform Engineering?

### Answer
Platform Engineering is the practice of building reusable internal platforms that help developers deliver software safely, quickly, and consistently. The platform provides golden paths, self-service infrastructure, CI/CD templates, observability, security policies, and deployment standards.

### Production Impact
A good platform reduces toil, improves compliance, shortens onboarding, and prevents every application team from reinventing Kubernetes, Terraform, CI/CD, and monitoring patterns independently.

### Best Practice
Treat the platform as a product. Measure adoption, developer satisfaction, deployment frequency, lead time, reliability, and support tickets.

### Trade-off
Too much standardization can frustrate advanced teams. Too little standardization creates risk and operational chaos.

---

## 11.2 Design Kubernetes platform for 10,000 developers

### Architecture
```text
Developer Portal / Backstage
        ↓
Golden Path Templates
        ↓
Git Repositories
        ↓
CI Pipeline
        ↓
ArgoCD / Flux
        ↓
Multi-cluster Kubernetes / OpenShift
        ↓
Observability + Security + Policy + Cost Controls
```

### Required Capabilities
- Multi-cluster design to avoid single-cluster blast radius.
- Namespace and tenant isolation.
- RBAC integrated with enterprise identity.
- GitOps for deployment consistency.
- NetworkPolicies for tenant isolation.
- Quotas and LimitRanges for resource fairness.
- Centralized observability.
- Self-service onboarding.
- Security scanning and admission policies.
- DR and backup strategy.

### AWS Implementation
Use EKS clusters per business domain or environment, AWS IAM Identity Center/IAM roles for access, AWS Load Balancer Controller, EBS/EFS CSI, KMS for encryption, CloudWatch/Prometheus for observability, and Route53 for routing.

### Azure Implementation
Use AKS clusters with Entra ID integration, Azure CNI/Overlay, Azure Policy, Key Vault CSI driver, Azure Monitor, Application Gateway/Front Door, and separate node pools per workload class.

### OpenShift Implementation
Use OpenShift projects, SCCs, Routes, OLM-managed operators, MachineSets, GitOps Operator, integrated monitoring, and cluster upgrade channels.

### Trade-off
One large shared cluster improves utilization but increases blast radius. Many smaller clusters improve isolation but increase operational overhead. For 10,000 developers, a federated multi-cluster platform is usually better.

---

# 12. Best Practice Statements — Expanded

## Kubernetes Best Practice
Use resource requests, limits, probes, PDBs, NetworkPolicies, RBAC, and GitOps because production Kubernetes reliability depends on predictable scheduling, controlled access, safe rollout, and fast recovery.

## OpenShift Best Practice
Use SCCs, Operators, MachineConfigPools, upgrade channels, and project-level isolation because OpenShift is designed for enterprise governance and should not be operated like a loose vanilla Kubernetes cluster.

## GitOps Best Practice
Make Git the source of truth and avoid manual cluster changes because manual changes create drift, reduce auditability, and make disaster recovery harder.

## Terraform Best Practice
Use remote state, locking, versioned modules, code review, and separate state boundaries because infrastructure changes can have large blast radius if state is corrupted or shared too broadly.

## SRE Best Practice
Define SLIs and SLOs before creating alerts because alerts without user-impact context create noise and alert fatigue.

## Python Automation Best Practice
Automation must be idempotent, logged, observable, and reversible because unsafe automation can multiply the impact of a small incident.

---

# Final Interview Formula

For every answer, use this structure:

```text
Concept → Production impact → Failure scenario → Troubleshooting → Fix → Prevention → Trade-off
```

This is the difference between a generic DevOps answer and a Principal / VP-level platform engineering answer.
