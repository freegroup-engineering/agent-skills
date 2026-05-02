---
name: wiki
description: Maintain a persistent, compounding shared knowledge base at `~/knowledge/wiki/` instead of letting facts decay in scratch files or per-agent memory. Use this skill when ingesting a new source (article, doc, transcript, decision), when querying for what's known about an entity/topic before doing related work, when noticing contradictions or stale claims, or when a strategic decision is finalized that other agents will need later. Trigger words: ingest, save to wiki, what do we know about, lookup, knowledge base, source doc, decision log, lint wiki, audit wiki, wikify, persist this, remember this, record this decision, capture this for the team.
---

# Wiki — shared, compounding knowledge for the agent swarm

A maintained markdown wiki at `~/knowledge/wiki/` is our shared brain. Unlike per-agent Claude Code memory (which is private behavioral context for one agent), the wiki is the **shared source of truth** every agent reads.

Pattern is from Karpathy's LLM-Wiki concept: raw sources → an LLM-maintained interlinked wiki → a small schema. The unlock is that LLMs are good at the boring cross-referencing that makes humans abandon wikis.

## When to invoke this skill

- **Before** doing significant work on an entity/topic — query the wiki first, don't redo discovery another agent already did.
- **After** ingesting any source longer than a few sentences (article, transcript, research note, long Discord message): persist what's worth keeping.
- **When** a strategic decision is made (pricing, architecture, hiring, partner choice): record it as a decision page so it survives compaction.
- **Periodically** when noticing a wiki page is stale or contradicts something newer: lint it.

## Layout

```
~/knowledge/
├── SCHEMA.md         ← the rules (read once, then the skill encodes them)
├── wiki/
│   ├── entities/     ← people, companies, products, places (one page per entity)
│   ├── topics/       ← concepts, ongoing initiatives (one page per topic)
│   ├── decisions/    ← record of major decisions (immutable once written, dated filename)
│   └── glossary/     ← short definitions of recurring terms
└── sources/          ← raw source archive (immutable; cited by wiki pages)
```

Existing legacy dirs (`ronen/`, `freegroup/`, `iniminimo/`, `freemans/`, `agent-log/`, `sub-agents/`) are flat docs from before this skill landed. They'll be migrated incrementally — when you touch a topic that has legacy content, fold it into the wiki and mark the legacy doc as migrated. Don't bulk-migrate; do it as work touches each area.

## Page format

Every wiki page is markdown with YAML frontmatter:

```markdown
---
title: Iniminimo Pricing Strategy
type: topic
tags: [iniminimo, pricing, gtm]
sources: [2026-04-15-mark-pricing-research, 2026-04-22-ronen-pricing-call]
updated: 2026-05-02
status: active
---

# Iniminimo Pricing Strategy

## Current model
...

## See also
- [[entity:iniminimo]]
- [[topic:iniminimo-positioning]]
- [[decision:2026-04-22-pricing-tier-structure]]
```

**Required frontmatter:**
- `title` — human-readable
- `type` — `entity` | `topic` | `decision` | `glossary`
- `updated` — ISO date of last meaningful edit
- `status` — `active` | `stale` | `deprecated`

**Optional but encouraged:**
- `tags` — flat list, lowercase
- `sources` — list of source IDs cited in the body (filenames in `sources/` minus `.md`)

**Internal links:**
- `[[entity:ronen-freeman]]` → `wiki/entities/ronen-freeman.md`
- `[[topic:gm-l8-interview-prep]]` → `wiki/topics/gm-l8-interview-prep.md`
- `[[decision:2026-04-30-vm-split]]` → `wiki/decisions/2026-04-30-vm-split.md`
- `[[glossary:dlq]]` → `wiki/glossary/dlq.md`

**Source citations in body:**
- Inline: `(see [src:2026-04-22-ronen-pricing-call])` after the claim that came from that source.
- Aggregated in frontmatter `sources:` list.

## The three operations

### 1. Ingest

When you receive a source worth keeping (article, doc, decision recap, voice transcript):

1. **Save the raw source first.** Create `sources/<YYYY-MM-DD>-<short-slug>.md`:
   ```markdown
   ---
   title: <human title>
   url: <if applicable>
   date: <YYYY-MM-DD>
   ingested_by: <agent name>
   ---

   <full raw content or transcript or summary>
   ```
   The slug should be 3-5 words, kebab-case. Date prefix sorts chronologically.

2. **Identify which wiki pages this source touches.** Usually 1-3 entities + 1 topic. Use `Grep` on `wiki/` for related terms first; don't create duplicate pages.

3. **For each touched page:**
   - If the page exists: read it, integrate the new information into the right section, add a `[src:<source-id>]` citation inline, append the source ID to frontmatter `sources:`, bump `updated:`.
   - If the page doesn't exist: create a stub. Don't try to write a comprehensive page from one source — let it grow over time.

4. **Cross-link.** When you mention another entity/topic that has its own page, use `[[type:slug]]` syntax. If the target page doesn't exist yet, create a stub with at least the frontmatter + a one-line description.

See `ingest.md` for worked examples and edge cases.

### 2. Query

Before doing real work on a known entity/topic:

1. `Grep` `~/knowledge/wiki/` for relevant terms (tag names, entity names, key phrases).
2. Read the matched pages. Follow `[[...]]` links one hop deep if the page references something you don't have context on.
3. **Synthesize the answer with citations.** Cite specific wiki pages (`see [[topic:foo]]`) and underlying sources (`per [src:bar]`).
4. **If you find gaps** (the question wasn't fully answered): say so explicitly. Don't fabricate. Suggest an ingest if a fresh source would close the gap.

The wiki is the first place to look. Per-agent memory + Discord history are second. Don't search Discord for facts that should be in the wiki.

### 3. Lint

Run periodically (cron once a month + ad-hoc when something feels off):

1. **Stale pages.** Any page with `updated` > 90 days and `status: active` → flag for review. Maybe still true, maybe not — needs human/agent eyeballs.
2. **Broken `[src:...]` refs.** Any source ID cited that doesn't exist as a file in `sources/`. Fix or remove.
3. **Dangling `[[type:slug]]` links.** Any internal link pointing to a non-existent page. Either create the missing stub or fix the link.
4. **Conflicts.** Harder — read pairs of pages on related topics and look for contradicting claims (e.g., two different prices, two different "current" versions of an architecture). Flag the conflict; let a human resolve.
5. **Missing connections.** If two pages obviously relate but neither links to the other, add the link.

The lint report is itself a markdown doc written to `~/knowledge/lint/<YYYY-MM-DD>.md`. Don't auto-fix everything — surface the diff for review.

See `lint.md` for the full lint procedure.

## Decision framework — should this go in the wiki?

Before saving anything, ask:

| Goes in wiki | Stays out |
|---|---|
| Persistent fact that compounds over time (Ronen's wife is Eve) | Ephemeral state (current sprint todo) |
| A decision that future agents need (we use Wyoming LLC, not Delaware) | Behavioral note about one agent (Mark prefers concise commits) — that's per-agent memory |
| Cross-agent context (how iniminimo's pipeline works) | Personal-to-Ronen observation (Ronen is grumpy after midnight) — that's user memory |
| Reference data (pricing tiers, brand colors, OAuth client ID location) | Active conversation context |
| External source worth preserving (research article, customer transcript) | A Slack message you can re-read |

Per-agent Claude Code memory and the wiki are complementary:
- **Per-agent memory** = how I work, who the user is from my perspective, behavioral feedback. **Private**.
- **Wiki** = what's true about the world we're operating in. **Shared**.

## Conventions cheat sheet

- Filenames: lowercase, kebab-case, descriptive. `entities/eve-freeman.md`, `topics/iniminimo-music-brand.md`, `decisions/2026-04-30-vm-split.md`.
- Decision filenames are date-prefixed (`YYYY-MM-DD-`) and never renamed.
- Other pages are renamed only on a structural change; treat slug as semi-stable (because incoming links break).
- Each page should be one screen-ish to start. Grow over time.
- When in doubt, prefer a new short page over an overstuffed one. Cross-link.
- Don't paste full source content into wiki pages — keep raw content in `sources/`, summarize in the wiki, cite the source.

## Anti-patterns

- **Ingesting transient context.** Do not save current-conversation state, sprint todos, or in-flight Discord threads. Wiki is for things that compound.
- **Writing without a source.** Every claim should be traceable. If you "just know" something, it probably belongs in per-agent memory, not the wiki.
- **Massive pages.** A 2000-line entity page is unmaintainable. Split into sub-topics with cross-links.
- **Silent edits.** When updating a page, bump `updated:` and add the new source to `sources:`. Don't edit-and-forget.
- **Bypass the schema.** If something doesn't fit a known type, talk to Freddie before inventing a new top-level dir.

## Read deeper only when needed

- `schema.md` — full schema rules and edge cases
- `ingest.md` — worked ingest examples
- `lint.md` — the lint algorithm + how to write a lint report
- `migration.md` — folding legacy `~/knowledge/<topic>/` flat docs into the wiki

Default: rely on this file. Open the deeper ones only when their topic is what you're about to do.
