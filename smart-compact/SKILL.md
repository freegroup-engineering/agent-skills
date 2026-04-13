---
name: smart-compact
description: Intelligently compact context with a handover guide that preserves what matters
---

# Smart Compact

When invoked, perform an intelligent context compaction by building a handover guide that captures your current state, then instruct the user to run `/compact`.

## Step 1: Assess Your Activity Level

Evaluate what you're currently doing based on the conversation so far:

- **Idle**: No active task. Last messages were casual chat, status checks, or the conversation just started. You're not in the middle of anything.
- **Light**: Simple task in progress — answering a question, making a small edit, looking something up. Low risk of losing context.
- **Moderate**: Multi-step task in progress — building a feature, debugging an issue, coordinating across files. Meaningful context would be lost.
- **Heavy**: Deep in a complex flow — large refactor, multi-file implementation, investigation with many findings, active debugging session with important state.

Be honest about the level. If in doubt, round up — it's better to capture too much than too little.

## Step 2: Determine Message Capture Depth

Based on the activity level, decide how much recent conversation context to summarize in the handover guide:

| Level    | Messages to summarize | Rationale |
|----------|----------------------|-----------|
| Idle     | Last 5-10 messages   | Just enough to remember what was discussed |
| Light    | Last 15-20 messages  | Capture the current small task |
| Moderate | Last 30-40 messages  | Capture the full task context and decisions |
| Heavy    | Last 50+ messages    | Capture everything — decisions, dead ends, findings |

## Step 3: Generate the Handover Guide

Write a markdown file with the following sections. Be specific and concrete — the version of you that reads this after compaction will have NO other context. Write as if briefing a competent replacement who has never seen this conversation.

```markdown
# Handover Guide
Generated: [current date and time]
Activity Level: [idle/light/moderate/heavy]

## Who I Am
[Your agent name, role, and working directory. Pull this from your CLAUDE.md.]

## What I Was Doing
[Specific description of the current task(s). Include:
- What was requested
- What approach you chose and why
- Current status (not started / in progress / blocked / nearly done)
- If blocked, what's blocking you]

## Key Decisions Made This Session
[List every non-trivial decision you made. Examples:
- "Chose to use X library instead of Y because..."
- "Decided to split the migration into two PRs because..."
- "User said to skip tests for now"
If no significant decisions, write "None — session was lightweight."]

## Pending Work
[Concrete list of what still needs to happen. Be actionable:
- BAD: "Finish the feature"
- GOOD: "Implement the /api/users endpoint in src/routes/users.ts — schema is defined, handler is stubbed out, need to add DB queries and validation"]

## Recent Context Summary
[Summarize the recent conversation based on the message depth from Step 2. Include:
- What the user asked for
- What you discussed or debated
- Any preferences or constraints the user stated
- Tone/mood of the conversation if relevant]

## Important State
[List any external state that matters:
- Files you created or modified (with paths)
- Git branches you're on, commits you made, PRs you opened
- Processes running (builds, servers, background jobs)
- Temporary files or artifacts you created
- Environment changes you made]

## Resume Instructions
[Step-by-step instructions for picking up exactly where you left off. Be explicit:
1. Read this handover guide
2. Check [specific file] for the current state of [thing]
3. The next action is to [concrete next step]
4. Then [subsequent steps...]

If idle, just write: "No active task. Read messages as they come in."]
```

## Step 4: Write the Guide

Write the handover guide to `compact-guide.md` in your current working directory (the agent's working directory). Overwrite any existing file at that path.

## Step 5: Inform the User

Send a short message confirming the guide is written. Include:
- The activity level you assessed
- The path where the guide was written
- Tell the user to now run `/compact` with an appropriate number of recent messages to keep (based on the activity level assessment: idle = 5, light = 15, moderate = 30, heavy = 50)

Example response:
> Handover guide written to `/path/to/your/agent/compact-guide.md`
> Activity level: **moderate** (mid-way through building the notification system)
> Run `/compact 30` to compact with the last 30 messages preserved.

Do NOT attempt to trigger compaction yourself. The user must run `/compact` manually.
