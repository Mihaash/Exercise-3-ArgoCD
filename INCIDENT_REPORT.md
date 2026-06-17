# ArgoCD OutOfSync Production Incident Report - payment-service

## 1. Investigation: `argocd app diff payment-service`
The diff reveals a mismatch in the `spec.replicas` field:
- **Live State:** `replicas: 5`
- **Git State:** `replicas: 3`

## 2. Findings

### What changed?
The live cluster state drifted from the Git configuration. Specifically, the deployment was scaled up to 5 replicas in the cluster, whereas the configuration in the Git repository specifies 3 replicas.

### Who changed it?
There are two primary possibilities:
1. **Manual Intervention:** A developer or operator manually executed a command like `kubectl scale deployment payment-service --replicas=5`.
2. **Horizontal Pod Autoscaler (HPA):** If an HPA is configured for this service, it may have automatically scaled the pods to 5 to handle increased traffic.

### How to prevent recurrence?
To ensure the environment remains in sync and prevent unauthorized or untracked changes:
1. **Enable/Verify Self-Healing:** Ensure ArgoCD's `selfHeal` policy is enabled. This will cause ArgoCD to automatically revert any manual changes in the cluster back to the state defined in Git.
2. **Handle HPA Correctly:** If the scaling was intended (via HPA), the `spec.replicas` field should be removed from the Deployment manifest or added to ArgoCD's `ignoreDifferences` configuration. This prevents ArgoCD from trying to "fix" the replica count while the HPA is managing it.
3. **Restrict RBAC:** Tighten Kubernetes RBAC permissions to prevent manual scaling or modifications of production deployments by individual users.
4. **Enforce GitOps Workflow:** Educate the team to perform all changes (including scaling) via Git commits rather than direct cluster access.
