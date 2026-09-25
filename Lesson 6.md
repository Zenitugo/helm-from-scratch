# AWS EKS
There are three ways to create kubernetes cluster on AWS and they are:
- Directly on AWS console
- Using Infrastructure as code (Terraform, CloudFormation etc)
- Using `eksctl`


## Create a Cluster
To create a cluster
```bash
eksctl create cluster \
  --name <your-cluster-name> \
  --region <your-region> \
  --node-type <instance-type> \
  --nodes 1 \
  --nodes-min 1 \
  --nodes-max 1
```

```bash
kubectl get nodes
```

```bash
kubectl get pods -A
```

```bash
kubectl get deployment
```

```bash
eksctl delete cluster --name my-nginx-cluster --region eu-central-1
```