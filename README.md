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

1. Finds the spec and checks it is approved and not split yet (sub-specs are matched by their `Parent spec` link, so two specs sharing a number don't get mixed up). If it is already split, offers to stop, add sub-specs or re-plan only the `Draft` ones.
2. Records the working tree as a baseline (repository guard) and refuses to start if the parent spec has uncommitted changes.
3. Reads `CLAUDE.md`/`AGENTS.md`, recent specs and `git log` to copy the language, header labels, commit style, branch naming and base branch, and asks about ambiguities in the spec that affect the split.
4. Proposes the split with **few cuts**: it starts from one PR and only cuts where it matters (risky change, real prerequisite, independent functionality, switch-over), usually 1–3 sub-specs, each cut with its reason. Sub-specs go by functionality, end to end first (by layer only when justified), steps per sub-spec with one commit message and its checks, and PR, branch and base per sub-spec. Every sub-spec passes seven validations, each with a reason: one sentence, verifiable result, reviewable alone, repo not broken, functional independence, justified dependencies and scope integrity. Every acceptance criterion of the parent is traced to a sub-spec, a step and a verification.
5. You confirm or adjust the split. **Nothing is written before this point.**
6. Writes `specs/NN.k-slug.md` per sub-spec (scope, dependencies, integration, risks, steps with commit messages, acceptance criteria, tests, how to verify, PR title and description, heads-up message for the reviewer) and adds a "Delivery in sub-specs" table with traceability to the parent spec. Then checks links and numbering and confirms with `git status`/`git diff` that only those files changed.

The parent spec and all sub-specs are committed together in the first step of PR 1 (`docs(spec): add delivery plan for SPEC NN`), so the delivery table never links to files that are missing from the base branch.

## What it does not do

- It only writes `.md` files in `specs/`. It never writes code, creates branches, commits, pushes or opens PRs.
- Sub-specs start as `Draft`: approve each one, then implement it (e.g. with `/spec-impl NN.1`, `/spec-impl NN.2`, …).

## Example output

```text
SPEC 09 — Part management  →  3 sub-specs / 3 PRs

09.1 Shared part rules                        PR 1 · branch spec09/1-part-rules ← base main
     1. Delivery plan                         docs(spec): add delivery plan for SPEC 09
     2. Code normalization                    feat(inventory): add part code normalization
     3. Input validation                      feat(inventory): add part input validation

09.2 Part management Lambda                   PR 2 · branch spec09/2-gestion-pieza ← base spec09/1-part-rules
     Depends on:    09.1 — needs `normalizeCode` from amplify/shared/part.ts (not in main yet)
     1. Mutation handler                      feat(inventory): add gestion-pieza mutation handler
     ...
```

## License

MIT
