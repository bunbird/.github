# Contributing

This is the default guide for every Bunbird repository that does not carry its own `CONTRIBUTING.md`. A repository's own guide adds its checks and boundaries on top of these rules.

- Write code, documentation, commit messages, pull requests and review comments in English (American spelling).
- Keep a pull request to one coherent responsibility. Do not hide unrelated refactors, generated output or formatting churn in it.
- Name the branch `<type>/bun-<number>-<slug>` when there is a Linear issue, `<type>/<slug>` otherwise.
- Pull requests are squash-merged and the title becomes the commit subject, so the title follows Conventional Commits (`type(scope): description`). The `Tools: Conventional Commit` check enforces it.
- Fill the pull request template, including what was deliberately left out and how the change was checked.
- Never commit a real secret, credential, token or production `.env` file. The `Tools: Gitleaks` check scans every pull request.
- The backlog lives in Linear (team BUN), not in GitHub issues.

Security problems go through [SECURITY.md](SECURITY.md), never through a public channel.
