# Wiki — Lint procedure

Lint catches the slow rot that kills wikis: stale claims, broken links, contradictions, missing connections. Run monthly via cron, ad-hoc when something feels off.

## What lint does (and doesn't)

Lint **surfaces** problems, it doesn't auto-fix them silently. Output is a markdown report at `~/knowledge/lint/<YYYY-MM-DD>.md` for humans (and the next agent run) to act on.

Don't auto-rewrite pages during a lint run — too easy to introduce regressions. The lint produces a diff proposal; an agent or human applies it deliberately.

## The seven lint checks

### 1. Stale pages

Find every page in `wiki/` with `status: active` and `updated:` more than 90 days ago.

```bash
find ~/knowledge/wiki -name "*.md" -type f
```

For each: parse the frontmatter, compute age in days from `updated:` to today. If > 90 and `status: active` → flag.

Output: list of stale pages with last-updated dates. Recommendation = "review and either (a) confirm still accurate and bump `updated:` to today, (b) update with new info, or (c) flip to `status: stale` if it's truly outdated and no longer load-bearing."

### 2. Broken `[src:...]` references

For each wiki page, extract every `[src:<source-id>]` reference. Verify each `<source-id>` exists as `sources/<source-id>.md`.

Output: list of (page, broken-ref) pairs. Recommendation = "find the source by date/topic and rename, or remove the citation if the source is genuinely lost."

### 3. Dangling `[[...]]` internal links

For each wiki page, extract every `[[type:slug]]` link. Verify the target file `wiki/<type>s/<slug>.md` exists. (Note: `entity` → `entities/`, `topic` → `topics/`, `decision` → `decisions/`, `glossary` → `glossary/`.)

Output: list of (page, dangling-link) pairs. Recommendation = "either create the missing stub page or fix the link."

### 4. Sources cited but not in frontmatter

For each wiki page, every `[src:X]` in the body should have `X` in the frontmatter `sources:` list. Mismatches mean either citations were added without updating frontmatter, or sources were removed but inline cites weren't.

Output: list of (page, in-body-but-not-in-frontmatter) pairs.

### 5. Sources in frontmatter but not cited in body

Inverse of #4. Less severe but worth flagging — usually means someone listed a source aspirationally without integrating it.

### 6. Contradictions across pages

Hardest check. For pages that share at least 2 tags or are linked to each other, do a semantic comparison: are there directly contradicting claims? (e.g., "iniminimo is a Wyoming LLC" on one page vs "iniminimo is a Delaware LLC" on another — these can't both be current.)

This needs an LLM pass. Don't try to do it via grep. The procedure:
- Find candidate page pairs (linked or shared tags)
- For each pair: read both, ask the model "do these directly contradict?"
- Threshold: only flag direct contradictions, not nuance differences

Be conservative. False positives waste reviewer time. Flag only when confidence is high.

### 7. Missing connections

For each page, scan the body for mentions of other entities/topics that have wiki pages but aren't `[[...]]`-linked. (e.g., the page mentions "Sara Servan" in plain text but doesn't link to `[[entity:sara-servan]]`.)

This is a usability lint, not a correctness lint. Lower priority than 1-6.

## Output format

`~/knowledge/lint/2026-05-02.md`:

```markdown
---
date: 2026-05-02
agent: <agent-that-ran-lint>
pages_scanned: 47
sources_scanned: 23
issues_found: 8
---

# Wiki Lint — 2026-05-02

## Stale pages (3)
- `topics/iniminimo-pricing.md` — last updated 2026-01-14 (108 days)
- `entities/sara-servan.md` — last updated 2026-01-22 (100 days)
- `decisions/2026-01-09-aws-region.md` — last updated 2026-01-09 (113 days)

## Broken source references (1)
- `topics/gm-l8-interview-prep.md` cites `[src:2026-04-29-jeff-conversation]` — file not found in `sources/`. Did the slug change?

## Dangling internal links (2)
- `topics/iniminimo-music-brand.md` → `[[entity:eve-freeman]]` — target file does not exist. Stub recommended.
- `decisions/2026-04-30-vm-split.md` → `[[topic:iniminimo-platform-architecture]]` — target file does not exist.

## Citation/frontmatter mismatch (1)
- `topics/iniminimo-gtm.md` — body cites `[src:2026-05-02-stripe-mrr-playbook-prelaunch]` but frontmatter `sources:` is missing it.

## Contradictions (1, high confidence)
- `entities/iniminimo.md` says "Wyoming LLC" / `decisions/2026-01-09-aws-region.md` references "Delaware C-corp". Latter is from before formation pivot — likely needs update or `status: deprecated`.

## Missing connections (2, low priority)
- `topics/influence.md` mentions "Sara Servan" without linking to `[[entity:sara-servan]]`.
- `entities/iniminimo.md` mentions "Wyoming" without linking to `[[topic:wyoming-llc-setup]]`.
```

## Cron schedule

Monthly is the right cadence. More frequent = noise, less frequent = rot accumulates faster than agents notice.

```
# Run on the 1st of each month at 09:00 Israel time
0 9 1 * * cd ~/hq && /home/ubuntu/.claude/local/claude --no-interactive run-lint
```

(Exact invocation depends on how we wire it — could be a Claude Code skill call, could be a shell script. Wire whichever is simpler.)

## When NOT to lint

- During heavy active-write periods (e.g., everyone ingesting at once during a research sprint). Lint reads the world; if it's mid-flight, false positives multiply.
- Right before a major migration. Migrate first, lint after, fix the inevitable post-migration breakages.
