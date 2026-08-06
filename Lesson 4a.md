# The Anatomy of a Kubernetes Deployment

```

apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "my-nginx-app.fullname" . }}
  labels:
    {{- include "my-nginx-app.labels" . | nindent 4 }}
spec:

```

Every Kubernetes resource follows a common structure:

- apiVersion tells Kubernetes which API version understands the resource.
- kind tells Kubernetes what type of resource to create.
- metadata identifies and describes the resource.
- spec defines the desired state or behavior of the resource.

Kubernetes validates the spec based on the kind. This means a Deployment spec cannot be used for a Service, even though both resources share the same top-level structure.

## 🧠 Here's an analogy

Imagine you're filling out an online job application.

Metadata:
- Your name
- Email
- Phone number

This identifies who you are.

Specification:
- Position you're applying for
- Salary expectation
- Availability

This describes what you want.

That's exactly how Kubernetes separates metadata from spec.

### Why do you think Kubernetes separates metadata from spec instead of putting everything under one section?

Metadata provides information about the resource, while the spec defines the desired behavior or desired state of the resource. If it was mixed together, it would be difficult to tell:

- Which fields identify the resource?
- Which fields describe what Kubernetes should create?


### Suppose I changed: `kind: Deployment` to `kind: Service` but left everything else the same. What do you think Kubernetes would do?

Throw an error because you told it to create a Service but the specs are peculiar to a Deployment

Kubernetes doesn't just read YAML. It validates it against the schema for the resource type.

Suppose you're filling out a passport application. If the form asks:

- Name
- Date of Birth
- Nationality

Instead, you enter:

- Car Engine Size
- Horsepower
- Fuel Type

The system won't say: "Close enough." It will reject it because those fields don't belong on a passport application.

Kubernetes behaves the same way.


---


# Labels and Selectors