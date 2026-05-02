# Wiki — Ingest workflow (worked examples)

The ingest operation turns a raw source into structured, cited entries in the wiki. It is the most common wiki op and the one most likely to be done sloppily. This file walks through it concretely.

## The shape of an ingest

```
RAW SOURCE
   │
   │  1. Save to sources/
   ▼
sources/<YYYY-MM-DD>-<slug>.md   (immutable archive)
   │
   │  2. Identify touched pages (entities, topics, decisions)
   ▼
For each touched page:
   - read existing
   - integrate the new info
   - add inline [src:source-id] citation
   - append source-id to frontmatter sources:
   - bump updated:
```

Whatever you do, do not skip step 1. The raw source is the audit trail. Wiki pages can be rewritten; sources cannot.

## Worked example 1 — A short article

**Source:** Ronen forwards an article URL about `Stripe's MRR Growth Playbook for Pre-Launch SaaS`.

**Step 1 — save raw:**

`sources/2026-05-02-stripe-mrr-playbook-prelaunch.md`:

```markdown
---
title: Stripe's MRR Growth Playbook for Pre-Launch SaaS
url: https://stripe.com/blog/...
date: 2026-05-02
ingested_by: freddie
---

<paste the article body or a faithful summary>
```

Slug rule: 3-5 words, kebab-case, descriptive of the SUBJECT not the source ("stripe-mrr-playbook-prelaunch", not "stripe-blog-post").

**Step 2 — figure out what's touched:**

Grep `wiki/` for `mrr`, `pricing`, `iniminimo`, `gtm`. Find:
- `wiki/topics/iniminimo-pricing.md` (exists)
- `wiki/topics/iniminimo-gtm.md` (exists)
- `wiki/entities/iniminimo.md` (exists)

Probably touches `iniminimo-pricing` (because the article is about pricing) and `iniminimo-gtm` (because launch). Skip `iniminimo` entity page — it's a top-level identity page, doesn't need every tactical input.

**Step 3 — integrate per page:**

Open `wiki/topics/iniminimo-pricing.md`. It has a section `## Tier structure`. The article suggests a 3-tier structure with the middle tier as the anchor. Add to that section:

```markdown
- Stripe's playbook recommends anchoring pricing on the middle tier — the highest tier exists to make middle look reasonable, not to be sold (see [src:2026-05-02-stripe-mrr-playbook-prelaunch]).
```

Update frontmatter:
```yaml
sources:
  - 2026-04-15-mark-pricing-research
  - 2026-05-02-stripe-mrr-playbook-prelaunch  # ← added
updated: 2026-05-02  # ← bumped
```

Repeat for `iniminimo-gtm.md`.

**Step 4 — cross-link:** if the article mentions a specific concept that should have its own page (e.g., "anchor pricing" — a specific term), either link to an existing `glossary/anchor-pricing.md` or create a stub:

```markdown
---
title: Anchor pricing
type: glossary
updated: 2026-05-02
status: active
---

# Anchor pricing

A pricing tactic where a high-priced premium tier exists primarily to reframe the middle tier as the reasonable choice. See also [[topic:iniminimo-pricing]].

Sources: [src:2026-05-02-stripe-mrr-playbook-prelaunch]
```

## Worked example 2 — A voice transcript / Discord conversation

**Source:** Ronen's voice note explaining why he picked Wyoming over Delaware for the LLC.

**Step 1 — save raw.** Voice transcripts go into `sources/` as full text. Date the day Ronen said it. Cite his name as the source author:

`sources/2026-04-22-ronen-on-wyoming-vs-delaware.md`:

```markdown
---
title: Ronen explains Wyoming vs Delaware LLC choice
source: Ronen Freeman (voice note in DM)
date: 2026-04-22
ingested_by: freddie
---

[Full transcript here, including "ums" stripped — keep substance]
```

**Step 2 — touched pages:**
- `wiki/decisions/2026-04-22-llc-state-wyoming.md` (new)
- `wiki/entities/iniminimo.md` (mention the entity formation choice)

**Step 3 — for the decision page, this is the FIRST entry, so create it:**

```markdown
---
title: Iniminimo LLC formed in Wyoming (not Delaware)
type: decision
tags: [iniminimo, legal, formation]
sources: [2026-04-22-ronen-on-wyoming-vs-delaware]
updated: 2026-04-22
status: active
---

# Decision: Iniminimo LLC in Wyoming

**Date:** 2026-04-22
**Decided by:** Ronen Freeman
**Status:** Active (LLC is filed)

## Choice
Wyoming LLC, formed via Incfile, registered agent Republic Registered Agent LLC (Casper, WY).

## Why (per [src:2026-04-22-ronen-on-wyoming-vs-delaware])
- Lower formation + annual maintenance cost than Delaware
- Stronger asset-protection statutes
- No state income tax
- Eve (sole member, US citizen) doesn't need NY/CA resident-agent overhead

## Tradeoffs accepted
- Slightly less prestige with US investors (Delaware is the default for VCs)
- Less legal precedent for edge cases

## Revisit when
- Raising institutional capital — VCs may want a Delaware C-corp conversion

## See also
- [[entity:iniminimo]]
- [[entity:eve-freeman]]
```

Decision pages are append-only when revisited. Don't overwrite — add a `## 2026-XX-XX update` section.

## Worked example 3 — A long doc / book chapter

**Source:** Cialdini's research summary I built into the influence skill earlier.

The skill itself is the wiki page. Don't duplicate it. Instead, in `wiki/topics/influence.md`:

```markdown
---
title: Influence (Cialdini-inspired persuasion framework)
type: topic
sources: [2026-04-18-cialdini-research-synthesis]
updated: 2026-04-18
---

# Influence

We use a custom skill at `~/hq/agent-skills/influence/` for persuasive writing decisions. See [src:2026-04-18-cialdini-research-synthesis] for the research synthesis the skill is built on.

The 7 principles in short: reciprocity, commitment/consistency, social proof, authority, liking, scarcity, unity. Pre-Suasion adds attention-framing.

## When to invoke
- Marketing copy
- Cold outreach
- Negotiation drafts
- Internal asks where you need behavior change

## See also
- [[topic:iniminimo-gtm]]
- [[topic:partner-outreach-playbook]]
```

The skill is the operational doc. The wiki page is the index entry pointing to it.

## Edge cases

**Source contradicts an existing wiki page.** Don't silently overwrite. Either: (a) the wiki was right and the source is wrong (note the disagreement on the source's page), or (b) the source is newer and right (update the wiki, link to the prior decision/source so the change is auditable). Lint will flag unresolved contradictions.

**Source is partly relevant.** Don't dump the whole thing into a wiki page. Cite only the parts that matter. The full source lives in `sources/` for anyone who wants depth.

**Don't know which page it touches.** Grep harder. If still nothing, you're either creating a new page (totally fine) or this source isn't wiki-worthy (also fine — not everything is).

**Multiple sources on the same fact.** Append to the `sources:` list. The page becomes more authoritative.

**The agent doing the ingest doesn't have wiki write context.** Per-agent memory and the wiki are separate. Read the SKILL.md once per session you intend to ingest in. Don't assume your private memory is the wiki.

## Quick checklist before declaring an ingest done

- [ ] Raw source saved to `sources/<date>-<slug>.md` with frontmatter
- [ ] Each touched page has the new content integrated (not just appended at the bottom)
- [ ] Each touched page has an inline `[src:<source-id>]` citation where the new info lives
- [ ] Each touched page's `updated:` is bumped
- [ ] Each touched page's `sources:` list includes the new source id
- [ ] Cross-links use `[[type:slug]]` format
- [ ] Any new entity/topic referenced has at least a stub page
