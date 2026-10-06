# Perturbation Log

For each system, make one deliberate change to an input or configuration, predict the outcome, run
it, and record what actually happened. See the starters in the Instructions, or design your own (your
own experiment earns more credit).

---

### System 1 — validated, routed pipeline

- **Change I made (file + what I changed):** Removed a required schema policy field (`deductible_limit`) from a sample policy fixture file.
- **Command I ran:** `.venv/bin/policy-extractor pipeline data/policies/`
- **What I predicted:** The pipeline would encounter a validation error due to the missing required field, exhaust its retry attempts, and trigger the router's fallback escalation path.
- **What actually happened (paste the key output line):** `{"status": "escalated", "reason": "missing_required_policy_field", "api_calls_made": 3}`
- **How this differs from the unperturbed run:** A standard run successfully passes schema checks and gets routed to `auto_approve` or `spot_check`, whereas the perturbed run halts and forces a manual escalation review after burning through retries.

---

### System 2 — schema-enforced two-pass extraction

- **Change I made (file + what I changed):** Modified an income verification fixture file to mismatch the line-item sum against the stated total monthly income.
- **Command I ran:** `.venv/bin/mortgage-extractor run fixtures/income_sum_mismatch.txt`
- **What I predicted:** The LLM would extract the stated text numbers directly, but the programmatic secondary validator would intercept the discrepancy and catch the arithmetic delta.
- **What actually happened (paste the key output line):** `{"field": "total_monthly_income", "calculated": 9642.17, "stated": 10892.17, "delta": -1250.0}`
- **How this differs from the unperturbed run:** An unperturbed valid document matches line items cleanly with a delta of `0.0`, whereas this perturbed version explicitly surfaces the hidden math error and audit discrepancy.

---

### System 3 — multi-source synthesis

- **Change I made (file + what I changed):** Introduced a simulated timeout flag to force one of the primary data source collectors (`logistics`) to drop offline.
- **Command I ran:** `.venv/bin/supply-chain-investigate investigate --simulate-timeout logistics`
- **What I predicted:** The multi-source coordinator would capture the network failure gracefully, isolate the unreachable source, and generate a briefing noting the missing collector without crashing the entire run.
- **What actually happened (paste the key output line):** `> Sources unavailable: logistics unavailable (timeout)`
- **How this differs from the unproven run:** A normal run aggregates all live sources successfully without warnings, whereas this perturbed run isolates the network drop and marks the collector as unreachable while still completing the report synthesis.