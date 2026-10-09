# Whole-lesson submission template

Create `learning/evidence/Mxx-Lyy.md` after completing the lesson. Attach logs or photos only when the lesson needs them. Answers and formal verification occur at the end; interim commits are optional.

```markdown
# Mxx-Lyy — <title>

## Context
- Starting commit:
- Board / revision:
- Wiring revision and changes:
- Tool versions or environment record:
- Actual source paths:

## Implementation summary
What changed and why. Mention deviations from the lesson.

## End-of-lesson checks
| Criterion / test | Procedure and input | Expected | Observed | Evidence |
|---|---|---|---|---|

## Understanding answers
1. ...
2. ...
3. ...

## Problems and limitations
What failed, how I investigated it, and anything still unverified.
```

Commit this with the implementation, push, then give the resulting full SHA to the reviewer. Do not try to embed a commit's own SHA in itself. Review results may be saved afterwards as `Mxx-Lyy-review-01.md`, referencing the reviewed SHA. Use revision 02 for a later review.

For photos, show signal labels and the complete power path. Avoid relying only on wire colours. For measurements, record method, units, duration, number of trials and limitations. For compiler output, preserve the command and exit result. Redact credentials and unrelated personal information before pushing publicly.
