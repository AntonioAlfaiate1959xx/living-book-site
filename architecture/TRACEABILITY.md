# Traceability

Every decision the system makes should be explainable backwards, from the decision to the evidence and the question that prompted it, and forwards, from a changed source to every passage, objective and assessment it touches. The chain is:

```
Question → Claim → Evidence → Passage → Learning objective → Assessment → Result → Decision
```

## The links and where they live

| Link | Stored in | Key |
|---|---|---|
| Question | `data/question-registry.json`, indexed in `memory/question_map.json` | `q001` … `q279` |
| Question → Claim | `data/claims/qNNN.json` (each answer section is a claim) | `q001-s01` … `q001-s06` |
| Claim → Evidence | `memory/evidence_register.json` → each claim's sources | permanent `SRC-n` numbers from `data/source-registry.json` |
| Claim → Passage | the claim's section text within the answer essay | claim id |
| Claim → Claim (across questions) | `memory/question_map.json` → `edges[]` | `{from, to, via_claim}` |
| Passage → Objective | `learning/pilot/LU-001.json` → `traceability.objective_to_claims` | `O2 → q001-s02` |
| Objective → Assessment | unit tasks and rubric criteria (`assessment_rubric`, `learning/rubrics/RUB-001.md`) | `RUB-001/counter` |
| Assessment → Result | `memory/evaluation_register.json` → `assessments[]` | `submission_id`, `rubric_hash` |
| Result → Decision | `memory/audit_log.jsonl` and the run's `proposals.json` | proposal `id`, audit `id` and `hash` |

## A worked example: Q001

Question q001 asks what education is for in a world where machines can know, reason and create. Its answer is stored as six claims, `q001-s01` to `q001-s06`, in `data/claims/q001.json`. Each claim's sources, credibility and last verification are recorded in the evidence register.

Learning unit LU-001 has four objectives, each mapped to the claims that support it. Objective O2 (vocational retraining is a legitimate but insufficient purpose, and vendor and policy framings must be read as positions) traces to `q001-s02`. That claim is also where q001 cross-references q007, so the question map holds the edge `q001 → q007 via q001-s02`. RUB-001's `counter` criterion (weight 0.25) assesses the reasoning. The rubric was locked with SHA-256 `3073ca3bf84e…` before any submission was scored, and every score records that hash. If real learners repeatedly scored below level 2 on that criterion, the Improvement Scientist would count the `weak_criterion` pattern. At three occurrences it would propose the worked-contrast experiment, and the proposal, its pre-registration hash and Antonio's decision would all land in the audit log.

Read forwards, the same chain answers a different question. If a source behind `q001-s02` is retracted, the Verifier flags the claim, and the Curator's impact trace lists q001, q007 (linked through that claim), objective O2 of LU-001 and the RUB-001 criteria that assess it. The Editor's revision plan and the Designer's unit-revision proposal both cite that trace.

## Integrity of the trail

- **Audit log.** Each line holds `id`, `ts`, `run_id`, `event`, `actor`, `payload`, `prev_hash` and `hash`. Each hash covers the previous one, so editing or deleting any line breaks every later one. `python -m council.run --verify-audit` checks the whole chain. `AuditLog` has no update, delete or truncate method.
- **Audit before handoff.** The orchestrator writes each agent's result before the next agent receives it, so the log shows what each agent knew when it acted.
- **Memory snapshots.** Before any write, the previous version of a collection is copied to `memory/history/<collection>/v00007.json` (one file per version) and indexed in `memory/changes.jsonl`. Rollback restores a snapshot as a new version, so even a rollback is recorded rather than erasing what came before.
- **Hashes on commitments.** Rubrics and experiment pre-registrations are hashed when they are locked. A changed hash is detectable and blocks scoring or evaluation.
- **Permanent identifiers.** Sources keep their `SRC-n` number for life; questions keep their id even when deprecated; archived editions keep their directory.
