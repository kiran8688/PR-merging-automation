# PR-merging-automation

A standardized project template equipped with automated pull request validation, multi-environment test execution, and native GitHub auto-merging.

---

## Overview

This repository serves as a baseline template for new projects. It includes a pre-configured GitHub Actions workflow (`.github/workflows/pr-review-automerge.yml`) that automatically evaluates incoming pull requests, runs relevant test suites based on detected project files, and enables GitHub's native auto-merge once all checks pass.

### How the Automation Works

1. **Trigger**: Executes on every `pull_request` event (`opened`, `synchronize`, `reopened`).
2. **Environment Detection & Testing**:
   - **Node.js**: Detects `package.json`, installs dependencies via `npm ci` (or `npm install`), and executes `npm test --if-present`.
   - **Python**: Detects `requirements.txt` or `pyproject.toml`, sets up Python 3.11, installs dependencies, and runs `pytest` if installed.
3. **Automated Merging**: Leverages the GitHub CLI (`gh pr merge --auto --merge`) using the repository's built-in `GITHUB_TOKEN`. Once all required checks and protections are satisfied, GitHub merges the PR into the target base branch automatically.

---

## Required GitHub Repository Settings

To allow the GitHub Actions runner to merge pull requests automatically, the following repository settings must be enabled on GitHub:

### 1. Enable Auto-Merge
1. Go to your repository on GitHub.
2. Navigate to **Settings** > **General**.
3. Scroll down to the **Pull Requests** section.
4. Check the box for **Allow auto-merge**.

### 2. Configure GitHub Actions Workflow Permissions
1. Navigate to **Settings** > **Actions** > **General**.
2. Scroll down to **Workflow permissions**.
3. Select **Read and write permissions** (required for GitHub Actions to write code and merge branches).
4. Check the box for **Allow GitHub Actions to create and approve pull requests**.
5. Click **Save**.

---

## Getting Started

### Creating a New Repository from this Template

#### Option A: Via GitHub Web Interface
1. Navigate to [kiran8688/PR-merging-automation](https://github.com/kiran8688/PR-merging-automation).
2. Click the green **Use this template** button at the top right, then select **Create a new repository**.
3. Enter your new repository name, select visibility (Public or Private), and click **Create repository**.
4. Follow the steps in [Required GitHub Repository Settings](#required-github-repository-settings) above.

#### Option B: Via GitHub CLI (`gh`)
Run the following command in your terminal:
```bash
gh repo create <new-repo-name> \
  --template kiran8688/PR-merging-automation \
  --public \
  --enable-auto-merge \
  --clone
```

---

## Recommended Branch Protection Rules

To prevent broken or untested code from merging automatically, configuring a branch protection rule on your default branch (`main` or `master`) is strongly recommended:

1. Navigate to **Settings** > **Branches**.
2. Under **Branch protection rules**, click **Add branch ruleset** or **Add rule**.
3. Set the target branch pattern to `main`.
4. Enable **Require status checks to pass before merging**.
5. Select `validate-and-merge` as a required status check.

With this protection enabled, GitHub holds the PR until the CI test job completes with a passing status before executing the auto-merge.

---

## Customizing the Test Pipeline

The automated workflow is located at `.github/workflows/pr-review-automerge.yml`. You can adapt the test runner steps to match your specific toolchain:

- **Alternative Package Managers**: Replace `npm ci` with `pnpm install` or `yarn install`.
- **Python Tooling**: Replace standard `pip` with `uv` or `poetry`.
- **Other Languages**: Add conditional setup steps for Go, Rust, or Java as needed.
