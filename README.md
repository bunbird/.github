# bunbird/.github

Organization-wide defaults for the Bunbird GitHub organization: the community health files every repository inherits, and the reusable workflows our repositories call instead of keeping their own copies.

This repository is public on purpose. GitHub applies default community health files only from a public `.github` repository, and a public repository (such as `plugins`) cannot call a reusable workflow kept in a private one. Nothing here is secret: it is the same CI our public `plugins` repository already shows.

## Community health files

| File | Applies to |
| --- | --- |
| `SECURITY.md` | Every repository without its own (`platform` and `plugins` keep theirs) |
| `CONTRIBUTING.md` | Every repository without its own (`platform` keeps its own) |
| `pull_request_template.md` | Every repository without its own `.github/pull_request_template.md` (`platform` keeps its own) |

| `.github/ISSUE_TEMPLATE/` (Bug, Feature, Task) | Every repository without its own issue templates |

A repository's own file always wins over the default.

## Issues and the board

Work is tracked on one board for the whole organization, [Bunbird](https://github.com/orgs/bunbird/projects/1), the way the BUN team works in Linear today. Every repository is linked to it.

| Linear | GitHub |
| --- | --- |
| Status: Backlog, Todo, In Progress, To Review, QA, Done | Board field Status: Backlog, Todo, In Progress, In Review, QA, Done |
| Canceled, Duplicate | Close the issue as "not planned" or "duplicate" |
| Labels Bug, Feature, Improvement | Issue type Bug, Feature (improvements included) or Task |
| Priority | Board field Priority: Urgent, High, Medium, Low |
| Projects (Product Roadmap, Platform MVP, ...) | Board field Initiative |
| Label group Stage | Board field Stage: discovery, planned, building, rolling out, released |
| Sub-issues | Sub-issues, plus "blocked by" dependencies |
| Labels Security, Open Question | Labels `security`, `needs-decision` (`labels.yml`) |
| Component labels | Each repository's own labels (`platform` keeps its `area/` family) |

An issue opened from one of the forms lands on the board by itself (`projects: ["bunbird/1"]` in the form). From the CLI, pass `--project Bunbird` to `gh issue create`. A new repository gets the forms by inheritance; to put its pull requests on the board too, add it to the board's Auto-add workflow.

The labels in `labels.yml` go into every repository. For a new repository the organization's default labels (Settings, Repository, Repository labels) should hold the same set; for an existing one:

```sh
repo=bunbird/<repo>
yq -o=tsv '.labels[] | [.name, .color, .description]' labels.yml |
  while IFS=$'\t' read -r name color description; do
    gh label create "$name" --color "$color" --description "$description" --force -R "$repo"
  done
```

## Reusable workflows

| Workflow | Replaces | Caller grants |
| --- | --- | --- |
| `reusable-actionlint.yml` | `ci-actionlint.yml` | `contents: read` |
| `reusable-zizmor.yml` | `ci-zizmor.yml` | `contents: read` |
| `reusable-conventional-commit.yml` | `conventional-commit.yml` | nothing |
| `reusable-gitleaks.yml` | `find-secrets.yml` | `contents: read` |

Each repository keeps its own workflow file with its own name, triggers and concurrency, and replaces the job body with a call pinned to a full commit SHA of this repository:

```yaml
name: "CI: Actionlint"

on:
  workflow_dispatch:
  pull_request:
    paths:
      - .github/**
  push:
    branches:
      - main
    paths:
      - .github/**

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

permissions: {}

jobs:
  actionlint:
    name: Actionlint
    permissions:
      contents: read
    uses: bunbird/.github/.github/workflows/reusable-actionlint.yml@<full sha of main> # main
```

The check then reports as `<caller job name> / <reusable job name>`, for example `Actionlint / Workflow schema check`. A ruleset that requires one of these checks must name it that way.

The workflows in this repository call the reusables through a local path, so a pull request here runs the version it changes.

## Changing a reusable workflow

1. Change it here in a pull request. The local callers run it on that pull request.
2. After the merge, move the pin in each calling repository to the new `main` SHA (`git ls-remote https://github.com/bunbird/.github main`), one small pull request per repository.

Action versions inside the reusables are pinned by full SHA and kept fresh by Dependabot here. Container images and the gitleaks binary are pinned by digest and checksum and are bumped by hand.
