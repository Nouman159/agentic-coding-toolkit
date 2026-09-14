---
name: daily-update
description: Write a professional end-of-day team progress update from today's git commit history in the user's own voice. Use when the user asks for a daily update, EOD summary, standup message, progress report, or "what did I work on today" recap to share with the team.
---

# Daily Update Generator

Turn today's git activity in the current repo into a progress update message ready to paste into Slack/Teams/email. Read `references/examples.md` before writing — it holds three style samples this skill must match.

## Step 1: Identify the author to filter by

Get the user's git identity for this repo: `git config user.name` and `git config user.email`. If commits in the log don't match either cleanly (e.g. multiple similar names), ask which identity to filter by instead of guessing.

## Step 2: Pull today's commits

Get local "today" bounds and list only this author's commits:

```
git log --author="<name or email>" --since=midnight --date=local --pretty=format:'%H|%ad|%s' --date=format:'%H:%M'
```

If the working directory isn't a git repo, or there are zero matching commits, say so plainly and ask whether to widen the range (yesterday, this week) instead of inventing content.

## Step 3: Read the real changes, not just the messages

Commit subjects are frequently terse or misleading ("fix bug", "wip", "updates"). For every commit from Step 2, inspect the actual diff — `git show --stat <hash>` for scope, and `git show <hash>` for content — to understand what functionally changed: which screens/modules/files, what broke and what now works, what was added. Build the update from the diffs, not from paraphrasing commit subjects.

Never state a detail (line counts, file counts, component names, behavior claims) that isn't actually evidenced by the diffs.

## Step 4: Group by logical area

Cluster the day's commits by platform (Web/Mobile/Backend) and/or by module/feature (based on file paths and diff content), whichever grouping the work actually falls into. Don't force headings when everything is one area — see Style A/B in the reference.

## Step 5: Pick the style that fits the shape of the day

Match against `references/examples.md`:

- **Style A** — lots of small, independent fixes/tweaks scattered across areas. Group under platform/area headings (only if >1 area exists). Bullet per fix: **bold short lead phrase**, colon, one sentence on before → after.
- **Style B** — the day clusters into a handful of modules/features with a few related changes each, but no single thing is a "whole feature shipped." One line per module: **Module:** comma-separated list of what changed. Roughly 1.2x the density of Style A per line — a touch more detail, still tight.
- **Style C** — a large feature, migration, or major milestone was completed (real scale: many files/lines/subsystems). Lowercase roman-numeral list (i, ii, iii…), each point a full narrative sentence or two naming the actual subsystems/components built, citing scale numbers only when the diff stats back them up.

If the day is mixed (e.g. mostly small fixes plus one big completed feature), lead with whichever dominates and use a short Style C-like callout for the milestone item within the same message rather than forcing everything into one mold.

## Step 6: Write it

- Professional, clear, concise, easy to scan — but don't strip out technical terms (component names, API/module names, real feature names) that the team needs to understand what shipped. Simplify the *sentence*, not the *vocabulary*.
- Describe behavior/impact (what was broken → what now works, what capability now exists), not implementation trivia like variable names.
- No fabrication: if a commit's purpose is unclear even after reading the diff, describe what changed structurally rather than guessing intent.
- Output the message as final text ready to paste — do not wrap it in extra commentary about the process. Briefly note if anything was excluded (e.g. WIP/merge commits) so the user can sanity-check coverage.

## Notes

- This only ever reads git history — it never commits, pushes, or sends the message anywhere. The user shares it themselves.
- If run in a repo with no commits from the user today, don't pad the message — report that plainly.
