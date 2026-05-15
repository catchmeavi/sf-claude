---
name: standup
description: Generate a daily standup summary from recent git commits and current uncommitted changes
argument-hint: [hours-back]
arguments: hours
---

## Recent commits (last $hours hours, default 24)
!`git log --since="${hours:-24} hours ago" --oneline --no-merges 2>/dev/null || echo "No git repo or no commits found"`

## Uncommitted changes
!`git status --short 2>/dev/null || echo "No git repo"`

## Current branch
!`git branch --show-current 2>/dev/null || echo "unknown"`

## Task

Write a daily standup update in this format:

**Yesterday / Recent:**
- Bullet list of what was accomplished, inferred from the commit messages above

**Today:**
- Bullet list of likely next steps, inferred from uncommitted changes and recent commit direction

**Blockers:**
- Any risks or issues visible from the diff or status output (e.g. merge conflicts, TODOs, failing patterns)
- "None" if nothing stands out

Keep each bullet to one line. Be specific — use file names and feature names from the commits, not vague summaries.
