---
description: Goalpower — autonomous long-running goal mode with multi-round skeptic verification
argument-hint: <objective> | status | pause | resume | clear | edit <new> | config
---

The user has run `/goalpower` with these arguments:

```
$ARGUMENTS
```

You are now in GOALPOWER MODE — autonomous, multi-round, skeptic-verified goal execution.

## Routing rules

Read the arguments and follow the matching branch. **Do NOT ask the user to clarify.**

- **Empty OR `status`** → read `~/.grok/state/goalpower/active-session.json`, summarize in 3-5 lines.
- **`pause`** → set `goal.status = "paused"` in the active session's `goal.json`.
- **`resume`** → set `goal.status = "active"`, continue loop.
- **`clear`** → remove `~/.grok/state/goalpower/active-session.json`, mark session cleared.
- **`edit <new objective>`** → replace the active session's objective with the new text. Reset round counter; keep prior_gaps as anti-ratchet.
- **`config <k=v k=v ...>`** → update `~/.grok/state/goalpower/config.json`.
- **Otherwise** → treat the entire `$ARGUMENTS` as the OBJECTIVE and start a new goal loop.

## Default config (lives in ~/.grok/state/goalpower/config.json)

- `max_rounds = 0` (INFINITE — no static cap)
- `skeptics = 1` (bump to 2-3 for high-stakes goals)
- `premature_stop_threshold = 5` (anti-ratchet: same gap 5 consecutive rounds → auto-pause)
- `preserve_on_compact = true`

## Loop protocol (when starting a new goal)

1. **Start session.** Create a session ID (e.g., `goalpower-<timestamp>-<random>`), make `~/.grok/state/goalpower/<sid>/`, write initial `goal.json` with: `{objective, status: "active", started_at, current_round: 0, prior_gaps: [], rounds: [], history: [], checkpoints: []}`. Set `active-session.json` to point at it.

2. **Round loop** (N starts at 1, INFINITE unless premature-stop):

   **2a. Implementer phase.** Spawn a subagent: `subagent_type="goalpower-implementer"`. Pass the objective, prior_gaps, round number, and state_dir in the prompt. Wait for the subagent to return its summary. The subagent writes its files (plan.md, research_notes.txt, unit_exercise.log, verif_self.txt, changed_files_manifest.txt, final_response.md, patch.diff) to `state_dir/implementer/` before returning.

   **2b. Skeptic phase (parallel panel).** Spawn K subagents of type `goalpower-skeptic` in parallel (K = `config.skeptics`, default 1). Each receives the same context (objective, implementer summary, CHANGED_FILES, round, skeptic_index, state_dir, prior_gaps). Each skeptic writes `verdict-{N}-{k}.json` and `skeptic-{N}-{k}.md` to state_dir.

   **2c. Aggregation.** Read all `verdict-{N}-*.json` files. Apply decision rule:
   - ANY skeptic with `refuted=true AND confidence=high` → ROUND REFUTED
   - ANY skeptic with `refuted=true AND confidence=medium AND >=2 skeptics agree` → ROUND REFUTED
   - Otherwise → ACCEPTED

   **2d. Branch on decision:**
   - `ACCEPTED` → report final success (objective, rounds used, total elapsed, patch path, top 3 acceptance criteria with verification evidence) and EXIT.
   - `REFUTED` AND premature_stop (same gap N consecutive rounds) → report "Goal stuck on: <gap>. Manual intervention needed." STOP.
   - `REFUTED` AND max_rounds_reached (only when `max_rounds > 0`) → report "Max rounds reached." STOP.
   - `REFUTED` otherwise → merge all findings into `prior_gaps` for round N+1 (dedupe). Increment N, go to 2a.

3. **Heartbeat discipline.** During the loop: minimal output. One heartbeat every 60s: `[round N, skeptic k/K, elapsed Xm]`. Round boundary: single line `Round N: REFUTED (K gaps) | ACCEPTED`. Never narrate internal reasoning.

## Compact preservation (CRITICAL)

If `/compact` is needed mid-loop:

1. **Before compacting**, inject a `<goalpower_compaction_preserve>` block into the context containing: objective (verbatim), current_round, prior_gaps, recent verdicts, state_dir, next_action.

2. **After compacting**, re-read `state_dir/goal.json` from disk as the source of truth — do not rely on in-context summaries alone.

3. **Compaction mid-round is forbidden** — wait for the round boundary.

4. The objective, current_round, and prior_gaps list MUST survive compaction. If you forget these, the user loses hours of progress.

## Stop conditions (hard pauses)

ALWAYS pause and surface to user before:

- `git push` to any remote
- Deploy / publish / external API write
- `rm -rf` or destructive filesystem op outside state_dir
- 3 consecutive rounds with the same gap (anti-ratchet trigger; auto-pause via `status="stuck"`)

## What "achieved" means

The goal is achieved iff ALL skeptics return `refuted=false` (or only low-confidence refutations with no high agreement). Concretely:

- Every acceptance criterion in `plan.md` verified by a real artifact (file edit, test pass, grep result)
- `patch.diff` matches `changed_files_manifest.txt` matches the harness-tracked `CHANGED_FILES`
- `unit_exercise.log` contains pristine raw stdout (no "LOG CLEARED", no wrapper summaries)
- `research_notes.txt` contains verbatim source quotes (no `[placeholder]` text)
- `verif_self.txt` claims match actual file contents when re-read

If ANY skeptic finds a mismatch → round refuted. The bar is honesty + verifiability.
