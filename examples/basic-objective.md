# Example: Basic objective

A simple refactor task that demonstrates the standard loop.

## Invocation

```
/goalpower Add type annotations to src/utils/format.ts and ensure the test suite passes
```

## Expected loop

**Round 1 (Implementer):**
- Reads `format.ts`, `format.test.ts`
- Adds annotations: `formatDate(date: Date): string`, `formatBytes(bytes: number): string`
- Runs tests — passes
- Writes `plan.md` with acceptance criteria
- Writes `unit_exercise.log` with raw test output

**Round 1 (Skeptic):**
- Re-reads `format.ts` — annotations present
- Re-reads `format.test.ts` — tests still reference same exports
- Re-runs tests independently — passes
- `verdict-1-0.json`:
  ```json
  {
    "refuted": false,
    "findings": [],
    "evidence": "format.ts:14 has `formatDate(date: Date): string`; tests green",
    "confidence": "high"
  }
  ```

**Aggregation:** ACCEPTED. **Total: 1 round, ~2 minutes.**
