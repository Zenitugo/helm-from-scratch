# Lesson 4.3 – Understanding the Pod Template, ReplicaSets, and Rollbacks


## Why Does the Pod Template Have Its Own Metadata and Spec?

At first, I thought the `metadata` and `spec` inside the Pod template were duplicates of the Deployment's `metadata` and `spec`.

I later realized they belong to **different Kubernetes resources**.

The Deployment has its own metadata because it is a Kubernetes resource.

The Pod also has its own metadata because it is another Kubernetes resource.

Similarly, the Deployment has its own specification, while every Pod also needs its own specification.

### Deployment Metadata

The Deployment metadata identifies and describes the Deployment itself.

Examples include:

- Name
- Labels
- Namespace
- Annotations

It answers the question:

> "Who am I as a Deployment?"



### Pod Template Metadata

The metadata inside the Pod template does **not** describe the Deployment.

Instead, it defines the metadata that Kubernetes should assign to every Pod created from that template.

For example:

```yaml
template:
  metadata:
    labels:
      app: frontend
```

If the Deployment creates three Pods, Kubernetes automatically labels all three Pods with:

```yaml
app: frontend
```

These labels are important because other Kubernetes resources use them to locate the Pods.

---

## Labels Inside the Pod Template

Another important lesson was understanding why the Pod template contains metadata.

The labels inside the Pod template are copied onto every Pod that Kubernetes creates.

For example:

```yaml
template:
  metadata:
    labels:
      app: frontend
```

Every Pod created from that template automatically receives:

```yaml
app: frontend
```

---

## Deployment Selectors

I initially confused the purpose of the Deployment selector.

I later learned that the selector does **not** find Deployments.

Instead, the Deployment uses its selector to find the Pods that it manages.

For example:

```yaml
selector:
  matchLabels:
    app: frontend
```

means:

> "Manage every Pod whose label is `app=frontend`."

This selector must match the labels defined in the Pod template.

If they do not match, the Deployment creates Pods but cannot recognize them as its own.

---

## Labels and Selectors Work Together

One of the simplest ways I learned to remember this concept is:

> **Labels identify resources. Selectors find resources.**

Pods receive labels.

Deployments and Services use selectors to find those Pods.

Everything in Kubernetes revolves around this relationship.

---


## One Pod Template Can Produce Many Pods

Another realization was understanding the difference between a Pod template and the Pods themselves.

The Deployment manages **one Pod template**, not one Pod.

If the Deployment specifies:

```yaml
replicas: 3
```

Kubernetes creates three identical Pods from the same Pod template.

For example:

```text
Deployment
      │
      ▼
Pod Template
      │
      ▼
Pod 1
Pod 2
Pod 3
```

This is similar to making multiple photocopies from a single original document.

---

## Replicas vs ReplicaSet

Although their names sound similar, they represent different concepts.

### Replicas

Replicas simply represent the desired number of Pods.

Example:

```yaml
replicas: 3
```

means:

> "I want three Pods."

Replicas are only a number.

### ReplicaSet

A ReplicaSet is a Kubernetes controller.

Its responsibility is to ensure that the desired number of Pods is always running.

If one Pod crashes, the ReplicaSet immediately creates another Pod to replace it.

This helped me understand that:

- Replicas specify **how many** Pods should exist.
- ReplicaSets ensure that the requested number always exists.

---

## Why Doesn't Kubernetes Modify the Existing Pods?

Pods are considered temporary (ephemeral).

Rather than changing an existing Pod, Kubernetes creates a new Pod from the updated Pod template.

This design keeps deployments predictable and reliable.

---

## Understanding Rollbacks

A rollback is essentially a rolling update in the opposite direction.

If version 1.28 contains a bug, Kubernetes does not manually edit the existing Pods.

Instead, it returns to the previous Pod template.

Since the previous ReplicaSet is still available, Kubernetes performs another rolling update, replacing the new Pods with Pods created from the older ReplicaSet.

The process looks like this:

```text
Pods (v1.28)

↓

Create Pod (v1.27)

↓

Wait until healthy

↓

Delete one Pod (v1.28)

↓

Repeat
```

Eventually, all Pods return to version 1.27.

---

## Why Kubernetes Creates New ReplicaSets

Initially, I wondered why Kubernetes creates a completely new ReplicaSet instead of modifying the old one.

I later understood that Kubernetes preserves previous ReplicaSets so that deployments can be rolled back safely.

Each ReplicaSet represents a snapshot of a specific Pod template.

Without these previous ReplicaSets, Kubernetes would have no previous version to restore.

---

## Helm's Role

Another important lesson was understanding Helm's responsibility.

Helm renders templates into Kubernetes manifests and submits them to the Kubernetes API.

Once the resources are created, Helm's work is complete.

From that point onward:

- Kubernetes performs rolling updates.
- Kubernetes performs rollbacks.
- Kubernetes manages ReplicaSets.
- Kubernetes creates and replaces Pods.

Helm simply provides Kubernetes with the desired configuration.

### How Helm Upgrade and Helm Rollback Work

One misconception I initially had was thinking that rolling back meant manually editing `values.yaml` or changing the image version back to its previous value.

I later learned that this is **not** how rollbacks are typically performed.

### Upgrading an Application

Suppose the current application is running:

```yaml
image: nginx:1.27
```

I update my chart (or `values.yaml`) to:

```yaml
image: nginx:1.28
```

Then I run:

```bash
helm upgrade my-nginx-app .
```

Helm renders the updated templates and sends a new Deployment definition to Kubernetes.

Kubernetes detects that the Pod template has changed and creates a new ReplicaSet.

The Deployment Controller then performs a rolling update, gradually replacing Pods running version `1.27` with Pods running version `1.28`.

Notice that Helm does **not** perform the rolling update itself.

Helm simply updates the Deployment resource.

Kubernetes performs the actual rollout.

---

## Rolling Back an Application

If the new version contains a bug, I do **not** manually edit the chart back to the previous version.

Instead, Helm keeps a history of every release revision.

For example:

```text
Revision 1 → nginx:1.27

Revision 2 → nginx:1.28
```

To return to the previous release, I can run:

```bash
helm rollback my-nginx-app 1
```

Helm retrieves the Deployment definition from Revision 1 and submits it back to Kubernetes.

Kubernetes then performs another rolling update, replacing the Pods running version `1.28` with Pods created from the previous Pod template.

This taught me that:

* **Helm stores release history.**
* **Kubernetes stores Pod template history through ReplicaSets.**
* **Helm requests the rollback, while Kubernetes performs it.**

In other words:

> **Helm manages release history, while Kubernetes manages application state.**

---
## ReplicaSet vs Horizontal Pod Autoscaler

| ReplicaSet                                       | HPA                                                               |
| ------------------------------------------------ | ----------------------------------------------------------------- |
| Ensures the requested number of Pods is running. | Decides whether the requested number should increase or decrease. |
| Replaces failed Pods.                            | Responds to workload changes.                                     |
| Watches Pod count.                               | Watches metrics (CPU, memory, custom metrics).                    |
| Maintains the current target.                    | Changes the target.                                               |





---

# My Biggest Takeaways

This lesson fundamentally changed how I think about Deployments.

I now understand that:

- A Deployment manages ReplicaSets.
- ReplicaSets manage Pods.
- Pods run one or more containers.
- A Pod template is the blueprint for every Pod Kubernetes creates.
- The Deployment selector identifies which Pods belong to the Deployment.
- Pod template labels become the labels on every Pod.
- Rolling updates replace Pods gradually using a new ReplicaSet.
- Rollbacks restore a previous ReplicaSet through another rolling update.
- Helm deploys resources, while Kubernetes manages their entire lifecycle after deployment.

One sentence summarizes this entire lesson for me:

> **Kubernetes doesn't modify Pods. It creates new Pods from a Pod template until the cluster matches the desired state.**


