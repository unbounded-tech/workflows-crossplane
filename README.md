# Crossplane GitHub Workflows

This repository contains shared GitHub Actions workflows for Crossplane projects, focusing on quality validation and CI/CD automation.

## Workflows

### Validate (`validate.yaml`)

A reusable workflow that validates Crossplane compositions and examples using the Upbound CLI and Crossplane beta validate command.

#### Inputs

- `composition` (optional): Composition YAML filename (default: `composition.yaml`)
- `examples` (required): Examples input. Accepts either a single example string or a JSON array. JSON arrays can be of strings (legacy) or objects (new format with observed_resources):
  - Single example: `"examples/claim.yaml"`
  - Array of strings: `["examples/claim.yaml", "examples/another.yaml"]`
  - Array of objects: `[{"example": "examples/claim.yaml", "observed_resources": "examples/observed-resources/step-1/"}]`
- `api_path` (required): Path to your API directory containing your XRD definitions
- `crossplane_version` (optional): Crossplane CLI version to install (default: `v2.0.2`)
- `error_on_missing_schemas` (optional): Whether to error on missing schemas during validation (default: `true`)
- `ghcr_user` (optional): GitHub Container Registry username for pushing images

#### Secrets

- `GH_PAT` (optional): Personal access token for GitHub Container Registry write access (required if `ghcr_user` is provided)

#### Permissions

Requires the following permissions in the calling workflow:
- `packages: write` (for GHCR access)
- `contents: write` (for checking out code)
- `issues: write` (for potential issue reporting)
- `pull-requests: write` (for pull request comments)

#### What It Does

1. Caches Upbound and Crossplane CLI installations
2. Authenticates with Upbound (without logging into registry)
3. Installs Crossplane CLI
4. Logs into GitHub Container Registry (if `ghcr_user` provided)
5. Builds the project using `up` (Upbound CLI)
6. Renders the composition example
7. Validates the example against XRD schemas
8. Performs additional validation on the rendered composition

#### Usage

To use this workflow in your Crossplane composition repository:

```yaml
name: Validate

on:
  pull_request:
    branches: [ main ]

jobs:
  validate:
    uses: unbounded-tech/workflows-crossplane/.github/workflows/validate.yaml@main
    with:
      examples: '[{"example": "examples/claim.yaml"}]'
      api_path: 'package'
    secrets:
      GH_PAT: ${{ secrets.GH_PAT }}
```

#### Prerequisites

- Your repository should contain Crossplane composition and XRD definitions
- Enable GitHub Container Registry access if pushing packages
- Ensure your examples are valid YAML and reference correct schemas

### Test (`test.yaml`)

A reusable workflow that runs Crossplane tests using the Upbound CLI testing framework.

#### Inputs

- `crossplane_version` (optional): Crossplane CLI version to install (default: `v2.0.2`)
- `ghcr_user` (optional): GitHub Container Registry username for pulling/pushing images

#### Secrets

- `GH_PAT` (optional): Personal access token for GitHub Container Registry write access (required if `ghcr_user` is provided)

#### Permissions

Requires the following permissions in the calling workflow:
- `packages: write` (for GHCR access)
- `contents: write` (for checking out code)
- `issues: write` (for potential issue reporting)
- `pull-requests: write` (for pull request comments)

#### What It Does

1. Caches Upbound and Crossplane CLI installations
2. Authenticates with Upbound (without logging into registry)
3. Installs Crossplane CLI
4. Logs into GitHub Container Registry (if `ghcr_user` provided)
5. Builds the project using `up` (Upbound CLI)
6. Runs all tests located in the `tests/` directory using `up test run`

#### Usage

```yaml
name: Test

on:
  push:
    branches: [ main ]

jobs:
  test:
    uses: unbounded-tech/workflows-crossplane/.github/workflows/test.yaml@main
    secrets:
      GH_PAT: ${{ secrets.GH_PAT }}
```

### E2E (`e2e.yaml`)

A reusable workflow that runs end-to-end tests for Crossplane compositions using Kind clusters and real cloud providers.

#### Inputs

- `crossplane_version` (optional): Crossplane CLI version to install (default: `v2.0.2`)
- `ghcr_user` (optional): GitHub Container Registry username for pulling/pushing images
- `pattern` (optional): Test file pattern (default: `tests/e2e*`)
- `timeout-minutes` (optional): Timeout in minutes for the e2e test step (default: `20`)
- `cleanup-timeout-minutes` (optional): Timeout in minutes for the cleanup step (default: `10`)
- `aws` (optional): Enable AWS credentials setup (default: `false`)
- `aws-use-oidc` (optional): Use GitHub OIDC to assume an AWS role (default: `false`)
- `aws-role-name` (optional): AWS IAM role name to assume via OIDC (default: `hops-github-actions`)
- `aws-role-arn` (optional): AWS IAM role ARN to assume via OIDC (overrides `aws-role-name` + `aws-account-id`)
- `aws-account-id` (optional): AWS account ID used to build role ARN when `aws-role-arn` is not provided
- `aws-region` (optional): AWS region for OIDC credential configuration (default: `us-east-1`)

#### Secrets

- `GH_PAT` (optional): Personal access token for GitHub Container Registry write access (required if `ghcr_user` is provided)
- `AWS_ACCESS_KEY_ID` (optional): AWS Access Key ID for E2E tests (required if `aws` is `true` and `aws-use-oidc` is `false`)
- `AWS_SECRET_ACCESS_KEY` (optional): AWS Secret Access Key for E2E tests (required if `aws` is `true` and `aws-use-oidc` is `false`)
- `AWS_SESSION_TOKEN` (optional): AWS Session Token for E2E tests (required for OIDC/assumed roles when providing credentials via secrets)

#### Permissions

Requires the following permissions in the calling workflow:
- `packages: read` (for GHCR access)
- `contents: read` (for checking out code)
- `id-token: write` (for OIDC authentication with AWS, if using assumable roles)

#### What It Does

1. Caches Upbound and Crossplane CLI installations
2. Authenticates with Upbound (without logging into registry)
3. Installs Crossplane CLI
4. Logs into GitHub Container Registry
5. Builds the project using `up` (Upbound CLI)
6. Creates AWS credentials file (if `aws: true`)
7. Creates ENV secrets for each environment variable
8. Runs e2e tests using `up test run` with `--e2e` flag
9. Waits for managed resources to be cleaned up

#### Usage

Basic usage without AWS:

```yaml
name: E2E Tests

on:
  push:
    branches: [ main ]

jobs:
  e2e:
    uses: unbounded-tech/workflows-crossplane/.github/workflows/e2e.yaml@main
    secrets:
      GH_PAT: ${{ secrets.GH_PAT }}
```

With AWS credentials using OIDC role assumption (recommended):

```yaml
name: E2E Tests

on:
  push:
    branches: [ main ]

permissions:
  id-token: write
  contents: read
  packages: read

jobs:
  e2e:
    uses: unbounded-tech/workflows-crossplane/.github/workflows/e2e.yaml@main
    with:
      aws: true
      aws-use-oidc: true
      aws-account-id: 123456789012
      aws-role-name: hops-github-actions
      aws-region: us-east-1
    secrets:
      GH_PAT: ${{ secrets.GH_PAT }}
```

With AWS credentials using OIDC assumable role via a separate job:

```yaml
name: E2E Tests

on:
  push:
    branches: [ main ]

permissions:
  id-token: write
  contents: read
  packages: read

jobs:
  aws-credentials:
    runs-on: ubuntu-latest
    outputs:
      aws_access_key_id: ${{ steps.creds.outputs.aws-access-key-id }}
      aws_secret_access_key: ${{ steps.creds.outputs.aws-secret-access-key }}
      aws_session_token: ${{ steps.creds.outputs.aws-session-token }}
    steps:
      - name: Configure AWS Credentials
        id: creds
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/github-actions-role
          aws-region: us-east-1
          output-credentials: true

  e2e:
    needs: aws-credentials
    uses: unbounded-tech/workflows-crossplane/.github/workflows/e2e.yaml@main
    with:
      aws: true
    secrets:
      GH_PAT: ${{ secrets.GH_PAT }}
      AWS_ACCESS_KEY_ID: ${{ needs.aws-credentials.outputs.aws_access_key_id }}
      AWS_SECRET_ACCESS_KEY: ${{ needs.aws-credentials.outputs.aws_secret_access_key }}
      AWS_SESSION_TOKEN: ${{ needs.aws-credentials.outputs.aws_session_token }}
```

With static AWS credentials (if OIDC is not available):

```yaml
name: E2E Tests

on:
  push:
    branches: [ main ]

jobs:
  e2e:
    uses: unbounded-tech/workflows-crossplane/.github/workflows/e2e.yaml@main
    with:
      aws: true
    secrets:
      GH_PAT: ${{ secrets.GH_PAT }}
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

#### Migrating from `e2e-aws.yaml`

The `e2e-aws.yaml` workflow is deprecated. To migrate to `e2e.yaml`, add `aws: true` to your inputs:

**Before:**
```yaml
jobs:
  e2e:
    uses: unbounded-tech/workflows-crossplane/.github/workflows/e2e-aws.yaml@main
    secrets:
      GH_PAT: ${{ secrets.GH_PAT }}
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

**After:**
```yaml
jobs:
  e2e:
    uses: unbounded-tech/workflows-crossplane/.github/workflows/e2e.yaml@main
    with:
      aws: true
    secrets:
      GH_PAT: ${{ secrets.GH_PAT }}
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

### E2E AWS (`e2e-aws.yaml`) - Deprecated

> **Deprecated:** Use `e2e.yaml` with `aws: true` instead. See migration guide above.

This workflow is maintained for backwards compatibility only and will be removed in a future release.

### Publish (`publish.yaml`)

A reusable workflow that publishes Crossplane packages to a registry using the Upbound CLI.

#### Inputs

- `tag` (required): Tag to use when publishing the package
- `crossplane_version` (optional): Crossplane CLI version to install (default: `v2.0.2`)
- `ghcr_user` (optional): GitHub Container Registry username for pushing packages

#### Secrets

- `GH_PAT` (optional): Personal access token for GitHub Container Registry write access (required if `ghcr_user` is provided)

#### Permissions

Requires the following permissions in the calling workflow:
- `packages: write` (for GHCR access)
- `contents: write` (for checking out code)
- `issues: write` (for potential issue reporting)
- `pull-requests: write` (for pull request comments)

#### What It Does

1. Caches Upbound and Crossplane CLI installations
2. Authenticates with Upbound (without logging into registry)
3. Installs Crossplane CLI
4. Logs into GitHub Container Registry (if `ghcr_user` provided)
5. Builds the project using `up` (Upbound CLI)
6. Publishes the built package to the configured registry with the specified tag

#### Usage

```yaml
name: Publish

on:
  release:
    types: [ published ]

jobs:
  publish:
    uses: unbounded-tech/workflows-crossplane/.github/workflows/publish.yaml@main
    with:
      tag: ${{ github.event.release.tag_name }}
    secrets:
      GH_PAT: ${{ secrets.GH_PAT }}
```

## Dependencies Management

This repository uses [Renovate](https://renovatebot.com/) for automated dependency updates on GitHub Actions versions and other dependencies. The configuration extends the recommended settings from the Renovate configuration preset.

## Contributing

Contributions to improve these workflows are welcome. Please ensure that new workflows follow GitHub Actions best practices and include comprehensive documentation.
