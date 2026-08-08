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
