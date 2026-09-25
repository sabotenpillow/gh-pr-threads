# gh-pr-threads

A [GitHub CLI](https://cli.github.com) extension that shows the review threads on a
pull request in a terminal UI and lets you reply to them without leaving the terminal.

```
gh pr-threads            # the PR for the current branch
gh pr-threads 123        # a specific PR
gh pr-threads 123 --repo cli/cli
```

## Keys

| Key | Action |
| --- | --- |
| `j` / `k` | move between threads |
| `J` / `K` | scroll the conversation |
| `r` | reply to the selected thread (`ctrl+s` sends, `esc` cancels) |
| `t` | resolve / unresolve the selected thread |
| `a` | show or hide resolved threads |
| `o` | open the thread on github.com |
| `R` | reload |
| `q` | quit |

## Install

```sh
gh extension install OWNER/gh-pr-threads
```

## Develop

```sh
go build -o gh-pr-threads .   # build
gh extension install .        # install this checkout as `gh pr-threads`
gh pr-threads                 # run it
```

`gh extension install .` symlinks the directory, so after any change just rebuild —
no reinstall needed. `gh extension remove pr-threads` when you're done.

## How it works

* Auth comes free from `gh`: `api.DefaultGraphQLClient()` in [`github.go`](github.go)
  reuses whatever `gh auth login` stored, including GHES hosts.
* Threads are read with the GraphQL `pullRequest.reviewThreads` connection. The REST
  endpoint (`/pulls/{n}/comments`) returns a *flat* comment list that you would have to
  re-group by `in_reply_to_id` yourself, and it can't tell you `isResolved`.
* Replies use the `addPullRequestReviewThreadReply` mutation; resolving uses
  `resolveReviewThread` / `unresolveReviewThread`.
* The TUI is [Bubble Tea](https://github.com/charmbracelet/bubbletea) — the same stack
  `gh-dash` uses. All network calls are `tea.Cmd`s so the UI never blocks.

## Release

Tag it; the workflow in `.github/workflows/release.yml` cross-compiles and uploads the
binaries that `gh extension install` looks for.

```sh
git tag v0.1.0 && git push origin v0.1.0
```
