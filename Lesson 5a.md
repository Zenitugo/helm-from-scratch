# Kubernetes Ingress
A Kubernetes Ingress is an API resource used to define rules for routing external HTTP/HTTPS traffic to different Kubernetes Services.

Instead of exposing every application with its own external endpoint, Ingress can provide a single entry point and route traffic to different Services based on rules such as:

- Hostnames
- URL paths
- TLS/HTTPS configuration

A simplified architecture is:
```text
Internet
    ↓
Load Balancer
    ↓
Ingress
    ├── /          → Frontend Service
    ├── /api       → Backend Service
    └── /payments  → Payments Service
```

> *Ingress routes traffic to Services. It does not normally route traffic directly to Pods.*

**Ingress allows us to have a single entry point**
```text
                 Internet
                    ↓
              Load Balancer
                    ↓
                 Ingress
              /     |      \
             /      |       \
            ↓       ↓        ↓
       Frontend   Backend   Payments
       Service    Service    Service
```

**Ingress is primarily concerned with HTTP/HTTPS routing**
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

**Ingress requires an ingress controller**. An **Ingress Controller** is responsible for actually implementing the Ingress rules. It can also be seen as the actual software running in the cluster that watches Ingress resources and configures the traffic-routing system accordingly.

Examples include controllers based on:
- AWS Load Balancer Controller
- NGINX
- Traefik
- HAProxy

Therefore:

The relationship is:
```text
Ingress Resource
       ↓
Ingress Controller
       ↓
Actual routing configuration
       ↓
Services
       ↓
Pods
```
The Ingress resource essentially says:
   > These are the routing rules I want.

The Ingress Controller says:
  > I will implement those rules.


Example of a basic ingress
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app
spec:
  rules:
    - host: example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80

          - path: /api
            pathType: Prefix
            backend:
              service:
                name: backend-service
                port:
                  number: 80
```

This creates two routing rules.

> Request 1:
> https://example.com/
>
> goes to:
>
> frontend-service:80


> Request 2:
> https://example.com/api
> 
> goes to:
> 
> backend-service:80

The Services then route the traffic to their respective Pods.



**Ingress path based routing allows multiple apps to share one external endpoint**
```text
example.com/
        ↓
Frontend Service

example.com/api
        ↓
Backend Service

example.com/auth
        ↓
Authentication Service
```
```text
                 example.com
                      ↓
                   Ingress
                 /    |     \
                /     |      \
               ↓      ↓       ↓
          Frontend  Backend   Auth
           Service  Service   Service
```

**Ingress also use host based routing**
```text
app.example.com
       ↓
Frontend Service

api.example.com
       ↓
Backend Service

admin.example.com
       ↓
Admin Service
```

```text
                     Ingress
                    /   |    \
                   /    |     \
                  ↓     ↓      ↓
          app.example   api.example   admin.example
                ↓             ↓              ↓
           Frontend        Backend          Admin
           Service         Service          Service
```

**Ingress path includes a `pathType`**
For example
```yaml
pathType: Prefix
```

```yaml
path: /api
pathType: Prefix
```
requests beginning with **/api** can match the rule.

For example:
```text
/api
/api/users
/api/orders
/api/products
```
can all match the /api prefix rule, subject to the controller's routing behavior.

Another path type is:
```yaml
pathType: Exact
```
It matches the specified path exactly.

for example:
```yaml
path: /api
pathType: Exact
```
is intended to match /api rather than acting as a general /api prefix rule.



