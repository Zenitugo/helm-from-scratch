# Lesson 4.5 — Kubernetes Resources

# Kubernetes Resources: Requests, Limits, Nodes, and Scheduling

## 1. What is a Kubernetes Node?

A **Node** is a machine that runs Kubernetes workloads (Pods).

In Amazon EKS, a Node is commonly an **EC2 instance** managed as part of a Kubernetes node group.

For example:

```text
EKS Cluster
│
├── Node 1 → EC2 instance
├── Node 2 → EC2 instance
└── Node 3 → EC2 instance
```

The underlying EC2 instances provide the actual:

* CPU
* Memory
* Storage
* Network resources

that Kubernetes workloads consume.

A Node might have, for example:

```text
4 vCPUs
16 GiB Memory
```

The Node does not necessarily make every bit of its resources available to Pods because the operating system and Kubernetes components also need resources. Kubernetes therefore works with the Node's **allocatable resources** when scheduling workloads.

---

## 2. Where does the Pod fit?

A Pod is the basic unit in Kubernetes that runs application containers.

The relationship is:

```text
Kubernetes Cluster
        ↓
      Node
        ↓
       Pod
        ↓
   Container
        ↓
   Application
```

A Pod needs a Node because the Node is the machine that actually provides the resources needed to run the Pod's containers.

A Pod does not normally choose the Node itself.

The **Kubernetes Scheduler** determines which suitable Node should run the Pod.

---

# 3. Why does Kubernetes need resource requests?

When a Pod is created, Kubernetes needs to determine:

> "Which Node has enough resources to run this Pod?"

For Kubernetes to make this decision, the Pod can declare its resource requirements using:

```yaml
resources:
  requests:
    cpu:
    memory:
```

For example:

```yaml
resources:
  requests:
    cpu: 500m
    memory: 512Mi
```

This tells Kubernetes:

> "For scheduling purposes, this container requires 500m CPU and 512Mi memory."

The scheduler uses these requests when deciding whether a Node can accommodate the Pod.

---

# 4. CPU Requests

CPU is commonly expressed in **CPU cores** or **millicores**.

Examples:

```text
1000m = 1 CPU
500m  = 0.5 CPU
250m  = 0.25 CPU
100m  = 0.1 CPU
```

Therefore:

```yaml
requests:
  cpu: 500m
```

means the container requests approximately **half of one CPU core**.

---

# 5. Memory Requests

Memory can be specified using units such as:

```text
Ki
Mi
Gi
```

For example:

```yaml
requests:
  memory: 512Mi
```

means the container requests **512 MiB of memory**.

The scheduler uses the requested memory when determining whether the Pod can fit on a Node.

---

# 6. How the Scheduler Uses Requests

Suppose a Node has:

```text
4 CPUs
```

and two existing Pods request:

```text
Pod A → 2 CPU
Pod B → 1 CPU
```

The requested CPU is:

```text
2 + 1 = 3 CPU
```

Approximately 1 CPU remains available for scheduling purposes.

If another Pod requests:

```yaml
requests:
  cpu: 2
```

the scheduler cannot place it on that Node because the Node cannot satisfy the Pod's requested CPU.

It will look for another suitable Node.

For example:

```text
Node A
Available: 1 CPU
Pod requires: 2 CPU

❌ Cannot schedule


Node B
Available: 3 CPU
Pod requires: 2 CPU

✅ Can schedule
```

This demonstrates an important rule:

> **Resource requests influence Pod scheduling.**

---

# 7. Requests Are Not the Maximum

One of the most important concepts I learned is that a resource request is **not necessarily the maximum amount of resource a container can use**.

For example:

```yaml
resources:
  requests:
    cpu: 500m
  limits:
    cpu: 1
```

The container requests:

```text
500m CPU
```

but can use more if resources are available, up to its configured limit.

For example:

```text
Request: 500m
Actual usage: 800m
Limit: 1 CPU
```

This is allowed because:

```text
800m < 1000m
```

The container does not have to remain at exactly 500m.

A useful mental model is:

> **Request = what Kubernetes uses for scheduling.**

> **Limit = the maximum resource usage allowed for the container.**

---

# 8. Resource Limits

Limits are defined using:

```yaml
resources:
  limits:
    cpu:
    memory:
```

For example:

```yaml
resources:
  requests:
    cpu: 500m
    memory: 512Mi

  limits:
    cpu: 1
    memory: 1Gi
```

This means:

```text
CPU
Request → 500m
Limit   → 1 CPU

Memory
Request → 512Mi
Limit   → 1Gi
```

The application normally requests the smaller amount, but it can use additional resources when available, up to its configured limits.

---

# 9. CPU Limits

CPU is a compressible resource.

If a container tries to use more CPU than its CPU limit allows, it can be **throttled**.

For example:

```text
CPU limit = 1 CPU
Application wants = 1.5 CPU
```

The container cannot simply consume 1.5 CPU beyond its limit.

Instead, CPU usage can be throttled.

This can result in the application becoming slower during periods of high CPU demand.

The Node itself does not suddenly gain additional CPUs just because a Pod wants more.

---

# 10. Memory Limits

Memory behaves differently from CPU.

Memory is not a resource that can be throttled in the same way CPU can.

Suppose:

```yaml
limits:
  memory: 1Gi
```

but the container attempts to consume:

```text
1.2Gi
```

The container may be terminated because it exceeded its memory limit.

This commonly appears as:

```text
OOMKilled
```

where OOM means:

> **Out Of Memory**

The Pod is not simply moved to another Node because it exceeded its memory limit.

---

# 11. CPU and Memory Behavior

A useful comparison is:

| Resource | Request             | Limit exceeded                        |
| -------- | ------------------- | ------------------------------------- |
| CPU      | Used for scheduling | Container can be throttled            |
| Memory   | Used for scheduling | Container may be killed (`OOMKilled`) |

Therefore:

```text
CPU
Request → Scheduling requirement
Limit   → Maximum CPU usage
Excess  → Throttling


Memory
Request → Scheduling requirement
Limit   → Maximum memory usage
Excess  → Possible OOMKilled
```

---

# 12. Requests vs Limits

The simplest way I understand the difference is:

```text
REQUEST
"What resources should Kubernetes account for when
deciding where to place me?"

LIMIT
"What is the maximum resource usage allowed for me?"
```

For example:

```yaml
resources:
  requests:
    cpu: 1
    memory: 512Mi

  limits:
    cpu: 2
    memory: 1Gi
```

The scheduler considers the Pod's request when deciding whether the Node can accommodate it.

The container can potentially use more resources than its request, but it should not exceed its configured limits.

---

# 13. Why Can Requests Be Lower Than Limits?

Requests can be lower than limits because applications don't always use the same amount of resources.

An application might normally use:

```text
500m CPU
```

but occasionally need:

```text
800m CPU
```

during a traffic spike.

Instead of reserving the maximum amount for scheduling, we can configure:

```yaml
requests:
  cpu: 500m

limits:
  cpu: 1
```

This allows the application to **burst** above its normal requested amount when resources are available.

This helps Kubernetes make more efficient use of Node resources.

---

# 14. Why Kubernetes Does Not Schedule Based on Current Usage

An important lesson is that the scheduler does not simply look at how much CPU a running Pod is using at that exact moment.

For example:

```text
Node:
4 CPUs

Pod A:
request = 2 CPU
current usage = 500m

Pod B:
request = 2 CPU
current usage = 500m
```

Even though the Pods are currently using only about 1 CPU combined, Kubernetes has accounted for their **2 CPU requests** when scheduling.

If another Pod requests:

```text
2 CPU
```

the scheduler will not simply say:

> "The existing Pods are only using 500m, so there is plenty of room."

It uses the resource requests to determine whether the Node can satisfy the new Pod's requirements.

Therefore:

> **Requests provide predictable scheduling guarantees rather than relying on current usage.**

---

# 15. Pods Can Compete for Resources

Suppose a Node has:

```text
4 CPUs
```

and three Pods have:

```text
Pod A
request: 1 CPU
limit: 3 CPU

Pod B
request: 1 CPU
limit: 3 CPU

Pod C
request: 1 CPU
limit: 3 CPU
```

The total requested CPU is:

```text
1 + 1 + 1 = 3 CPU
```

Therefore, the Pods can potentially be scheduled because their requests fit within the Node's available capacity.

However, their combined CPU limits could theoretically be:

```text
3 + 3 + 3 = 9 CPU
```

even though the Node only has 4 CPUs.

The Node cannot physically provide 9 CPUs.

If the Pods all demand high CPU simultaneously, they compete for the available CPU and can experience CPU throttling/contention.

This does not mean Kubernetes automatically creates more CPU on that Node.

---

# 16. What Happens When a Pod Does Not Fit?

Suppose:

```text
Node A
4 CPU
```

and existing Pods have requests totaling:

```text
4 CPU
```

A new Pod requests:

```text
2 CPU
```

The scheduler cannot place that Pod on Node A.

It will look for another suitable Node.

If no Node has enough allocatable resources, the Pod may remain **Pending** until sufficient resources become available or another suitable Node is added.

This is different from the application crashing.

The Pod may simply be waiting to be scheduled.

---

# 17. Connection to Amazon EKS

In EKS, a simplified architecture looks like:

```text
AWS
│
└── EKS Cluster
      │
      ├── EC2 Node
      │     ├── Pod
      │     │    └── Container
      │     │
      │     └── Pod
      │          └── Container
      │
      └── EC2 Node
            ├── Pod
            │    └── Container
            │
            └── Pod
                 └── Container
```

The EC2 instances provide the underlying compute resources.

Kubernetes manages how those resources are allocated to Pods.

When a Pod is created:

```text
Pod
 ↓
Scheduler examines resource requests
 ↓
Scheduler examines available Nodes
 ↓
Suitable Node selected
 ↓
Pod runs on that Node
 ↓
Container consumes CPU and memory
```

---

# 18. Relationship Between Deployment, ReplicaSet, Pod, Node, and Container

Putting everything together:

```text
Helm
  │
  │ renders templates
  ▼
Kubernetes API
  │
  ▼
Deployment
  │
  │ manages
  ▼
ReplicaSet
  │
  │ creates
  ▼
Pod
  │
  │ contains
  ▼
Container
  │
  │ runs on
  ▼
Node
  │
  │ provides
  ▼
CPU + Memory
```

The Scheduler is responsible for deciding **which Node** the Pod should run on.

Resource requests help the Scheduler make that decision.

---

# Key Takeaways

### Node

> A machine that runs Kubernetes workloads. In EKS, this is commonly an EC2 instance.

### Pod

> The Kubernetes unit that runs one or more containers.

### Resource Request

> The amount of CPU and memory Kubernetes accounts for when scheduling the Pod.

### Resource Limit

> The maximum CPU and memory the container is allowed to use.

### CPU exceeding its limit

> Can result in CPU throttling.

### Memory exceeding its limit

> Can result in the container being killed, commonly reported as `OOMKilled`.

### Scheduler

> Chooses a suitable Node for a Pod based on resource requests and other scheduling constraints.

### Most important mental model

```text
Request
   ↓
Used primarily for scheduling

Limit
   ↓
Maximum allowed usage

Node
   ↓
Provides the actual CPU and memory

Pod
   ↓
Consumes those resources

Scheduler
   ↓
Places the Pod on a suitable Node
```

The key distinction I learned is:

> **Requests answer "Where can this Pod run?" while limits answer "How much can this container use?"**
