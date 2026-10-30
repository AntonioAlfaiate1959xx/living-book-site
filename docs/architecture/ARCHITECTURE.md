# Learning & Performance Architecture

The Living Book on AI in education already renews itself: a council of AI agents refreshes its answers every month, each edition is archived in `docs/versions/`, and Antonio's own questions and hand-edited answers are preserved across rewrites. This architecture adds what that process lacked: a way for the book to learn from evidence about its own accuracy, from evidence about whether readers actually learn from it, and from evidence about which of its own methods work. It does so without giving any agent the power to publish.

## Three principles

1. **Agents propose; independent evaluation decides; humans approve.** No agent can approve its own output. Every consequential change (published text, rubrics, experiments, releases) reaches Antonio as a proposal.
2. **Constraints live in code, not in prompts.** The orchestrator applies its checks to every agent result unconditionally. There is no flag that disables them, and an agent cannot talk its way past a permissions table.
3. **Everything is versioned and reversible.** Memory writes are snapshotted before they happen; the audit log is append-only and hash-chained; archived editions are never touched.

## Layers

```
                ┌──────────────────────── Governance ────────────────────────┐
                │ config.py: permissions · risk classes · quality gates      │
                │ orchestrator.py: 9 hard constraints · audit-before-handoff │
                │ Human approval: Antonio reviews the nightly issue / PR     │
                └────────────────────────────────────────────────────────────┘
   Knowledge loop                 Learner loop                    System loop
   (accuracy, currency)           (understanding, transfer)       (better methods)
   Research Scout                 Learning Designer               Improvement Scientist
   → Evidence Verifier            → Assessor                      Release Steward
   → Knowledge Curator            → Tutor
   → Editor
   Release Steward
                ┌──────────────────────── Shared memory ─────────────────────┐
                │ memory/: question_map · evidence_register · edition_archive│
                │ learning_unit_library · evaluation_register · audit_log    │
                └────────────────────────────────────────────────────────────┘
                ┌──────────────────────── Canonical book (read-only) ────────┐
                │ data/question-registry.json · data/claims/ · editions/     │
                │ docs/versions/ · quality/a7-approval.json                  │
                └────────────────────────────────────────────────────────────┘
```

The canonical book stays the source of truth. `council/repo.py` reads it; nothing in the council writes to it. Memory is an index of the book plus the working state the book never tracked: verification status per claim, learning units, assessments, experiments and the audit trail.

## The three loops

**Knowledge loop** (nightly). The Research Scout ranks questions by refresh need (stale sources, single-source claims, preprint-only claims, disputed claims) and returns candidates within the approved topics and source types. The Evidence Verifier checks each affected claim against the source registry: source count, credibility mix, staleness, preprint status, vendor-authored evidence and, optionally, live link resolution. The Knowledge Curator traces the impact of each change across the 708 cross-reference edges and into learning units, and keeps disputes visible rather than resolving them. The Editor drafts minimal revision plans that respect the book's voice and never shorten an essay; author-protected answers (q272–q279) get extension plans only. The Release Steward reports whether a release would be permissible.

**Learner loop.** The Learning Designer turns a book question into a unit with traceable objectives, a baseline probe, retrieval practice, a practical task, delayed checks and a transfer task, and drafts the rubric. Once Antonio locks the rubric, the Assessor scores submissions against it without ever seeing the book's model answer, and refuses to score if the rubric hash has changed. The Tutor then builds hint ladders (recall → locate → contrast) for the weakest criteria. It may support but never certify.

**System loop.** The Improvement Scientist reads the audit log and evaluation register for recurring failures. A pattern that recurs three times becomes a pre-registered experiment: hypothesis, control, primary metric, success threshold and guardrails, hashed at proposal time and run in shadow mode first. See `EXPERIMENTS.md`.

## Automation by risk

| Risk | Examples | What happens |
|---|---|---|
| Low | source candidates, verification status, impact traces, gap flags, support plans, calibration scores | Applied to council memory only (never to published text), logged, reversible |
| Medium | conflict flags, source refreshes, new or revised learning units, release blockers | Proposal only; surfaced for human review |
| High | answer revisions, experiments, releases | Proposal only; human approval **and** all quality gates |
| Forbidden | deleting history, editing protected paths, changing a locked rubric, releasing without an archive | Rejected by the orchestrator and logged as a gate violation |

Risk comes from `config.PROPOSAL_RISK`. An agent cannot downgrade it.

## Hard constraints (enforced in `council/orchestrator.py`)

1. Agents emit only the proposal kinds and read only the collections their row in `PERMISSIONS` allows.
2. Proposal risk is assigned centrally; anything above low requires approval.
3. No agent approves its own proposal.
4. No proposal targets a protected path: `.github/`, `quality/a7-approval.json`, `docs/versions/`, `editions/ledger.json`, `editions/baselines/`, `archive/legacy/`, `memory/audit_log.jsonl`, `memory/history/`.
5. History is never deleted. `MemoryStore` refuses writes that remove entries from append-only lists.
6. Locked rubrics cannot change, and the Assessor never receives protected answers.
7. The Tutor cannot certify mastery.
8. No release without an archived previous edition and `approved: true` in the A7 gate. The council never publishes.
9. Every agent result is written to the audit log before the next agent sees it.

## Publication policy: PR first

The council's output is a report, not a commit. In dry-run mode (the default for the nightly workflow) nothing is persisted at all. In live mode, only low-risk working-memory updates are applied. Changes to the book itself reach main the same way every edition does: as a pull request that Antonio reviews, behind the A7 gate and the monthly archive.

## Cost

The council is deterministic by default and makes no LLM calls. `--use-llm` enables a budgeted OpenAI wrapper (`council/llm.py`) capped at `LLM_MAX_CALLS_PER_RUN`, with every call's estimated cost written to the audit log. The nightly knowledge loop completes in seconds on a GitHub runner at no API cost.

## Related documents

- `AGENT_ROLES.md`: each agent's responsibility and authority boundary
- `TRACEABILITY.md`: the chain from question to decision
- `QUALITY_DIMENSIONS.md`: the seven quality dimensions
- `EXPERIMENTS.md`: how methods are improved safely
- `memory/MEMORY.md`, `learning/LEARNING_SYSTEM.md`, `quality/QUALITY.md`: operational guides
