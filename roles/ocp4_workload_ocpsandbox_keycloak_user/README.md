# ocp4_workload_ocpsandbox_keycloak_user

User provisioning role for OcpSandbox clusters with Keycloak (RHBK) authentication.

## When to Use

Use this role as the **first workload** in any OcpSandbox (`config: namespace`) catalog item where the cluster uses Keycloak for authentication. It handles kubeconfig setup, cluster discovery, and user creation -- things that `config: namespace` does NOT do automatically.

## Why It Exists

The `config: namespace` pattern in AgnosticD v2 runs workloads on `localhost` with no automatic cluster auth setup or discovery. This role fills that gap for sandbox deployments by:

1. Writing a kubeconfig from the sandbox-provided SA token
2. Discovering cluster info (ingress domain, console URL, API URL)
3. Creating a Keycloak user via REST API so the student can log in

Without this role, downstream workloads would have no cluster connection and no way to discover ingress domains or console URLs.

## How to Use

### In AgV catalog (`common.yaml`)

```yaml
config: namespace

# Provision order -- keycloak_user must be first
workloads:
- agnosticd.namespaced_workloads.ocp4_workload_ocpsandbox_keycloak_user
- agnosticd.namespaced_workloads.ocp4_workload_ocpsandbox_gitea_user
# ... other workloads

# Destroy order -- keycloak_user must be last (cleans up cluster connection)
remove_workloads:
# ... other workloads in reverse
- agnosticd.namespaced_workloads.ocp4_workload_ocpsandbox_gitea_user
- agnosticd.namespaced_workloads.ocp4_workload_ocpsandbox_keycloak_user

# Variables
ocp4_workload_ocpsandbox_keycloak_user_username: "user1-{{ guid }}"
ocp4_workload_ocpsandbox_keycloak_user_password: "{{ common_password }}"
ocp4_workload_ocpsandbox_keycloak_user_keycloak_realm: sso
```

The role uses `ACTION` variable (set automatically by AgnosticD): `provision` runs `workload.yml`, `destroy` runs `remove_workload.yml`.

### Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `_openshift_api_url` | `{{ sandbox_openshift_api_url }}` | Cluster API URL (from sandbox) |
| `_openshift_api_token` | `{{ cluster_admin_agnosticd_sa_token }}` | SA token (from sandbox) |
| `_username` | `user1-{{ guid }}` | Username to create |
| `_password` | (required) | Password for the user |
| `_keycloak_namespace` | `keycloak` | Namespace where RHBK is deployed |
| `_keycloak_realm` | `openshift` | Keycloak realm for user creation |
| `_keycloak_admin_secret` | `keycloak-initial-admin` | K8s secret with admin creds |

All variables are prefixed with `ocp4_workload_ocpsandbox_keycloak_user`.

### Facts Set for Downstream Roles

- `openshift_cluster_ingress_domain` -- e.g. `apps.cluster-x2r6h.dynamic.redhatworkshops.io`
- `openshift_console_url` -- e.g. `https://console-openshift-console.apps...`
- `openshift_api_url` -- e.g. `https://api.cluster-x2r6h.dynamic.redhatworkshops.io:6443`

### Destroy

Removes: Keycloak user, OCP Identity, OCP User object, per-user namespaces, ClusterRoleBindings.

## Prerequisites

- Cluster must have RHBK (Keycloak) deployed via `ocp4_workload_authentication_keycloak`
- `cluster_admin_agnosticd_sa_token` and `sandbox_openshift_api_url` provided by sandbox
