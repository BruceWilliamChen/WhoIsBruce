---
title: "Claude Has Amnesia. Here's the System I Built to Fix It."
date: 2026-04-28
description: "Claude's default memory is flat and sessions compress context. Here's the logging system I built on top to make long-term human-AI collaboration actually work."
---

I've been using Claude Code heavily for the past few months, and memory was the first thing that broke on me. Not in a dramatic way — just a slow friction. Close a session, open a new one, and spend the first ten minutes reconstructing context that should already be there. The longer a project ran, the worse it got.

So I started tinkering. First just dumping notes into CLAUDE.md. Then organizing them. Then realizing the notes were getting messy and unstructured. Each time I hit a wall I'd rethink the approach — always with the same question in mind: how do I make this feel less like a one-shot tool and more like a real long-term collaborator?

Part of what clicked was thinking about git. Git gives you a perfect record of what changed — every commit, every diff, every file touched. But it tells you nothing about the conversation that led to it. The why is always missing. The decisions, the dead ends, the "we tried this and it didn't work" — gone the moment the session closes. I wanted the equivalent of that reasoning layer, but for AI sessions.

So I kept refining. What you're reading is the current state of that system — the version that actually worked.

## The problem with default memory

Claude does have memory — two kinds, actually.

The first is `CLAUDE.md`, a markdown file that gets loaded at the start of every session. It's great for preferences and workflow rules: "use TypeScript", "ask before writing files", "don't add comments unless the why is non-obvious". Think of it as a standing brief you hand Claude at the door.

The second is the built-in `/memory` command, which lets Claude save notes to a persistent file mid-session.

Both are useful. Both have the same fundamental limitation: they're flat. No time dimension, no project scope, no structure. Everything piles into the same file. You can't ask "what's the current state of the auth feature?" and get a real answer. You can't trace a decision back to why it was made. The longer you work on something, the more the file becomes noise.

There's a second problem that compounds this: context compression. Claude Code runs inside a context window, and when that window fills up, earlier messages get automatically compressed into summaries. Details get lost. Decisions made early in a long session get flattened. If you're deep in a complex feature and the window is nearly full, you're not working with the same fidelity you started with.

The practical response is to switch sessions proactively — before compression degrades quality. I personally do it around 60% context usage, but that's just my own call. The point is: switching sessions is good hygiene, not a last resort. And if you're going to do it regularly, you need a structured handoff.

For a one-off task, the defaults are probably fine. But if you're treating Claude as a real long-term collaborator — carrying a project across weeks, building on past decisions, maintaining continuity across sessions — the cracks show fast. Human-AI collaboration only stays consistent if both sides can pick up the thread. The human remembers; Claude needs a system to do the same. That's what this is.

## The directory layout

The system has four parts, all living under a dedicated Claude workspace directory.

**PLANS** come first — before any implementation starts. When you name a project and feature, Claude creates three files: a design doc (what we're building and why), a dev plan (step-by-step implementation), and a test plan (how to verify it works). Claude drafts them, you review and approve before a single line of code gets written. The point is to capture the "why" up front, before it gets buried under a hundred commits.

**WORKLOGS** are the long-term memory. Organized by project and feature, written at the end of every session. Each feature gets its own folder with an index and dated daily logs. More on these in the next section — they're the core of the system.

**SESSIONS** are a lightweight time layer on top of worklogs. A session file doesn't contain content — it's just a list of pointers to whichever feature worklogs were touched that day. Two views, cross-referenced: WORKLOGS answer "what happened to this feature over time?", SESSIONS answer "what did I touch in this conversation?"

**KNOWLEDGE** is where surprises get documented. Undocumented behaviors, gotchas, things that cost an hour to figure out. Organized by domain — `astro/`, `postgres/`, whatever you're working in. The rule is simple: before touching any area with known quirks, read the relevant knowledge files. When something bites you, write it down so it can't bite you again.

```
PLANS/{project}/{feature}/
  design.md
  dev.md
  test.md

WORKLOGS/{project}/
  INDEX.md
  {feature}/
    INDEX.md
    YYYY-MM-DD.md

SESSIONS/YYYY-MM-DD/
  {session-id}.md

KNOWLEDGE/{domain}/
  {slug}.md
```

## Two views, one history

The system is built around two complementary indexes — WORKLOGS and SESSIONS. They cover the same work from different angles.

**WORKLOGS** are feature-centric. They answer: *what happened to this thing over time?* If you want to know the full history of the auth feature — every session that touched it, every decision made, where it stands today — you go to its worklog.

**SESSIONS** are time-centric. They answer: *what did I work on in this conversation?* A session file doesn't contain any content — it's just a list of pointers to whichever feature worklogs were touched that day.

They cross-reference each other:

```
SESSIONS/2026-04-27/abc123.md
│  → my-app / auth        WORKLOGS/my-app/auth/2026-04-27.md
│  → my-app / dashboard   WORKLOGS/my-app/dashboard/2026-04-27.md
│
▼
WORKLOGS/my-app/
  INDEX.md                  [all features, status, last touched]
  auth/
    INDEX.md ◄──────────────[records session abc123 → back-ref to SESSION]
    2026-04-27.md ◄─────────[pointed to by SESSION]
  dashboard/
    INDEX.md ◄──────────────[records session abc123 → back-ref to SESSION]
    2026-04-27.md ◄─────────[pointed to by SESSION]
```

A session points forward to worklogs. A worklog INDEX points back to the sessions that touched it. Neither duplicates the other — the content lives in one place, the time index lives in another.

Because the structure is consistent and predictable, Claude can answer natural language queries by resolving them to file paths:

| You say | Claude does |
|---|---|
| "continue" / "pick up" / "resume" | Reads feature INDEX → opens latest log → summarizes → waits for confirmation |
| "where did I leave off on auth?" | Opens `WORKLOGS/my-app/auth/INDEX.md` → finds latest daily file |
| "what did I work on last session?" | Opens `SESSIONS/` → finds latest date → reads pointer file |
| "what's the current state of the project?" | Opens `WORKLOGS/my-app/INDEX.md` |
| "full history of the dashboard feature?" | Opens `WORKLOGS/my-app/dashboard/INDEX.md` → reads all daily logs |

## WORKLOGS: the real memory

A worklog is a structured markdown file Claude writes at the end of every session. Not a freeform summary — a precise, scannable snapshot of a feature's state at a specific point in time, designed to be read by a future session.

Each feature gets its own folder with an index and dated daily logs:

```
WORKLOGS/my-app/
  INDEX.md               ← all features, status, last touched
  auth/
    INDEX.md             ← every session that touched auth
    2026-04-24.md
    2026-04-27.md
  dashboard/
    INDEX.md
    2026-04-25.md
```

The feature INDEX is a running table — date, session ID, one-line summary. The daily logs are the actual content. Each one follows the same fixed structure:

```markdown
# Worklog — {project} / {feature} — YYYY-MM-DD

## State of the feature
## {Component}
Dense snapshot of where this component is right now.

## Decisions & reasoning
## {Decision}
What was chosen and why.

## What to do next
## 1. {Next action}
Concrete, ordered, ready to pick up immediately.

## Watch out for
## {Gotcha}
Hard-won lessons that shouldn't have to be relearned.
```

Each section serves a distinct purpose. **State** is the current snapshot — what exists, how it works, exact implementation details. **Decisions** is the reasoning layer — the why behind the what, which is always the first thing you forget. **What to do next** is the handoff — the first thing the next session should pick up. **Watch out for** is institutional memory — the stuff that cost an hour to figure out, documented so it never costs another.

The specificity is the whole point. Not "we built the auth flow" — but exactly what was built, how the token refresh works, what edge case was handled and how. The kind of detail that lets you resume without a warmup lap.

## SESSIONS: the time index

A session file is intentionally lightweight. No content — just a header and a list of pointers to whichever feature worklogs were touched that day:

```markdown
# Session — 2026-04-27 — abc123
Model: claude-sonnet-4-6 | Outcome: in-progress

→ my-app / auth       WORKLOGS/my-app/auth/2026-04-27.md
→ my-app / dashboard  WORKLOGS/my-app/dashboard/2026-04-27.md
```

That's it. The actual substance lives in the worklogs — sessions just know where to find it. This is deliberate: if sessions contained content, you'd end up with two sources of truth that drift apart. Instead, sessions are a pure index. Follow the pointers to get the detail.

The link is bidirectional. Sessions point forward to worklogs; each feature's `INDEX.md` records the session IDs that touched it, pointing back. You can navigate either direction — from a session to its worklogs, or from a feature's history back to the sessions that shaped it.

## A week in the life

Here's what the system looks like in practice — two projects, multiple features, across a week of real work. Some days have multiple sessions because context hit the threshold and got swapped out mid-day.

```
Monday
  Session a1b2 → my-app / auth           ← morning session
                 my-app / dashboard
  Session a1b3 → my-app / auth           ← afternoon, context swapped

Wednesday
  Session c3d4 → my-app / auth
                 side-project / api

Friday
  Session e5f6 → side-project / api      ← morning
  Session e5f7 → my-app / dashboard      ← afternoon
                 side-project / api
```

By end of week, the tree looks like this:

```
WORKLOGS/
  my-app/
    INDEX.md                ← auth: in-progress | dashboard: in-progress
    auth/
      INDEX.md              ← a1b2 (Mon AM), a1b3 (Mon PM), c3d4 (Wed)
      2026-04-21.md         ← two sessions merged into one daily log
      2026-04-23.md
    dashboard/
      INDEX.md              ← a1b2 (Mon AM), e5f7 (Fri PM)
      2026-04-21.md
      2026-04-25.md
  side-project/
    INDEX.md                ← api: in-progress
    api/
      INDEX.md              ← c3d4 (Wed), e5f6 (Fri AM), e5f7 (Fri PM)
      2026-04-23.md
      2026-04-25.md         ← two sessions merged into one daily log

SESSIONS/
  2026-04-21/a1b2.md
  2026-04-21/a1b3.md        ← same day, second session
  2026-04-23/c3d4.md
  2026-04-25/e5f6.md
  2026-04-25/e5f7.md        ← same day, second session
```

Multiple sessions in a day write into the same daily worklog file — the log accumulates, the session index just gets another row. Come Monday the following week, you say "let's continue on auth." Claude opens the feature INDEX, finds the latest entry — Wednesday's log — reads it, and summarizes where things stand before writing a line. No re-explaining context. Just pick up the thread.

## The tradeoff

This system adds overhead. Logs get written at session end, indexes stay updated, and Claude has to follow a structure it doesn't come with out of the box. It's overkill for a one-off task.

But for ongoing, multi-session work on real projects — it pays back fast. The thread doesn't break between sessions. Decisions don't get relitigated. Gotchas get documented once. And when you're juggling multiple features across multiple projects, the indexes do the navigation for you.

The deeper point is about what kind of collaborator you want Claude to be. A one-shot tool that forgets everything when you close the tab, or something closer to a real long-term working partner that knows the history, remembers the reasoning, and picks up where you left off.

The model itself can't give you that — not yet. But the system around it can.
