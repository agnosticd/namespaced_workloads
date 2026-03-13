# ocp4_workload_tenant_namespace

Creates one or more OpenShift namespaces for a tenant user, applies resource controls, and grants RBAC access.

## What it does

- Creates namespaces named `{username}-{suffix}`, or a single namespace named after the user when no suffixes are defined
- Applies a `LimitRange` to every namespace to set container resource defaults
- Creates a `ClusterResourceQuota` (default) scoped to all tenant namespaces via a shared `tenant:` label, giving the user a flexible resource pool rather than fixed per-namespace caps
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
