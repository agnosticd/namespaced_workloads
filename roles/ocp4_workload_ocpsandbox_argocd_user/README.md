# ocp4_workload_ocpsandbox_argocd_user

User provisioning role for OcpSandbox clusters with ArgoCD (OpenShift GitOps).

## When to Use

Use this role in any OcpSandbox catalog item where students need their own ArgoCD AppProject for GitOps deployments. Typically runs after `ocp4_workload_ocpsandbox_keycloak_user` (which sets up cluster connectivity).

## Why It Exists

OpenShift GitOps is deployed at the cluster level with no per-user access. This role creates an AppProject scoped to the student's namespaces, allowing ArgoCD ApplicationSets to deploy resources only into namespaces ending with `-{username}`.

## How to Use

### In AgV catalog (`common.yaml`)

```yaml
# Provision order
workloads:
- agnosticd.namespaced_workloads.ocp4_workload_ocpsandbox_keycloak_user
- agnosticd.namespaced_workloads.ocp4_workload_ocpsandbox_argocd_user
# ... other workloads that create ApplicationSets in this project

# Destroy order (reverse) -- argocd_user cleans up ApplicationSets + Applications + AppProject
remove_workloads:
# ... other workloads
- agnosticd.namespaced_workloads.ocp4_workload_ocpsandbox_argocd_user
- agnosticd.namespaced_workloads.ocp4_workload_ocpsandbox_keycloak_user

# Variables
ocp4_workload_ocpsandbox_argocd_user_username: "user1-{{ guid }}"
ocp4_workload_ocpsandbox_argocd_user_cluster_resource_whitelist:
- group: ""
  kind: Namespace
- group: rbac.authorization.k8s.io
  kind: ClusterRole
- group: rbac.authorization.k8s.io
  kind: ClusterRoleBinding
```

The role uses `ACTION` variable (set automatically by AgnosticD): `provision` runs `workload.yml`, `destroy` runs `remove_workload.yml`.

### Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `_openshift_api_url` | `{{ sandbox_openshift_api_url }}` | Cluster API URL |
| `_openshift_api_token` | `{{ cluster_admin_agnosticd_sa_token }}` | SA token |
| `_username` | `user1-{{ guid }}` | Username for the AppProject |
| `_namespace` | `openshift-gitops` | ArgoCD namespace |
| `_project_prefix` | `appproject-` | Prefix for AppProject name |
| `_cluster_resource_whitelist` | `[]` | Cluster-scoped resources the project can manage |

All variables are prefixed with `ocp4_workload_ocpsandbox_argocd_user`.

### What It Creates

- AppProject `{prefix}{username}` in `openshift-gitops` namespace
- Destinations: `*-{username}` (all namespaces ending with the username)
- sourceRepos: `*` (all repos allowed)
- User role with full application RBAC within the project
- Group binding for `{username}` (Keycloak group mapping)

### Destroy

Cleanup order: ApplicationSets -> Applications -> wait for deletion -> AppProject.

## Prerequisites

- OpenShift GitOps deployed via `ocp4_workload_openshift_gitops` (cluster provisioning)
- `ocp4_workload_ocpsandbox_keycloak_user` must run first (for cluster connectivity)
