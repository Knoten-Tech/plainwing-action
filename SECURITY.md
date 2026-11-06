# Security

## Report a vulnerability

Email hello@plainwing.com with the subject "Security". Include the Action version or commit, what you did and what happened. There is no separate security mailbox yet, so that address is the only route; please do not open a public issue for a vulnerability.

## What this Action reads, runs and posts

This repository holds one composite Action. It installs the `plainwing` npm package and runs its engine on the runner.

Reads
- The workspace that `actions/checkout` left, and its git history (the Action needs `fetch-depth: 0`).
- The pull request event: title, body, number and the base and head commits.
- `plainwing.yml` at the merge base.

Runs
- `npm pack` and `npm install --ignore-scripts` for the exact `plainwing` version named in `action.yml`. The tarball's sha512 is compared with the value pinned in `action.yml` before anything is installed, and the Action stops on a mismatch.
- `python3 -I` on the engine from that package.
- The pull request's own install and test commands inside a Docker container that mounts a `git archive` copy of the change (no `.git`), receives no environment, has no Docker socket, sets `no-new-privileges`, drops every Linux capability when the project is Node (a Python container runs as root and keeps Docker's default capabilities), and limits memory, CPU and processes. The network stays open for dependency installs.

Posts
- One pull request comment, updated in place, through the GitHub API with `github-token`, plus workflow annotations and the job summary. They carry the claims, verdicts, file paths and line numbers of changed lines no test ran and those changed source lines inside a prompt for a coding agent.

Never
- It sends nothing to Knoten Technologies or any other service, makes no model call and has no telemetry. The only network calls the engine makes itself go to the GitHub API.
- It does not push, merge, approve, label or edit the repository.
- It does not pass the token, or any other environment variable, to the containers that run the tests.

## Pinning

Tags can be moved by anyone who can push to this repository, so pin the Action to a full commit SHA and let Dependabot propose updates:

    - uses: Knoten-Tech/plainwing-action@<full 40 character commit SHA>

`@v0` follows the latest release of this repository; use it only if you accept that.

## Limits

- Run it under `pull_request`, never `pull_request_target`: it runs the pull request's own code.
- Fork pull requests get a read-only token, so the result goes to the job summary only.
- A secret written literally in a changed source line appears in the comment, as the line is quoted.
