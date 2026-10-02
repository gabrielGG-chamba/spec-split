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

1. Finds the spec and checks it is approved and not split yet.
2. Reads `CLAUDE.md`/`AGENTS.md`, recent specs and `git log` to copy the language, header labels, commit style, branch naming and base branch.
3. Proposes the split: sub-specs by functionality (never by file or layer), steps per sub-spec, one commit message per step, and PR, branch and base per sub-spec. Every sub-spec passes four checks:
   - It is described in one sentence without "and".
   - It has a result someone else can verify.
   - It can be reviewed without reading the other sub-specs.
   - Merging it does not leave the repo broken.
4. You confirm or adjust the split. **Nothing is written before this point.**
5. Writes `specs/NN.k-slug.md` per sub-spec (scope, steps with commit messages, acceptance criteria, how to verify, PR title and description, heads-up message for the reviewer) and adds a "Delivery in sub-specs" table to the parent spec.

## What it does not do

- It only writes `.md` files in `specs/`. It never writes code, creates branches, commits, pushes or opens PRs.
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
