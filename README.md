# Pipeline as a Service Stack

Reusable GitHub Actions workflow that centralizes Terraform validation across all infrastructure projects.

> **Version 1.0.0** | Production Ready

---

## What's Included

| Stage | Tool | Feature Flag | Description |
|-------|------|-------------|-------------|
| 1 | **terraform fmt** | Always | Code formatting (recursive) |
| 2 | **TFLint** | `enable_tflint` | Linting & best practices |
| 3 | **tfsec** | `enable_tfsec` | Security scanning (SARIF to GitHub Security tab) |
| 4 | **Checkov** | `enable_checkov` | Policy compliance (SARIF to GitHub Security tab) |
| 5 | **tf-cost** | `enable_tf_cost` | Cost estimation (placeholder for future implementation) |
| 6 | **terraform-docs** | `generate_tfdocs` | Documentation generation with PR comment on drift |
| 7 | **Validation Summary** | Always | Consolidated status report in GitHub Step Summary |

---

## Quick Start

### Basic Usage
```yaml
jobs:
  validate:
    uses: your-org/pipeline-as-a-service-stack/.github/workflows/pipeline-core.yaml@main
    with:
      terraform_dir: terraform
      terraform_version: '~1.9.0'
```

### Full Features
```yaml
jobs:
  validate:
    uses: your-org/pipeline-as-a-service-stack/.github/workflows/pipeline-core.yaml@main
    secrets: inherit
    permissions:
      contents: read
      pull-requests: write
      security-events: write
    with:
      terraform_dir: terraform
      terraform_version: '~1.9.0'
      enable_tflint: true
      enable_tfsec: true
      enable_checkov: true
      generate_tfdocs: true
      enable_tf_cost: false
      soft_fail: false
```

---

## Input Parameters

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `terraform_dir` | string | `terraform` | Terraform working directory |
| `terraform_version` | string | `~1.9.0` | Terraform version |
| `enable_tflint` | boolean | `true` | Run TFLint linting |
| `enable_tfsec` | boolean | `true` | Run tfsec security scan |
| `enable_checkov` | boolean | `true` | Run Checkov policy scan |
| `enable_tf_cost` | boolean | `false` | Cost estimation (coming soon) |
| `generate_tfdocs` | boolean | `true` | Generate terraform-docs and check drift |
| `soft_fail` | boolean | `false` | Continue pipeline on validation errors |

### Required Permissions

The caller workflow must grant:

```yaml
permissions:
  contents: read
  pull-requests: write
  security-events: write
```

---

## Pipeline Stages

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│ terraform    │───►│   TFLint     │───►│    tfsec     │───►│   Checkov    │
│    fmt       │    │  (optional)  │    │  (optional)  │    │  (optional)  │
└──────────────┘    └──────────────┘    └──────────────┘    └──────────────┘
                                                                    │
                    ┌──────────────┐    ┌──────────────┐            │
                    │  Validation  │◄───│ terraform    │◄───────────┘
                    │   Summary    │    │    docs      │
                    └──────────────┘    └──────────────┘
```

Each stage uses `continue-on-error: true` and reports status to the final **Validation Summary**, which consolidates all results into a GitHub Step Summary table.

### Caching

- Terraform providers cached via `.terraform.lock.hcl` hash
- TFLint plugins cached via `.tflint.hcl` hash

### SARIF Reports

Both **tfsec** and **Checkov** upload SARIF reports to GitHub Security tab, enabling:
- Security findings in the repository Security tab
- Code scanning alerts with line-level annotations

---

## Usage Example

### Multi-Environment Validation
```yaml
name: Validate All Environments

on:
  pull_request:
    paths: ['dev/**', 'staging/**', 'prod/**']

jobs:
  validate-dev:
    uses: your-org/pipeline-as-a-service-stack/.github/workflows/pipeline-core.yaml@main
    secrets: inherit
    with:
      terraform_dir: dev
      soft_fail: true

  validate-staging:
    uses: your-org/pipeline-as-a-service-stack/.github/workflows/pipeline-core.yaml@main
    secrets: inherit
    with:
      terraform_dir: staging
      soft_fail: true

  validate-prod:
    uses: your-org/pipeline-as-a-service-stack/.github/workflows/pipeline-core.yaml@main
    secrets: inherit
    with:
      terraform_dir: prod
      soft_fail: false
```

---

## Repository Structure

```
pipeline-as-a-service-stack/
├── .github/
│   └── workflows/
│       └── pipeline-core.yaml    # Reusable validation workflow
├── .gitignore
└── README.md
```

---

## Benefits

- Eliminates 70+ lines of duplicate validation code per project
- Centralized maintenance (one file to update for all consumers)
- Consistent validation across all infrastructure projects
- SARIF security reports integrated with GitHub Security tab
- Configurable per-project via feature flags
- Future-proof (ready for tf-cost, driftctl, sentinel, etc.)

---

## Support

**Issues?** Create an issue in this repository.

---

**Version**: 1.0.0
**Last Updated**: February 2026
