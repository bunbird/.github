# bunbird/.github

Organization-wide defaults for the Bunbird GitHub organization: the community health files every repository inherits, and the reusable workflows our repositories call instead of keeping their own copies.

This repository is public on purpose. GitHub applies default community health files only from a public `.github` repository, and a public repository (such as `plugins`) cannot call a reusable workflow kept in a private one. Nothing here is secret: it is the same CI our public `plugins` repository already shows.

## Community health files

| File | Applies to |
| --- | --- |
| `SECURITY.md` | Every repository without its own (`platform` and `plugins` keep theirs) |
| `CONTRIBUTING.md` | Every repository without its own (`platform` keeps its own) |
| `pull_request_template.md` | Every repository without its own `.github/pull_request_template.md` (`platform` keeps its own) |

A repository's own file always wins over the default. There are no issue templates: the backlog lives in Linear (team BUN).

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
