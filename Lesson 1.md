# Lesson 1 - Why Helm?

## Why wasn't Kubernetes YAML enough?

Helm was created because it would require a lot of files and yaml lines to deploy an app on kubernetes and manage it across multiple envirfonments

## What problem does Helm solve?

Helm helps you package, configure, and manage Kubernetes applications in a reusable and repeatable way.

## What is a Helm chart?

A Helm chart is a package that contains everything needed to deploy an application on Kubernetes.

When you use Helm, what do you think happens?

Helm creates the Deployment, Service, and Ingress, but it generates them from templates using the values  provided.

This is the stages it undergoes for helm to assist in the deployment of an app
1. You create a chart that be read by Helm. This is why the chart is called Helm chart.
2. Helm reads the chart.
3. Helm replaces all the template placeholders with actual values.
4. Helm produces ordinary Kubernetes YAML.
5. Helm sends that YAML to the Kubernetes API server.
6. Kubernetes creates the Deployment, Service, Ingress, Pods, etc.


## Questions I Asked While Learning.
### If Helm converts the chart, why is it called a Helm chart?*
It's called a Helm chart because it's the package that Helm knows how to process. 
Suppose you have Microsoft Word. You create a document (report.docx). Microsoft Word opens and edits it. It is calleda Word document because it's the type of file that Microsoft Word understands. The document isn't Microsoft Word itself but it's just a document for Microsoft Word. The same idea applies to Helm.

### Does Helm stay running after deployment?*
It exits because it has done its job which is of rendering the manifest and sending to Kubernetes

### What exactly is inside a Helm chart, and why does Helm need all those files?*
- Template/: The files in templates directory is a template for kubernetes manifest. These are YAML files with placeholders


### Suppose there were no templates/ folder. Imagine your Helm chart only contained values.yaml. Could Helm still generate a Deployment? Why or why not?

Helm cannot generate a deployment manifest because there are no template files to substitute the values in value.yaml.

### What is the relationship between templates/ and values.yaml?
Templates are reusable files that contain place holders that depends on helm to extract values from values.yaml and replace in oredr to produce kubernetes manifest


## Key takeaway

Kubernetes does not understand Helm charts. Kubernetes only understands Kubernetes YAML. Helm converts the chart into regular Kubernetes manifests, and those are what Kubernetes deploys.