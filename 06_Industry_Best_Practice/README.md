# DevBoard CI/CD — Industry Best Practice

## 1. Main Idea

### CI — GitHub Actions

Use GitHub Actions mainly for:

```text
Code
 ↓
Lint
 ↓
Test
 ↓
Build Docker Image
 ↓
Push Image
 ↓
Docker Hub / Container Registry
```

### CD — Deployment Tool

The deployment method depends on the infrastructure.

```text
Docker / VM
    → Deployment script / platform tooling

AWS ECS
    → Terraform + AWS deployment mechanisms

Kubernetes
    → Argo CD
```

---

# 2. Industry-Oriented Architecture

```text
Developer
    │
    │ git push
    ▼
GitHub
    │
    ▼
GitHub Actions
    │
    ├── Lint
    ├── Test
    ├── Docker Build
    └── Push Image
           │
           ▼
     Container Registry
      (Docker Hub/ECR)
           │
           ├───────────────┐
           ▼               ▼
        AWS ECS        Kubernetes
           │               │
      Terraform /      Argo CD
      AWS tooling         │
           │               │
           ▼               ▼
       Containers      Kubernetes Pods
```

---

# 3. Why Separate CI and CD?

A common industry approach is:

```text
CI
 ↓
Build + Test + Image
 ↓
Registry
 ↓
CD system
 ↓
Deployment
```

This gives the deployment system responsibility for deployment instead of making GitHub Actions directly control production infrastructure.

Benefits include:

* Better separation of responsibilities
* Easier deployment management
* GitOps support with Kubernetes
* Better audit/history of deployments
* Infrastructure managed separately
* Easier rollback strategies

---

# 4. GitHub Actions CD — Learning/Demo

For a beginner/demo project, you can use:

```text
GitHub Actions
      ↓
Self-hosted Runner
      ↓
Docker Hub
      ↓
docker compose pull
      ↓
docker compose up -d
      ↓
EC2
```

Example:

```yaml
runs-on: self-hosted
```

The self-hosted runner is installed on your own server.

### Demo Flow

```text
GitHub Actions
      ↓
Self-hosted Runner
      ↓
Docker Login
      ↓
docker compose pull
      ↓
docker compose up -d
```

This is a **good learning exercise** for understanding:

* Self-hosted runners
* Docker deployment
* EC2
* Docker Compose
* Secrets
* CI/CD workflow dependencies

---

# 5. Important Learning Note

Do not confuse:

```text
"GitHub Actions CD is useless"
```

with:

```text
"GitHub Actions CD is not the only deployment architecture."
```

GitHub Actions can perform deployments, including through self-hosted runners.

For your **demo project**, it is perfectly useful to learn:

```text
GitHub Actions
      ↓
Self-hosted Runner
      ↓
EC2
      ↓
Docker Compose
```

But while preparing for industry DevOps/SRE work, also learn dedicated deployment approaches.

---

# 6. AWS ECS — Terraform

For ECS, learn Infrastructure as Code with Terraform.

Terraform can manage infrastructure such as:

```text
VPC
 │
 ├── Subnets
 ├── Security Groups
 │
 └── ECS
      ├── Cluster
      ├── Task Definition
      └── Service
```

Basic idea:

```text
Terraform
    ↓
AWS Infrastructure
    ↓
ECS
    ↓
Containers
```

GitHub Actions can still handle CI:

```text
GitHub Actions
    ↓
Test
    ↓
Build Image
    ↓
Push to ECR
```

Terraform manages the infrastructure separately.

---

# 7. Kubernetes — Argo CD

For Kubernetes, learn **Argo CD** for GitOps-style continuous delivery.

Architecture:

```text
Developer
    ↓
GitHub
    ↓
GitHub Actions
    ↓
Build Docker Image
    ↓
Push Image
    ↓
Update Kubernetes manifests
    ↓
Git Repository
    ↓
Argo CD
    ↓
Kubernetes Cluster
    ↓
Pods
```

Argo CD watches the desired configuration in Git and synchronizes the Kubernetes cluster with it.

---

# 8. Three Deployment Models to Learn

## Model 1 — Demo Project

```text
GitHub Actions
      ↓
Self-hosted Runner
      ↓
EC2
      ↓
Docker Compose
```

**Purpose:** Learn GitHub Actions deployment and self-hosted runners.

---

## Model 2 — AWS ECS

```text
GitHub Actions
      ↓
Build + Test + Push
      ↓
ECR
      ↓
AWS ECS
      ↑
Terraform manages infrastructure
```

**Purpose:** Learn AWS container deployment + Infrastructure as Code.

---

## Model 3 — Kubernetes

```text
GitHub Actions
      ↓
Build + Test + Push
      ↓
Container Registry
      ↓
Git Repository
      ↓
Argo CD
      ↓
Kubernetes
```

**Purpose:** Learn GitOps and Kubernetes CD.

---

# 9. What You Should Learn

```text
LEVEL 1 — Beginner
───────────────────
GitHub Actions CI
      ↓
Docker
      ↓
Docker Hub
      ↓
Self-hosted Runner
      ↓
EC2 + Docker Compose


LEVEL 2 — AWS
───────────────────
GitHub Actions
      ↓
Docker
      ↓
Amazon ECR
      ↓
Terraform
      ↓
AWS ECS


LEVEL 3 — Kubernetes
───────────────────
GitHub Actions
      ↓
Docker
      ↓
Container Registry
      ↓
GitOps Repository
      ↓
Argo CD
      ↓
Kubernetes / EKS
```

---

# 10. Simple Interview Explanation

> **"I use GitHub Actions for CI to lint, test, build Docker images, and push them to a container registry. For learning, I have also implemented CD using a self-hosted GitHub Actions runner on EC2 with Docker Compose. For industry-oriented deployment, I am learning Terraform for AWS ECS and Argo CD for Kubernetes-based GitOps CD."**

---

# 11. Final Revision

```text
GitHub Actions
      │
      ├── CI
      │    ├── Lint
      │    ├── Test
      │    ├── Build
      │    └── Push Image
      │
      └── CD Learning
           └── Self-hosted Runner → EC2 → Docker Compose


Industry-oriented deployment
      │
      ├── AWS ECS
      │    └── Terraform
      │
      └── Kubernetes
           └── Argo CD
```

## Key Point

```text
GitHub Actions → Learn CI
Self-hosted Runner → Learn demo CD
Terraform → Learn AWS infrastructure
Argo CD → Learn Kubernetes GitOps CD
```

This gives you a practical progression from **beginner CI/CD → AWS DevOps → Kubernetes GitOps**.
