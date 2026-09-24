# private-llmaas-multitenant

GitOps bootstrap for the multi-tenant LLMaaS workshop on OpenShift, deployed by an
agnosticv catalog item. The catalog item applies `bootstrap/values.yaml` to the
`bootstrap/` Helm chart, which renders an app-of-apps ArgoCD Application set in
`openshift-gitops`; each child Application sources a leaf chart or values under
`deployments/`.

## bootstrap/ (app-of-apps)

| Application | Deploys | Source | Pin |
|---|---|---|---|
| `openshift-ai-operator` | RHOAI operator Subscription (channel `stable-3.5`, CSV `rhods-operator.3.5.1`, **Manual** approval) into `redhat-ods-operator` | `jharmison-redhat/openshift-setup` `charts/install-operators` + values `deployments/openshift-ai-operator/values.yaml` | SHA `f6beed4b481d5345397fdbd8f6749c846181ee1e` |
| `openshift-ai` | DataScienceCluster (v2 components: `aigateway`+`modelsAsAService` Managed, `ogx` Managed, `kserve` w/o MaaS, `trainingoperator` Removed) and DSCI | `jharmison-redhat/openshift-setup` `charts/openshift-ai` + values `deployments/openshift-ai/values.yaml` | same SHA |
| `rhcl-operator` (connectivitylink.yaml) | RHCL operator into `kuadrant-system` (`rhcl-operator.v1.4.3`) | `jharmison-redhat/openshift-setup` `charts/rhcl` | same SHA |
| `nvidia-gpu-enablement` | NVIDIA GPU operator enablement | `jharmison-redhat/openshift-setup` `charts/nvidia-gpu-enablement` | same SHA |
| `models-as-a-service` | MaaS stack from `opendatahub-io/models-as-a-service` `deployment/overlays/openshift` (v0.2.1: Gateway, maas-controller, observability) with kustomize patches for the Gateway listener | upstream repo | commit `8ee0617b3aa3efb0dad62a3a7f8a7f9e809948b0` |
| `models` (wave 1) | `LLMInferenceService` + `MaaSModelRef` per model into `llm` | this repo `deployments/models` | this repo |
| `maas-access` (wave 2) | `MaaSSubscription` (premium tier, 100k tokens/min) + `MaaSAuthPolicy` into `models-as-a-service` | this repo `deployments/maas-access` | this repo |
| `rhoai-extras` (wave 2) | PostSync hook Jobs: enable dashboard observability, bounce Kuadrant operator if unhealthy, set Authorino SSL env vars | this repo `deployments/rhoai-extras` | this repo |
| `workspace` | per-user DevWorkspaces | this repo `deployments/workspace` | this repo |
| `devspaces` | DevSpaces CheCluster + Subscription | this repo `deployments/devspaces` | this repo |
| `grafana` | Grafana instance, dashboards, console link | this repo `deployments/grafana` | this repo |
| `cluster-monitoring` | cluster monitoring ConfigMap | this repo `deployments/cluster-monitoring` | this repo |
| `slack-mcp` (kubernetes-mcp-server.yaml) | Slack MCP server | this repo `deployments/slack-mcp` | this repo |

All own-repo Applications follow `bootstrap.values.repoURL` /
`bootstrap.values.targetRevision`. Supporting resources rendered directly by the
bootstrap chart (GatewayClass, base Gateway, gitops, user groups, MCP config/RBAC,
`redhat-ods-applications` namespace) live in `bootstrap/templates/extra-resources/`.

## deployments/

Leaf Helm charts (Chart.yaml + templates) or plain values files consumed by the
bootstrap chart via `$values` references. Each directory is one child Application's
source path.

## RHOAI 3.5.1 notes

- DSC v2 component set: `aigateway` and `modelsAsAService` Managed (MaaS API is
  operator-managed in `redhat-ai-gateway-infra`), `ogx` Managed replaces the
  3.4-era `llamastackoperator` (per-user `LlamaStackDistribution` CRs are gone;
  Gen AI Studio Playground comes from ogx via the dashboard).
- Operator channel `stable-3.5` with `installPlanApproval: Manual` — the pinned
  `install-operators` chart's approval/wait Jobs drive sync ordering, so other
  Applications must not force the InstallPlan to Automatic.
- MaaS access (subscriptions/auth policies) syncs in wave 2 after `models`
  (wave 1); the `rhoai-extras` PostSync Jobs remediate known 3.5 startup races
  (Kuadrant dependency probe caching, Authorino SSL CA env).
