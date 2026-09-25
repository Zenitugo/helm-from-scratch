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





## Understanding Semantic Versioning (SemVer)

Semantic Versioning (SemVer) is a versioning system used to communicate the nature and significance of changes made to software.

A version number follows this format:

```text
MAJOR.MINOR.PATCH
```

For example:

```text
1.2.3
```

Each component represents a different type of change.

### 1. MAJOR Version

The major version increases when changes introduce incompatibilities with previous versions.

For example:

```text
1.2.3 → 2.0.0
```

A major version increase indicates that users may need to modify their configurations, integrations, or usage of the application to work with the new version.

When the major version increases, the minor and patch components reset to zero.

### 2. MINOR Version

The minor version increases when new features are introduced in a backward-compatible manner.

For example:

```text
1.2.3 → 1.3.0
```

This means new functionality has been added without breaking existing functionality.

When the minor version increases, the patch component resets to zero because a new minor-version series has begun.

### 3. PATCH Version

The patch version increases when backward-compatible bug fixes or other small corrections are made.

For example:

```text
1.2.3 → 1.2.4
```

Only the patch component increases.

---

## Why Do Version Numbers Reset to Zero?

Version numbers follow a hierarchical structure:

```text
MAJOR.MINOR.PATCH
```

Each component tracks changes within the scope defined by the components before it.

When a higher-level component increases, the lower-level components reset to zero.

### Example 1: Increasing the Patch Version

Starting version:

```text
1.2.3
```

After fixing a bug:

```text
1.2.4
```

The patch increases because the change is a backward-compatible bug fix.

### Example 2: Increasing the Minor Version

Starting version:

```text
1.2.3
```

After adding a backward-compatible feature:

```text
1.3.0
```

The minor version increases from 2 to 3, and the patch resets to zero.

The new minor version begins its own patch sequence:

```text
1.3.0 → 1.3.1 → 1.3.2
```

### Example 3: Increasing the Major Version

Starting version:

```text
1.3.2
```

After introducing a breaking change:

```text
2.0.0
```

The major version increases from 1 to 2, while the minor and patch components reset to zero.

The new major version begins its own minor and patch sequences:

```text
2.0.0 → 2.0.1 → 2.1.0 → 2.1.1
```

### Versioning Summary

| Type of Change | Example |
|---|---|
| Backward-compatible bug fix | `1.2.3 → 1.2.4` |
| Backward-compatible new feature | `1.2.3 → 1.3.0` |
| Breaking change | `1.2.3 → 2.0.0` |

The general convention is to increase the component that corresponds to the most significant type of change and reset the components to its right.

---

## Understanding Helm Chart Versioning

In Helm, the `Chart.yaml` file contains two important version fields:

```yaml
version: 0.1.0
appVersion: "1.0.0"
```

These fields serve different purposes.

### 1. `version`

The `version` field represents the version of the Helm chart itself.

It changes when the chart's templates, configuration structure, or other chart components are updated.

For example:

```yaml
version: 0.1.0
```

If a backward-compatible chart improvement is introduced, the chart version might become:

```yaml
version: 0.2.0
```

If a chart bug is fixed without changing its compatibility, the patch version might increase:

```yaml
version: 0.1.1
```

If a change breaks compatibility with existing chart configurations, the major version may increase.

### 2. `appVersion`

The `appVersion` field identifies the application version that the chart is associated with.

For example:

```yaml
appVersion: "1.0.0"
```

This field is informational metadata. Helm does not automatically use it to determine the Docker image tag or application deployment version.

The image tag must be configured separately in the chart's templates or values.

### Example Chart.yaml

```yaml
apiVersion: v2
name: orderly-platform
description: A Helm chart for Orderly Platform
type: application
version: 0.1.0
appVersion: "1.0.0"
```

In this example:

- `version: 0.1.0` identifies the Helm chart version.
- `appVersion: "1.0.0"` identifies the application version associated with the chart.

The chart version and application version can change independently because they track different things.