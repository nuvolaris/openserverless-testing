# Operator Testing Specification

## Goal

Provide a release-oriented testing flow between:

- `openserverless-operator`
- `openserverless-testing`

so that:

1. pushing an operator release tag builds and publishes the operator image
2. PR-triggered operator testing is enabled only by specific PR tags/labels
3. all operator tests run in `openserverless-testing`
4. a successful operator release build triggers `openserverless-testing`
5. `openserverless-testing` materializes release-style test tags such as:
   - `kind-0.1.0-incubating.123456`
   - `k3s-0.1.0-incubating.123456`
6. the logs clearly show which operator tag was tested on which target

## Canonical Contract

### Operator release tag

The operator release tag is the source of truth for a releasable build:

- `0.1.0-incubating.123456`

### Testing tag

The testing repository receives one tag per target platform:

- `<test>-<operator-tag>`

Examples:

- `kind-0.1.0-incubating.123456`
- `k3s-0.1.0-incubating.123456`
- `eks-0.1.0-incubating.123456`

Meaning:

- run the `kind`, `k3s`, or `eks` testing suite
- using operator image tag `0.1.0-incubating.123456`

## Required Operator Flows

### 1. Release tag flow

Trigger:

- `push.tags: [0-9]*`

Expected behavior:

1. checkout code with submodules
2. resolve operator image repository
3. build the operator image
4. publish the image
6. trigger `openserverless-testing`
7. request generation of one tag per target, at least:
   - `kind-<operator-tag>`
   - `k3s-<operator-tag>`

### 2. PR tag-triggered flow

Trigger:

- `pull_request_target` on a PR carrying a testing tag/label such as:
  - `kind`
  - `k3s`
  - `eks`

Expected behavior:

1. resolve the PR head SHA and head ref
2. dispatch the request to `openserverless-testing`
3. let `openserverless-testing` build the PR image and run the requested suite

## Required Testing Repository Flows

### 1. Dispatch-to-tag bridge

`openserverless-testing` must accept a dispatch request from `openserverless-operator` and create one testing tag per requested target.

Minimum payload:

- `operator_tag`
- `targets`
- optional `operator_image`

### 2. Release tag execution

`openserverless-testing` must react to pushed tags in the format:

- `<test>-<operator-tag>`

For these runs it must:

1. resolve the test selector from the tag prefix
2. resolve the operator tag from the suffix
3. build or patch an `OPS_ROOT` that uses:
   - `${OPERATOR_IMAGE}:${OPERATOR_TAG}`
4. run the matching suite
5. print in the logs exactly which testing tag is under execution

## Operator Image Variable

The testing flow must support a configurable operator image repository.

Default:

- `apache/openserverless-operator`

Allowed override examples:

- `nuvolaris/openserverless-operator`
- `ghcr.io/nuvolaris/openserverless-operator`

The operator image repository and the operator tag together define the image under test:

- `${OPERATOR_IMAGE}:${OPERATOR_TAG}`

## Logging Requirements

At minimum, logs must show:

- the operator image repository
- the operator tag
- the generated testing tags
- the exact testing tag currently running

Example expected log lines:

- `Operator image: apache/openserverless-operator:0.1.0-incubating.123456`
- `Dispatching testing tags: kind-0.1.0-incubating.123456 k3s-0.1.0-incubating.123456`
- `Running testing tag: kind-0.1.0-incubating.123456`

## Gaps Closed By This Spec

This specification supersedes the older contracts based on:

- PR labels like `<platform>-<architecture>`
- PR labels like `<test>-<hash>`

For release validation the canonical flow is now:

- operator release tag -> publish operator image -> testing repo tags -> downstream infrastructure tests

For PR validation the canonical flow is:

- PR tag/label -> testing repo build/test

## Minimum Deliverables

1. `openserverless-operator` workflow for release-tag build/publish/dispatch
2. `openserverless-operator` workflow for PR tag-triggered dispatch
3. `openserverless-testing` workflow that builds/tests operator PRs
4. `openserverless-testing` workflow that creates release-style testing tags on dispatch
5. `openserverless-testing` support for parsing `<test>-<operator-tag>`
6. `openserverless-testing` support for injecting the operator image repository under test
