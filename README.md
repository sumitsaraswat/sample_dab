# sample_dab

Databricks Asset Bundle with CI/CD pipeline for automated deployment to multiple environments.

## 📋 Table of Contents

- [Overview](#overview)
- [Project Structure](#project-structure)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [CI/CD Pipeline](#cicd-pipeline)
- [Deployment Environments](#deployment-environments)
- [Usage](#usage)
- [Troubleshooting](#troubleshooting)

---

## 🎯 Overview

This project implements a **Databricks Asset Bundle (DAB)** with a complete **CI/CD pipeline** using GitHub Actions. It supports automated deployment to multiple environments with validation gates and approval workflows.

### Features

- ✅ **Automated CI/CD** - GitHub Actions workflow for continuous integration and deployment
- ✅ **Multiple Environments** - Support for dev and production (West US) workspaces
- ✅ **Validation Gates** - Automatic bundle validation on pull requests
- ✅ **Approval Workflow** - Manual approval required for production deployments
- ✅ **GitOps** - Infrastructure and configuration as code in Git
- ✅ **Release Tagging** - Automatic tagging for production releases

---

## 📁 Project Structure

```
sample_dab/
├── .github/
│   └── workflows/
│       └── databricks-cicd.yml    # CI/CD pipeline configuration
├── resources/
│   └── sample_dab.job.yml         # Databricks job definitions
├── src/
│   └── notebook.ipynb             # Databricks notebooks
├── fixtures/                       # Test fixtures
├── scratch/                        # Scratch/exploration notebooks
├── databricks.yml                 # Bundle configuration
├── .gitignore                     # Git ignore rules
└── README.md                      # This file
```

---

## 🔧 Prerequisites

### Local Development

1. **Databricks CLI**
   ```bash
   curl -fsSL https://raw.githubusercontent.com/databricks/setup-cli/main/install.sh | sh
   ```

2. **Authentication**
   ```bash
   databricks configure
   ```

3. **Git**
   ```bash
   git clone https://github.com/sumitsaraswat/sample_dab.git
   cd sample_dab
   ```

### CI/CD Setup (One-time)

1. **GitHub Secrets** - Add these secrets to your repository:
   - Go to: `Settings` → `Secrets and variables` → `Actions`
   - Add the following secrets:
   
   | Secret Name | Description | Example |
   |-------------|-------------|---------|
   | `DATABRICKS_HOST` | Dev workspace URL | `https://adb-984752964297111.11.azuredatabricks.net` |
   | `DATABRICKS_TOKEN` | Dev workspace token | `dapic558...` |
   | `WESTUS_HOST` | West US workspace URL | `https://westus.azuredatabricks.net` |
   | `WESTUS_TOKEN` | West US workspace token | `dapid6dd...` |

2. **GitHub Environments** (Optional but recommended for production)
   - Go to: `Settings` → `Environments`
   - Create environment: `dev`
   - Create environment: `westus-production`
   - For `westus-production`, add protection rules:
     - ✅ Required reviewers (add yourself or team members)
     - ✅ Wait timer (optional, e.g., 5 minutes)

3. **IP Access Lists** (Important!)
   - GitHub Actions runners need API access to Databricks workspaces
   - Either:
     - **Option A**: Temporarily disable IP Access Lists (for testing)
     - **Option B**: Add GitHub Actions IP ranges to your Databricks IP Access Lists
   
   Get GitHub Actions IPs:
   ```bash
   curl https://api.github.com/meta | jq -r '.actions[]'
   ```

---

## 🚀 Getting Started

### Local Development

1. **Deploy to Dev**
   ```bash
   databricks bundle deploy --target dev
   ```

2. **Run the Job**
   ```bash
   databricks bundle run sample_dab_job --target dev
   ```

3. **Validate Changes**
   ```bash
   databricks bundle validate --target dev
   ```

4. **View Deployment Summary**
   ```bash
   databricks bundle summary --target dev
   ```

### Making Changes

1. **Create a feature branch**
   ```bash
   git checkout -b feature/my-changes
   ```

2. **Make your changes**
   - Edit notebooks in `src/`
   - Update job definitions in `resources/`
   - Modify bundle config in `databricks.yml`

3. **Commit and push**
   ```bash
   git add .
   git commit -m "Description of changes"
   git push origin feature/my-changes
   ```

4. **Create a Pull Request**
   - Go to: https://github.com/sumitsaraswat/sample_dab
   - Click "Compare & pull request"
   - CI will automatically validate your changes

---

## 🔄 CI/CD Pipeline

### Pipeline Overview

```
Pull Request → CI Validation → Merge → CD Deployment (Dev) → Manual Deployment (Prod)
```

### Workflow Triggers

| Event | Trigger | What Happens |
|-------|---------|--------------|
| **Pull Request** | Create/update PR to `main` | CI validation runs |
| **Push to Main** | Merge PR or direct push | Automatic deployment to dev |
| **Manual Trigger** | Workflow dispatch | Manual deployment to production |

### CI: Continuous Integration (Pull Request)

**When:** Pull request is created or updated

**What it does:**
1. ✅ Validate bundle configuration
2. ✅ Check project structure
3. ✅ Report status on PR

**Result:**
- ✅ **Pass** - PR can be merged
- ❌ **Fail** - PR cannot be merged (fix errors first)

### CD: Continuous Deployment (Automatic)

**When:** Code is merged to `main` branch

**What it does:**
1. ✅ Validate bundle
2. ✅ Deploy to dev workspace
3. ✅ Update Databricks jobs
4. ✅ Report deployment status

**Result:**
- Changes are automatically live in dev environment
- Job name: `[dev <username>] sample_dab_job`

### CD: Continuous Deployment (Manual - Production)

**When:** Manually triggered via GitHub Actions

**What it does:**
1. ✅ Skip validation (to avoid IP ACL issues)
2. ⏸️ Wait for manual approval
3. ✅ Deploy to West US production workspace
4. ✅ Update Databricks jobs
5. ✅ Create release tag

**How to trigger:**
1. Go to: https://github.com/sumitsaraswat/sample_dab/actions
2. Click: "Databricks DAB CI/CD"
3. Click: "Run workflow"
4. Select:
   - **Branch:** `main`
   - **Environment:** `westus_prod`
5. Click: "Run workflow"
6. Wait for "Review deployments" button
7. Click: "Review deployments"
8. Approve the deployment

**Result:**
- Changes are deployed to West US production
- Job name: `sample_dab_job` (no dev prefix)
- Release tag created: `westus-release-YYYYMMDD-HHMMSS`

---

## 🌍 Deployment Environments

### Dev Environment

- **Workspace:** https://adb-984752964297111.11.azuredatabricks.net/
- **Mode:** Development
- **Deployment:** Automatic (on merge to main)
- **Job Prefix:** `[dev <username>]`
- **Schedule:** Paused (dev mode)
- **Path:** `/Workspace/Users/sumit.saraswat@databricks.com/.bundle/sample_dab/dev`

### West US Production Environment

- **Workspace:** https://westus.azuredatabricks.net/
- **Mode:** Production
- **Deployment:** Manual (with approval)
- **Job Prefix:** None
- **Schedule:** Active
- **Path:** `/Workspace/Users/sumit.saraswat@databricks.com/.bundle/sample_dab/westus_prod`

---

## 📖 Usage

### Scenario 1: Fix a Bug

```bash
# 1. Create feature branch
git checkout -b feature/fix-bug

# 2. Fix the bug in your notebook
# Edit src/notebook.ipynb

# 3. Test locally
databricks bundle deploy --target dev
databricks bundle run sample_dab_job --target dev

# 4. Commit and push
git add .
git commit -m "Fix: Resolve data processing bug"
git push origin feature/fix-bug

# 5. Create PR on GitHub
# CI will validate your changes

# 6. Merge PR after approval
# CD automatically deploys to dev

# 7. Verify in dev workspace
# Check job runs successfully

# 8. Deploy to production (when ready)
# Use manual workflow trigger
```

### Scenario 2: Add a New Notebook

```bash
# 1. Create feature branch
git checkout -b feature/new-notebook

# 2. Add new notebook
# Create src/new_notebook.ipynb

# 3. Update job definition
# Edit resources/sample_dab.job.yml
# Add new task referencing the notebook

# 4. Test locally
databricks bundle deploy --target dev
databricks bundle run sample_dab_job --target dev

# 5. Push and create PR
git add .
git commit -m "Add new notebook for data analysis"
git push origin feature/new-notebook

# 6. CI validates, merge PR
# 7. CD deploys to dev automatically
```

### Scenario 3: Production Release

```bash
# 1. Ensure all changes are tested in dev
# Verify job runs successfully

# 2. Go to GitHub Actions
# https://github.com/sumitsaraswat/sample_dab/actions

# 3. Click "Databricks DAB CI/CD" → "Run workflow"

# 4. Select:
#    - Branch: main
#    - Environment: westus_prod

# 5. Click "Run workflow"

# 6. Approve when prompted

# 7. Verify in production workspace
# Check https://westus.azuredatabricks.net/

# 8. Note the release tag created
# Example: westus-release-20250115-143022
```

---

## 🔍 Troubleshooting

### Issue: CI Validation Fails with IP Access Error

**Error:**
```
Source IP address: X.X.X.X is blocked by Databricks IP ACL
```

**Solution:**
1. Go to your Databricks workspace
2. Navigate to: `Settings` → `IP Access Lists`
3. Either:
   - Disable IP Access List (temporary, for testing)
   - Add GitHub Actions IP ranges to allow list

### Issue: Profile Not Found Error

**Error:**
```
/home/runner/.databrickscfg has no westus profile configured
```

**Solution:**
- Ensure `databricks.yml` does not specify a profile for `westus_prod` target
- The workflow creates a `[DEFAULT]` profile dynamically
- Remove any `profile: westus` line from `databricks.yml`

### Issue: Deployment Fails - Authentication Error

**Error:**
```
Error: cannot parse config file
```

**Solution:**
1. Verify GitHub Secrets are set correctly:
   - `DATABRICKS_HOST` - Should be full URL with `https://`
   - `DATABRICKS_TOKEN` - Should be valid personal access token
2. Check secrets at: https://github.com/sumitsaraswat/sample_dab/settings/secrets/actions

### Issue: Manual Workflow Not Triggering

**Problem:**
"Run workflow" button doesn't appear

**Solution:**
1. Ensure you're on the "Actions" tab
2. Click "Databricks DAB CI/CD" in the left sidebar
3. The "Run workflow" button should appear on the right
4. If not, check if the workflow file is on the `main` branch

### Issue: Approval Not Required for Production

**Problem:**
Production deploys without approval

**Solution:**
1. Go to: `Settings` → `Environments`
2. Click on `westus-production`
3. Add "Required reviewers" protection rule
4. Add yourself or team members as reviewers

---

## 📚 Additional Resources

- **Databricks Asset Bundles Documentation:** https://docs.databricks.com/dev-tools/bundles/
- **GitHub Actions Documentation:** https://docs.github.com/en/actions
- **Databricks CLI:** https://docs.databricks.com/dev-tools/cli/
- **CI/CD Best Practices:** https://docs.databricks.com/dev-tools/bundles/ci-cd.html

---

## 🎯 Key Concepts

### CI (Continuous Integration)
Automatically test and validate code changes before merging to ensure code quality.

### CD (Continuous Deployment)
Automatically deploy validated code to target environments without manual intervention.

### GitOps
Use Git as the single source of truth for infrastructure and application configuration.

### Approval Workflow
Manual gate that requires human approval before deploying to production environments.

### Release Tagging
Create Git tags to mark specific commits that were deployed to production for audit trail.

---

## 📝 Notes

- **Development Mode:** Jobs have a `[dev <username>]` prefix and schedules are paused
- **Production Mode:** Jobs use actual name without prefix and schedules are active
- **IP Access Lists:** Ensure GitHub Actions IPs are allowed in Databricks workspaces
- **Authentication:** Uses Databricks personal access tokens stored as GitHub Secrets
- **Environments:** Separate workspaces for dev and production isolation

---

## 🤝 Contributing

1. Create a feature branch
2. Make your changes
3. Create a pull request
4. Wait for CI validation
5. Get code review approval
6. Merge to main

---

## 📄 License

This project was generated using the Databricks default-python template.

---

## 🆘 Support

For issues or questions:
- Check the [Troubleshooting](#troubleshooting) section
- Review [Databricks documentation](https://docs.databricks.com/dev-tools/bundles/)
- Check [GitHub Actions logs](https://github.com/sumitsaraswat/sample_dab/actions)

---

**Built with ❤️ using Databricks Asset Bundles and GitHub Actions**
