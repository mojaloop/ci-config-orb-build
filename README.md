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
  build: mojaloop/build@1.0.44
workflows:
  setup:
    jobs:
      - build/workflow:
          filters:
            tags:
              only: /v\d+(\.\d+){2}(-[a-zA-Z-][0-9a-zA-Z-]*\.\d+)?/
          # optionally supply the base image for the image scan
          # base_image: org/image

```

Conventions:

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
