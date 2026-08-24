# 🔴🔴🔴 THE WORKING MODEL — how we talk to Ronen. Applies to EVERY agent (Nova, Cleo, Margo, Boaz, Mark) and every subagent.
Ronen 2026-08-24. Standing process, not a preference. He set it so he never has to explain it again — so do not make him.

1. **One thread per ticket**, opened off **your private channel with Ronen** (not off the agent working channel — peer channels are peer traffic). Create it with `~/bin/discord-thread <channel_id> "<INI-NN · short title>" <his_message_id>`; a thread started from a message takes that message's ID as its own channel ID, so pass it straight back as `chat_id`. **Then add a row to `~/.claude/channel-topics.md`** — a thread nobody registered is one the next agent answers in blind.
2. **Send ONE short message, then WAIT for his reply.** Never a second before he responds. His words: *"youre again just messsaging and messaging and not giving me a chance to respond."* Most-repeated correction on record. ~120 words, answer first, no emoji, no trailing recap.
3. **Never mix threads.** Resolve the incoming `chat_id` against `~/.claude/channel-topics.md` BEFORE answering, and answer only that channel's topic. Something urgent from another area goes in **its own channel as its own message** — never as a tail.
4. **Cross-agent chat happens OUTSIDE his ticket threads**, in the shared channel for that pair. Mentioning another agent inside his thread pushes it to them and pollutes the thread he opened for himself.
5. **Full-stack tickets split into one subtask per agent** (BE owner / FE owner), each with its own owner, status and thread. INI-54 → INI-64 (BE, Nova) / INI-65 (FE, Cleo) is the worked example. The two owners settle the cross-side contract between themselves, not in his thread.
6. **Several tickets run in parallel**, one thread each. **Only Ronen moves a ticket to Done**; you move it at every other stage, and `In Review` never means finished.

Full page, channel map and Discord ID table: **`~/knowledge/wiki/topics/agent-working-model.md`**. Anything addressed to the swarm goes there or here — NEVER in one agent's `~/.claude/projects/<slug>/memory/`, which no other agent can read.

# 🔴🔴🔴 EXTREMELY IMPORTANT — delegate to subagents/background, conserve main-loop tokens
Ronen 2026-06-25: "SUBAGENTS AND BACKGROUND TASKS AS MUCH AS POSSIBLE BUT NOT IF IT WILL DIMINISH THE OUTCOMES... YOU ARE BURNING TOKENS LIKE CRAZY."
- DEFAULT to dispatching work to subagents (`Agent`, `run_in_background: true`) and `Bash` `run_in_background: true`. The main loop should orchestrate + decide, not do bulk reading/building/grepping inline.
- Keep on the main loop ONLY: quick chat replies, money/deploy/live-provider steps (one at a time), and decisions that genuinely need my judgment — anything where delegating would diminish the outcome.
- Don't read large files / sweep the codebase inline — dispatch an Explore/general-purpose agent and keep the conclusion, not the dumps.
- This is a hard cost rule. Bias hard toward delegation; only stay inline when quality would actually suffer.

# graphify
- **graphify** (`~/.claude/skills/graphify/SKILL.md`) - any input to knowledge graph. Trigger: `/graphify`
When the user types `/graphify`, invoke the Skill tool with `skill: "graphify"` before doing anything else.

# Shared swarm knowledge — `~/knowledge/` and the `wiki` skill

`~/knowledge/` is the shared brain of the iniminimo swarm (Nova, Cleo, Margo). Maintained as an LLM-managed wiki (Karpathy's pattern). Read `~/knowledge/SCHEMA.md` for the layout, `~/.claude/skills/wiki.md` for the full operating rules (symlinked to `~/hq/agent-skills/wiki/SKILL.md`).

- **Ingest** when a substantive source arrives (article, transcript, decision recap): persist to `~/knowledge/sources/<date>-<slug>.md`, integrate into touched wiki pages.
- **Query** before doing significant work on a known entity/topic: grep `~/knowledge/wiki/` first.
- **Lint** periodically to catch stale claims and dangling links.
- Wiki = compounding facts about the world (cite or omit). Per-agent memory at `~/.claude/projects/.../memory/` = behavioral / how-I-work — keep them separate.
