<div align="center">

# ⚡ Grok Build CLI Plugin Goalpower

### The Grok Goal plugin — autonomous goal mode for Grok Build CLI with multi-round skeptic verification

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Grok Build CLI](https://img.shields.io/badge/Grok%20Build%20CLI-%E2%89%A50.2.72-blueviolet)](https://docs.x.ai/build)
[![Type: Plugin](https://img.shields.io/badge/Type-Plugin-orange)](#)
[![Version](https://img.shields.io/badge/version-2.0.2-blue)](./CHANGELOG.md)

**Stop accepting "I'm done" on faith. Make every completion claim pass a panel of adversarial skeptics.**

> **The Grok Goal plugin** — when you need a Grok autonomous agent that actually finishes the job, not just claims to.

</div>

---

## 🔍 Looking for a Grok goal plugin?

You found it. **Grok Build CLI Plugin Goalpower** is a plugin for [Grok Build CLI](https://docs.x.ai/build) that adds a **persistent, multi-round Grok goal execution loop** to your AI coding agent. It works with your existing custom model setup (GLM, Claude, GPT, local — any model Grok Build CLI supports).

If you searched for any of these, this is the right repo:

- `grok plugin goal` · `grok goal plugin` · `grok goal mode`
- `grok build cli goal` · `grok /goal` · `grok autonomous agent`
- `grok long-running task` · `grok skeptic verification`
- `grok anti-fabrication` · `grok verifiable AI agent`

This is the canonical **Grok plugin for goal**-mode workflows.

---

## 🎯 What is the Grok Plugin Goalpower?

When you run `/goalpower <objective>`, the Grok plugin orchestrates a loop:

1. **Implementer subagent** works the Grok goal autonomously and writes plan/research/logs/manifest/patch to a state directory.
2. **K Skeptic subagents** (default 1, bump to 2-3 for high-stakes) audit the Implementer's claim. Each writes a structured verdict JSON.
3. **Panel aggregation** — any high-confidence refutation → gaps feed back to the Implementer for Grok goal round N+1.
4. **Anti-ratchet contract** — prior gaps are re-audited every round.
5. **Grok goal loop exits** when all Skeptics accept, or `premature_stop_threshold` (default 5) consecutive rounds produce the same gap.

---

## ✨ Features

| Feature | What it does |
|---|---|
| 🔄 **Infinite round loop** | Runs until goal is provably achieved (no arbitrary round caps) |
| 🕵️ **Multi-Skeptic panel** | Spawn 1-3 parallel Skeptic subagents for high-stakes goals |
| 🛡️ **Anti-ratchet contract** | Prior gaps re-audited every round; can't escape by doing new work |
| 📝 **Honesty anchor** | `changed_files_manifest.txt` diffed against harness `CHANGED_FILES` |
| 🧠 **Compact preservation** | Goal state survives `/compact` via state files on disk |
| 🧱 **Premature-stop detection** | Same gap N rounds in a row → graceful pause for manual intervention |
| 📜 **Verdict persistence** | Every Skeptic verdict saved to disk for later audit |
| 🎛️ **Slash command + subcommands** | Full control from the Grok TUI |
| 🤖 **Custom agent types** | Registers `goalpower-implementer` + `goalpower-skeptic` agent types |

---

## 📦 Installation

### Option A — Install via `grok plugin install` (recommended)

```bash
grok plugin install emco1234/grok-cli-plugin-goalpower --trust
```

### Option B — Manual clone

```bash
git clone https://github.com/emco1234/grok-cli-plugin-goalpower.git \
  ~/.grok/plugins/goalpower
```

Then enable in the TUI via `/plugins` (Space to toggle), or via CLI:

```bash
grok plugin enable goalpower
```

### Verify installation

```bash
grok inspect
# Should show:
#   Plugins: goalpower (user, enabled) — 1 skills, 2 agents
#   Agents:  goalpower-implementer, goalpower-skeptic
#   Skills:  goalpower
```

Restart Grok Build CLI. Run `/goalpower` in the TUI.

---

## 🚀 Quick start

```
/goalpower Refactor src/auth.ts to use the new token format and update all callers
```

You'll see heartbeats like:

```
[round 1, skeptic 0/1, elapsed 2m]
Round 1: REFUTED (3 gaps) — fabrication: patch only .grok/*; .py edits claimed
[round 2, skeptic 0/1, elapsed 5m]
Round 2: REFUTED (1 gap) — placeholder text in research_notes.txt
[round 3, skeptic 0/1, elapsed 11m]
Round 3: ACCEPTED — all acceptance criteria verified
```

### Sub-commands

| Command | Action |
|---|---|
| `/goalpower <objective>` | Start a new goal |
| `/goalpower status` | Print current state |
| `/goalpower pause` | Pause after current round |
| `/goalpower resume` | Resume from pause |
| `/goalpower clear` | Drop session state (code edits untouched) |
| `/goalpower edit <new objective>` | Replace objective, keep prior_gaps |
| `/goalpower config <key=value>` | Update config live |

---

## ⚙️ Configuration

Default config lives at `~/.grok/state/goalpower/config.json`:

```json
{
  "max_rounds": 0,
  "skeptics": 1,
  "auto_continue": true,
  "pause_on_push": true,
  "premature_stop_threshold": 5,
  "preserve_on_compact": true
}
```

| Option | Default | Description |
|---|---|---|
| `max_rounds` | `0` | Hard cap on rounds. `0` = infinite (anti-ratchet is the only cap) |
| `skeptics` | `1` | Parallel Skeptics per round (bump to 2-3 for high-stakes) |
| `premature_stop_threshold` | `5` | Anti-ratchet: same gap 5 rounds → auto-pause |
| `auto_continue` | `true` | Drive the loop forward automatically |
| `pause_on_push` | `true` | Always pause before `git push` / deploy / external writes |
| `preserve_on_compact` | `true` | Re-read state from disk after `/compact` |

### Live config updates

```
/goalpower config skeptics=3 premature_stop_threshold=7
```

---

## 🏗️ How it works

### Agent types

The plugin registers two custom agent types via `.md` files in `agents/`:

- **`goalpower-implementer`** — works the objective, writes plan/research/logs/manifest/patch
- **`goalpower-skeptic`** — adversarial auditor, writes verdict JSON + markdown report

Spawn them via `spawn_subagent(subagent_type=...)` from the orchestrator.

### The loop

```
┌─────────────────────────────────────────────────────────────┐
│  Round N                                                     │
│                                                              │
│  ┌──────────────────┐    spawns    ┌──────────────────┐    │
│  │   Orchestrator   │ ───────────► │   IMPLEMENTER    │    │
│  │   (you + plugin) │              │   subagent       │    │
│  └──────────────────┘              └─────────┬────────┘    │
│                                              │             │
│              writes plan.md, research_notes.txt,           │
│              unit_exercise.log, verif_self.txt,            │
│              changed_files_manifest.txt, patch.diff        │
│                                              │             │
│                                              ▼             │
│                                    ┌──────────────────┐    │
│              spawns K parallel     │   SKEPTIC #0     │    │
│              ──────────────────►   │   SKEPTIC #1     │    │
│                                    │   SKEPTIC #2     │    │
│                                    └─────────┬────────┘    │
│                                              │             │
│              each writes verdict-{N}-{k}.json              │
│              + skeptic-{N}-{k}.md                           │
│                                              │             │
│                                              ▼             │
│                                    ┌──────────────────┐    │
│                                    │   AGGREGATION    │    │
│                                    └─────────┬────────┘    │
│                                              │             │
│                          ┌───────────────────┼───────────┐ │
│                          ▼                   ▼           ▼ │
│                       ACCEPTED          REFUTED       STUCK │
│                          │                   │           │ │
│                       DONE            feed gaps back    ─► pause │
│                                      for round N+1           │
└─────────────────────────────────────────────────────────────┘
```

### Honesty anchor

Every Skeptic verifies one critical invariant before anything else:

> **Does `implementer/changed_files_manifest.txt` match the harness-tracked `CHANGED_FILES`?**

If not — `refuted: true, confidence: high`. Fabricated file claims are the #1 source of false "complete" verdicts in single-agent systems, and Goalpower makes them impossible to slip through.

---

## 📁 Plugin structure

```
grok-goalpower/
├── plugin.json                              # Plugin manifest
├── skills/
│   └── goalpower/
│       └── SKILL.md                         # Slash command + loop protocol
├── agents/
│   ├── goalpower-implementer.md             # Implementer agent definition
│   └── goalpower-skeptic.md                 # Skeptic agent definition
├── commands/
│   └── goalpower.md                         # Slash command template
├── prompts/
│   └── goalpower/                           # (Reserved for future use)
├── LICENSE
└── README.md
```

State lives outside the plugin (at `~/.grok/state/goalpower/<session-id>/`) so it survives plugin updates.

---

## 💡 When to use Grok Goalpower

✅ **Great fits:**
- "Refactor this module and update all callers" (multi-file, easy to claim done prematurely)
- "Make the test suite green" (clear acceptance criteria)
- "Write the migration and verify it on a copy of prod data"
- "Implement the feature from this spec — don't skip anything"

❌ **Not great fits:**
- Quick one-shot edits (overhead not worth it)
- Pure Q&A ("what does this function do?")
- Tasks with no objective acceptance criteria

---

## 🧪 Examples

See [`examples/`](./examples) for:
- Basic objective — simple refactor goal
- High-stakes — multi-skeptic production rollout
- Debugging patterns — what Skeptics catch

---

## 🛠️ Development

```bash
git clone https://github.com/emco1234/grok-cli-plugin-goalpower.git
cd grok-cli-plugin-goalpower
# Symlink into ~/.grok/plugins/ for live testing:
ln -sf "$PWD" ~/.grok/plugins/goalpower
```

Open the Grok TUI, run `/plugins`, press `r` to reload.

See [CONTRIBUTING.md](./CONTRIBUTING.md) for the full dev guide.

---

## 🗺️ Roadmap

- [ ] Hooks for auto-injecting goal state on `/compact`
- [ ] Cost/token estimation per round
- [ ] Skeptic personas (security-focused, perf-focused, correctness-focused)
- [ ] Verdict diff visualization between rounds

See [open issues](https://github.com/emco1234/grok-cli-plugin-goalpower/issues). PRs welcome.

---

## 🤝 Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md) and the [Code of Conduct](./CODE_OF_CONDUCT.md).

---

## 📄 License

MIT — see [`LICENSE`](./LICENSE).

---

## ⭐ Stargazers over time

[![Star History Chart](https://api.star-history.com/svg?repos=emco1234/grok-cli-plugin-goalpower&type=Date)](https://star-history.com/#emco1234/grok-cli-plugin-goalpower&Date)

<div align="center">

**If the Grok Goal plugin saved you from a fabricated "done" — consider [starring ⚡](https://github.com/emco1234/grok-cli-plugin-goalpower/stargazers) the repo.**

</div>
