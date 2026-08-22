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
The Kubernetes service tyoes are:
- Cluster IP
- NodePort
- LoadBalancer

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

### NodePort
if you want traffic from outside the cluster to reach your Service, one option to achieve this is to use a node port
```yaml
type: NodePort
```
A NodePort exposes a Service on a specific port on each Kubernetes Node. It is called node port because the Service is exposed through a port on the Node.

For instance a node can be seen as:
```text
EKS Cluster
│
├── Node 1 → EC2
├── Node 2 → EC2
└── Node 3 → EC2
```
kubernetes opens port on those node/ec2

```text
Node 1
10.0.1.10:30080

Node 2
10.0.2.10:30080

Node 3
10.0.3.10:30080
```

By default kubernetes node ports are allocated from `30000–32767`

Suppose we have:
```yaml
ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080
```
We now have different ports
```text
NodePort      → 30080
Service port  → 80
Target port   → 8080
```
Traffic can flow like this:
```text
External Client
      │
      │ :30080
      ▼
   Kubernetes Node
      │
      ▼
   Service :80
      │
      ▼
   Pod :8080
      │
      ▼
 Application
```
NodePort isn't usually the most convenient way to expose production applications directly to the Internet. This is where Loadbalancer becomes useful.


### LoadBalancer
A service can use:
```yaml
type: LoadBalancer
```
On a cloud platform such as AWS, Kubernetes can integrate with cloud load-balancing infrastructure.
```text
Internet
   │
   ▼
AWS Load Balancer
   │
   ▼
Kubernetes Service
   │
   ▼
Pods
```

## Service Port
The service port are:
- port
- container port
- target port

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

If you don't explicitly specify targetPort, Kubernetes normally defaults it to the same value as port.

In Summary
1. `port` belongs to Service. It means
    > The Service accepts traffic on port 80. 

2. `target port` belongs to service. It means
    > Send that traffic to port 8080 on the selected Pods.

3. `container port` belongs to Deployment. It means
    >  Belongs to the Pod's container definition.


