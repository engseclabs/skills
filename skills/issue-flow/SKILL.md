---
name: issue-flow
description: >-
  End-to-end workflow for turning an idea into shipped code via a GitHub issue,
  an optional design doc (as the first PR), and stacked implementation PRs, while
  keeping the issue updated with status. Starts with a Socratic idea-vetting pass
  (clarifying questions, alternatives, a proposed design) that you can skip for
  small/obvious work. Use when the user wants to start a new feature, epic, or
  non-trivial change and mentions an issue, design doc, spec, PR flow, or "let's
  plan this out." Also triggers on "grill me on this idea", "vet this", "help me
  scope this", or "start a new issue for X".
---

# Issue Flow

A lightweight, GitHub-native workflow: **vet, issue, (design doc PR), stacked PRs, keep the issue updated.**

Adapted from the `obra/superpowers` brainstorming pattern and issue-lifecycle patterns, tailored to my conventions: plain `git` and `gh`, `gh stack` for stacked PRs, design docs in `docs/design/`, the issue as the single source of truth.

## Guiding principles

- **The GitHub issue is the tracking anchor.** No local progress files. Status and the PR checklist live on the issue. When a design doc exists, the doc owns the detail (see the next principle); with no doc, the issue also carries the decisions.
- **Issue and design doc have distinct jobs. Don't duplicate.** The **design doc** is the complete build spec: detailed enough to implement everything from, the single source of truth for acceptance criteria and decisions. The **issue** is my dashboard: a short summary, the approach in brief, a link to the doc, and the PR-checklist progress tracker. It is not a second copy of the criteria. If the same acceptance criteria are landing in both, cut them from the issue and point at the doc.
- **Vet before you build, but don't gate trivial work.** The idea-vetting pass is Socratic and skippable, not a hard blocker. Recommend it and let me opt out.
- **Cheapest review first.** Review the approach (design doc PR) before any implementation exists. Review each implementation slice independently (stacked PRs).
- **Stack only when work genuinely depends on prior work.** Independent slices get independent PRs off `main`. Use [`gh stack`](https://github.com/github/gh-stack) for anything stacked. It automates branch creation, rebasing, and PR base-chaining, so stacking no longer carries the manual rebase/retarget risk it used to.
- **Write issue, PR, and doc bodies newspaper-style, for the reader.** Lead with the summary: the what and why in the first paragraph, then details in descending order of importance, so a reviewer who stops after the lede still has the gist. Write for whoever reads the body, not for yourself. Leave out your own working context: the originating prompt, how the task was framed, "the brief said X," what you corrected along the way. The reader cares what the code should do, not how you got here. State current behavior and decisions as plain facts. This applies to issue bodies, PR descriptions, comments, and checked-in design docs.
- **Never hard-wrap prose in issue, PR, or comment bodies. One line per paragraph or list item.** GitHub renders Markdown and reflows paragraphs to the reader's viewport on its own. A newline inserted mid-paragraph (e.g. wrapping at ~80 columns like source code) becomes a forced line break in the rendered output: ragged, mid-sentence, and wrong on every screen width. Write each paragraph as one unbroken line, each bullet as one line, and let the browser wrap. Press Enter only to start a genuinely new block: a new paragraph (blank line between), a new list item, or a heading. This is the opposite of the hard-wrapping convention for commit messages and checked-in `.md` files. The distinction is the renderer: GitHub comment, issue, and PR fields reflow; a file viewed as raw text does not. When you build a body with a heredoc, do not wrap it to terminal width; let each paragraph run long.
- **Follow the `ai-writing-improver` skill's style rules** for every issue, comment, PR description, and design doc. It governs content and tone; the two principles above govern structure and Markdown line breaks.

---

## Phase 0: Vet the idea (Socratic, skippable)

**Offer this first.** Say something like: *"Want me to run a quick vetting pass before we cut the issue: clarifying questions, a couple of alternatives, a proposed shape? Or is this well-scoped enough to skip straight to the issue?"*

If I skip, go to Phase 1. If the work is a one-line fix or an obvious change, proactively suggest skipping.

If I want it (or the work is architectural, spans components, or has real decisions to record), run the pass:

1. **Explore context.** Read relevant existing files, recent commits, and any related design docs in `docs/design/`. Ground the conversation in what's there.
2. **Ask clarifying questions, one at a time.** Prefer multiple choice. Focus on understanding, not pitching solutions. Cover:
   - What problem are you actually solving? (the real goal behind the ask)
   - What are the constraints? (compatibility, data, deadlines, blast radius)
   - How will you know it worked? (success or acceptance criteria)
   - What's explicitly out of scope?
3. **Grill the assumptions.** Surface implicit assumptions, dependency chains between decisions, and gaps. Name the risky one. Push back where the plan is thin. That is the "grill me" value, not a rubber stamp.
4. **Propose 2 to 3 approaches** with trade-offs and a recommendation.
5. **Present the design in digestible sections** (scale length to complexity), asking for approval as you go. Cover as relevant: architecture, components, data flow, error handling, testing, rollout.

Phase 0 produces a shared understanding and, if warranted, the raw material for a design doc. Get a clear "yes, this shape is right" before moving on.

---

## Phase 1: Create the GitHub issue (the anchor)

Create the tracking issue **up front**. Everything references it.

```bash
gh issue create --title "<concise title>" --label enhancement --body "$(cat <<'EOF'
## Problem
<what's wrong / missing and why it matters>

## Goal
<the desired end state>

## Acceptance criteria
- [ ] <verifiable outcome>
- [ ] <verifiable outcome>

## Plan (PR checklist)
- [ ] Design doc: `docs/design/<topic>.md`   (only if warranted)
- [ ] <impl slice 1>
- [ ] <impl slice 2>

## Out of scope
- <thing we are deliberately not doing>
EOF
)"
```

- Pick a label that exists in the repo (`gh label list`). This repo has `bug`, `documentation`, `enhancement`, `question`, and others.
- The **Plan checklist becomes the live status tracker.** Each box is a PR.
- Capture the Phase 0 decisions here so they survive the conversation.

Note the issue number returned; call it `#N` below.

---

## Phase 2: Design doc as the first PR (only if warranted)

Skip this for small, localized changes. Add a design doc when the work is architecturally non-trivial, spans components, or has decisions worth recording. This repo already treats `docs/design/` this way (see `docs/design/identity-first-sessions.md`).

**Convention for this repo:** one markdown file per design under `docs/design/`, with a top-of-file link back to the tracking issue:

```markdown
# <Title>

For status and rollout see https://github.com/<owner>/<repo>/issues/N

## 1. Summary
...
```

Ship it as its own PR so the *approach* gets reviewed before code exists:

```bash
git switch -c design/<topic>
# write docs/design/<topic>.md
git add docs/design/<topic>.md
git commit -m "Design doc: <topic>

Part of #N"
git push -u origin design/<topic>
gh pr create --base main --title "Design: <topic>" --body "Design doc for #N. Part of #N."
```

After it merges, check the design-doc box on the issue (Phase 4).

The `doc-coauthoring` skill is a good companion for drafting the doc itself.

---

## Phase 3: Implementation PRs (stacked only when needed)

**Decide: stack or independent?**
- Slices are *independent* (touch different areas, no ordering): separate branches off `main`, separate PRs. Simpler, with nothing to keep in sync.
- Slice B genuinely builds on unmerged slice A: **stack** B on A, using [GitHub stacked pull requests](https://github.blog/changelog/2026-07-30-stacked-pull-requests-are-now-in-public-preview/) and the [`gh stack`](https://github.com/github/gh-stack) CLI extension. Install once with `gh extension install github/gh-stack`.

### Independent PR (default)
```bash
git switch main && git pull
git switch -c feat/<slice>
# ...work, commit...
gh pr create --base main --title "<slice>" --body "Part of #N."
```

### Stacked PR
`gh stack` owns branch creation, base-branch chaining, and rebasing. It
replaces manually creating branches off each other and retargeting PRs by hand.

```bash
gh stack init feat/slice-a           # first layer, based on main
# ...commit slice A...
gh stack add feat/slice-b            # next layer, based on slice A
# ...commit slice B...
gh stack submit                      # push branches, open one PR per layer, link them as a Stack
```

Each PR's base is set automatically to the branch below it, so reviewers see only that layer's diff. A **stack map** on each PR shows how it fits into the whole change. `gh stack view` shows the current stack and its PR links.

**When a lower layer needs changes:** fix it, then `gh stack rebase` cascades the fix through every branch above it, and `gh stack push` updates all the open PRs at once. GitHub retargets and rebases the upper PRs automatically when a lower one merges. No manual base-branch surgery, and no risk of a PR silently merging into an already-merged, now-dead parent branch (the failure mode the old manual-stacking flow was prone to).

**Merging:** `gh stack merge` lands one or more layers bottom-up in a single all-or-nothing operation, respecting existing branch protections and required checks. Merge the whole stack at once, or pass a PR number to merge only up to that layer and leave the rest open.

A 2-layer stack is still worth it, since `gh stack` removes the rebase/retarget overhead that used to make manual stacking risky. For very small, tightly coupled changes, one PR with well-separated commits is still simpler than any stack.

### PR body linking convention
- Intermediate PRs: **`Part of #N`** (keeps the issue open).
- The final PR that completes the issue: **`Closes #N`** (auto-closes on merge).

---

## Phase 3.5: Review before opening the PR

Review happens at **four** points, each catching something different. Don't collapse them:

| Gate | When | Catches |
|------|------|---------|
| 1. Design review | the design-doc PR (Phase 2) | wrong *approach*, before code exists |
| 2. Self-review | before opening each impl PR | scope creep, missed acceptance criteria |
| 3. Independent review | on the diff | what the implementer rationalized past |
| 4. CI | every PR push | test/build/lint failures (see Phase 3.6) |

**Gate 3 is the high-value one, and the key is independence.** The model that wrote the code is a poor reviewer of its own code: it defends the choices it already made. Run a fresh review pass; don't just re-read your own diff.

**Before opening each implementation PR, run `/code-review` on the working diff:**

- Run `/code-review` and surface the findings to me.
- Triage: fix real defects now; note anything deferred as a `Part of #N` follow-up or a new issue. Don't silently drop findings.
- Re-run after applying fixes if the changes were substantial.

For a meaty change I can opt into `/code-review ultra`, a multi-agent cloud review that is more thorough than a single pass. It is **user-triggered and billed**. Recommend it, but never attempt to launch it yourself.

Only open the PR once gates 2 and 3 are clean (or findings are consciously deferred).

---

## Phase 3.6: Let CI run (gate 4)

This repo runs vet, test, and build on every PR (`.github/workflows/ci.yaml`). After pushing, confirm CI is green before merging:

```bash
gh pr checks --watch
```

If CI fails, fix it before requesting review or merging. A red PR is not ready.

> [!IMPORTANT]
> **A green check does not always mean the whole repo passed. Confirm what CI actually runs.** This repo is **multiple Go modules**: the root module and `ingestor/` (its own `go.mod`). `go test ./...` from the root does **not** descend into `ingestor/`. CI must run the ingestor's tests explicitly (`working-directory: ingestor`), or an entire module's tests go silently unchecked and a regression there merges green. If you change a module, verify CI covers it. Locally, test **every** module you touched: `go test ./...` **and** `go -C ingestor test ./...`.

---

## Phase 4: Keep the issue updated (as work happens, not after)

Update the issue continuously. It's the status dashboard.

- **On merge**, check the box for that slice:
  ```bash
  gh issue edit N --body "$(gh issue view N --json body -q .body | sed 's/- \[ \] <slice 1>/- [x] <slice 1>/')"
  ```
  (Or edit the body directly. The point is that the checklist reflects reality.)
- **On meaningful progress, blockers, or scope changes**, drop a comment:
  ```bash
  gh issue comment N --body "Design doc merged (#<pr>). Starting slice 1."
  ```
- **The final PR's `Closes #N`** closes the issue on merge. If the issue was long-running, leave a closing comment summarizing what shipped.

---

## Quick reference

| Step | Command |
|------|---------|
| Create issue | `gh issue create --title … --label enhancement --body …` |
| Design PR | `git switch -c design/<t>` → PR `--base main` |
| Impl PR (indep.) | `git switch -c feat/<s> main` → PR `--base main` |
| Impl PR (stacked) | `gh stack init feat/a` → `gh stack add feat/b` → `gh stack submit` |
| Fix a lower layer | edit, commit, `gh stack rebase`, `gh stack push` |
| Merge a stack | `gh stack merge` (bottom-up, all-or-nothing; retargets upper PRs automatically) |
| Review diff (gate 3) | `/code-review` (or `/code-review ultra`, user-triggered) |
| Watch CI (gate 4) | `gh pr checks --watch` |
| Update issue box | `gh issue edit N --body …` |
| Comment status | `gh issue comment N --body …` |
| Link (intermediate) | `Part of #N` in PR body |
| Link (final) | `Closes #N` in PR body |
