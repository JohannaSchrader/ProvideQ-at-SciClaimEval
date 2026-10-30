# ProvideQ at NTCIR-19 SciClaimEval: Prompt Templates

Prompt templates used in the paper *ProvideQ at the NTCIR-19 SciClaimEval Task* (NTCIR-19, 2026).

Placeholders in braces (e.g. `{claim}`) or angle brackets (e.g. `<CLAIM>`) are filled per instance. Every Subtask 1 prompt returns a JSON object with a `reason` of at most 100 words and the predicted label. All experiments use temperature 0.

## Contents

- [Subtask 1: Table Claim Verification](#subtask-1-table-claim-verification)
- [Subtask 1: Figure Claim Verification](#subtask-1-figure-claim-verification)
- [Subtask 2: Evidence Selection](#subtask-2-evidence-selection)

## Subtask 1: Table Claim Verification

### Generic CoT

```text
You are a scientific claim verification assistant.
Use the table caption and optional context to understand the
table before reading the table itself.

Caption:
{caption}
Context:
{context_block}
Claim:
{claim}
Evidence format:
{evidence_format}
Table evidence:
{evidence}

Think through these steps internally:
1. Identify the claim type: comparison, ranking, aggregation,
   trend, or existence.
2. Locate the relevant rows and columns in the table.
3. Extract exact values or textual evidence.
4. If the claim says all, most, both, consistently, overall,
   average, or range, check the full relevant scope.
5. Compare the evidence with the claim.
6. Decide Supported or Refuted.

Rules:
- Use only the provided table evidence, caption, and context.
- Do not use external knowledge.
- Do not rely on only one cell if the claim is broad.
- Do not infer formatting information such as bold, underline,
  color, or symbols unless it is explicitly present in the
  provided evidence.
- If the evidence contradicts the claim, answer Refuted.
- If the evidence is insufficient to support the claim,
  answer Refuted.
- Keep the reason under 100 words.
- Output valid JSON only.

Output schema:
{"reason": "...", "pred_label": "Supported" or "Refuted"}
```

### Overview CoT

```text
You are a scientific claim verification assistant.
First build a high-level understanding of the table, then verify
the claim.

Caption:
{caption}
Context:
{context_block}
Claim:
{claim}
Evidence format:
{evidence_format}
Table evidence:
{evidence}

Think through these steps internally:
1. Table overview: identify what the table is about, what the main
   row groups/entities are, what the main column groups/metrics
   are, and whether higher/lower values, symbols, footnotes, or
   formatting cues are explicitly represented in the evidence.
2. Important evidence map: using the caption, context, and full
   table overview, decide which rows, columns, groups, metrics, or
   notes are important for checking this claim.
3. Claim type and scope: identify whether the claim is about
   comparison, ranking, aggregation, trend, existence, formatting,
   symbols, or interpretation; also identify whether it concerns
   one cell, several rows, most/all cases, or an overall pattern.
4. Evidence extraction: extract the exact relevant values or text
   from all relevant rows/columns, including exceptions if the
   claim is broad.
5. Verification: compare the extracted evidence against the claim.
   For broad claims, judge the full relevant scope rather than a
   single example.
6. Decision: answer Supported only when the table evidence
   supports the claim; otherwise answer Refuted.

Rules:
- Use only the provided table evidence, caption, and context.
- Do not use external knowledge.
- Do not rely on surface word overlap alone.
- Do not rely on only one cell if the claim is broad.
- Do not infer formatting information such as bold, underline,
  color, or symbols unless it is explicitly present in the
  provided evidence.
- If the evidence contradicts the claim, answer Refuted.
- If the evidence is insufficient to support the claim, answer
  Refuted.
- Keep the reason under 100 words.
- Output valid JSON only.

Output schema:
{"reason": "...", "pred_label": "Supported" or "Refuted"}
```

### Targeted CoT

```text
You are a scientific claim verification assistant.
Use the caption and optional context to understand the table
topic, terminology, metrics, and experimental setting. Make the
final decision using only the provided table evidence, caption,
and context.

Caption:
{caption}
Context:
{context_block}
Claim:
{claim}
Evidence format:
{evidence_format}
Table evidence:
{evidence}

Think through these checks internally before answering:
1. Table overview: identify the main row groups/entities, column
   groups/metrics, and whether higher/lower values or visual
   markers are explicitly represented.
2. Evidence mapping: locate the exact row, column, cell, table
   note, caption phrase, or context phrase needed to verify the
   claim.
3. Scope and detail check: if the claim uses all, both, most,
   generally, overall, across, average, range, each, or some
   datasets, verify the full scope. Also check whether any
   entity, metric, dataset, number, comparison direction,
   ranking, or exception differs from the evidence.
4. Decision: answer Supported only when the mapped evidence
   supports the claim; otherwise answer Refuted.

Formatting rules:
- Keep the reason under 100 words and include the key evidence
  values or text.
- Output valid JSON only.

Output schema:
{"reason": "...", "pred_label": "Supported" or "Refuted"}
```

### Two-round table verification, round 1 (evidence sheet)

```text
You extract a compact evidence sheet for scientific table claim
verification. Do not make a final Supported/Refuted decision.
Your task is to ground the claim in the current table.

Caption:
{caption}
Context:
{context}
Claim:
{claim}
Table evidence:
{table_evidence}

Output valid JSON only. Keep it compact.

Rules:
- Split the claim into the smallest table-checkable parts.
- For each part, identify exact row names, column names, metrics,
  groups, baselines, and values needed.
- If a comparison is claimed, extract values for both sides.
- If the claim uses all, most, overall, average, consistently,
  best, highest, lowest, respectively, or across, include the full
  relevant scope or state what scope is checked.
- If a calculation is needed, show the formula and computed
  result.
- If there is a directly relevant contradictory cell, include it
  as possible_counter_evidence.
- Treat caption/context as clues for locating evidence, but trust
  current table values over narrative text.
- Mark background or non-table-checkable phrases separately.

Schema:
{
  "claim_parts": [
    {
      "part": "...",
      "type": "lookup/comparison/ranking/aggregation/range/
               significance/calculation/background",
      "needed_evidence": ["row/column/metric..."],
      "extracted_values": ["..."],
      "calculation": "none or formula/result",
      "local_status": "supports/refutes/unclear/background"
    }
  ],
  "possible_counter_evidence": ["..."],
  "background_or_not_in_table": ["..."],
  "notes": "..."
}
```

### Two-round table verification, round 2 (verifier)

```text
You are a scientific table claim verifier.
Decide whether the claim is Supported or Refuted using the
evidence sheet extracted from the current table.

Caption:
{caption}
Context:
{context}
Claim:
{claim}
Evidence sheet:
{evidence_sheet}

Decision rules:
- Supported means the core table-checkable claim is supported by
  the extracted table evidence.
- Refuted means a core value, comparison, direction, scope,
  significance, ranking, method/baseline mapping, or calculation
  is contradicted.
- Do not refute solely because a secondary background phrase is
  not in the table, if the core table relation is supported.
- Concrete claim details are core unless they are only broad
  explanatory context.
- If the claim says matching/equal, check both sides.
- If the claim says all/only/consistently/overall/average/best,
  check the full relevant scope.
- If the evidence sheet is unclear or insufficient for the core
  claim, answer Refuted.
- Keep the reason under 100 words and mention the key values.

Output valid JSON only:
{"reason": "...", "pred_label": "Supported" or "Refuted"}
```

## Subtask 1: Figure Claim Verification

### Visual-grounding CoT

```text
You are a scientific figure claim verification assistant.
Decide whether the claim is Supported or Refuted using the
figure image, caption, and context.

Caption:
{caption}
Context:
{context}
Claim:
{claim}

Use caption/context to locate and interpret the figure; use
the image to verify the claim.

Think internally:
1. Use the claim and caption to find the relevant panel or
   region; if none is specified, use the whole figure.
2. Ground the claim to the correct legend item, color, marker,
   line, bar, axis, category, or caption-defined symbol.
3. Check whether higher or lower visual values support the
   claim.
4. Verify the exact visual relation stated in the claim, such
   as comparison, trend, value, rank, exception, or scope.
5. Do not decide from caption wording or overall plausibility
   alone.

Output valid JSON only. Keep the reason under 100 words and
mention the decisive visual evidence.
{"reason": "...", "pred_label": "Supported" or "Refuted"}
```

### Legend-first two-round prompting, round 1 (grounding)

```text
You are a legend-first scientific figure grounding assistant.
Create a compact visual grounding map for verifying the claim.
Do not decide Supported or Refuted.

Caption:
{caption}
Context:
{context}
Claim:
{claim}

First identify the claim-critical visual elements:
1. target panel or region, if the claim/caption specifies one
2. legend mapping: color, line style, marker, bar, or symbol
   -> method/category
3. axis/metric direction: what higher/lower values mean
4. the exact visual relation needed by the claim

Rules:
- Do not output a final label.
- Do not infer support from the caption alone.
- If a value, legend item, color, marker, line style, or panel
  is not readable, write "not readable".
- If the legend mapping is uncertain, make that explicit
  instead of guessing.
- Output valid JSON only.

Output schema:
{
  "figure_type": "...",
  "target_panel":
    "whole figure / panel label / not specified / not readable",
  "claim_entities": ["..."],
  "legend_map": [
    {
      "visual_style": "...",
      "label": "...",
      "confidence": "high/medium/low"
    }
  ],
  "axis_or_metric_direction": "...",
  "claim_relevant_visual_relation": "...",
  "key_visual_observations": ["..."],
  "uncertain_items": ["..."]
}
```

### Legend-first two-round prompting, round 2 (verifier)

```text
You are a scientific figure claim verification assistant.
Use the legend-first grounding as a guide. Verify the final
decision using the original figure, caption, context, and
grounding. If the grounding conflicts with the figure, trust
the figure.

Caption:
{caption}
Context:
{context}
Claim:
{claim}
Figure grounding:
{first_round_grounding}

Think through these checks internally:
1. Use the correct target panel or region. If none is specified,
   use the whole figure.
2. Verify the exact legend item, color, marker, line, bar,
   category, or symbol referenced by the claim.
3. Check whether the axis/metric direction makes higher or
   lower values support the claim.
4. Verify the exact relation: comparison, trend, rank, value,
   exception, or scope.
5. If the legend mapping or target series is uncertain, do not
   infer support from the overall trend.

Formatting rules:
- Keep the reason under 100 words.
- Output valid JSON only.

Output schema:
{
  "reason": "...",
  "pred_label": "Supported" or "Refuted"
}
```

### Claim-check two-round prompting, round 1 (grounding)

This first-round prompt is shared by the uncalibrated and the general calibrated claim-check variants.

```text
You are a scientific figure claim-check grounding assistant.
Create compact, neutral visual grounding for verifying the
claim. Do not decide Supported or Refuted.

Caption:
{caption}
Context:
{context}
Claim:
{claim}

Extract only claim-relevant information from the current image.
Do not assume the claim is true. Observations may support or
contradict the claim.

Grounding tasks:
1. Read current image text: panel titles, legend labels, axis
   labels, and visible category labels.
2. Bind each claim entity/expression to the exact current-image
   label, panel, legend item, color, line, marker, bar, or
   category.
3. For each claim-critical part, report what is visible in the
   current image and whether it matches, conflicts with, or is
   uncertain for the claim.
4. For multi-panel figures, bind each relevant panel title to
   its visible pattern before using it for the claim.
5. For "increase", "decrease", "higher", "lower",
   "minimal increase", or "only", check both the direction/sign
   and the scope.

Rules:
- Do not output a final label.
- Do not search only for supporting evidence.
- Trust current image labels, legends, panel titles, axes, and
  category names over caption/context.
- If a value, label, marker, panel title, or small text is not
  readable, write "not readable".
- Keep claim_check_results short: at most 6 items.
- Output valid JSON only.

Output schema:
{
  "figure_type": "...",
  "current_image_text": {
    "panel_titles": ["..."],
    "legend_labels": ["..."],
    "axis_labels": ["..."],
    "category_labels": ["..."]
  },
  "target_region":
    "whole figure / panel label / region / not specified /
     not readable",
  "claim_entities": ["..."],
  "legend_or_panel_bindings": [
    {
      "claim_entity_or_expression": "...",
      "mapped_legend_or_panel": "...",
      "visual_style_or_pattern": "...",
      "confidence": "high/medium/low"
    }
  ],
  "axis_direction": "...",
  "claim_check_results": [
    {
      "claim_part": "...",
      "mapped_legend_or_panel": "...",
      "observed_in_current_image": "...",
      "relation_to_claim": "match/conflict/uncertain",
      "confidence": "high/medium/low"
    }
  ],
  "uncertain_items": ["..."]
}
```

### Claim-check, uncalibrated, round 2 (verifier)

```text
You are a scientific figure claim verification assistant.
Use the neutral claim-check grounding as intermediate evidence,
but verify it against the original figure. Decide whether the
claim is Supported or Refuted.

Caption:
{caption}
Context:
{context}
Claim:
{claim}
Neutral claim-check grounding:
{first_round_grounding}

Decision rules:
1. The grounding may contain match, conflict, and uncertain
   observations. Do not ignore conflict observations.
2. Trust current image labels, legends, markers, panel titles,
   axes, colors, and category names over caption/context.
3. Verify that each claim entity/expression is mapped to the
   correct current-image legend item, panel, category, or
   visual pattern.
4. For multi-panel figures, use the current
   panel-title-to-visual-pattern binding. If the claim is
   supported only after ignoring or reassigning the current
   panel title, label it Refuted.
5. For "increase", "decrease", "higher", "lower",
   "minimal increase", "similar", or "only", check both the
   magnitude and the direction/sign/scope.
6. If any claim-critical part has a high-confidence conflict,
   label the claim Refuted unless the conflict is clearly
   irrelevant.
7. If exact small text or numeric values are not readable,
   mention uncertainty rather than inventing evidence.

Formatting rules:
- Keep the reason under 100 words.
- Output valid JSON only.

Output schema:
{
  "reason": "...",
  "pred_label": "Supported" or "Refuted"
}
```

### Claim-check general calibrated, round 2 (verifier)

This template replaces only the second-round prompt; the first round is the claim-check grounding prompt above.

```text
You are a scientific figure claim verification assistant.
Use the claim-check grounding as intermediate evidence, but
verify it against the original figure. Decide whether the claim
is Supported or Refuted.

Caption:
{caption}
Context:
{context}
Claim:
{claim}
Claim-check grounding:
{first_round_grounding}

Decision calibration:
1. First identify the core visual assertion of the claim: the
   main entity/method/category, target panel, metric, direction,
   trend, comparison, rank, or value needed for verification.
2. When the grounding reports a conflict, decide whether it is:
   A. a direct contradiction of the core visual assertion, or
   B. a wording/interpretation issue, minor local deviation,
      missing secondary detail, or uncertainty.
3. Usually label Refuted only for type A conflicts.
4. Do not overturn a claim only because of type B issues if the
   main visual pattern or comparison is supported.
5. Keep hard current-image evidence strict: legend labels,
   panel titles, category labels, axis direction, rank
   direction, and explicit above/below or higher/lower
   relations define the evidence.
6. If the claim is supported only by ignoring or reassigning a
   current image label, legend item, panel title, category, or
   rank direction, label Refuted.
7. If the core visual relation is supported but a secondary
   phrase is imperfect or uncertain, label Supported and
   mention the limitation.
8. If exact small text or numeric values are not readable,
   mention uncertainty rather than inventing evidence.

Formatting rules:
- Keep the reason under 100 words.
- Output valid JSON only.

Output schema:
{
  "reason": "...",
  "pred_label": "Supported" or "Refuted"
}
```

## Subtask 2: Evidence Selection

### Table contrastive prompt

```text
You are a scientific table evidence selection assistant.
Task: Choose which of two table evidences supports the claim.
The two evidences are paired candidates for the same claim.
Compare their table content and choose the one that better
supports the claim.

Caption:
<CAPTION>
Context:
<CONTEXT>
Claim:
<CLAIM>
Evidence ID 1:
<TABLE EVIDENCE 1>
Evidence ID 2:
<TABLE EVIDENCE 2>

Think internally in this order:
1. Decompose the claim into claim-critical checks: entities,
   methods, rows, columns, metrics, values, comparison direction,
   ranking, scope words such as all, only, most, average, and
   range, and any before/after or across-dataset condition.
2. Build a minimal evidence map for Evidence ID 1 and Evidence
   ID 2 at only those claim-critical table positions.
3. Compare the two evidence maps directly. Focus on relevant
   differences in values, labels, row/column alignment, ranking,
   comparison direction, or scope.
4. Select the evidence whose table content supports the core
   claim-critical checks. Do not choose based on caption or
   context alone because they are shared.
5. If one evidence satisfies the main quantitative or comparative
   claim and the other does not, choose the satisfying evidence.

Decision calibration:
- Do not reject an evidence because of minor explanatory wording
  if the core table claim is supported.
- For "respectively", match items in order.
- For "all", "only", "most", "average", "overall", and "across",
  check the exact relevant scope.
- If both evidences look similar, choose the one with the
  stronger exact match to the claim-critical values or
  comparisons.

Output valid JSON only:
{
  "reason": "Evidence ID X supports the claim because ...;
             Evidence ID Y fails because ...",
  "pred_label": "evidence_id_1" or "evidence_id_2"
}
```

### Figure two-round prompt, round 1 (comparative grounding)

```text
You are a scientific figure comparison assistant.
Task: Compare two figure evidences for the same claim.
Do not choose the final evidence. Only extract claim-relevant
visual grounding and differences. Be neutral: do not search only
for supporting evidence, and do not prefer Evidence ID 1.

Caption: <CAPTION>
Context: <CONTEXT>
Claim: <CLAIM>
Evidence ID 1 is the first image.
Evidence ID 2 is the second image.

Focus on:
- target panel or region
- legend labels and their color, marker, line, or bar mapping
- category and entity labels
- axis meaning and direction
- values, ranks, trends, relative ordering, or comparisons
- scope words such as all, only, most, before/after,
  respectively, and across panels

Output valid JSON only:
{
  "claim_focus": ["claim-critical visual checks"],
  "evidence_1_map": {
    "target_panel_or_region": "...",
    "legend_or_category_mapping": "...",
    "axis_or_direction": "...",
    "claim_related_observation": "...",
    "match_or_conflict": "match/conflict/uncertain"
  },
  "evidence_2_map": {
    "target_panel_or_region": "...",
    "legend_or_category_mapping": "...",
    "axis_or_direction": "...",
    "claim_related_observation": "...",
    "match_or_conflict": "match/conflict/uncertain"
  },
  "relevant_differences": ["..."],
  "uncertain_items": ["..."]
}
```

### Figure two-round prompt, round 2 (selection)

```text
You are a scientific figure evidence selection assistant.
Task: Choose which of two figure evidences supports the claim.
Use the Round 1 visual grounding as intermediate evidence, but
verify it against the current images.

Caption: <CAPTION>
Context: <CONTEXT>
Claim: <CLAIM>
Round 1 visual grounding:
<GROUNDING OUTPUT>
Evidence ID 1 is the first image.
Evidence ID 2 is the second image.

Decision rules:
1. Do not choose based on caption or context alone.
2. Compare the candidates only on claim-critical visual checks.
3. Trust the current image labels, legends, panel titles, axes,
   colors, markers, and bars.
4. Evidence ID 1 is not the default answer.
5. For approximate wording, judge the intended visual trend or
   range rather than requiring pixel-level equality.
6. If Round 1 is uncertain or inconsistent with the images,
   re-check the images directly before choosing.

Output valid JSON only:
{
  "reason": "Evidence ID 1: match/conflict ... Evidence ID 2:
             match/conflict ... Therefore Evidence ID X supports
             the claim better because ...",
  "pred_label": "evidence_id_1" or "evidence_id_2"
}
```
