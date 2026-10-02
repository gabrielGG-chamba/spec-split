# SPEC 02 — Split quality: decisions, validation and repo protection

> **Status:** Approved
> **Depends on:** SPEC 01
> **Date:** 2026-10-02
> **Objective:** Make `/spec-split` produce plans a developer can execute in order, with reviewable PRs and no surprises when integrating them.

---

## Why this spec exists

SPEC 01 covers the whole operational flow, but the skill applies split rules more than it reasons about architecture and dependencies. A review of the skill found these gaps:

1. "UI → services → integration" reads like a default. Layer splits can produce PRs that deliver nothing usable and depend heavily on each other.
2. Stacked PRs are allowed without justifying the dependency, and there is no instruction for when an earlier PR changes during review.
3. "Every criterion in exactly one sub-spec" forces artificial assignments for cross-cutting criteria, and does not trace a criterion to the step and test that prove it.
4. "Every commit builds and passes every check" is too rigid for intermediate steps (e.g. a shared contract change), and can push toward artificial steps.
5. The repo restrictions are stated, not verified: no `git status` before/after, no check that only authorized files changed.
6. The four checks can be ticked on a split that is still wrong (a PR that only prepares unused code passes them).

---

## Scope

**In** (all in `skills/spec-split/SKILL.md`):

- **Repository guard.** `git status` before starting and after writing; stop on uncommitted changes in the files the skill will touch; never overwrite or delete; after writing, the only changes are the new sub-specs and the authorized parent sections; stop and ask on anything unexpected.
- **Spec validation.** Before splitting, detect ambiguous requirements, contradictory criteria, undefined dependencies and pending decisions; ask when they affect the split.
- **Functional split.** End-to-end verifiable functionalities first; split by layer only when each piece has its own purpose, can be tested and leaves the app working. PR size judged by complexity, risk, number of behaviors, review difficulty and testability, not by looking big.
- **Seven validations** per sub-spec: the four checks plus functional independence, justified dependencies and scope integrity. A proposal that fails is fixed before asking for approval.
- **Dependencies.** Every stacked PR states exactly what it needs from the previous one and why it cannot target the base branch; no stacking without a real dependency. Shared contracts (models, APIs, schemas, permissions, data structures) get a compatible sequence. Instructions for when an earlier PR changes during review.
- **Traceability matrix.** Criterion → sub-spec → step → verifying test. Cross-cutting criteria are decomposed, or their dependency and end-to-end check are explained; never assigned artificially.
- **Verifiable commits.** Every commit leaves the repo coherent, with the checks relevant to that step (build, unit, lint, integration assigned per step). No artificial steps.
- **Conventions checked against real examples** for file names, branches, PR titles and commit messages.
- **Output format.** Each sub-spec adds: verifiable result, dependencies (with reason), risks, tests (new, existing to run, manual check), criteria covered, integration (base branch and reason). The proposal shows dependencies and risks per PR and the matrix.
- **Final coherence check** before finishing: links between sub-specs, no missing criteria, parent changed only in authorized places.
- `README.md` updated to describe the new behavior.

**Out of scope:**

- Running builds or tests: the skill still only plans.
- Creating branches, commits or PRs in the target repo.
- Changing the sub-spec file naming (`NN.k-slug.md`).

---

## Data model

No persisted data. New sections in the sub-spec template: **Verifiable result**, **Dependencies**, **Risks**, **Tests**, **Criteria covered**, **Integration**. The parent's "Delivery in sub-specs" section gets a traceability table.

---

## Implementation plan

1. This spec.
2. Repository guard.
3. Spec validation before splitting.
4. Functional split and the seven validations.
5. Justified dependencies and stacked PRs.
6. Traceability matrix and cross-cutting criteria.
7. Per-step checks for commits and conventions checked against real examples.
8. Dependencies, risks and tests in the proposal and the sub-spec template.
9. Final coherence check.
10. README.

---

## Acceptance criteria

- [ ] With uncommitted changes in the parent spec, the skill stops before proposing anything and asks.
- [ ] After writing, the skill runs `git status` and reports that only the new sub-specs and the parent spec changed; on anything else it stops.
- [ ] The skill never writes to a path that already exists, except the parent spec.
- [ ] A spec with contradictory criteria or an undefined dependency produces questions before the proposal.
- [ ] A proposal never contains a PR whose only result is unused code, unless the user accepts it explicitly with the reason written.
- [ ] Every stacked PR states what it needs from the previous one; an independent PR targets the base branch.
- [ ] The proposal includes a matrix: criterion → sub-spec → step → test; a cross-cutting criterion is decomposed or explained.
- [ ] Each step lists the checks that apply to it; no step exists only to add a commit.
- [ ] Each sub-spec has Verifiable result, Dependencies, Risks, Tests, Criteria covered and Integration.
- [ ] The final message includes the coherence check result.

---

## Decisions

- **Yes:** layer splits stay allowed, but only with a justification per piece. Some specs (e.g. a backend capability before its UI) genuinely need them.
- **Yes:** a PR that prepares code with no user-visible effect is acceptable when its result is still verifiable by tests and the dependency is justified; it is flagged, not forbidden.
- **Yes:** `git diff` is added to the allowed tools, read-only, to verify the parent spec changed only in authorized sections.
- **No:** running the project's checks from the skill. It plans which checks apply per step; `/spec-impl` runs them.

---

## Risks

| Risk | Mitigation |
| --- | --- |
| More validations make the skill slower and the proposal longer | The matrix and validations are compact tables; the proposal stays one message. |
| The user has unrelated uncommitted work | Only changes to the files the skill touches block it; other changes are reported and left alone. |
