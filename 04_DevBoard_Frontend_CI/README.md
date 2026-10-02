```yaml
name: CI

on:
  push:
    branches: [fix/lint]

jobs:

  # ==========================================================
  # JOB 1 — LINT
  # ==========================================================
  code-lint:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v7

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: "npm"

      - name: Install Dependencies
        run: npm install

      - name: Run Lint (Biome)
        run: npm run lint


  # ==========================================================
  # JOB 2 — BUILD AND PUSH
  # ==========================================================
  build-and-push:

    # IMPORTANT:
    # This job will run ONLY after code-lint succeeds.
    needs: [code-lint]

    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v7

      - name: Docker Setup [Login]
        uses: docker/login-action@v4
        with:
          username: ${{ vars.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Docker Build and Push
        uses: docker/build-push-action@v7
        with:
          push: true
          tags: ${{ vars.DOCKERHUB_USERNAME }}/devboard-fe-master:latest   
```


# GitHub Actions — DevBoard Frontend CI

## 1. Goal

The goal of this workflow is:

> **Whenever code is pushed to the `fix/lint` branch, lint the DevBoard Frontend code, then build the Docker image and push it to Docker Hub.**

Pipeline:

```text
Developer
    │
    │ git push
    ▼
fix/lint branch
    │
    ▼
GitHub Actions
    │
    ▼
Code Lint
    │
    ▼
Docker Build
    │
    ▼
Docker Hub
```

---

# 2. Complete Workflow

```yaml
# ============================================================
# GOAL
# ============================================================
#
# Build the DevBoard Frontend Docker image
# and push it to Docker Hub after linting the code.
#

# Name of the GitHub Actions workflow.
name: CI


# ============================================================
# WORKFLOW TRIGGER
# ============================================================

# 'on' defines when the workflow should run.
#
# 'push' means the workflow runs whenever code is pushed.
#
# 'branches' limits the workflow to the specified branch.
on:
  push:
    branches: [fix/lint]


# ============================================================
# JOBS
# ============================================================

jobs:


  # ==========================================================
  # JOB 1 — CODE LINT
  # ==========================================================

  # This job checks the frontend source code.
  code-lint:

    # Run this job on a GitHub-hosted Ubuntu runner.
    runs-on: ubuntu-latest

    steps:

      # ------------------------------------------------------
      # STEP 1 — CHECKOUT CODE
      # ------------------------------------------------------

      # Downloads the repository code into the runner.
      #
      # The runner needs the source code before it can
      # install dependencies and run the linter.
      - name: Checkout Code
        uses: actions/checkout@v4


      # ------------------------------------------------------
      # STEP 2 — SETUP NODE.JS
      # ------------------------------------------------------

      # Installs/configures Node.js on the runner.
      - name: Setup Node.js
        uses: actions/setup-node@v4

        with:

          # Use Node.js version 20.
          node-version: 20

          # Enable npm dependency caching.
          #
          # This can make future workflow runs faster because
          # npm packages can be restored from the cache.
          cache: "npm"


      # ------------------------------------------------------
      # STEP 3 — INSTALL DEPENDENCIES
      # ------------------------------------------------------

      # Installs the dependencies listed in package.json.
      #
      # For reproducible CI builds, many projects prefer:
      #
      # npm ci
      #
      # when package-lock.json is committed.
      - name: Install Dependencies
        run: npm install


      # ------------------------------------------------------
      # STEP 4 — RUN LINTER
      # ------------------------------------------------------

      # Runs the lint script defined in package.json.
      #
      # Example:
      #
      # "scripts": {
      #   "lint": "biome check ."
      # }
      #
      # Therefore:
      #
      # npm run lint
      #
      # runs Biome.
      - name: Run Lint (Biome)
        run: npm run lint


  # ==========================================================
  # JOB 2 — BUILD AND PUSH
  # ==========================================================

  # This job builds the Docker image and pushes it
  # to Docker Hub.
  build-and-push:

    runs-on: ubuntu-latest

    steps:

      # ------------------------------------------------------
      # STEP 1 — CHECKOUT CODE
      # ------------------------------------------------------

      # Gets the source code required for the Docker build.
      - name: Checkout code
        uses: actions/checkout@v4


      # ------------------------------------------------------
      # STEP 2 — LOGIN TO DOCKER HUB
      # ------------------------------------------------------

      # Logs in to Docker Hub.
      #
      # DOCKERHUB_USERNAME is stored as a GitHub Repository
      # Variable.
      #
      # DOCKERHUB_TOKEN is stored as a GitHub Secret.
      #
      # Secrets should be used for sensitive information.
      - name: Docker Setup [Login]
        uses: docker/login-action@v4
        with:
          username: ${{ vars.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}


      # ------------------------------------------------------
      # STEP 3 — BUILD AND PUSH DOCKER IMAGE
      # ------------------------------------------------------

      # Builds the Docker image using the Dockerfile
      # in the repository and pushes it to Docker Hub.
      - name: Docker Build and Push
        uses: docker/build-push-action@v7
        with:

          # Push the image after building it.
          push: true

          # Docker image name:
          #
          # DOCKERHUB_USERNAME/devboard-fe-master:latest
          #
          # Example:
          #
          # anand/devboard-fe-master:latest
          tags: ${{ vars.DOCKERHUB_USERNAME }}/devboard-fe-master:latest
```

---

# 3. Workflow Structure

There are **two jobs**:

```text
jobs
 │
 ├── code-lint
 │
 └── build-and-push
```

By default, jobs without dependencies can run **independently/in parallel**.

So your current workflow is actually:

```text
                GitHub Actions
                      │
             ┌────────┴────────┐
             ▼                 ▼
        code-lint        build-and-push
             │                 │
             ▼                 ▼
           Biome          Docker Build
                               │
                               ▼
                           Docker Hub
```

---

# 4. Important: Does Your Current Code Wait for Lint?

### No.

Your current workflow does **not** contain:

```yaml
needs: [code-lint]
```

Therefore:

```text
code-lint ─────────→ Lint

build-and-push ────→ Docker Build → Push
```

They can start independently.

This means:

```text
Lint FAIL
   │
   └──────────────┐
                  │
                  ▼
          Build may still run
                  │
                  ▼
             Push image
```

That is usually **not what you want** when your goal is:

> Build and push only after successful linting.

---

# 5. Recommended Structure

Add:

```yaml
needs: [code-lint]
```

to the `build-and-push` job.

Example:

```yaml
build-and-push:
  needs: [code-lint]
  runs-on: ubuntu-latest
```

Now the dependency becomes:

```text
code-lint
    │
    │ SUCCESS
    ▼
build-and-push
    │
    ▼
Docker Hub
```

If linting fails:

```text
code-lint
    │
    │ FAIL
    ▼
build-and-push
    │
    X
  NOT RUN
```

---

# 6. Recommended CI Pipeline

```text
                  Developer
                      │
                      │ git push
                      ▼
                fix/lint branch
                      │
                      ▼
                GitHub Actions
                      │
                      ▼
                ┌───────────┐
                │ Code Lint │
                │  Biome    │
                └─────┬─────┘
                      │
                 SUCCESS
                      │
                      ▼
              ┌───────────────┐
              │ Docker Build  │
              └───────┬───────┘
                      │
                      ▼
              ┌───────────────┐
              │ Docker Push   │
              └───────┬───────┘
                      │
                      ▼
                 Docker Hub
```

---

# 7. Understanding `npm run lint`

This command:

```bash
npm run lint
```

does not automatically mean "run Biome."

It runs whatever command is defined as `lint` in `package.json`.

Example:

```json
{
  "scripts": {
    "lint": "biome check ."
  }
}
```

Then:

```bash
npm run lint
```

executes:

```bash
biome check .
```

So the flow is:

```text
GitHub Actions
      ↓
npm run lint
      ↓
package.json
      ↓
"lint": "biome check ."
      ↓
Biome
      ↓
Check source code
```

---

# 8. Understanding npm Dependency Caching

This section:

```yaml
with:
  node-version: 20
  cache: "npm"
```

does two things.

### Node version

```yaml
node-version: 20
```

Uses Node.js 20.

### npm cache

```yaml
cache: "npm"
```

Allows `setup-node` to cache npm dependencies based on the project's lockfile.

The goal is to make repeated workflow runs faster.

---

# 9. Docker Hub Login

This section:

```yaml
- name: Docker Setup [Login]
  uses: docker/login-action@v4
  with:
    username: ${{ vars.DOCKERHUB_USERNAME }}
    password: ${{ secrets.DOCKERHUB_TOKEN }}
```

logs the GitHub Actions runner into Docker Hub.

Flow:

```text
GitHub Actions Runner
        │
        │ Username
        │ +
        │ Docker Hub Token
        ▼
    Docker Hub
        │
        ▼
     Logged In
```

---

# 10. Variables vs Secrets

You are using:

```yaml
${{ vars.DOCKERHUB_USERNAME }}
```

and:

```yaml
${{ secrets.DOCKERHUB_TOKEN }}
```

### `vars`

Used for configuration values that don't need to be treated as secrets.

Example:

```text
DOCKERHUB_USERNAME
```

### `secrets`

Used for sensitive credentials.

Example:

```text
DOCKERHUB_TOKEN
```

The Docker Hub access token should **not** be written directly into the YAML file.

---

# 11. Docker Image Tag

You have:

```yaml
tags: ${{ vars.DOCKERHUB_USERNAME }}/devboard-fe-master:latest
```

The general Docker image format is:

```text
USERNAME/IMAGE_NAME:TAG
```

Your image:

```text
DOCKERHUB_USERNAME/devboard-fe-master:latest
```

For example:

```text
anand/devboard-fe-master:latest
```

Conceptually:

```text
anand
 │
 └── Docker Hub username

devboard-fe-master
 │
 └── Image name

latest
 │
 └── Image tag
```

---

# 12. Build and Push

This action:

```yaml
uses: docker/build-push-action@v7
```

can build and push a Docker image.

With:

```yaml
push: true
```

the image is pushed after the build succeeds.

Flow:

```text
Dockerfile
    │
    ▼
docker/build-push-action
    │
    ├── Build image
    │
    ▼
Docker Image
    │
    ▼
Docker Hub
```

---

# 13. Lint vs Build

These are two different operations.

### Lint

```text
Source Code
    ↓
Biome
    ↓
Code Quality Check
```

### Build

```text
Source Code
    ↓
Dockerfile
    ↓
Docker Build
    ↓
Docker Image
```

Therefore:

```text
Source Code
     │
     ▼
   Lint
     │
     ▼
 Docker Build
     │
     ▼
 Docker Image
     │
     ▼
 Docker Hub
```

---

# 14. CI Pipeline Concept

This workflow is a **CI (Continuous Integration)** pipeline with an image-publishing step.

```text
Developer
    ↓
git push
    ↓
GitHub
    ↓
GitHub Actions
    ↓
Lint
    ↓
Docker Build
    ↓
Docker Push
    ↓
Docker Hub
```

Later, a CD stage could pull this image and deploy it to:

```text
Docker Hub
    ↓
AWS EC2
    OR
AWS ECS
    OR
AWS EKS
    OR
Kubernetes
```

---

# 15. Final Recommended Structure

For the intended behavior, use:

```yaml
build-and-push:
  needs: [code-lint]
  runs-on: ubuntu-latest
```

Then the final pipeline is:

```text
               git push
                  │
                  ▼
            GitHub Actions
                  │
                  ▼
             Code Lint
               Biome
                  │
             ┌────┴────┐
             │         │
           PASS       FAIL
             │         │
             ▼         X
       Docker Build
             │
             ▼
        Docker Image
             │
             ▼
          Docker Hub
```

## Quick Revision

```text
name
  ↓
Workflow name

on
  ↓
Trigger

jobs
  ↓
Jobs

runs-on
  ↓
Runner

uses
  ↓
Use a GitHub Action

with
  ↓
Action configuration

run
  ↓
Run shell command

needs
  ↓
Job dependency

vars
  ↓
Repository variables

secrets
  ↓
Sensitive credentials

push: true
  ↓
Push Docker image
```

### One-line pipeline

> **Git Push → Lint with Biome → Docker Build → Docker Hub Push**
