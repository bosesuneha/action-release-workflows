# Release Workflows

Release Workflows automate the process of creating releases based on the CHANGELOG.md file in the project. 

## How it works.

The release workflow is triggered when changes are pushed to the main branch and there are updates in the CHANELOG.md file. When a new release is initiated, the workflow executes the following steps:
1. It parses the CHANGELOG.md file to extract the release notes and identifies the most recent version of the release.
2. Next, it checks if a tag for the latest version already exists. If a tag is found, the subsequent workflows are halted to avoid duplication.
3. If there is no existing tag for the latest version, a new branch is created from the main branch. The new branch is named as `releases/<latest-release-version>`.
4. Once the release branch is set up, the workflow publishes the release to the GitHub repository. The release is tagged with the corresponding version number.

## Usage

1. Start by creating a CHANGELOG.md file to record the changes made in each release of your project.

2. Next, create a workflow YAML file in your project that will be triggered whenever a push is made to the main branch.

3. Your release.yaml workflow file should follow a structure similar to the one outlined below:
   
```
name: release

on:
   push:
      branches:
         - main
      paths:
         - CHANGELOG.md
   workflow_dispatch:
```

The release workflow can be triggered in two ways: automatically when changes are made to the CHANGELOG.md file and pushed to the main branch, or manually through the GitHub Actions UI. 

```
jobs:
   extractChangelog:
      uses: bosesuneha/action-release-workflows/.github/workflows/extract_changelog.yaml@main
      with:
         changelogPath: ./CHANGELOG.md
   checkVersion:
      needs: extractChangelog
      runs-on: ubuntu-latest
      outputs:
         exists: ${{ steps.tag-exists.outputs.exists }}
      steps:
         - name: check if tag exists
           uses: mukunku/tag-exists-action@5dfe2bf779fe5259360bb10b2041676713dcc8a3 # v1.1.0
           id: tag-exists
           with:
              tag: ${{ needs.extractChangelog.outputs.version }}
           env:
              GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
   createBranch:
      needs: [extractChangelog, checkVersion]
      if: needs.checkVersion.outputs.exists == 'false'
      uses: bosesuneha/action-release-workflows/.github/workflows/create_js_branch.yaml@main
      with:
         version: ${{ needs.extractChangelog.outputs.version }}
   createRelease:
      needs: [extractChangelog, checkVersion, createBranch]
      permissions:
         contents: write
      uses: bosesuneha/action-release-workflows/.github/workflows/create_release.yaml@main
      with:
         branch: releases/${{ needs.extractChangelog.outputs.version }}
         version: ${{ needs.extractChangelog.outputs.version }}
         body: ${{ needs.extractChangelog.outputs.body }} 
```

The `extractChangelog` job executes an action that extracts the relevant information from the CHANGELOG.md file. It retrieves the latest release version, the body of the changelog, and determines if the version is a pre-release.

The `checkVersion` job verifies whether a tag with the latest release version already exists. If a tag is found, subsequent jobs are suspended to avoid duplication. This job takes the extracted release version from the CHANGELOG.md as input.

The `createBranch` job creates a branch named `releases/<latest-release-version>` from the main branch. It utilizes the release version as input.

Finally, the `createRelease` job calls the workflow responsible for creating the release. It passes the release branch name, release version, and changelog body as inputs.



