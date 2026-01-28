# GitHub Action: Terraform Destroy and Summary

![Release](https://github.com/subhamay-bhattacharyya-gha/tf-destroy-action/actions/workflows/release.yaml/badge.svg)&nbsp;![Commit Activity](https://img.shields.io/github/commit-activity/t/subhamay-bhattacharyya-gha/tf-destroy-action)&nbsp;![Last Commit](https://img.shields.io/github/last-commit/subhamay-bhattacharyya-gha/tf-destroy-action)&nbsp;![Release Date](https://img.shields.io/github/release-date/subhamay-bhattacharyya-gha/tf-destroy-action)&nbsp;![Repo Size](https://img.shields.io/github/repo-size/subhamay-bhattacharyya-gha/tf-destroy-action)&nbsp;![File Count](https://img.shields.io/github/directory-file-count/subhamay-bhattacharyya-gha/tf-destroy-action)&nbsp;![Issues](https://img.shields.io/github/issues/subhamay-bhattacharyya-gha/tf-destroy-action)&nbsp;![Top Language](https://img.shields.io/github/languages/top/subhamay-bhattacharyya-gha/tf-destroy-action)&nbsp;![Custom Endpoint](https://img.shields.io/endpoint?url=https://gist.githubusercontent.com/bsubhamay/e15bf74830c9bea324a8b87e1f11a2fb/raw/tf-destroy-action.json?)

A GitHub Composite Action to safely destroy Terraform-managed infrastructure with **multi-cloud support** (AWS, Azure, GCP, Databricks, Snowflake) and multiple backend options (S3, HCP Terraform Cloud). Displays a clear, timestamped summary in the GitHub Actions Step Summary.

---

## 🔧 Features

- **Multi-Cloud Support**: Works with AWS, Azure, GCP, Databricks, and Snowflake
- **Multiple Backend Options**: Supports AWS S3 and HCP Terraform Cloud backends
- **OIDC Authentication**: Secure authentication using Workload Identity Federation
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
| `terraform-dir`   | Relative path to Terraform configuration directory                  | No       | `tf`    | string |
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

*Required when `cloud-provider` is `aws`

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

| Name                  | Description                                                     | Required | Default | Type   |
|-----------------------|-----------------------------------------------------------------|----------|---------|--------|
| `databricks-host`     | Databricks workspace URL                                        | No*      | —       | string |
| `databricks-token`    | Databricks personal access token                                | No*      | —       | string |

*Required when `cloud-provider` is `databricks`

### Snowflake Authentication (when `cloud-provider: snowflake`)

| Name                     | Description                                                  | Required | Default | Type   |
|--------------------------|--------------------------------------------------------------|----------|---------|--------|
| `snowflake-account`      | Snowflake account identifier                                 | No*      | —       | string |
| `snowflake-user`         | Snowflake user name                                          | No*      | —       | string |
| `snowflake-role`         | Snowflake role name                                          | No*      | —       | string |
| `snowflake-private-key`  | Snowflake private key for authentication                     | No*      | —       | string |

*Required when `cloud-provider` is `snowflake`

### Platform Mode (when `cloud-provider: platform`)

When using `platform` mode, the action automatically detects which cloud provider directories exist in your `infra/` folder and validates only the required inputs for those providers. Provide credentials for each provider you use:

| Provider Directory | Required Inputs |
|-------------------|-----------------|
| `infra/aws`       | `aws-region`, `aws-role-to-assume` |
| `infra/azure`     | `azure-client-id`, `azure-tenant-id`, `azure-subscription-id` |
| `infra/gcp`       | `gcp-wif-provider`, `gcp-service-account` |
| `infra/databricks`| `databricks-host`, `databricks-token` |
| `infra/snowflake` | `snowflake-account`, `snowflake-user`, `snowflake-role`, `snowflake-private-key` |

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
          # Core Configuration
          terraform-dir: infra/aws/tf
          ci-pipeline: 'true'
          
          # Backend Configuration
          backend-type: s3
          cloud-provider: aws
          
          # AWS S3 Backend
          s3-bucket: ${{ vars.AWS_TF_STATE_BUCKET }}
          s3-region: ${{ vars.AWS_REGION }}
          s3-key-prefix: environments/prod
          
          # AWS Authentication
          aws-region: ${{ vars.AWS_REGION }}
          aws-role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
```

> **Note:** Configure these values in your GitHub repository:
> - `AWS_REGION` (Variable): Your AWS region (e.g., `us-east-1`)
> - `AWS_TF_STATE_BUCKET` (Variable): Your S3 bucket name for Terraform state (e.g., `my-company-terraform-state`)
> - `AWS_ROLE_ARN` (Secret): Your IAM role ARN (e.g., `arn:aws:iam::123456789012:role/GitHubActionsRole`)
> 
> Never hardcode IAM role ARNs, bucket names, or regions in your workflow files. Store sensitive values like role ARNs as GitHub repository secrets and non-sensitive values like regions and bucket names as repository variables for security.

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
          # Core Configuration
          terraform-dir: infra/aws/tf
          
          # Backend Configuration
          backend-type: remote
          cloud-provider: aws
          
          # HCP Terraform Cloud Backend
          tfc-token: ${{ secrets.TFC_API_TOKEN }}
          
          # AWS Authentication
          aws-region: ${{ vars.AWS_REGION }}
          aws-role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
```

> **Note:** Configure these values in your GitHub repository:
> - `AWS_REGION` (Variable): Your AWS region (e.g., `us-east-1`)
> - `AWS_ROLE_ARN` (Secret): Your IAM role ARN (e.g., `arn:aws:iam::123456789012:role/GitHubActionsRole`)
> - `TFC_API_TOKEN` (Secret): Your HCP Terraform Cloud API token
> 
> Store sensitive values like role ARNs and API tokens as GitHub repository secrets for security.

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
          # Core Configuration
          terraform-dir: infra/azure/tf
          
          # Backend Configuration  
          backend-type: s3
          cloud-provider: azure
          
          # AWS S3 Backend (cross-cloud scenario)
          s3-bucket: ${{ vars.AWS_TF_STATE_BUCKET }}
          s3-region: ${{ vars.AWS_REGION }}
          
          # Azure Authentication
          azure-client-id: ${{ secrets.AZURE_CLIENT_ID }}
          azure-tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          azure-subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
```

> **Note:** Configure these values in your GitHub repository:
> - `AWS_REGION` (Variable): Your AWS region for S3 backend (e.g., `us-east-1`)
> - `AWS_TF_STATE_BUCKET` (Variable): Your S3 bucket name for Terraform state (e.g., `my-company-terraform-state`)
> - `AZURE_CLIENT_ID` (Secret): Your Azure client ID for authentication
> - `AZURE_TENANT_ID` (Secret): Your Azure tenant ID for authentication
> - `AZURE_SUBSCRIPTION_ID` (Secret): Your Azure subscription ID for authentication
> 
> Never hardcode authentication credentials, bucket names, or regions in your workflow files. Store sensitive values like Azure credentials as GitHub repository secrets and non-sensitive values like regions and bucket names as repository variables for security.

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
          # Core Configuration
          terraform-dir: infra/azure/tf
          
          # Backend Configuration
          backend-type: remote
          cloud-provider: azure
          
          # HCP Terraform Cloud Backend
          tfc-token: ${{ secrets.TFC_API_TOKEN }}
          
          # Azure Authentication
          azure-client-id: ${{ secrets.AZURE_CLIENT_ID }}
          azure-tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          azure-subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
```

> **Note:** Configure these values in your GitHub repository:
> - `TFC_API_TOKEN` (Secret): Your HCP Terraform Cloud API token
> - `AZURE_CLIENT_ID` (Secret): Your Azure client ID for authentication
> - `AZURE_TENANT_ID` (Secret): Your Azure tenant ID for authentication
> - `AZURE_SUBSCRIPTION_ID` (Secret): Your Azure subscription ID for authentication
> 
> Store all authentication values as GitHub repository secrets for security.

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
          # Core Configuration
          terraform-dir: infra/gcp/tf
          
          # Backend Configuration
          backend-type: s3
          cloud-provider: gcp
          
          # AWS S3 Backend (cross-cloud scenario)
          s3-bucket: ${{ vars.AWS_TF_STATE_BUCKET }}
          s3-region: ${{ vars.AWS_REGION }}
          
          # GCP Authentication
          gcp-wif-provider: ${{ secrets.GCP_WIF_PROVIDER }}
          gcp-service-account: ${{ secrets.GCP_SERVICE_ACCOUNT }}
```

> **Note:** Configure these values in your GitHub repository:
> - `AWS_REGION` (Variable): Your AWS region for S3 backend (e.g., `us-east-1`)
> - `AWS_TF_STATE_BUCKET` (Variable): Your S3 bucket name for Terraform state (e.g., `my-company-terraform-state`)
> - `GCP_WIF_PROVIDER` (Secret): Your GCP Workload Identity Federation provider (e.g., `projects/123456789/locations/global/workloadIdentityPools/github-pool/providers/github-provider`)
> - `GCP_SERVICE_ACCOUNT` (Secret): Your GCP service account email (e.g., `terraform-sa@my-project.iam.gserviceaccount.com`)
> 
> Store sensitive values like GCP credentials as GitHub repository secrets and non-sensitive values like regions and bucket names as repository variables for security.

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
          # Core Configuration
          terraform-dir: infra/gcp/tf
          release-tag: v1.2.3
          
          # Backend Configuration
          backend-type: remote
          cloud-provider: gcp
          
          # HCP Terraform Cloud Backend
          tfc-token: ${{ secrets.TFC_API_TOKEN }}
          
          # GCP Authentication
          gcp-wif-provider: ${{ secrets.GCP_WIF_PROVIDER }}
          gcp-service-account: ${{ secrets.GCP_SERVICE_ACCOUNT }}
```

> **Note:** Configure these GCP values in your GitHub repository:
> - `TFC_API_TOKEN` (Secret): Your HCP Terraform Cloud API token
> - `GCP_WIF_PROVIDER` (Secret): Your GCP Workload Identity Federation provider (e.g., `projects/123456789/locations/global/workloadIdentityPools/github-pool/providers/github-provider`)
> - `GCP_SERVICE_ACCOUNT` (Secret): Your GCP service account email (e.g., `terraform-sa@my-project.iam.gserviceaccount.com`)
> 
> Store all GCP authentication values as GitHub repository secrets for security.

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

    steps:
      - name: Destroy Terraform Infrastructure
        uses: subhamay-bhattacharyya-gha/tf-destroy-action@main
        with:
          # Core Configuration
          terraform-dir: infra/databricks/tf
          
          # Backend Configuration
          backend-type: s3
          cloud-provider: databricks
          
          # AWS S3 Backend (cross-cloud scenario)
          s3-bucket: ${{ vars.AWS_TF_STATE_BUCKET }}
          s3-region: ${{ vars.AWS_REGION }}
          
          # Databricks Authentication
          databricks-host: ${{ secrets.DATABRICKS_HOST }}
          databricks-token: ${{ secrets.DATABRICKS_TOKEN }}
```

> **Note:** Configure these values in your GitHub repository:
> - `AWS_REGION` (Variable): Your AWS region for S3 backend (e.g., `us-east-1`)
> - `AWS_TF_STATE_BUCKET` (Variable): Your S3 bucket name for Terraform state (e.g., `my-company-terraform-state`)
> - `DATABRICKS_HOST` (Secret): Your Databricks workspace URL (e.g., `https://dbc-12345678-9abc.cloud.databricks.com`)
> - `DATABRICKS_TOKEN` (Secret): Your Databricks personal access token
> 
> Store sensitive values like Databricks credentials as GitHub repository secrets and non-sensitive values like regions and bucket names as repository variables for security.

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

    steps:
      - name: Destroy Terraform Infrastructure
        uses: subhamay-bhattacharyya-gha/tf-destroy-action@main
        with:
          # Core Configuration
          terraform-dir: infra/databricks/tf
          
          # Backend Configuration
          backend-type: remote
          cloud-provider: databricks
          
          # HCP Terraform Cloud Backend
          tfc-token: ${{ secrets.TFC_API_TOKEN }}
          
          # Databricks Authentication
          databricks-host: ${{ secrets.DATABRICKS_HOST }}
          databricks-token: ${{ secrets.DATABRICKS_TOKEN }}
```

> **Note:** Configure these values in your GitHub repository:
> - `TFC_API_TOKEN` (Secret): Your HCP Terraform Cloud API token
> - `DATABRICKS_HOST` (Secret): Your Databricks workspace URL (e.g., `https://dbc-12345678-9abc.cloud.databricks.com`)
> - `DATABRICKS_TOKEN` (Secret): Your Databricks personal access token
> 
> Store all authentication values as GitHub repository secrets for security.

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

    steps:
      - name: Destroy Terraform Infrastructure
        uses: subhamay-bhattacharyya-gha/tf-destroy-action@main
        with:
          # Core Configuration
          terraform-dir: infra/snowflake/tf
          
          # Backend Configuration
          backend-type: s3
          cloud-provider: snowflake
          
          # AWS S3 Backend (cross-cloud scenario)
          s3-bucket: ${{ vars.AWS_TF_STATE_BUCKET }}
          s3-region: ${{ vars.AWS_REGION }}
          
          # Snowflake Authentication
          snowflake-account: ${{ secrets.SNOWFLAKE_ACCOUNT }}
          snowflake-user: ${{ secrets.SNOWFLAKE_USER }}
          snowflake-role: ${{ secrets.SNOWFLAKE_ROLE }}
          snowflake-private-key: ${{ secrets.SNOWFLAKE_PRIVATE_KEY }}
```

> **Note:** Configure these values in your GitHub repository:
> - `AWS_REGION` (Variable): Your AWS region for S3 backend (e.g., `us-east-1`)
> - `AWS_TF_STATE_BUCKET` (Variable): Your S3 bucket name for Terraform state (e.g., `my-company-terraform-state`)
> - `SNOWFLAKE_ACCOUNT` (Secret): Your Snowflake account identifier (e.g., `xy12345.us-east-1`)
> - `SNOWFLAKE_USER` (Secret): Your Snowflake user name
> - `SNOWFLAKE_ROLE` (Secret): Your Snowflake role name
> - `SNOWFLAKE_PRIVATE_KEY` (Secret): Your Snowflake private key for authentication
> 
> Store sensitive values like Snowflake credentials as GitHub repository secrets and non-sensitive values like regions and bucket names as repository variables for security.

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

    steps:
      - name: Destroy Terraform Infrastructure
        uses: subhamay-bhattacharyya-gha/tf-destroy-action@main
        with:
          # Core Configuration
          terraform-dir: infra/snowflake/tf
          
          # Backend Configuration
          backend-type: remote
          cloud-provider: snowflake
          
          # HCP Terraform Cloud Backend
          tfc-token: ${{ secrets.TFC_API_TOKEN }}
          
          # Snowflake Authentication
          snowflake-account: ${{ secrets.SNOWFLAKE_ACCOUNT }}
          snowflake-user: ${{ secrets.SNOWFLAKE_USER }}
          snowflake-role: ${{ secrets.SNOWFLAKE_ROLE }}
          snowflake-private-key: ${{ secrets.SNOWFLAKE_PRIVATE_KEY }}
```

> **Note:** Configure these values in your GitHub repository:
> - `TFC_API_TOKEN` (Secret): Your HCP Terraform Cloud API token
> - `SNOWFLAKE_ACCOUNT` (Secret): Your Snowflake account identifier (e.g., `xy12345.us-east-1`)
> - `SNOWFLAKE_USER` (Secret): Your Snowflake user name
> - `SNOWFLAKE_ROLE` (Secret): Your Snowflake role name
> - `SNOWFLAKE_PRIVATE_KEY` (Secret): Your Snowflake private key for authentication
> 
> Store all authentication values as GitHub repository secrets for security.

### Platform Mode (Multi-Provider)

The `platform` mode allows you to manage infrastructure across multiple cloud providers in a single workflow. The action automatically detects which provider directories exist in your `infra/` folder and authenticates with each one.

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

    steps:
      - name: Destroy Terraform Infrastructure
        uses: subhamay-bhattacharyya-gha/tf-destroy-action@main
        with:
          # Core Configuration
          terraform-dir: infra
          
          # Backend Configuration
          backend-type: s3
          cloud-provider: platform
          
          # AWS S3 Backend
          s3-bucket: ${{ vars.AWS_TF_STATE_BUCKET }}
          s3-region: ${{ vars.AWS_REGION }}
          
          # AWS Authentication (required for S3 backend and if infra/aws exists)
          aws-region: ${{ vars.AWS_REGION }}
          aws-role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          
          # Azure Authentication (if infra/azure exists)
          azure-client-id: ${{ secrets.AZURE_CLIENT_ID }}
          azure-tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          azure-subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
          
          # GCP Authentication (if infra/gcp exists)
          gcp-wif-provider: ${{ secrets.GCP_WIF_PROVIDER }}
          gcp-service-account: ${{ secrets.GCP_SERVICE_ACCOUNT }}
          
          # Databricks Authentication (if infra/databricks exists)
          databricks-host: ${{ secrets.DATABRICKS_HOST }}
          databricks-token: ${{ secrets.DATABRICKS_TOKEN }}
          
          # Snowflake Authentication (if infra/snowflake exists)
          snowflake-account: ${{ secrets.SNOWFLAKE_ACCOUNT }}
          snowflake-user: ${{ secrets.SNOWFLAKE_USER }}
          snowflake-role: ${{ secrets.SNOWFLAKE_ROLE }}
          snowflake-private-key: ${{ secrets.SNOWFLAKE_PRIVATE_KEY }}
```

> **Note:** In platform mode, the action:
> - Automatically detects which provider directories exist under `infra/` (e.g., `infra/aws`, `infra/azure`, `infra/gcp`, `infra/databricks`, `infra/snowflake`)
> - Only validates and authenticates with providers that have corresponding directories
> - Requires AWS credentials when using S3 backend, regardless of other providers
> - Allows you to provide credentials for multiple providers and only uses what's needed
>
> Configure only the secrets/variables for the providers you actually use in your infrastructure.

---

## 🔒 Security Best Practices

### OIDC Authentication
This action uses OpenID Connect (OIDC) for secure, keyless authentication where supported:

- **AWS**: Configure IAM roles with GitHub OIDC provider
- **Azure**: Set up Workload Identity Federation with GitHub
- **GCP**: Configure Workload Identity Federation pools
- **Databricks**: Uses personal access token authentication
- **Snowflake**: Uses private key authentication

### Secrets Management
- Store sensitive values like `tfc-token`, `databricks-token`, and `snowflake-private-key` in GitHub Secrets
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
2. **Authentication Failures**: Verify OIDC setup and role permissions
3. **State Key Conflicts**: Use `s3-key-prefix` to organize state files
4. **Missing Terraform Files**: Ensure `terraform-dir` points to valid Terraform configuration

### Validation
The action validates all inputs before execution and provides clear error messages for missing or invalid configurations.

---

## License

MIT