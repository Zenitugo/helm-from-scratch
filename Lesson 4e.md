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

---

# Temporary Failures, Consecutive Failures, and Startup Probes

## Temporary vs. Consecutive Probe Failures

Kubernetes does not necessarily treat a single failed health check as a serious problem.

The `failureThreshold` determines how many **consecutive** probe failures must occur before Kubernetes considers the probe to have failed.

For example:

```yaml
failureThreshold: 3
```

means Kubernetes needs **three consecutive failures**.

```text
Probe 1 → ❌  Failure count: 1
Probe 2 → ❌  Failure count: 2
Probe 3 → ❌  Failure count: 3
                     ↓
              Threshold reached
```

However, if a successful probe occurs, the consecutive failure count is reset.

```text
Probe 1 → ❌  Failure count: 1
Probe 2 → ❌  Failure count: 2
Probe 3 → ✅  Failure count resets
Probe 4 → ❌  Failure count: 1
Probe 5 → ❌  Failure count: 2
Probe 6 → ❌  Failure count: 3
                     ↓
              Threshold reached
```

This allows Kubernetes to tolerate **temporary problems** without immediately taking action.

For example, a brief network delay or temporary application slowdown might cause one failed probe. If the next probe succeeds, Kubernetes does not treat the failure as a persistent health problem.

The important distinction is:

> **A temporary failure is an isolated failure. A consecutive failure is a series of failures without a successful probe in between.**



## Startup Probe

A **startup probe** is useful for applications that take a long time to initialize.

Without a startup probe, a slow-starting application could be mistaken for an unhealthy application by its liveness probe.

For example:

```text
Container starts
      ↓
Application is still initializing
      ↓
Liveness probe starts too early
      ↓
Probe fails repeatedly
      ↓
Container is restarted
      ↓
Application starts again
      ↓
The cycle can repeat
```

A startup probe gives the application time to finish starting before Kubernetes begins relying on the liveness and readiness checks.

For example:

```yaml
startupProbe:
  httpGet:
    path: /health
    port: 8080
```

The startup probe essentially asks:

> **"Has the application finished starting successfully?"**

Once the startup probe succeeds, Kubernetes can begin applying the liveness and readiness probes normally.

### The three probes together

```text
Startup Probe
     ↓
"Has the application started?"
     ↓
       YES
        │
        ├──────────────┐
        ▼              ▼
   Liveness         Readiness
"Still healthy?"  "Ready for traffic?"
```

The simple mental model is:

* **Startup** → Gives a slow-starting application time to initialize.
* **Liveness** → Determines whether the application should continue running.
* **Readiness** → Determines whether the application should receive traffic.


Important: for **liveness** and startup probes, successThreshold must be 1. Values greater than 1 are used with **readiness** probes.
