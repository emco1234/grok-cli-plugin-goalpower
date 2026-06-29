---
name: goalpower
description: Autonomous long-running goal execution with multi-round skeptic verification. Use when the user runs /goalpower and provides a high-level objective. Spawns an Implementer subagent to work the objective, then 1-3 Skeptic subagents audit the claim "goal achieved" against the actual files/patch/logs. If any high-confidence skeptic refutes, gaps are fed back and the next round begins. Loops until all skeptics agree or premature-stop is hit.
disable-model-invocation: true
---

# Goalpower — Autonomous Goal Mode (Skeptic-Verified)

You are now in **GOALPOWER MODE**. This is a long-running, autonomous, multi-round goal execution loop with mandatory adversarial verification.

## The Loop (run this exactly)

```
ROUND N (N starts at 1):

  1. IMPLEMENTER phase
     - Spawn a subagent: subagent_type="goalpower-implementer"
     - The Implementer works autonomously and writes to STATE_DIR/implementer/:
        * plan.md, research_notes.txt, unit_exercise.log,
        * verif_self.txt, changed_files_manifest.txt, final_response.md, patch.diff
     - Returns a brief summary + path to STATE_DIR.

  2. SKEPTIC phase (parallel panel)
     - Spawn K skeptics in parallel: subagent_type="goalpower-skeptic"
     - Each skeptic re-audits claims vs real files, writes verdict-{N}-{k}.json + skeptic-{N}-{k}.md
     - Default verdict: refuted=true, confidence=high

  3. PANEL AGGREGATION (you do this)
     - Read all verdict-{N}-*.json files
     - If ANY skeptic has refuted=true AND confidence=high → ROUND REFUTED
     - Otherwise → ACCEPTED

  4. LOOP or EXIT
     - If ACCEPTED → report success, EXIT
     - If REFUTED → feed merged gaps back for round N+1
     - If premature_stop_threshold (5) consecutive rounds same gap → auto-pause
```

## Sub-Commands

| Command | Action |
|---|---|
| `/goalpower <objective>` | Start a new goal |
| `/goalpower status` | Print current state |
| `/goalpower pause` | Pause after current round |
| `/goalpower resume` | Resume from pause |
| `/goalpower clear` | Drop session state |
| `/goalpower edit <new objective>` | Replace objective, keep prior_gaps |
| `/goalpower config <key=value>` | Update config live |

## Configuration

```
max_rounds      = 0           # 0 = INFINITE (only premature_stop caps)
skeptics        = 1           # bump to 2-3 for high-stakes goals
premature_stop_threshold = 5  # anti-ratchet: same gap 5 consecutive rounds → pause
preserve_on_compact = true    # goal state survives /compact
```

State: `~/.grok/state/goalpower/<session-id>/` (atomic writes, mutation-queued).

## Anti-Ratchet Contract (CRITICAL)

Prior gaps from previous rounds MUST be re-audited every round. They don't disappear because new code was written. If the same gap persists for `premature_stop_threshold` consecutive rounds → auto-pause with `status="stuck"`.

## Honesty Anchor (#1 fabrication check)

Every Skeptic verifies: does `implementer/changed_files_manifest.txt` match the harness-tracked `CHANGED_FILES`? Mismatch = fabrication = highest-severity finding.

## Compact Preservation

On `/compact` mid-goal, inject a `<goalpower_compaction_preserve>` block with: objective, current_round, prior_gaps, recent verdicts, state_dir. State files on disk are source of truth.

## Stop conditions (hard pauses)

- `git push` to any remote
- Deploy / publish / external API write
- `rm -rf` or destructive filesystem op outside state_dir
- 3 consecutive rounds with same gap (anti-ratchet trigger)
