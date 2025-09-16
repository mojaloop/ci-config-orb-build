@ -0,0 +1,187 @@
# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is the Mojaloop Build CircleCI Orb (`mojaloop/build`) - a reusable CI/CD configuration package for standardizing build, test, and deployment workflows across Mojaloop Node.js projects. The orb provides a comprehensive set of jobs and commands for continuous integration pipelines.

## Architecture

The repository follows CircleCI Orb structure:
- **src/@orb.yml**: Main orb configuration defining version, description, and external orb dependencies
- **src/commands/**: Reusable command definitions for common CI tasks
- **src/jobs/**: Job definitions for various CI/CD stages (build, test, publish, etc.)
- **src/workflows/**: Workflow templates that orchestrate jobs
- **src/executors/**: Execution environments (Docker, machine)

## Key Components

### Jobs
- **Setup & Dependencies**: `setup`, `test_dependencies`, `test_deprecations`
- **Testing**: `test_unit`, `test_integration`, `test_functional`, `test_coverage`, `test_lint`
- **Security**: `vulnerability_check`, `audit_licenses`, `license_scan`, `grype_scan`
- **Build**: `build_local`
- **Publishing**: `publish_docker`, `publish_npm` (with snapshot and prerelease variants)
- **Release**: `release`, `github_release`

### Commands
- **Environment Setup**: `configure_nvm`, `configure_git`, `npm_auth`
- **Dependencies**: `aws_dependencies`, `docker_dependencies`, `machine_dependencies`
- **Validation**: `check_dockerfile`, `check_npm_private`, `pr_title_check`
- **Publishing**: `npm_publish`, `trigger_downstream`
- **Utilities**: `slack_pass_fail`, `export_version_from_package`

## Development Commands

### Testing the Orb Locally
```bash
# Validate orb configuration
circleci orb validate src/@orb.yml

# Pack the orb for testing
circleci orb pack src/ > orb.yml

# Process config with local orb
circleci config process .circleci/config.yml
```

### Publishing Updates
```bash
# Check current version
circleci orb info mojaloop/build | grep "Latest"

# Publish development version for testing
circleci orb publish src/ mojaloop/build@dev:first

# Promote to production (done via GitHub release)
circleci orb publish promote mojaloop/build@dev:first patch
```

## Orb Workflow Parameters

The orb accepts parameters for customizing resource classes for each job. Available parameters follow the pattern:
- `{job_name}_resource_class`: Controls the compute resources (small, medium, large, xlarge, 2xlarge)

## Publishing Triggers

The orb publishes artifacts based on:
- **Release tags**: `v#.#.#` triggers Docker and npm releases
- **Snapshot tags**: `v#.#.#-snapshot.#`, `v#.#.#-hotfix.#`, `v#.#.#-perf.#`
- **Prerelease branches**: `major/*`, `minor/*`, `patch/*`

## Additional Workflows

Projects can extend the standard workflow by creating `.circleci/additional-workflows.yaml` in their repository. The orb will automatically merge these custom jobs into the build pipeline.

## Grype Vulnerability Scanning

Projects must include a `.grype.yaml` configuration file to control vulnerability scanning. The `grype_scan` job automatically determines whether to scan Docker images or source code based on configuration.

### Scan Type Configuration

The orb supports both Docker image scanning and source code scanning in a single job:

1. **Explicit Configuration**: Add `scan-type` to `.grype.yaml`:
   ```yaml
   scan-type: source  # For library repos without Docker images
   # OR
   scan-type: image   # For repos that build Docker images
   ```

2. **Auto-detection** (when `scan-type` not specified):
   - If `Dockerfile` exists → performs image scan
   - If no `Dockerfile` → performs source code scan

3. **Disable Scanning**: Set `disabled: true` to skip all scanning

### Scan Types

- **Image Scan** (when `scan-type: image` or Dockerfile exists):
  - Scans the Docker image built in the Build job
  - Requires Docker image tar in workspace from Build job
  - Uses: `grype <image-name>`

- **Source Scan** (when `scan-type: source` or no Dockerfile):
  - Scans source code and dependencies directly
  - Analyzes package.json, package-lock.json, and other dependency files
  - Uses: `grype dir:.`

### Configuration Options

- Use `ignore` section to suppress false positives by CVE ID
- Configure output formats (table, json)
- Set search scope for image scans (`squashed`, `all-layers`)

## Expected Project Structure

Projects using this orb should have:
- `package.json` with scripts: `lint`, `dep:check`, `audit:check`, `test:xunit`, `test:integration`, `test:functional`, `test:coverage-check`
- `Dockerfile` (if building Docker images)
- `.nvmrc` for Node.js version management
- `.grype.yaml` for vulnerability scanning configuration

## Repository Maintenance Scripts

### Automated Grype Configuration Updates

Three bash scripts are available for automating grype configuration updates across Mojaloop library repositories:

#### 1. `update-grype-configs.sh`
Main script that processes all Mojaloop library repositories to:
- Check if repository uses mojaloop/build orb
- Update orb version to v1.1.2
- Add `scan-type: source` to `.grype.yaml`
- Update dependencies (`npm i`, `npm run dep:update`)
- Create development branch `chore/grype-src-scan-pi28`
- Commit changes with conventional commit format

#### 2. `update-grype-already-updated.sh`
Specialized script for repositories that already have orb v1.1.2 but need grype configuration:
- Processes specific repos (central-services-error-handling, central-services-stream, etc.)
- Skips repositories with Dockerfiles (not libraries)
- Adds/updates `scan-type: source` in `.grype.yaml`
- Runs dependency updates
- Creates commits on development branch

#### 3. `check-repo-status.sh`
Status checking script to verify update progress:
- Checks for existence of development branch
- Verifies `.grype.yaml` configuration
- Lists repositories ready to push
- Generates push commands for updated repos

### Usage Example
```bash
# Run main update script
./update-grype-configs.sh

# Process repos that already have orb v1.1.2
./update-grype-already-updated.sh

# Check status of all updates
./check-repo-status.sh

# Push changes (manual step per repo)
cd /Users/jc/ml/{repo-name}
git push -u origin chore/grype-src-scan-pi28
```

### Library Repositories Managed

The following Mojaloop libraries (no Dockerfile) are managed by these scripts:
- central-services-shared
- central-services-error-handling
- central-services-database
- central-services-stream
- central-services-metrics
- central-services-error
- central-services-logger
- sdk-standard-components
- api-snippets
- ml-number
- object-store-lib
- inter-scheme-proxy-cache-lib
- database-lib

Note: Only repositories using the mojaloop/build orb and without Dockerfiles are updated.