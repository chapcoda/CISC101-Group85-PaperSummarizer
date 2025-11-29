# Module 03 — Guardrails

---

## Change Log
- 2025-11-29: Strengthened `evidence_mode = "strict"` behavior and standardized section warning messages to satisfy Requirement B (Strict Evidence Mode + Section Warnings).

---

## Purpose
Mitigate hallucinations, verify section summaries against the source text, and enforce strict evidence rules before final rendering.

## Inputs
- **Section summaries**: Rows from Module 02  
  `{ section_name, status, summary_text, notes }`
- **Source chunks**: Per-section text and chunks from Module 01.
- **Config**:
  - `evidence_mode` ∈ {"normal", "strict"}

## Policies

### Evidence verification
- Every **sentence and bullet** in a section summary should be traceable to the corresponding section text.
- Unsupported or speculative claims must be removed or replaced with a standardized insufficiency message.

### Strict Evidence Mode
When `evidence_mode = "strict"`:

- Only include claims that:
  - Are explicitly present in the source text, or  
  - Are clearly and directly implied without requiring outside knowledge.
- If the model cannot confidently trace a summary statement to the text, it must replace that statement with:

  > “The source text does not provide enough information in strict evidence mode.”

- No guessing, extrapolation, or rewriting of missing details.

### Standardized section warnings
The following warnings must be used consistently:

- **Empty section** (no usable text):
  > “Section skipped: no usable text provided.”

- **Very short section** (< 50 words, but not empty):
  > “Section very short: summary may be incomplete.”

- **Insufficient information (normal mode)**:
  > “The source text does not provide enough information.”

- **Insufficient information (strict evidence mode)**:
  > “The source text does not provide enough information in strict evidence mode.”

These messages should appear in the `notes` field for each affected section so that Module 04 can surface them in the final output.

## Steps

1. **Align summaries with source text**
   - For each section:
     - Retrieve the original section text or chunk list from Module 01.
     - For each sentence or bullet in `summary_text`:
       - Check that its content is supported by the section text.
       - If a statement is unsupported:
         - In `normal` mode: remove or weaken it and, if needed, use the normal-mode insufficiency warning.
         - In `strict` mode: replace it with the strict-mode insufficiency message.

2. **Enforce section-based warnings**
   - If section status is `missing`:
     - Ensure no fabricated summary is present.
     - Notes should indicate that the section is missing (e.g., “Section missing: no text provided in source.”).
   - If section status is `empty`:
     - Ensure `summary_text` is empty.
     - Notes must contain:  
       > “Section skipped: no usable text provided.”
   - If section status indicates `short` (< 50 words):
     - Allow a brief summary, but ensure notes contain:  
       > “Section very short: summary may be incomplete.”

3. **Consistency and contradiction checks**
   - Compare summaries across sections to detect:
     - Obvious contradictions (e.g., different reported sample sizes, conflicting main results).
     - Scope violations (e.g., claiming causal conclusions when the text reports only correlation).
   - If a potential contradiction is detected:
     - Prefer the more conservative interpretation.
     - Optionally append a note about ambiguity or limited information.

4. **Finalize verified sections**
   - Update each section row with:
     - Cleaned `summary_text` containing only verified statements.
     - `notes` field including any appropriate warnings:
       - Missing/empty/short section warnings.
       - Insufficiency messages (normal or strict).
   - Pass the refined rows forward to Module 04.

## Outputs
- **Verified section summaries**:  
  `{ section_name, status, verified_summary_text, notes }`
- **Consolidated warnings**:
  - Missing sections.
  - Empty or very short sections.
  - Insufficient information (normal and strict modes).
- A guardrail-enforced dataset ready for Module 04 (Rendering & Refinement) to display to the user without hallucinations.

