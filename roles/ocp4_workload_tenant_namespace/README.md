# ocp4_workload_tenant_namespace

Creates one or more OpenShift namespaces for a tenant user, applies resource controls, and grants RBAC access.

## What it does

- Creates namespaces named `{username}-{suffix}`, or a single namespace named after the user when no suffixes are defined
- Applies a `LimitRange` to every namespace to set container resource defaults
- Creates a `ClusterResourceQuota` (default) selecting namespaces by the user's `openshift.io/requester` annotation, giving the user a shared resource pool
- Grants the user the configured RBAC role in each namespace

## Usage

Set `ocp4_workload_tenant_namespace_username` and optionally define the namespaces to create:

```yaml
ocp4_workload_tenant_namespace_username: "user-{{ guid }}"

ocp4_workload_tenant_namespace_suffixes:
- suffix: myapp
- suffix: mydb
```

Leave `suffixes` empty to create a single namespace named after the user.

See [`defaults/main.yml`](defaults/main.yml) for all variables and their descriptions, including quota sizing and how to switch between ClusterResourceQuota and per-namespace ResourceQuota.

## Cluster quota scope

With cluster quota enabled, the role sets `openshift.io/requester` to `ocp4_workload_tenant_namespace_username` on every namespace it creates, including a single namespace with no suffixes. Existing labels and other metadata are preserved; the requester annotation takes precedence over custom metadata. OpenShift also sets it on projects requested by that user through the ProjectRequest API (for example, `oc new-project`), so those projects share the same quota without needing a tenant label.

Additional namespaces created directly by privileged automation, including GitOps, must explicitly carry the same requester annotation to join the quota. Quota membership does not apply this role's LimitRange or RBAC to those namespaces. Cleanup still deletes only the namespaces declared through this role, not other namespaces matching the quota.

With `ocp4_workload_tenant_namespace_use_cluster_quota: false`, per-namespace ResourceQuota behavior is unchanged and the role does not add the requester annotation.
