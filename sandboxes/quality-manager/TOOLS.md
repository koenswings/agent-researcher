# TOOLS.md — Quality Manager

## Environment

- **QM workspace:** `/home/node/workspace/hq/quality-manager`
- **Read access to all repos:** `/home/node/workspace/` (engine, console-ui, website, hq)

## GitHub Access

Use `gh` CLI to list and review open PRs:
```
gh pr list --repo koenswings/engine
gh pr list --repo koenswings/console-ui
gh pr list --repo koenswings/website
gh pr list --repo koenswings/hq
gh pr view <number> --repo koenswings/<repo>
gh pr diff <number> --repo koenswings/<repo>
```

Leave review comments:
```
gh pr comment <number> --repo koenswings/<repo> --body "..."
```

## Notes

_(Add org name when confirmed. Add any repo-specific quirks to watch for.)_
