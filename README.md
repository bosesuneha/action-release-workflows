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

2. Next, create a workflow YAML file in your project that will be triggered whenever a push is made to the main branch with changes in CHANGELOG.md file.

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

The `release` workflow can be triggered in two ways: automatically when changes are made to the CHANGELOG.md file and pushed to the main branch, or manually through the GitHub Actions UI. 

```
jobs:
   release:
      uses: bosesuneha/action-release-workflows/.github/workflows/release.yaml@main
      with:
         changelogPath: ./CHANGELOG.md
   
```

The `release` job calls the reusable workflow in `action-release-workflows` and takes as input the path to the CHANELOG.md file.



