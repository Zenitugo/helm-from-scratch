# Liveness and Readiness Probe
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
