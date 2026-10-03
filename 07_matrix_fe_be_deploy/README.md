## CI.yml

```yaml
# Goal - I have to Lint, Build, Test the code for Frontend & Backend
# then Push the images to DockerHub 
name: CI

on: 
    push:
        # paths:
        #   - '**'
        #   - '!.github/workflows/**'
        branches: [feat/matrix-docker-build]

jobs:
    lint-frontend:
        # Github runner
        runs-on: ubuntu-latest
        steps:
            - name: Checkout Code
              uses: actions/checkout@v7

            - name: Setup NodeJs
              uses: actions/setup-node@v6
              with:
                node-version: '20'
                cache: npm
                cache-dependency-path: frontend/package-lock.json
            
            - name: Install npm Packages
              run: npm install
              working-directory: frontend

            - name: Run Linter
              run: npm run lint
              working-directory: frontend

            - name: Run Tests
              run: npm run test 
              working-directory: frontend
            
                
    lint-backend:
        # Github runner
        runs-on: ubuntu-latest
        steps:
            - name: Checkout Code
              uses: actions/checkout@v7

            - name: Setup Go
              uses: actions/setup-go@v6
              with:
                go-version: '1.23'
                go-version-file: 'go.mod'
                cache-dependency-path: go.sum
            
            - name: Run Go Formatter
              run: go fmt
              working-directory: backend

            - name: Run Go Vet
              run: go vet
              working-directory: backend
            
            - name: Run Tests
              run: go test 
              working-directory: backend
          

    build-and-push:
        needs: [lint-frontend,lint-backend] 
        runs-on: ubuntu-latest
        strategy:
          matrix:
            folder: ['frontend','backend']
        steps:
            - name: Checkout Code
              uses: actions/checkout@v7

            - name: Docker Setup [Login]
              uses: docker/login-action@v4
              with:
                  username: ${{ vars.DOCKERHUB_USERNAME }}
                  password: ${{ secrets.DOCKERHUB_TOKEN }}

            - name: Docker Build and Push
              uses: docker/build-push-action@v7
              with:
                context: ./${{ matrix.folder}}
                push: true
                tags: ${{ vars.DOCKERHUB_USERNAME }}/devboard-${{ matrix.folder}}:latest
          

    deploy:
        needs: [lint-frontend,lint-backend,build-and-push]
        uses: ./.github/workflows/cd.yml
        secrets: inherit
```

## CD.yml

```yaml
# Goal: To deploy the built images from CI Steps once the CI Worklow is Succeeded

name: CD

on: 
    workflow_call:

jobs:
    deploy:
        
        runs-on: self-hosted
        steps:
            - name: Code Checkout
              uses: actions/checkout@v7

            - name: Copy Example Env to main env
              run: cp .env.example .env

            - name: Docker Setup [Login]
              uses: docker/login-action@v4
              with:
                username: ${{ vars.DOCKERHUB_USERNAME }}
                password: ${{ secrets.DOCKERHUB_TOKEN }}

            - name: Deploy the containers with Docker Compose
              run: |
                docker compose pull
                docker compose up -d
```

## matrix.yml

```yaml
# Goal To install multiple versions of Go and do linting for multiple versions
name: Go Linter

on: 
    workflow_dispatch:

jobs:
    code-format:
        runs-on: ubuntu-latest
        strategy:
            fail-fast: false
            matrix: 
                go: ['1.22','1.23','1.24']
        steps:
            - name: Code Checkout
              uses: actions/checkout@v7

            - name: Setup Go
              uses: actions/setup-go@v6
              with:
                go-version: ${{ matrix.go }}
                go-version-file: 'go.mod'
                cache-dependency-path: go.sum
            
            - name: Run Go Formatter
              run: go fmt
              working-directory: backend

            - name: Run Go Vet
              run: go vet
              working-directory: backend
```


# GitHub Actions — CI/CD + Matrix Build

## 1. Project Goal

The goal of this GitHub Actions setup is:

```text
Developer
   │
   │ git push
   ▼
GitHub Repository
   │
   ▼
┌──────────────────────────────┐
│          CI Workflow         │
│           ci.yml             │
└──────────────────────────────┘
   │
   ├── Frontend
   │    ├── Install dependencies
   │    ├── Lint
   │    └── Test
   │
   ├── Backend
   │    ├── Format
   │    ├── Vet
   │    └── Test
   │
   ▼
Build Docker Images
   │
   ├── Frontend Image
   └── Backend Image
   │
   ▼
Push Images
   │
   ▼
Docker Hub
   │
   ▼
CD Workflow
   │
   ▼
Self-Hosted Runner / Server
   │
   ├── docker compose pull
   └── docker compose up -d
   │
   ▼
Running Application
```

---

# 2. Complete CI/CD Architecture

```text
                         DEVELOPER
                             │
                             │ git push
                             ▼
                    ┌─────────────────┐
                    │     GitHub      │
                    │   Repository    │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │     CI.yml      │
                    │  Continuous     │
                    │  Integration    │
                    └────────┬────────┘
                             │
              ┌──────────────┴──────────────┐
              │                             │
              ▼                             ▼
      ┌───────────────┐             ┌───────────────┐
      │Frontend Lint   │             │ Backend Lint  │
      │                │             │               │
      │ npm install    │             │ go fmt        │
      │ npm run lint   │             │ go vet        │
      │ npm run test   │             │ go test       │
      └───────┬───────┘             └───────┬───────┘
              │                             │
              └──────────────┬──────────────┘
                             │
                         needs:
                             │
                             ▼
                 ┌──────────────────────┐
                 │ Docker Build & Push │
                 │     Matrix Job      │
                 └──────────┬───────────┘
                            │
                 ┌──────────┴──────────┐
                 │                     │
                 ▼                     ▼
          Frontend Image         Backend Image
                 │                     │
                 └──────────┬──────────┘
                            │
                            ▼
                      ┌───────────┐
                      │ DockerHub │
                      └─────┬─────┘
                            │
                            ▼
                       deploy job
                            │
                            ▼
                     ┌──────────────┐
                     │    CD.yml    │
                     └──────┬───────┘
                            │
                            ▼
                   Self-Hosted Runner
                            │
                            ▼
                  docker compose pull
                            │
                            ▼
                  docker compose up -d
                            │
                            ▼
                     APPLICATION
```

---

# 3. What is CI?

## CI = Continuous Integration

Continuous Integration means automatically checking the code whenever developers push code to GitHub.

Typical CI activities:

```text
Code Push
   ↓
Checkout Code
   ↓
Install Dependencies
   ↓
Lint
   ↓
Test
   ↓
Build
   ↓
Create Docker Image
   ↓
Push Image
```

### Why CI?

CI helps detect problems before deploying the application.

For example:

```text
Developer writes code
        ↓
      git push
        ↓
GitHub Actions
        ↓
Lint ❌
        ↓
Pipeline stops
        ↓
Application is NOT deployed
```

If all checks pass:

```text
Lint ✅
Test ✅
Build ✅
Docker Push ✅
        ↓
       CD
```

---

# 4. What is CD?

## CD = Continuous Deployment

CD takes the successfully built application and deploys it to a server.

In this project:

```text
DockerHub
    │
    │ docker compose pull
    ▼
Self-Hosted Server
    │
    │ docker compose up -d
    ▼
Application Running
```

---

# 5. CI Workflow — `ci.yml`

The CI workflow performs:

```text
Frontend Lint + Test
          │
          ├──────────────┐
          │              │
          ▼              ▼
Backend Lint + Test   Both Successful
                         │
                         ▼
                 Docker Build & Push
                         │
                         ▼
                    DockerHub
                         │
                         ▼
                       CD
```

---

# 6. Workflow Name

```yaml
name: CI
```

This gives the workflow the name:

```text
CI
```

You can see this name in:

```text
GitHub
  → Repository
  → Actions
```

---

# 7. Workflow Trigger

```yaml
on:
  push:
    branches: [feat/matrix-docker-build]
```

This means the workflow runs when code is pushed to:

```text
feat/matrix-docker-build
```

Example:

```bash
git checkout feat/matrix-docker-build

git add .

git commit -m "add CI matrix build"

git push origin feat/matrix-docker-build
```

Then:

```text
git push
   ↓
GitHub
   ↓
CI workflow starts
```

---

# 8. Frontend Lint Job

```yaml
lint-frontend:
  runs-on: ubuntu-latest
```

GitHub creates a temporary Ubuntu runner.

```text
GitHub
   │
   ▼
Ubuntu Runner
   │
   └── Run frontend CI commands
```

---

## Checkout Code

```yaml
- name: Checkout Code
  uses: actions/checkout@v7
```

This downloads the repository code into the GitHub runner.

```text
GitHub Repository
       │
       │ checkout
       ▼
Ubuntu Runner
       │
       ├── frontend/
       ├── backend/
       ├── Dockerfiles
       └── ...
```

---

# 9. Setup Node.js

```yaml
- name: Setup NodeJs
  uses: actions/setup-node@v6
  with:
    node-version: '20'
    cache: npm
    cache-dependency-path: frontend/package-lock.json
```

This installs/configures Node.js 20.

The npm cache can make future workflow runs faster.

---

# 10. Install Frontend Packages

```yaml
- name: Install npm Packages
  run: npm install
  working-directory: frontend
```

Because:

```yaml
working-directory: frontend
```

the command runs inside:

```text
frontend/
```

Equivalent:

```bash
cd frontend
npm install
```

---

# 11. Frontend Lint

```yaml
- name: Run Linter
  run: npm run lint
  working-directory: frontend
```

This checks the frontend source code for coding problems.

```text
Frontend Code
     │
     ▼
npm run lint
     │
 ┌───┴────┐
 ▼        ▼
PASS     FAIL
 │        │
 ▼        ▼
Next    Stop Job
```

---

# 12. Frontend Tests

```yaml
- name: Run Tests
  run: npm run test
  working-directory: frontend
```

This executes frontend tests.

```text
npm run test
      │
      ▼
 Test Cases
      │
 ┌────┴────┐
 ▼         ▼
PASS      FAIL
 │         │
 ▼         ▼
Continue  Job fails
```

---

# 13. Backend Lint Job

```yaml
lint-backend:
  runs-on: ubuntu-latest
```

A separate GitHub runner is used for the backend job.

```text
GitHub
  │
  ├── Ubuntu Runner #1
  │       └── Frontend
  │
  └── Ubuntu Runner #2
          └── Backend
```

The two jobs can run independently.

---

# 14. Setup Go

```yaml
- name: Setup Go
  uses: actions/setup-go@v6
  with:
    go-version: '1.23'
    go-version-file: go.mod
    cache-dependency-path: go.sum
```

This configures the Go environment.

Your backend is located in:

```text
backend/
```

and contains:

```text
go.mod
go.sum
```

---

# 15. Go Formatter

```yaml
- name: Run Go Formatter
  run: go fmt
  working-directory: backend
```

Equivalent:

```bash
cd backend
go fmt
```

`go fmt` formats Go source code according to Go's standard formatting rules.

---

# 16. Go Vet

```yaml
- name: Run Go Vet
  run: go vet
  working-directory: backend
```

`go vet` looks for suspicious or potentially incorrect Go code.

```text
Go Source Code
      │
      ▼
    go vet
      │
 ┌────┴────┐
 ▼         ▼
PASS      Problem
 │         │
 ▼         ▼
Continue  Fail
```

---

# 17. Backend Tests

```yaml
- name: Run Tests
  run: go test
  working-directory: backend
```

This executes the backend tests.

```text
Backend
   │
   ▼
go test
   │
   ▼
Tests
   │
 ┌─┴─┐
 ▼   ▼
PASS FAIL
```

---

# 18. `needs` — Job Dependency

The Docker build job contains:

```yaml
needs: [lint-frontend, lint-backend]
```

This is very important.

It means:

```text
lint-frontend ──────┐
                    │
                    ▼
              build-and-push
                    ▲
                    │
lint-backend ───────┘
```

Docker images are built only after both lint jobs succeed.

### If frontend fails:

```text
Frontend ❌
     │
     ▼
build-and-push
     │
     X
   SKIPPED
```

### If backend fails:

```text
Backend ❌
     │
     ▼
build-and-push
     │
     X
   SKIPPED
```

### If both pass:

```text
Frontend ✅
Backend  ✅
    │
    ▼
Docker Build
    │
    ▼
Docker Push
```

---

# 19. Matrix Strategy

Your Docker job uses:

```yaml
strategy:
  matrix:
    folder: ['frontend', 'backend']
```

This is called a **GitHub Actions Matrix Strategy**.

Instead of writing two separate jobs:

```text
Build Frontend
Build Backend
```

you write one job and give it multiple values.

GitHub automatically creates:

```text
matrix.folder = frontend
matrix.folder = backend
```

Visual:

```text
                 build-and-push
                       │
                  Matrix Strategy
                       │
              ┌────────┴────────┐
              │                 │
              ▼                 ▼
        folder=frontend    folder=backend
              │                 │
              ▼                 ▼
       Frontend Image       Backend Image
```

---

# 20. Matrix Example

```yaml
strategy:
  matrix:
    folder: ['frontend', 'backend']
```

GitHub effectively creates:

```text
Job 1:
matrix.folder = frontend

Job 2:
matrix.folder = backend
```

So the same steps execute twice.

---

# 21. DockerHub Login

```yaml
- name: Docker Setup [Login]
  uses: docker/login-action@v4
  with:
    username: ${{ vars.DOCKERHUB_USERNAME }}
    password: ${{ secrets.DOCKERHUB_TOKEN }}
```

GitHub Actions logs into DockerHub.

The values are stored in GitHub repository settings.

```text
GitHub Repository
       │
       ├── Variables
       │      └── DOCKERHUB_USERNAME
       │
       └── Secrets
              └── DOCKERHUB_TOKEN
```

### Why use a secret?

Never write your DockerHub password/token directly inside YAML.

Bad:

```yaml
password: my-password
```

Better:

```yaml
password: ${{ secrets.DOCKERHUB_TOKEN }}
```

---

# 22. Docker Build and Push

```yaml
- name: Docker Build and Push
  uses: docker/build-push-action@v7
  with:
    context: ./${{ matrix.folder }}
    push: true
    tags: ${{ vars.DOCKERHUB_USERNAME }}/devboard-${{ matrix.folder }}:latest
```

The important variable is:

```yaml
${{ matrix.folder }}
```

When matrix value is:

```text
frontend
```

the context becomes:

```text
./frontend
```

and image becomes:

```text
USERNAME/devboard-frontend:latest
```

When matrix value is:

```text
backend
```

the context becomes:

```text
./backend
```

and image becomes:

```text
USERNAME/devboard-backend:latest
```

---

# 23. Matrix Docker Flow

```text
                 Matrix
                   │
          ┌────────┴────────┐
          │                 │
          ▼                 ▼
      frontend           backend
          │                 │
          ▼                 ▼
     ./frontend         ./backend
          │                 │
          ▼                 ▼
 Docker Build          Docker Build
          │                 │
          ▼                 ▼
devboard-frontend    devboard-backend
          │                 │
          └────────┬────────┘
                   │
                   ▼
               DockerHub
```

---

# 24. DockerHub Images

After successful CI:

```text
DockerHub
│
├── <username>/devboard-frontend:latest
│
└── <username>/devboard-backend:latest
```

These images can later be pulled by the deployment server.

---

# 25. Deploy Job

```yaml
deploy:
  needs: [lint-frontend, lint-backend, build-and-push]
  uses: ./.github/workflows/cd.yml
  secrets: inherit
```

This job depends on:

```text
lint-frontend
       │
lint-backend
       │
build-and-push
       │
       ▼
     deploy
```

Only after all three succeed will the CD workflow be called.

---

# 26. Reusable Workflow

This line:

```yaml
uses: ./.github/workflows/cd.yml
```

means:

> Call another GitHub Actions workflow from this workflow.

This is called a **Reusable Workflow**.

The CD workflow contains:

```yaml
on:
  workflow_call:
```

`workflow_call` allows another workflow to call it.

Architecture:

```text
CI.yml
  │
  │ workflow_call
  ▼
CD.yml
```

---

# 27. `secrets: inherit`

```yaml
secrets: inherit
```

This allows the reusable workflow to receive the secrets available to the calling workflow.

So CD can use:

```yaml
${{ secrets.DOCKERHUB_TOKEN }}
```

---

# 28. CD Workflow — `cd.yml`

The purpose of CD is:

```text
DockerHub
    │
    │ pull latest images
    ▼
Self-Hosted Runner
    │
    ▼
Docker Compose
    │
    ▼
Running Containers
```

---

# 29. CD Trigger

```yaml
on:
  workflow_call:
```

This workflow is not being triggered directly by a normal push.

It is designed to be called from another workflow.

```text
CI.yml
   │
   │ calls
   ▼
CD.yml
```

---

# 30. Self-Hosted Runner

```yaml
runs-on: self-hosted
```

Unlike:

```yaml
runs-on: ubuntu-latest
```

GitHub does not create a temporary GitHub-hosted machine.

Instead, the job runs on your own registered machine.

```text
GitHub
   │
   │ sends job
   ▼
Your Server / EC2
   │
   └── Self-Hosted Runner
```

---

# 31. CD Checkout

```yaml
- name: Code Checkout
  uses: actions/checkout@v7
```

The repository is checked out onto the self-hosted runner.

This gives the server access to files such as:

```text
docker-compose.yml
.env.example
Docker configuration
```

---

# 32. Environment File

```yaml
- name: Copy Example Env to main env
  run: cp .env.example .env
```

This creates:

```text
.env.example
     │
     │ cp
     ▼
.env
```

Docker Compose can then read variables from `.env`.

For example:

```text
POSTGRES_USER
POSTGRES_PASSWORD
POSTGRES_DB
```

---

# 33. DockerHub Login on Server

```yaml
- name: Docker Setup [Login]
  uses: docker/login-action@v4
  with:
    username: ${{ vars.DOCKERHUB_USERNAME }}
    password: ${{ secrets.DOCKERHUB_TOKEN }}
```

The self-hosted runner logs into DockerHub.

Why?

Because it needs to pull the Docker images.

```text
DockerHub
    │
    │ authentication
    ▼
Self-Hosted Runner
```

---

# 34. Docker Compose Pull

```yaml
docker compose pull
```

This downloads the latest images from DockerHub.

```text
DockerHub
   │
   ├── devboard-frontend:latest
   │
   └── devboard-backend:latest
   │
   ▼
Self-Hosted Server
```

---

# 35. Docker Compose Up

```yaml
docker compose up -d
```

This starts the application containers in detached mode.

```text
docker compose up -d
          │
          ▼
┌──────────────────────┐
│ Docker Compose       │
├──────────────────────┤
│ Frontend Container   │
│ Backend Container    │
│ PostgreSQL Container │
└──────────────────────┘
```

`-d` means:

```text
Detached mode
```

The command returns while containers continue running in the background.

---

# 36. Complete CI → CD Flow

```text
                         Git Push
                            │
                            ▼
                    ┌───────────────┐
                    │    CI.yml     │
                    └───────┬───────┘
                            │
              ┌─────────────┴─────────────┐
              │                           │
              ▼                           ▼
      ┌────────────────┐          ┌────────────────┐
      │Frontend Lint   │          │ Backend Lint   │
      │                │          │                │
      │npm install     │          │go fmt          │
      │npm run lint    │          │go vet          │
      │npm run test    │          │go test         │
      └───────┬────────┘          └───────┬────────┘
              │                           │
              └─────────────┬─────────────┘
                            │
                            ▼
                    ┌───────────────┐
                    │ Matrix Build  │
                    └───────┬───────┘
                            │
                 ┌──────────┴──────────┐
                 │                     │
                 ▼                     ▼
           Frontend Build        Backend Build
                 │                     │
                 ▼                     ▼
           Docker Image          Docker Image
                 │                     │
                 └──────────┬──────────┘
                            │
                            ▼
                       DockerHub
                            │
                            ▼
                         deploy
                            │
                            ▼
                    ┌───────────────┐
                    │    CD.yml     │
                    └───────┬───────┘
                            │
                            ▼
                  Self-Hosted Runner
                            │
                            ▼
                  docker compose pull
                            │
                            ▼
                  docker compose up -d
                            │
                            ▼
                    🚀 Application
```

---

# 37. Matrix Workflow — `matrix.yml`

Your third workflow is for testing multiple Go versions.

```yaml
name: Go Linter

on:
  workflow_dispatch:
```

`workflow_dispatch` means you can manually start the workflow from:

```text
GitHub
   ↓
Actions
   ↓
Go Linter
   ↓
Run workflow
```

---

# 38. Why Matrix Testing?

Suppose your application should work with:

```text
Go 1.22
Go 1.23
Go 1.24
```

Instead of creating three separate jobs, use a matrix.

```yaml
strategy:
  fail-fast: false
  matrix:
    go: ['1.22', '1.23', '1.24']
```

GitHub creates:

```text
                 Go Matrix
                    │
       ┌────────────┼────────────┐
       │            │            │
       ▼            ▼            ▼
     Go 1.22      Go 1.23      Go 1.24
       │            │            │
       ▼            ▼            ▼
     Format       Format       Format
       │            │            │
       ▼            ▼            ▼
      Vet           Vet          Vet
```

---

# 39. `fail-fast: false`

```yaml
fail-fast: false
```

This means if one matrix job fails, GitHub does not immediately cancel the other matrix jobs.

Example:

```text
Go 1.22 → PASS
Go 1.23 → FAIL
Go 1.24 → still runs
```

Without this setting, GitHub may cancel other in-progress matrix jobs after a failure.

---

# 40. Matrix Go Version

```yaml
go-version: ${{ matrix.go }}
```

GitHub substitutes the matrix value.

For example:

```text
matrix.go = 1.22
```

becomes:

```yaml
go-version: '1.22'
```

Then another matrix execution uses:

```text
matrix.go = 1.23
```

and so on.

---

# 41. Matrix Visualization

```text
                  matrix.go
                      │
        ┌─────────────┼─────────────┐
        │             │             │
        ▼             ▼             ▼
      1.22           1.23          1.24
        │             │             │
        ▼             ▼             ▼
    Setup Go       Setup Go      Setup Go
        │             │             │
        ▼             ▼             ▼
      go fmt        go fmt        go fmt
        │             │             │
        ▼             ▼             ▼
      go vet        go vet        go vet
```

The benefit is that the same CI logic is automatically tested against multiple versions.

---

# 42. Important Difference Between the Two Matrix Examples

You have **two different uses of matrix**.

### Docker Matrix

```yaml
matrix:
  folder: ['frontend', 'backend']
```

Purpose:

```text
Build multiple applications/images
```

```text
frontend → Docker Image
backend  → Docker Image
```

### Go Matrix

```yaml
matrix:
  go: ['1.22', '1.23', '1.24']
```

Purpose:

```text
Test code against multiple Go versions
```

```text
Go 1.22 → Test
Go 1.23 → Test
Go 1.24 → Test
```

---

# 43. Matrix Concept in One Line

> **Matrix allows one GitHub Actions job definition to run multiple times with different combinations of values.**

Example:

```yaml
matrix:
  os: [ubuntu-latest, windows-latest]
  node: [18, 20]
```

Can create combinations such as:

```text
Ubuntu + Node 18
Ubuntu + Node 20
Windows + Node 18
Windows + Node 20
```

---

# 44. Workflow Dependency Summary

```text
CI.yml

lint-frontend ────────┐
                      │
                      ▼
                build-and-push
                      │
lint-backend ─────────┘
                      │
                      ▼
                    deploy
                      │
                      ▼
                    CD.yml
```

And inside Docker:

```text
build-and-push
      │
      ▼
   Matrix
      │
 ┌────┴─────┐
 ▼          ▼
frontend   backend
 ▼          ▼
Docker     Docker
Build      Build
 ▼          ▼
Push       Push
 └────┬─────┘
      ▼
  DockerHub
```

---

# 45. CI vs CD

| CI                     | CD                    |
| ---------------------- | --------------------- |
| Continuous Integration | Continuous Deployment |
| Checks code            | Deploys application   |
| Lint                   | Pull images           |
| Test                   | Start containers      |
| Build                  | Docker Compose        |
| Create Docker image    | Production/server     |
| Push image             | Running application   |

---

# 46. GitHub Runner Types

## GitHub-hosted Runner

Used in CI:

```yaml
runs-on: ubuntu-latest
```

```text
GitHub
  │
  ▼
Temporary Ubuntu Runner
  │
  ▼
Lint / Test / Build
```

## Self-hosted Runner

Used in CD:

```yaml
runs-on: self-hosted
```

```text
GitHub
  │
  ▼
Your EC2 / Server
  │
  ▼
Docker Compose
  │
  ▼
Application
```

---

# 47. Important Security Concept

Store sensitive values in:

```text
GitHub Repository
        │
        ├── Variables
        │     └── DOCKERHUB_USERNAME
        │
        └── Secrets
              └── DOCKERHUB_TOKEN
```

Use:

```yaml
${{ vars.DOCKERHUB_USERNAME }}
```

for a repository variable.

Use:

```yaml
${{ secrets.DOCKERHUB_TOKEN }}
```

for a secret.

Never hard-code credentials inside:

```text
ci.yml
cd.yml
Dockerfile
docker-compose.yml
```

---

# 48. Final Project Structure

```text
devboard/
│
├── frontend/
│   ├── package.json
│   ├── package-lock.json
│   ├── Dockerfile
│   └── ...
│
├── backend/
│   ├── go.mod
│   ├── go.sum
│   ├── Dockerfile
│   └── ...
│
├── docker-compose.yml
├── .env.example
│
└── .github/
    └── workflows/
        │
        ├── ci.yml
        ├── cd.yml
        └── matrix.yml
```

---

# 49. Complete Mental Model

Remember the complete pipeline like this:

```text
                 CODE
                  │
                  ▼
               GitHub
                  │
                  ▼
             ┌─────────┐
             │   CI    │
             └────┬────┘
                  │
        ┌─────────┴─────────┐
        ▼                   ▼
    Frontend              Backend
      Lint                  fmt
      Test                  vet
                            Test
        │                   │
        └─────────┬─────────┘
                  │
                  ▼
              Matrix Build
                  │
          ┌───────┴───────┐
          ▼               ▼
       Frontend         Backend
       Docker            Docker
        Image             Image
          │                 │
          └────────┬────────┘
                   ▼
                DockerHub
                   │
                   ▼
                  CD
                   │
                   ▼
          Self-Hosted Server
                   │
                   ▼
          docker compose pull
                   │
                   ▼
          docker compose up -d
                   │
                   ▼
              Application
```

## One-line interview explanation

> **My CI/CD pipeline uses GitHub Actions to lint and test the frontend and backend, uses a matrix strategy to build and push both Docker images to DockerHub, and then calls a reusable CD workflow that runs on a self-hosted runner to pull the latest images and deploy them using Docker Compose.**

---

# 50. Key Commands to Remember

```bash
# Frontend
npm install
npm run lint
npm run test

# Backend
go fmt
go vet
go test

# Docker
docker build
docker push
docker compose pull
docker compose up -d

# Git
git add .
git commit -m "message"
git push
```

## Most Important GitHub Actions Concepts

```text
Workflow
   ↓
Job
   ↓
Step
   ↓
Action / Command

needs
   ↓
Job dependency

matrix
   ↓
Run same job with multiple values

workflow_call
   ↓
Reusable workflow

workflow_dispatch
   ↓
Manual workflow execution

runs-on
   ↓
Select runner

secrets
   ↓
Sensitive credentials

vars
   ↓
Repository variables
```
