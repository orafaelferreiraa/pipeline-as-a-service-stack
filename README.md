# Pipeline as a Service Stack

Reusable GitHub Actions workflow (`workflow_call`) that centralizes Terraform validation across all infrastructure projects. Consumers invoke a single workflow reference — all stages, caching, SARIF reports, and summary reporting are handled internally.

> **Version 1.0.0** | Production Ready

---

## What's Included

| Stage | Tool | Feature Flag | Description |
|-------|------|-------------|-------------|
| 1 | **terraform fmt** | Always | Code formatting (recursive) |
| 2 | **TFLint** | `enable_tflint` | Linting & best practices |
| 3 | **tfsec** | `enable_tfsec` | Security scanning (SARIF to GitHub Security tab) |
| 4 | **Checkov** | `enable_checkov` | Policy compliance (SARIF to GitHub Security tab) |
| 5 | **tf-cost** | `enable_tf_cost` | Cost estimation (infracost / cloud.tf — placeholder) |
| 6 | **terraform-docs** | `generate_tfdocs` | Documentation generation with drift detection + PR comment |
| 7 | **Validation Summary** | Always | Consolidated status table in GitHub Step Summary |

---

## Quick Start

### Basic Usage
```yaml
jobs:
  validate:
    uses: orafaelferreiraa/pipeline-as-a-service-stack/.github/workflows/pipeline-core.yaml@main
    with:
      terraform_dir: terraform
      terraform_version: '~1.9.0'
```

### Full Features
```yaml
jobs:
  validate:
    uses: orafaelferreiraa/pipeline-as-a-service-stack/.github/workflows/pipeline-core.yaml@main
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
┌──────────────┐    ┌──────────────┐    ┌──────────────┐            │
│  Validation  │◄───│ terraform    │◄───│   tf-cost    │◄───────────┘
│   Summary    │    │    docs      │    │  (optional)  │
└──────────────┘    └──────────────┘    └──────────────┘
```

Stages run **sequentially** with `continue-on-error: true` and `if: always()` gates — every stage executes regardless of prior failures. The final **Validation Summary** consolidates all outcomes into a GitHub Step Summary table.

### Validation Summary Output

The summary step generates a table in GitHub Step Summary:

| Check | Result |
|-------|--------|
| Format | ✅ |
| TFLint | ✅ |
| tfsec | ✅ |
| Checkov | ⊘ (skipped) |
| Docs | ✅ |

- **✅** — passed
- **❌** — failed
- **⊘** — skipped (feature flag disabled)

### Soft Fail Behavior

When `soft_fail: true`, the pipeline reports failures as **warnings** but exits with code 0 — the calling workflow continues. When `soft_fail: false` (default), any failure causes the summary step to `exit 1`, blocking the caller.

### Caching

| Cache | Key | Path |
|-------|-----|------|
| Terraform providers | `.terraform.lock.hcl` hash | `$terraform_dir/.terraform`, `~/.terraform.d/plugin-cache` |
| TFLint plugins | `.tflint.hcl` hash | `~/.tflint.d/plugins` |

### SARIF Reports

Both **tfsec** and **Checkov** upload SARIF reports to GitHub Security tab:

| Tool | SARIF Category | Action |
|------|---------------|--------|
| tfsec | `tfsec` | `github/codeql-action/upload-sarif@v4` |
| Checkov | `checkov` | `github/codeql-action/upload-sarif@v4` |

Enables:
- Security findings in the repository Security tab
- Code scanning alerts with line-level annotations

### Terraform Docs — Drift Detection

When `generate_tfdocs: true`, the stage:
1. Renders documentation via `terraform-docs/gh-actions@v1.3.0` (inject mode into `README.md`)
2. Checks `git diff` for changes
3. If drift detected on a PR, posts a comment with instructions to regenerate locally

---

## Usage Examples

### Multi-Environment Validation
```yaml
name: Validate All Environments

on:
  pull_request:
    paths: ['dev/**', 'staging/**', 'prod/**']

jobs:
  validate-dev:
    uses: orafaelferreiraa/pipeline-as-a-service-stack/.github/workflows/pipeline-core.yaml@main
    secrets: inherit
    with:
      terraform_dir: dev
      soft_fail: true

  validate-staging:
    uses: orafaelferreiraa/pipeline-as-a-service-stack/.github/workflows/pipeline-core.yaml@main
    secrets: inherit
    with:
      terraform_dir: staging
      soft_fail: true

  validate-prod:
    uses: orafaelferreiraa/pipeline-as-a-service-stack/.github/workflows/pipeline-core.yaml@main
    secrets: inherit
    with:
      terraform_dir: prod
      soft_fail: false  # Block merge on failure
```

### Platform as a Service Stack Integration
```yaml
name: Validate Platform Stack

on:
  pull_request:
    branches: [main]
    paths: ['terraform/**']

jobs:
  validate:
    uses: orafaelferreiraa/pipeline-as-a-service-stack/.github/workflows/pipeline-core.yaml@main
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
```

### Security-Only Scan
```yaml
jobs:
  security:
    uses: orafaelferreiraa/pipeline-as-a-service-stack/.github/workflows/pipeline-core.yaml@main
    secrets: inherit
    permissions:
      contents: read
      security-events: write
    with:
      terraform_dir: terraform
      enable_tflint: false
      enable_tfsec: true
      enable_checkov: true
      generate_tfdocs: false
```

---

## Repository Structure

```
pipeline-as-a-service-stack/
├── .github/
│   └── workflows/
│       └── pipeline-core.yaml    # Reusable validation workflow (workflow_call)
├── .gitignore
└── README.md
```

---

## Toolchain & Versions

| Tool | Version | Purpose |
|------|---------|---------|
| Terraform | `~1.9.0` (configurable) | `fmt`, provider caching |
| TFLint | `latest` | Linting & best practices |
| tfsec | `v1.0.3` (action) | Security scanning → SARIF |
| Checkov | latest (pip) | Policy compliance → SARIF |
| terraform-docs | `v1.3.0` (action) | Documentation drift detection |
| Python | `3.12` | Checkov runtime |
| GitHub Actions | `actions/checkout@v4`, `actions/cache@v4`, `actions/upload-artifact@v4` | Core actions |

---

## Benefits

- **Eliminates 70+ lines** of duplicate validation code per project
- **Centralized maintenance** — one file to update for all consumers
- **Consistent validation** across all infrastructure projects
- **SARIF security reports** integrated with GitHub Security tab
- **Feature-flag driven** — enable/disable stages per project
- **Soft fail mode** — non-blocking validation for development environments
- **Drift detection** — terraform-docs checks and alerts on PR when docs are out of date
- **Future-proof** — ready for tf-cost (infracost, cloud.tf), driftctl, sentinel, etc.

---

## Related Stacks

| Stack | Description |
|-------|-------------|
| [platform-as-a-service-stack](../platform-as-a-service-stack/) | Azure infrastructure platform (Terraform) |
| [tfmodules-as-a-service-stack](../tfmodules-as-a-service-stack/) | Reusable Terraform modules |

---

## Support

**Issues?** Create an issue in this repository.

---

**Version**: 1.0.0  
**Last Updated**: February 2026
