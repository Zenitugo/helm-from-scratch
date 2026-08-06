# Understanding `deployment.yaml`

The `deployment.yaml` file is a **Helm template** that defines how a Kubernetes Deployment should be created. Unlike a standard Kubernetes Deployment manifest, this file is not meant to be applied directly to a cluster. Instead, it contains placeholders and template logic that Helm processes before generating the final Kubernetes manifest.

This template is designed to be **reusable**. Rather than hardcoding values such as the number of replicas, container image, ports, or resource limits, it retrieves them from `values.yaml`. This separation allows the same template to be used across multiple environments (development, staging, production) by simply providing different values.

---

## Key Realization

One of the biggest lessons I learned is that **`deployment.yaml` contains the logic, while `values.yaml` contains the data.**

Helm combines both files to produce the final Kubernetes Deployment manifest.

The flow looks like this:

```text
values.yaml
      │
      ▼
deployment.yaml (template)
      │
      ▼
Helm renders the template
      │
      ▼
Kubernetes Deployment Manifest
      │
      ▼
Kubernetes Cluster
```

---

# Understanding the Template

## Metadata

```yaml
metadata:
  name: {{ include "my-nginx-app.fullname" . }}
```

The Deployment name is not hardcoded. Instead, Helm generates it dynamically using a helper function. This makes the chart reusable because the same template can be installed multiple times with different release names without causing naming conflicts.

---

## Replica Count

```yaml
replicas: {{ .Values.replicaCount }}
```

Instead of specifying a fixed number of Pods, the Deployment retrieves the replica count from `values.yaml`.

Example:

```yaml
replicaCount: 3
```

Helm renders:

```yaml
replicas: 3
```

If the application needs five replicas instead of three, only `values.yaml` needs to be updated. The template remains unchanged.

This demonstrates one of Helm's biggest advantages: **reusable templates with configurable values.**

---

## Conditional Logic

One of the most interesting parts of the template is:

```yaml
{{- if not .Values.autoscaling.enabled }}
replicas: {{ .Values.replicaCount }}
{{- end }}
```

This means:

* If autoscaling is **disabled**, include the `replicas` field.
* If autoscaling is **enabled**, omit the `replicas` field entirely.

Initially, I wondered why Helm would remove the replica count.

The reason is that when the Horizontal Pod Autoscaler (HPA) is enabled, Kubernetes should control the number of Pods instead of using a fixed replica count.

Having both a fixed replica count and an HPA trying to control scaling would create conflicting sources of truth.

---

## Container Image

```yaml
image:
  "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
```

The image consists of two parts:

* Repository
* Tag

Both are configurable.

For example:

```yaml
image:
  repository: nginx
  tag: "1.27"
```

renders:

```yaml
image: nginx:1.27
```

If no tag is provided:

```yaml
tag: ""
```

Helm automatically uses the chart's `appVersion` from `Chart.yaml`.

This was my first introduction to Helm's `default` function.

---

## Image Pull Policy

```yaml
imagePullPolicy: {{ .Values.image.pullPolicy }}
```

The pull policy is also configurable through `values.yaml`.

Example:

```yaml
image:
  pullPolicy: IfNotPresent
```

This determines when Kubernetes should pull the container image.

---

## Container Port

```yaml
containerPort: {{ .Values.service.port }}
```

The container port comes directly from:

```yaml
service:
  port: 80
```

This reinforced an important lesson:

The template simply reads values from `values.yaml`; it does not contain the actual configuration.

---

## Resources

```yaml
resources:
```

CPU and memory requests and limits are defined in `values.yaml`.

Instead of editing the template every time resource requirements change, only the configuration file needs updating.

---

## Node Scheduling

The Deployment also retrieves scheduling-related settings from `values.yaml`:

* `nodeSelector`
* `affinity`
* `tolerations`

These determine **where** Pods are scheduled in the cluster.

Keeping these values outside the template allows different environments to have different scheduling policies without modifying the Deployment template itself.

---

## Security Settings

The template references:

* `podSecurityContext`
* `securityContext`

These allow security configurations to be customized without changing the Deployment template.

---

# A Pattern I Noticed

While studying the template, I realized almost every configurable field points back to `values.yaml`.

Some examples include:

| Value in `values.yaml` | Used in `deployment.yaml`        |
| ---------------------- | -------------------------------- |
| `replicaCount`         | `replicas`                       |
| `image.repository`     | Container image                  |
| `image.tag`            | Container image tag              |
| `image.pullPolicy`     | Image pull policy                |
| `service.port`         | Container port                   |
| `resources`            | CPU and memory limits            |
| `nodeSelector`         | Node scheduling                  |
| `affinity`             | Scheduling rules                 |
| `tolerations`          | Scheduling rules                 |
| `podSecurityContext`   | Pod security configuration       |
| `securityContext`      | Container security configuration |

This helped me realize that **`values.yaml` acts as the control panel for the Deployment template.**

---

# Comparing Helm to Terraform

One of the biggest "aha!" moments during this lesson was noticing a similar design pattern in Terraform.

In Terraform:

* Modules contain reusable infrastructure code.
* `terraform.tfvars` supplies environment-specific values.

In Helm:

* Templates contain reusable Kubernetes manifests.
* `values.yaml` supplies environment-specific values.

Both tools separate reusable logic from configuration, making it possible to reuse the same template or module across multiple environments.


## Connecting Kubernetes HPA to AWS Auto Scaling Groups

While studying the `deployment.yaml` template, I came across this conditional block:

```yaml
{{- if not .Values.autoscaling.enabled }}
replicas: {{ .Values.replicaCount }}
{{- end }}
```

At first, I wondered why Helm would completely remove the `replicas` field when autoscaling is enabled.

Then I connected it to something I already understood from AWS.

An AWS Auto Scaling Group (ASG) automatically increases or decreases the number of EC2 instances based on metrics such as CPU utilization. Similarly, Kubernetes uses a Horizontal Pod Autoscaler (HPA) to automatically increase or decrease the number of Pods based on metrics like CPU or memory utilization.

This comparison helped me understand why the Deployment template only includes a fixed replica count when autoscaling is disabled.

* **Without HPA:** The Deployment uses the value from `replicaCount` to maintain a fixed number of Pods.
* **With HPA:** Kubernetes automatically adjusts the replica count based on workload, so specifying a fixed number of replicas would conflict with the HPA's responsibility.

One important distinction I learned is that the AWS and Kubernetes components are not exact equivalents:

| AWS                                   | Kubernetes                              |
| ------------------------------------- | --------------------------------------- |
| CloudWatch (collects metrics)         | Metrics Server (collects metrics)       |
| Auto Scaling Group (scales resources) | Horizontal Pod Autoscaler (scales Pods) |
| EC2 Instances                         | Pods                                    |

Another realization was that **Helm is not involved once the application has been deployed**. Helm renders the templates into Kubernetes manifests, sends them to the Kubernetes API, and exits.

From that point onward, Kubernetes manages the application.

If CPU usage increases a week later, Helm does nothing. The Metrics Server provides the metrics, the HPA decides whether scaling is necessary, and the Deployment updates the number of Pods accordingly.

This reinforced an important lesson for me:

> **Helm's responsibility is to deploy Kubernetes resources. Kubernetes' responsibility is to manage those resources after deployment.**




---

# My Biggest Takeaway

Before this lesson, I thought `deployment.yaml` was just another Kubernetes Deployment file.

Now I understand that it is **a reusable blueprint**.

The template rarely changes.

Instead, the configuration changes through `values.yaml`, and Helm combines both to generate a standard Kubernetes Deployment manifest.

This design keeps charts reusable, reduces duplication, and makes it easy to deploy the same application to different environments with different configurations.
