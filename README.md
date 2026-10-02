# spec-split

A [Claude Code](https://claude.com/claude-code) skill that splits an **approved spec** into sub-specs: each sub-spec is one specific functionality and becomes one pull request, and each step of a sub-spec is one commit.

| Level | Rule |
| --- | --- |
| Spec (card) | An objective, not a delivery unit. Split into specific, measurable functionalities. |
| Sub-spec | One functionality → one pull request. |
| Step | One commit. |

Use it after writing a spec and before implementing it, when the spec is too big for a single PR.

## Install

With [Vercel's `skills` CLI](https://github.com/vercel-labs/skills) (Node.js 18+):

```sh
npx skills add gabrielGG-chamba/spec-split -a claude-code
```

Or by hand: copy `skills/spec-split/` to `~/.claude/skills/spec-split/` (every project) or `.claude/skills/spec-split/` (one project).

Restart Claude Code and run `/spec-split`.

## Usage

```text
/spec-split                     # pick the spec to split
/spec-split 09                  # spec by number
/spec-split 09-part-management  # spec by file name
```

The repo needs a `specs/` folder with the spec, and its status must mean "Approved" (`Approved`, `Aprobado`, …).

## Flow

1. Finds the spec, checks it is approved and not split yet, and runs `git status`: if the parent spec has uncommitted changes, it stops.
2. Reads `CLAUDE.md`/`AGENTS.md`, recent specs and `git log` to copy the language, header labels, commit style, branch naming and base branch, checking each name against a real example in the repo.
3. Validates the spec: ambiguous requirements, contradictory criteria, undefined dependencies and pending decisions. If they affect the split, it asks first.
4. Proposes the split:
   - **End-to-end functionalities first.** A layer split only when each piece has its own purpose, can be tested and leaves the app working. Size alone is not a reason to split.
   - **Seven validations per sub-spec**, each with a reason: one sentence without "and", verifiable result, reviewable alone, repo not broken, functional independence, justified dependencies, scope integrity.
   - **Justified dependencies.** A PR is stacked only when it needs something exact from an earlier PR; shared contracts follow add → use → remove. Each stacked sub-spec says what to do if the earlier PR changes during review.
   - **Traceability matrix**: every parent criterion → sub-spec → step → test. Cross-cutting criteria are decomposed or get an end-to-end check.
   - **Steps = commits**, each with the checks that apply to it (no artificial steps); the last step runs every check.
5. You confirm or adjust the split. **Nothing is written before this point.**
6. Writes `specs/NN.k-slug.md` per sub-spec (verifiable result, scope, dependencies, integration, risks, steps with commit and checks, criteria covered, tests, how to verify, PR draft, reviewer heads-up) and adds a "Delivery in sub-specs" section with the traceability table to the parent spec.
7. Final coherence check (links, numbering, branches, criteria) and `git status`/`git diff` again: only the new sub-specs and the authorized parent sections may have changed. Anything else, it stops and asks.

## What it does not do

- It only creates sub-specs in `specs/` and adds two authorized parts to the parent spec. It never overwrites or deletes a file, never writes code or configuration, and never creates branches, commits, pushes or opens PRs.
- Sub-specs start as `Draft`: approve each one, then implement it (e.g. with `/spec-impl NN.1`, `/spec-impl NN.2`, …).

## Example output

```text
SPEC 09 — Part management  →  3 sub-specs / 3 PRs

09.1 Shared part rules                        PR 1 · branch spec09/1-part-rules ← base main
     1. Sub-spec                              docs(spec): add sub-spec 09.1 for shared part rules
     2. Code normalization                    feat(inventory): add part code normalization
     3. Input validation                      feat(inventory): add part input validation

09.2 Part management Lambda                   PR 2 · branch spec09/2-gestion-pieza ← base spec09/1-part-rules
     ...
```

## License

MIT
