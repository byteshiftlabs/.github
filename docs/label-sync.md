# Label sync

GitHub inherits community health files from this repo automatically. It does
**not** inherit labels — those are per-repo with no native inheritance. So
each repo opts in by calling a reusable workflow that applies
[`labels.json`](../labels.json).

## Adding it to a repo

Create `.github/workflows/labels.yml`:

```yaml
name: Sync labels

on:
  workflow_dispatch:
  schedule:
    - cron: '0 4 * * 1'   # Mondays, to pick up manifest changes

permissions:
  issues: write

jobs:
  labels:
    uses: byteshiftlabs/.github/.github/workflows/reusable-label-sync.yml@main
```

It uses the calling repo's own `GITHUB_TOKEN`, so there's no cross-repo PAT
to manage. Works for private repos — it only reads `labels.json` from this
public repo.

## Changing the label set

Edit `labels.json` here. Repos pick it up on their next scheduled run, or
immediately via **Actions → Sync labels → Run workflow**.

Renaming a label in the manifest creates a *new* label rather than renaming
the existing one, because the sync matches on name. To rename, do it in the
GitHub UI on one repo first (which preserves it on existing issues), then
update the manifest to match.

## Pruning

Sync is non-destructive by default: it creates and updates, and leaves
anything not in the manifest alone. To remove labels that aren't in the
manifest:

```yaml
    with:
      prune: true
```

**Deleting a label removes it from every issue and PR using it**, and that
can't be undone. Check what a repo has that the manifest doesn't before
turning this on:

```bash
gh api "repos/byteshiftlabs/<repo>/labels?per_page=100" --jq '.[].name' | sort > /tmp/have
jq -r '.[].name' labels.json | sort > /tmp/want
comm -23 /tmp/have /tmp/want   # would be deleted
```

## Known duplication

The current manifest is the union of what the repos already had, so it
carries several overlapping labels that predate it:

| Overlap | Labels |
|---|---|
| CI | `ci`, `ci/cd`, `ci_cd`, `ci-feature` |
| Bugs | `bug`, `bugfix`, `fix` |
| Docs | `docs`, `documentation`, `docs_refactor` |
| Tests | `testing`, `tests` |
| Tooling | `tool`, `tooling` |
| Refactor | `refactor`, `code_refactor` |
| Features | `feature`, `enhancement` |
| Merge-ready | `approved`, `ready_to_merge`, `confirmed` |

Consolidating means picking one name per concept, relabelling existing
issues, then pruning. Worth doing, but it rewrites history on open items,
so it's a deliberate exercise rather than something to fold into a sync.
