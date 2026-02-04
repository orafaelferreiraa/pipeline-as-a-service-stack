# 🚀 Pipeline Core Validation - Quick Start

## Overview

A **reusable GitHub Actions workflow** that centralizes Terraform validation across all infrastructure projects.

**Location**: `workflows/pipeline-core.yaml`  
**Status**: ✅ Production Ready  
**Version**: 1.0.0

---

## What's Included

✅ **terraform fmt** - Code formatting validation  
✅ **terraform validate** - Syntax validation  
✅ **TFLint** - Linting & best practices  
✅ **tfsec** - Security scanning (SARIF reports)  
✅ **Checkov** - Policy compliance (SARIF reports)  
✅ **terraform-docs** - Documentation generation  
✅ **tf-cost** - Placeholder for future implementation  

---

## Quick Start (30 seconds)

### Basic Usage
```yaml
jobs:
  validate:
    uses: your-org/pipeline-as-a-service-stack/workflows/pipeline-core.yaml@main
    with:
      terraform_dir: terraform
      terraform_version: 1.9.0
```

### With Full Features
```yaml
jobs:
  validate:
    uses: your-org/pipeline-as-a-service-stack/workflows/pipeline-core.yaml@main
    with:
      terraform_dir: terraform
      terraform_version: 1.9.0
      enable_tflint: true
      enable_tfsec: true
      enable_checkov: true
      generate_tfdocs: true
      soft_fail: false
    secrets:
      AZURE_CLIENT_ID: ${{ secrets.AZURE_CLIENT_ID }}
      AZURE_CLIENT_SECRET: ${{ secrets.AZURE_CLIENT_SECRET }}
      AZURE_TENANT_ID: ${{ secrets.AZURE_TENANT_ID }}
      AZURE_SUBSCRIPTION_ID: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
```

---

## Input Parameters

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `terraform_dir` | string | `terraform` | Terraform working directory |
| `terraform_version` | string | `~1.9.0` | Terraform version |
| `generate_tfdocs` | boolean | `true` | Generate documentation |
| `enable_tflint` | boolean | `true` | Run TFLint |
| `enable_tfsec` | boolean | `true` | Run tfsec |
| `enable_checkov` | boolean | `true` | Run Checkov |
| `enable_tf_cost` | boolean | `false` | Cost estimation (coming soon) |
| `soft_fail` | boolean | `false` | Continue on errors |

---

## Real Example

### Multi-Tenant Validation
```yaml
name: Validate All Environments

on:
  pull_request:
    paths: ['dev/**', 'staging/**', 'prod/**']

jobs:
  validate-dev:
    uses: your-org/pipeline-as-a-service-stack/workflows/pipeline-core.yaml@main
    with:
      terraform_dir: dev
      soft_fail: true

  validate-staging:
    uses: your-org/pipeline-as-a-service-stack/workflows/pipeline-core.yaml@main
    with:
      terraform_dir: staging
      soft_fail: true

  validate-prod:
    uses: your-org/pipeline-as-a-service-stack/workflows/pipeline-core.yaml@main
    with:
      terraform_dir: prod
      soft_fail: false
```

---

## Benefits

- ✅ Eliminates 70+ lines of duplicate validation code
- ✅ Centralized maintenance (one file to update)
- ✅ Consistent validation across all projects
- ✅ Future-proof (ready for tf-cost, driftctl, etc.)
- ✅ SARIF security reports to GitHub Security tab
- ✅ Configurable per-project

---

## Documentation

- 📖 [GETTING-STARTED.md](docs/GETTING-STARTED.md) - 5-minute setup guide
- 📚 [COMPLETE-GUIDE.md](docs/COMPLETE-GUIDE.md) - Full reference
- 🔄 [ADOPTION-GUIDE.md](docs/ADOPTION-GUIDE.md) - Migration path
- 💡 [EXAMPLES.md](docs/EXAMPLES.md) - Real-world examples

---

## Support

**Questions?** See the documentation files above.

**Issues?** Create an issue in this repository.

---

**Ready to use!** Start with the basic example above and customize as needed.
