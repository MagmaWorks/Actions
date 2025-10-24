# Actions
### A repo for shared github actions

The **CI Dispatcher** is an easy-to-use, automated CI tool designed to run GitHub Actions workflows from events from your local repo.  

---

### What You Get

- **Build** the project on pull requests - preventing merging code to main that does not compile.
- Run **Test** to ensure all test pass succesfully before merging
- **Coverage** using [codecov.io](https://app.codecov.io/gh/MagmaWorks) to track codecoverage which is reported in each PR.
- A **Draft release** with automated **semantic versioning** is created in your repo when merging to main.
- Automatically tag and push **NuGet** package when releasing a package.
- **Unify CI logic** across repositories — no need to duplicate `.github/workflows`.  

---

## Contents

- [How to use](#how-to-use) – See how to integrate and call the dispatcher from your repository.  
- [How it works](#how-it-works) – Dive into the internal logic and event flow of the dispatcher.

---

# How to use

Add this single workflow file in your repo at:
.github/workflows/ci.yml

```yaml
name: CI using shared actions

on:
  push:
    branches: [ main ]
  pull_request:
    types: [opened, edited, synchronize]
  release:
    types: [published]

jobs:
  ci:
    uses: MagmaWorks/Actions/.github/workflows/ci-dotnet.yml@main
    secrets: inherit
```

That’s it — this one file triggers the correct pipeline for:
- Pull Requests: lint, build, test, and upload coverage
- Merge to main: build, test, package, and create a draft release
- Published Releases: strip build numbers, retag, and push to NuGet

### Optional inputs
You can override default behaviour directly in the job call:

```yaml
jobs:
  ci:
    uses: MagmaWorks/Actions/.github/workflows/ci-dotnet.yml@main
    secrets: inherit
jobs:
  ci:
    uses: MagmaWorks/Actions/.github/workflows/ci-dotnet.yml@main
    secrets: inherit
    with:
      dotnet: '8.0.x'    # Optional .NET version
      lint: false        # Disable linting
      codecov: false     # Skip Codecov upload
```

# How it works
All logic runs through `ci-dotnet.yml`, which dispatches to specialised workflows depending on the event type.

ci-dotnet.yml
| Event                 | Workflow                                                                                                               | Description                                      |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------ |
| `pull_request`        | [`dotnet-build-test.yml`](https://github.com/MagmaWorks/Actions/blob/main/.github/workflows/dotnet-build-test.yml)     | Lints, builds, tests, and uploads coverage       |
| `push` (merge to main)         | [`dotnet-pack-release.yml`](https://github.com/MagmaWorks/Actions/blob/main/.github/workflows/dotnet-pack-release.yml) | Packages the project and creates a draft release |
| `release` (published) | [`release-push-nuget.yml`](https://github.com/MagmaWorks/Actions/blob/main/.github/workflows/release-push-nuget.yml)   | Republishes release assets to NuGet.org          |


### dotnet-build-test.yml

Runs on PRs and pushes.
Steps:
- Checkout repository
- Setup .NET (optional)
- Run dotnet format (optional lint step)
- Build in Release configuration
- Test with coverage collection
- Upload coverage artifact
- Outputs: coverage → consumed by Codecov upload job

### codecov-upload.yml

Uploads coverage reports to codecov.io.
Uses organisation secret `CODECOV_TOKEN` (without having it passed down).

Steps:
- Checkout
- Download coverage artifact
- Upload to Codecov

### dotnet-pack-release.yml

Triggered on push to main.
Builds and packages all .nupkg / .snupkg files, removes any -preview suffixes, and creates or updates a draft release tagged as Major.Minor.Patch.Build.

Steps:
- Checkout with full history
- Setup .NET (optional)
- dotnet pack in Release mode
- Parse version from package name
- Create or update draft GitHub release

### release-push-nuget.yml

Triggered on release publication.
Republishes the previously drafted packages to NuGet and cleans up old drafts.

Requires: `NUGET_API_KEY` secret which must be passed from the local repo stub. By passing `secrets: inherit` in the local repo stub we pass the global organisation secret.

Steps:
- Download release assets
- Strip build number from tag (1.2.3.4 → 1.2.3)
- Update .nuspec version and repackage
- Push version tag to Git
- Push .nupkg to NuGet.org
- Update release (finalize + attach new assets)
- Delete old draft releases


### Notes

- Supports .NET 8 and newer
- Reusable workflows can be called directly if fine-grained control is needed, or mixed with private repo requirements.
- Default behaviour should cover 99% of use cases: PR validation, release drafting, and NuGet publishing
- All jobs run on ubuntu-latest
