# Terraform — Consolidated Interview Notes & Q&A

> Focus: Senior DevOps / Platform / Cloud interviews  
> Context: Terraform / Terraform Enterprise (TFE) / HCP Terraform with Azure

---

## 1. Terraform Mental Model

Terraform should be understood as a reconciliation system between:

```text
Terraform Configuration (Desired State)
              |
              v
        Terraform Plan
          /         \
         /           \
Terraform State   Actual Infrastructure
                  (Azure / AWS / etc.)
```

Terraform configuration defines what we **want**.

Terraform state stores Terraform's mapping between resource addresses in code and real infrastructure objects.

During a normal plan, Terraform queries the provider and compares the actual infrastructure with the configuration/state to determine what must change.

### Easy rule to remember

```text
Code      = Desired state
State     = Terraform's mapping/known information
Cloud     = Actual state
Plan      = Difference / reconciliation decision
Apply     = Execute the approved change
```

---

# 2. Core Terraform Concepts

## Provider

A provider is the plugin Terraform uses to communicate with an external API.

Examples:

- AzureRM -> Azure
- AWS -> AWS
- Kubernetes -> Kubernetes API
- GitHub -> GitHub API

Example:

```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 4.0"
    }
  }
}

provider "azurerm" {
  features {}
}
```

Concept:

```text
Terraform
   |
AzureRM Provider
   |
Azure APIs
```

---

## Resource

A resource means:

> Terraform should create/manage this object.

Example:

```hcl
resource "azurerm_resource_group" "aks" {
  name     = "rg-aks-prod"
  location = "Central India"
}
```

---

## Data Source

A data source means:

> The object already exists; Terraform only needs to read information from it.

Example:

```hcl
data "azurerm_virtual_network" "existing" {
  name                = "vnet-prod"
  resource_group_name = "rg-network-prod"
}
```

Then:

```hcl
vnet_id = data.azurerm_virtual_network.existing.id
```

Typical enterprise use:

```text
Network team creates VNet
        |
Platform Terraform uses data source
        |
Creates AKS in existing network
```

---

## Variable

Input passed into Terraform or a module.

```hcl
variable "environment" {
  type = string
}
```

Examples:

- dev
- stage
- prod
- region
- node count
- subnet ID
- VM size

---

## Locals

Locals are internal calculated/reusable values.

```hcl
locals {
  cluster_name = "aks-${var.environment}"
}
```

Use them to avoid repeating expressions.

---

## Output

Outputs expose useful values.

```hcl
output "cluster_id" {
  value = azurerm_kubernetes_cluster.main.id
}
```

Typical uses:

- pass values to other automation
- expose IDs
- module outputs
- troubleshooting

---

# 3. Modules

A module is a reusable set of Terraform resources.

Example structure:

```text
modules/
└── aks/
    ├── main.tf
    ├── variables.tf
    └── outputs.tf

environments/
├── dev/
├── stage/
└── prod/
```

All environments can call the same module:

```text
Dev   ----\
Stage ----- > AKS Module
Prod  ----/
```

But provide different inputs such as:

- subscription
- VNet/subnet
- node count
- VM size
- region
- tags
- autoscaling
- DNS

### Interview answer

> I build reusable modules for common infrastructure and keep environment-specific configuration outside the module. Dev, stage and production use the same tested module but supply different inputs and have isolated state.

---

# 4. Terraform State

Terraform state maps Terraform resource addresses to real infrastructure resources.

Example:

```text
Terraform code:

azurerm_resource_group.prod

          |
          | state mapping
          v

Azure:

/subscriptions/.../resourceGroups/rg-prod
```

State is important because Terraform needs to know which real object corresponds to which Terraform resource.

### State may contain sensitive information

Therefore:

- Do not store tfstate in Git.
- Use a remote backend.
- Restrict access with RBAC.
- Enable storage protection/versioning as appropriate.
- Treat state as sensitive data.

---

# 5. Remote Backend — Azure

Typical Azure backend:

```text
Terraform / TFE
      |
      v
Azure Storage Account
      |
Blob Container
      |
*.tfstate
```

Example:

```hcl
terraform {
  backend "azurerm" {
    storage_account_name = "sttfstateprod"
    container_name       = "tfstate"
    key                  = "aks/prod.tfstate"
    use_azuread_auth     = true
  }
}
```

For enterprise environments use:

- Azure Storage / HCP Terraform / TFE remote state
- Microsoft Entra ID / workload identity where possible
- least-privilege RBAC
- protected storage
- environment-separated state

---

# 6. State Locking

State locking prevents two engineers or pipelines from modifying the same state concurrently.

```text
Engineer A ----                -> Shared Remote State
Engineer B ----/
```

With locking:

```text
Engineer A
   |
Acquire Lock
   |
Plan/Apply
   |
Update State
   |
Release Lock

Engineer B
   |
Cannot acquire lock
   |
Wait / Fail
```

### Interview answer

> In a shared Terraform environment, remote state must support locking. If two engineers apply simultaneously against the same state, the first run gets the lock and the second cannot modify the state until that lock is released.

Avoid disabling locking unless handling a very specific recovery scenario.

---

# 7. Plan / Apply / Destroy

## terraform plan

Calculates the difference between the desired configuration and actual infrastructure.

Typical indicators:

```text
+    create
~    update
-    destroy
-/+  replace
```

Enterprise flow often saves the plan:

```bash
terraform plan -out=tfplan
terraform apply tfplan
```

This ensures the reviewed plan is the one being applied.

---

## terraform apply

Executes the proposed changes.

---

## terraform destroy

Destroys resources managed by the configuration.

Production environments should protect destructive actions with:

- approvals
- policy checks
- restricted permissions
- review

---

# 8. Drift

Drift means the actual infrastructure no longer matches the Terraform desired configuration.

Example:

```text
Terraform Code = VM size D4
Terraform State = VM size D4
Azure           = VM size D8
```

Someone manually changed Azure.

During the next normal plan, Terraform reads the actual resource and detects the difference.

Terraform generally plans to return the infrastructure to what is defined in code.

```text
Actual Azure = D8
Terraform desired = D4

Plan:
D8 -> D4
```

If the manual change should become permanent:

1. Update Terraform code to D8.
2. Run plan.
3. Review.
4. Apply.

Do not use manual state editing as the normal solution.

---

# 9. Drift Scenarios — Very Important

## Scenario A — Resource manually deleted, but still exists in Terraform code

Before:

```text
Code  = Resource exists
State = Resource exists
Azure = Resource exists
```

Someone manually deletes it:

```text
Code  = Resource exists
State = Resource recorded
Azure = MISSING
```

Next plan:

Terraform detects that the actual resource is missing.

Because the resource is still required by the configuration:

```text
Plan = CREATE
```

After approved apply:

```text
Terraform recreates the resource.
```

### Interview answer

> If a Terraform-managed resource is manually deleted but remains in the configuration, Terraform detects that it no longer exists during the next refresh/plan and proposes to recreate it. Once approved and applied, Terraform restores the desired state.

---

## Scenario B — VM manually resized/scaled

Terraform code:

```text
VM Size = D4
```

Someone manually changes Azure:

```text
Azure VM Size = D8
```

Now:

```text
Code  = D4
State = previously D4
Azure = D8
```

Next plan detects drift and usually proposes:

```text
D8 -> D4
```

If applied, Terraform restores the value defined by code.

### If the D8 change is intentional

Update code:

```hcl
vm_size = "Standard_D8s_v5"
```

Then plan/apply.

---

## Scenario C — Resource removed from Terraform code but still exists in Azure

```text
Code  = Resource removed
State = Resource exists
Azure = Resource exists
```

Terraform knows it previously managed that resource, but it is no longer declared.

Normal plan:

```text
Plan = DESTROY
```

After apply, Terraform destroys the cloud resource and removes its state association.

---

## Scenario D — Resource manually deleted from Azure AND removed from Terraform code

This was our follow-up scenario.

```text
Code  = Resource removed
State = Resource still recorded
Azure = Resource already deleted manually
```

During the next normal run Terraform discovers that:

- the resource no longer exists in Azure
- the configuration no longer wants it

Therefore:

```text
No resource needs to be recreated.
No real resource remains to be destroyed.
State is reconciled/cleaned up.
```

### Easy comparison

```text
Deleted from Azure only
        |
        v
Terraform RECREATES

Deleted from code only
        |
        v
Terraform DESTROYS

Deleted from Azure + code
        |
        v
Nothing to recreate/destroy
Terraform reconciles state
```

---

# 10. Intentional External Changes / Autoscaling

Sometimes another controller intentionally owns an attribute.

Example:

Terraform declares:

```text
VMSS instances = 3
```

Autoscaler changes:

```text
3 -> 7
```

If Terraform still owns the attribute, the next apply may try:

```text
7 -> 3
```

For attributes intentionally controlled by another system:

```hcl
lifecycle {
  ignore_changes = [
    instances
  ]
}
```

### Important

Do not use `ignore_changes` merely to hide drift.

Use it only where ownership is deliberately shared or delegated to another controller.

---

# 11. Lifecycle

Important lifecycle controls:

## prevent_destroy

```hcl
lifecycle {
  prevent_destroy = true
}
```

Prevents Terraform from planning destruction through Terraform.

Important:

> It does NOT stop a user from manually deleting the resource directly in Azure.

---

## create_before_destroy

```hcl
lifecycle {
  create_before_destroy = true
}
```

When replacement is required, create the new resource before deleting the old one where supported.

---

## ignore_changes

```hcl
lifecycle {
  ignore_changes = [
    tags
  ]
}
```

Terraform does not reconcile those specified attribute changes.

---

# 12. depends_on

Terraform normally detects dependencies automatically through references.

Example:

```hcl
subnet_id = azurerm_subnet.aks.id
```

Terraform understands:

```text
Subnet
  |
  v
AKS
```

Use explicit `depends_on` only for dependencies Terraform cannot infer.

```hcl
depends_on = [
  azurerm_role_assignment.aks_network
]
```

### Interview answer

> I prefer implicit dependencies through resource references because Terraform can build the dependency graph automatically. I use depends_on only for hidden dependencies that are not represented through an attribute reference.

---

# 13. count vs for_each

## count

Good when instances are nearly identical.

```hcl
resource "azurerm_resource_group" "example" {
  count = 3

  name = "rg-${count.index}"
}
```

Addresses:

```text
resource[0]
resource[1]
resource[2]
```

---

## for_each

Better when each resource has a stable logical identity.

```hcl
for_each = {
  dev   = "centralindia"
  stage = "centralindia"
  prod  = "eastus"
}
```

Addresses:

```text
resource["dev"]
resource["stage"]
resource["prod"]
```

### Interview answer

> I use count for almost identical repeated instances and for_each when objects have meaningful stable keys. for_each is often safer for independently identified resources because removing one key does not renumber every later object.

---

# 14. Dynamic Blocks

Dynamic blocks generate repeated nested blocks from a collection.

Example:

```hcl
dynamic "security_rule" {
  for_each = var.rules

  content {
    name = security_rule.value.name
  }
}
```

Concept:

```text
Collection
   |
for_each
   |
Generate repeated nested blocks
```

Use dynamic blocks for repeated nested configuration, not as a replacement for normal resource-level for_each.

---

# 15. Import

Scenario:

```text
Resource exists in Azure
       |
Terraform has no state mapping
```

Bring the resource under Terraform management.

Modern configuration-driven approach:

```hcl
import {
  to = azurerm_resource_group.example
  id = "/subscriptions/.../resourceGroups/rg-existing"
}
```

Traditional CLI:

```bash
terraform import azurerm_resource_group.example <resource-id>
```

Then:

1. Run plan.
2. Compare Terraform configuration with real resource.
3. Adjust configuration.
4. Reach the desired clean/expected plan.

---

# 16. Resource Exists but Not in State

Do NOT simply apply blindly.

Use import:

```text
Existing Azure Resource
       |
terraform import / import block
       |
Terraform State
       |
terraform plan
       |
Reconcile code with resource
```

---

# 17. State Lost or Corrupted

First action:

```text
STOP
 |
Do not blindly apply
```

Recovery flow:

```text
Check remote backend
      |
Check state version / backup
      |
Restore known-good state
      |
terraform plan
      |
Validate against Azure
```

If no usable state exists:

```text
Terraform configuration
      +
Existing Azure infrastructure
      |
Import resources
      |
Plan/reconcile
```

Avoid hand-editing or force-pushing state except for controlled recovery situations.

---

# 18. Refactoring Without Recreating Resources

Suppose:

```hcl
resource "azurerm_resource_group" "old" {}
```

becomes:

```hcl
resource "azurerm_resource_group" "prod" {}
```

Without state-aware refactoring Terraform may interpret that as:

```text
Destroy old address
Create new address
```

Use a moved block:

```hcl
moved {
  from = azurerm_resource_group.old
  to   = azurerm_resource_group.prod
}
```

Older/manual state operation:

```bash
terraform state mv ...
```

For maintained Terraform code, prefer declarative moved blocks where practical.

---

# 19. Provider Upgrade

Example:

```hcl
required_providers {
  azurerm = {
    source  = "hashicorp/azurerm"
    version = "~> 4.0"
  }
}
```

Upgrade process:

```text
Read provider release notes
        |
Update provider constraint
        |
terraform init -upgrade
        |
Review .terraform.lock.hcl
        |
terraform validate
        |
terraform plan
        |
Test DEV
        |
STAGE
        |
PROD
```

Do not blindly upgrade a production provider.

Check for:

- breaking changes
- deprecated arguments
- changed defaults
- force-replacement behavior
- state/schema migrations

---

# 20. Failed Partial Apply

Example:

```text
Resource Group   ✓
VNet             ✓
Subnet           ✓
AKS              X
```

Terraform does not provide a transactional rollback of every resource.

Successfully completed resources are normally recorded in state.

Recovery:

```text
Investigate failure
      |
Verify state and cloud
      |
Fix root cause
      |
terraform plan
      |
Review remaining changes
      |
terraform apply
```

Do not automatically delete everything manually.

---

# 21. Workspaces — Two Different Meanings

This distinction is important.

## Terraform CLI workspaces

Example:

```bash
terraform workspace new dev
terraform workspace new stage
terraform workspace new prod
```

Concept:

```text
Same Terraform configuration
        |
        +-- dev state
        +-- stage state
        +-- prod state
```

CLI workspaces primarily provide separate state instances for the same configuration.

For strongly isolated enterprise environments, separate root configurations/backends/workspaces/projects/accounts/subscriptions may be preferable depending on the platform design.

---

## TFE / HCP Terraform Workspaces

An HCP Terraform / Terraform Enterprise workspace is much more than a CLI state workspace.

Conceptually:

```text
TFE / HCP Organization
        |
        +-- Project: Platform
                |
                +-- aks-dev
                +-- aks-stage
                +-- aks-prod
```

A managed workspace can provide:

- state
- variables
- run history
- VCS integration
- execution
- permissions
- approvals
- policy checks
- auditability

---

# 22. How to Allow a Team to Create TFE/HCP Workspaces

Enterprise design:

```text
Organization
    |
Project
    |
Team
    |
Project-level permissions
    |
Create/manage workspaces
```

Use least privilege.

Do not give users full organization-owner access merely so they can create workspaces.

### Interview answer

> I would place the team's workspaces under an appropriate TFE/HCP Terraform project and assign the team project-level permissions that allow workspace creation and management. I would use a built-in role such as Maintain where suitable or a custom role with only the required workspace permissions. Production workspace permissions would normally be stricter than development.

---

# 23. Applying Policies to Workspaces

Policy examples:

- approved Azure regions
- mandatory tags
- deny public IPs
- restrict oversized VM SKUs
- enforce encryption
- restrict specific resource types
- prevent risky destructive changes
- enforce networking/security standards

Concept:

```text
Policy Repository
       |
Policy Set
       |
Project / Workspace scope
       |
Terraform Plan
       |
Policy Evaluation
       |
   +---+---+
   |       |
 PASS     FAIL
   |       |
Apply    Block / controlled override
```

### Interview answer

> Workspace permissions control who can create/run/manage a workspace. Policy-as-code controls what infrastructure those workspaces are allowed to create. I would attach policy sets at the appropriate project/workspace scope so policy evaluation happens as part of the Terraform run before production apply.

---

# 24. Environment Separation

Typical design:

```text
Reusable Modules
      |
      +----------------+
      |       |        |
     DEV    STAGE     PROD
      |       |        |
   State   State      State
```

Important enterprise principles:

- isolated state
- environment-specific variables
- least-privilege identities
- separate approvals
- tighter production RBAC
- reusable versioned modules
- avoid one giant state containing unrelated infrastructure

---

# 25. Secrets

Never hard-code secrets in:

- `main.tf`
- committed `terraform.tfvars`
- Git
- pipeline YAML
- plain workspace variables

Prefer:

```text
CI/TFE
  |
OIDC / Workload Identity
  |
Azure Entra ID
  |
Azure
```

For application/platform secrets use appropriate systems such as:

- Azure Key Vault
- Vault
- secret-management integrations

### Important interview distinction

`sensitive = true` mainly controls display/redaction.

Do not assume that marking a value sensitive automatically means it cannot exist in state.

Modern Terraform also supports mechanisms such as ephemeral values/resources and provider write-only arguments where supported.

---

# 26. Enterprise Terraform Pipeline

Recommended explanation:

```text
Developer
    |
Git Branch
    |
Pull Request
    |
    +--> terraform fmt -check
    |
    +--> terraform validate
    |
    +--> lint
    |
    +--> IaC security scan
    |
    v
terraform plan
    |
Plan Review
    |
Policy / Governance Check
    |
Approval
    |
Controlled Apply
    |
Azure
```

Supporting controls:

```text
                Remote State
                   |
              State Locking
                   |
Terraform CI/TFE --+-- RBAC
                   |
              Audit Trail
                   |
          OIDC / Workload Identity
```

### Strong enterprise interview answer

> In a multi-engineer environment, infrastructure changes should go through Git PRs. CI runs terraform fmt, validate, lint/security checks and a plan. The plan is reviewed and evaluated against governance policies before an authorized pipeline performs the apply. State is remote and locked, identities use least-privilege RBAC, production applies are controlled, and the complete run history provides an audit trail.

---

# 27. Interview Q&A Bank

## Q1. What is the difference between a resource and a data source?

**Answer:**

A resource is managed by Terraform and can be created, updated or destroyed. A data source reads information about an existing object without making Terraform responsible for creating that object.

---

## Q2. Why does Terraform need state?

**Answer:**

Terraform state maps resource addresses in the Terraform configuration to real infrastructure objects and stores metadata Terraform needs to calculate changes. It allows Terraform to determine what it already manages and what must be created, changed or removed.

---

## Q3. Why use remote state?

**Answer:**

For team environments, local state is unsafe and hard to coordinate. Remote state provides a shared authoritative state location, access control, locking support, backup/versioning options and better operational control.

---

## Q4. What happens if two engineers run apply simultaneously?

**Answer:**

When the backend supports state locking, the first operation acquires the lock and the second operation cannot modify the same state until the lock is released. This prevents concurrent state modification and potential corruption.

---

## Q5. Someone manually deletes a Terraform-managed resource. What happens?

**Answer:**

If the resource remains in Terraform configuration, the next normal plan detects that the actual resource is missing and plans to recreate it. Applying the plan restores the desired state.

---

## Q6. Someone manually changes a VM from D4 to D8, but Terraform says D4. What happens?

**Answer:**

Terraform detects the drift during the next plan. Because the desired configuration still says D4, Terraform normally plans to change D8 back to D4. If D8 is the new intended configuration, I update the Terraform code first and then plan/apply.

---

## Q7. What if the resource is removed from code but still exists in Azure?

**Answer:**

Because the resource is still tracked in state but no longer exists in configuration, Terraform normally proposes to destroy the Azure resource. After apply, the resource and its state association are removed.

---

## Q8. What if the resource was manually deleted in Azure AND removed from Terraform code, but still exists in state?

**Answer:**

Terraform discovers during the run that the actual resource no longer exists and also sees that the configuration no longer wants it. There is nothing to recreate and nothing in Azure to destroy. The stale state association is reconciled/removed through the normal run.

---

## Q9. What if autoscaling changes VMSS capacity?

**Answer:**

If Terraform owns the capacity attribute, a later apply may try to restore the value declared in Terraform. When another controller intentionally owns that attribute, I design that ownership explicitly and may use lifecycle ignore_changes for that particular attribute.

---

## Q10. Does prevent_destroy stop somebody deleting the resource from Azure Portal?

**Answer:**

No. prevent_destroy protects against Terraform planning the destruction of that resource. It cannot prevent an Azure user with sufficient permissions from manually deleting the resource. Azure RBAC, resource locks and governance controls must handle that risk.

---

## Q11. A resource exists in Azure but is not in Terraform state. What do you do?

**Answer:**

I import it into the appropriate Terraform resource address, then run plan and reconcile the Terraform configuration with the actual resource. I do not create a duplicate resource or blindly modify state.

---

## Q12. Terraform state is lost. What do you do?

**Answer:**

First stop automated applies. Check the remote backend's state history/versioning/backup and restore a known-good state where possible. Then run plan and validate it against the actual infrastructure. If state cannot be recovered, rebuild the mappings carefully by importing existing resources.

---

## Q13. Why should state not be committed to Git?

**Answer:**

State can contain sensitive values and infrastructure details, and multiple engineers editing local copies can cause conflicts or corruption. Enterprise Terraform should use a protected remote backend with appropriate RBAC and locking.

---

## Q14. How do you use one module for dev, stage and prod?

**Answer:**

I keep infrastructure logic in reusable modules and have separate environment/root configurations supply environment-specific inputs. Each important environment uses isolated state, and production usually has stricter permissions and approval policies.

---

## Q15. count or for_each?

**Answer:**

I use count for homogeneous repeated instances and for_each when resources have stable logical identities. Stable keys generally make for_each easier to manage when individual instances are added or removed.

---

## Q16. When do you use depends_on?

**Answer:**

Terraform already discovers dependencies from references. I use depends_on only when there is a real dependency Terraform cannot infer from resource attributes.

---

## Q17. When would you use ignore_changes?

**Answer:**

Only when an attribute is intentionally managed outside Terraform, for example autoscaler-controlled capacity or another agreed external controller. I do not use it as a shortcut to hide unexplained drift.

---

## Q18. How do you refactor Terraform without recreating resources?

**Answer:**

If only the Terraform resource/module address changes, I use a moved block so Terraform can preserve the existing resource association. For controlled state operations there is also terraform state mv, but declarative moved blocks are preferable for maintainable refactoring where practical.

---

## Q19. How do you handle a provider upgrade?

**Answer:**

Review release notes and breaking changes, update the version constraint, run terraform init -upgrade, review the dependency lock file, validate and plan, then promote through dev, stage and production rather than upgrading production blindly.

---

## Q20. What happens if terraform apply fails halfway?

**Answer:**

Terraform is not a full transaction that automatically rolls everything back. Successfully completed resources are normally recorded in state. I investigate the failure, confirm state and actual infrastructure, fix the root cause, rerun plan, review the remaining actions and then apply again.

---

## Q21. What is the difference between CLI workspaces and TFE/HCP workspaces?

**Answer:**

Terraform CLI workspaces primarily provide multiple state instances for the same configuration. TFE/HCP Terraform workspaces are managed execution boundaries that can also include state, variables, VCS integration, run history, permissions, approvals and policy evaluation.

---

## Q22. How would you allow a team to create TFE workspaces?

**Answer:**

I would put the workspaces under the appropriate project and give the team project-level permissions that include workspace creation/management, using least privilege. I would not grant organization-owner rights just to let engineers create workspaces.

---

## Q23. How would you apply governance policies to newly created workspaces?

**Answer:**

I would define policy-as-code in policy sets and scope those policy sets appropriately to projects/workspaces. That allows every relevant plan to be evaluated against enterprise guardrails before apply.

---

## Q24. How do you pass secrets to Terraform?

**Answer:**

I avoid committing secrets to Terraform files or Git. For Terraform's Azure authentication I prefer OIDC/workload identity or managed identity instead of long-lived client secrets. Application secrets should be obtained from systems such as Azure Key Vault or Vault, and access to sensitive state and variables must be tightly controlled.

---

## Q25. What is drift?

**Answer:**

Drift is a difference between Terraform's desired configuration and the actual infrastructure, usually caused by out-of-band changes. A normal Terraform plan refreshes the current provider data and shows how Terraform intends to reconcile the difference.

---

# 28. Four Drift Cases to Memorize

This is the fastest revision table:

| Terraform Code | Actual Azure | Typical Terraform Result |
|---|---|---|
| Resource exists | Manually deleted | **Recreate** |
| Resource removed | Resource still exists | **Destroy** |
| Resource removed | Already manually deleted | **Reconcile state; no cloud action needed** |
| Attribute = D4 | Manually changed to D8 | **Change back to D4 unless code is updated** |

---

# 29. Final Interview Summary

If asked to summarize your Terraform approach:

> I treat Terraform configuration as the desired state and keep shared infrastructure state in a secure remote backend with locking and least-privilege access. Changes go through Git pull requests, validation, security checks, plan review, policy enforcement and controlled apply. I use reusable modules with isolated environment state, avoid manual cloud changes, detect drift through plan, and resolve intentional drift by updating code. For Terraform Enterprise/HCP Terraform I use projects, workspaces, team RBAC, policy sets, controlled production permissions and an auditable run history.

---

# 30. Commands Worth Remembering

```bash
terraform init
terraform init -upgrade

terraform fmt
terraform fmt -check

terraform validate

terraform plan
terraform plan -out=tfplan
terraform plan -refresh-only

terraform apply
terraform apply tfplan

terraform destroy

terraform import <address> <resource-id>

terraform state list
terraform state show <address>
terraform state mv <source> <destination>
terraform state pull

terraform workspace list
terraform workspace new dev
terraform workspace select dev
```

Use state-changing commands carefully in shared/production environments.
