# Contributing to Grok Goalpower

Thanks for taking the time to contribute! 🎉

## Code of Conduct

By participating you agree to abide by the [Code of Conduct](./CODE_OF_CONDUCT.md). Report unacceptable behavior to **info@contentplanning.ai**.

## Development setup

### Prerequisites

- [Grok Build CLI](https://docs.x.ai/build) >= 0.2.72
- Git

### Get the code

```bash
git clone https://github.com/emco1234/grok-goalpower.git
cd grok-goalpower
```

### Manual testing in Grok Build CLI

1. **Symlink your dev checkout into the plugins directory:**

```bash
# macOS / Linux
ln -sf "$PWD" ~/.grok/plugins/goalpower

# Windows (PowerShell, admin)
New-Item -ItemType SymbolicLink -Path "$env:USERPROFILE\.grok\plugins\goalpower" -Target "$PWD"
```

2. **Reload plugins in the TUI:**

Open Grok, run `/plugins`, press `r` to reload. Or use CLI:

```bash
grok plugin disable goalpower
grok plugin enable goalpower
```

3. **Verify discovery:**

```bash
grok inspect | grep -i goalpower
# Expected: Plugin enabled, 2 agents, 1 skill
```

## Architecture overview

| Component | File | Responsibility |
|---|---|---|
| Skill | `skills/goalpower/SKILL.md` | Slash command discovery + high-level loop protocol |
| Command | `commands/goalpower.md` | `$ARGUMENTS` routing template for sub-commands |
| Implementer | `agents/goalpower-implementer.md` | Agent definition: works objective, writes plan/research/logs/manifest/patch |
| Skeptic | `agents/goalpower-skeptic.md` | Agent definition: adversarial auditor, writes verdict JSON |
| Manifest | `plugin.json` | Declares components + metadata |

## Key invariants

These are load-bearing. Do not break them in a PR without discussion:

1. **Honesty anchor** — every Skeptic MUST diff `changed_files_manifest.txt` against harness `CHANGED_FILES`
2. **Anti-ratchet** — prior gaps MUST be re-audited every round
3. **Infinite loop by default** — `max_rounds = 0`. Anti-ratchet is the only soft cap
4. **Compact preservation** — state files on disk are source of truth; in-context block is a fast-resume hint
5. **No personal paths** in any plugin file — use `~/` and `$HOME` only

## Coding standards

- **Markdown strict** — every `.md` file must have valid YAML frontmatter where applicable
- **No shell-specific paths** in plugin files — portable across macOS/Linux/Windows
- **Agent definitions** must use the canonical fields: `name`, `description`, optional `model`, `capability_mode`

## Commit message conventions

This project uses [Conventional Commits](https://www.conventionalcommits.org/):

- `feat` — new feature
- `fix` — bug fix
- `docs` — documentation only
- `refactor` — code change that neither fixes a bug nor adds a feature
- `chore` — build, deps, config, tooling

Example:

```
feat(skeptic): add per-prior-gap status output in verdict details_md

Each verdict now includes a "Per-Prior-Gap Status" section that
classifies every prior gap as FIXED / PARTIAL / UNFIXED / REGRESSED.
```

## Pull request flow

1. **Open an issue first** for non-trivial changes
2. **Fork & branch** from `main`:
   ```bash
   git checkout -b feat/my-feature
   ```
3. **Commit using conventional commits.** Small, focused commits preferred.
4. **Test manually** in Grok TUI before opening PR
5. **Open a PR** against `main`. Fill in the PR template

## Releasing

Releases are cut from `main` and tagged with semver:

```bash
# 1. Update CHANGELOG.md (move entries from [Unreleased] to new version)
# 2. Bump version in plugin.json
# 3. Commit + tag
git commit -m "chore(release): vX.Y.Z"
git tag -a vX.Y.Z -m "vX.Y.Z"
git push origin main
git push origin vX.Y.Z
# 4. Create GitHub Release via the UI or:
gh release create vX.Y.Z --notes-from-tag
```

## License

By contributing, you agree that your contributions will be licensed under the [MIT License](./LICENSE).

## Questions?

Open a [discussion](https://github.com/emco1234/grok-goalpower/discussions) or email **info@contentplanning.ai**.
