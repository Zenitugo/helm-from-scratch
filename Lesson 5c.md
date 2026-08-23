# Helm Commands Quick Reference

Helm commands are used to **create, inspect, validate, install, upgrade, and manage Helm releases**.



## 1. `helm create`

Creates a new Helm chart with the standard directory structure.

```bash
helm create my-nginx-app
```

Creates:

```text
my-nginx-app/
├── Chart.yaml
├── values.yaml
└── templates/
```


## 2. `helm lint`

Checks a Helm chart for potential errors and problems.

```bash
helm lint my-nginx-app
```

Think:

> **"Is my chart structured and written correctly?"**


## 3. `helm template`

Renders Helm templates into Kubernetes YAML **without installing anything**.

```bash
helm template my-nginx-app ./my-nginx-app
```

Think:

> **"Show me the Kubernetes manifests Helm will generate."**

Useful for inspecting:

```text
values.yaml
     +
templates
     ↓
Rendered Kubernetes YAML
```


## 4. `helm install`

Installs a Helm chart and creates a Helm release.

```bash
helm install my-nginx ./my-nginx-app
```

The first `my-nginx` is the **release name**.

The second `./my-nginx-app` is the chart.


## 5. `helm list`

Shows Helm releases installed in the current namespace.

```bash
helm list
```

Useful for seeing:

```text
Release
Status
Revision
Chart
Namespace
```


## 6. `helm status`

Shows information about a particular Helm release.

```bash
helm status my-nginx
```


## 7. `helm get manifest`

Shows the Kubernetes manifests associated with a Helm release.

```bash
helm get manifest my-nginx
```

Useful when you want to see what Kubernetes resources Helm installed.


## 8. `helm upgrade`

Updates an existing Helm release using a new chart configuration.

```bash
helm upgrade my-nginx ./my-nginx-app
```

For example:

```text
Current:
nginx:1.27

values.yaml changed:

nginx:1.28

        ↓

helm upgrade

        ↓

Kubernetes Deployment

        ↓

Rolling Update
```

Helm sends the new desired configuration to Kubernetes; the **Deployment controller performs the rolling update**.


## 9. `helm history`

Shows the revision history of a Helm release.

```bash
helm history my-nginx
```

Example:

```text
REVISION   STATUS
1          superseded
2          deployed
```

Each revision represents a recorded version of the Helm release.


## 10. `helm rollback`

Rolls a Helm release back to a previous revision.

```bash
helm rollback my-nginx 1
```

You **do not manually edit `values.yaml` just to perform the rollback**.

Helm uses the stored release history to restore the previous configuration.


## 11. `helm uninstall`

Removes an installed Helm release.

```bash
helm uninstall my-nginx
```

This removes the Kubernetes resources managed by that release.


# Helm Lifecycle

The commands can be remembered as:

```text
helm create
      ↓
helm lint
      ↓
helm template
      ↓
helm install
      ↓
helm list / helm status
      ↓
helm upgrade
      ↓
helm history
      ↓
helm rollback
      ↓
helm uninstall
```

### Most important commands to remember

```text
helm lint
→ Validate the chart

helm template
→ Render YAML without installing

helm install
→ Create a new release

helm upgrade
→ Update an existing release

helm history
→ View release revisions

helm rollback
→ Return to a previous revision

helm uninstall
→ Remove the release
```
