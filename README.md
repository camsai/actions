# Actions

Composite GitHub actions for CICD workflows. Meant to reduce the repetitive logic in code repositories by providing reusable functionality. 

Actions are organized by domain/purpose/action where domain is the programming language and purpose is the intent of the action. 

By default, actions take the name `action.yml`, hence the use of a directory structure to differentiate them.

The `git/` actions follow [mat3ra/actions](https://github.com/mat3ra/actions), with this
organization's own bot name and email in the commits, tags and releases they make.

## Releasing on the built-in token

`git/version` and `git/release` run on a job's built-in `GITHUB_TOKEN`, so a repository
that calls them needs no personal token. Check out with `fetch-depth: 0` so every tag is
present, grant the job `contents: write`, and pass `github-token: ${{ github.token }}`.

A release created with the built-in token fires no `release` event: GitHub starts no
workflow for events that token causes, so that workflows cannot trigger one another in a
loop. List the workflows such a release should start in `dispatch-workflows`, one per line,
each a workflow file name and then any `gh workflow run` fields. `workflow_dispatch` is
the one event the built-in token may cause, so the job also needs `actions: write`, and
each listed workflow needs a `workflow_dispatch` trigger. They run on the new tag:

```yaml
jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      actions: write
    steps:
      - uses: actions/checkout@v5
        with:
          fetch-depth: 0
      - id: version
        uses: camsai/actions/git/version@main
      - uses: camsai/actions/git/release@main
        with:
          github-token: ${{ github.token }}
          tag-name: ${{ steps.version.outputs.tag-name }}
          dispatch-workflows: |
            container.yml -f publish=true
            release.yml -f tag=${{ steps.version.outputs.tag-name }}
```
