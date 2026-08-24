# Agent working model — how we talk to Ronen and to each other

**Rule (all agents: Nova, Cleo, Margo, Boaz, Mark + every subagent). Set by Ronen, 2026-08-24.
This is standing process, not a preference. He should never have to explain it again.**

He changed this because the old way was one firehose channel, several agents talking at once,
and him having to reconstruct which message belonged to which piece of work. The cost landed
entirely on him, which is why the rules below are all about *his* reading experience, not ours.

## 1. One thread per ticket

Every ticket gets its own Discord thread, opened off **your private channel with Ronen** — not
off the agent working channel. Peer channels are for peer traffic; a ticket thread there means
he has to read our cross-talk to find his own conversation.

Open it with `~/bin/discord-thread <channel_id> "<INI-NN · short title>" <his_message_id>`.
A thread started from a message takes that message's ID as its own channel ID — pass it
straight back to the reply tool as `chat_id`.

Then **add a row to `~/.claude/channel-topics.md`**. That file is box-global and every agent
reads it. A thread nobody registered is a thread the next agent will answer in blind.

## 2. One message, then stop

Send **one** short message and **wait for his reply**. Do not send a second before he responds.
His words: *"youre again just messsaging and messaging and not giving me a chance to respond."*

This is the most-repeated correction on record. It is not about length — it is about not
stacking three questions on him at once, because he can only answer the last one.

Length: ~120 words, lead with the answer, no emoji, no trailing recap. See [[comms]].

## 3. Never mix threads

Resolve the incoming `chat_id` against `~/.claude/channel-topics.md` **before** answering, and
answer only that channel's topic. A status roundup spanning everything you did is the wrong
answer even when every line in it is true — it buries the thing he asked about.

If something urgent from another area needs him, send it **in its own channel**, as its own
message. Never as a tail on an unrelated answer.

## 4. Cross-agent chat happens outside the ticket threads

When you need another agent's input, message them in the **shared channel for that pair**, not
in Ronen's ticket thread. Mentioning another agent inside his thread pushes the message to them
and pollutes the thread he opened for himself.

Every outbound message opens with `<@id>` for the intended recipient — see the ID table below.

## 5. Full-stack tickets split into per-agent subtasks

A ticket that spans backend and frontend gets **one subtask per agent**, each with its own
owner, its own status, and its own thread. Ronen talks BE in the backend owner's thread and FE
in the frontend owner's thread; the two owners talk to each other in their shared channel.

**INI-54 is the worked example**: INI-64 (BE, Nova) / INI-65 (FE, Cleo), with the cross-side
payload contract settled between the two agents, not in his thread.

## 6. Several tickets run in parallel

One thread each. Parallel is the point — the threads are what make it readable.

## 7. Only Ronen moves a ticket to Done

Move it at every other stage yourself. `In Review` never means finished; say what is still
open when you move it there.

## Channel map

Authoritative pair/topic map is `~/.claude/channel-topics.md` — this section is the
who-talks-to-whom summary. Add rows there, not here, when a channel is created.

| pair / purpose | channel |
|---|---|
| Ronen ↔ Nova (private) | `1491891485283582134` |
| Ronen ↔ Cleo (design/creative) | `1491732936217854042` |
| Nova ↔ Mark (legal gate, vendor matrix, consent) | `1541444694242758706` |
| Agent working channel (Nova / Cleo / Mark peer traffic) | `1539952466144133170` |
| All-agents | `1493503507557388368` |
| SRE / alerts (Boaz) | `1525775081014558760` |

**Gaps, 2026-08-24 — ask Ronen, do not invent:** no known dedicated shared channel for
Nova↔Boaz, Cleo↔Mark, Cleo↔Boaz, or Mark↔Boaz. Margo is reachable only via Cleo — the relay
bot has no access to `#inmo-creative` (`1498299327502745752`).

## Discord user IDs

Open every outbound message with the recipient's `<@id>`.

| who | id |
|---|---|
| Ronen | `1490332987370246144` |
| Nova | `1490456900221800621` |
| Cleo | `1490684851764986026` |
| Mark | `1491529730606760068` |
| Margo | `1498294808584917153` |
| Boaz | `1493008892739715282` |
| Freddie | `1490338295497101403` |

## Why this is written here and not only in memory

Per-agent memory under `~/.claude/projects/<slug>/memory/` is **private to one agent's working
directory**. The `<@id>` rule above sat in Nova's memory for over a month with entries
explicitly addressed to Cleo, who could not see them. Anything addressed to the swarm goes in
`~/knowledge/` or `~/.claude/CLAUDE.md`, never in one agent's memory dir.

See also: [[worktree-discipline]], [[ticket-status-workflow]].
