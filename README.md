# .github (account root)

Account-level config for `bradk50`. This repo is the single place the Claude Code
GitHub Action is defined; individual repos only carry a thin caller.

## What lives here

- `.github/workflows/claude.yml` — the shared, reusable workflow. Defines the runner,
  permissions, timeout, and how the credential is passed. Nothing triggers it by itself.
- `workflow-templates/claude.yml` — the per-repo caller, published as a template so it
  appears under **Actions → New workflow** in every repo in this account.
- `workflow-templates/claude.properties.json` — metadata for that template.

## How a repo turns it on

Add `.github/workflows/claude.yml` to the repo containing:

```yaml
on:
  issue_comment:
    types: [created]
  issues:
    types: [opened, assigned]

jobs:
  claude:
    if: contains(github.event.comment.body, '@claude') || contains(github.event.issue.body, '@claude')
    uses: bradk50/.github/.github/workflows/claude.yml@main
    secrets: inherit
```

Then mention `@claude` in an issue or PR. Claude works in a checkout of the repo, replies
in the thread, and opens a pull request rather than pushing to the default branch.

## Requirements, per repo or account

1. The Claude GitHub App installed on the repo (or on all repos at once).
2. One credential secret on the repo: `CLAUDE_CODE_OAUTH_TOKEN` (from `claude setup-token`,
   bills against the Claude subscription) or `ANTHROPIC_API_KEY` (bills the Console API).
   A personal account has no org-level secrets, so each repo holds its own copy.
3. The caller workflow present on the default branch — `issue_comment` events only run
   workflows that exist on the default branch.

## Changing behaviour everywhere

Edit `.github/workflows/claude.yml` here and push to `main`. Every repo calling `@main`
picks the change up on its next run. Pin a caller to a commit SHA instead of `main` if you
want a repo frozen on a known-good version.

## Caveats

- A caller can only pass secrets it has; `secrets: inherit` forwards the repo's own secrets
  into this shared workflow.
- Because callers run this repo's `main` as trusted code, write access to this repo is
  equivalent to write access to every calling repo's CI. Keep it private to trusted owners.
- Bots are rejected as triggering actors unless listed in `allowed_bots`.
