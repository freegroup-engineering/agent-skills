# Unexercised controls — a defect hidden by things going well

**Rule (all agents: Nova, Cleo, Margo, Boaz, Mark + subagents). Written 2026-08-24 from a day
in which the same object turned up in five different subsystems.**

A control that has never been asked to say *no* is not a working control. It is an untested
one, and the longer it has been quiet the less you know about it.

## The tell is the absence of a symptom

You cannot find these by looking for failures, because they do not produce any. The search key
is the opposite. Ask three questions of any guard, gate, alert, or check:

1. **Has it ever fired?** A control that has never fired is evidence of nothing. Find one
   occasion where it said no, or treat it as unverified.
2. **Has anything ever been unhappy with its output?** A stream of satisfied consumers means
   the output is correct *or* nobody compares it against what was asked for.
3. **What would the world look like if it were broken?** If the answer is *"exactly like it
   looks now"*, it is untested — regardless of its age, its greenness, or how many times it
   has run.

🔴 **The corollary is the counter-intuitive half: a long clean run makes a control LESS
trustworthy, not more.** The streak is the concealment, not the reassurance.

## The evidence — one day, five instances

- **`claude-review`** turned an unparsed verdict into a green required check on 20 of 24 merges.
  It survived because twenty consecutive reviews said APPROVE. A run of `REQUEST_CHANGES` would
  have exposed it in a day — and when one did occur (PR #746, run `32624500305`) the gate
  rendered the objection as a passing check.
- **The same workflow posts no inline review comments at all** across 24 PRs, though its prompt
  asks for them. Nobody noticed *because the issue comments it posts instead are good.*
- **`layers.py`** points at a directory that does not exist on this box, so 58 of 911 published
  strings stop at the injection layer and never reach the sensitive check. The "published
  corpus is clean" result is real for 853 strings, not 911.
- **A guard's own off-switch.** The `childrensNotice` refusal (PR #756) disables itself when the
  document ships. That off-switch was mutation-tested — replace the derivation with a hardcoded
  `false` and the test must go red — because *the thing that turns a check off is the thing
  nobody tests.*
- **`has_all_required()`** can never return `True` for any user today, and nothing calls it, so
  nothing observes that.

Older members of the same family: an alert bound to a secret that never fires; a rate limiter
that fails open with no metric, so nobody can know what got through
(see the board's Redis fail-open ticket, **INI-102** — *not* INI-101).

## What to do about one

**Break it and watch.** A test that has never been red is the unfireable control one box down —
so prove the test can fail before trusting it green. Mutate the implementation, assert the test
goes red, restore.

And when fixing one, check the fix is not another: **four separate times in one day the obvious
repair for a fail-open would have created a fail-open.** Flipping `exit 0` to `exit 1` would
have blocked 20 of 24 legitimate merges and been disabled within a day. A retry loop whose
exhaustion is indistinguishable from a genuine absence reintroduces the bug behind something
that looks handled.

See also: [[agent-working-model]].
