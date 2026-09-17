# 🚀 PR-merging-automation

[![GitHub Template](https://img.shields.io/badge/GitHub-Template-blue?logo=github&style=flat-square)](https://github.com/kiran8688/PR-merging-automation)
[![CI / Auto-Merge](https://img.shields.io/badge/CI-Auto--Merge-success?logo=githubactions&logoColor=white&style=flat-square)](https://github.com/kiran8688/PR-merging-automation/actions)
[![Platform Support](https://img.shields.io/badge/Runtime-Node.js%20%7C%20Python%20%7C%20Agnostic-informational?style=flat-square)](#customizing-the-test-pipeline)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](LICENSE)

An enterprise-grade, standardized GitHub starter template engineered to automate incoming pull request validation, execute dynamic test suites across multi-stack projects, and seamlessly merge vetted contributions into production branches without manual review bottlenecks.

---

## 📌 Table of Contents

- [Overview](#-overview)
- [System Architecture](#-system-architecture)
- [How the Workflow Operates](#-how-the-workflow-operates)
- [Step-by-Step Repository Setup](#-step-by-step-repository-setup)
  - [Step 1: Mark as a Template Repository](#step-1-mark-as-a-template-repository)
  - [Step 2: Commit the CI/CD Workflow](#step-2-commit-the-cicd-workflow)
  - [Step 3: Enable Auto-Merge in Repository Settings](#step-3-enable-auto-merge-in-repository-settings)
  - [Step 4: Configure GitHub Actions Permissions](#step-4-configure-github-actions-permissions)
  - [Step 5: Configure Branch Protection Rules](#step-5-configure-branch-protection-rules)
- [Creating New Projects from this Template](#-creating-new-projects-from-this-template)
- [Customizing the Test Pipeline](#-customizing-the-test-pipeline)
- [Troubleshooting & Verification](#-troubleshooting--verification)

---

## 📖 Overview

In modern continuous delivery, pull requests often stall in review backlogs—even when all tests and checks pass cleanly. **PR-merging-automation** eliminates this latency by transforming pull request handling into an autonomous, event-driven pipeline:

- **Dynamic Environment Detection**: Automatically identifies the project ecosystem (Node.js, Python, or generic assets) based on manifest files.
- **Automated Quality Gates**: Executes build and test suites in clean, isolated virtual environments on `ubuntu-latest`.
- **Hands-Free Auto-Merging**: Couples GitHub's native `gh pr merge --auto` API with branch protection rules to ensure code is merged the instant all criteria turn green.
- **Fail-Safe Operation**: Automatically blocks merging and flags failed runs if unit tests, syntax checks, or merge conflicts occur.

---

## 🏗 System Architecture

The diagram below illustrates the end-to-end event flow when a pull request is submitted or updated:

```text
       +-------------------------------------------------------------+
       |                  Developer / Bot / Contributor               |
       +-------------------------------------------------------------+
                                      │
                                      │ Opens / Updates Pull Request
                                      ▼
             +───────────────────────────────────────────────────+
             |         GitHub Event Hook: `pull_request`         |
             |       (types: opened, synchronize, reopened)       |
             +───────────────────────────────────────────────────+
                                      │
                                      │ Triggers Workflow
                                      ▼
             +───────────────────────────────────────────────────+
             |           GitHub Actions Runner Container          |
             |                   (ubuntu-latest)                 |
             +───────────────────────────────────────────────────+
                                      │
                         [ Inspect Repository Files ]
                                      │
             ┌────────────────────────┴────────────────────────┐
             ▼                                                 ▼
   +--------------------+                            +--------------------+
   |    package.json    |                            |  requirements.txt  |
   |      Detected      |                            |  pyproject.toml    |
   +--------------------+                            +--------------------+
             │                                                 │
             ├─ Setup Node.js (v18)                            ├─ Setup Python (v3.11)
             ├─ npm ci || npm install                          ├─ pip install -r requirements.txt
             └─ npm test --if-present                          └─ pytest (if installed)
             │                                                 │
             └────────────────────────┬────────────────────────┘
                                      │
                                      ▼
                          [ Test Suite Evaluation ]
                                      │
                    ┌─────────────────┴─────────────────┐
                    │                                   │
              [ Tests PASS ]                      [ Tests FAIL ]
                    │                                   │
                    ▼                                   ▼
   +──────────────────────────────────+   +──────────────────────────────────+
   |   Execute Auto-Merge Command:    |   |     Halt Pipeline Execution      |
   |   `gh pr merge --auto --merge`   |   |     Status Check Marked Failed   |
   +──────────────────────────────────+   +──────────────────────────────────+
                    │                                   │
                    ▼                                   ▼
   +──────────────────────────────────+   +──────────────────────────────────+
   |   GitHub Branch Protection Gate  |   |   PR Blocked from Merging        |
   |   (Verifies all required checks) |   |   (Requires Developer Revision)  |
   +──────────────────────────────────+   +──────────────────────────────────+
                    │
                    ▼
   +──────────────────────────────────+
   |    PR Merged into Base Branch    |
   |   (`main` / `master` updated)    |
   +──────────────────────────────────+
```

---

## ⚙ How the Workflow Operates

The underlying workflow is governed by `.github/workflows/pr-review-automerge.yml`:

| Stage | Responsibility | Enforcement Mechanism |
| :--- | :--- | :--- |
| **Permissions** | Grants the runner permission to inspect and merge PRs | `contents: write`, `pull-requests: write`, `checks: read` |
| **Workspace Checkout** | Fetches the full Git commit history of the PR branch | `actions/checkout@v4` |
| **Node.js Suite** | Installs dependencies and runs unit/smoke tests | Conditional check: `hashFiles('package.json') != ''` |
| **Python Suite** | Sets up isolated Python runtime and triggers Pytest | Conditional check: `hashFiles('requirements.txt', 'pyproject.toml') != ''` |
| **Merge Orchestration** | Instructs the GitHub API to merge upon check completion | GitHub CLI: `gh pr merge "${{ github.event.pull_request.number }}" --auto --merge` |

---

## 🚀 Step-by-Step Repository Setup

Follow these exact steps to ensure the automation runs reliably without permission or status check errors:

### Step 1: Mark as a Template Repository
1. On GitHub, navigate to your repository overview: [kiran8688/PR-merging-automation](https://github.com/kiran8688/PR-merging-automation).
2. Click **Settings** in the top navigation bar.
3. Under the **General** tab (directly beneath the *Repository name* field), check the box:
   - [x] **Template repository**
4. *This activates the green **"Use this template"** button for quick inheritance on future projects.*

---

### Step 2: Commit the CI/CD Workflow
Ensure `.github/workflows/pr-review-automerge.yml` is present in your repository with the following definition:

```yaml
name: Automated PR Review & Merge

on:
  pull_request:
    types: [opened, synchronize, reopened]

permissions:
  contents: write
  pull-requests: write
  checks: read

jobs:
  validate-and-merge:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Setup Node.js
        if: hashFiles('package.json') != ''
        uses: actions/setup-node@v4
        with:
          node-version: 18

      - name: Run Tests (Node.js)
        if: hashFiles('package.json') != ''
        run: |
          npm ci || npm install
          npm test --if-present

      - name: Setup Python
        if: hashFiles('requirements.txt') != '' || hashFiles('pyproject.toml') != ''
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Run Tests (Python)
        if: hashFiles('requirements.txt') != '' || hashFiles('pyproject.toml') != ''
        run: |
          if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
          if command -v pytest &> /dev/null; then pytest; fi

      - name: Enable Auto-Merge
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          gh pr merge "${{ github.event.pull_request.number }}" --auto --merge \
            --subject "Auto-merged PR #${{ github.event.pull_request.number }}" || echo "Verify auto-merge settings in repository settings."
```

---

### Step 3: Enable Auto-Merge in Repository Settings
By default, GitHub disallows programmatic auto-merges on repositories. You must turn this on:
1. Navigate to **Settings** > **General**.
2. Scroll down to the **Pull Requests** heading.
3. Check the box for:
   - [x] **Allow auto-merge**
4. *(Optional but recommended)* Check **Automatically delete head branches** to keep repository branches clean after each merge.

---

### Step 4: Configure GitHub Actions Permissions
The GitHub Actions default token (`GITHUB_TOKEN`) must have write and PR management rights:
1. Navigate to **Settings** > **Actions** > **General**.
2. Scroll down to the **Workflow permissions** section.
3. Select **Read and write permissions**.
4. Check the box:
   - [x] **Allow GitHub Actions to create and approve pull requests**.
5. Click **Save**.

> ⚠️ **Important**: If this permission is omitted, GitHub CLI will reject the merge request with:
> `GraphQL: Resource not accessible by integration (enablePullRequestAutoMerge)`

---

### Step 5: Configure Branch Protection Rules
Branch protections prevent untested code from being merged prematurely:
1. Navigate to **Settings** > **Branches**.
2. Click **Add branch ruleset** (or **Add rule**).
3. Set **Branch name pattern** to `main`.
4. Enable:
   - [x] **Require status checks to pass before merging**
   - [x] **Require branches to be up to date before merging**
5. In the status checks search bar, find and select **validate-and-merge**.
6. Click **Create** / **Save changes**.

---

## 🛠 Creating New Projects from this Template

### Method A: Via GitHub Web Interface
1. Visit [kiran8688/PR-merging-automation](https://github.com/kiran8688/PR-merging-automation).
2. Click the green **Use this template** button at the top right > select **Create a new repository**.
3. Choose your repository name and visibility, then click **Create repository**.
4. Complete **Step 3** and **Step 4** in your new repository settings.

### Method B: Via GitHub CLI (`gh`)
Run this single command from your terminal to scaffold, clone, and configure your new repository:
```bash
gh repo create my-new-app \
  --template kiran8688/PR-merging-automation \
  --public \
  --enable-auto-merge \
  --clone
```

---

## 🔧 Customizing the Test Pipeline

The pipeline is intentionally modular. You can easily modify `.github/workflows/pr-review-automerge.yml` to support your preferred tooling:

### Using `uv` or `poetry` for Python
```yaml
      - name: Setup uv
        if: hashFiles('pyproject.toml') != ''
        uses: astral-sh/setup-uv@v3
      - name: Run Tests via uv
        if: hashFiles('pyproject.toml') != ''
        run: uv run pytest
```

### Using `pnpm` for Node.js
```yaml
      - name: Setup pnpm
        if: hashFiles('pnpm-lock.yaml') != ''
        uses: pnpm/action-setup@v3
        with:
          version: 9
      - name: Run Tests via pnpm
        if: hashFiles('pnpm-lock.yaml') != ''
        run: pnpm install && pnpm test --if-present
```

---

## 🔍 Troubleshooting & Verification

To verify that your automation functions end-to-end:

1. **Create a Test Branch**:
   ```bash
   git checkout -b test-automation
   echo "Automated CI verification" >> TEST.md
   git add TEST.md
   git commit -m "test: verify automated review and auto-merge"
   git push origin test-automation
   ```
2. **Open a Pull Request**:
   ```bash
   gh pr create --title "Test: verify automated PR merge" --body "Testing automated workflow execution"
   ```
3. **Monitor the Actions Tab**:
   * Navigate to the **Actions** tab in your repository.
   * Verify that the `validate-and-merge` job runs, dependencies install, tests pass, and the PR transitions automatically to **Merged**.
