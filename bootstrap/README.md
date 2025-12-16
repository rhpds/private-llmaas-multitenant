# Bootstrap Helm Chart

This Helm chart creates the **bootstrap Application** for the app-of-apps pattern in ArgoCD.

## Architecture

The bootstrap chart creates a single ArgoCD Application that points to the `applications/` Helm chart, which contains all child applications.

```
bootstrap Application (parent)
  └── manages applications/ Helm chart
      ├── kubernetes-mcp-server
      ├── llama-stack-operator
      ├── llama-stack-shared
      ├── slack-mcp
      ├── nvidia-gpu-enablement
      ├── openshift-ai
      ├── llama-stack-instance (ApplicationSet)
      │   ├── llama-stack-instance-user1
      │   └── llama-stack-instance-user2
      └── workspace (ApplicationSet)
          ├── workspace-user1
          └── workspace-user2
```

## Structure

- **bootstrap/** - Creates the bootstrap Application (app-of-apps parent)
- **applications/** - Helm chart containing all child Applications and ApplicationSets

## Installation

### Initial Setup (One-Time)

Install the bootstrap chart using Helm:

```bash
helm install bootstrap ./bootstrap -n openshift-gitops
```

Or with custom values:

```bash
helm install bootstrap ./bootstrap -n openshift-gitops -f custom-values.yaml
```

### Verify Installation

1. **Check the bootstrap Application is created**:
   ```bash
   kubectl get application bootstrap -n openshift-gitops
   ```

2. **Watch ArgoCD create all child applications**:
   ```bash
   kubectl get applications -n openshift-gitops -w
   ```

3. **View in ArgoCD UI**:
   - Navigate to your ArgoCD dashboard
   - You should see the bootstrap Application with all child apps

### How It Works

1. Helm installs the chart and creates:
   - The bootstrap Application (app-of-apps parent)
   - All other Applications and ApplicationSets as children
2. The bootstrap Application points to this `bootstrap` Helm chart in Git
3. ArgoCD takes over and manages the bootstrap Application going forward
4. All changes to the chart in Git are automatically synced by ArgoCD
5. The `helm.sh/resource-policy: keep` annotation ensures the bootstrap Application persists even if Helm is uninstalled

## Configuration

### bootstrap/values.yaml

Contains only the bootstrap Application configuration:
- `bootstrap.repoURL`: Git repository URL
- `bootstrap.targetRevision`: Branch or tag to use
- `bootstrap.path`: Path to applications chart (should be `applications`)

### applications/values.yaml

Contains all child application configurations:
- `user`: User count and prefix for multi-tenancy
- `workspace`: DevWorkspace configuration
- `slackMcpSecret`: Slack MCP credentials
- `llamaStackInstance`: Per-user Llama Stack configuration
- `kubernetesMcpServer`: Kubernetes MCP server config
- `llamaStackOperator`: Llama Stack operator config
- `llamaStackShared`: Shared Llama Stack resources
- `slackMcp`: Slack MCP server config
- `nvidiaGpuEnablement`: NVIDIA GPU configuration
- `openshiftAi`: OpenShift AI operator config

## Making Changes

After the initial Helm installation, all subsequent changes should be made via Git:

1. **Update values.yaml** with your desired configuration
2. **Commit and push** to your Git repository
3. **ArgoCD automatically syncs** the changes to your cluster
4. **Verify** the changes in ArgoCD UI or CLI

The bootstrap Application will automatically detect and apply changes from Git, maintaining true GitOps workflow.

## Uninstalling

To remove the bootstrap and all child applications:

```bash
# Delete the bootstrap Application (this will cascade delete all child apps)
kubectl delete application bootstrap -n openshift-gitops

# If you used Helm install, you can also run:
helm uninstall bootstrap -n openshift-gitops
```

**Note**: The bootstrap Application has `helm.sh/resource-policy: keep`, so it will persist even after `helm uninstall`. You should delete it via kubectl or ArgoCD.
