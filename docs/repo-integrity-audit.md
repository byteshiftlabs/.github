# Repo Integrity Audit

A checklist-driven audit for repo integrity and quality. Same checklist for both
scenarios, only what you're checking against differs:

- **Repo audit** — run it over an existing repo to find what's already wrong,
  whether that's a periodic check or the pre-release gate before shipping.
- **Repo creation** — run it while drafting a new repo's contents, to make
  sure you can proceed without gaps instead of discovering them later.

Grounded in what was actually run against ThunderOS and premise, plus the
release-gate checklist absorbed from `rubric/core/production-ready-check.md`.

## Table of contents

- [1. Architecture](#1-architecture)
- [2. Documentation integrity](#2-documentation-integrity)
- [3. Code correctness & reachability](#3-code-correctness--reachability)
- [4. Code quality / clean code](#4-code-quality--clean-code)
- [5. Robustness](#5-robustness)
- [6. Testing](#6-testing)
- [7. Reproducibility](#7-reproducibility)
- [8. Security & secrets](#8-security--secrets)
- [9. Dependency & environment accuracy](#9-dependency--environment-accuracy)
- [10. Cross-repo / org-level consistency](#10-cross-repo--org-level-consistency)
- [11. Release preparation](#11-release-preparation)
- [Findings handling](#findings-handling)

## 1. Architecture

### Structure

- **Layered separation** — interface → logic → data, no layer reaching past
  its neighbor.
- **Dependency direction** — dependencies flow inward; no circular
  dependencies between modules.
- **Single responsibility** — each module has one clear job.
- **Module size limits** — a module that's grown past a reasonable size
  (roughly 500 lines) is a signal to split it.
- **Clear interfaces** between components.

### Boundaries

- **Configuration externalized** — no hardcoded paths, URLs, or credentials.
- **Single, clear entry point.**
- **Fail-fast validation** — inputs validated at system boundaries, not
  deep inside call chains.
- **State minimization** — no more mutable/shared state than the design
  needs.

## 2. Documentation integrity

### Accuracy of claims

- **Fabricated or unverifiable claims.** Numbers, timings, or results
  ("tested", "done", a percentage, a benchmark) asserted in docs with nothing
  in the code or a test run to back them.
- **Stale claims.** Docs describing an old shape of the code — a directory
  that moved, a file that was deleted, a tree diagram nobody updated.
- **Changelog/roadmap accuracy.** Entries claiming a feature is "done" or
  "tested" checked against what's actually built and wired in today, not
  what was true when the entry was written.
- **Test-claims-to-reality gap.** A test file that exists and is referenced
  in docs or the changelog as tested, but isn't actually wired into the
  build/CI (`make test` or equivalent) — well-formed source that never runs.
- **AI-disclosure accuracy.** Where docs carry an AI-assistance disclosure,
  check it still accurately describes how the project is actually being
  developed.

### Drift against source

- **Drifted references.** Anything pinning documentation to a specific line
  number in source (`literalinclude :lines:`, a comment citing "see line
  N") silently goes wrong the next time that source file changes. Prefer
  stable markers over line numbers.
- **Claims that don't match enforcement.** A doc says a rule is "mandatory"
  (no magic numbers, no single-letter names) — check whether the actual lint
  config (`.clang-tidy`, CI) enforces it or has that check turned off.
- **Doc examples match code.** Function signatures, code examples, and
  module descriptions in docs (`.rst`/`.md`) compared against the actual
  code — especially after a refactor moved, renamed, or split something.
- **Quick Start actually works.** Follow the README's Quick Start from a
  clean clone; every command must work exactly as documented.

### Content hygiene

- **Duplicated / irrelevant / obsolete content.** Cross-file duplication,
  sections that no longer apply, placeholder text that outlived its purpose.
- **Plain English.** Rewrite for a reader with no prior context on the
  project — no jargon, no convoluted wording, no unexplained acronyms.
- **Dead links.** Check every external/internal link resolves — a link
  checker or a manual `curl -I` pass, not an assumption.

### Legal & provenance

- **License/legal accuracy.** The LICENSE file matches what README/
  CONTRIBUTING claim the project is licensed under; any vendored or
  third-party code carries the license it's actually distributed under.
- **Badge/metric honesty.** Don't add a coverage/quality/build badge until
  the number behind it is genuinely measured — a fabricated or misleading
  badge is worse than no badge.
- **Copyright and attribution consistency.** Copyright years are current;
  author/org names match across LICENSE, README, docs config, and source
  file headers.

### Process

- **Commit/PR message convention drift.** If CONTRIBUTING mandates a message
  style (e.g. past-participle form), spot-check recent commits against it
  rather than assuming the convention is followed.

## 3. Code correctness & reachability

### Reachability

- **Dead code, verified two independent ways**, not asserted from a single
  grep:
  1. Build-system trace — does the Makefile/build script's source list
     actually compile this file in?
  2. Call-graph trace — is the resulting symbol ever called, or the
     resulting linker section ever reached, from a real entry point?
  A file can pass #1 and still be dead at #2 (compiled in, never called), or
  the reverse never happens but is worth checking either way. Cross-verify:
  reaching the same dead-code conclusion via two independent passes raises
  confidence over a single grep.
- **Reachability ground truth first.** Before judging any individual file,
  establish the actual call order from the real entry point (e.g. trace
  `main()` top to bottom) and use that as the reference every other
  reachability claim gets checked against.

### Correctness

- **Confirmed vs. suspected**, tied to an actual verification command.
  "CONFIRMED" means a specific `grep`/`nm`/`objdump`/`readelf`/build command
  was run and its output read — cite it. "SUSPECTED" means it looks wrong
  but wasn't fully traced — say so rather than asserting it as fact.
- **Unchecked trust boundaries.** Anything reading a value from disk, from a
  user-mode caller, or from hardware/a device, used before validating it
  against its actual bounds.

## 4. Code quality / clean code

### Naming & constants

- **Magic numbers** — unexplained numeric or string constants with no named
  constant or comment.
- **Naming** — ambiguous or single-letter identifiers outside tight,
  conventional scopes (loop counters, etc).

### Structure

- **DRY / KISS violations** — logic duplicated across files that should be
  one shared function; complexity that isn't earning its keep.
- **Complexity and size limits** — if the project claims a function-size or
  complexity ceiling, verify the linter actually enforces it (see
  "Drift against source" under documentation integrity — this is the code
  side of the same question).
- **Dead stub files** — a file that's only a TODO placeholder with no real
  implementation. Either implement it or remove it, including from every
  build target that references it.

### Comments

- **Comments explain why, not what** — complex logic has a comment
  explaining the reasoning, not a restatement of the code.

## 5. Robustness

### Error handling

- **Domain-specific errors** over generic/unlabeled ones.
- **User-facing messages** are clear and actionable, not raw internal
  errors or stack traces surfaced directly to the user.
- **Logging** configured at appropriate levels (not everything at one
  level).

### Input handling

- **Fail-fast validation** at entry points — inputs checked before they
  propagate deep into the system (ties to Boundaries under architecture).
- **Edge cases handled**: empty/null inputs, boundary values (0, negative,
  max), invalid types and malformed data, network/IO failures and timeouts,
  permission/authorization errors, configuration errors.

### Resource management

- **Resource cleanup on every path** — anything acquired (a file handle, a
  lock, allocated memory) is released on both the success path and every
  error path, not just the happy path.

## 6. Testing

### Coverage

- **Coverage** — unit tests for the public API, integration tests for
  module interactions, edge cases included.

### Reliability

- **Clean-clone verification** — all tests pass on a fresh clone with
  warnings enabled, zero errors and zero warnings.
- **No flaky tests** — a test that only sometimes fails is a bug in the
  test (or in what it's testing), not background noise to ignore.

## 7. Reproducibility

### Pinning

- **Dependencies pinned** to exact versions, with a lockfile committed.
- **Toolchain version documented** — language/compiler version stated
  explicitly.

### Build behavior

- **No default optimization flags** unless the project explicitly needs
  them — keep debug builds predictable.
- **System dependencies documented.**

### Environment

- **`.env.example` provided** for any environment variable the project
  reads.
- **Random seeds set** for ML/stochastic projects, so results reproduce.

## 8. Security & secrets

- **Placeholder-only secrets.** `.env.example` (or equivalent) contains only
  placeholder values, never a real credential.
- **Log redaction.** Anything that logs requests/responses redacts API keys
  and other secrets before writing them out.

## 9. Dependency & environment accuracy

- **Declared vs. enforced versions.** A minimum toolchain/dependency version
  stated in docs (`requirements.txt`, a "Minimum version" header) actually
  enforced by CI or the build, not just asserted in prose — the same
  enforcement-mismatch pattern as "Drift against source" in documentation
  integrity, applied to dependency versions instead.

## 10. Cross-repo / org-level consistency

Applies when the repo belongs to an org with shared templates (a `.github`
repo defining reusable workflows, `labels.json`, or a CONTRIBUTING
template, as this one does for byteshiftlabs).

- **Template drift.** Labels, CI conventions, and CONTRIBUTING content
  actually match what the org's shared `.github` repo defines, rather than
  having drifted out of sync since the repo was created.

## 11. Release preparation

### Repository hygiene

- **Repository cleanup** — `.gitignore` excludes build artifacts, caches,
  and secrets; no compiled binaries or packaged archives committed unless
  the project explicitly requires them; no critical TODOs left unresolved;
  LICENSE present; CONTRIBUTING.md present if the project accepts
  contributions.

### Process

- **Git hygiene** — before opening a PR, the creator is assigned to it and
  the correct existing label(s) are applied. If no suitable label exists,
  suggest creating one and wait for approval before continuing.

### Versioning

- **Versioning** — semantic versioning applied on release: patch for
  backward-compatible fixes/hardening/docs/CI/test-only changes, minor for
  backward-compatible feature additions, major for breaking changes. The
  git tag and the GitHub Release title use the exact same version string.
  Release notes group changes under at least Summary, Changes, and
  Verification.

## Findings handling

- Confirmed findings above a severity threshold: fix directly.
- Suspected findings, or confirmed findings below the threshold: file as a
  follow-up issue rather than fixing blind.
- Ask the user for feedback when in doubt. A deletion candidate whose value
  is unclear, a severity/fix-vs-issue call that could go either way, a claim
  that can't be verified from the repo alone — surface it and ask rather
  than guessing silently.
- Keep the findings file itself local-only during the audit — it's a
  working document, not something to commit.
