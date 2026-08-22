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

### Service Port
nginx chart has
```yaml
service:
  type: ClusterIP
  port: 80
```
The Service's port is the port on which the Service accepts traffic. For example:
```yaml
Service
ClusterIP
Port: 80
```
Another application inside the cluster can communicate with the Service on port 80.

### Container Port
In our deployment file we have:
```yaml
containers:
  - name: nginx
    ports:
      - name: http
        containerPort: 80
```
This describes the port associated with the container/application. 

This means we can have same port number for both service port and container port
```yaml
Service
port: 80

       ↓

Pod
containerPort: 80
```
and also different port number for both service port and container port
```yaml
Service
port: 80

       ↓

Pod
containerPort: 8080
```
When this happens, this is when target port is introduced


### Target Port
Suppose your app listens on `8080` and you want kubernetes service to expose `80`, you can write:
```yaml
spec:
  type: ClusterIP
  port: 80
  targetPort: 8080
```
Trrafic flow becomes:
```text
Client
   │
   │ :80
   ▼
Service
   │
   │ targetPort: 8080
   ▼
Pod
   │
   ▼
Container :8080
```
So
> `port` = the Service's port.
>
> `targetPort` = the port on the Pod/container where the traffic should ultimately go.

In Summary
1. `port` belongs to Service. It means
    > The Service accepts traffic on port 80. 

2. `target port` belongs to service. It means
    > Send that traffic to port 8080 on the selected Pods.

3. `container port` belongs to Deployment. It means
    >  Belongs to the Pod's container definition.