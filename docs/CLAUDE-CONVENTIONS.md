# byteshiftlabs conventions for AI coding agents

This is the canonical source for how commits, PRs, issues, and docs should
read across every byteshiftlabs repository. It exists so an agent working in
any single repo produces output indistinguishable from one working in any
other.

Every repo should carry a local `CLAUDE.md` that includes this file's rules —
either by copying this document in full or by summarizing it faithfully.
`@`-imports in `CLAUDE.md` only resolve local paths, so this content has to be
copied into each repo rather than referenced remotely; keep copies in sync by
hand when this file changes.

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

- State what changed and why, organized by file or by theme — not a
  chronological narration of the work session.
- Keep it to what's needed to review the change. A one-file bugfix does not
  need a multi-section report; a PR touching several unrelated concerns can.
- **Verification is a claim, not a formality.** Say plainly what was actually
  run — tests, a manual build, a live check — and distinguish that from
  reasoning done by reading the code alone. Never write steps as if they were
  executed when they were only inferred.
- Do not pad with impact analysis, threat scenarios, or downstream reasoning
  beyond what was actually established. If something is speculative, say so
  in one line instead of building it out as if it were confirmed.
- No hedging about unrelated platforms, toolchains, or use cases the PR
  doesn't touch.

## Issues

- Open with a one-line statement of how the issue was found: read the code,
  reproduced it, hit it in production, etc. This is not optional — it's what
  lets whoever picks it up calibrate how much to trust the rest of the report.
- If reproduction steps were not actually run, do not write a "Reproduction"
  or "How to reproduce" section. Use something like "What would confirm or
  kill it" instead, describing what a reader could do to settle the question.
- Do not build out severity/impact/exploit-scenario detail beyond what's
  actually been established. State the mechanism (e.g. "an unchecked index
  into a buffer sized from input"), not a story about what an attacker could
  theoretically achieve with it, unless that story has actually been tested.
- Don't single out one example of a broader class as if it were uniquely
  affected (e.g. naming one unsupported cartridge type when all unsupported
  types are equally unsupported) — describe the class.

## Documentation (README / CONTRIBUTING / ROADMAP / SECURITY)

- State the platform and toolchain versions the project is actually
  developed and tested on, then stop. No "other platforms may work",
  no "untested rather than unsupported", no invitation for issues/PRs from
  people on unlisted platforms. Naming the tested configuration is complete
  information on its own.
- Every version number, tool requirement, and coverage claim in a doc must be
  checked against the actual repo state (build files, CI config, test
  directory) before being written or left standing. Don't inherit a stale
  claim just because it was already there.
- A "not yet covered" or "known gap" statement must be re-verified before
  every edit that touches the surrounding text — these rot fast and silently.
- Don't repeat the same information (e.g. a dependency install list) in more
  than one file if it can live in one place and be linked to. Duplication is
  how these docs drift out of sync with each other and with the code.

## General

- When a claim in this file or in repo docs turns out to be wrong or
  unverifiable, fix it immediately rather than leaving it for later — treat
  documentation debt the same as code debt.
