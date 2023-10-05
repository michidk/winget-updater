<h1> <img src="https://github.com/michidk/winget-updater/blob/main/.github/github-actions-logo.png" width="32" height="32" alt="Logo" /> WinGet Updater (GitHub Action) </h1>

[![GitHub issues][github-issues-badge]](https://github.com/michidk/winget-updater/issues)
[![GitHub release (latest by date)][github-release-badge]](https://github.com/michidk/winget-updater/releases)
[![GitHub Repo stars][github-repo-stars-badge]](https://github.com/michidk/winget-updater/stargazers)
[![GitHub][github-license-badge]](https://github.com/michidk/winget-updater?tab=MIT-1-ov-file#readme)

A GitHub action which automatically updates WinGet packages, based on Komac.

## 📖 Example Usage

```yaml
name: Update WinGet Packages

on:
  workflow_dispatch:
  schedule:
    - cron:  '0 3 * * *' # Scheduled to run daily at 03:00

jobs:
  update:
    name: Update package ${{ matrix.id }}
    runs-on: ubuntu-latest

    strategy:
      matrix:
        include:
          - id: "michidk.vscli"
            repo: "michidk/vscli"
            url: "https://github.com/michidk/vscli/releases/download/v{VERSION}/vscli-x86_64-pc-windows-msvc.zip"
          - id: "Casey.Just"
            repo: "casey/just"
            url: "https://github.com/casey/just/releases/download/{VERSION}/just-{VERSION}-x86_64-pc-windows-msvc.zip"
    steps:
    - name: Update Packages
      uses: michidk/winget-updater@v1
      with:
        package-id: ${{ matrix.id }}
        repo: ${{ matrix.repo }}
        url: ${{ matrix.url }}

  cleanup:
    name: Cleanup branches
    needs: update # Not necessarily needed as PRs don't get closed that quick but still nice to have it in order
    runs-on: ubuntu-latest

    steps:
    - name: Run Komac
      uses: michidk/run-komac@v1
      with:
        args: 'branch cleanup --token=${{ secrets.KOMAC_TOKEN }}'
```


## ⚒️ Configuration Options for WinGet Updater Action

- `komac-version`: Specifies which version of Komac to use.
  - **Required**: ❌
  - **Default**: `1.11.0`

- `package-id`: The package identifier of the package to update.
  - **Required**: ✅

- `repo`: The GitHub repository to check for the latest release of the package.
  - **Required**: ✅

- `url`: The URL template to the latest release of the package. Use `{VERSION}` as a placeholder for the version number.
  - **Required**: ✅

<h2> 🚀 Integrating with <a href="https://github.com/russellbanks/Komac"> <img src="https://github.com/vedantmgoyal2009/winget-releaser/blob/main/.github/komac-logo.svg" height="24px" style="vertical-align:bottom" alt="Komac logo" /> </a></h2>

This GitHub action leverages [Komac][komac-repo] to generate and submit manifests to the [Windows Package Manager Community Repository][winget-pkgs-repo]. Kudos to [Russell Banks][russellbanks-github-profile] for developing Komac which powers this action.

[github-issues-badge]: https://img.shields.io/github/issues/michidk/winget-updater?logo=target
[github-release-badge]: https://img.shields.io/github/v/release/michidk/winget-updater?logo=github
[github-repo-stars-badge]: https://img.shields.io/github/stars/michidk/winget-updater?logo=githubsponsors
[github-license-badge]: https://img.shields.io/github/license/michidk/winget-updater?logo=gnu
[winget-pkgs-repo]: https://github.com/microsoft/winget-pkgs
[komac-repo]: https://github.com/russellbanks/komac
