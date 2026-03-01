# 🛠️ Centralized DevOps Actions Library

This repository serves as the **single source of truth** for reusable GitHub Actions workflows across the organization. It is designed to standardize CI/CD processes, including infrastructure management, containerization, and application deployment.

## 🏗️ Available Workflow Categories

### ☁️ Infrastructure as Code (Terragrunt/Terraform)

Standardized lifecycle management for AWS infrastructure using Terragrunt.

* **Plan & Cost:** Validation, HCL formatting, and cost estimation via Infracost.
* **Apply:** Secure deployment using plan artifacts to ensure consistency.
* **Drift Detection:** Automated monitoring of infrastructure state with GitHub Issue integration.
* **Destroy:** Controlled decommissioning with manual approval gates and automatic cleanup PRs.

### 🐳 Containerization & Registry (Coming Soon)

* **Docker Build & Push:** Standardized multi-arch builds and security scanning.
* **Image Tagging:** Logic for semantic versioning and commit-based tags.

### ☸️ Kubernetes & Orchestration (Coming Soon)

* **Kubernetes Deploy:** Reusable manifests or Helm chart deployments.
* **Config Validation:** Pre-deploy linting and security checks.

### 🧪 Quality Gates & Testing (Coming Soon)

* **Code Analysis:** Integration with tools like SonarQube.
* **Automated Testing:** Unit, integration, and E2E test execution.

## 🛠 Prerequisites & Security

### 🔐 Authentication & IAM

Most workflows in this repository utilize **OIDC (OpenID Connect)** to authenticate with AWS securely, eliminating the need for persistent access keys.

* **AWS OIDC Role**: You must configure the `AWS_ACTIONS_ROLE_ARN` secret in the calling repository to allow the workflow to assume the necessary IAM role.
* **Permissions**: The assumed role must have sufficient permissions to manage infrastructure resources, post PR comments, or create GitHub Issues when drift is detected.

### 📢 Notifications

A centralized Slack notification system is integrated across all workflows to provide immediate feedback on job execution.

* **Slack Webhook**: The `SLACK_WEBHOOK_URL` secret is required at the organization or repository level.
* **Real-time Feedback**: Notifications include visual status indicators (Success, Failure, Cancelled), direct links to workflow logs, and commit details.

---

### 📦 Requirements Table

| Secret / Input | Description | Scope |
| :--- | :--- | :--- |
| `AWS_ACTIONS_ROLE_ARN` | IAM Role ARN for OIDC authentication. | Infrastructure / K8S |
| `SLACK_WEBHOOK_URL` | (Optional) Webhook URL for sending alerts to Slack channels. | Global |
| `GITHUB_TOKEN` | Automatic token for GitHub API interactions (PRs, Issues). | Global |
| `INFRACOST_API_KEY` | (Optional) API Key for cost estimation in Terragrunt plans. | Infrastructure |

### 🛡️ Security Best Practices

1. **No Hardcoded Secrets**: Never include credentials in code; always use `secrets: inherit` when calling workflows.
2. **Environment Protection**: Destructive actions (such as `terragrunt-destroy`) require manual approval through protected GitHub Environments.
3. **Artifact Validation**: Deployments rely on downloading and validating previously approved plan artifacts to prevent "last-minute" unauthorized changes between the plan and apply phases.

---

## 📲 How to Use

All workflows are designed to be triggered via `workflow_call` from other repositories.

### Example: Calling a Workflow

```yaml
jobs:
  set-env:
    name: Set Environment
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

  terragrunt-plan:
    name: Terragrunt Plan Workflow
    needs: set-env
    uses: darthfabax-org/actions/.github/workflows/terragrunt-plan.yml@main
    secrets: inherit

  terragrunt-apply:
    name: Terragrunt Apply Workflow
    needs: [terragrunt-plan, set-env]
    if: needs.terragrunt-plan.outputs.has_changes == 'true'
    uses: darthfabax-org/actions/.github/workflows/terragrunt-apply.yml@main
    strategy:
      fail-fast: false
      matrix:
        directory: ${{ fromJSON(needs.terragrunt-plan.outputs.matrix) }}
    with:
      working_directory: ${{ matrix.directory }}
    secrets: inherit
```

## ⚖️ License

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
