# Understanding `Service.yaml`

Pod IP addresses are ephemeral, meaning they can change when Pods are recreated. This is why Kubernetes Services are important: they provide a stable network endpoint for accessing a group of Pods without clients needing to know or track the individual Pod IP addresses.

```text
Pod IPs → ephemeral
    ↓
Pods can be replaced
    ↓
Their IPs can change
    ↓
Service provides stable endpoint
    ↓
Service routes traffic to matching Pods
```

## Kubernetes Service
A Kubernetes Service is an abstraction that provides a stable network endpoint for accessing a group of Pods. Instead of:

```text
Client
  ↓
Pod A
```
we have:
```text
Client
  ↓
Service
  ↓
Pods
```
For example:
```text
                 Service
                    │
          ┌─────────┼─────────┐
          ↓         ↓         ↓
        Pod A     Pod B     Pod C
```
Pods can be created, destroyed, or replaced, while the Service remains the stable entry point

## Service Types

### Cluster IP
This is the default kubernetes service type

When you create:
```yaml
spec:
  type: ClusterIP
```
kubernetes gives the service a stable virtual IP address

ClusterIP is primarily for communication inside the Kubernetes cluster. The frontend can communicate with the backend through its Service. But an external user on the Internet doesn't normally connect directly to the ClusterIP.

For external access, we can use things such as:
```text
Internet
    ↓
LoadBalancer / Ingress
    ↓
Service
    ↓
Pods
```
