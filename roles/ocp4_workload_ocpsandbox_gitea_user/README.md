# ocp4_workload_ocpsandbox_gitea_user

User provisioning role for OcpSandbox clusters with Gitea.

## When to Use

Use this role in any OcpSandbox catalog item where students need their own Gitea user account and repositories. Typically runs after `ocp4_workload_ocpsandbox_keycloak_user` (which sets up cluster connectivity).

## Why It Exists

The Gitea operator deploys a shared Gitea instance with an admin account, but no per-user accounts. This role creates individual student accounts via the Gitea REST API and optionally migrates (forks) repositories from GitHub into each student's account.

## How to Use

### In AgV catalog (`common.yaml`)

```yaml
# Provision order
workloads:
- agnosticd.namespaced_workloads.ocp4_workload_ocpsandbox_keycloak_user
- agnosticd.namespaced_workloads.ocp4_workload_ocpsandbox_gitea_user
# ... other workloads

# Destroy order (reverse)
remove_workloads:
# ... other workloads
- agnosticd.namespaced_workloads.ocp4_workload_ocpsandbox_gitea_user
- agnosticd.namespaced_workloads.ocp4_workload_ocpsandbox_keycloak_user

# Variables
ocp4_workload_ocpsandbox_gitea_user_username: "user1-{{ guid }}"
ocp4_workload_ocpsandbox_gitea_user_password: "{{ common_password }}"
ocp4_workload_ocpsandbox_gitea_user_repositories:
- name: mcp
  repo: https://github.com/rhpds/ocpsandbox-mcp-with-openshift-gitops
  private: false
```

The role uses `ACTION` variable (set automatically by AgnosticD): `provision` runs `workload.yml`, `destroy` runs `remove_workload.yml`.

### Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `_openshift_api_url` | `{{ sandbox_openshift_api_url }}` | Cluster API URL |
| `_openshift_api_token` | `{{ cluster_admin_agnosticd_sa_token }}` | SA token |
| `_username` | `user1-{{ guid }}` | Gitea username to create |
| `_password` | (required) | Password for the user |
| `_namespace` | `gitea` | Namespace where Gitea is deployed |
| `_admin_user` | (auto-discovered) | Gitea admin username (from Gitea CR) |
| `_admin_password` | (auto-discovered) | Gitea admin password (from Gitea CR) |
| `_repositories` | `[]` | List of repos to migrate for the user |

All variables are prefixed with `ocp4_workload_ocpsandbox_gitea_user`.

### Key Behaviors

- Admin creds auto-discovered from the Gitea CR if not provided
- User creation is idempotent: 201 = new, 422 = already exists (updates password)
- Repo migration is idempotent: 201 = migrated, 409 = already exists
- Destroy uses `?purge=true` to delete the user and all their data

### Destroy

Removes: Gitea user and all their repositories (purge).

## Prerequisites

- Gitea deployed via `ocp4_workload_gitea_operator` (cluster provisioning)
- `ocp4_workload_ocpsandbox_keycloak_user` must run first (for cluster connectivity)
