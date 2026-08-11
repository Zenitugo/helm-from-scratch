# Kubernetes ServiceAccount

A **ServiceAccount** provides an identity for a workload running inside Kubernetes.

A Pod can use a ServiceAccount when an application running inside the Pod needs to communicate with the Kubernetes API or, in an EKS environment, potentially access AWS resources through an associated IAM role.

---

## 1. Why Does a Pod Need an Identity?

Imagine an application running inside a Pod wants to communicate with the Kubernetes API.

For example, it might want to request information about Pods:

```text
Application
     ↓
Kubernetes API
```

Kubernetes needs to know:

> "Who is making this request?"

It should not automatically allow every application to perform every action.

A ServiceAccount provides the workload with an identity.

The basic relationship is:

```text
Pod
 ↓
ServiceAccount
 ↓
Identity
 ↓
Kubernetes API
```

The ServiceAccount therefore allows Kubernetes to identify the workload making an API request.

---

# 2. ServiceAccount vs Human User

A ServiceAccount should not be confused with a normal human user identity.

A simple distinction is:

```text
Human
  ↓
User identity

Application / Pod
  ↓
ServiceAccount
```

A human might use:

```bash
kubectl get pods
```

using their own Kubernetes credentials.

An application running inside a Pod, however, can use its assigned ServiceAccount when communicating with the Kubernetes API.

Therefore:

> **User identities are generally associated with people, while ServiceAccounts provide identities for workloads.**

---

# 3. Assigning a ServiceAccount to a Pod

A Pod can be associated with a ServiceAccount using:

```yaml
serviceAccountName: my-nginx-app
```

In a Deployment, this appears inside the Pod template:

```yaml
spec:
  template:
    spec:
      serviceAccountName: my-nginx-app
```

This means:

> "Pods created from this template should use the `my-nginx-app` ServiceAccount."

The Deployment does not create the ServiceAccount itself. It simply references the ServiceAccount that should be used by the Pods.

---

# 4. ServiceAccount in a Helm Chart

In my Helm chart, `values.yaml` contains:

```yaml
serviceAccount:
  create: true
  automount: true
  annotations: {}
  name: ""
```

The chart also contains:

```text
templates/
├── deployment.yaml
└── serviceaccount.yaml
```

The `serviceaccount.yaml` template defines the ServiceAccount resource.

The Deployment then references it through:

```yaml
serviceAccountName: {{ include "my-nginx-app.serviceAccountName" . }}
```

Helm renders this expression into the actual ServiceAccount name.

The relationship is:

```text
values.yaml
      ↓
serviceaccount.yaml
      ↓
ServiceAccount resource
      ↓
deployment.yaml
      ↓
serviceAccountName
      ↓
Pod uses ServiceAccount
```

---

# 5. ServiceAccount Does Not Automatically Give Permissions

An important distinction is:

> **A ServiceAccount provides an identity, but the identity still needs permissions.**

Simply assigning a ServiceAccount to a Pod does not mean the Pod can perform every action in Kubernetes.

Kubernetes uses **Role-Based Access Control (RBAC)** to determine what that ServiceAccount is allowed to do.

The relationship can be represented as:

```text
Pod
 ↓
ServiceAccount
 ↓
RBAC permissions
 ↓
Kubernetes API
```

For example, a ServiceAccount might be given permission to:

```text
Read Pods
```

but not:

```text
Delete Pods
```

This follows the principle of giving workloads only the permissions they actually need.

---

# 6. Kubernetes Permissions vs AWS Permissions

When working with EKS, it is important not to confuse Kubernetes RBAC permissions with AWS IAM permissions.

### Kubernetes permissions

A ServiceAccount can be associated with Kubernetes RBAC resources such as:

```text
ServiceAccount
      ↓
Role / ClusterRole
      ↓
RoleBinding / ClusterRoleBinding
      ↓
Kubernetes API permissions
```

These permissions determine what the workload can do **inside Kubernetes**.

For example:

> "This application can read Pods."

---

### AWS permissions

In Amazon EKS, a Kubernetes ServiceAccount can also be associated with an AWS IAM role using mechanisms such as **IAM Roles for Service Accounts (IRSA)**.

Conceptually:

```text
Pod
 ↓
Kubernetes ServiceAccount
 ↓
AWS IAM Role
 ↓
AWS permissions
 ↓
S3 / Secrets Manager / DynamoDB / etc.
```

For example:

> "This application can read objects from a particular S3 bucket."

This allows workloads to access AWS services without putting long-lived AWS access keys directly inside the container.

---

# 7. Why This Is Useful

Imagine an application needs to read files from an S3 bucket.

A poor approach would be to put AWS credentials directly into the container:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

A better approach is to give the workload an identity that can assume an appropriate IAM role.

Conceptually:

```text
Application
    ↓
Pod
    ↓
ServiceAccount
    ↓
IAM Role
    ↓
S3 permissions
```

The application can then access the AWS resource according to the permissions attached to the IAM role.

---

# 8. `automountServiceAccountToken`

The Helm chart's `values.yaml` contains:

```yaml
serviceAccount:
  automount: true
```

This controls whether the ServiceAccount's credentials/token are automatically made available to the Pod.

When a workload needs to communicate with the Kubernetes API using its ServiceAccount identity, the associated credentials can be made available inside the Pod.

However, if an application does not need to communicate with Kubernetes, automatically providing credentials may be unnecessary.

Therefore, in security-conscious deployments, this setting should be considered carefully.

---

# 9. Why the Deployment References a ServiceAccount

The Deployment is responsible for managing the application's Pods.

The ServiceAccount is responsible for providing an identity.

Therefore, they have separate responsibilities:

```text
Deployment
   ↓
Manages Pods

ServiceAccount
   ↓
Provides workload identity
```

The Deployment connects them using:

```yaml
serviceAccountName:
```

This is another example of Kubernetes separating responsibilities between different resources.

---

# 10. Complete Mental Model

The overall relationship can be visualized as:

```text
                    Deployment
                         │
                         │ manages
                         ▼
                       Pod
                         │
                         │ uses
                         ▼
                  ServiceAccount
                    /         \
                   /           \
                  ▼             ▼
             Kubernetes       AWS IAM
                RBAC            Role
                  │               │
                  ▼               ▼
          Kubernetes API      AWS Services
                             S3, Secrets
                             DynamoDB, etc.
```

The two permission paths are separate:

```text
ServiceAccount
      │
      ├── Kubernetes RBAC
      │       ↓
      │   Kubernetes permissions
      │
      └── EKS IAM integration
              ↓
          AWS permissions
```

---

# Key Takeaways

### ServiceAccount

> Provides an identity for a workload running inside Kubernetes.

### `serviceAccountName`

> Specifies which ServiceAccount a Pod should use.

### RBAC

> Determines what the ServiceAccount is allowed to do inside Kubernetes.

### EKS / IAM

> A ServiceAccount can be associated with an AWS IAM role so that a workload can access AWS services according to the role's permissions.

### `automount`

> Controls whether the ServiceAccount credentials are automatically made available to the Pod.

### Most important distinction

> **Identity and permissions are not the same thing.**

A ServiceAccount answers:

> **"Who is this workload?"**

RBAC or IAM answers:

> **"What is this workload allowed to do?"**

This distinction is especially important when working with Kubernetes and AWS EKS.
