Shouldn't it be easy to semantically bump a version tag and create a release?

Two excellent actions exist for these respective tasks:
- mathieudutour/github-tag-action
- ncipollo/release-action

This reusable workflow combines the two for ease of use. Each action's inputs are mostly exposed. Some opinionated parameters are applied. Caveat emptor.

### Example

```yml
name: release

on:
  push:
    branches: [main]

permissions:
  contents: write  # Needed to push a new tag.

jobs:
  release:
    environment: production
    if: github.event_name == 'push'
    uses: w0de/autoversion-autorelease-action/.github/workflows/tag-and-release.yml@main
    with:
      create_annotated_tag: true
    secrets:
      token: ${{ github.token }}
```

### Inputs

```yml
    secrets:
      token:
        description: The github token to use when applying tag.
        required: true

    inputs:
      # mathieudutour/github-tag-action
      create_annotated_tag:
        description: Create an annotated rather than a lightweight tag.
        required: false
        default: false
      default_bump:
        description: Can be patch, minor or major.
        required: false
        default: patch
      release_branches:
        description: Comma separated list of branches (JavaScript regular expression accepted) that will generate the release tags.
        required: false
        default: main
      pre_release_branches:
        description: Comma separated list of branches (JavaScript regular expression accepted) that will generate the release tags.
        required: false
        default: .*
      tag_prefix:
        required: false
        default: v

      # ncipollo/release-action
      allowUpdates:
        description: An optional flag which indicates if we should update a release if it already exists. Defaults to false.
        required: false
        default: false
      artifactErrorsFailBuild:
        description: An optional flag which indicates if artifact read or upload errors should fail the build.
        required: false
        default: true
      artifacts:
        description: An optional set of paths representing artifacts to upload to the release. This may be a single path or a comma delimited list of paths (or globs)
        required: false
        default: ''
      artifactContentType:
        description: The content type of the artifact. Defaults to raw
        required: false
        default: ''
      draft:
        description: Optionally marks this release as a draft release. Set to true to enable.
        required: false
        default: false
      generateReleaseNotes:
        description: Generate release notes instead of using tag step's changelog.
        required: false
        default: false
      makeLatest:
        description: Indicates if the release should be the latest release or not.
        required: false
        default: true
      name:
        description: Override default version tag name with this custom value.
        required: false
        default: null
      updateOnlyUnreleased:
        description: When allowUpdates is enabled, this will fail the action if the release it is updating is not a draft or a prerelease.
        required: false
        default: false

      # both
      commit_sha:
        description: The commit SHA value to add the tag. If specified, it uses this value instead GITHUB_SHA. It could be useful when a previous step merged a branch into github.ref.
        required: false
        default: null
      dry_run:
        description: |
          Generate new version tag and output to log. One of:
          - false
          - true (output putative new tag to log)
          - "draft" (apply new version tag prefixed with "draft_", create draft release)
          - "draft-cleanup" (same as "draft", but release and tag are deleted after creation)
        required: false
        default: false
```

### Outputs

```yml
    outputs:
      # mathieudutour/github-tag-action
      changelog:
        description:  The conventional changelog since the previous tag.
        value: ${{ jobs.main.outputs.changelog }}
      new_version:
        description: The auto-bumped version.
        value: ${{ jobs.main.outputs.new_version }}
      new_tag:
        description: The auto-bumped version tag.
        value: ${{ jobs.main.outputs.new_tag }}
      release_type:
        description: The auto-bumped versions release type.
        value: ${{ jobs.main.outputs.release_type }}

      # ncipollo/release-action
      artifact_upload_url:
        description: The URL for uploading assets to the release.
        value: ${{ jobs.main.outputs.artifact_upload_url }}
      release_id:
        description: The identifier of the created release.
        value: ${{ jobs.main.outputs.release_id }}
      release_url:
        description: The HTML URL of the release.
        value: ${{ jobs.main.outputs.release_url }}
```
