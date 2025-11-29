01_intake_setup.md
Purpose
Prepare inputs, normalize sections, detect issues, and establish chunking strategy while preserving section boundaries.

Inputs
Paper text: Full raw or extracted text.

Section list: User-provided list; case-insensitive normalization.

Summary level: “short” or “detailed”.

Evidence mode: “normal” or “strict”.

Audience: “expert” or “lay”.

Steps
Validate inputs:
...
Global config: {summary_level, evidence_mode, audience}.

Outputs
Intake report: Normalized section list and detection results.
Section metadata: Per-section structured records.
Chunk plan: Ordered chunks per section for Module 02.
