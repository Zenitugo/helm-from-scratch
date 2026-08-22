# Lesson 4.4: Liveness and Readiness Probe
Imagine we have:
```
Pod
└── Nginx container
```

Kubernetes can see that the container is running.

- But here's the problem: **A running container doesn't necessarily mean a healthy application.**

For example, imagine your application gets stuck because of a bug.

The container/process might still technically be running:
```
Container: RUNNING ✅
Application: STUCK ❌
```
From Kubernetes' perspective, simply seeing the process running isn't enough to know whether the application is functioning correctly.

So Kubernetes needs a way to ask the application about its health. That's where the probe comes in

## What is a Probe?
A probe is essentially a health check that Kubernetes performs against a container.

Think of Kubernetes asking: **"Are you okay?"**

The application needs to respond in a way that tells Kubernetes whether it is healthy.

There are different types of probes, but the two we're focusing on now are:

- Liveness Probe
- Readiness Probe

## Liveness Probe
Liveness = Is the application alive and functioning?

The liveness probe helps Kubernetes determine whether the application inside the container is still healthy enough to continue running.

Suppose:
```
Pod
└── Application
       ❌ Frozen
```
The container may still technically be running, but the application isn't responding properly. The liveness probe can detect that.

If the liveness check repeatedly fails, Kubernetes can restart the container.

So in summary:
```
Liveness Probe
      │
      ▼
"Is this application still alive?"
      │
   ┌──┴──┐
   │     │
  YES    NO
   │     │
   ▼     ▼
Keep   Restart
running container
```

## Readiness Probe
Readiness probe = "Is this application ready to receive traffic?"

Imagine your application has just started. The container is running:
```
Container: RUNNING ✅
```

But your application is still:
- Loading configuration
- Connecting to the database
- Loading data
- Initializing something
So:
```
Container: RUNNING ✅
Application: NOT READY ❌
```

Should Kubernetes immediately send users' requests to it? No.

That's what the readiness probe helps prevent.

If the readiness check fails, Kubernetes can keep the Pod running but remove it from the Service's available endpoints, so it doesn't receive normal Service traffic.


## Liveness & Readiness Probe in Deploment.yaml 

You have:
```
livenessProbe:
  httpGet:
    path: /
    port: http
```
This tells Kubernetes: `"Check the / endpoint of this container using HTTP."`

And:
```
readinessProbe:
  httpGet:
    path: /
    port: http
```
means: `"Also check the / endpoint to determine whether this Pod is ready to receive traffic."`

So Kubernetes might make an HTTP request like:

GET /

If the application responds successfully, the probe passes.

If it repeatedly fails, Kubernetes takes the appropriate action depending on which probe failed.

---

## How Do You know an error is a Liveness or Readiness Probe Error?


| What you observe                           | What it might indicate                 |
| ------------------------------------------ | -------------------------------------- |
| `Liveness probe failed`                    | Liveness problem                       |
| `Readiness probe failed`                   | Readiness problem                      |
| `RESTARTS` keeps increasing                | Often a liveness/container crash issue |
| Pod is `Running` but `0/1` Ready           | Often readiness failure                |
| Traffic stops reaching one Pod             | Could be readiness failure             |
| Container repeatedly gets killed/restarted | Could be liveness failure              |

### What Liveness Probe Errors Look Like
```
Liveness probe failed: HTTP probe failed with statuscode: 500
```
or
```
Liveness probe failed: Get "http://10.x.x.x:80/": context deadline exceeded
```
If the failure continues, you'll typically see evidence that Kubernetes restarted the container, such as:
```
Killing container nginx
```
And you can check:
```
kubectl get pod <pod-name>
```
Something like this can show up
```
NAME       READY   STATUS    RESTARTS
my-app     1/1     Running   3
```
The `RESTARTS` count is an important clue.

###  What Readiness Probe Errors Look Like
```
Readiness probe failed: HTTP probe failed with statuscode: 503
```
or
```
Readiness probe failed: connection refused
```
The important difference is what happens afterward. The container can remain running:
```
STATUS      Running
RESTARTS    0
```
but the pod may show:
```
READY       0/1
```
instead of:
```
READY       1/1
```


## One Command to Remember
When a Pod is behaving strangely:
```
kubectl describe pod <pod-name>
```
Look at:
```
Events:
```
That's often where Kubernetes tells you what happened. 

And for a deeper look at the container's restart/crash history:
```
kubectl logs <pod-name> --previous
```
The --previous option is particularly useful when the container has already restarted, and you want the logs from the previous container instance.









