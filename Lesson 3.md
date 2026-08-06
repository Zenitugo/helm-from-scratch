# Values.yaml
Today was one of those learning sessions that changed how I think about Helm.

I didn't just learn what `values.yaml` is—I learned *why* it exists.

Some of my biggest takeaways:
* `values.yaml` isn't just a configuration file; it's the **control panel** for the templates. Each major section in `values.yaml` maps to one or more templates in the `templates/` directory. Instead of modifying the templates directly, I can usually change the values in `values.yaml`, and Helm will render the templates with those new values.

For example:

  -  `replicaCount` → `deployment.yaml`
  -  `service` → `service.yaml`
  -  `ingress` → `ingress.yaml`
  -  `autoscaling` → `hpa.yaml`


Instead of editing templates for different environments, you usually keep the templates the same and change the values.

One of my biggest "aha!" moments came when I realized this follows the same pattern as **Terraform modules**.

In Terraform:

- Reusable modules + environment-specific `tfvars` files = infrastructure.

In Helm:

- Reusable templates + environment-specific values files = Kubernetes manifests.

Another interesting discussion was about organizing values files. My instinct was to keep them in a dedicated `values/` directory:

```text
values/
├── dev.yaml
├── stage.yaml
└── prod.yaml
```

I learned that Helm doesn't require all values files to live in the root of the chart. You can organize them however you like and point Helm to the appropriate file using the `-f` flag. The important lesson is that engineering is often about trade-offs between **clarity** and **avoiding duplication**, not about finding a single "correct" structure.

Today's lesson reminded me that learning a new tool isn't just about memorizing commands, it's about understanding the design principles behind it. The more I learn Helm, the more I see familiar patterns across Terraform, Ansible, and other DevOps tools.

---

