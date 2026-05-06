---
title: "Claude, We've Talked About This. (Enter CLAUDE.md.)"
date: 2026-05-05
description: "Why CLAUDE.md is the most important file in my Claude Code setup, and how I've structured my own."
---

## The default Claude doesn't know you

Claude is brilliant out of the box. Open the terminal, type a question, get a thoughtful answer. The same is true in Claude Code: you can drop into any project and start working immediately.

But there's a catch I noticed pretty quickly. Every new conversation, Claude starts from zero. It doesn't know what tools I prefer, how I structure projects, what I've decided not to do, or what I've already tried. It re-learns the basics every single time, and I burn the first ten minutes of each session re-explaining myself.

That gets old fast. There has to be a way to fix it. There is, and it's a single file.

## What CLAUDE.md is

CLAUDE.md is the answer to "how do I tell Claude this once and have it stick." It's a markdown file that Claude Code reads at the start of every session. Anything I put in it becomes part of how Claude shows up to work. Every time.

There are two places it can live:

- **Global**: `~/.claude/CLAUDE.md`. Applies to every project on the machine.
- **Project-local**: `./CLAUDE.md` in any repo. Applies only when Claude runs in that project. Useful for team-shared conventions.

I went global only. My way of working is mine, not the project's. The conventions I want Claude to follow (how I plan, how I document, how I want bash commands run) follow me across every codebase. If I forked a team project tomorrow, I'd still want the same defaults.

The project-local file makes sense for teams. If five people contribute to a repo and you all agree on a style guide or want Claude to use specific commands, `./CLAUDE.md` keeps everyone aligned. I just don't have that need yet.

When both files exist, Claude reads both. They compose, with project-local generally winning conflicts. So the two scopes are complementary, not exclusive. Global defaults, project overrides, both at once.

## The philosophy: a contract for how we work

The reason CLAUDE.md exists as a category is simple: separate what Claude needs to know about *me* from what it needs to know about the *code*. Claude is already good at reading code. It can figure out my test runner, my framework, my file structure, my dependencies. What it can't infer is who I am, how I think, what conventions I care about, what I've decided not to do. CLAUDE.md is for the second category.

I think of it as a contract I write once that defines the terms of every future collaboration. Not a knowledge base. Not a notebook for facts. A contract that says: "this is how we work together."

That framing matters because it changes what belongs inside. A knowledge base is exhaustive: every fact you might need. A contract is sparse: only the rules that govern how we work. CLAUDE.md is the second. Which is also why most CLAUDE.md files I've seen are too long. They're trying to be reference docs when they should be charters.

## What to do: walking my CLAUDE.md, section by section

My CLAUDE.md has five top-level sections. Each answers one question about how Claude and I work together:

- **Work Practices**: how do we structure a session from start to finish?
- **Research**: where should Claude look first when it needs information?
- **Knowledge**: where do durable lessons and gotchas live?
- **Skills**: where do reusable how-to recipes live?
- **Tool Use**: small habits that make tool calls cleaner.

Work Practices is the heaviest. It contains the rules that shape every session. The other four are smaller but each pulls weight. I'll walk through them in order, quote the actual rule, and explain why each one is in there. Most got added because something annoyed me one too many times.

### Bootstrap

The first thing in my Work Practices section is a simple rule for the start of every session:

> When the user mentions a task or feature to work on:
> 1. Confirm the project name and feature/task name
> 2. Ensure `~/Documents/Claude/WORKLOGS/{project}/` exists
> 3. Ensure `~/Documents/Claude/PLANS/{project}/{feature}/` exists, and create three placeholder files if they don't exist:
>    - `design.md`: what we're building and why, key decisions
>    - `dev.md`: step-by-step implementation
>    - `test.md`: how to verify it works
> 4. If the user signals continuity ("continue", "pick up", "resume"), read the latest worklog entry and summarize before proceeding.

This rule got added after the fifth time I caught myself burning ten minutes hunting for context at the start of a session. "What was I working on yesterday? Where's the plan doc? Did I write a worklog?" None of those questions are interesting to me, but every session was forcing me to answer them by hand.

Now Claude does the lookup. If I open with "let's pick up the blog post," Claude reads the latest worklog and gives me a one-paragraph summary: where we left off, what the next step is, what was unresolved. I confirm or redirect, and we're working in under a minute.

The continuity signal piece is what really makes it click. Words like "continue," "pick up," "resume" are reliable triggers that say "I'm not asking for something new, I'm resuming context." Claude doesn't have to guess.

### Always clarify before starting

Right after Bootstrap, the second rule is about not jumping into work too fast:

> Before working on any task (coding, planning, research, etc.):
> - Ask clarifying or prerequisite questions if anything is unclear
> - Then draft a plan (including test and dev plans where relevant) and let the user revise before proceeding
> - For trivial tasks, use judgment. Skip if it's obvious.

The pain that drove this rule: Claude's default mode is to be helpful by acting fast. Useful for a quick question, dangerous for any non-trivial task. I'd ask for "a small refactor" and Claude would draft 200 lines of code based on its best guess at what I meant. Often very wrong, and now I had to either rewrite or correct.

The fix is the inverse default: when in doubt, ask. The rule explicitly carves out "for trivial tasks, use judgment" so Claude doesn't ping-pong on every typo fix. But for anything substantive, the default tilts toward clarification.

This single rule probably saved me more rework time than any other.

### Ask before write operations; read freely

The asymmetry between safe and risky actions:

> - **Write operations**: always ask before writing unless it's part of an already agreed plan
> - **Read operations** (files, websites, documents): proceed freely, no need to ask
> - **OS operations**: use judgment. Ask if risky (deleting files, killing processes), proceed if benign (listing files, checking status).

Reads are cheap and reversible. Nothing changes, I just learn. Writes change the world: a new file, an edit, a moved directory. Mistakes there cost real time.

So the rule is: read freely (don't bother me with permission prompts for `cat` or `ls`), but pause before writes. Same idea for OS operations. `ls` is harmless, `rm -rf` is not.

This probably maps to how a careful new collaborator would behave. The defaults Claude ships with are slightly more cautious about reads than I want and slightly more aggressive about writes. The rule corrects both.

### Project setup

A short rule for when I want to start working but haven't picked a project:

> When the user indicates they want to work on a project (e.g. "let's start working on projects", "let's work on something"):
> - List all projects in `~/Documents/Projects/` in a table format (name + short description if known)
> - Ask which project to work on
> - `cd` into the chosen project before starting

This is mostly a quality-of-life thing. I work on a few projects in parallel and sometimes I'll open Claude without knowing which one I want to touch. Without this rule, Claude would either guess (wrong) or ask me which project (then I'd have to remember the names). Now Claude lists what's there, with a description if it can derive one. I pick. Two seconds instead of fifteen.

### Plan and implement step-by-step

This is the most procedural rule in the file:

> For any non-trivial task:
> 1. Clarify, draft plan, get approval.
> 2. When exiting plan mode with an approved design, STOP. Do not write any code yet:
>    - Run bootstrap if not already done
>    - Write the agreed design to plan docs (design.md, dev.md, test.md)
>    - Only then begin implementation
> 3. Implement one step at a time. Don't jump ahead.
> 4. Verify each step before moving to the next.
> 5. After each step, ask: did anything surprising happen? If yes, write it to KNOWLEDGE before moving on.
> 6. At task completion, write a worklog entry.

This rule governs the rhythm of every non-trivial task: clarify, plan, get approval, document the plan, then implement step by step.

The "step at a time" piece is the part I had to add explicitly. Claude defaults to batching: "I'll do all five things in one go." For most tasks I want the opposite. Do step one, show me, let me confirm or redirect, then step two. Otherwise I'm reviewing 500 lines of diff in one shot and missing things.

### End-of-session documentation

The most detailed rule in the file. The trigger and the structure:

> When the user signals session end (goodbye, wrapping up, stopping, etc.):
> 1. For each feature touched, write `WORKLOGS/{project}/{feature}/YYYY-MM-DD.md` using a structured format (state of the feature, decisions, what to do next, watch out for).
> 2. Update the feature's INDEX.
> 3. Update the project's INDEX.
> 4. Write `SESSIONS/YYYY-MM-DD/{session-id}.md` as pointers to the worklogs touched.
> 5. Update auto-memory `MEMORY.md` with current project state.

I wrote a separate post about this system (Claude Has Amnesia), so I won't reproduce the rationale here. Short version: every session gets two views written down. WORKLOGS organizes by feature, SESSIONS organizes by time. The two views point at each other so I can navigate either way.

The trigger phrase is what makes the rule operational. "Goodbye," "wrapping up," "let's stop here." Any of those tell Claude to flip into documentation mode before the conversation closes. Without the trigger, the documentation step is too easy to forget.

### Research

A short escalation ladder for when Claude needs information it doesn't already have:

> Before searching the web or asking the user, follow this escalation ladder:
> 1. Check `~/Documents/Claude/KNOWLEDGE/` for prior documentation.
> 2. Search the codebase. Scoped search, never brute-force the entire repo.
> 3. Search the web. Official docs, known solutions.
> 4. Ask the user. Last resort, not first resort.

The default Claude behavior is to ask the user pretty quickly when it doesn't know something. Polite but inefficient. Most of the time the answer is in something Claude could check itself. The ladder says: try cheap local sources first, escalate outward only if needed.

The "never brute-force the entire repo" line is small but matters. Without it, Claude would happily grep through 50,000 files, eat tokens, and find irrelevant matches.

### Knowledge

The KNOWLEDGE folder gets a short rule:

> Domain knowledge, corner cases, and gotchas live in `~/Documents/Claude/KNOWLEDGE/`, organized by domain (e.g. `mcp-protocol/`, `astro/`, `vite/`). Each folder contains slug-named files by topic.

KNOWLEDGE is for the things you'd write down on a sticky note next to your monitor. "Remember, Astro 6 stripped the .md from post.id" or "the Slack rate limit is 50 msg/sec, not 100." Stuff that's not in the docs but bit me once.

The rule prompts Claude to read relevant files before touching an area with known quirks, and to write new notes when something surprising happens. Over time the folder becomes a personal compendium of "stuff I've learned the hard way."

### Skills

And the SKILLS folder gets its own short rule:

> Skills live in `~/.claude/SKILLS/`, organized by topic. Each skill is a folder containing skill.md (Claude-facing recipe) and readme.md (human-facing explanation).

Where KNOWLEDGE is conceptual ("watch out for X"), Skills are procedural ("here's how to do Y"). Each skill is a self-contained how-to that future sessions can reuse without me re-explaining.

For example, I have a skill that captures my exact workflow for writing a blog post: where to put the markdown file, what frontmatter to include, how to wire it to the blog index. Future me doesn't have to remember any of it. Claude reads the skill and runs the workflow.

### Tool use

The smallest section, but consequential. Right now it has one rule:

> **Prefer parallel single-command Bash calls over chained ones**
>
> Don't chain harmless commands with `&&`, `||`, `;`, or `|` just to save round-trips. The auto-allow list only covers single simple commands. Compound forms break auto-allow and trigger permission prompts even when every part is harmless.

I added this one a few days ago after realizing Claude was prompting me for permission on basically every read because it kept chaining commands like `ls foo && cat bar && echo "---"`. Each of those is auto-allowed individually, but the chain isn't.

Switching to parallel single-command calls killed the prompts. Small rule, big quality-of-life win.

## What NOT to do

A few anti-patterns I've fallen into or seen others fall into. Fix any of these in your own file and it'll get tighter and more useful immediately.

- **Don't dump memory or state.** "We're mid-migration on auth" doesn't belong here. CLAUDE.md is for durable rules, not facts about today. Use the auto-memory system or worklogs for state. The contract framing helps: a contract doesn't carry today's news.
- **Don't write rules Claude can already derive.** "Use TypeScript" when there's a `tsconfig.json` is noise. Claude reads the codebase already. Rules should encode things the codebase doesn't reveal: how I work, what I prefer, what I've decided not to do.
- **Don't ban behavior with a single negative.** "Never use force-push" without the *why* leaves Claude unable to judge edge cases. Always include the reasoning. A rule with "because force-push to main can lose collaborator commits" lets Claude generalize: it'll also be careful with other history-rewriting operations, even ones not on the rule list.
- **Don't write it as a wall of prose.** Structured sections with headers beat stream-of-consciousness. Claude scans the file. So should you, when you come back to edit.
- **Don't put secrets or anything sensitive in it.** The file is read every session and paraphrased into context. Treat it as semi-public. API keys, passwords, anything you wouldn't want to accidentally leak: not here.
- **Don't optimize for completeness.** An empty or vestigial section is worse than no section. Only include what's earned its place. If a rule isn't pulling weight, delete it. The file should be lean enough to read in two minutes.

## How to evolve a CLAUDE.md

CLAUDE.md isn't write-once. The rules I have today aren't the rules I had a month ago, and the file will keep changing. A few habits that help:

- **Add a rule when you correct Claude on the same thing twice.** The first correction is just a normal mid-session course correction. The second correction means the default isn't right. That's the trigger to write a rule.
- **Delete a rule when it stops giving you value.** Maybe the underlying tool changed. Maybe the workflow it shaped is no longer relevant. Maybe you wrote it during a frustrated moment and it doesn't actually save time. Delete without ceremony.
- **Periodically prune.** Otherwise the file bloats and Claude starts ignoring or contradicting itself. Every month or so I read through the whole file and ask: would I write this rule again if I were starting fresh? If no, it goes.
- **Treat the file as a living document, not setup-once.** This is the biggest mindset shift. Most people write a CLAUDE.md once, treat it as done, and never touch it again. Mine has been edited dozens of times. Each edit took ten seconds and paid for itself within the next session.

A concrete example: the parallel-bash-calls rule I quoted earlier got added a few days ago, after one specific session where I noticed the prompt-spam pattern and identified the cause. Total time from "I notice this is annoying" to "the rule is in CLAUDE.md" was maybe five minutes. The next session felt the difference immediately.

## Closing

CLAUDE.md is the smallest investment with the biggest return in my Claude Code setup. A few dozen lines of markdown bought me a Claude that knows how I work, doesn't ask me the same questions twice, and stays consistent across every project I touch.

If you've been using Claude Code without one, the first version takes about an hour to write and starts paying off the next session. If you've been using one but treating it as static, try treating it as living instead. Edit it when something annoys you. Delete rules that don't earn their place.

This post is the second half of a pair. The first one ([Claude Has Amnesia](#blog/claude-logging-system)) covered the directory structure I use to give Claude memory across sessions. This one covered the configuration file that ties it all together. Together they're the blueprint of how I work with Claude every day.

If you've built your own CLAUDE.md, I'd love to hear what's in it.
