# Extra Things I Learnt while Creating Orderly

## Understanding Version Numbers in Helm

When working with Helm charts and applications, version numbers help track changes and identify different releases.

There are two important version fields in the `Chart.yaml` file:

- `version`: Represents the version of the Helm chart.
- `appVersion`: Represents the version of the application associated with the chart.

### 1. Semantic Versioning (SemVer)

A common convention for version numbers is Semantic Versioning, which follows this format:

`MAJOR.MINOR.PATCH`

For example: `1.2.3`

| Component | Meaning |
|---|---|
| MAJOR | Introduces changes that break backward compatibility. |
| MINOR | Adds new features while maintaining backward compatibility. |
| PATCH | Fixes bugs without introducing new features or breaking compatibility. |

### 2. Examples of Version Progression

Imagine Orderly Platform is being developed and released over time.

| Version | Description |
|---|---|
| `1.0.0` | First stable release |
| `1.0.1` | Bug fix |
| `1.1.0` | New feature added |
| `2.0.0` | Major change that breaks backward compatibility |

These are conventions that help teams communicate changes between releases.

### 3. Understanding `version` and `appVersion`

In a Helm chart's `Chart.yaml` file:

```yaml
apiVersion: v2
name: orderly-platform
description: A Helm chart for Orderly Platform
type: application
version: 0.1.0
appVersion: "1.0.0"