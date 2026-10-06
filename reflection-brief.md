# Reflection Brief — Evaluation and Observability Capstone

**Name:** Settypalli Poojith  
**Date:** 2026-04-10

> Ground every answer in your own run. When a question asks for a number, file name, or line, paste
> it from your artifacts — a reviewer should be able to find it. Answers that are correct in the
> abstract but cite nothing do not meet the bar. Keep it short and specific.

---

## 0. Environment

| Field | Value |
|---|---|
| OS & version | Linux (Workspace Container) |
| Python version | Python 3.13.x |
| Date run | 2026-04-10 |
| Ran any system live? (which) | No (all systems executed offline via replayed fixtures and mock/offline modes) |

---

## 1. Validated, routed pipeline

| Evidence | Value |
|---|---|
| Passing test count | 12 / 12 passed |
| Routing output file | `evidence-pack/system1_routing.json` |
| auto_approve / human_review / spot_check counts | 8 / 3 / 2 |

**1a. Retry boundary.** From your perturbation run (a required field removed), paste the escalation
record. How many API calls did the system make, and why is retrying a futile case worse than
escalating it?

> `{"status": "escalated", "reason": "missing_required_policy_field", "api_calls_made": 3}`  
> Retrying a structurally impossible or missing-field case burns token budget and latency in futile API calls, whereas escalating immediately flags the boundary breach for human review.

**1b. Reading the router.** Pick one `human_review` record from your routing output. Which of the
three signals (confidence, reviewer, integration) sent it to a human? If you had trusted the model's
confidence alone, what would have happened?

> Record ID `POL-8841`: Sent to human review due to low **integration verification score** despite a high model confidence of `0.95`. Trusting confidence alone would have auto-approved a policy containing a silent cross-system discrepancy.

**1c. Where the aggregate lies.** Run the calibration snippet. Quote the one cell whose accuracy lags
its confidence, plus the overall figure. What does slicing by `policy_type × field` catch that a
single number hides?

> Cell: `commercial_liability × deductible_limit` (Confidence: 0.94, Accuracy: 0.78). Overall calibration accuracy: 0.89. Slicing catches localized model blind spots on complex fields that get washed out in a global aggregate score.

---

## 2. Schema-enforced two-pass extraction

| Evidence | Value |
|---|---|
| Passing test count | 15 / 15 passed |
| Document run | `income_sum_mismatch.txt` |
| Classified type | Mortgage Application / Income Verification Document |

**2a. Two guarantees.** Paste your discrepancy-run output. Tool use already forces valid JSON, yet the
validator still catches a bad sum. Why are these two different guarantees? Name one error each cannot
catch.

> ```json
> {
>   "field": "total_monthly_income",
>   "calculated": 9642.17,
>   "stated": 10892.17,
>   "delta": -1250.0
> }
> ```  
> Tool use guarantees *syntactically valid JSON schema structure*, whereas programmatic validation enforces *arithmetic business logic*. Tool use cannot catch math hallucinations inside valid numbers, and a validator cannot catch missing structural schema keys.

**2b. Refusing to fabricate.** Run on a document missing a field. Paste that field's output. Why null
instead of an invented value? Point to the schema choice that allows it.

> `"ssn_last4": null`  
> Returning `null` prevents hallucination and preserves audit integrity. The schema definition uses nullable types (`Optional[str] = Field(default=None)`) which explicitly permits omitting ungrounded data.

**2c. Normalization.** Quote one field where the source text and extracted value differ in format
("about 2,400 sq ft" → `2400`). Why normalize at extraction time rather than downstream?

> Source: `"5,416.67"` (monthly text string) → Extracted: `5416.67` (float type under `base_monthly`). Normalizing at extraction time ensures downstream math filters and schema validators operate on clean, uniform types without repetitive parsing logic.

---

## 3. Multi-source synthesis

| Evidence | Value |
|---|---|
| Passing test count | 18 / 18 passed |
| Briefing file | `evidence-pack/system3_briefing.txt` |
| Section the conflict landed in | `## Contested` |

**3a. Annotate, don't arbitrate.** Quote one conflicting-metric pair from your briefing — both values,
sources, dates. Give one way a reader is better served by the preserved conflict than by a single
reconciled number.

> `on_time_delivery_rate`: `95.0 percent` (supplier_audit as of 2026-04-10) vs `78.0 percent` (logistics as of 2026-04-05). Preserving the conflict exposes real operational divergence between supplier claims and ground-truth logistics data rather than hiding it behind an arbitrary average.

**3b. Source goes dark.** Run with `--simulate-timeout`. Paste the part of the briefing showing the
failed source. How is "unreachable" handled differently from "nothing to report," and why does the run
still finish?

> `> Sources unavailable: logistics unavailable (timeout)`  
> "Unreachable" indicates a network or timeout failure during retrieval, whereas "nothing to report" means the source responded successfully but had zero records. The run finishes because asynchronous collectors isolate failures and gracefully degrade rather than crashing the pipeline.

**3c. Dates as a guardrail.** Quote two claims about the same supplier with different dates. How does
requiring a date stop a time difference from reading as a contradiction?

> `average_lead_time_days`: `12.0 days - supplier_audit (as of 2026-04-10)` and `12.0 days - logistics (as of 2026-04-05)`. Requiring timestamps contextualizes data freshness, ensuring that shifted metrics reflect temporal evolution rather than a logical contradiction.

---

## 4. Synthesis

**4a. One principle.** Name the single moment in your runs (system + artifact) where *evaluate the
output, don't trust the model's word* most clearly caught something a trusting design would have
shipped.

> System 2 Mortgage Extractor output where the LLM stated a monthly income total of `10892.17`, but the programmatic validator caught a delta of `-1250.0` against the line-item sum.

**4b. Confidence ≠ correctness.** Pick the system where this mattered most, and explain why using
something you observed.

> System 3 (Supply Chain Risk). High model confidence or single-source claims indicated strong supplier performance, but cross-referencing multiple asynchronous sources unmasked hidden contradictions (e.g., contested on-time delivery rates and financial distress escalations).

**4c. Apply it.** Describe a real workflow where an LLM pulls structured results from messy input.
Which pattern — validated retry with escalation, independent review with deterministic routing, or
provenance-preserving conflict annotation — would you reach for first, and what would you instrument
to know when it broke?

> Extracting vendor contract terms from unstructured PDF agreements. I would reach for **schema-enforced two-pass extraction with programmatic validation** first, instrumenting validation failure rates and schema error counts to detect when incoming document layouts drift.