# Applications Helm Chart

This Helm chart contains all child applications managed by the bootstrap app-of-apps pattern.

## Overview

This chart is **automatically managed by ArgoCD** through the bootstrap Application. You should not install this chart directly with Helm.

## Contents

This chart contains the following ArgoCD Applications and ApplicationSets:

### Applications (Static)
- **kubernetes-mcp-server** - Kubernetes MCP server deployment
- **llama-stack-operator** - Llama Stack Kubernetes operator
- **llama-stack-shared** - Shared Llama Stack resources
- **slack-mcp** - Slack MCP server deployment
- **nvidia-gpu-enablement** - NVIDIA GPU operator and configuration
- **openshift-ai** - OpenShift AI operator

### ApplicationSets (Dynamic)
- **llama-stack-instance** - Creates per-user Llama Stack instances
- **workspace** - Creates per-user DevWorkspace environments

### Secrets
- **mcp-secret** - Slack MCP and model API credentials

## Configuration

All configuration is managed through `applications/values.yaml`. See the main bootstrap README for details on each section.

## Making Changes

To modify any child application configuration:

1. Edit `applications/values.yaml` or the templates in `applications/templates/`
2. Commit and push to Git
3. The bootstrap Application will automatically sync the changes via ArgoCD

**Do not modify applications directly in ArgoCD** - all changes should be made via Git to maintain GitOps workflow.

## Structure

```
applications/
├── Chart.yaml                       # Helm chart metadata
├── values.yaml                      # All application configurations
├── templates/
│   ├── application-kubernetes-mcp-server.yaml
│   ├── application-llama-stack-operator.yaml
│   ├── application-llama-stack-shared.yaml
│   ├── application-slack-mcp.yaml
│   ├── applicationset-llama-stack-instance.yaml
│   ├── applicationset-workspace.yaml
│   ├── mcp-secret.yaml
│   ├── nvidia-gpu-enablement.yaml
│   └── openshift-ai.yaml
└── README.md
```
