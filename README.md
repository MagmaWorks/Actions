# Actions
Shared github actions for public repos in MagmaWorks.

Three workflows are available: On Pull Request, On Merge to Main and On Release.


### [On Pull Request](https://github.com/MagmaWorks/Actions/blob/main/.github/workflows/on-pull-request.yml)

#### Inputs
Accepted inputs (all optional) are:
- `codecov: false` (default = true) - set if the workflow shall upload test coverage reports to [codecov.io](https://app.codecov.io/gh/MagmaWorks)
- `lint: false` (default = true) - set if the workflow shall lint (auto-fix errors and style) using `dotnet format`
- `dotnet: '8.0.x'` (default not set) - set if the workflow shall install a specific dotnet version

#### Steps
- Lint (auto-fix errors and style) 
  - Setup dotnet (optional)
  - Lint (optional)
- Build and test project
  - Setup dotnet (optional)
  - Build project in release `dotnet build --configuration Release`
  - Test project in release
  - Upload coverage reports to workflow as artifacts
- Upload coverage reports to codecov.io (optional)
  - Download coverage reports from workflow artifacts
  - Upload coverage reports to codecov.io


### [On Merge to Main](https://github.com/MagmaWorks/Actions/blob/main/.github/workflows/on-merge-to-main.yml)

#### Inputs
Accepted inputs (all optional) are:
- `codecov: false` (default = true) - set if the workflow shall upload test coverage reports to [codecov.io](https://app.codecov.io/gh/MagmaWorks)(https://github.com/MagmaWorks/Actions/blob/main/.github/workflows/on-pull-request.yml)
- `dotnet: '8.0.x'` (default not set) - set if the workflow shall install a specific dotnet version

#### Steps
- Build, test, package and create release
  - Setup dotnet (optional)
  - Build project in release
  - Test project in release
  - Upload coverage reports to workflow as artifacts
  - Package project
  - Get version from Package name
  - Create and push new Major.Minor.Patch.Build tag
  - Create new draft release
- Upload coverage reports to codecov.io (optional)
  - Download coverage reports from workflow artifacts
  - Upload coverage reports to codecov.io


### [On Merge to Main](https://github.com/MagmaWorks/Actions/blob/main/.github/workflows/on-release.yml)

#### Inputs
Accepted inputs (all optional) are:
- `codecov: false` (default = true) - set if the workflow shall upload test coverage reports to [codecov.io](https://app.codecov.io/gh/MagmaWorks)(https://github.com/MagmaWorks/Actions/blob/main/.github/workflows/on-pull-request.yml)
- `dotnet: '8.0.x'` (default not set) - set if the workflow shall install a specific dotnet version

#### Steps
- Tag version
  - Strip Build from Tag version (`4.3.2.1` => `4.3.2`)
  - Create and push new Major.Minor.Patch git tag
- Build, test, package and create release
  - Setup dotnet (optional)
  - Build project in release
  - Test project in release
  - Upload coverage reports to workflow as artifacts
  - Package project
  - Update draft release to latest, non-draft
  - Push nupkg file to nuget.org
  - Delete previous draft releases
- Upload coverage reports to codecov.io (optional)
  - Download coverage reports from workflow artifacts
  - Upload coverage reports to codecov.io
