# Contributing to GSD 2.0

This is a fork of [glittercowboy/get-shit-done](https://github.com/glittercowboy/get-shit-done) with additional features (rollback/recovery, adaptive context, model routing).

## How to Contribute

### Report Bugs
Open an [issue](https://github.com/itsjwill/GSD-2.0-Get-Shit-Done-Cost-saver-/issues) with:
- Steps to reproduce
- Expected vs actual behavior
- Your runtime (Claude Code, OpenCode, Codex, etc.)

### Submit Changes
1. Fork this repo
2. Create a branch: `git checkout -b feat/your-feature`
3. Make changes
4. Test: `npm test`
5. Submit a PR

### What We Need Help With
- **Wiring model routing into workflows** — `get-shit-done/core/model-router.md` defines routing rules but workflows don't use `<model_preference>` tags yet
- **Parallel execution engine** — independent tasks should run concurrently
- **Quality gate integration** — auto-run lint/test/build after task execution
- **Testing** — more test coverage for edge cases

### Sync Strategy
This fork stays synced with upstream via periodic merges. 2.0-specific files live in:
- `get-shit-done/core/model-router.md`
- `get-shit-done/core/context-loader.md`
- `commands/gsd/rollback.md`
- `commands/gsd/recover.md`
- `commands/gsd/resume-task.md`
- `get-shit-done/references/schemas.md`

## Code Style
- Commands: Markdown with XML structure
- Workflows: `<purpose>`, `<process>`, `<step>` tags
- References: `<purpose>` header, clear sections
