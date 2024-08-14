Shouldn't it be easy to semantically bump a version tag and create a release?

Two excellent actions exist for these respective tasks:
- mathieudutour/github-tag-action
- ncipollo/release-action

This reusable workflow combines the two for ease of use. Each action's inputs are mostly exposed. Some opinionated parameters are applied. Caveat emptor.

### Example

#### Create releases with pushes to main
```yml
name: release

on:
  push:
    branches: [main]
  
permissions:
  contents: write  # Required push tags.

jobs:
  release:
    environment: production
    uses: w0de/autoversion-autorelease-action/.github/workflows/tag-and-release.yml@main
    with:
      release_branches: main
```

#### Create prereleases with opened pull requests
```yml
name: release

on:
  pull_request:
    branches: [main]
    types: [opened]
  
permissions:
  contents: write  # Required push tags.

jobs:
  release:
    uses: w0de/autoversion-autorelease-action/.github/workflows/tag-and-release.yml@main
    with:
      release_branches: main
```

### Release types

mathieudutour/github-tag-action uses semantic versioning. Valid release types: patch, minor, major. Each may be prefixed by "pre" for prereleases.

To set default release type, use `default_bump`. This is fallback for parsed bump type - mathieudutour/github-tag-action [parses commit messages to determine bump type](https://github.com/mathieudutour/github-tag-action?tab=readme-ov-file#bumping) ("fix:" -> patch, "feat:" -> minor, "BREAKING CHANGE" -> major).

### Inputs

```yml
      artifacts:
        description: An optional set of paths representing artifacts to upload to the release. This may be a single path or a comma delimited list of paths (or globs)
        required: false
        type: string
      artifacts_content_type:
        description: The content type of the artifacts. Defaults to raw.
        required: false
        type: string
        default: raw
      bump_from_branch_name:
        description: Adjust semantic version bump based on branch name prefix. Branches prefixed with "patch/", "minor/", or "major/" will adjust version bump.
        required: false
        default: false
        type: boolean
      commit_sha:
        description: The commit SHA value to add the tag. If specified, it uses this value instead GITHUB_SHA. It could be useful when a previous step merged a branch into github.ref.
        required: false
        type: string
      default_bump:
        description: Can be patch, minor or major.
        required: false
        default: patch
        type: string
      draft_pre_releases:
        description: Create draft releases for prerelease branches.
        required: false
        default: true
        type: boolean
      dry_run:
        description: Output putative new tag to log. Do not apply tag or create release.
        required: false
        default: false
        type: boolean
      name:
        description: Override default release with this custom value.
        required: false
        type: string
      release_branches:
        description: Comma separated list of branches (JavaScript regular expression accepted) that will generate the release tags.
        required: false
        default: main
        type: string
      skip_pre_releases:
        description: Do not create releases for prerelease branches. Overrides draftPrereleases.
        required: false
        default: true
        type: boolean
```

### Outputs

```yml
      # mathieudutour/github-tag-action
      changelog:
        description: The conventional changelog since the previous tag.
        value: ${{ jobs.autorelease.outputs.changelog }}
      new_version:
        description: The auto-bumped version.
        value: ${{ jobs.autorelease.outputs.new_version }}
      new_tag:
        description: The auto-bumped version tag.
        value: ${{ jobs.autorelease.outputs.new_tag }}
      release_type:
        description: The auto-bumped versions release type.
        value: ${{ jobs.autorelease.outputs.release_type }}

      # ncipollo/release-action
      artifact_upload_url:
        description: The URL for uploading assets to the release.
        value: ${{ jobs.autorelease.outputs.artifact_upload_url }}
      release_id:
        description: The identifier of the created release.
        value: ${{ jobs.autorelease.outputs.release_id }}
      release_url:
        description: The HTML URL of the release.
        value: ${{ jobs.autorelease.outputs.release_url }}
```
