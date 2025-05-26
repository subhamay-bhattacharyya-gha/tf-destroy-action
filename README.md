# GitHub Action: Terraform Destroy and Summary

![Release](https://github.com/subhamay-bhattacharyya-gha/tf-destroy-action/actions/workflows/release.yaml/badge.svg)&nbsp;![Commit Activity](https://img.shields.io/github/commit-activity/t/subhamay-bhattacharyya-gha/tf-destroy-action)&nbsp;![Last Commit](https://img.shields.io/github/last-commit/subhamay-bhattacharyya-gha/tf-destroy-action)&nbsp;![Release Date](https://img.shields.io/github/release-date/subhamay-bhattacharyya-gha/tf-destroy-action)&nbsp;![Repo Size](https://img.shields.io/github/repo-size/subhamay-bhattacharyya-gha/tf-destroy-action)&nbsp;![File Count](https://img.shields.io/github/directory-file-count/subhamay-bhattacharyya-gha/tf-destroy-action)&nbsp;![Issues](https://img.shields.io/github/issues/subhamay-bhattacharyya-gha/tf-destroy-action)&nbsp;![Top Language](https://img.shields.io/github/languages/top/subhamay-bhattacharyya-gha/tf-destroy-action)&nbsp;![Custom Endpoint](https://img.shields.io/endpoint?url=https://gist.githubusercontent.com/bsubhamay/e15bf74830c9bea324a8b87e1f11a2fb/raw/tf-destroy-action.json?)

A GitHub Composite Action to safely destroy Terraform-managed infrastructure using remote backend and show a clear, timestamped summary in the GitHub Actions Step Summary.

---

## 🔧 Features

- Initializes Terraform backend using S3 and DynamoDB
- Computes dynamic backend key based on CI pipeline context
- Creates a destroy plan (`terraform plan -destroy`) and parses it via JSON
- Runs `terraform destroy` with full logging
- Summarizes resources destroyed in a Markdown table:
  - Resource Type
  - Resource Name
  - Address
  - Timestamp
- Displays destruction logs and status in GitHub Step Summary

---

## 📥 Inputs

| Name              | Description                                                         | Required | Default |
|-------------------|---------------------------------------------------------------------|----------|---------|
| `s3-bucket`        | S3 bucket for Terraform backend                                     | Yes      | —       |
| `s3-region`        | AWS region where the S3 bucket is located                           | Yes      | —       |
| `dynamodb-table`   | DynamoDB table name for state locking                               | Yes      | —       |
| `terraform-dir`    | Directory where Terraform code is located                           | No       | `tf`    |
| `ci-pipeline`      | Whether this is a CI pipeline (adds commit SHA to backend key path) | No       | `true`  |

---

## 🚀 Example Usage

```yaml
name: Terraform Destroy Workflow

on:
  workflow_dispatch:

jobs:
  terraform-destroy:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read

    steps:
      - name: Configure AWS Credentials via OIDC
        uses: aws-actions/configure-aws-credentials@v2
        with:
          role-to-assume: arn:aws:iam::123456789012:role/GitHubActionsTerraformRole
          aws-region: us-east-1

      - name: Destroy Terraform Infrastructure
        uses: subhamay-bhattacharyya-gha/tf-destroy-action@main
        with:
          terraform-dir: tf
          s3-bucket: gha-terraform-state
          s3-region: us-east-1
          dynamodb-table: gha-terraform-lock

```

## License

MIT
