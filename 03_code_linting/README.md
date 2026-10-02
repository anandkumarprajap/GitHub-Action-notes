# GitHub Actions — Automatic Code Linting

## 1. What is the Goal of This Workflow?

The goal is to automatically check source code whenever code changes are pushed to GitHub.

For example:

```text
Developer changes Python code
          ↓
       git push
          ↓
    GitHub Actions
          ↓
    Install Linter
          ↓
    Check the code
          ↓
   ┌──────┴──────┐
   ↓             ↓
PASS           FAIL
   ↓             ↓
Continue       Fix code
```

The linter checks things such as:

* Syntax problems
* Formatting
* Indentation
* Unused variables/imports
* Code-style problems
* Common programming mistakes

> A linter is mainly a **static code checker**. It checks source code without running the complete application.

---

# 2. Basic Linting Workflow

Your workflow:

```yaml
# ============================================================
# GOAL
# ============================================================

# Whenever Python code changes,
# GitHub Actions runs the linter.
#
# The linter checks the Python source code
# for code-quality and style problems.

name: Auto Linter


# ============================================================
# TRIGGER
# ============================================================

# 'on' defines when the workflow should run.
#
# 'push' means the workflow runs when code is pushed.
#
# 'paths' limits the workflow to changes matching
# the specified Python-file pattern.

on:
  push:
    paths: ["**/*.py"]


# ============================================================
# JOBS
# ============================================================

jobs:

  # Job ID
  code-linting:

    # GitHub-hosted Ubuntu runner
    runs-on: ubuntu-latest

    steps:

      # ======================================================
      # STEP 1 — CHECKOUT CODE
      # ======================================================

      # Downloads/checks out the repository code
      # into the GitHub Actions runner.
      - name: Checkout/Clone the code
        uses: actions/checkout@v4


      # ======================================================
      # STEP 2 — SET UP PYTHON
      # ======================================================

      # Installs/configures Python on the runner.
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.14'


      # ======================================================
      # STEP 3 — INSTALL LINTER
      # ======================================================

      # Installs Ruff, a Python linter.
      - name: Install Linter
        run: pip install ruff


      # ======================================================
      # STEP 4 — RUN LINTER
      # ======================================================

      # Ruff checks the Python files in the repository.
      #
      # If Ruff finds problems, this command can return
      # a non-zero exit code and the GitHub Actions job fails.
      - name: Check the files for linting
        run: ruff check .
```

---

# 3. Linting Process

The complete process is:

```text
Developer
    │
    │ Changes code
    ▼
Git
    │
    │ git push
    ▼
GitHub Repository
    │
    ▼
GitHub Actions
    │
    ├── Checkout code
    │
    ├── Setup language
    │
    ├── Install linter
    │
    └── Run linter
            │
            ▼
       Code Checker
         /       \
       PASS      FAIL
        │          │
        ▼          ▼
    Continue    Fix code
```

---

# 4. Why Do We Use `actions/checkout`?

```yaml
- name: Checkout/Clone the code
  uses: actions/checkout@v4
```

The GitHub Actions runner is a fresh environment.

The runner needs your repository's source code before it can check it.

Therefore:

```text
GitHub Repository
       ↓
actions/checkout
       ↓
Runner
       ↓
Source Code Available
```

Without checking out the repository, the runner generally would not have your project files available to lint.

---

# 5. Why Do We Set Up Python?

```yaml
- name: Set up Python
  uses: actions/setup-python@v5
  with:
    python-version: '3.14'
```

This prepares the requested Python version on the runner.

Then we can run Python-related commands:

```bash
python --version
pip --version
ruff check .
```

---

# 6. Why Do We Install Ruff?

```yaml
- name: Install Linter
  run: pip install ruff
```

`ruff` is a Python linting tool.

After installation:

```bash
ruff check .
```

checks the Python source code.

---

# 7. What Happens When Ruff Finds an Error?

Example:

```text
Developer
    ↓
git push
    ↓
GitHub Actions
    ↓
ruff check .
    ↓
Problem found
    ↓
Job FAILED
```

The failed workflow tells the developer:

> Fix the code before continuing with the pipeline.

---

# 8. Python Example

For a Python project:

```text
Python
  ↓
Ruff
  ↓
ruff check .
```

Example project:

```text
my-python-project/
├── app.py
├── utils.py
├── requirements.txt
└── tests/
    └── test_app.py
```

Workflow:

```yaml
name: Python Lint

on:
  push:
    paths: ["**/*.py"]

jobs:
  lint:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.14'

      - name: Install Ruff
        run: pip install ruff

      - name: Run Ruff
        run: ruff check .
```

---

# 9. Flask Linting

Flask is a Python web framework.

Therefore:

```text
Flask
  ↓
Python
  ↓
Python Linter
  ↓
Ruff
```

Example Flask project:

```text
flask-app/
├── app.py
├── routes.py
├── models.py
├── requirements.txt
└── tests/
```

The same Python linting workflow can be used:

```yaml
- name: Install Ruff
  run: pip install ruff

- name: Run Ruff
  run: ruff check .
```

### Important

Ruff checks the Python source code.

It does not replace Flask application testing.

A complete Flask CI pipeline can be:

```text
Checkout
   ↓
Setup Python
   ↓
Install dependencies
   ↓
Lint
   ↓
Run tests
```

---

# 10. Django Linting

Django is also a Python framework.

Therefore:

```text
Django
  ↓
Python
  ↓
Ruff
```

Example:

```text
django-project/
├── manage.py
├── requirements.txt
├── config/
├── users/
├── products/
└── tests/
```

GitHub Actions:

```yaml
name: Django Lint

on:
  push:
    paths: ["**/*.py"]

jobs:
  lint:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.14'

      - name: Install Ruff
        run: pip install ruff

      - name: Run Ruff
        run: ruff check .
```

---

# 11. Node.js Linting

Node.js applications commonly use JavaScript or TypeScript.

A popular linting tool is:

```text
ESLint
```

Flow:

```text
Node.js
   ↓
JavaScript / TypeScript
   ↓
ESLint
   ↓
Code Check
```

Example workflow:

```yaml
name: Node Lint

on:
  push:
    paths:
      - "**/*.js"
      - "**/*.ts"

jobs:
  lint:
    runs-on: ubuntu-latest

    steps:

      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '22'

      - name: Install dependencies
        run: npm ci

      - name: Run ESLint
        run: npm run lint
```

Usually the project contains something like:

```json
{
  "scripts": {
    "lint": "eslint ."
  }
}
```

So:

```bash
npm run lint
```

runs ESLint according to the project's configuration.

---

# 12. Express.js Linting

Express.js is a Node.js web framework.

Therefore:

```text
Express
   ↓
Node.js
   ↓
JavaScript / TypeScript
   ↓
ESLint
```

Example Express project:

```text
express-app/
├── src/
│   ├── app.js
│   ├── routes/
│   ├── controllers/
│   └── middleware/
├── package.json
└── package-lock.json
```

Workflow:

```yaml
name: Express Lint

on:
  push:
    paths:
      - "**/*.js"

jobs:
  lint:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '22'

      - name: Install dependencies
        run: npm ci

      - name: Run ESLint
        run: npm run lint
```

---

# 13. React Linting

React applications commonly use JavaScript or TypeScript.

Therefore:

```text
React
  ↓
JavaScript / TypeScript
  ↓
ESLint
  ↓
Code Check
```

Example React project:

```text
react-app/
├── src/
│   ├── components/
│   ├── pages/
│   ├── App.jsx
│   └── main.jsx
├── package.json
└── package-lock.json
```

Workflow:

```yaml
name: React Lint

on:
  push:
    paths:
      - "**/*.js"
      - "**/*.jsx"
      - "**/*.ts"
      - "**/*.tsx"

jobs:
  lint:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '22'

      - name: Install dependencies
        run: npm ci

      - name: Run ESLint
        run: npm run lint
```

---

# 14. MERN Application Linting

MERN means:

```text
M = MongoDB
E = Express.js
R = React
N = Node.js
```

Typical structure:

```text
MERN Application
       │
       ├── Frontend
       │      └── React
       │           └── ESLint
       │
       └── Backend
              └── Node.js + Express
                   └── ESLint
```

For a MERN application, you can run linting separately:

```text
GitHub Actions
      │
      ├───────────────┐
      ▼               ▼
Frontend          Backend
   │                 │
React             Node/Express
   │                 │
ESLint            ESLint
```

Example:

```yaml
name: MERN Lint

on:
  push:
    paths:
      - "frontend/**"
      - "backend/**"

jobs:

  frontend-lint:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: acti
```
