```yaml
# Workflows [No indentation required for naming the workflow]
name: Hello

# On tells when the workflow should run (Give the indentation by tab)
on:
  workflow_dispatch: 

# Job tells What the workflow should do, there can be one or more then one jobs , hence we write a jobs
# jobs is an objects so needs indentation

jobs:
  # 1st (job Give indentation)
  say-hello:
    # Created on Ephermal Container with Ubuntu running
    runs-on: ubuntu-latest
    # The ephermal container will have steps to execute, hence name those steps and use run: run has the command
    steps:
      - name: Say hello to everyone
        run: echo "Hello Dosto"
```


# GitHub Actions — Basic Workflow Notes

## 1. Basic GitHub Actions Workflow

GitHub Actions workflow is a YAML file used to define:

* **When** the workflow should run
* **What jobs** should run
* **Which runner** should execute the jobs
* **Which steps/commands** should be executed

Example:

```yaml
# ============================================================
# GitHub Actions Workflow
# ============================================================

# 'name' defines the name of the workflow.
# This name appears in the GitHub Actions tab.
name: Hello


# ============================================================
# WHEN SHOULD THIS WORKFLOW RUN?
# ============================================================

# 'on' defines when the workflow should run.
#
# workflow_dispatch means:
# The workflow can be started manually from GitHub.
#
# GitHub Repository
#       ↓
# Actions
#       ↓
# Hello
#       ↓
# Run workflow
on:
  workflow_dispatch:


# ============================================================
# JOBS
# ============================================================

# 'jobs' contains the work that GitHub Actions should perform.
#
# A workflow can have:
# - One job
# - Multiple jobs
#
# Example:
#
# jobs:
#   build:
#   test:
#   deploy:
#
# 'jobs' is a YAML object/mapping, so its contents
# must be indented.
jobs:


  # ==========================================================
  # JOB 1
  # ==========================================================

  # 'say-hello' is the unique ID of this job.
  say-hello:


    # ========================================================
    # RUNNER
    # ========================================================

    # 'runs-on' specifies the runner/machine
    # where this job will execute.
    #
    # ubuntu-latest means GitHub provides
    # an Ubuntu-based runner.
    #
    # The runner is temporary (ephemeral) for the job.
    runs-on: ubuntu-latest


    # ========================================================
    # STEPS
    # ========================================================

    # 'steps' contains the individual tasks
    # that the job will execute.
    steps:


      # ------------------------------------------------------
      # STEP 1
      # ------------------------------------------------------

      # 'name' gives a human-readable name to the step.
      # This name appears in the GitHub Actions UI/logs.
      - name: Say hello to everyone


        # 'run' executes a shell command on the runner.
        #
        # echo is a Linux shell command.
        # It prints "Hello Dosto" in the workflow log.
        run: echo "Hello Dosto"
```

---

# 2. Workflow Execution Flow

```text
User
  │
  │ Clicks "Run workflow"
  ▼
GitHub Actions
  │
  ▼
Workflow: Hello
  │
  ▼
Job: say-hello
  │
  ▼
Runner: ubuntu-latest
  │
  ▼
Step: Say hello to everyone
  │
  ▼
run: echo "Hello Dosto"
  │
  ▼
Output:
Hello Dosto
```

---

# 3. GitHub Actions YAML Hierarchy

```text
name
 │
 └── Workflow name

on
 │
 └── Trigger
      │
      └── workflow_dispatch

jobs
 │
 └── say-hello
      │
      ├── runs-on
      │
      └── steps
           │
           └── name
                │
                └── run
```

---

# 4. Important Keywords

| Keyword             | Meaning                           |
| ------------------- | --------------------------------- |
| `name`              | Name of the workflow              |
| `on`                | Defines when the workflow runs    |
| `workflow_dispatch` | Manually starts the workflow      |
| `jobs`              | Contains one or more jobs         |
| `say-hello`         | Job ID                            |
| `runs-on`           | Defines the runner/machine        |
| `ubuntu-latest`     | Ubuntu-based GitHub-hosted runner |
| `steps`             | Contains tasks of a job           |
| `name`              | Name of a step                    |
| `run`               | Executes a shell command          |

---

# 5. Important YAML Indentation

YAML depends heavily on indentation.

Use **spaces**, not tabs.

Recommended:

```yaml
jobs:
  say-hello:
    runs-on: ubuntu-latest
    steps:
      - name: Say hello
        run: echo "Hello Dosto"
```

### Hierarchy

```text
jobs:
  └── say-hello:
       ├── runs-on:
       └── steps:
            └── step
                 ├── name:
                 └── run:
```

Incorrect indentation can cause the GitHub Actions workflow to fail.

---

# 6. What Happens Internally?

When the user manually runs the workflow:

```text
1. GitHub receives the workflow trigger
            ↓
2. GitHub creates a temporary runner
            ↓
3. Runner uses Ubuntu
            ↓
4. GitHub starts the 'say-hello' job
            ↓
5. The step is executed
            ↓
6. echo "Hello Dosto" runs
            ↓
7. Output appears in Actions logs
```

---

# 7. Final Concept

Remember the basic GitHub Actions structure:

```text
WORKFLOW
   │
   ├── Trigger
   │     └── on
   │
   └── Jobs
         │
         └── Job
              │
              ├── Runner
              │     └── runs-on
              │
              └── Steps
                    │
                    ├── Step 1
                    ├── Step 2
                    └── Step 3
```

### One-line definition

> **GitHub Actions workflow = Trigger + Jobs + Runner + Steps + Commands**
