<h1> <img src="https://github.com/vedantmgoyal9/winget-releaser/blob/main/.github/github-actions-logo.png" width="32" height="32" alt="Logo" /> WinGet Updater (GitHub Action) </h1>

[![GitHub release (latest by date)](https://img.shields.io/github/v/release/michidk/winget-updater?logo=github)](https://github.com/michidk/winget-updater/releases)
[![GitHub](https://img.shields.io/github/license/michidk/winget-updater)](https://github.com/michidk/winget-updater?tab=MIT-1-ov-file#readme)

A GitHub action which automatically updates WinGet packages, based on Komac.

This project is heavily inspired by [vedantmgoyal2009/winget-releaser](https://github.com/vedantmgoyal2009/winget-releaser).
The main differences are:

- This action will update your package to the latest release available on GitHub automatically. It does not need to react to an `on: release` event in GitHub actions.
- It is a composite action, which consists of other GitHub actions, which makes it very simple (no big Typescript project).

## Getting Started 🚀

1. At least **one** version of your package should already be present in the [Windows Package Manager Community Repository](https://github.com/microsoft/winget-pkgs). The action will use that version as a base to create manifests for new versions of the package.

2. You will need to create a _classic_ Personal Access Token (PAT) with `public_repo` scope. _New_ fine-grained PATs aren't supported by the action. Review [this issue](https://github.com/vedantmgoyal2009/winget-releaser/issues/172) for information.

3. Fork the [winget-pkgs](https://github.com/microsoft/winget-pkgs) repository under the same account/organization as your repository on which you want to use this action. Ensure that the fork is up-to-date with the upstream repository (see [this issue](https://github.com/vedantmgoyal2009/winget-releaser/issues/32) for why this is important). You can do this using one of the following methods:
https://github.com/vedantmgoyal2009/winget-releaser
- Give `workflow` permission to the token you created in Step 1. This will allow the action to automatically update your
  fork with the upstream repository.
- You can use **[Pull App](https://github.com/wei/pull)** which keeps your fork up-to-date with the upstream repository via automated pull requests.

4. Add the action to your workflow file (e.g. `.github/workflows/<name>.yml`).

> [!IMPORTANT]
> The action will only work when the release is **published** (not a draft), because the release assets (binaries) aren't available publicly until the release is published.

> [!NOTE]
> In case you're pinning the action to a commit hash, you'll need to update the hash frequently to get the latest features & bug fixes. Therefore, it is **highly** recommended to setup dependabot auto-updates for your repository. Check out [keeping your actions up to date with Dependabot](https://docs.github.com/en/actions/security-guides/encrypted-secrets#using-encrypted-secrets-in-a-workflow) for guidance on how to do this. (Yes, it also supports updating actions pinned to a commit hash!)


## 📖 Example Usage

Minimal example:

```yaml
name: Update WinGet Packages

on: workflow_dispatch

jobs:
  update:
    name: Update Package
    runs-on: ubuntu-latest
    steps:
    - name: Update Packages
      uses: michidk/winget-updater@v1
      with:
        komac-token: ${{ secrets.KOMAC_TOKEN }}
        identifier: "michidk.vscli"
        repo: "michidk/vscli"
        url: "https://github.com/michidk/vscli/releases/download/v{VERSION}/vscli-x86_64-pc-windows-msvc.zip"
```

Use a matrix to update multiple packages at once. Can also be combined with [Run Komac](https://github.com/michidk/run-komac) to automatically clean up branches after a PR has been merged:

```yaml
name: Update WinGet Packages

on:
  workflow_dispatch:
  schedule:
    - cron:  '0 3 * * *' # Scheduled to run daily at 03:00

jobs:
  update:
    name: Update package ${{ matrix.id }}
    runs-on: ubuntu-22.04

    strategy:
      matrix:
        include:
          - id: "Casey.Just"
            repo: "casey/just"
            url: "https://github.com/casey/just/releases/download/{VERSION}/just-{VERSION}-x86_64-pc-windows-msvc.zip"
          - id: "michidk.vscli"
            repo: "michidk/vscli"
            url: "https://github.com/michidk/vscli/releases/download/v{VERSION}/vscli-x86_64-pc-windows-msvc.zip"
            # Multi URL package entry
          - id: "michidk.vscli"
            repo: "michidk/vscli"
            url: '"https://github.com/michidk/vscli/releases/download/v{VERSION}/vscli-x86_64-pc-windows-msvc.zip https://github.com/michidk/vscli/releases/download/v{VERSION}/vscli-i686-pc-windows-msvc.zip"'

    steps:
    - name: Update Packages
      uses: michidk/winget-updater@latest
      with:
        komac-token: ${{ secrets.KOMAC_TOKEN }}
        identifier: ${{ matrix.id }}
        repo: ${{ matrix.repo }}
        url: ${{ matrix.url }}

  cleanup:
    name: Cleanup branches
    needs: update # Not necessarily needed as PRs don't get closed that quick but still nice to have it in order
    runs-on: ubuntu-22.04

    steps:
    - name: Run Komac
      uses: michidk/run-komac@latest
      with:
        args: 'cleanup --only-merged --token=${{ secrets.KOMAC_TOKEN }}'
```

For a real-world example, have a look at my WinGet package updater repository: [michidk/winget](https://github.com/michidk/winget)

## ⚒️ Configuration Options

- `komac-version`: Specifies which version of Komac to use.
  - **Required**: ❌
  - **Default**: `2.6.0`
- `komac-token`: The GitHub token to use for authentication. The token should have the `public_repo` scope.
  - **Required**: ✅
  - ⚠ **WARNING**: Do **not** directly put the token in the action. Instead, create a repository secret containing the token and use that in the workflow. Refer to [using encrypted secrets in a workflow](https://docs.github.com/en/actions/security-guides/encrypted-secrets#using-encrypted-secrets-in-a-workflow) for more information.
- `identifier`: The package identifier of the package to be updated in the [WinGet Community Repository](https://github.com/microsoft/winget-pkgs).
  - **Required**: ✅
  - **Example**: `michidk.vscli`
- `repo`: The GitHub repository to check for the latest release.
  - **Required**: ✅
  - **Example**: `michidk/vscli`
- `url`: The URL(s) to the latest release. Use the placeholder `{VERSION}` to specify where the version should be inserted. The placeholder contains the version without `v`, e.g. `1.2.3`. Multiple URLs may be specified by wrapping a space-separated string in single and double quotes as in the second example below.
  - **Required**: ✅
  - **Example 1**: `https://github.com/michidk/vscli/releases/download/v{VERSION}/vscli-x86_64-pc-windows-msvc.zip`
  - **Example 2**: `url: '"https://github.com/michidk/vscli/releases/download/v{VERSION}/vscli-x86_64-pc-windows-msvc.zip https://github.com/michidk/vscli/releases/download/v{VERSION}/vscli-i686-pc-windows-msvc.zip"'`
- `version`: Manually specify the version. This is useful for cases where the latest release's tag doesn't follow the `vX.X.X` or `x.x.x` patterns.
  - **Required**: ❌
  - **Example**: `1.0.0`
- `custom-fork-owner`: The owner of the `winget-pkgs` repo fork to use. If not specified, the user that created the `komac-token`.
  - **Required**: ❌
  - **Example**: `michidk`
- `ghes-token`: Token for GHES authentification.
  - **Required**: ❌

<h2> 🚀 Integrating with <a href="https://github.com/russellbanks/Komac"> <img src="https://rawcdn.githack.com/michidk/winget-updater/7ef56d9c40feb29e1592c0bf6c65eb1af3e77d4e/.github/images/komac-logo.svg" height="24px" style="vertical-align:bottom" alt="Komac logo" /> </a></h2>

This GitHub action leverages [Komac](https://github.com/russellbanks/komac) to generate and submit manifests to the [Windows Package Manager Community Repository](https://github.com/microsoft/winget-pkgs). Kudos to [Russell Banks](https://github.com/russellbanks) for developing Komac which powers this action.
Also huge thanks to [vedantmgoyal2009](https://github.com/vedantmgoyal2009) for creating an awesome GitHub action, which this one is heavily inspired from.
