# GitHub Action: Terraform Destroy and Summary

![Release](https://github.com/subhamay-bhattacharyya-gha/tf-destroy-action/actions/workflows/release.yaml/badge.svg)&nbsp;![Commit Activity](https://img.shields.io/github/commit-activity/t/subhamay-bhattacharyya-gha/tf-destroy-action)&nbsp;![Last Commit](https://img.shields.io/github/last-commit/subhamay-bhattacharyya-gha/tf-destroy-action)&nbsp;![Release Date](https://img.shields.io/github/release-date/subhamay-bhattacharyya-gha/tf-destroy-action)&nbsp;![Repo Size](https://img.shields.io/github/repo-size/subhamay-bhattacharyya-gha/tf-destroy-action)&nbsp;![File Count](https://img.shields.io/github/directory-file-count/subhamay-bhattacharyya-gha/tf-destroy-action)&nbsp;![Issues](https://img.shields.io/github/issues/subhamay-bhattacharyya-gha/tf-destroy-action)&nbsp;![Top Language](https://img.shields.io/github/languages/top/subhamay-bhattacharyya-gha/tf-destroy-action)&nbsp;![Custom Endpoint](https://img.shields.io/endpoint?url=https://gist.githubusercontent.com/bsubhamay/e15bf74830c9bea324a8b87e1f11a2fb/raw/tf-destroy-action.json?)

A GitHub Composite Action to safely destroy Terraform-managed infrastructure with **multi-cloud support** (AWS, Azure, GCP, Databricks, Snowflake) and multiple backend options (S3, HCP Terraform Cloud). Displays a clear, timestamped summary in the GitHub Actions Step Summary.

---

## 🔧 Features

- **Multi-Cloud Support**: Works with AWS, Azure, GCP, Databricks, and Snowflake
- **Multiple Backend Options**: Supports AWS S3 and HCP Terraform Cloud backends
- **OIDC Authentication**: Secure authentication using Workload Identity Federation (AWS, Azure, GCP)
- **Flexible Authentication**: Databricks and Snowflake use environment variables set by caller workflow
- **Input Validation**: Comprehensive validation with clear error messages
- **Dynamic Backend Configuration**: Computes backend keys based on CI pipeline context
- **Destroy Planning**: Creates and analyzes destroy plans via JSON output
- **Comprehensive Logging**: Full terraform destroy logging with debug output
- **Summary Reports**: Detailed destruction summary in GitHub Step Summary with:
  - Resource Type
  - Resource Name  
  - Address
  - Timestamp

---

## 📥 Inputs

### Core Configuration

| Name              | Description                                                         | Required | Default | Type   |
|-------------------|---------------------------------------------------------------------|----------|---------|--------|
| `tf-config-path`  | Relative path from repository root to Terraform configuration directory | No       | `tf`    | string |
| `release-tag`     | Git release tag to check out (uses latest commit if omitted)       | No       | `""`    | string |
| `ci-pipeline`     | Include commit SHA in state key for CI/CD pipelines                | No       | `false` | string |

### Backend Configuration

| Name              | Description                                                         | Required | Default | Type   |
|-------------------|---------------------------------------------------------------------|----------|---------|--------|
| `backend-type`    | Backend type: `s3` for AWS S3 or `remote` for HCP Terraform Cloud  | No       | `s3`    | string |
| `cloud-provider`  | Target cloud provider or platform: `aws`, `azure`, `gcp`, `databricks`, `snowflake`, or `platform` | **Yes**  | —       | string |

### AWS S3 Backend (when `backend-type: s3`)

| Name              | Description                                                         | Required | Default | Type   |
|-------------------|---------------------------------------------------------------------|----------|---------|--------|
| `s3-bucket`       | S3 bucket name for Terraform state storage                         | No*      | —       | string |
| `s3-region`       | AWS region where S3 bucket is located                              | No*      | —       | string |
| `s3-key-prefix`   | Optional prefix for S3 state key                                   | No       | `""`    | string |

*Required when `backend-type` is `s3`

### HCP Terraform Cloud Backend (when `backend-type: remote`)

| Name              | Description                                                         | Required | Default | Type   |
|-------------------|---------------------------------------------------------------------|----------|---------|--------|
| `tfc-token`       | HCP Terraform Cloud API token (pass as secret)                     | No*      | —       | string |

*Required when `backend-type` is `remote`

### AWS Authentication (when `cloud-provider: aws`)

| Name                 | Description                                                      | Required | Default | Type   |
|----------------------|------------------------------------------------------------------|----------|---------|--------|
| `aws-region`         | AWS region for authentication                                    | No*      | —       | string |
| `aws-role-to-assume` | AWS IAM role ARN to assume via OIDC                            | No*      | —       | string |

*Required when `cloud-provider` is `aws` or when using S3 backend

### Azure Authentication (when `cloud-provider: azure`)

| Name                    | Description                                                   | Required | Default | Type   |
|-------------------------|---------------------------------------------------------------|----------|---------|--------|
| `azure-client-id`       | Azure client ID for authentication                           | No*      | —       | string |
| `azure-tenant-id`       | Azure tenant ID for authentication                           | No*      | —       | string |
| `azure-subscription-id` | Azure subscription ID for authentication                     | No*      | —       | string |

*Required when `cloud-provider` is `azure`

### GCP Authentication (when `cloud-provider: gcp`)

| Name                  | Description                                                     | Required | Default | Type   |
|-----------------------|-----------------------------------------------------------------|----------|---------|--------|
| `gcp-wif-provider`    | GCP Workload Identity Federation provider                      | No*      | —       | string |
| `gcp-service-account` | GCP service account email for authentication                   | No*      | —       | string |

*Required when `cloud-provider` is `gcp`

### Databricks Authentication (when `cloud-provider: databricks`)

Databricks authentication should be configured via environment variables in the caller workflow before invoking this action.

| Environment Variable | Description                                                     |
|---------------------|------------------------------------------------------------------|
| `DATABRICKS_HOST`   | Databricks workspace URL (e.g., `https://dbc-xxxxx.cloud.databricks.com`) |
| `DATABRICKS_TOKEN`  | Databricks personal access token                                 |

### Snowflake Authentication (when `cloud-provider: snowflake`)

Snowflake authentication should be configured via environment variables in the caller workflow before invoking this action.

| Environment Variable         | Description                                              |
|-----------------------------|----------------------------------------------------------|
| `SNOWFLAKE_ACCOUNT`         | Snowflake account identifier (e.g., `xy12345.us-east-1`) |
| `SNOWFLAKE_USER`            | Snowflake user name                                      |
| `SNOWFLAKE_ROLE`            | Snowflake role name                                      |
| `SNOWFLAKE_PRIVATE_KEY_PATH`| Path to Snowflake private key file                       |

### Platform Mode (when `cloud-provider: platform`)

When using `platform` mode, the action automatically detects which cloud provider directories exist in your `infra/` folder and validates only the required inputs for those providers.

| Provider Directory | Required Inputs / Environment Variables |
|-------------------|----------------------------------------|
| `infra/aws`       | `aws-region`, `aws-role-to-assume` (action inputs) |
| `infra/azure`     | `azure-client-id`, `azure-tenant-id`, `azure-subscription-id` (action inputs) |
| `infra/gcp`       | `gcp-wif-provider`, `gcp-service-account` (action inputs) |
| `infra/databricks`| `DATABRICKS_HOST`, `DATABRICKS_TOKEN` (environment variables) |
| `infra/snowflake` | `SNOWFLAKE_ACCOUNT`, `SNOWFLAKE_USER`, `SNOWFLAKE_ROLE`, `SNOWFLAKE_PRIVATE_KEY_PATH` (environment variables) |

> **Note:** When using S3 backend with platform mode, AWS credentials are always required for backend access.

---

## 🚀 Example Usage

### AWS with S3 Backend

```yaml
name: Terraform Destroy - AWS S3 Backend

on:
  workflow_dispatch:

jobs:
  terraform-destroy:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read

    steps:
      - name: Destroy Terraform Infrastructure
        uses: subhamay-bhattacharyya-gha/tf-destroy-action@main
        with:
          tf-config-path: infra/aws/tf
          ci-pipeline: 'true'
          backend-type: s3
          cloud-provider: aws
          s3-bucket: ${{ vars.AWS_TF_STATE_BUCKET }}
          s3-region: ${{ vars.AWS_REGION }}
          s3-key-prefix: environments/prod
          aws-region: ${{ vars.AWS_REGION }}
          aws-role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
```

> **Note:** Configure these values in your GitHub repository:
> - `AWS_REGION` (Variable): Your AWS region (e.g., `us-east-1`)
> - `AWS_TF_STATE_BUCKET` (Variable): Your S3 bucket name for Terraform state
> - `AWS_ROLE_ARN` (Secret): Your IAM role ARN

### AWS with HCP Terraform Cloud Backend

```yaml
name: Terraform Destroy - AWS with Terraform Cloud

on:
  workflow_dispatch:

jobs:
  terraform-destroy:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read

    steps:
      - name: Destroy Terraform Infrastructure
        uses: subhamay-bhattacharyya-gha/tf-destroy-action@main
        with:
          tf-config-path: infra/aws/tf
          backend-type: remote
          cloud-provider: aws
          tfc-token: ${{ secrets.TFC_API_TOKEN }}
          aws-region: ${{ vars.AWS_REGION }}
          aws-role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
```

### Azure with S3 Backend

```yaml
name: Terraform Destroy - Azure with S3 Backend

on:
  workflow_dispatch:

jobs:
  terraform-destroy:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read

    steps:
      - name: Destroy Terraform Infrastructure
        uses: subhamay-bhattacharyya-gha/tf-destroy-action@main
        with:
          tf-config-path: infra/azure/tf
          backend-type: s3
          cloud-provider: azure
          s3-bucket: ${{ vars.AWS_TF_STATE_BUCKET }}
          s3-region: ${{ vars.AWS_REGION }}
          aws-region: ${{ vars.AWS_REGION }}
          aws-role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          azure-client-id: ${{ secrets.AZURE_CLIENT_ID }}
          azure-tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          azure-subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
```

### Azure with HCP Terraform Cloud Backend

```yaml
name: Terraform Destroy - Azure with Terraform Cloud

on:
  workflow_dispatch:

jobs:
  terraform-destroy:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read

    steps:
      - name: Destroy Terraform Infrastructure
        uses: subhamay-bhattacharyya-gha/tf-destroy-action@main
        with:
          tf-config-path: infra/azure/tf
          backend-type: remote
          cloud-provider: azure
          tfc-token: ${{ secrets.TFC_API_TOKEN }}
          azure-client-id: ${{ secrets.AZURE_CLIENT_ID }}
          azure-tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          azure-subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
```

### GCP with S3 Backend

```yaml
name: Terraform Destroy - GCP with S3 Backend

on:
  workflow_dispatch:

jobs:
  terraform-destroy:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read

    steps:
      - name: Destroy Terraform Infrastructure
        uses: subhamay-bhattacharyya-gha/tf-destroy-action@main
        with:
          tf-config-path: infra/gcp/tf
          backend-type: s3
          cloud-provider: gcp
          s3-bucket: ${{ vars.AWS_TF_STATE_BUCKET }}
          s3-region: ${{ vars.AWS_REGION }}
          aws-region: ${{ vars.AWS_REGION }}
          aws-role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          gcp-wif-provider: ${{ secrets.GCP_WIF_PROVIDER }}
          gcp-service-account: ${{ secrets.GCP_SERVICE_ACCOUNT }}
```

### GCP with HCP Terraform Cloud Backend

```yaml
name: Terraform Destroy - GCP with Terraform Cloud

on:
  workflow_dispatch:

jobs:
  terraform-destroy:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read

    steps:
      - name: Destroy Terraform Infrastructure
        uses: subhamay-bhattacharyya-gha/tf-destroy-action@main
        with:
          tf-config-path: infra/gcp/tf
          backend-type: remote
          cloud-provider: gcp
          tfc-token: ${{ secrets.TFC_API_TOKEN }}
          gcp-wif-provider: ${{ secrets.GCP_WIF_PROVIDER }}
          gcp-service-account: ${{ secrets.GCP_SERVICE_ACCOUNT }}
```

### Databricks with S3 Backend

```yaml
name: Terraform Destroy - Databricks with S3 Backend

on:
  workflow_dispatch:

jobs:
  terraform-destroy:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    env:
      # Databricks authentication via environment variables
      DATABRICKS_HOST: ${{ secrets.DATABRICKS_HOST }}
      DATABRICKS_TOKEN: ${{ secrets.DATABRICKS_TOKEN }}

    steps:
      - name: Destroy Terraform Infrastructure
        uses: subhamay-bhattacharyya-gha/tf-destroy-action@main
        with:
          tf-config-path: infra/databricks/tf
          backend-type: s3
          cloud-provider: databricks
          s3-bucket: ${{ vars.AWS_TF_STATE_BUCKET }}
          s3-region: ${{ vars.AWS_REGION }}
          aws-region: ${{ vars.AWS_REGION }}
          aws-role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
```

> **Note:** Databricks authentication is configured via environment variables at the job level. The Terraform Databricks provider will automatically use these environment variables.

### Databricks with HCP Terraform Cloud Backend

```yaml
name: Terraform Destroy - Databricks with Terraform Cloud

on:
  workflow_dispatch:

jobs:
  terraform-destroy:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    env:
      # Databricks authentication via environment variables
      DATABRICKS_HOST: ${{ secrets.DATABRICKS_HOST }}
      DATABRICKS_TOKEN: ${{ secrets.DATABRICKS_TOKEN }}

    steps:
      - name: Destroy Terraform Infrastructure
        uses: subhamay-bhattacharyya-gha/tf-destroy-action@main
        with:
          tf-config-path: infra/databricks/tf
          backend-type: remote
          cloud-provider: databricks
          tfc-token: ${{ secrets.TFC_API_TOKEN }}
```

### Snowflake with S3 Backend

```yaml
name: Terraform Destroy - Snowflake with S3 Backend

on:
  workflow_dispatch:

jobs:
  terraform-destroy:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    env:
      # Snowflake authentication via environment variables
      SNOWFLAKE_ACCOUNT: ${{ secrets.SNOWFLAKE_ACCOUNT }}
      SNOWFLAKE_USER: ${{ secrets.SNOWFLAKE_USER }}
      SNOWFLAKE_ROLE: ${{ secrets.SNOWFLAKE_ROLE }}

    steps:
      - name: Set Snowflake Private Key
        run: |
          printf '%b' "${{ secrets.SNOWFLAKE_PRIVATE_KEY }}" > /tmp/snowflake_key.p8
          echo "SNOWFLAKE_PRIVATE_KEY<<EOF" >> $GITHUB_ENV
          cat /tmp/snowflake_key.p8 >> $GITHUB_ENV
          echo "EOF" >> $GITHUB_ENV

      - name: Debug Key Format
        run: |
          echo "First 50 chars of key:"
          echo "${SNOWFLAKE_PRIVATE_KEY:0:50}"
          echo "---"
          echo "Key contains literal backslash-n: "
          if [[ "$SNOWFLAKE_PRIVATE_KEY" == *'\\n'* ]]; then
            echo "YES - problem found"
          else
            echo "NO"
          fi
          echo "---"
          echo "Line count:"
          echo "$SNOWFLAKE_PRIVATE_KEY" | wc -l

      - name: Destroy Terraform Infrastructure
        uses: subhamay-bhattacharyya-gha/tf-destroy-action@main
        with:
          tf-config-path: infra/snowflake/tf
          backend-type: s3
          cloud-provider: snowflake
          s3-bucket: ${{ vars.AWS_TF_STATE_BUCKET }}
          s3-region: ${{ vars.AWS_REGION }}
          aws-region: ${{ vars.AWS_REGION }}
          aws-role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
```

> **Note:** Snowflake authentication requires setting up environment variables and the private key before invoking the action. The `printf '%b'` command properly handles escaped newlines in the private key. The debug step can be removed once authentication is working correctly.

### Snowflake with HCP Terraform Cloud Backend

```yaml
name: Terraform Destroy - Snowflake with Terraform Cloud

on:
  workflow_dispatch:

jobs:
  terraform-destroy:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    env:
      # Snowflake authentication via environment variables
      SNOWFLAKE_ACCOUNT: ${{ secrets.SNOWFLAKE_ACCOUNT }}
      SNOWFLAKE_USER: ${{ secrets.SNOWFLAKE_USER }}
      SNOWFLAKE_ROLE: ${{ secrets.SNOWFLAKE_ROLE }}

    steps:
      - name: Set Snowflake Private Key
        run: |
          printf '%b' "${{ secrets.SNOWFLAKE_PRIVATE_KEY }}" > /tmp/snowflake_key.p8
          echo "SNOWFLAKE_PRIVATE_KEY<<EOF" >> $GITHUB_ENV
          cat /tmp/snowflake_key.p8 >> $GITHUB_ENV
          echo "EOF" >> $GITHUB_ENV

      - name: Debug Key Format
        run: |
          echo "First 50 chars of key:"
          echo "${SNOWFLAKE_PRIVATE_KEY:0:50}"
          echo "---"
          echo "Key contains literal backslash-n: "
          if [[ "$SNOWFLAKE_PRIVATE_KEY" == *'\\n'* ]]; then
            echo "YES - problem found"
          else
            echo "NO"
          fi
          echo "---"
          echo "Line count:"
          echo "$SNOWFLAKE_PRIVATE_KEY" | wc -l

      - name: Destroy Terraform Infrastructure
        uses: subhamay-bhattacharyya-gha/tf-destroy-action@main
        with:
          tf-config-path: infra/snowflake/tf
          backend-type: remote
          cloud-provider: snowflake
          tfc-token: ${{ secrets.TFC_API_TOKEN }}
```

### Platform Mode (Multi-Provider)

The `platform` mode allows you to manage infrastructure across multiple cloud providers in a single workflow.

```yaml
name: Terraform Destroy - Platform Mode (Multi-Provider)

on:
  workflow_dispatch:

jobs:
  terraform-destroy:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    env:
      # Databricks authentication (if infra/databricks exists)
      DATABRICKS_HOST: ${{ secrets.DATABRICKS_HOST }}
      DATABRICKS_TOKEN: ${{ secrets.DATABRICKS_TOKEN }}
      # Snowflake authentication (if infra/snowflake exists)
      SNOWFLAKE_ACCOUNT: ${{ secrets.SNOWFLAKE_ACCOUNT }}
      SNOWFLAKE_USER: ${{ secrets.SNOWFLAKE_USER }}
      SNOWFLAKE_ROLE: ${{ secrets.SNOWFLAKE_ROLE }}

    steps:
      - name: Set Snowflake Private Key (if needed)
        if: ${{ secrets.SNOWFLAKE_PRIVATE_KEY != '' }}
        run: |
          printf '%b' "${{ secrets.SNOWFLAKE_PRIVATE_KEY }}" > /tmp/snowflake_key.p8
          echo "SNOWFLAKE_PRIVATE_KEY<<EOF" >> $GITHUB_ENV
          cat /tmp/snowflake_key.p8 >> $GITHUB_ENV
          echo "EOF" >> $GITHUB_ENV

      - name: Destroy Terraform Infrastructure
        uses: subhamay-bhattacharyya-gha/tf-destroy-action@main
        with:
          tf-config-path: infra
          backend-type: s3
          cloud-provider: platform
          s3-bucket: ${{ vars.AWS_TF_STATE_BUCKET }}
          s3-region: ${{ vars.AWS_REGION }}
          # AWS Authentication (required for S3 backend)
          aws-region: ${{ vars.AWS_REGION }}
          aws-role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          # Azure Authentication (if infra/azure exists)
          azure-client-id: ${{ secrets.AZURE_CLIENT_ID }}
          azure-tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          azure-subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
          # GCP Authentication (if infra/gcp exists)
          gcp-wif-provider: ${{ secrets.GCP_WIF_PROVIDER }}
          gcp-service-account: ${{ secrets.GCP_SERVICE_ACCOUNT }}
```

> **Note:** In platform mode:
> - The action detects which provider directories exist under `infra/`
> - AWS, Azure, and GCP authentication is handled via action inputs
> - Databricks and Snowflake authentication must be set via environment variables in the caller workflow
> - AWS credentials are always required when using S3 backend

---

## 🔒 Security Best Practices

### OIDC Authentication
This action uses OpenID Connect (OIDC) for secure, keyless authentication where supported:

- **AWS**: Configure IAM roles with GitHub OIDC provider
- **Azure**: Set up Workload Identity Federation with GitHub
- **GCP**: Configure Workload Identity Federation pools

### Environment Variable Authentication
For Databricks and Snowflake, authentication is handled via environment variables set in the caller workflow:

- **Databricks**: Set `DATABRICKS_HOST` and `DATABRICKS_TOKEN` environment variables
- **Snowflake**: Set `SNOWFLAKE_ACCOUNT`, `SNOWFLAKE_USER`, `SNOWFLAKE_ROLE`, and `SNOWFLAKE_PRIVATE_KEY_PATH` environment variables

### Secrets Management
- Store sensitive values like `tfc-token`, `DATABRICKS_TOKEN`, and `SNOWFLAKE_PRIVATE_KEY` in GitHub Secrets
- Never hardcode credentials in workflow files
- Use environment-specific secrets for multi-environment setups
- For Snowflake, ensure private keys are stored securely and never committed to version control

### Permissions
Ensure your GitHub Actions workflow has minimal required permissions:
```yaml
permissions:
  id-token: write    # Required for OIDC
  contents: read     # Required for checkout
```

---

## 🐛 Troubleshooting

### Debug Output
The action includes comprehensive debug output showing all input values. Check the "Debug - Print All Inputs" step in your workflow logs.

### Common Issues

1. **Backend Configuration Errors**: Ensure required inputs match your `backend-type` and `cloud-provider`
2. **Authentication Failures**: Verify OIDC setup and role permissions for AWS/Azure/GCP, or environment variables for Databricks/Snowflake
3. **State Key Conflicts**: Use `s3-key-prefix` to organize state files
4. **Missing Terraform Files**: Ensure `tf-config-path` points to valid Terraform configuration
5. **Databricks/Snowflake Auth Issues**: Ensure environment variables are set at the job level before invoking the action

### Validation
The action validates all inputs before execution and provides clear error messages for missing or invalid configurations.

---

## License

MIT