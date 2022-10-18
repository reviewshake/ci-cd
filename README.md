# ci-cd

Reusable (Composite) Github Actions for Shake CI/CD pipelines

File name should be action.yml or action.yaml

For details see [this info](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action)

## PERSONAL_ACCESS_TOKEN repo

This action requires passing a variable named inputs.github-token.

That token is the value of the organizational secret PERSONAL_ACCESS_TOKEN, provided by each client pipeline using this composite action.

That secret was generated by creating a Personal Access Token for the `shake-robot` GitHub user. These are the scopes required:

- repo (so it's able to clone any repos like ci-cd or k8s-manifests repo)
- write:packages (so it can create a new GitHub release or even push a docker image to the GitHub registry)

NOTE: For details see [this documentation](https://docs.github.com/en/developers/apps/building-oauth-apps/scopes-for-oauth-apps)

Please store this token in 1password, in the `Infrastructure` vault, in the `shake-robot GitHub User Personal Access Token` login
