# AWS ALB vs NLB with Kubernetes Ingress

When using Kubernetes on Amazon EKS, it is important to understand the difference between an **Application Load Balancer (ALB)** and a **Network Load Balancer (NLB)**.

The choice depends mainly on the **type of traffic** and the **network layer at which routing is required**.


## 1. ALB + Ingress

An **Application Load Balancer (ALB)** operates primarily at **Layer 7 (application layer)** and is designed for HTTP/HTTPS traffic.

It is a natural fit for Kubernetes **Ingress** because Ingress provides HTTP/HTTPS routing rules.

A typical architecture is:

```text
Internet
    ↓
AWS Application Load Balancer (ALB)
    ↓
Ingress rules
    ├── /          → frontend-service
    ├── /api       → backend-service
    └── /auth      → auth-service
```

### ALB is useful when you need:

* HTTP/HTTPS traffic
* Path-based routing
* Host-based routing
* HTTPS/TLS termination
* Application-level routing rules

### Example

Suppose an application has:

```text
example.com/
example.com/api
example.com/payments
```

An ALB can route the requests based on the URL path:

```text
                    Internet
                       ↓
                      ALB
                       ↓
                    Ingress
                  /    |     \
                 /     |      \
                ↓      ↓       ↓
           Frontend  Backend  Payments
           Service   Service   Service
```

Therefore:

```text
https://example.com/
        ↓
Frontend Service

https://example.com/api
        ↓
Backend Service

https://example.com/payments
        ↓
Payments Service
```


## 2. NLB + Kubernetes Service

A **Network Load Balancer (NLB)** operates primarily at **Layer 4** and is designed for network-level traffic such as TCP and UDP.

A common EKS architecture is:

```text
Internet
    ↓
AWS Network Load Balancer (NLB)
    ↓
Kubernetes Service
    ↓
Pods
```

NLB is useful when you need:

* TCP traffic
* UDP traffic
* Network-level load balancing
* High-performance network traffic
* Applications that don't require HTTP/HTTPS path-based routing

For example:

```text
TCP Client
    ↓
NLB
    ↓
Kubernetes Service
    ↓
Pods
```

Unlike an ALB, an NLB does not make routing decisions based on HTTP paths such as:

```text
/api
/payments
/admin
```



## 3. ALB vs NLB

| Feature            | ALB                                                     | NLB                                        |
| ------------------ | ------------------------------------------------------- | ------------------------------------------ |
| Primary layer      | Layer 7                                                 | Layer 4                                    |
| Main traffic       | HTTP/HTTPS                                              | TCP/UDP and network traffic                |
| Path-based routing | Yes                                                     | No                                         |
| Host-based routing | Yes                                                     | No HTTP host routing                       |
| TLS/HTTPS features | Yes                                                     | Supports TLS at network level              |
| Kubernetes Ingress | Natural fit                                             | Not the typical Ingress choice             |
| Kubernetes Service | Can be used through integrations, but Ingress is common | Commonly used with `Service: LoadBalancer` |
| URL-aware routing  | Yes                                                     | No                                         |
| Typical EKS use    | Web applications                                        | Network-level applications                 |


## The Easy Mental Model

Think about the **layer at which the load balancer operates**.

### ALB

```text
ALB
 ↓
Layer 7
 ↓
Understands HTTP/HTTPS
 ↓
Can inspect application-level requests
 ↓
Can route based on host/path
```

For example:

```text
/api       → Backend
/payments  → Payments
/          → Frontend
```

### NLB

```text
NLB
 ↓
Layer 4
 ↓
Works with network connections
 ↓
TCP / UDP
 ↓
Does not perform HTTP path-based routing
```



### ALB + Ingress Architecture

For a typical web application on EKS:

```text
                         Internet
                            │
                            ▼
                     ┌─────────────┐
                     │     ALB     │
                     └──────┬──────┘
                            │
                            ▼
                       Ingress
                     /     |      \
                    /      |       \
                   ▼       ▼        ▼
              Frontend  Backend   Auth
              Service   Service   Service
                   │       │        │
                   ▼       ▼        ▼
                 Pods    Pods      Pods
```

The responsibilities are separated:

```text
ALB
→ External entry point

Ingress
→ HTTP/HTTPS routing rules

Service
→ Stable networking to Pods

Pod
→ Runs the application
```

---

### NLB + Service Architecture

For a network-level application:

```text
                         Internet
                            │
                            ▼
                     ┌─────────────┐
                     │     NLB     │
                     └──────┬──────┘
                            │
                            ▼
                         Service
                            │
                     ┌──────┼──────┐
                     ▼      ▼      ▼
                   Pod A  Pod B  Pod C
```

Here, the Service provides access to the Pods while the NLB provides the external network entry point.

---

## Important Distinction: Ingress ≠ ALB

It is incorrect to think:

> **Ingress = ALB**

A better mental model is:

```text
Ingress
→ Kubernetes resource that defines HTTP/HTTPS routing rules.

Ingress Controller
→ Software that implements those rules.

ALB
→ AWS load-balancing infrastructure that can be used by the AWS Load Balancer Controller to implement Ingress.
```

So on EKS:

```text
Ingress
    ↓
AWS Load Balancer Controller
    ↓
ALB
    ↓
Kubernetes Services
    ↓
Pods
```

The exact behavior depends on the controller and its configuration.



## Important Distinction: NLB ≠ Ingress

Similarly, 
> **"NLB is another type of Ingress."**

A more useful mental model is:

```text
ALB + Ingress
→ Application-level HTTP/HTTPS routing

NLB + Service
→ Network-level TCP/UDP exposure
```

This isn't an absolute rule for every possible AWS/EKS configuration, but it is a strong default mental model.


## When Should I Choose ALB?

Choose **ALB + Ingress** when you're building a web application and need routing such as:

```text
example.com/
example.com/api
example.com/admin
```

or:

```text
www.example.com
api.example.com
admin.example.com
```

Example architecture:

```text
Internet
    ↓
ALB
    ↓
Ingress
    ↓
Services
    ↓
Pods
```


## When Should I Choose NLB?

Choose **NLB** when your application primarily requires network-level load balancing, especially for:

```text
TCP
UDP
```

or when you specifically need NLB characteristics.

Example:

```text
Internet
    ↓
NLB
    ↓
Service
    ↓
Pods
```

---

### Final Mental Model

The simplest way to remember the difference is:

```text
                    AWS EKS
                       │
             ┌─────────┴─────────┐
             │                   │
            ALB                 NLB
             │                   │
        Layer 7              Layer 4
             │                   │
        HTTP/HTTPS          TCP/UDP
             │                   │
          Ingress             Service
             │                   │
          Routing              Pods
```
