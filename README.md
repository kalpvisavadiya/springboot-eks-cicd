# Spring Boot + Amazon EKS CI/CD

A sample Spring Boot 3 / Java 17 application packaged as a Docker image, pushed to Amazon ECR, and deployed to Amazon EKS with Helm through GitHub Actions.

## Repository contents

- `src/` — Spring Boot REST app and tests
- `Dockerfile` — container image
- `helm/springboot-app/` — Kubernetes Helm chart
- `.github/workflows/ci-cd.yml` — test, build, push, and deploy pipeline

## GitHub configuration

In **Settings → Secrets and variables → Actions**, add repository secrets:

- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`

Add repository variables:

- `AWS_REGION` = `ap-south-1`
- `EKS_CLUSTER_NAME` = the exact cluster name created by your Terraform repository (default starter name: `devops-eks-dev-cluster`)
- `ECR_REPOSITORY` = `devops-app`

Use an IAM principal with the required ECR permissions and EKS cluster access. The IAM principal used by Actions must be authorized by the cluster's EKS access configuration/Kubernetes RBAC. Do not use AWS root account keys. Prefer least-privilege IAM permissions and rotate access keys regularly.

## Deploy flow

A push to `main` or manual workflow dispatch runs Maven tests, builds the JAR and Docker image, pushes the image to ECR with the Git commit SHA as its tag, configures `kubectl`, then deploys using Helm.

Prerequisites: the EKS cluster and its worker nodes must already exist and be healthy; the cluster must be reachable by the GitHub Actions runner; and the AWS identity must be able to create/use the ECR repository and deploy to the cluster.

## Run locally

```bash
mvn clean verify
mvn spring-boot:run
```

Then open `http://localhost:8080/` or `http://localhost:8080/health`.

## Costs and security

The Kubernetes Service defaults to `LoadBalancer`, which can create an AWS load balancer and incur charges. EKS, EC2 worker nodes, NAT gateways, ECR storage, and load balancers may incur charges. Avoid committing credentials or `.env` files.
