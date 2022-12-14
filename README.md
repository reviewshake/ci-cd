# ci-cd

Reusable (Composite) Github Actions for Shake CI/CD pipelines

File name should be action.yml or action.yaml

For details see [this info](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action)

## Goal

The purpose of this action is to standardize the Shake CI/CD pipelines avoiding code duplication and enforcing these features:

* Include a branch protection rule to only allow merging of code to main via Pull Requests.
* Force the approval of each Pull Request by at least one user in the CODEOWNERS file.
* Automated update of CHANGELOG file based on the body of the Pull Request.
* Automatically create a Release with the next semver version based on Pull Requests tags.
* Scans reporitory to check if there are credentials stored as plain text.

## Repo Visibility

This repository is *public*. However, it doesn't contain any credentials or sensitive info.

If we set the repository to private, we'll need to checkout this repo on each client repo before using it. Also, the [post-run command will fail](https://stackoverflow.com/questions/69034292/how-do-you-use-a-composite-action-that-exists-in-a-private-repository)

It's possible to set repositories as [Internal](https://dev.to/n3wt0n/finally-custom-github-actions-in-internal-repos-4l91) but this option is only available for Enterprise GitHub accounts.

## Client Repo Configuration

Add the following labels to the client repo (the repo that uses this action):

- release-major
- release-minor
- release-patch

These labels can be used to tag a Pull Request before merging it into main branch so we can choose how to increase the semantic version

## PERSONAL_ACCESS_TOKEN repo

This action requires passing a variable named inputs.github-token.

That token is the value of the organizational secret PERSONAL_ACCESS_TOKEN, provided by each client pipeline using this composite action.

That secret was generated by creating a Personal Access Token for the `shake-robot` GitHub user. These are the scopes required:

- repo (so it's able to clone any repos like ci-cd or k8s-manifests repo)
- write:packages (so it can create a new GitHub release or even push a docker image to the GitHub registry)

Please set Expiration to Never.

NOTE: For details see [this documentation](https://docs.github.com/en/developers/apps/building-oauth-apps/scopes-for-oauth-apps)

Please store this token in 1password, in the `Infrastructure` vault, in the `shake-robot GitHub User Personal Access Token` login

## Usage

On a different repo create a file named (for example) `.github/workflows/ci-cd.yaml`

```
name: deploy-production

on:
  pull_request:
    branches:
      - master
    types:
      - closed

jobs:
  # Reuse platform-ci (https://docs.github.com/en/actions/using-workflows/reusing-workflows)
  ci:
    name: platform-ci
    uses: ./.github/workflows/platform-ci.yml
    secrets: inherit

  build:
    if: github.event.pull_request.merged == true
    runs-on: ubuntu-latest
    needs: ci
    steps:
      # Shake CI/CD Action
      - name: Run CI/CD composite action
        uses: reviewshake/ci-cd@main
        with:
          app-name: "reviewshake-app"
          app-env: "production"
          docker-repository: "shakeventures/reviewshake-app"
          argocd-values-file: "applications/reviewshake-app/values-production.yaml"

          github-token: ${{ secrets.PERSONAL_ACCESS_TOKEN }}
          docker-username: ${{ secrets.DOCKER_HUB_USERNAME }}
          docker-password: ${{ secrets.DOCKER_HUB_PASSWORD }}
          argocd-repo-ssh-key: ${{ secrets.REPO_K8S_MANIFESTS_SSHKEY }}
          slack-webhook-url: '' # No slack webhook created for prod yet

          docker-prebuild-commands: |
            export SECRET_KEY_BASE=${{ secrets.SECRET_KEY_BASE }}
            export GITHUB_PACKAGE_TOKEN=${{ secrets.GH_PACKAGE_TOKEN }}
            echo //npm.pkg.github.com/:_authToken=${{ secrets.GH_PACKAGE_TOKEN }} >> .npmrc

          docker-build-args: |
            --build-arg GITHUB_SHA=$GITHUB_SHA \
            --build-arg GITHUB_REF=$GITHUB_REF \
            --build-arg SECRET_KEY_BASE=${{ secrets.SECRET_KEY_BASE }} \
            --build-arg BUNDLE_GEMS__GRAPHQL__PRO=${{ secrets.BUNDLE_GEMS__GRAPHQL__PRO }} \
            --build-arg BUNDLE_GEMS__CONTRIBSYS__COM=${{ secrets.BUNDLE_GEMS__CONTRIBSYS__COM }} \
```

## Testing

Please use the [ci-cd-test](https://github.com/reviewshake/ci-cd-test) repo to test modifications to this action.

