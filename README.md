# Plainwing Action

Checks, on every pull request, whether the tests really ran the code that changed. It posts one comment with a verdict per claim and the changed lines that never ran, and updates that comment on later pushes. Nothing leaves the runner.

This repository holds only the wrapper. It installs the [`plainwing`](https://www.npmjs.com/package/plainwing) npm package on the runner and runs it; the checking itself is in that package.

## Use it

Pin the Action to a full commit SHA, which nobody can move, and let Dependabot propose updates. Replace `<full commit SHA>` with the commit of the release you reviewed:

    name: Plainwing
    on: pull_request
    permissions:
      contents: read
      pull-requests: write
    jobs:
      plainwing:
        runs-on: ubuntu-latest
        steps:
          - uses: actions/checkout@v4
            with: { fetch-depth: 0 }
          - uses: Knoten-Tech/plainwing-action@<full commit SHA>

`@v0` also works and follows the latest release, but a tag can be moved by anyone who can push here. The Action compares the sha512 of the `plainwing` package it installs with the value pinned in `action.yml` and stops on a mismatch. See `SECURITY.md` for what the Action reads, runs and posts and where to report a vulnerability.

`fetch-depth: 0` is required so the merge base can be computed. The runner needs Docker, Node and Python 3.9 or later, which `ubuntu-latest` has.

The Action runs the pull request's own code (its install and test commands), so use it only under the `pull_request` event, never `pull_request_target`. For a pull request from a fork the token is read-only, so the result goes to the job summary only and the conclusion is neutral; that fork path has not been run against a real fork pull request yet.

## Inputs

All optional. `fail-on` defaults to `never`: the comment and line annotations appear and the job stays green. Set `fail-on: not-tested` to fail the job on a failed, not tested or unverified claim, or `fail-on: ran-not-checked` to fail it on ran, not checked as well. A `fail-on` in `plainwing.yml` at the merge base applies when it is stricter, so a pull request cannot loosen the gate it is judged by. `image`, `kind`, `install`, `test`, `apt` and `timeout` are described in `action.yml`.

## Licence

This wrapper is MIT. The `plainwing` package it installs is under the Functional Source License (FSL-1.1-ALv2); read its `LICENSE` before using it commercially.
