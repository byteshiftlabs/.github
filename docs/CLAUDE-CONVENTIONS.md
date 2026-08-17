# byteshiftlabs conventions for AI coding agents

Canonical rules for commits, PRs, issues, and docs across all byteshiftlabs
repositories.

Copy this into each repo's own `CLAUDE.md` — `@`-imports only resolve local
paths, so it can't be referenced remotely.

## Commits

- Past tense, describing what the commit did: `Added MBC2 cartridge support`,
  `Fixed timer overflow on DIV reset`. Not `Add`, not `Fix:`, not `WIP`,
  `fix`, `update`, or `misc`.
- One logical change per commit. Prefer several small commits over one that
  mixes unrelated fixes.
- Branch naming: `label/brief-description` — `feature/mbc2-support`,
  `fix/timer-overflow`, `refactor/ppu-cleanup`, `docs/memory-map`.
- **No AI attribution.** No `Co-Authored-By: Claude ...` trailer, no
  "Generated with Claude Code" line, anywhere in a commit or a PR body.
  Authorship reads as the repo owner's.

## Pull requests

Use these three sections, in this order — don't invent a different structure
per PR:

```markdown
## Summary

What changed and why. The problem, stated plainly.

## Changes

How it was done. Organized by theme or file.

## Testing

What was actually run.
```

- Organize by theme or file, never as a narration of the work session.
- Keep length proportional to the change. A one-file bugfix doesn't need a
  multi-section report.
- **Verification is a claim.** Say what was actually run, and mark anything
  established by reading the code alone. Never write a check as though it
  were executed when it was inferred.
- Don't pad with impact analysis or downstream reasoning beyond what was
  established. Speculative? One line saying so.
- No hedging about unrelated platforms, toolchains, or use cases the PR
  doesn't touch.

## Issues

Use these three sections, in this order:

```markdown
## Summary

What's wrong, and exactly where. Link the specific line with a permalink
pinned to a commit SHA, not a branch — branch links rot as lines move.

## Context

The supporting facts, including what was actually checked and how.

## Options

Possible resolutions, laid out rather than prescribed.
```

- Say in Context how it was found — read the code, reproduced, hit in
  production. It tells the reader how much to trust the rest.
- Never write reproduction steps that weren't run. Say what would settle it
  instead.
- Offer options rather than dictating one fix. Say if you'll implement it.
- State the mechanism ("an unchecked index into a buffer sized from input"),
  not an untested story about what it could lead to.
- Describe the class, not one arbitrary member of it.

## Documentation (README / CONTRIBUTING / ROADMAP / SECURITY)

- Name the platform and toolchain actually developed and tested on, then
  stop. No "other platforms may work", no invitations to users on unlisted
  ones.
- Check every version number, tool requirement, and coverage claim against
  the repo before writing it or leaving it standing. Stale claims already in
  the file are not exempt.
- Re-verify "not yet covered" and "known gap" statements on every edit that
  touches them. They rot silently.
- Keep shared information (dependency lists, versions) in one file and link
  to it. Duplication is how these drift.

## Third-party material

- Don't reference copyrighted or trademarked material where a neutral
  alternative works. Examples, placeholders, sample data and test fixtures
  get generic names — `game.gb`, not `tetris.gb`.
- Naming a real product is fine when it states a technical fact the reader
  needs ("the header must match the Nintendo logo", "MBC5 was used by
  Pokémon Crystal"). That's descriptive, and removing it would make the
  documentation wrong. It's arbitrary filler that causes trouble.
- Same test for media: no screenshots or recordings of commercial games in
  a README. Use homebrew with a known license, or the project's own UI.

## General

- When a claim in this file or in repo docs turns out to be wrong or
  unverifiable, fix it immediately rather than leaving it for later — treat
  documentation debt the same as code debt.
