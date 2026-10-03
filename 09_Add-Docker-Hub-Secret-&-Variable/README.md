# Add Docker Hub Secret and Variable in GitHub Actions

## Step 1: Login to Docker Hub

Go to **Docker Hub** and log in to your account.

```text
Docker Hub
   ↓
Account Settings
   ↓
Personal Access Tokens
   ↓
Generate New Token
```

Copy the generated Docker Hub token.

> Keep the token secret. Do not put it inside the Dockerfile or GitHub workflow.

---

## Step 2: Open GitHub Repository

Login to **GitHub** and open your project repository.

```text
GitHub
   ↓
Repository
   ↓
Settings
   ↓
Secrets and variables
   ↓
Actions
```

---

## Step 3: Add Docker Hub Token as Secret

Under **Repository secrets**, click:

```text
New repository secret
```

Add:

```text
Name:
DOCKERHUB_TOKEN

Secret:
<Paste Docker Hub Access Token>
```

Then click:

```text
Add secret
```

---

## Step 4: Add Docker Hub Username as Variable

Go to:

```text
Settings
   ↓
Secrets and variables
   ↓
Actions
   ↓
Variables
```

Click:

```text
New repository variable
```

Add:

```text
Name:
DOCKERHUB_USERNAME

Value:
anandkumarprajap
```

Then click:

```text
Add variable
```

---

## Final Setup

```text
GitHub Repository
       |
       ↓
Settings
       |
       ↓
Secrets and variables
       |
       └── Actions
            |
            ├── Secrets
            |    └── DOCKERHUB_TOKEN
            |
            └── Variables
                 └── DOCKERHUB_USERNAME
```

### GitHub Actions Usage

```yaml
username: ${{ vars.DOCKERHUB_USERNAME }}
password: ${{ secrets.DOCKERHUB_TOKEN }}
```

**Remember:**

```text
Docker Hub Token    → GitHub Secret
Docker Hub Username → GitHub Variable
```


....................................................................................................................................................................................



# GitHub Actions — Docker Hub Secrets & Variables

## Goal

The goal is to allow **GitHub Actions** to:

```text
Build Docker Image
       ↓ 
Login to Docker Hub
       ↓
Push Docker Image
       ↓
Deploy Application
```

We should **never put the Docker Hub password or access token directly inside the workflow or Dockerfile**.

Instead, we use:

* GitHub **Variables** → Docker Hub username
* GitHub **Secrets** → Docker Hub access token

---

# 1. Overall Flow

```text
Developer
    |
    | git push
    ↓
GitHub Repository
    |
    ↓
GitHub Actions
    |
    ↓
Self-Hosted Runner
    |
    ├── Lint
    ├── Build
    ├── Test
    |
    ↓
Docker Login
    |
    ├── DOCKERHUB_USERNAME
    └── DOCKERHUB_TOKEN
            |
            ↓
       Docker Hub
            |
            ↓
       Docker Image
            |
            ↓
       docker push
            |
            ↓
        Deployment
```

---

# 2. Create Docker Hub Access Token

First, log in to **Docker Hub**.

Go to:

```text
Docker Hub
    ↓
Account / Profile
    ↓
Account Settings
    ↓
Personal access tokens
    ↓
Generate new token
```

Create a token.

Example token name:

```text
github-actions
```

Give the token the permissions required to push images to the repository.

After generating the token:

```text
Copy the token
```

### Important

The token works like a password.

Do **not** put it inside:

```text
Dockerfile
README.md
GitHub workflow YAML
Source code
```

---

# 3. Open GitHub Actions Secrets and Variables

Go to your GitHub repository:

```text
GitHub Repository
       ↓
Settings
       ↓
Secrets and variables
       ↓
Actions
```

You will see:

```text
Secrets
Variables
```

---

# 4. Add Docker Hub Username as a Variable

Under:

```text
Repository variables
```

click:

```text
New repository variable
```

Add:

```text
Name:
DOCKERHUB_USERNAME
```

Value:

```text
<your-dockerhub-username>
```

Example:

```text
DOCKERHUB_USERNAME = anandkumarprajap
```

### Why Variable?

The Docker Hub username is generally not a sensitive credential.

Therefore:

```text
DOCKERHUB_USERNAME
        ↓
GitHub Variable
```

---

# 5. Add Docker Hub Token as a Secret

Go to:

```text
Repository Settings
       ↓
Secrets and variables
       ↓
Actions
       ↓
Repository secrets
```

Click:

```text
New repository secret
```

Add:

```text
Name:
DOCKERHUB_TOKEN
```

Value:

```text
<paste-your-docker-hub-access-token>
```

Then click:

```text
Add secret
```

Now GitHub contains:

```text
Variables
└── DOCKERHUB_USERNAME
    └── your-dockerhub-username

Secrets
└── DOCKERHUB_TOKEN
    └── ***************
```

---

# 6. Variable vs Secret

| Item                    | GitHub Type | Example              |
| ----------------------- | ----------- | -------------------- |
| Docker Hub username     | Variable    | `DOCKERHUB_USERNAME` |
| Docker Hub access token | Secret      | `DOCKERHUB_TOKEN`    |

Simple rule:

```text
Username
   ↓
Variable

Password / Token / API Key
   ↓
Secret
```

---

# 7. Dockerfile Does NOT Store Credentials

A Dockerfile is used to build the Docker image.

Example:

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

RUN npm run build
```

Do **not** write:

```dockerfile
DOCKER_USERNAME=...
DOCKER_PASSWORD=...
DOCKER_TOKEN=...
```

The Dockerfile doesn't need your Docker Hub credentials to build the image.

---

# 8. GitHub Actions Docker Login

Use the official Docker login action:

```yaml
- name: Login to Docker Hub
  uses: docker/login-action@v3
  with:
    username: ${{ vars.DOCKERHUB_USERNAME }}
    password: ${{ secrets.DOCKERHUB_TOKEN }}
```

Important syntax:

```text
vars
 ↓
GitHub Variables

secrets
 ↓
GitHub Secrets
```

Therefore:

```yaml
${{ vars.DOCKERHUB_USERNAME }}
```

and:

```yaml
${{ secrets.DOCKERHUB_TOKEN }}
```

---

# 9. Build Docker Image

After login, build the image.

### Frontend

```yaml
- name: Build Frontend Docker Image
  run: |
    docker build \
      -t ${{ vars.DOCKERHUB_USERNAME }}/devboard-frontend:latest \
      ./frontend
```

### Backend

```yaml
- name: Build Backend Docker Image
  run: |
    docker build \
      -t ${{ vars.DOCKERHUB_USERNAME }}/devboard-backend:latest \
      ./backend
```

---

# 10. Push Docker Image

After building the image:

```yaml
- name: Push Frontend Image
  run: |
    docker push \
      ${{ vars.DOCKERHUB_USERNAME }}/devboard-frontend:latest
```

Backend:

```yaml
- name: Push Backend Image
  run: |
    docker push \
      ${{ vars.DOCKERHUB_USERNAME }}/devboard-backend:latest
```

---

# 11. Complete Docker Login + Build + Push

Example:

```yaml
- name: Login to Docker Hub
  uses: docker/login-action@v3
  with:
    username: ${{ vars.DOCKERHUB_USERNAME }}
    password: ${{ secrets.DOCKERHUB_TOKEN }}

- name: Build Frontend Image
  run: |
    docker build \
      -t ${{ vars.DOCKERHUB_USERNAME }}/devboard-frontend:latest \
      ./frontend

- name: Push Frontend Image
  run: |
    docker push \
      ${{ vars.DOCKERHUB_USERNAME }}/devboard-frontend:latest

- name: Build Backend Image
  run: |
    docker build \
      -t ${{ vars.DOCKERHUB_USERNAME }}/devboard-backend:latest \
      ./backend

- name: Push Backend Image
  run: |
    docker push \
      ${{ vars.DOCKERHUB_USERNAME }}/devboard-backend:latest
```

---

# 12. Complete CI/CD Flow

```text
                    Developer
                        |
                        | git push
                        ↓
                GitHub Repository
                        |
                        ↓
                 GitHub Actions
                        |
                        ↓
             Self-Hosted EC2 Runner
                        |
            ┌───────────┴───────────┐
            ↓                       ↓
       Frontend CI             Backend CI
            |                       |
          Lint                    Lint
            |                       |
          Build                   Build
            |                       |
          Test                    Test
            └───────────┬───────────┘
                        ↓
                  Docker Login
                        |
          ┌─────────────┴─────────────┐
          ↓                           ↓
DOCKERHUB_USERNAME             DOCKERHUB_TOKEN
    Variable                      Secret
          |                           |
          └─────────────┬─────────────┘
                        ↓
                   Docker Hub
                        |
                        ↓
                  Docker Build
                        |
                        ↓
                   Docker Push
                        |
                        ↓
              Docker Hub Repository
                        |
                        ↓
                   Deployment
```

---

# 13. Devboard Example

For the Devboard project:

```text
Docker Hub
    |
    ├── devboard-frontend
    |
    └── devboard-backend
```

GitHub Actions:

```text
GitHub
  |
  ↓
Self-Hosted EC2
  |
  ├── Frontend Lint
  ├── Frontend Build
  ├── Backend Build
  ├── Tests
  |
  ↓
Docker Login
  |
  ↓
Build Images
  |
  ├── devboard-frontend
  └── devboard-backend
  |
  ↓
Docker Push
  |
  ↓
Docker Hub
```

---

# 14. Deployment

After images are pushed to Docker Hub, the deployment job can pull the latest images.

Example:

```bash
docker pull <username>/devboard-frontend:latest
docker pull <username>/devboard-backend:latest
```

Then:

```bash
docker compose up -d
```

Or use a deployment script:

```bash
chmod +x deploy.sh
./deploy.sh
```

Example `deploy.sh`:

```bash
#!/bin/bash

set -e

docker compose pull

docker compose up -d

docker compose ps
```

---

# 15. Important: `run.sh` vs `deploy.sh`

These two files have different purposes.

### `run.sh`

```bash
./run.sh
```

Purpose:

```text
Start GitHub Actions Self-Hosted Runner
```

Flow:

```text
EC2
 ↓
./run.sh
 ↓
GitHub Actions Runner
 ↓
Online
 ↓
Waiting for jobs
```

### `deploy.sh`

```bash
./deploy.sh
```

Purpose:

```text
Deploy Application
```

Flow:

```text
GitHub Actions
      ↓
Self-Hosted Runner
      ↓
./deploy.sh
      ↓
Docker Compose
      ↓
Application
```

---

# 16. Final GitHub Actions Architecture

```text
                         Git Push
                            |
                            ↓
                    GitHub Repository
                            |
                            ↓
                     GitHub Actions
                            |
                            ↓
                  Self-Hosted EC2 Runner
                            |
              ┌─────────────┴─────────────┐
              ↓                           ↓
         Frontend CI                  Backend CI
              |                           |
            Lint                        Lint
              |                           |
            Build                       Build
              |                           |
            Test                        Test
              └─────────────┬─────────────┘
                            ↓
                       Docker Login
                            |
                  ┌─────────┴─────────┐
                  ↓                   ↓
             Username              Token
              Variable             Secret
                  |                   |
                  └─────────┬─────────┘
                            ↓
                       Docker Hub
                            |
                            ↓
                       Build Image
                            |
                            ↓
                       Push Image
                            |
                            ↓
                       Docker Hub
                            |
                            ↓
                        Deploy
                            |
                            ↓
                       Application
```

---

# 17. Quick Revision

```text
1. Create Docker Hub Access Token
        ↓
2. GitHub Repository
        ↓
3. Settings
        ↓
4. Secrets and variables
        ↓
5. Add DOCKERHUB_USERNAME
   → Variable
        ↓
6. Add DOCKERHUB_TOKEN
   → Secret
        ↓
7. GitHub Actions
        ↓
8. docker/login-action
        ↓
9. docker build
        ↓
10. docker push
        ↓
11. Docker Hub
        ↓
12. Deployment
```

## Most Important Commands

```bash
# Build
docker build -t username/devboard-frontend:latest ./frontend

# Login is handled by GitHub Actions

# Push
docker push username/devboard-frontend:latest
```

## Most Important GitHub Actions Syntax

```yaml
${{ vars.DOCKERHUB_USERNAME }}
```

```yaml
${{ secrets.DOCKERHUB_TOKEN }}
```

### Remember

```text
Dockerfile
    ↓
Build Image

GitHub Variable
    ↓
Docker Hub Username

GitHub Secret
    ↓
Docker Hub Token

docker/login-action
    ↓
Authenticate

docker build
    ↓
Create Image

docker push
    ↓
Push Image to Docker Hub
```




........................................
