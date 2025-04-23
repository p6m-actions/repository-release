# Repository Release Action

![Latest Release](https://img.shields.io/github/v/release/p6m-actions/repository-release?style=flat-square&label=Latest%20Release&color=blue)

A GitHub Action that automates versioned releases for GitHub Actions and other
repositories. This action creates and manages both minor releases and major
version branches following semantic versioning principles.

## Features

- Creates minor version releases (e.g., v1.0, v1.1, v1.2)
- Creates major version branches with proper versioning setup
- Automatically tags releases with both specific version (v1.2) and major
  version (v1)
- Uses a `.version-line` file to track major version for each branch

## Usage

### Basic Usage

```yaml
name: Release Repository
on:
  workflow_dispatch:
    inputs:
      release_type:
        description: "Type of release to create"
        required: true
        default: "minor_release"
        type: choice
        options:
          - "minor_release"
          - "major_branch"
jobs:
  release:
    runs-on: ubuntu-latest

    permissions:
      contents: write

    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Create Release
        uses: p6m-actions/repository-release@v1
        with:
          release_type: ${{ github.event.inputs.release_type }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

## Inputs

| Input          | Description                                         | Required | Default         |
| -------------- | --------------------------------------------------- | -------- | --------------- |
| `release_type` | Type of release (`minor_release` or `major_branch`) | Yes      | `minor_release` |
| `github_token` | GitHub token for creating releases and tags         | Yes      |                 |

## Release Strategy

This action implements a specific release management strategy:

1. **Version Tracking**:

   - The current major version for a branch is stored in the `.version-line` file
   - If this file doesn't exist, it defaults to major version `1`

2. **Minor Releases**:

   - Created from the current branch (`main` or a `vX-dev` branch)
   - Automatically determines the next minor version based on existing tags
   - Creates a tag for both the specific version (e.g., `v1.2`) and the major
     version (`v1`)

3. **Major Branches**:
   - When creating a new major version from `main`:
     - Creates a legacy branch (`vX-dev`) for the current major version
     - Sets `.version-line` in the legacy branch correctly
     - Updates `.version-line` in `main` to the new major version
     - Cannot create major branches from legacy branches
