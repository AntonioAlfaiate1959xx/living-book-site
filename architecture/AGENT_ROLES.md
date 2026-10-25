# Agent roles and authority boundaries

The council has ten members: one orchestrator and nine specialists. Each specialist is a class in `council/agents/` with a `run(task, context, memory)` method that returns an `AgentResult` (proposals, evidence, confidence, `requires_approval`, notes, errors). Read and propose rights come from `config.PERMISSIONS`, which the orchestrator enforces. The boundaries below are the substance of the design. A capable agent with an unclear boundary is a liability; a modest agent with a precise one is useful.

## Orchestrator (`council/orchestrator.py`)

**Responsibility.** Routes tasks in dependency order for each loop, retries transient failures (`MAX_RETRIES`, with backoff), filters every result through the hard constraints, writes each result to the audit log before handing it on, applies low-risk updates to memory in live mode, and writes `proposals.json`, `summary.md` and `audit.jsonl` for each run.
**Can.** Reject proposals, skip agents whose dependencies failed, strip protected answers from the Assessor's context.
**Cannot.** Approve medium- or high-risk proposals, disable a constraint, write to the canonical book or any protected path.

## Research Scout

**Responsibility.** Find where the book most needs fresh evidence, and propose candidate sources within `APPROVED_TOPICS` and `APPROVED_SOURCE_TYPES`.
**Reads.** question map, evidence register. **Proposes.** `source_candidate` (low).
**Cannot.** Add anything to the book. A candidate is a lead, not knowledge.

## Evidence Verifier

**Responsibility.** Check each claim's support: number of sources against the two-source minimum, share of high-credibility sources, staleness (90 days), preprint-only status, vendor-authored evidence, and live link resolution when `--check-links` is set.
**Reads/writes.** evidence register. **Proposes.** `verification_status` (low), `source_refresh` (medium).
**Cannot.** Delete or soften a disputed claim. Disputes stay visible with their counter-positions.

## Knowledge Curator

**Responsibility.** Keep the question map coherent: trace what a change to one answer affects (cross-referenced questions, learning units built on it), flag gaps and conflicting claims.
**Reads.** question map, learning units, evidence register. **Writes.** question map. **Proposes.** `impact_trace`, `gap_flag` (low), `conflict_flag` (medium).
**Cannot.** Resolve a conflict by choosing a side. That is editorial judgement.

## Learning Designer

**Responsibility.** Build learning units whose every objective traces to a section of a book answer, with baseline, retrieval, practical, delayed and transfer stages and a draft rubric.
**Reads.** question map, learning units, evidence register. **Writes.** learning unit library (drafts). **Proposes.** `new_learning_unit`, `unit_revision` (medium).
**Cannot.** Lock a rubric, change a locked rubric, or publish a unit.

## Tutor

**Responsibility.** Give learners graduated support: a hint ladder (recall → locate → contrast) for each weak rubric criterion, ordered by how often learners struggle with it.
**Reads.** learning units. **Proposes.** `support_plan` (low).
**Cannot.** Score, certify mastery, or reveal the assessed task's answer.

## Assessor

**Responsibility.** Score submissions against a locked rubric, criterion by criterion, with a quoted justification for each level. Mark scores `pending_human` until moderated.
**Reads.** learning units (minus protected answers), evaluation register. **Writes.** evaluation register. **Proposes.** `assessment_result` (low).
**Cannot.** See the book's model answer, score against an unlocked or altered rubric (hash mismatch), or moderate its own scores.

## Editor

**Responsibility.** Turn verified evidence into minimal revision plans: which sections to refresh, which sources to add, which cross-references to update. Plans follow the book's voice (essayistic, no bullet points, no first person) and never shorten an essay. Author-protected answers (q272–q279) receive extension plans only.
**Reads.** question map, evidence register, edition archive. **Proposes.** `revision` (high).
**Cannot.** Edit `data/claims/`. Every revision needs Antonio's approval.

## Improvement Scientist

**Responsibility.** Detect failure patterns that recur at least three times and propose pre-registered experiments from a playbook (see `EXPERIMENTS.md`).
**Reads.** audit log, learning units, evaluation register. **Writes.** evaluation register (experiment records). **Proposes.** `experiment` (high).
**Cannot.** Deploy an experiment, or change its success threshold after registration.

## Release Steward

**Responsibility.** Decide whether a release would be permissible: previous edition archived, A7 gate approved, hard quality constraints (from `quality/quality_profile.json`) not failing, with unmeasured hard constraints reported. Index archived editions.
**Reads.** evidence register, evaluation register, edition archive, A7 gate. **Writes.** edition archive index. **Proposes.** `archive_index` (low), `release_blocker` (medium), `release` (high).
**Cannot.** Publish, merge, tag, or write the A7 gate.

## Separation of duties at a glance

| Duty | Who proposes | Who checks | Who decides |
|---|---|---|---|
| New evidence | Research Scout | Evidence Verifier | Antonio |
| Answer revision | Editor | Verifier + Curator + quality gates | Antonio |
| Learning unit | Learning Designer | Assessor calibration | Antonio locks the rubric |
| Learner score | Assessor | Human moderation | Moderator |
| Method change | Improvement Scientist | Shadow run against pre-registered threshold | Antonio |
| Release | Release Steward | A7 gate + archive check | Antonio |
