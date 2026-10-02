# GitHub Actions CI/CD Pipeline — Basic Structure

## 1. What is this Workflow?

This GitHub Actions workflow demonstrates a basic **CI/CD pipeline**:

```text
User
  │
  ▼
GitHub Actions
  │
  ▼
Code
  │
  ▼
Build
  │
  ▼
Test
  │
  ▼
Deploy
```

The workflow contains **4 jobs**:

1. `code` → Clone/checkout the code
2. `build` → Build the application using Docker
3. `test` → Run test cases
4. `deploy` → Deploy the application

---

# 2. Complete Workflow

```yaml
# ============================================================
# WORKFLOW NAME
# ============================================================

# Name of the GitHub Actions workflow.
# This name appears in the GitHub Actions tab.
name: CICD


# ============================================================
# WORKFLOW TRIGGER
# ============================================================

# 'on' defines when the workflow should run.
#
# workflow_dispatch means:
# The workflow can be started manually.
#
# GitHub Repository
#       ↓
# Actions
#       ↓
# CICD
#       ↓
# Run workflow
on:
  workflow_dispatch:


# ============================================================
# JOBS
# ============================================================

# 'jobs' contains all jobs in this workflow.
jobs:


  # ==========================================================
  # JOB 1 — CODE
  # ==========================================================

  # Job ID = code
  # This job is responsible for getting the source code.
  code:

    # Run this job on a GitHub-hosted Ubuntu runner.
    runs-on: ubuntu-latest

    # Steps are the individual tasks of this job.
    steps:

      # Step 1
      - name: clone the code

        # Executes a shell command.
        #
        # NOTE:
        # This only prints a message.
        # It does NOT actually clone the repository.
        #
        # For real checkout, normally use:
        # uses: actions/checkout@v4
        run: echo "Cloning the code"


  # ==========================================================
  # JOB 2 — BUILD
  # ==========================================================

  # Job ID = build
  build:

    # 'needs' creates a dependency.
    #
    # This means:
    # build will start only after the 'code' job succeeds.
    #
    # Flow:
    # code → build
    needs: [code]

    # Build job runs on Ubuntu.
    runs-on: ubuntu-latest

    steps:

      # Step 1
      - name: Build using Docker

        # This currently only prints a message.
        # It does NOT actually build a Docker image.
        #
        # A real pipeline could use:
        # docker build -t my-app .
        run: echo "Building using Docker"


  # ==========================================================
  # JOB 3 — TEST
  # ==========================================================

  # Job ID = test
  test:

    # Test starts only after the build job succeeds.
    #
    # Flow:
    # code → build → test
    needs: [build]

    # Test job runs on Ubuntu.
    runs-on: ubuntu-latest

    steps:

      # Step 1
      - name: Running test cases

        # Currently only prints a message.
        # A real project would execute actual test commands.
        #
        # Example:
        # npm test
        # pytest
        # go test ./...
        run: echo "Testing the app"


  # ==========================================================
  # JOB 4 — DEPLOY
  # ==========================================================

  # Job ID = deploy
  deploy:

    # Deploy depends on BOTH build and test.
    #
    # Therefore deploy starts only when:
    #
    # build → SUCCESS
    # test  → SUCCESS
    #
    # Flow:
    # code → build → test
    #              ↘
    #                deploy
    #
    # Because test already depends on build,
    # the practical flow becomes:
    #
    # code → build → test → deploy
    #
    # 'needs: [build, test]' explicitly requires
    # both jobs to complete successfully.
    needs: [build, test]

    # Deploy job runs on Ubuntu.
    runs-on: ubuntu-latest

    steps:

      # Step 1
      - name: Deploying on the machine

        # Currently only prints a message.
        # It does NOT actually deploy the application.
        #
        # A real pipeline could use:
        # SSH
        # AWS
        # Docker
        # Kubernetes
        # Argo CD
        # etc.
        run: echo "Deploying the code"
```

---

# 3. Pipeline Structure

The jobs execute in this order:

```text
              GitHub Actions
                    │
                    ▼
              ┌──────────┐
              │   code   │
              └────┬─────┘
                   │
                   │ needs: [code]
                   ▼
              ┌──────────┐
              │  build   │
              └────┬─────┘
                   │
                   │ needs: [build]
                   ▼
              ┌──────────┐
              │   test   │
              └────┬─────┘
                   │
                   │ needs: [build, test]
                   ▼
              ┌──────────┐
              │  deploy  │
              └──────────┘
```

---

# 4. Understanding `needs`

`needs` is used to create a **dependency between jobs**.

## Example

```yaml
build:
  needs: [code]
```

Meaning:

```text
code
  ↓
build
```

The `build` job waits for the `code` job to complete successfully.

---

## Multiple Dependencies

```yaml
deploy:
  needs: [build, test]
```

Meaning:

```text
build ──────┐
            ├──→ deploy
test  ──────┘
```

The `deploy` job waits for both `build` and `test`.

---

# 5. Complete CI/CD Flow

```text
┌──────────────┐
│     User     │
└──────┬───────┘
       │
       │ Run workflow
       ▼
┌──────────────────┐
│  GitHub Actions  │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ CODE             │
│ Get source code  │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ BUILD            │
│ Docker build     │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ TEST             │
│ Test application │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ DEPLOY           │
│ Deploy app       │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│    End User      │
└──────────────────┘
```

---

# 6. CI/CD Meaning

## CI — Continuous Integration

CI mainly focuses on:

```text
Code
  ↓
Build
  ↓
Test
```

The purpose is to automatically build and test changes.

---

## CD — Continuous Delivery / Deployment

CD focuses on:

```text
Tested Application
       ↓
    Deploy
       ↓
    Server
       ↓
   End User
```

---

# 7. Job vs Step

### Job

A **job** is a group of steps that runs on a runner.

Example:

```yaml
build:
  runs-on: ubuntu-latest
  steps:
    ...
```

### Step

A **step** is an individual task inside a job.

Example:

```yaml
steps:
  - name: Build using Docker
    run: echo "Building using Docker"
```

---

# 8. Runner

```yaml
runs-on: ubuntu-latest
```

This tells GitHub:

> Run this job on a GitHub-hosted Ubuntu runner.

Each job can have its own runner.

```text
code   → Ubuntu runner
build  → Ubuntu runner
test   → Ubuntu runner
deploy → Ubuntu runner
```

---

# 9. Important Note About This Example

The commands in this example are currently only `echo` commands.

For example:

```bash
echo "Building using Docker"
```

This **does not actually build a Docker image**.

Similarly:

```bash
echo "Cloning the code"
```

does **not actually clone the repository.

For a real GitHub Actions pipeline, we would replace them with actual actions/commands such as:

```yaml
- uses: actions/checkout@v4
```

and:

```bash
docker build -t my-app .
```

and actual test/deployment commands.

---

# 10. Short Revision Notes

```text
name
  ↓
Workflow name

on
  ↓
Workflow trigger

jobs
  ↓
Contains jobs

runs-on
  ↓
Defines runner

steps
  ↓
Contains tasks

run
  ↓
Executes shell command

needs
  ↓
Creates job dependency
```

### Final CI/CD Pipeline

```text
CODE
  ↓
BUILD
  ↓
TEST
  ↓
DEPLOY
  ↓
END USER
```

### One-line Definition

> **GitHub Actions CI/CD pipeline automates the process of getting code, building the application, testing it, and deploying it.**
