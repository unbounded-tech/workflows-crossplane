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
