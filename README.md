# byteshiftlabs

This repository is the shared defaults for every byteshiftlabs repository.

Because it's named `.github` on the `byteshiftlabs` account, GitHub
automatically uses the files below as the fallback for any repo that
doesn't define its own:

- `CONTRIBUTING.md`
- `SECURITY.md`
- `.github/PULL_REQUEST_TEMPLATE.md`
- `.github/ISSUE_TEMPLATE/`

This `README.md` also renders as the profile README on the account page —
that's a side effect of the special repo name, not the primary purpose of
this file.

## What lives here

| Path | Purpose | How it applies |
|---|---|---|
| `CONTRIBUTING.md`, `SECURITY.md` | Fallback community health files | Automatic — inherited by any repo without its own |
| `.github/PULL_REQUEST_TEMPLATE.md`, `.github/ISSUE_TEMPLATE/` | PR/issue forms | Automatic — inherited the same way |
| `.github/workflows/reusable-*.yml` | Reusable CI pipelines | Opt-in — a repo's `.github/workflows/ci.yml` calls one with `uses:` |
| `docs/CLAUDE-CONVENTIONS.md` | Canonical rules for commit/PR/issue/doc writing, for AI agents and humans alike | Manual — copy into each repo's own `CLAUDE.md` |
| `docs/README-template.md`, `docs/ROADMAP-template.md` | Section skeletons | Manual — used as a starting structure, not synced automatically |

Repo-specific files (an actual `CONTRIBUTING.md` with real build commands,
for instance) always take precedence over what's here.

## Using a reusable workflow

A caller repo's `.github/workflows/ci.yml` becomes a thin wrapper:

```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  ci:
    uses: byteshiftlabs/.github/.github/workflows/reusable-cpp-ci.yml@main
    with:
      os: ubuntu-22.04
      apt_packages: "libsdl2-dev"
      build_docs: true
```

Pin to a tag (`@v1`) instead of `@main` once this repo has releases, so a
change here can't silently change every downstream repo's CI in the same
push.

Two reusable workflows exist right now, modeled on the two stacks actually
in use:

- `reusable-cpp-ci.yml` — CMake + cppcheck (built from source, cached per
  runner image) + CTest + optional Sphinx docs
- `reusable-python-ci.yml` — pytest matrix + optional pinned-lockfile job +
  ruff + wheel build/smoke-test

Add a new reusable workflow here, rather than a one-off in a single repo,
the next time a different stack (Rust, embedded/CMake-without-cppcheck,
etc.) needs CI.

## Status

Not yet consumed by any repo. `gbglow` and `premise` currently have their
own inline `ci.yml`; migrating them to call these reusable workflows, and
copying `docs/CLAUDE-CONVENTIONS.md` into their local `CLAUDE.md`, is
follow-up work.
