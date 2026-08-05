# 📚 Lesson 2: Creating Your First Helm Chart

**Step 1: Create the chart**
```
helm create my-nginx-app

```
```
tree my-nginx-app

zenitugo@Zenitugo:~/helm-from-scratch$ tree my-nginx-app/
my-nginx-app/
├── Chart.yaml
├── charts
├── templates
│   ├── NOTES.txt
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── service.yaml
│   ├── serviceaccount.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml

3 directories, 10 files

```

The files are divided into:
- Metadata (information about the chart)
- Configuration (the values)
- Templates (the blueprints)

Chart.yaml → Contains the metadata about the Helm chart.

values.yaml → Contains the configuration values that Helm uses during rendering.

templates/ → Contains the Kubernetes manifest blueprints (templates) that Helm renders into actual Kubernetes manifests.

## Chart.yaml

### aiVersion: v2 me
It tells Helm which chart specification this file follows.

### name: my-nginx-app 
The name of the chart

### description

This is simply documentation. It helps humans understand what the chart is for.


### Chart Types

Helm supports two types of charts:

* **Application Chart**: A deployable chart that contains Kubernetes templates and can be installed on a Kubernetes cluster.
* **Library Chart**: A non-deployable chart that contains reusable helper templates and functions for other charts to use. It is similar to a Python library or a shared Ansible role—it provides reusable functionality rather than running on its own.



### `version` vs `appVersion`

These two fields represent different things:

* **`version`** – The version of the **Helm chart**. Increment this whenever you make changes to the chart itself, such as modifying templates, updating `values.yaml`, or changing chart metadata.

* **`appVersion`** – The version of the **application** being deployed. Update this when the application's version changes (for example, deploying Nginx 1.27 instead of 1.26).

### Example Scenarios

* **Fixed a typo or modified a Helm template**

  * ✅ Update: `version`
  * ❌ No change: `appVersion`

* **Upgraded the application from Nginx 1.26 to Nginx 1.27**

  * ✅ Update: `appVersion`
  * ❌ No change: `version`

* **Changed both the Helm chart and the application version**

  * ✅ Update both `version` and `appVersion`.

---

## Key Takeaway

A Helm chart has its own lifecycle, separate from the application it deploys. The chart version tracks changes to the deployment package, while the application version tracks changes to the software being deployed.
