# Experiments: improving methods safely

The system loop exists so the council gets better at its work without anyone redefining "better" after seeing the results. The Improvement Scientist (`council/agents/improvement_scientist.py`) is the only agent that proposes method changes, and it can only propose them.

## Lifecycle

```
recurring pattern (≥3) → proposed → approved by Antonio → shadow → evaluated against pre-registered threshold → adopted | rejected
```

1. **Detection.** The agent counts patterns in the audit log (agent errors, Evidence Verifier flag codes such as `preprint_only` or `under_supported`) and in the evaluation register (rubric criteria scored below level 2). A pattern becomes eligible at three occurrences, and only if the same hypothesis is not already open.
2. **Pre-registration.** Each proposal is an `Experiment` with a hypothesis, intervention, control, primary metric, success threshold, guardrail metrics, sample and duration. `preregistration_hash` (SHA-256 over hypothesis, intervention, control, primary metric and success threshold) is computed when the proposal is created. Changing any of them later produces a different hash, so a re-specified experiment is detectable and must be evaluated against the hash Antonio approved.
3. **Approval.** Experiments are high risk. They appear in the nightly review issue and do nothing until Antonio approves them. Approval is recorded in `evaluation_register.decisions`.
4. **Shadow mode.** An approved experiment first runs alongside the current method without affecting published text or learners' scores. For knowledge-loop experiments, the council produces both outputs and records both; for learner-loop experiments, the intervention is offered to a cohort while the locked rubric and assessment stay unchanged.
5. **Evaluation.** At the end of the pre-registered duration, the primary metric is compared with the threshold and the guardrails are checked. The result, whether it passed or not, is appended to `evaluation_register.experiments` and `shadow_runs`, which are append-only.
6. **Decision.** Antonio adopts or rejects. Adoption changes `config.py` or an agent through an ordinary pull request that cites the experiment id.

## The current playbook

| Pattern | Hypothesis (abridged) | Primary metric | Pre-registered threshold |
|---|---|---|---|
| `preprint_only` | Searching for published versions of preprint-only citations raises peer-reviewed support | share of refreshed claims with ≥1 peer-reviewed source | +15 pp over 2 cycles, no drop in claims refreshed |
| `under_supported` | Ranking single-source claims first shrinks the under-supported set faster | claims with <2 sources | ≥20% reduction in 30 days vs projected trend |
| `citation_not_anchored` | Naming the cited author in the sentence improves traceability | share of revised claims with an anchored citation | ≥90%, no essay shortened |
| `weak_criterion` | A worked contrast example before the practical task lifts the weakest criterion on transfer | mean level on that criterion at the 30-day check | +0.5 level vs control |
| `agent_error` | Schema validation before dispatch removes recurring errors | agent errors per run | zero over 7 nightly runs |

Edition 119 starts with 9 preprint-only and 125 under-supported claims (baseline `BASE-E119-EVIDENCE`). These are the denominators the first two experiments would be judged against.

## Safety rules

- **No self-deployment.** The agent that proposes an experiment cannot start it, extend it or declare it successful.
- **No moving goalposts.** Thresholds are hashed at registration. A re-specified experiment is a new experiment with a new id.
- **Guardrails are binding.** A primary-metric win that breaches a guardrail (for example, coverage or essay length falling) counts as a failure.
- **Learners are protected.** Interventions never change a locked rubric, never alter a learner's recorded score, and never withhold the standard unit from anyone. Synthetic calibration data is never reported as learner evidence.
- **Negative results are kept.** Failed experiments stay in the register so the same idea is not quietly retried.
- **Small and reversible first.** Prefer changes to ranking, templates or hint order over changes to published text.
