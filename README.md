# NuGet Publisher

A composite GitHub Action that builds, packs, and publishes a NuGet package to both [nuget.org](https://nuget.org) and GitHub Packages.

## Features

- Extracts package version from the git tag (`v*`)
- Restores dependencies, builds, and packs a .NET project
- Publishes to **nuget.org** via OIDC authentication (temporary API key)
- Publishes to **GitHub Packages** as a secondary feed
- Skips duplicate packages on both feeds

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

## Inputs

| Input               | Required | Description                                                 |
|---------------------|----------|-------------------------------------------------------------|
| `DOTNET_VERSION`    | Yes      | Version of the .NET SDK to use (e.g. `8.x`, `9.0`)         |
| `PROJECT_FILE_PATH` | Yes      | Path to the `.csproj` file, relative to repository root     |
| `NUGET_USER`        | Yes      | Username for NuGet OIDC login (gets a temporary API key)    |
| `GITHUB_TOKEN`      | Yes      | `${{ secrets.GITHUB_TOKEN }}` for GitHub Packages push      |

## How it works

1. **Version**: The action reads the version from the triggering git tag (e.g. `v1.2.3` → `1.2.3`). Make sure to push a tag like `vX.Y.Z` to trigger the workflow.
2. **Build & Pack**: Runs `dotnet restore`, `dotnet build`, and `dotnet pack` in sequence.
3. **NuGet publish**: Uses the [NuGet/login](https://github.com/NuGet/login) action to obtain a temporary API key via OIDC and pushes the package to `api.nuget.org`.
4. **GitHub Packages publish**: Pushes the same package to the GitHub Packages NuGet feed under the `viklover` organization.

## Trigger example

```yaml
on:
  push:
    tags:
      - 'v*'
```

## Prerequisites

- [NuGet OIDC federation](https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/configuring-openid-connect-in-nuget) must be configured for your NuGet.org account.
- GitHub Packages must be enabled for the repository or organisation.
