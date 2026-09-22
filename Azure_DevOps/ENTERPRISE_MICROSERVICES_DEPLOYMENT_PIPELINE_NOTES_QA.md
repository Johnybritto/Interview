# Enterprise Microservices Deployment Pipeline — Azure DevOps Interview Notes & Q&A

> **Interview positioning:** My stronger hands-on CI/CD experience is with GitLab. I understand the enterprise CI/CD architecture and can map those concepts to Azure DevOps. For application-security scanning, my responsibility is best described as integrating and enforcing the pipeline gates rather than administering the SAST/DAST products themselves.

---

## 1. The Core Design

For a large number of microservices, do **not** create one giant pipeline and do **not** let every application team invent a completely different pipeline.

Use:

- one small pipeline definition per microservice
- centrally maintained reusable pipeline templates
- standard security gates
- standard artifact/promotion rules
- environment-level production controls
- separate application code from environment configuration

### Whiteboard

```text
Developer
   |
   v
Feature Branch
   |
   v
Pull Request
   |
   +--> Branch Policy / Review
   |
   v
CI
------------------------------------------------
Lint
Unit Test
SAST
SCA / Dependency Scan
Secret Scan
   |
   v
Docker Build
   |
   +--> Container Image Scan
   +--> Optional SBOM
   |
   v
Push Image to ACR
   |
   +--> Store immutable image digest
   +--> Helm chart / manifest artifact
   |
   v
CD
------------------------------------------------
Deploy DEV
   |
   +--> Smoke Test
   +--> Integration Test
   +--> DAST where applicable
   |
   v
Deploy STAGE
   |
   +--> Regression / integration / policy checks
   |
   v
Production Approval / Environment Gate
   |
   v
Deploy PROD
   |
   +--> Rolling / Canary / Blue-Green
   |
   v
Post-deployment Validation
   |
   +--> Healthy --> Complete
   |
   +--> Failed  --> Rollback
```

---

# 2. Most Important Principle: Build Once, Promote the Same Artifact

The strongest principle to defend is:

> **CI creates the artifact. CD promotes the artifact. CD does not rebuild the application for every environment.**

Example:

```text
payment-service@sha256:abc123

DEV   --> sha256:abc123
STAGE --> sha256:abc123
PROD  --> sha256:abc123
```

Avoid:

```text
payment-service-dev
payment-service-stage
payment-service-prod
```

if they are independently rebuilt.

## Why?

Because the exact binary/image tested in DEV and STAGE should be the one promoted to PROD.

Benefits:

- immutability
- reproducibility
- traceability
- easier rollback
- better auditability
- eliminates environment-specific rebuild drift

---

# 3. How to Scale This for Hundreds of Microservices

Example:

```text
Central Platform Pipeline Repository
|
+-- templates/
|   +-- ci-template.yml
|   +-- security-scan.yml
|   +-- docker-build.yml
|   +-- image-scan.yml
|   +-- helm-package.yml
|   +-- deploy-aks.yml
|   +-- smoke-test.yml
|
+-- standards/
    +-- security thresholds
    +-- deployment rules
    +-- naming conventions
```

Application repositories remain small:

```text
payment-service/
orders-service/
customer-service/
inventory-service/
```

Each application consumes reusable templates and provides parameters such as:

```text
service name
Dockerfile path
Helm chart
deployment environment
resource requirements
```

## Why central templates?

Without central templates, 200 services can eventually become 200 different pipeline implementations.

Typical drift:

```text
Service A --> scans properly
Service B --> skips image scan
Service C --> deploys differently
Service D --> stores credentials in variables
Service E --> has no production gate
```

With centrally managed templates:

```text
Application Team
    --> application code and service-specific settings

Platform Team
    --> pipeline standards
    --> common build templates
    --> deployment patterns
    --> security integration
    --> agent pools
    --> production controls
```

This gives development teams autonomy while preserving governance.

---

# 4. Azure DevOps Concepts — Learn Through GitLab Mapping

My stronger hands-on experience is GitLab, so use this mental translation.

| GitLab | Azure DevOps |
|---|---|
| `.gitlab-ci.yml` | `azure-pipelines.yml` |
| Pipeline | Pipeline |
| Stage | Stage |
| Job | Job |
| `script:` / commands | Step / Task |
| GitLab Runner | Azure DevOps Agent |
| Runner fleet | Agent Pool |
| Shared / hosted runner | Microsoft-hosted Agent |
| Self-managed Runner | Self-hosted Agent |
| CI/CD Variables | Variables / Variable Groups |
| `include` / templates | YAML Templates |
| Environment | Environment |
| Protected environment | Environment + Approvals / Checks |
| Artifacts | Pipeline Artifacts |
| Cache | Pipeline Cache |
| Credentials/integration | Service Connection |
| Conditional rules | Conditions |
| Deployment job | Deployment Job |

## Interview wording

> My stronger practical CI/CD experience has been with GitLab. Azure DevOps uses very similar orchestration concepts. A GitLab Runner is comparable to an Azure DevOps Agent, reusable GitLab includes/templates map to Azure YAML templates, and protected deployment controls map to Azure DevOps Environments and approvals/checks. I therefore design around the CI/CD architecture first and then implement it using the constructs of the selected platform.

---

# 5. Azure DevOps Hierarchy

Remember:

```text
Pipeline
   |
   +-- Stage
        |
        +-- Job
             |
             +-- Step / Task
```

Example:

```text
Stage: Build
|
+-- Job: Test
|    +-- checkout
|    +-- unit test
|
+-- Job: Security
     +-- SAST
     +-- dependency scan

Stage: DeployDev
|
+-- Deployment Job
     +-- helm upgrade
     +-- smoke test
```

---

# 6. Azure DevOps Agent

The **Agent** executes the job.

```text
Azure DevOps
     |
     | schedules job
     v
Agent
     |
     +--> checkout
     +--> build
     +--> docker
     +--> helm
     +--> kubectl
     +--> terraform
```

## GitLab equivalent

```text
Azure DevOps Agent ~= GitLab Runner
```

This is the easiest way to explain it.

---

# 7. Microsoft-hosted vs Self-hosted Agent

| Area | Microsoft-hosted | Self-hosted |
|---|---|---|
| Managed by | Microsoft | Organization |
| Machine lifecycle | Ephemeral | Organization controlled |
| Custom tooling | Usually installed per job | Can be preinstalled |
| Private-network access | More restrictive | Very useful |
| Maintenance | Microsoft | Platform team |
| Persistent local cache | Limited | Possible |
| Good use case | generic CI | private AKS/internal systems |

A good architecture could be:

```text
Microsoft-hosted Agent
    |
    +--> lint
    +--> unit test
    +--> SAST/SCA
    +--> build
```

and:

```text
Self-hosted Agent
    |
    +--> Private VNet
            |
            v
        Private AKS
```

## Decision to defend

If the AKS API is private and reachable only through the corporate/private network, a self-hosted or private agent with the required network path is often appropriate.

---

# 8. Service Connections

A Service Connection answers:

> How is Azure DevOps authorized to access Azure resources?

Conceptually:

```text
Azure DevOps Pipeline
       |
       | Service Connection
       v
Microsoft Entra / Azure Identity
       |
       v
Azure resources
       |
       +--> ACR
       +--> AKS
       +--> Resource Group
```

Prefer separate access boundaries such as:

```text
dev-service-connection
stage-service-connection
prod-service-connection
```

rather than one over-privileged global connection.

Use least privilege.

Where possible, prefer federated/workload identity rather than storing long-lived credentials.

---

# 9. Variable Groups and Secrets

Variable groups are useful for shared configuration such as:

```text
AKS_CLUSTER
RESOURCE_GROUP
ACR_NAME
environment identifiers
```

Do not use pipeline YAML as a password store.

Conceptually:

```text
Normal configuration --> Variable Group
Secrets              --> Key Vault / secret integration
Identity              --> Managed/Federated identity where possible
```

---

# 10. Azure DevOps Environments

Do not confuse an Azure DevOps **Environment** with simply setting:

```text
ENVIRONMENT=prod
```

An Azure DevOps Environment is useful as a deployment/control boundary.

Examples:

```text
DEV
STAGE
PROD
```

It can be associated with:

- deployment history
- approval/check controls
- resource protection
- production governance

This is especially useful because important production checks should not rely only on application-owned YAML.

---

# 11. Deployment Job

Use a deployment-oriented job for deployments rather than treating every deployment as just another generic build job.

Mental model:

```text
Normal Job
    --> build/test/general execution

Deployment Job
    --> deploy release to an Environment
    --> deployment history / environment semantics
```

---

# 12. Artifacts vs Cache

This is a common follow-up question.

## Artifact

Something the pipeline produces and another stage/release requires.

Examples:

- compiled binary
- Helm chart
- manifests
- package
- test report where retained as an output

Think:

> **Artifact = correctness / deliverable**

## Cache

Used mainly to speed up pipeline execution.

Examples:

- Maven `~/.m2`
- npm package cache
- pip cache

Think:

> **Cache = performance optimization**

Do not make production deployment depend on something that exists only in cache.

---

# 13. Parallel Jobs

Independent work can run in parallel.

Example:

```text
               +--> Unit Tests --------+
               |                        |
Commit / PR ---+--> SAST ---------------+--> Build
               |                        |
               +--> Dependency Scan ----+
```

Benefits:

- shorter pipeline duration
- faster developer feedback

But parallelism must be controlled because agents/executors are finite resources.

---

# 14. Conditions

Conditions determine whether stages/jobs execute.

Examples:

```text
PR
 --> lint + tests + scans

main branch
 --> build + deploy DEV

release branch/tag
 --> stage flow

production
 --> only after required gate/check
```

Use conditions to express workflow logic without maintaining completely separate pipeline implementations.

---

# 15. SAST, SCA, Image Scan and DAST

This is the security section to know well enough for interview discussion.

## Simplest memory trick

```text
SAST       = inspect our source code
SCA        = inspect third-party dependencies
Image Scan = inspect the built container image
DAST       = test the running application
```

| Security control | What it examines | Typical pipeline location |
|---|---|---|
| SAST | source code | CI, before image promotion |
| SCA | libraries/dependencies | CI |
| Secret scan | accidental credentials/secrets | CI / PR |
| Image scan | image OS packages/layers | after container build |
| DAST | running web app/API | after deployment to non-prod |

---

# 16. SAST — Static Application Security Testing

SAST analyzes source code without requiring the application to be running.

```text
Source Code
    |
    v
SAST Scanner
    |
    +--> insecure coding patterns
    +--> injection risks
    +--> unsafe API use
    +--> security defects
```

Typical placement:

```text
PR
 ↓
Unit Test
 ↓
SAST
 ↓
Build
```

Possible products in the industry include tools such as Checkmarx, Fortify, Veracode, Snyk Code or security analysis integrated with code-quality platforms.

**Do not claim a particular product unless it was actually used in your environment.**

## Interview positioning

> SAST was integrated into the CI flow to analyze the application source and fail or flag builds according to the organization's vulnerability policy. My focus from the platform side was the pipeline integration and enforcement of the gate rather than administration of the security product itself.

---

# 17. SCA — Software Composition Analysis

SCA examines third-party/open-source dependencies.

Examples:

```text
Maven dependencies
npm packages
Python packages
NuGet packages
```

It helps identify known vulnerabilities in components that our application consumes.

Important distinction:

```text
SAST --> problems in our code
SCA  --> problems in dependencies we consume
```

---

# 18. Container Image Scan

This happens after the container is built.

```text
Docker Build
    |
    v
Container Image
    |
    v
Image Scanner
    |
    +--> OS package vulnerabilities
    +--> vulnerable runtime packages
    +--> vulnerable base image components
```

Example classes of issues:

- vulnerable OpenSSL package
- vulnerable OS libraries
- outdated runtime packages
- CVEs inherited from the base image

Do not confuse image scanning with SAST.

---

# 19. DAST — Dynamic Application Security Testing

DAST requires a running application.

```text
Build
  |
  v
Deploy to DEV / Security Environment
  |
  v
Running Application / API
  |
  v
DAST Scanner
  |
  +--> runtime/web vulnerabilities
  +--> authentication problems
  +--> exposed endpoints
  +--> injection behavior
  +--> security-header issues
```

Typical placement:

```text
Build
 ↓
Image Scan
 ↓
Deploy DEV/Test
 ↓
Smoke Test
 ↓
DAST
 ↓
Promotion
```

Potential products in the industry include OWASP ZAP, Burp Suite Enterprise, Invicti and similar tools.

Again, do not state a specific product unless you know that it was used.

---

# 20. Safe SAST/DAST Interview Answer

Use this answer if asked about your direct exposure:

> Security scanning was integrated as part of the CI/CD pipeline. The security team or security tooling owners handled much of the scanner configuration, so I was not the administrator of the SAST or DAST product itself.
>
> From the pipeline perspective, SAST ran early in CI against the source code. DAST was different because it required a deployed/running application, so it ran against an appropriate non-production endpoint.
>
> Our platform responsibility was to integrate those security stages, provide the required artifact or endpoint, enforce the agreed severity threshold and prevent promotion when a mandatory security gate failed.

This is much stronger than pretending to be a product administrator.

---

# 21. Full Pipeline with Security Placement

```text
Pull Request
     |
     v
Lint
     |
     v
Unit Test
     |
     +--> SAST
     +--> SCA
     +--> Secret Scan
     |
     v
Docker Build
     |
     v
Image Scan
     |
     v
Push ACR
     |
     v
Deploy DEV
     |
     +--> Smoke Test
     +--> Integration Test
     +--> DAST
     |
     v
STAGE
     |
     v
Approval / Policy Gate
     |
     v
PRODUCTION
```

---

# 22. Why Run Security Checks at Different Points?

Because they inspect different things.

```text
Source code -----------> SAST
Dependencies ----------> SCA
Container filesystem --> Image Scan
Running application ---> DAST
```

One scanner does not replace all of the others.

---

# 23. What Happens When a Security Scan Fails?

Do not say that every vulnerability automatically blocks all deployments.

A realistic enterprise answer is:

> The security organization defines the severity policy. For example, Critical/High findings above an accepted threshold can block promotion, while lower-severity findings may be recorded and tracked according to policy. There also needs to be an approved exception process because false positives and accepted risks exist.

Pipeline:

```text
Security Scan
     |
     +--> Pass --> continue
     |
     +--> Policy violation
              |
              v
         block promotion
              |
              +--> remediate
              |
              +--> approved exception if governance allows
```

---

# 24. Promotion Strategy

Three concepts can work together.

## Approach 1 — Build Once and Promote the Same Image

Preferred base principle:

```text
Commit
   |
   v
Image digest abc123
   |
   +--> DEV
   +--> STAGE
   +--> PROD
```

This is the most important rule.

---

## Approach 2 — Helm Values Per Environment

Keep application binary/image unchanged while configuration differs.

```text
values-dev.yaml
values-stage.yaml
values-prod.yaml
```

Typical differences:

- replicas
- CPU/memory
- autoscaling
- ingress configuration
- environment URLs
- feature/config values

The image digest remains the same.

---

## Approach 3 — GitOps

A GitOps-based design could be:

```text
Application Repo
      |
      v
Azure DevOps CI
      |
      +--> Build/Test/Scan
      |
      v
ACR
      |
      v
GitOps Repository
      |
      v
Argo CD
      |
      v
AKS
```

Example GitOps structure:

```text
gitops/
+-- dev/
|   +-- payment-values.yaml
+-- stage/
|   +-- payment-values.yaml
+-- prod/
    +-- payment-values.yaml
```

Promotion becomes a Git change such as:

```diff
-image: sha256:old111
+image: sha256:abc123
```

Then Argo CD reconciles the desired state into the target cluster.

## Strong interview point

With GitOps:

```text
Azure DevOps --> CI / validation / promotion
Argo CD      --> Kubernetes reconciliation/deployment
```

---

# 25. Deployment Strategies

## Rolling Update

Default choice for many stateless microservices.

```text
V1 V1 V1 V1
V2 V1 V1 V1
V2 V2 V1 V1
V2 V2 V2 V1
V2 V2 V2 V2
```

Advantages:

- low additional infrastructure
- native Kubernetes deployment behavior
- minimal downtime when configured correctly

Use readiness probes and appropriate surge/unavailable settings.

---

## Recreate

```text
V1 V1 V1
   |
stop all
   |
V2 V2 V2
```

Use only when versions cannot safely coexist or the application requires it.

Trade-off:

- downtime

---

## Blue/Green

```text
             +--> BLUE  V1
Users --> LB |
             +--> GREEN V2
```

Deploy and validate V2 separately, then switch traffic.

Benefits:

- very fast traffic switch
- fast fallback

Trade-off:

- extra infrastructure/resource cost
- database compatibility still needs careful design

---

## Canary

```text
V1 --> 95%
V2 -->  5%
```

Then progressively increase traffic:

```text
5% --> 20% --> 50% --> 100%
```

Observe:

- 5xx/error rate
- latency
- pod health
- application metrics
- business transaction failures
- logs/traces

Use for higher-risk/high-value services where gradual exposure is beneficial.

---

# 26. Post-deployment Validation

A successful Helm command does not prove the application is healthy.

Validate:

```text
Deployment completed
       |
       v
Pods Ready?
       |
       v
Readiness OK?
       |
       v
Service reachable?
       |
       v
Smoke test?
       |
       v
Dependencies reachable?
       |
       v
Application metrics healthy?
```

Possible checks:

- pod readiness
- service endpoint
- API health
- database connectivity
- dependent services
- error rate
- latency
- key business transaction

---

# 27. Rollback Strategy

Rollback should be tied to measurable failure conditions.

```text
Deploy
  |
  v
Observe
  |
  +--> smoke test failed?
  +--> readiness failed?
  +--> error rate increased?
  +--> latency degraded?
  |
 YES
  |
  v
Rollback
  |
  v
Previous known-good image digest
```

Example:

```text
New:
sha256:abc123

Previous known good:
sha256:def456
```

Immutable artifacts make rollback much safer.

---

# 28. Frequently Asked Interview Q&A

## Q1. Why not maintain one large pipeline for all microservices?

Because it creates tight coupling. A failure/change in one service or pipeline section can affect unrelated services, and independent teams cannot deploy independently.

Use per-service pipelines that consume centrally governed reusable templates.

---

## Q2. Why central templates?

To standardize:

- build conventions
- security stages
- artifact handling
- deployment logic
- production controls

while allowing application teams to supply service-specific parameters.

---

## Q3. What is the difference between an Azure DevOps Agent and GitLab Runner?

They serve essentially the same execution role.

The CI/CD orchestration service schedules a job and the Agent/Runner executes that job on compute.

```text
Azure Agent ~= GitLab Runner
```

---

## Q4. When would you use a self-hosted Azure DevOps Agent?

When I need:

- connectivity to private AKS
- connectivity to internal/private services
- preinstalled/custom tooling
- tighter runtime control
- persistent local caching where appropriate

---

## Q5. Why not rebuild for production?

Because the production artifact would no longer be the exact artifact validated in earlier environments.

Build once and promote the same immutable image digest.

---

## Q6. Why use an image digest instead of just a tag?

A tag can potentially move. A digest identifies the exact image content.

For strong traceability:

```text
repository@sha256:...
```

is preferable for promotion.

---

## Q7. How does configuration change between environments if the image remains the same?

Through external configuration such as:

- Helm values
- ConfigMaps
- Secrets
- environment-specific GitOps overlays
- external configuration services where appropriate

The application image itself remains unchanged.

---

## Q8. SAST vs DAST?

```text
SAST = scans code without running application
DAST = tests the running application/API
```

SAST runs early in CI. DAST runs after deployment to an appropriate test environment.

---

## Q9. SAST vs SCA?

```text
SAST --> analyzes our source code
SCA  --> analyzes third-party dependencies
```

---

## Q10. Image scanning vs SAST?

SAST analyzes the source code.

Image scanning analyzes the built image including operating-system packages and runtime dependencies/layers.

---

## Q11. Where would you put DAST?

After deploying to a non-production environment where the application or APIs are reachable.

Example:

```text
Build --> Deploy DEV/Test --> Smoke --> DAST --> Promote
```

---

## Q12. Which SAST/DAST product did you use?

Do not invent a tool.

Safe answer:

> The scanner itself was owned/configured primarily by the security/tooling team. My responsibility was integrating the scanner into the CI/CD workflow, passing the required source/artifact/endpoint, enforcing the agreed result threshold and ensuring a failed mandatory gate prevented promotion.

If the actual product is remembered later, add the product name.

---

## Q13. What happens when SAST finds a Critical vulnerability?

The exact handling should be policy driven.

For example:

```text
Critical/High beyond policy threshold
        |
        v
Block promotion
        |
        +--> fix
        |
        +--> approved risk exception where governance permits
```

Do not hard-code arbitrary severity policies without organizational agreement.

---

## Q14. Why perform scanning before Docker build?

Cheap checks should fail early.

If unit tests/SAST/SCA already show a blocking issue, there is no reason to spend more compute building and publishing an image.

This is the **fail-fast principle**.

---

## Q15. Why run an image scan if SAST already passed?

Because the container may introduce vulnerabilities from:

- the base image
- OS packages
- language runtime packages

SAST does not provide the same visibility.

---

## Q16. Artifact vs cache?

```text
Artifact = pipeline output required by downstream consumption
Cache    = performance optimization
```

Never depend on a cache as the only copy of a deployable release artifact.

---

## Q17. Why use Azure DevOps Environments?

For deployment traceability and environment-specific governance such as production approvals/checks.

They represent more than a simple environment-name variable.

---

## Q18. Why keep approvals outside application YAML where possible?

Because if all protection exists only in a YAML file controlled by the application repository, changes to that YAML could weaken the deployment control.

Protect the production resource/environment using platform-level checks and access controls.

---

## Q19. How do you authenticate from Azure DevOps to Azure?

Through a Service Connection tied to an Azure identity.

Prefer:

- least privilege
- environment-specific scope
- workload/federated identity where possible

instead of storing long-lived credentials.

---

## Q20. What is your default Kubernetes deployment strategy?

For a normal stateless microservice, rolling update is the default.

For high-risk releases requiring controlled exposure, consider canary.

For rapid traffic switching and strong isolation, consider blue/green.

Recreate is generally reserved for applications that cannot have old and new versions coexist.

---

## Q21. How do you decide whether a canary is successful?

Use measurable technical and business signals:

- error rate
- latency
- availability
- readiness
- application exceptions
- transaction failures
- relevant business KPIs

Do not decide solely because pods are Running.

---

## Q22. How do you roll back?

Promote/redeploy the previous known-good immutable image/chart/configuration and validate recovery.

For GitOps, revert the desired-state change and let the reconciler restore it.

---

## Q23. How would this work with Argo CD?

```text
Azure DevOps
   |
   +--> build
   +--> test
   +--> scan
   +--> push ACR
   +--> update GitOps repository
               |
               v
             Argo CD
               |
               v
              AKS
```

Azure DevOps does not need to run `kubectl` against every cluster if Argo CD owns Kubernetes reconciliation.

---

## Q24. Why is GitOps attractive for many clusters/services?

It provides:

- declarative desired state
- Git-based audit history
- pull-based reconciliation
- consistent promotion model
- drift correction
- separation between CI and cluster reconciliation

---

## Q25. How would 500 microservices share the platform without sharing one pipeline?

Each service has its own pipeline invocation/configuration but consumes centrally versioned templates.

```text
500 service repos
       |
       v
shared pipeline templates
       |
       +--> standard CI
       +--> standard security
       +--> standard publishing
       +--> standard deployment/promotion
```

---

# 29. One-Minute Interview Answer

> For a large microservices estate, I would avoid both a monolithic pipeline and hundreds of independently designed pipelines. Each microservice would own a lightweight pipeline definition, while common CI/CD logic would come from centrally managed templates.
>
> On a pull request we run linting, unit tests, SAST, dependency and secret scanning. After those gates pass, we build the container once, scan the resulting image and push it to ACR. The exact immutable image digest is then promoted through Dev, Stage and Production rather than rebuilding for each environment.
>
> Dev deployments are normally automatic and followed by smoke and integration tests. DAST, where required, runs against the deployed non-production application because it needs a running endpoint. Production is protected using environment-level controls, approvals and least-privilege service connections.
>
> For AKS, rolling deployment is my normal default for stateless services, while canary or blue/green can be used when the release risk justifies more controlled traffic movement. After deployment I validate readiness, application health, error rate and latency, and roll back to the previous known-good artifact if those checks fail.
>
> My stronger hands-on CI/CD background is GitLab, but the architecture maps closely: GitLab Runners correspond to Azure DevOps Agents, CI templates map to Azure YAML templates, and protected deployment concepts map to Azure DevOps Environments and checks.

---

# 30. What NOT to Overclaim

Do not claim:

- deep Azure DevOps administration if you have not done it
- administration/configuration of a specific SAST/DAST product without direct experience
- that DAST runs directly against source code
- that SAST replaces dependency/image scanning
- that every security finding must always block deployment
- that production is rebuilt separately
- that a successful Helm command proves application health

Instead say:

> I understand the architecture and the pipeline integration. My deeper hands-on CI/CD experience is GitLab, and I map those concepts into Azure DevOps. For specialist security scanners, I worked/integrated at the pipeline layer while the security/tooling team owned the scanner configuration.

---

# 31. Five Things to Remember Before the Interview

1. **Azure DevOps Agent ~= GitLab Runner.**
2. **Build once; promote the same immutable image digest.**
3. **SAST = source, SCA = dependencies, image scan = container, DAST = running application.**
4. **Production protection belongs at the environment/resource governance layer, not only inside editable pipeline YAML.**
5. **For many microservices, use per-service pipelines backed by centralized reusable templates.**

---

# 32. Final Whiteboard to Memorize

```text
                         CENTRAL PIPELINE TEMPLATES
                                  |
              +-------------------+-------------------+
              |                   |                   |
          Service A           Service B           Service C
              |                   |                   |
              +-------------------+-------------------+
                                  |
                                  v
                           Pull Request / CI
                                  |
                    +-------------+-------------+
                    |             |             |
                 Unit Test       SAST          SCA
                    |             |             |
                    +-------------+-------------+
                                  |
                                  v
                              Build Image
                                  |
                                  v
                              Image Scan
                                  |
                                  v
                                 ACR
                                  |
                         immutable digest
                                  |
                                  v
                               DEV
                     Smoke / Integration / DAST
                                  |
                                  v
                              STAGE
                                  |
                                  v
                        Approval / Policy
                                  |
                                  v
                              PROD / AKS
                                  |
                      Rolling / Canary / Blue-Green
                                  |
                                  v
                      Post-deploy Health Validation
                          |                 |
                        PASS               FAIL
                          |                 |
                       Complete          Rollback
```
