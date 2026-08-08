# AGENTS.md Snippets

Copy-paste conventions from my `AGENTS.md` / `CLAUDE.md` files, as shown in my videos on coding with AI agents:

- [The 4 Levels of Coding With AI](https://youtu.be/Z30YKmBEXf4)
- The live multi-agent session (link coming when it's published)

These work in any agent instruction file — `AGENTS.md`, `CLAUDE.md`, or equivalent. Adjust the wording to your own setup; the ideas are what matter.

---

## 1. Context re-entry (the context-switching fix)

When you're juggling several agents at once, every summary an agent hands you is a cold start for *you*. This block makes agents write for that. It lives in my root instruction file, shared across all repositories.

```markdown
## Context re-entry (multi-project juggling)

I am juggling several projects, each with several concurrent sessions, and have usually lost
the thread by the time I return to any one of them. Write every user-facing message for cold
re-entry — assume I remember nothing from the scrollback:

- **Open with a recap.** Before any summary, decision point, or question: 2–3 plain sentences on
  what we were just working on, why, and where it stands now.
- **Plain language.** No invented codenames, abbreviations, or callbacks like "the earlier fix"
  or "option B from before" — restate the thing in place, every time.
- **Self-contained questions.** When asking me to decide something, the question itself
  must carry everything needed to answer it: the background, the options, the tradeoffs, and
  your recommendation. Never require scrolling back.
- **One question at a time.** When a summary or decision point holds several open questions or
  next steps, say so up front ("three decisions are waiting; here's the first"), then present
  only the first and wait for the answer before raising the next. Never dump them all at once —
  it's too much mental load.
- **Anchor the work.** Name the project, branch, and PR when reporting status — several other
  sessions look just like this one.
- **End with the next action.** Close long updates with the single thing waiting on me,
  or say explicitly that nothing is.
```

## 2. Git worktrees for parallel agents

```markdown
## Worktrees

When starting any new feature or fix, begin by creating a separate git worktree from the base
branch and do all work inside it, so parallel agents never overwrite each other's changes.
After the work is merged, clean up by removing the worktree. For the next task, create a fresh
worktree from the latest base branch — don't reuse old trees.
```

## 3. TDD is mandatory

```markdown
## TDD is mandatory

Every change follows **failing test first → implement → verify**:
1. Write the test(s) that capture the desired behavior and watch them **fail** (red).
2. Implement the minimum to make them pass.
3. Run the suite + typecheck and confirm green.

Don't write implementation before a failing test exists. When fixing a bug, reproduce it with a
failing test first.

## Verify before claiming "done"

Never report something as working without running it. "Done" means: relevant tests green,
typecheck clean, and — for user-facing flows — exercised end to end (e.g. Playwright for web
flows). If tests fail or a step was skipped, say so plainly with the output.
```

## 4. Builder/driver split for the no-mistakes gate (the token optimization)

This is the optimization from the live session: the agent that *built* a feature is sitting on a huge context (often 150k–200k tokens). If that same agent drives the review gate — which takes many turns of monitoring and simple responses — that entire context is resent on every turn. Instead, the builder hands off, and a fresh, cheap, tiny-context agent drives the gate. A park→decide→resume roundtrip costs ~30k tokens on a driver vs ~200k on a builder.

Adapt to your setup; mine targets the [no-mistakes gate](https://github.com/kunchenguid/no-mistakes).

```markdown
## Orchestrating the gate (builder/driver split)

- **Builders never drive the gate.** A builder agent builds, commits on its branch, and ends its
  task with a `HANDOFF: INTENT` paragraph — a thorough statement of what changed and why, for
  the reviewer. Its large transcript is read once and never resumed for gate-driving.
- **A fresh tiny driver agent per worktree** (cheap model, few-k-token context) runs the gate:
  it starts the review with the handed-off intent, monitors progress, and answers the gate's
  questions.
- **Gate rules for the driver:** apply auto-fixable findings; approve info-only findings; for
  anything that needs a human decision, PARK — quote the finding verbatim and end the task so
  the orchestrator can relay it to me, then resume the driver with my decision. Resume a
  builder only when a finding needs real code fixes.
- Never end a subagent's turn while a gate run is active — its background processes are
  orphaned the moment the turn ends.
```

**Bonus — cross-provider review:** having a different provider review than the one that built (e.g. Claude Code implements, Codex reviews, or vice versa) catches a different distribution of bugs, and spreads the token load across two subscriptions.

---

## Tools mentioned in the videos

- [no-mistakes gate](https://github.com/kunchenguid/no-mistakes) by Kun Chen — automated review, tests, and screenshot evidence before code reaches you
- [Grill Me skill](https://www.aihero.dev/skills-grill-me) by Matt Pocock — makes the agent interview you through every design decision, one question at a time
- [Wispr Flow](https://wisprflow.ai) — speech-to-text; most of my agent prompts are dictated

## More

- 🚀 [Altitude](https://learnaltitude.com) — my app that teaches you to code by building a real project
- 💬 [The Workshop](https://jasonk.us/workshop) — my free community; I answer every question
