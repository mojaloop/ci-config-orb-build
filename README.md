# mojaloop/build

[![CircleCI Build Status](https://circleci.com/gh/mojaloop/ci-config-orb-build.svg?style=shield "CircleCI Build Status")](https://circleci.com/gh/mojaloop/ci-config-orb-build)
[![CircleCI Orb Version](https://badges.circleci.com/orbs/mojaloop/build.svg)](https://circleci.com/developer/orbs/orb/mojaloop/build)
[![GitHub License](https://img.shields.io/badge/license-APACHE_2.0-lightgrey.svg)](https://raw.githubusercontent.com/mojaloop/ci-config-orb-build/main/LICENSE)

_**mojaloop/build is a CircleCI orb for node.js build jobs in Mojaloop CI/CD pipelines**_

## Usage

To use the `mojaloop/build` orb in your CircleCI configuration, turn on
`Enable dynamic config using setup workflows` in the `Advanced Settings` of your
project settings CircleCI. Then include the following in your `.circleci/config.yml`:

```yaml
version: 2.1
setup: true
orbs:
  build: mojaloop/build@1.1.17
workflows:
  setup:
    jobs:
      - build/workflow:
          context: org-global
          filters:
            tags:
              only: /v\d+(\.\d+){2}(-[a-zA-Z-][0-9a-zA-Z-]*\.\d+)?/
          # optionally supply the resource_class for the jobs
          # pr_title_check_resource_class: medium
          # setup_resource_class: medium
          # test_dependencies_resource_class: medium
          # test_deprecations_resource_class: medium
          # test_lint_resource_class: medium
          # test_unit_resource_class: medium
          # test_coverage_resource_class: medium
          # vulnerability_check_resource_class: medium
          # license_audit_resource_class: medium
          # build_local_resource_class: medium
          # test_integration_resource_class: medium
          # test_functional_resource_class: medium
          # license_scan_resource_class: medium
          # grype_scan_resource_class: medium
          # release_resource_class: medium
          # github_release_resource_class: medium
          # publish_docker_resource_class: medium
          # publish_npm_resource_class: medium
          # publish_docker_snapshot_resource_class: medium
          # publish_npm_snapshot_resource_class: medium
          # publish_npm_prerelease_resource_class: medium
          # publish_docker_prerelease_resource_class: medium
```

### Vulnerability Scan Configuration

The repo using the orb must declare a `.grype.yaml` file in the root of the repo.

#### Scan Type

**Source code scan is the default for all repos.** No `scan-type` field is needed — just omit it.

To opt into Docker image scanning instead, set `scan-type: image` in `.grype.yaml`. Image scanning requires a custom workflow override to ensure the Build job completes before Grype scan (since Grype runs in parallel with Build by default).

To disable scanning entirely, set `disabled: true`.

#### Example `.grype.yaml`

```yaml
# No scan-type needed — source scan is the default for all repos
# Set to true to disable Grype scanning completely
disabled: false

ignore:
  # Ignore cross-spawn vulnerabilities by CVE ID due to false positive
  - vulnerability: "GHSA-3xgq-45jj-v275"
    package:
      name: "cross-spawn"

quiet: false
check-for-app-update: false
```

### Additional Workflows

The orb supports merging additional workflows and jobs into the standard build pipeline. This feature allows you to extend the CI/CD pipeline with custom jobs while maintaining the benefits of the standard workflow.

To use this feature, create a `.circleci/additional-workflows.yaml` file in your repository root. The orb will automatically detect this file and merge it with the standard workflow configuration.

#### Example: Adding a Custom Job

Create `.circleci/additional-workflows.yaml`:

```yaml
jobs:
  custom-build:
    docker:
      - image: cimg/base:stable
    steps:
      - checkout
      - run:
          name: Custom build step
          command: |
            echo "Running custom build logic"
            # Add your custom commands here

workflows:
  build_and_test:
    jobs:
      - custom-build:
          name: Custom Build Job
          context: org-global
          filters:
            tags:
              only: /v\d+(\.\d+){2}/
          requires:
            - Lint
            - Unit tests
```

#### How Additional Workflows Merge Works

1. The orb first generates the standard `build_and_test` workflow
2. If `.circleci/additional-workflows.yaml` exists, it merges the additional configuration
3. Jobs from the additional file are added to the existing workflow
4. Workflow definitions are merged, allowing you to extend the standard pipeline

This feature is particularly useful for:
- Building additional Docker images
- Adding custom deployment steps
- Running specialized tests or scans
- Integrating with external services

#### Important Notes

- The additional workflows file must be valid YAML
- Job names should be unique to avoid conflicts
- You can reference standard jobs in `requires` sections (e.g., `Lint`, `Unit tests`, `Integration tests`, etc.)
- The merge process uses deep merge, so nested configurations are properly combined

### Conventions

- If a `Dockerfile` is present in the root of the repository, it will be used to
  build and publish an image.
- If a `package.json` is present in the root of the repository and it does not have
  private=true, the package will be published to npm for
  the applicable branches and tags.
- The following scripts are expected in the `package.json` file:
  - `lint`
  - `dep:check`
  - `audit:check`
  - `test:xunit`
  - `test:integration`
  - `test:functional`
  - `test:coverage-check`

### Publish conditions

The orb will trigger a publish when one of the following conditions is met:

- A release tag (e.g. `v#.#.#`) is pushed to the repository.
- A snapshot tag (e.g. `v#.#.#-snapshot.#`, `v#.#.#-hotfix.#`, , `v#.#.#-perf.#`)
  is pushed to the repository.
- A commit to a pre-release branch (e.g. `major/*`, `minor/*`, `patch/*`) passed
  the tests.

---

## Resources

[CircleCI Orb Registry Page](https://circleci.com/developer/orbs/orb/mojaloop/build) -
The official registry page of this orb for all versions, executors, commands,
and jobs described.

[CircleCI Orb Docs](https://circleci.com/docs/orb-intro/#section=configuration) -
Docs for using, creating, and publishing CircleCI Orbs.

### How to Contribute

We welcome [issues](https://github.com/mojaloop/ci-config-orb-build/issues) to
and [pull requests](https://github.com/mojaloop/ci-config-orb-build/pulls)
against this repository!

### How to Publish An Update

1. Merge pull requests with desired changes to the main branch.
   - For the best experience, squash-and-merge and use [Conventional Commit Messages](https://conventionalcommits.org/).
2. Find the current version of the orb.
   - You can run `circleci orb info mojaloop/build | grep "Latest"` to see the
     current version.
3. Create a [new Release](https://github.com/mojaloop/ci-config-orb-build/releases/new)
   on GitHub.
   - Click "Choose a tag" and _create_ a new [semantically versioned](http://semver.org/)
     tag. (ex: v1.0.0)
   - We will have an opportunity to change this before we publish if needed
     after the next step.
4. Click _"+ Auto-generate release notes"_.
   - This will create a summary of all of the merged pull requests since the
     previous release.
   - If you have used _[Conventional Commit Messages](https://conventionalcommits.org/)_
     it will be easy to determine what types of changes were made, allowing you
     to ensure the correct version tag is being published.
5. Now ensure the version tag selected is semantically accurate based on the
   changes included.
6. Click _"Publish Release"_.
   - This will push a new tag and trigger your publishing pipeline on CircleCI.
