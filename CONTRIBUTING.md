# Contributing

This is the default contributing guide for byteshiftlabs repositories that
don't define their own. If the repo you're looking at has its own
`CONTRIBUTING.md`, follow that one instead — it will have the actual build
commands and tool versions for that project.

## Workflow

1. Fork the repository and create a branch named `label/brief-description`:
   - `feature/short-description`
   - `fix/short-description`
   - `refactor/short-description`
   - `docs/short-description`
2. Make your change. Keep each commit to one logical unit of work.
3. Confirm the project's build/test/lint steps pass locally — check the
   repo's README for how to run them.
4. Open a pull request against `main` with a clear description of what
   changed and why, and what you actually ran to check it.

## Commit messages

Past tense, describing what the commit did:

```
Added MBC2 cartridge support
Fixed timer overflow on DIV reset
Removed deprecated save-state V1 loader
```

No `WIP`, `fix`, `update`, or `misc` commit messages.

## License

By contributing, you agree your contribution is licensed under this
repository's license.
