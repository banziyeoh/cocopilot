---
name: oe-compliance-engine
description: "Quantitative Operational Excellence evaluation engine with weighted scoring, evidence-backed assessments, risk exposure analysis, and automated remediation planning. Trigger: When evaluating repository compliance against OE checklists, generating compliance scores, or creating remediation plans."
---

# OE Compliance Engine

## Overview

A purpose-built evaluation engine for assessing repositories against Operational Excellence checklists. Unlike simple pass/fail approaches, this engine provides quantitative scoring (0-10 per item, 0-100 aggregate), evidence-backed assessments with code references, risk-weighted prioritization, and automated remediation plans with effort estimates.

## Skill Dependencies

This skill works in conjunction with:

- `conventions` — General coding standards for evaluation benchmarks
- `architecture-patterns` — SOLID, Clean Architecture, DDD evaluation criteria
- `critical-partner` — Rigorous review of evaluation outputs
- `process-documentation` — Documentation of evaluation findings
- `english-writing` — All reports generated in English

## Objective

Enable AI agents to perform rigorous, evidence-backed OE compliance evaluations that produce quantitative scores, risk matrices, and actionable remediation plans — not just compliance labels.

---

## When to Use

Use this skill when:

- Evaluating repository compliance against any OE checklist
- Generating quantitative compliance scores for leadership reporting
- Creating risk-weighted remediation plans
- Comparing compliance across repositories or across time
- Building executive summaries from technical audit findings

Do not use when:

- Performing simple code review (use `code-review-expert` instead)
- Writing tests (use `tdd-workflow` instead)
- Designing architecture from scratch (use `architecture-patterns` instead)

---

## Critical Patterns

### ✅ REQUIRED [CRITICAL]: Quantitative Scoring Framework

Every checklist item MUST receive a numeric score, not just a label.

```markdown
## Scoring Scale (0-10 per item)

| Score | Label               | Definition                                                      |
| ----- | ------------------- | --------------------------------------------------------------- |
| 0     | Absent              | No evidence of implementation. Zero effort detected.            |
| 1-2   | Critical Gap        | Minimal traces exist but fundamentally broken or incomplete.    |
| 3-4   | Non-Compliant       | Partial attempt exists but fails to meet minimum requirements.  |
| 5-6   | Partially Compliant | Core requirement met with significant gaps in depth or breadth. |
| 7-8   | Largely Compliant   | Solid implementation with minor gaps or improvement areas.      |
| 9     | Fully Compliant     | Meets all requirements with evidence. Minor polish possible.    |
| 10    | Exemplary           | Exceeds requirements. Could serve as reference implementation.  |

## Impact Weights

| Impact Level | Weight Multiplier |
| ------------ | ----------------- |
| Critical     | 3.0               |
| High         | 2.0               |
| Medium       | 1.5               |
| Low          | 1.0               |
| Optional     | 0.5               |

## Aggregate Score Formula

Compliance Score = (Σ item_score × item_weight) / (Σ max_score × item_weight) × 100

## Grade Thresholds

| Grade | Score Range | Status                                    |
| ----- | ----------- | ----------------------------------------- |
| A+    | 95-100      | Exemplary — reference implementation      |
| A     | 90-94       | Excellent — production-hardened           |
| B+    | 85-89       | Strong — minor improvements needed        |
| B     | 75-84       | Good — targeted improvements recommended  |
| C     | 60-74       | Fair — significant gaps require attention |
| D     | 40-59       | Poor — systemic improvements needed       |
| F     | 0-39        | Failing — fundamental rework required     |
```

### ✅ REQUIRED [CRITICAL]: Evidence Collection Protocol

Every assessment MUST provide structured evidence. Never score without proof.

```markdown
## Evidence Record Format

### [Checklist Item Name]

- **Score**: 7/10
- **Impact**: High (×2.0)
- **Weighted Score**: 14/20
- **Evidence Type**: Code Reference | Configuration | Documentation | Absence
- **Evidence**:
  - ✅ Found: `src/middleware/errorHandler.ts` — Global error handler with structured responses
  - ✅ Found: `src/utils/ApiError.ts` — Custom error class with status codes
  - ⚠️ Gap: No error correlation IDs for distributed tracing
  - ❌ Missing: No standardized error response schema (e.g., RFC 7807)
- **Files Analyzed**: [list of files examined]
- **Search Queries Used**: [list of grep/search patterns executed]
- **Confidence**: High | Medium | Low
- **Confidence Justification**: [why this confidence level]
```

### ✅ REQUIRED [CRITICAL]: Risk Exposure Matrix

Generate a risk matrix that quantifies exposure, not just identifies problems.

```markdown
## Risk Exposure Calculation

Risk Exposure = (10 - item_score) × impact_weight × probability_factor

### Probability Factors

| Factor          | Value | When to Apply                              |
| --------------- | ----- | ------------------------------------------ |
| Active Exploit  | 3.0   | Known vulnerability in production          |
| High Likelihood | 2.0   | Common attack vector, no mitigation        |
| Moderate        | 1.5   | Possible but requires specific conditions  |
| Low Likelihood  | 1.0   | Theoretical risk, mitigating factors exist |
| Negligible      | 0.5   | Edge case, minimal real-world impact       |

### Risk Priority Matrix Output

| Priority | Risk Item          | Score | Exposure | Effort | ROI  |
| -------- | ------------------ | ----- | -------- | ------ | ---- |
| P0       | [Critical item]    | 2/10  | 48.0     | S      | 24.0 |
| P1       | [High-risk item]   | 4/10  | 18.0     | M      | 6.0  |
| P2       | [Medium-risk item] | 5/10  | 11.25    | L      | 2.25 |

ROI = Risk Exposure / Effort Score (S=2, M=3, L=5, XL=8)
```

### ✅ REQUIRED [CRITICAL]: Remediation Plan Generator

Every non-compliant item MUST have an actionable remediation plan.

```markdown
## Remediation Plan Format

### [Item Name] — Priority P0

- **Current Score**: 3/10
- **Target Score**: 8/10
- **Risk Exposure**: 21.0
- **Effort Estimate**: Medium (3-5 days)
- **Prerequisites**: [other items that must be fixed first]
- **Remediation Steps**:
  1. [Specific action with file paths]
  2. [Configuration change with exact values]
  3. [Code pattern to implement with pseudo-code]
- **Validation Criteria**:
  - [ ] [How to verify fix is correct]
  - [ ] [Test to run]
  - [ ] [Metric to check]
- **Assignable To**: [Role/Team — e.g., "Backend Team", "DevOps", "Security"]
- **Sprint Fit**: [Can fit in current sprint | Requires dedicated sprint]
```

### ✅ REQUIRED [CRITICAL]: Trend Tracking Format

Enable comparison across evaluations with structured history.

```markdown
## Trend Record

### Evaluation History

| Date       | Overall Score | Grade | Items Evaluated | Compliant | Gaps | Delta |
| ---------- | ------------- | ----- | --------------- | --------- | ---- | ----- |
| 2026-03-26 | 72/100        | C     | 43              | 28        | 15   | —     |
| 2026-04-15 | 81/100        | B     | 43              | 34        | 9    | +9    |

### Category Trends

| Category       | Previous | Current | Delta | Trend |
| -------------- | -------- | ------- | ----- | ----- |
| Security       | 65       | 78      | +13   | ↑     |
| Error Handling | 58       | 62      | +4    | ↑     |
| Testing        | 82       | 84      | +2    | →     |
| Observability  | 45       | 45      | 0     | →     |
```

### ✅ REQUIRED: Executive Summary Template

Generate a leadership-ready summary, not just a technical report.

```markdown
## Executive Summary

### Compliance Dashboard

| Metric                   | Value       |
| ------------------------ | ----------- |
| Overall Compliance Score | 72/100 (C)  |
| Critical Items Passing   | 18/25 (72%) |
| High-Risk Exposures      | 4           |
| Estimated Remediation    | 15-20 days  |
| Top Priority             | [Item name] |

### Key Findings (Top 3)

1. **[Finding]** — Impact: [X], Risk: [Y], Fix effort: [Z]
2. **[Finding]** — Impact: [X], Risk: [Y], Fix effort: [Z]
3. **[Finding]** — Impact: [X], Risk: [Y], Fix effort: [Z]

### Recommendation

[One paragraph: what to fix first, why, and expected score improvement]
```

### ❌ NEVER: Score Without Evidence

Never assign a compliance score without examining actual code, configuration, or documentation. An assessment without evidence is an opinion, not an evaluation.

### ❌ NEVER: Use Only Labels

Never output just "Compliant" / "Non-Compliant" without the numeric score, evidence, and remediation plan.

### ❌ NEVER: Ignore Impact Weights

A Critical item scoring 5/10 is far worse than an Optional item scoring 3/10. Always apply impact weights in aggregate calculations.

---

## Decision Tree

```
Starting OE Evaluation?
  → Check for existing report at /agent-output/ → YES → Load and diff-update
  → Check for project_metadata_report.md → YES → Load for context
  → NO → Proceed with full evaluation

For each checklist category:
  → Dispatch subagent with category-specific instructions
  → Subagent collects evidence (code search, file analysis, config review)
  → Subagent scores each item (0-10) with evidence records
  → Subagent generates remediation plans for items < 8/10

After all categories evaluated:
  → Calculate weighted aggregate score
  → Generate risk exposure matrix
  → Sort remediations by ROI (risk exposure / effort)
  → Generate executive summary
  → Generate trend comparison (if previous report exists)
  → Write multi-section output report

Confidence < Medium on any item?
  → Flag for manual review
  → Search deeper with additional queries
  → If still low confidence → Mark as "Requires Manual Verification"
```

---

## Conventions

Refer to [conventions](../conventions/SKILL.md) for general coding standards.
Refer to [architecture-patterns](../architecture-patterns/SKILL.md) for SOLID, Clean Architecture, and DDD evaluation criteria.

### OE-Engine-Specific

- Always use numeric scores (0-10), never just text labels
- Always include evidence with file paths
- Always calculate weighted aggregates
- Always generate remediation plans for scores below 8
- Always track search queries used (reproducibility)
- Sort remediation by ROI, not alphabetically
- Use impact weights consistently throughout the report

---

## Edge Cases

### Monorepo with Multiple Apps

When evaluating a monorepo, score each app independently, then calculate an aggregate:

```
Overall = weighted_average(app_scores)
Report per-app scores alongside aggregate
```

### Missing Categories

If a checklist category is N/A for the repo (e.g., gRPC for a frontend-only project):

```
Score: N/A
Evidence: "Category not applicable — no gRPC service detected"
Weight in aggregate: Excluded (do not penalize)
```

### Incomplete Evidence

When evidence is ambiguous or insufficient:

```
Confidence: Low
Score: Best estimate with ±2 range noted
Action: "Requires manual verification by [Role]"
```

### First-Time Evaluation

No previous report exists:

```
Trend: "Baseline evaluation — no historical data"
Delta: N/A
Recommendation: "Establish quarterly evaluation cadence"
```
