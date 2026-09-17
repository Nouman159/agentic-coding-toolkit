# agentic-coding-toolkit

A complete, production-tested setup for working with Claude Code — proper CLAUDE.md structure, custom skills, trigger rules, and the guardrails that keep AI-agent-built code from falling apart before it ships.

## What's in here

```
.claude/
├── rules/
│   ├── coding-practices.md   # DRY/KISS/YAGNI, naming, testing, error handling, security
│   └── scalability-guide.md  # Architecture, data access, APIs, background work, infrastructure
└── skills/
    └── daily-update/         # Generate an EOD progress update from today's git history
```

Drop the `.claude/` folder into any repo (or copy over the pieces you want) and Claude Code will pick up the rules and skills automatically.

## Rules

### `coding-practices.md`

Project-wide conventions Claude Code applies to every change: naming and file
structure, comment discipline, documentation triggers, efficiency and
concurrency patterns, testing expectations, and error-handling/security
requirements (no hardcoded secrets, RLS as access control, authentication vs.
authorization, validate at the boundary, etc.). Read it once — it's written to
be enforced, not skimmed.

### `scalability-guide.md`

The system-design counterpart: applies when a change decides *where code lives*
or *what it costs at scale*. Covers the priority order every decision resolves
against (correctness → security → maintainability → performance → scalability →
complexity), following the architecture already in the repo instead of
introducing a competing one, layer separation and feature-owned modules,
querying the database the way the data is actually shaped (pagination,
indexes, no N+1), API contracts that survive their consumers, moving expensive
work into background jobs, stateless servers, caching with a real invalidation
story, monitoring, blast-radius analysis before changing shared code, and
choosing infrastructure for the traffic you have rather than the traffic you
imagine.

The two files split on a single line: `coding-practices.md` governs how the
code you're writing should read; `scalability-guide.md` governs where it
belongs and what it costs. Overlapping concerns (caching, frontend data volume,
testing depth, authorization) live in one place only — cross-referenced, not
duplicated.

## Skills

Skills are reusable playbooks Claude Code loads for a specific kind of task. Invoke one explicitly with `/<skill-name>`, or just ask for what it does in plain language — Claude Code matches your request against each skill's description automatically.

### `daily-update`

Turns today's git commit history in the current repo into a professional, ready-to-paste EOD/standup update — in your own voice, not a generic changelog.

- Filters commits to your own git identity for "today" (local time).
- Reads the actual diffs, not just commit subjects, so terse messages like `fix bug` or `wip` don't produce a vague update.
- Groups the update by platform/module and picks the right shape for the day: a bullet list of small scattered fixes, a module-by-module summary, or a narrative write-up for a big feature/migration — see `references/examples.md` for the three voice patterns it matches against.
- Never commits, pushes, or sends anything — it only reads git history and hands you text to paste into Slack, Teams, or email yourself.

**Usage:**

```
/daily-update
```

or just ask Claude Code: *"give me a daily update for the team"* / *"what did I work on today?"*

If you're not in a git repo, or you made no commits today, it says so plainly instead of inventing content.

## Using this in your own project

1. Copy the `.claude/` folder (or the specific `rules/`/`skills/` subfolder you want) into your project root.
2. Adjust `coding-practices.md` to match your stack — the four principles (DRY/KISS/YAGNI/separation of concerns) generalize, but file layout and language-specific bits (e.g. `src/lib/`) should reflect your repo.
3. Skills work out of the box — they only assume a local git repository.

## Contributing

This toolkit is meant to be forked and adapted. If you build a skill or rule that's broadly useful (not tied to a specific product or company), PRs are welcome — keep examples and references generic so anyone can drop them into their own project.
