# Module 02 — Section Loop

---

## Change Log
- 2025-11-29: Clarified `summary_level` behavior and standardized warning messages to satisfy Requirement A (short vs detailed summaries per section).

---

## Purpose
Iterate through each section in order and produce section-level summaries and warnings according to the PS2 specification and the system prompt.

## Inputs
- **Section metadata**: From Module 01 (section name, text, status flags, word count, chunks).
- **Config**:  
  - `summary_level` ∈ {"short", "detailed"}  
  - `evidence_mode` ∈ {"normal", "strict"}  

## Rules

### Section status handling
- **Missing section**  
  - Do **not** fabricate content.  
  - Mark status as `Missing`.  
  - Summary field should remain empty or explicitly say no summary is possible.  
  - Notes must include a warning.

- **Empty section** (no usable text)  
  - Do not summarize.  
  - Mark status as `Empty (skipped)`.  
  - Notes must include the standardized warning:  
    > “Section skipped: no usable text provided.”

- **Short section** (< 50 words)  
  - Summarize if any usable text exists.  
  - Mark status as `Present (short)`.  
  - Notes must include the standardized warning:  
    > “Section very short: summary may be incomplete.”

- **Present / normal section**  
  - Summarize strictly from the section text.

### Summary level behavior
- If `summary_level = "short"`:
  - Generate **only** a compact summary:
    - **1–2 sentences** capturing the core contribution or purpose of the section.
  - No bullet list is added.

- If `summary_level = "detailed"`:
  - Generate a **short paragraph** (3–5 sentences) that synthesizes key ideas.
  - Then add a **bullet list of 3–5 key points**, such as:
    - Important methods or procedures.
    - Main results or findings.
    - Limitations or caveats explicitly stated.
    - Important definitions or assumptions.

### Evidence and safety
- All summaries must remain within the content of the section text (no hallucinations).
- If `evidence_mode = "strict"`, any obviously unsupported or speculative statement should be avoided; Module 03 will further enforce this.

## Steps

For each section in the normalized order from Module 01:

1. **Read section metadata**:  
   - Name, text, status (`missing`, `empty`, `short`, `present`), word count, chunks.

2. **Handle by status**:
   - If `status = missing`:
     - Add a row with:
       - `Status = "Missing"`
       - `Summary = ""` (or a neutral note)
       - `Notes` including a missing-section warning.
   - If `status = empty`:
     - Add a row with:
       - `Status = "Empty (skipped)"`
       - `Summary = ""`
       - `Notes` including:  
         > “Section skipped: no usable text provided.”
   - If `status = short`:
     - Generate a summary from available text.
     - Add a row with:
       - `Status = "Present (short)"`
       - `Summary` generated per `summary_level` rules.
       - `Notes` including:  
         > “Section very short: summary may be incomplete.”
   - If `status = present`:
     - Generate a summary from the section text per `summary_level` rules.
     - Add status `Present`.
     - Notes may remain empty unless other flags apply.

3. **Generate summary content**:
   - If `summary_level = "short"`:
     - Produce a **1–2 sentence** summary capturing the section’s main idea.
   - If `summary_level = "detailed"`:
     - Produce:
       - A **short paragraph** summarizing the section.
       - A **bullet list of 3–5 key points** highlighting methods, results, or important details.

4. **Record notes and flags**:
   - Ensure that any status such as missing, empty, short, or strict-mode issues is recorded in the `Notes` column so Module 03 and Module 04 can expose them to the user.

## Outputs
- **Section-by-section summary rows**, each of the form:  
  `{ section_name, status, summary_text, notes }`
- Flags in notes that surface:
  - Missing sections.
  - Empty / skipped sections.
  - Short sections with incomplete summaries.
- Structured data for Module 03 (Guardrails) to further verify and enforce evidence constraints.
