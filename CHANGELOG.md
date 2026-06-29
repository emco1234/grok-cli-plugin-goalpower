# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Planned

- Hooks for auto-injecting goal state on `/compact`
- Cost/token estimation per round
- Skeptic personas (security-focused, perf-focused, correctness-focused)
- Verdict diff visualization between rounds

## [2.0.2] — 2026-06-29

### Added — Initial public release

- **Plugin manifest** (`plugin.json`) declaring components: 1 skill, 2 agents, 1 command
- **Skill** (`skills/goalpower/SKILL.md`) — defines `/goalpower` slash command + loop protocol
- **Agents** (`agents/*.md`) — defines `goalpower-implementer` and `goalpower-skeptic` custom agent types that Grok Build CLI registers as spawnable `subagent_type` values
- **Command** (`commands/goalpower.md`) — slash command template with `$ARGUMENTS` routing for sub-commands (status/pause/resume/clear/edit/config)
- **Loop protocol**: Implementer works objective → K parallel Skeptics audit → aggregation → loop until accepted or premature_stop
- **Anti-ratchet contract**: prior gaps re-audited every round; same gap 5 consecutive rounds → auto-pause
- **Honesty anchor**: `changed_files_manifest.txt` diffed against harness `CHANGED_FILES` by every Skeptic
- **Compact preservation**: state files on disk (`goal.json`, `verdict-*.json`, `skeptic-*.md`, `patch.diff`) survive `/compact`
- **Infinite round loop** by default (`max_rounds = 0`); anti-ratchet is the only soft cap
- **Live config** updates via `/goalpower config key=value`
- Full README with diagrams, examples, configuration reference
- CONTRIBUTING.md, CODE_OF_CONDUCT.md, SECURITY.md, examples, GitHub Actions CI

### Why this exists

Users wanted a standalone Grok Goal plugin that works with any model Grok Build CLI supports — including custom OpenAI-compatible providers (Z.AI GLM, local models, etc.) — without depending on a specific backend. This plugin provides that as a self-contained installable unit.

[Unreleased]: https://github.com/emco1234/grok-cli-plugin-goalpower/compare/v2.0.2...HEAD
[2.0.2]: https://github.com/emco1234/grok-cli-plugin-goalpower/releases/tag/v2.0.2
