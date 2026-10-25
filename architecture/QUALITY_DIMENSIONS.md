# Quality dimensions

A living book can get better in ways that conflict with one another. More questions can mean thinner answers, faster refreshes can mean weaker verification, and more tutoring support can mean less independent learning. The quality profile names seven dimensions so these trade-offs are made openly. It also marks two dimensions as non-negotiable, so they are never traded at all.

The profile lives in `quality/quality_profile.json`; the measurement code is in `quality/generate_report.py`; the operational guide is `quality/QUALITY.md`.

## 1. Accuracy (hard constraint, minimum 0.90)

**Question.** Does each claim say what its sources support?
**Method.** Mean Evidence Verifier confidence over reviewed claims. Confidence falls with fewer than two sources, a low share of high-credibility sources, staleness beyond 90 days, preprint-only support, vendor-authored evidence and unresolved links.
**Honesty rule.** Unscored until 80% of claims have been reviewed, because the first claims reviewed are the ones flagged for problems and a partial mean would be biased.
**Why hard.** A book about evidence in education that misstates its evidence fails at its one job.

## 2. Coverage (minimum 0.95)

**Question.** Does every active question have a full answer?
**Method.** Share of active questions with answers of at least 1,700 words (the refreshed floor; the editorial target is 2,000–2,200).
**Edition 119.** 238 of 238.

## 3. Coherence (minimum 0.60)

**Question.** Is the book an argument or a pile of essays?
**Method.** Share of active questions with at least one explicit cross-reference edge in the question map.
**Edition 119.** 237 of 238, across 708 edges. The next refinement is edge reciprocity and chapter-level connectivity.

## 4. Historical integrity (hard constraint, minimum 1.00)

**Question.** Can a reader still read, and cite, every past edition?
**Method.** For every entry in `docs/versions/versions.json`, check that the page, manifest and compressed content snapshot exist, decompress, and agree on version and data-file count.
**Edition 119.** 8 of 8.
**Why hard.** Readers cite editions. An edition that silently changes or disappears breaks every citation to it.

## 5. Operational reliability (minimum 0.95)

**Question.** Does the council do its work without failing?
**Method.** Agent success rate across recorded council runs (minimum three runs). Gate violations and rejected proposals are shown on the dashboard alongside it.

## 6. Learning effectiveness (minimum 0.67)

**Question.** Do readers who work through a unit meet its objectives without help?
**Method.** Mean moderated rubric share across real learner assessments. 0.67 matches RUB-001's proficiency threshold.
**Honesty rule.** Synthetic calibration samples are excluded. Scores count only after human moderation.

## 7. Independence and transfer (minimum 0.50)

**Question.** Can learners apply the idea somewhere new, alone?
**Method.** Share of independent transfer-task assessments (an unseen context, no Tutor support) reaching the proficient band. For LU-001 this is the ministry-policy memo.
**Why separate.** Performance on a practised task can rise while transfer stays flat. Measuring only the first would reward teaching to the test.

## How the dimensions are used

- The **Release Steward** blocks a release proposal if a hard constraint is measured and failing, and reports unmeasured hard constraints as warnings.
- The **Improvement Scientist** uses the tradeable dimensions as guardrail metrics. An experiment that improves one dimension by degrading another fails its pre-registered guardrail.
- **Antonio** sees all seven on the dashboard, with the evidence and method behind each number.
- **Null means unmeasured, not zero.** A dimension stays empty until its evidence threshold is met.
