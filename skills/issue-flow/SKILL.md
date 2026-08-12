---
name: issue-flow
description: >-
  End-to-end workflow for turning an idea into shipped code via a GitHub issue,
  an optional design doc (as the first PR), and stacked implementation PRs — while
  keeping the issue updated with status. Starts with a Socratic idea-vetting pass
  (clarifying questions, alternatives, a proposed design) that you can skip for
  small/obvious work. Use when the user wants to start a new feature, epic, or
  non-trivial change and mentions an issue, design doc, spec, PR flow, or "let's
  plan this out." Also triggers on "grill me on this idea", "vet this", "help me
  scope this", or "start a new issue for X".
---

# Issue Flow

A lightweight, GitHub-native workflow: **vet → issue → (design doc PR) → stacked PRs → keep the issue updated.**

Adapted from the `obra/superpowers` brainstorming pattern and issue-lifecycle
patterns, tailored to this user's conventions (plain `git` + `gh`, design docs in
`docs/design/`, issue as the single source of truth).

## Guiding principles

- **The GitHub issue is the tracking anchor.** No local progress files. Status
  and the PR checklist live on the issue. When a design doc exists, the *doc* is
  the source of truth for detail (see the division-of-labor principle below);
  when there's no doc, the issue also carries the decisions.
- **Issue and design doc have distinct jobs — don't duplicate.** The **design
  doc** is the complete build spec: detailed enough to implement everything from,
  the single source of truth for acceptance criteria and decisions. The **issue**
  is the user's dashboard: a high-level summary, the approach in brief, a link to
  the doc, and the PR-checklist progress tracker — *not* a second copy of the
  criteria. If you find yourself pasting the same acceptance criteria into both,
  cut it from the issue and point at the doc.
- **Vet before you build — but don't gate trivial work.** The idea-vetting pass is
  Socratic and *skippable*, not a hard blocker. Recommend it; let the user opt out.
- **Cheapest review first.** Review the *approach* (design doc PR) before any
  implementation exists. Review each implementation slice independently (stacked PRs).
- **Stack only when work genuinely depends on prior work.** Independent slices get
  independent PRs off `main`. Stacking has real rebase cost — earn it.
- **Write issue/PR/doc bodies newspaper-style, for the reader.** Lead with a big
  top-down summary — the *what* and *why* in the first paragraph — then details in
  descending order of importance, so a reviewer who stops after the lede still has
  the gist. Write for whoever reads the issue/PR/doc, not for yourself: leave out
  your own working context — the originating prompt, how the task was framed to
  you, "the brief said X," what you had to correct along the way. The reader cares
  what the code should do, not how you got here. State current behavior and
  decisions as plain facts. This applies to issue bodies, PR descriptions,
  comments, **and** checked-in design docs.
- **Never hard-wrap prose in issue/PR/comment bodies — one line per paragraph or
  list item.** GitHub renders Markdown: it reflows paragraphs to the reader's
  viewport on its own. A newline you insert mid-paragraph (e.g. wrapping at ~80
  columns like source code) becomes a *forced* line break in the rendered output —
  ragged, mid-sentence, and wrong on every screen width. So write each paragraph
  as a single unbroken line, and each bullet as a single line, letting the browser
  wrap. Only ever press Enter to start a genuinely new block: a new paragraph
  (blank line between), a new list item, or a heading. This is the opposite of the
  hard-wrapping convention for commit messages and checked-in `.md` files — the
  distinction is the renderer: GitHub comment/issue/PR fields reflow, a file
  viewed as raw text does not. When you build a body with a heredoc, do **not**
  wrap it to terminal width; let each paragraph run long.
- **Follow the writing-style rules below** for every issue, comment, PR
  description, and design doc.

## Writing style (issues, PRs, comments, design docs)

Apply these rules to all prose you write for the reader. They govern content and
tone; the two principles above govern structure and Markdown line breaks.

- Use an inverted-pyramid structure. Lead with the main conclusion, outcome, or
  recommendation, then supporting information in descending order of importance.
- Optimize for information density. Every sentence should have a purpose. Stop
  when the useful information is complete.
- Use plain, direct language, active voice, and concrete verbs. Prefer precise
  verbs to adverb-modified weak verbs.
- Assume a technically literate reader.
- Be concise without becoming cryptic. Use short paragraphs and vary sentence
  length naturally.
- Use semantic Markdown: heading levels for hierarchy, lists for actual sets or
  sequences, tables for comparisons or mappings, blockquotes for quotations, code
  formatting for identifiers and examples.
- Use inline links to internal and external references with concise, descriptive
  link text. Link to source material instead of duplicating it, while retaining
  enough context to understand the point.
- Never use em dashes. Use periods, commas, parentheses, or colons. Hyphens in
  compound words are fine.
- Never use plus signs or ampersands for "and" in prose; write "and". (Code
  identifiers and literal command syntax keep their real characters.)
- Avoid stock AI prose: canned openings, filler transitions, rhetorical
  questions, fake contrasts such as "not X, but Y," and inflated words such as
  "delve," "pivotal," "transformative," "seamless," or "revolutionary."

---

## Phase 0 — Vet the idea (Socratic, skippable)

**Offer this first.** Say something like: *"Want me to run a quick vetting pass
before we cut the issue — clarifying questions, a couple of alternatives, a
proposed shape? Or is this well-scoped enough to skip straight to the issue?"*

If the user skips, go to Phase 1. If the work is a one-line fix or an obvious
change, proactively suggest skipping.

If they want it (or the work is architectural / multi-component / has real
decisions to record), run the pass:

1. **Explore context.** Read relevant existing files, recent commits, and any
   related design docs in `docs/design/`. Ground the conversation in what's there.
2. **Ask clarifying questions — one at a time.** Prefer multiple choice. Focus on
   *understanding*, not pitching solutions. Cover:
   - What problem are you actually solving? (the real goal behind the ask)
   - What are the constraints? (compat, data, deadlines, blast radius)
   - How will you know it worked? (success criteria / acceptance criteria)
   - What's explicitly out of scope?
3. **Grill the assumptions.** Surface implicit assumptions, dependency chains
   between decisions, and gaps. Name the risky one. Push back where the plan is
   thin — this is the "grill me" value, not a rubber stamp.
4. **Propose 2–3 approaches** with trade-offs and a recommendation.
5. **Present the design in digestible sections** (scale length to complexity),
   asking for approval as you go. Cover as relevant: architecture, components,
   data flow, error handling, testing, rollout.

The output of Phase 0 is a shared understanding and (if warranted) the raw
material for a design doc. Get a clear "yes, this shape is right" before moving on.

---

## Phase 1 — Create the GitHub issue (the anchor)

Create the tracking issue **up front** — everything references it.

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
- [ ] Design doc — `docs/design/<topic>.md`   ← only if warranted
- [ ] <impl slice 1>
- [ ] <impl slice 2>

## Out of scope
- <thing we are deliberately not doing>
EOF
)"
```

- Pick a label that exists in the repo (`gh label list`). This repo has:
  `bug`, `documentation`, `enhancement`, `question`, etc.
- The **Plan checklist becomes the live status tracker** — each box is a PR.
- Capture the Phase 0 decisions here so they survive the conversation.

Note the issue number returned; call it `#N` below.

---

## Phase 2 — Design doc as the first PR (only if warranted)

Skip this for small/localized changes. Add a design doc when the work is
architecturally non-trivial, spans components, or has decisions worth recording.
This repo already treats `docs/design/` this way (see
`docs/design/identity-first-sessions.md`).

**Convention for this repo:** one markdown file per design under `docs/design/`,
with a top-of-file link back to the tracking issue:

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

## Phase 3 — Implementation PRs (stacked only when needed)

**Decide: stack or independent?**
- Slices are *independent* (touch different areas, no ordering) → separate branches
  off `main`, separate PRs. Simpler; no restacking.
- Slice B genuinely builds on unmerged slice A → **stack** B on A.

### Independent PR (default)
```bash
git switch main && git pull
git switch -c feat/<slice>
# ...work, commit...
gh pr create --base main --title "<slice>" --body "Part of #N."
```

### Stacked PR
Base each branch on the previous branch, and target the PR at the parent branch —
**not** `main` — so the diff shows only that slice:

```bash
git switch -c feat/slice-a main        # slice A off main
# ...commit, push...
gh pr create --base main --title "Slice A" --body "Part of #N."

git switch -c feat/slice-b feat/slice-a  # slice B off slice A
# ...commit, push...
gh pr create --base feat/slice-b --title "Slice B" --body "Part of #N. Stacked on Slice A."
```

> [!WARNING]
> **The #1 way stacked PRs lose commits: merging the child while its base is still the parent branch.**
> When A merges, GitHub does **not** reliably auto-retarget B to `main` — the auto-retarget only fires if A's branch is *deleted* on merge, and even then it's unreliable after a squash/rebase merge. If B still has `base = feat/slice-a` when you click merge, **B merges into `feat/slice-a` (a now-dead branch), not `main` — and its commits never reach `main`.** They look "merged" (PR shows purple/MERGED) but are absent from `main`. This exact failure cost us a re-do (see the recovery recipe below).

**Stack hygiene — retarget is a HARD GATE, not a cleanup step:**

1. **Merge bottom-up** (A before B), one at a time.
2. **After A merges, BEFORE touching B, retarget + rebase B onto `main`:**
   ```bash
   gh pr edit <B> --base main                                   # retarget the PR first
   git switch feat/slice-b
   git rebase --onto main feat/slice-a feat/slice-b
   git push --force-with-lease
   ```
3. **Verify B's base is actually `main` before merging it** — never merge on faith:
   ```bash
   gh pr view <B> --json baseRefName -q .baseRefName            # MUST print "main"
   ```
   If this does not print `main`, do **not** merge — retarget (step 2) first.
4. If the base moves, restack the whole chain in order (A→B→C), repeating steps 2–3 per level.

**Recovery — a child already merged into the dead parent branch (commits missing from `main`):**
The commits still exist on the child's head branch. Recreate them onto `main` as a fresh PR:
```bash
git switch -c <slice>-to-main origin/main
git cherry-pick <first>^..<last>          # the child's own commits (git log main..origin/feat/slice-b)
git push -u origin <slice>-to-main
gh pr create --base main --title "<slice> (re-target to main)" --body "Content of #<B>, re-targeted to main. Supersedes #<B>. Part of #N."
```

**Simpler alternative — avoid the trap entirely:** for a 2-PR stack, once A merges you can just retarget B to `main` and rebase (steps 2–3). For deeper stacks or when retargeting keeps going wrong, prefer **one PR with well-separated commits** over a stack — reviewers can read commit-by-commit, and there's no base to lose.

### PR body linking convention
- Intermediate PRs: **`Part of #N`** (keeps the issue open).
- The final PR that completes the issue: **`Closes #N`** (auto-closes on merge).

---

## Phase 3.5 — Review before opening the PR

Review happens at **four** points, catching different things — don't collapse them:

| Gate | When | Catches |
|------|------|---------|
| 1. Design review | the design-doc PR (Phase 2) | wrong *approach*, before code exists |
| 2. Self-review | before opening each impl PR | scope creep, missed acceptance criteria |
| 3. Independent review | on the diff | what the implementer rationalized past |
| 4. CI | every PR push | test/build/lint failures (see Phase 3.6) |

**Gate 3 is the high-value one, and the key is independence:** the model that
wrote the code is a poor reviewer of its own code — it defends the choices it
already made. So run a *fresh* review pass, don't just re-read your own diff.

**Before opening each implementation PR, run `/code-review` on the working diff:**

- Run `/code-review` and surface the findings to the user.
- Triage: fix real defects now; note anything deferred as `Part of #N` follow-up
  or a new issue. Don't silently drop findings.
- Re-run after applying fixes if the changes were substantial.

For a meaty change the user can opt into `/code-review ultra` — a multi-agent
cloud review, more thorough than a single pass. It is **user-triggered and
billed**; recommend it, but never attempt to launch it yourself.

Only open the PR once gate 2 + 3 are clean (or findings are consciously deferred).

---

## Phase 3.6 — Let CI run (gate 4)

This repo runs vet/test/build on every PR (`.github/workflows/ci.yaml`). After
pushing, confirm CI is green before merging:

```bash
gh pr checks --watch
```

If CI fails, fix it before requesting review or merging — a red PR is not ready.

> [!IMPORTANT]
> **A green check does not always mean the whole repo passed — confirm what CI actually runs.** This repo is **multiple Go modules**: the root module and `ingestor/` (its own `go.mod`). `go test ./...` from the root does **not** descend into `ingestor/`. CI must run the ingestor's tests explicitly (`working-directory: ingestor`), or an entire module's tests are silently unchecked and a regression there merges green. If you change a module, verify CI covers it — and locally, test **every** module you touched: `go test ./...` **and** `go -C ingestor test ./...`.

---

## Phase 4 — Keep the issue updated (as work happens, not after)

Update the issue continuously — it's the status dashboard.

- **On merge**, check the box for that slice:
  ```bash
  gh issue edit N --body "$(gh issue view N --json body -q .body | sed 's/- \[ \] <slice 1>/- [x] <slice 1>/')"
  ```
  (Or just edit the body directly — the point is the checklist reflects reality.)
- **On meaningful progress, blockers, or scope changes**, drop a comment:
  ```bash
  gh issue comment N --body "Design doc merged (#<pr>). Starting slice 1."
  ```
- **The final PR's `Closes #N`** closes the issue on merge — but leave a closing
  comment summarizing what shipped if the issue was long-running.

---

## Quick reference

| Step | Command |
|------|---------|
| Create issue | `gh issue create --title … --label enhancement --body …` |
| Design PR | `git switch -c design/<t>` → PR `--base main` |
| Impl PR (indep.) | `git switch -c feat/<s> main` → PR `--base main` |
| Impl PR (stacked) | `git switch -c feat/b feat/a` → PR `--base feat/a` |
| Review diff (gate 3) | `/code-review` (or `/code-review ultra` — user-triggered) |
| Watch CI (gate 4) | `gh pr checks --watch` |
| Restack after A merges | `gh pr edit <B> --base main` + `git rebase --onto main feat/a feat/b` + `git push --force-with-lease` |
| **Verify base before merging a child** | `gh pr view <B> --json baseRefName -q .baseRefName` → **must be `main`** |
| Update issue box | `gh issue edit N --body …` |
| Comment status | `gh issue comment N --body …` |
| Link (intermediate) | `Part of #N` in PR body |
| Link (final) | `Closes #N` in PR body |
