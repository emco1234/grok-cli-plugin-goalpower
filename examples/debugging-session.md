# Example: When Skeptics caught what the Implementer missed

Real examples of fabrication patterns the Skeptic panel catches.

## Pattern 1: Phantom file edits

**Implementer claim:** "Edited src/auth.ts and src/auth.test.ts"

**Skeptic finding:**
```json
{
  "kind": "bug",
  "location": "implementer/changed_files_manifest.txt vs CHANGED_FILES",
  "detail": "Implementer claims src/auth.test.ts edited but CHANGED_FILES only contains src/auth.ts. Manifest is fabricated."
}
```

## Pattern 2: Placeholder research

**Implementer claim:** "Researched the API and documented findings"

**Skeptic finding:**
```json
{
  "kind": "gap",
  "location": "implementer/research_notes.txt:15-22",
  "detail": "Contains [verbatim from tool ...] placeholder text instead of actual quotes."
}
```

## Pattern 3: Curated logs

**Implementer claim:** "All tests pass — see unit_exercise.log"

**Skeptic finding:**
```json
{
  "kind": "gap",
  "location": "implementer/unit_exercise.log:1-18",
  "detail": "Only 18 lines; 'BLOCK1 partial' + powershell syntax errors; missing POINTS/MEAN_VEL observables. Looks curated, not pristine."
}
```

## Pattern 4: Stale plan

**Implementer claim:** "Plan executed successfully"

**Skeptic finding:**
```json
{
  "kind": "gap",
  "location": "implementer/plan.md",
  "detail": "Plan unchanged from round 1; still lists 'TODO: figure out acceptance criteria'."
}
```

## Pattern 5: Skipped prior gap (anti-ratchet trigger)

**Round N Implementer claim:** "All gaps addressed"

**Round N Skeptic finding:**
```json
{
  "kind": "gap",
  "location": "PRIOR_GAPS / item 2",
  "detail": "Gap from round N-2 unfixed: unit_exercise.log still has 'LOG CLEARED' on line 5. Anti-ratchet violation."
}
```
