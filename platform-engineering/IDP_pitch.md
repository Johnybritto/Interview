I have worked as part of an Internal Developer Platform initiative, where multiple platform capabilities came together to provide self-service to application teams.

My primary involvement was around Kubernetes namespace onboarding and GitOps-based provisioning.

The flow started with the developer entering the required application and environment details through a self-service portal. Those inputs were validated against predefined platform guardrails, such as allowed cluster, namespace conventions, resource limits, RBAC requirements, and other mandatory controls.

Once the request passed validation, the automation created or updated the required GitLab repository and generated the necessary Kubernetes configuration files.

Git was treated as the source of truth. ArgoCD monitored the repository and synchronized the desired configuration to the appropriate Kubernetes cluster.

From the developer's point of view, they did not need to manually understand or create all the underlying Kubernetes objects. They simply provided the required information through the portal, while the platform automated the provisioning in a standardized and controlled way.

I was mainly involved in the Kubernetes and GitOps side of this workflow, particularly around namespace provisioning, required Kubernetes resources, configuration generation, and making sure the GitLab-to-ArgoCD-to-cluster flow worked correctly.

I see an IDP as a broader platform rather than a single tool. It typically includes areas such as the developer portal, automation, Git, CI/CD, Kubernetes, GitOps, security, RBAC, networking, secrets, and observability. In an enterprise environment, these capabilities are usually built and operated by multiple platform engineers or teams, with different people owning different parts of the overall developer experience.
