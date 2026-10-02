---
name: spec-split
description: Split an approved spec into sub-specs where every sub-spec is one specific functionality delivered as one pull request, and every step of a sub-spec is one commit. Reads the spec and the project's conventions, proposes the split (sub-specs, steps, commit messages, PR titles, branches), asks for confirmation, then writes one `specs/NN.k-slug.md` per sub-spec and a delivery table in the parent spec. Never writes code, creates branches, commits or opens PRs. Use after `/spec` and before `/spec-impl`, when a spec is too big for a single PR, e.g. "/spec-split", "/spec-split 09", "/spec-split 09-part-management", "split this spec into PRs".
argument-hint: "[NN | NN-slug | path to spec]"
allowed-tools: Read, Glob, Grep, Write, Edit, AskUserQuestion, Bash(ls:*), Bash(cat:*), Bash(git log:*), Bash(git branch:*), Bash(git status:*), Bash(git diff:*)
---

# /spec-split — Split a spec into one PR per functionality

## Session context

Specs in this repo:
!`ls specs/ 2>/dev/null || echo "The specs/ folder does not exist"`

Recent commit style:
!`git log --oneline -15 2>/dev/null || echo "Not a git repository"`

Branches:
!`git branch -a 2>/dev/null`

Working tree at start (baseline for the repository guard):
!`git status --porcelain 2>/dev/null || echo "Not a git repository"`

---

You take a spec that is already well written and cut it into **specific deliverables**:

| Level | Rule |
| --- | --- |
| Spec (card) | It is an **objective**, not a delivery unit. It is split into specific, measurable functionalities. |
| Sub-spec | **One functionality.** It becomes one pull request. |
| Step | **One commit.** Steps are small, ordered and each one leaves the repo working. |
| Pull request | **Delivers one functionality.** One spec can (and usually does) produce several PRs. |

**You don't write code here.** You only write Markdown files in `specs/`, and only after the user confirms the split.

Argument received: `$ARGUMENTS` (spec number, slug or path; empty means "ask").

Talk to the user in the language they write in. Write the sub-specs in the **parent spec's language**, using its header labels and section names.

---

## Hard rules

1. **Split by functionality, end to end first.** Never by file, by hours, or by layer as a default. A piece is a functionality when it can be described in one sentence and has a result someone can verify. A layer split is allowed only when each piece justifies itself (Phase 3.1).
2. **Every sub-spec passes the seven validations** (Phase 3.2). If one fails, fix the split before showing it. Ticking boxes is not enough: each validation needs a one-line reason.
3. **One step = one commit.** Every step has exactly one commit message. No step says "and also…"; if it does, it is two steps.
4. **Never write before confirmation.** Phase 4 is the only phase that writes files, and only after the user approves the split in Phase 3.
5. **Only new sub-specs and the authorized parent sections.** You create `specs/NN.k-slug.md` files and edit the parent spec only where Phase 4.2 says. Never code, configuration, or any file outside `specs/`. Never `git checkout`, `git commit`, `git push`, `git stash`, `git reset`, `git restore`, nor `gh`. Branch names and PR titles are proposals written in the sub-specs; `/spec-impl` and the author act on them.
6. **Don't rewrite the parent spec.** You only add the delivery header line and the "Delivery in sub-specs" section. Its scope, decisions and criteria stay as they are.
7. **Don't invent scope.** Everything in a sub-spec comes from the parent spec. If the split reveals something the parent does not cover (a missing migration, a missing test setup), ask; don't add it silently.
8. **Every user decision is an `AskUserQuestion`.** Max 4 options; the recommended option goes first with ` (Recommended)` in its label.
9. **Never overwrite, never delete.** Before writing a sub-spec, check its path does not exist; if it does, stop and ask. Never remove or revert changes the user already made, committed or not.
10. **The repository guard is mandatory** (next section). If it finds anything unexpected, stop and ask; do not try to fix it.

---

## Repository guard

**Before starting** (right after locating the spec in Phase 1), read the baseline in the session context (`git status --porcelain`):

- If the **parent spec** has uncommitted changes, or any path you are going to create already appears there, stop. Show the lines and ask the user to commit or stash them first. Never stash or revert yourself.
- Other uncommitted changes (code, other specs) don't block you: list them to the user once ("these changes are yours and won't be touched") and leave them alone.
- If it is not a git repository, say the guard cannot verify anything and ask whether to continue.

**While writing** (Phase 4):

- Before each `Write`, check the target path does not exist (`ls specs/<file>`). If it does, stop and ask.
- The parent spec is the only existing file you edit, and only with `Edit` (never `Write`), only in the places Phase 4.2 allows.

**After writing**, run `git status --porcelain` and `git diff --stat` again and compare with the baseline:

- New entries must be exactly the sub-spec files you created (`??`) and the parent spec (` M`).
- `git diff -- <parent spec>` must contain only added lines: the header line and the "Delivery in sub-specs" section. No removed or changed lines.
- If anything else appears or changed (another file, a removed line, a file you did not create), stop: tell the user exactly what is different and ask how to proceed. Do not undo it yourself.

---

## Phase 1 — Locate the spec

If `$ARGUMENTS` is empty: list the specs from the session context and ask which one to split (use `AskUserQuestion` with the most recent approved specs as options). Stop until they answer.

If it has a value: find the file in `specs/` by full name, number (`09`) or slug (`part-management`). If several files share the number, ask which one. If none matches, show the list and ask.

Run the "Before starting" part of the repository guard. Then read the spec and check:

- **Status.** Continue only if it means "Approved" in any language (`Approved`, `Aprobado`, `aprobado`, `Aprovado`, …). If it is `Draft`/`In review`, say: "Approve the spec before splitting it: the split depends on its scope and decisions." and stop. If it is `Implemented`/`Obsolete`, say there is nothing to split and stop.
- **Already split.** If the spec says it is a parent spec, has a "Delivery in sub-specs" section, or `specs/NN.1-*` already exists, show the existing sub-specs and ask whether to stop (recommended) or re-plan only the sub-specs not yet implemented. Never overwrite an existing sub-spec file.

## Phase 2 — Learn the project and validate the spec

### 2.1 Conventions

Read, in this order, stopping when you have enough:

1. `CLAUDE.md`, then `AGENTS.md` (conventions, commands, test policy).
2. The two most recent specs, and any existing sub-specs (`NN.k-*.md`) — they are the best model for format, language and level of detail. Match them.
3. The session context above:
   - **Commit style.** If `git log` shows Conventional Commits (`type(scope): message`), use them with the scopes the repo already uses. Otherwise copy the dominant style. If there is no history, use Conventional Commits.
   - **Commit language.** The language `git log` uses, not the spec's.
   - **Branch naming and base.** The prefix the repo uses (`specNN/k-slug`, `spec-NN-slug`, `feat/…`). The base branch: `integration`, `develop` or `main`, whichever the recent feature branches target.
   - **Test commands.** From `package.json`/`CLAUDE.md` (`pnpm test:unit`, `npm test`, …), to write "how to verify".

Never run commands beyond those in `allowed-tools`. Never read secrets or `.env` files.

### 2.2 Validate the spec before splitting

An approved spec can still have holes that only show up when you try to cut it. Read the whole spec and look for:

| Problem | Example |
| --- | --- |
| **Ambiguous requirement** | "The list updates quickly" — on save, on an interval, in real time? |
| **Contradictory criteria** | One criterion says the field is optional, another rejects saving without it. |
| **Undefined dependency** | "Uses the pricing service" with no spec, file or API that defines it. |
| **Pending decision** | "TBD", "to be decided", "option A or B", an open question in the decisions section. |
| **Scope gap** | A criterion that needs a change the scope does not list (a migration, a permission, a test setup). |

For each one, decide whether it **affects the split** (it changes which sub-spec owns something, the order, a dependency between PRs, or whether a PR can be tested alone):

- **Affects the split:** ask before proposing anything, with `AskUserQuestion` (concrete options, your recommendation first). Record the answer; it goes into the affected sub-spec's scope or decisions, quoted as "Clarified during split: …".
- **Doesn't affect the split:** don't block. List it in the proposal under "Open points in the parent spec" and copy it into the affected sub-spec's risks, so `/spec-impl` sees it.

Never resolve an ambiguity by guessing. If the answers change the spec's scope, recommend updating the parent spec first (the user does it) and stop.

## Phase 3 — Propose the split

### 3.1 Find the functionalities

Go through the spec's scope, data model, implementation plan and acceptance criteria and group them into functionalities.

**Start from end-to-end slices.** The first question is: which behaviors can a user (or a calling system) observe, and can each one be delivered through every layer it needs in one PR? A slice like "update a single part from the edit screen" (UI + service + backend for that one behavior) is preferred over "all the UI", "all the services", "all the backend".

**Split by layer only when it is justified.** A layer piece (e.g. a backend capability before its UI) is acceptable only if all of these hold, and you write the reason in the proposal:

- It has its **own purpose** beyond "the next PR needs it" (e.g. a contract other teams consume, a migration that must run first, a risky change better reviewed alone).
- Its result is **verifiable** on its own (unit/service tests, a query, a script), even if users don't see it yet.
- It leaves the **app working**: nothing half-wired is reachable by users.

If a layer piece fails any of those, merge it into the slice that uses it.

Other useful questions:

- Which part is a **prerequisite** for others? (Shared rules, schema, helpers.) It goes first, must be additive, and must pass the conditions above.
- Which change **removes** or **switches** something? (Retiring old permissions, switching the UI to the new path.) It goes after the new path exists.
- What can be **left out** of the first PRs without breaking anything? Prefer additive PRs first, the switch-over last.

**Size is not a reason by itself.** Don't split because a PR "looks big". Judge each candidate PR by:

| Factor | Split when… |
| --- | --- |
| Behaviors | It delivers more than one behavior a reviewer must reason about separately. |
| Complexity | It mixes a hard change (concurrency, data migration, permissions) with routine ones. |
| Risk | A failure in one part would force reverting unrelated parts. |
| Review difficulty | The reviewer needs to hold several contexts at once to approve it. |
| Testability | Parts of it can't be tested until other parts exist. |

A large but uniform, single-behavior PR (e.g. the same rename across many files) stays as one.

If the spec is small enough for one PR, say so and propose a single sub-spec — or no split at all, and recommend running `/spec-impl` directly.

### 3.2 Validate every sub-spec

Each sub-spec must pass **all seven**, each with a one-line reason (not just a ✓):

1. **One sentence.** It is described in one sentence without "and"/"y".
2. **Verifiable result.** Someone else can verify it (a test, a screen, a query, a command). Name it.
3. **Reviewable alone.** It can be reviewed without reading the other sub-specs.
4. **Repo not broken.** Merging it leaves the repo coherent: nothing half-wired is reachable by users, and the checks that apply pass.
5. **Functional independence.** It delivers a useful result, or it only prepares code that nothing can use yet? If it only prepares, it must meet the layer-split conditions in 3.1; otherwise merge it into the slice that uses it.
6. **Justified dependencies.** Every dependency on another sub-spec is real and stated (3.4). If it can be integrated against the base branch, it must be.
7. **Scope integrity.** Every requirement, constraint, business rule and decision of the parent that belongs here is kept; nothing is dropped, softened or added. A behavior that is not in the parent is invented: remove it, or ask (hard rule 7).

If a proposal fails any validation, fix it (merge, re-split, reorder, re-target) and re-validate **before** asking for approval. Never show a failing proposal hoping the user catches it.

And every PR must avoid the "split it" signals:

| Good PR | Split it when |
| --- | --- |
| Understood by reading the title. | The title needs "and" or a list. |
| Touches one functionality. | It mixes several behaviors, or several layers that don't serve the same behavior. |
| Reviewed in one sitting. | The reviewer would need to schedule time to read it. |
| Testable on its own. | Testing it requires waiting for another PR. |

### 3.3 Break each sub-spec into steps

- Step 1 is always the sub-spec itself: `docs(spec): add sub-spec NN.k for <functionality>` (in the repo's commit style and language). The reviewer reads the plan first.
- Each following step is one small, verifiable change that compiles and passes the project's checks on its own. Its tests go **in the same commit** unless the project's conventions say otherwise.
- The commit message names the step it closes: `type(scope): <what this step does>`. No "WIP", "fixes", "more changes", no two actions joined by "and".
- Order the steps in the order the implementation is reasoned about (structure → behavior → presentation; model → logic → wiring → tests against real services).
- A step with no tests of its own says why (e.g. "the schema is tested in step 6").

### 3.4 Dependencies, shared contracts and base branches

**Default: independent.** Every PR targets the repo's base branch unless it has a real dependency.

**A dependency is real** only when the PR needs code from another PR that is not yet in the base branch to compile, to be tested, or to make sense. For each dependency, state:

- **What exactly** it needs (a file, a type, a function, a schema field, a permission).
- **Why** it cannot target the base branch (e.g. "imports `normalizeCode` from `amplify/shared/part.ts`, created in NN.1").

If you cannot name the exact thing, there is no dependency: target the base branch. Don't stack PRs only to keep them in order.

**Shared contracts.** Identify every change to a model, API, GraphQL/DB schema, permission or shared data structure that other sub-specs read or write. For each one, define a compatible sequence:

1. **Additive first:** add the new field/endpoint/permission without removing or changing the old one. Existing code keeps working.
2. **Consumers next:** the sub-specs that use the new contract.
3. **Removal last:** retire the old field/endpoint/permission in its own sub-spec, once nothing uses it.

A sub-spec that changes a contract in a breaking way while others still depend on the old shape is not allowed; re-order or split it into additive + removal.

**Base branch per PR:**

- Independent → the repo's base branch.
- Dependent → the branch of the PR it depends on (stacked). If it depends on two earlier PRs, target the latest one in the chain and say so.

**When an earlier stacked PR changes during review** (write this in every stacked sub-spec's Integration section):

- After the earlier PR gets new commits, the author updates the dependent branch on top of it (`git rebase <earlier-branch>` or merge, following the repo's convention) and re-runs that sub-spec's checks.
- If the change touches the contract the dependent PR uses, the author re-reads the dependent sub-spec and updates its steps before continuing; the reviewer of the dependent PR is told what changed.
- After the earlier PR is merged, the dependent PR is retargeted to the base branch.

### 3.5 Show the proposal


Show the whole split in one message, before asking anything:

```text
SPEC NN — <title>  →  n sub-specs / n PRs

NN.1 <functionality>                         PR 1 · branch specNN/1-slug ← base main
     Objective: <one sentence, no "and">
     Verify: <how someone else verifies it>
     1. Sub-spec                             docs(spec): add sub-spec NN.1 for …
     2. <step>                               feat(scope): …
     3. <step>                               test(scope): …

NN.2 <functionality>                         PR 2 · branch specNN/2-slug ← base specNN/1-slug
     ...

Validations: NN.1 7/7 · NN.2 7/7 · …  (one line of reason per validation)
Parent criteria covered: <criterion → NN.k> (every acceptance criterion of the parent maps to exactly one sub-spec)
```

Then a short note on any judgment call (why a piece went first, why two things stayed together).

**Base branch.** For every PR, show its base. For every stacked PR, show what it needs from the earlier PR and why (3.4). Note that stacked PRs must be merged in order.

**Parent criteria coverage.** Every acceptance criterion of the parent spec must land in exactly one sub-spec. If one is not covered, the split is incomplete: fix it before showing it.

Ask with `AskUserQuestion`:

- **Write the sub-specs (Recommended)**
- **Adjust the split** — then ask what to change (merge, split further, reorder, rename, move a step), apply it, re-run the checks and show the proposal again.
- **Cancel** — write nothing.

Repeat until the user picks "Write the sub-specs" or "Cancel".

## Phase 4 — Write the files

Only after confirmation. Follow the "While writing" part of the repository guard for every file.

### 4.1 Sub-spec files

Write `specs/NN.k-slug.md` for each sub-spec (kebab-case slug from its objective; match how existing sub-specs are named if there are any). Use the parent's language and labels. Shape:

```markdown
# SPEC NN.k — <Functionality title>

> **Status:** Draft
> **Parent spec:** [SPEC NN](NN-parent-slug.md)
> **PR:** k of n — branch `<branch>`, base `<base>`
> **Previous:** [SPEC NN.(k-1)](…) · **Next:** [SPEC NN.(k+1)](…)
> **Objective:** <one sentence, no "and">

---

## Scope

**In:**

- <exact items taken from the parent spec, with real file and symbol names>

**Out of scope:** <what the next sub-specs do; name them>. <What stays unchanged, e.g. "no file in src/ changes">.

---

## Steps

Every step is one commit, with its tests in the same commit. Every commit builds and passes `<project checks>`. If a step that was not planned appears during implementation, add it here and close it with its own commit.

1. **Sub-spec.** This file.
   - Commit: `docs(spec): add sub-spec NN.k for <functionality>`
2. **<Step title>.** <What changes, which files, which tests.>
   - Commit: `<type(scope): message>`
...

---

## Acceptance criteria

- [ ] <criteria taken from the parent spec, verifiable, boolean>

---

## How to verify

<Commands and manual checks someone else can run to confirm the result.>

---

## Pull request

**Title:** `<what functionality it delivers — no "and">`

**Description:**

> <One paragraph: what this PR delivers and what it does not (link to the next sub-spec).>
>
> **What was tested:** <tests added/run>.
> **How to verify:** <short steps>.
> Sub-spec: `specs/NN.k-slug.md` · Parent: SPEC NN · PR k of n<, base `<base>`: merge after PR k-1>.

**Heads-up to the reviewer (send before opening the PR):**

> <"I'm about to open a PR for <functionality>, I'll assign it to you." in the parent spec's language.>
```

Omit **Previous** / **Next** when they don't exist. Mark `Status` as `Draft` (or the repo's equivalent): the user approves each sub-spec before `/spec-impl`.

### 4.2 Parent spec

Edit the parent spec, without touching anything else:

1. Add a header line after the existing ones (same labels language): `> **Type:** parent spec. Delivered in n sub-specs, one per PR (see "Delivery in sub-specs").`
2. Add this section at the end (or before the risks section, if the existing parent specs in the repo do so):

```markdown
---

## Delivery in sub-specs

| Sub-spec | Functionality | PR | Branch | Base |
| --- | --- | --- | --- | --- |
| [NN.1](NN.1-slug.md) | <objective> | 1 of n | `<branch>` | `<base>` |
| … | … | … | … | … |

<"Stacked PRs: merge in order." when applicable.>
```

### 4.3 Repository guard after writing

Run the "After writing" part of the repository guard. If it fails, stop there and report; do not send the final message.

### 4.4 Final message

Tell the user, and stop:

- The files created and the parent spec edited, with the repository guard result ("only these files changed").
- The sub-specs are `Draft`: re-read and approve each one, then run `/spec-impl NN.1` (and so on, in order).
- If PRs are stacked: merge them in order and retarget the next PR's base after each merge.
- Reminder: send the heads-up to the reviewer before opening each PR.

Do not offer to implement, create branches or open PRs.

---

## Quick reference

- Is the spec deliverable in one go? If not, split it by functionality.
- Does every sub-spec have one specific, measurable purpose (one sentence, no "and")?
- Is the sub-spec written before implementing (step 1)?
- Does every commit close one step?
- Does every PR deliver one functionality?
- Is every acceptance criterion of the parent covered by exactly one sub-spec?
