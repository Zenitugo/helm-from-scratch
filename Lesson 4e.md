## How Kubernetes Probes Check Application Health

Kubernetes uses **liveness** and **readiness probes** to check the health of an application running inside a container.

In my Helm chart, the probes use `httpGet`:

```yaml
livenessProbe:
  httpGet:
    path: /
    port: http

readinessProbe:
  httpGet:
    path: /
    port: http
```

The `httpGet` tells Kubernetes to make an HTTP request to the specified endpoint.

For example:

```yaml
path: /
```

means Kubernetes checks:

```text
GET /
```

If the application responds successfully, the probe passes. HTTP responses in the **200–399 range** are generally considered successful for an HTTP probe. Responses such as `404`, `500`, or `503` cause the probe to fail.

The `port` tells Kubernetes which port on the container it should use for the health check.

In my chart:

```yaml
ports:
  - name: http
    containerPort: 80
```

Therefore:

```yaml
port: http
```

refers to the named container port `http`, which corresponds to port `80`.

The port can also be specified directly:

```yaml
port: 80
```

If the application uses HTTPS on port `443`, the probe should specify HTTPS:

```yaml
livenessProbe:
  httpGet:
    path: /
    port: 443
    scheme: HTTPS
```

Therefore, the basic mental model is:

```text
Kubernetes Probe
      ↓
HTTP request
      ↓
Application endpoint
      ↓
Specified container port
      ↓
Successful response?
      ↓
Yes → Probe passes
No  → Probe fails
```

The important thing I learned is that the probe is checking the **application endpoint inside the container**, not simply checking whether the container process exists.

---

# Probe Timing and Failure Settings
Consider this: Your application takes 30 seconds to start. If Kubernetes starts checking the liveness probe immediately, it might get:
```
Application starting...
       ↓
GET / → ❌
       ↓
Kubernetes: "Unhealthy!"
```

But the application isn't actually broken. It's simply not ready yet. This is why Kubernetes gives us timing controls:
```yaml
initialDelaySeconds:
periodSeconds:
timeoutSeconds:
failureThreshold:
successThreshold:
```

1. **initialDelaySeconds**
```yaml
initialDelaySeconds: 30
```
This means: `Wait 30 seconds after the container starts before performing the first probe.`

So:
```
Container starts
      ↓
Wait 30 seconds
      ↓
First health check
```
This is useful for applications that take time to initialize. For example:
```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
```
Kubernetes won't immediately start judging the application's liveness.


2. **periodSeconds**
```yaml
periodSeconds: 10
```
This means: `Perform the probe every 10 seconds.` So after the initial delay:
```
30s → Check
40s → Check
50s → Check
60s → Check
70s → Check
```
This refers to the frequency of the health check.


3. **timeoutSeconds**
```yaml
timeoutSeconds: 2
```
This means: `Give the application 2 seconds to respond to each probe.`

Imagine Kubernetes sends:
```
GET /health
```
If the application responds within 2 seconds:
```
Response → ✅
```
If it doesn't respond within 2 seconds:
```
Timeout → ❌
```
This prevents Kubernetes from waiting indefinitely for a health check.


4. **failureThreshold**

This one is particularly important.
```
failureThreshold: 3
```
It means: `How many consecutive probe failures should occur before Kubernetes considers the probe failed?`

Suppose:
```
Check 1 → ❌
Check 2 → ❌
Check 3 → ❌
```
Now you've reached:
```
failureThreshold = 3
```
So Kubernetes takes the appropriate action. For a liveness probe, that can mean restarting the container. For a readiness probe, the Pod remains not ready.


5. **successThreshold**

This one works in the opposite direction.
```
successThreshold: 2
```
It means: `How many consecutive successful probes are required before Kubernetes considers the probe successful again?`

For example:
```
Failure
   ↓
Check → ✅
Check → ✅
   ↓
Ready
```
The application has to successfully pass the required number of checks before Kubernetes considers it healthy/ready again.

Important: for **liveness** and startup probes, successThreshold must be 1. Values greater than 1 are used with **readiness** probes.
