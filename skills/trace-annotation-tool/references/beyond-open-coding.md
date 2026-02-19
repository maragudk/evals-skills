# Beyond Open Coding: Adding Structure to the Annotation Tool

After completing open coding and building a failure taxonomy (via the `failure-taxonomy`
skill), the user may want to extend their annotation tool to support **structured labeling**
— applying the defined binary failure modes back to each trace.

## When to Extend

Extend the tool only after:

1. The user has completed open coding (freeform annotations on 50-100+ traces)
2. A failure taxonomy exists with 3-7 binary failure modes, each with clear definitions
3. The user wants to re-label traces against the structured taxonomy

Do not add structured tags during the initial open coding phase. Premature structure
biases the annotator toward pre-defined categories and prevents discovery of unexpected
failure modes.

## What to Add

### Failure Mode Checkboxes

Add a checkbox for each failure mode below the freeform notes field. Each checkbox
represents a binary column: present (1) or absent (0).

Display the failure mode name and its one-line definition as a tooltip or subtitle to
reduce cognitive load (the HCI principle of "recognition rather than recall").

A trace can have multiple failure modes checked — they are independent binary columns.

### Keyboard Shortcuts for Tags

Assign number keys (1-7) to toggle each failure mode checkbox, matching the order
displayed in the UI. This lets expert annotators tag quickly without using the mouse.

### Updated Annotations Format

Extend the annotations JSONL to include structured fields:

```json
{
  "trace_id": "abc-123",
  "status": "fail",
  "notes": "SQL query missed the pet-friendly constraint",
  "failure_modes": {
    "missing_sql_constraint": 1,
    "persona_mismatch": 0,
    "hallucinated_data": 0,
    "inappropriate_tone": 0
  },
  "timestamp": "2025-01-15T10:32:00Z"
}
```

### Filter and Review Modes

Once structured labels exist, add filtering to the tool:

- **Show only unreviewed traces**: For the initial labeling pass
- **Show only traces with a specific failure mode**: For focused review of one category
- **Show only deferred traces**: To revisit uncertain cases with fresh eyes
- **Show only traces where freeform notes don't match any failure mode**: To catch
  annotations that need a new category or revised definitions

### Progress by Failure Mode

Extend the progress indicator to show counts per failure mode:

```
Reviewed: 87 / 100
Pass: 52 | Fail: 30 | Defer: 5
---
Missing SQL Constraint: 12 (14%)
Persona Mismatch: 8 (9%)
Hallucinated Data: 6 (7%)
Inappropriate Tone: 4 (5%)
```

## Connecting to Measurement

The structured annotations produced by the extended tool feed directly into the
**Measure** phase of the evaluation lifecycle:

- Each failure mode column becomes the ground-truth labels for building an
  **LLM-as-Judge** evaluator (see the `llm-as-a-judge` skill).
- The human-labeled examples become the training, dev, and test splits for
  calibrating automated judges.
- Error rates from the structured labels help prioritize which failure modes to
  address first in the **Improve** phase.
