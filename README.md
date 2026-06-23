# NuGet Publisher
A composite GitHub Action that builds, packs, and publishes a NuGet package to both [nuget.org](https://nuget.org) and GitHub Packages.

## Usage

```yaml
jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: viklover/nuget-publish@v1
        with:
          DOTNET_VERSION: '8.x'
          PROJECT_FILE_PATH: src/MyProject/MyProject.csproj
          NUGET_USER: ${{ secrets.NUGET_USER }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

> **Important**: This action must be triggered by a tag push (`v*`). The package version is extracted from the tag name (e.g. `v1.2.3` → version `1.2.3`). Without a tag, the version will be incorrect.

## Inputs

| Input               | Required | Description                                                 |
|---------------------|----------|-------------------------------------------------------------|
| `DOTNET_VERSION`    | Yes      | Version of the .NET SDK to use (e.g. `8.x`, `9.0`)         |
| `PROJECT_FILE_PATH` | Yes      | Path to the `.csproj` file, relative to repository root     |
| `NUGET_USER`        | Yes      | Username for NuGet OIDC login (gets a temporary API key)    |
| `GITHUB_TOKEN`      | Yes      | `${{ secrets.GITHUB_TOKEN }}` for GitHub Packages push      |

## Example of workflow 

```yaml
name: NuGet publish

on:
  push:
    tags:
      - 'v*.*.*'

jobs:
  build-and-publish:
    runs-on: ubuntu-latest
    environment: release
    permissions:
      id-token: write
      contents: read
      packages: write

    steps:
      - uses: viklover/nuget-publish@v1
        with:
          DOTNET_VERSION: "7.0.x"
          PROJECT_FILE_PATH: JsonHelper.Net/JsonHelper.Net.csproj
          NUGET_USER: viklover
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Prerequisites

- [NuGet OIDC federation](https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/configuring-openid-connect-in-nuget) must be configured for your NuGet.org account.
- GitHub Packages must be enabled for the repository or organisation.
