# ocp4_workload_tenant_openshift_gitops

Ansible role to configure tenant-scoped access to an existing OpenShift GitOps (ArgoCD) instance. This role creates an ArgoCD AppProject for a single user and configures authentication and RBAC without requiring multi-user provisioning logic.

## Requirements

- OpenShift GitOps operator already installed and running in `openshift-gitops` namespace
- Cluster admin API credentials for the target cluster
- Python passlib library (for bcrypt password hashing)

## Role Variables

### Required Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `ocp4_workload_tenant_openshift_gitops_openshift_api_url` | OpenShift API URL | `{{ sandbox_openshift_api_url }}` |
| `ocp4_workload_tenant_openshift_gitops_openshift_api_token` | Cluster admin API token | `{{ cluster_admin_agnosticd_sa_token \| trim }}` |
| `ocp4_workload_tenant_openshift_gitops_user` | Username for ArgoCD access | `user-{{ guid }}` |
| `ocp4_workload_tenant_openshift_gitops_user_password` | Password for ArgoCD login | `{{ common_password \| default('openshift') }}` |

### Optional Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `ocp4_workload_tenant_openshift_gitops_user_project_prefix` | Prefix for AppProject name | `appproject-` |
| `ocp4_workload_tenant_openshift_gitops_user_cluster_resource_whitelist` | Cluster-scoped resources allowed in AppProject | `[{ group: "", kind: Namespace }]` |

## AppProject Naming

The AppProject name is constructed as: `{{ prefix }}-{{ user }}`

Example: With default prefix `appproject-` and user `user-abc123`, the AppProject name will be `appproject-user-abc123`

## Namespace Access Pattern

Users can deploy applications to namespaces matching the pattern: `{{ username }}-*`

Example: User `user-abc123` can deploy to:
- `user-abc123-dev`
- `user-abc123-prod`
- `user-abc123-gitops`

## Dependencies

None

## Example Playbook

```yaml
---
- name: Configure tenant GitOps access
  hosts: localhost
  gather_facts: false
  tasks:
  - name: Setup ArgoCD tenant access
    ansible.builtin.include_role:
      name: ocp4_workload_tenant_openshift_gitops
    vars:
      ocp4_workload_tenant_openshift_gitops_user: "user-demo"
      ocp4_workload_tenant_openshift_gitops_user_cluster_resource_whitelist:
      - group: ""
        kind: Namespace
      - group: rbac.authorization.k8s.io
        kind: ClusterRole
```

## Example with Custom Configuration

```yaml
---
- name: Configure tenant GitOps with custom settings
  hosts: localhost
  gather_facts: false
  tasks:
  - name: Setup ArgoCD tenant access
    ansible.builtin.include_role:
      name: ocp4_workload_tenant_openshift_gitops
    vars:
      ocp4_workload_tenant_openshift_gitops_openshift_api_url: "https://api.cluster.example.com:6443"
      ocp4_workload_tenant_openshift_gitops_openshift_api_token: "sha256~abc123..."
      ocp4_workload_tenant_openshift_gitops_user: "developer1"
      ocp4_workload_tenant_openshift_gitops_user_project_prefix: "tenant-"
```

## What This Role Does

### Provision (`ACTION=provision`)

1. **Creates AppProject** - Creates a tenant-scoped ArgoCD AppProject
2. **Registers User Account** - Adds user account to ArgoCD configuration (`argocd-cm`)
3. **Sets Password** - Generates bcrypt hash and stores in ArgoCD secret (`argocd-secret`)
4. **Configures RBAC** - Maps user to AppProject role in ArgoCD RBAC (`argocd-rbac-cm`)
5. **Outputs User Data** - Provides ArgoCD URL, username, password, and AppProject name

### Destroy (`ACTION=destroy`)

1. **Removes AppProject** - Deletes the tenant's AppProject
2. **Removes User Account** - Cleans up user account from `argocd-cm`
3. **Removes Password** - Cleans up password from `argocd-secret`

## User Access

After provisioning, users can log in to ArgoCD:

- **URL**: Retrieved from the `openshift-gitops-server` route
- **Username**: Value of `ocp4_workload_tenant_openshift_gitops_user`
- **Password**: Value of `ocp4_workload_tenant_openshift_gitops_user_password`

## Output Variables

The role outputs the following data via `agnosticd_user_info`:

```yaml
openshift_gitops_server: "https://openshift-gitops-server-openshift-gitops.apps.cluster.example.com"
openshift_gitops_user: "user-abc123"
openshift_gitops_password: "openshift"
openshift_gitops_appproject: "appproject-user-abc123"
```

## Cluster Resource Whitelist

Control which cluster-scoped resources users can manage in their AppProject:

```yaml
ocp4_workload_tenant_openshift_gitops_user_cluster_resource_whitelist:
- group: ""
  kind: Namespace
- group: rbac.authorization.k8s.io
  kind: ClusterRole
- group: rbac.authorization.k8s.io
  kind: ClusterRoleBinding
```

## RBAC Behavior

The role **appends** to existing RBAC policy in `argocd-rbac-cm` using `state: patched`. This preserves other users' RBAC configurations.

## License

GPL-3.0-or-later

## Author Information

Red Hat GPTE
