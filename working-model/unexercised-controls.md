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
- **A hardcoded path that RESOLVES is worse than one that errors.** `layers.py:11` and
  `run_guard.py:42` both hardcoded `/home/ubuntu/myprojects/iniminimo-cleo/drafts/themes` — one
  agent's *authoring* drafts in a *different checkout*. That directory exists, so nothing ever
  failed. The harness ran cleanly and reported **2,244 strings, 107 injection hits, 80 IP hits**
  — confident numbers describing the wrong artifact, with nothing in the output naming the
  checkout. The published corpus is **911 strings, 58 injection, 1 IP**. The guard-corpus
  README's headline figures are the wrong ones, off by ~2.5x.
  *A broken tool announces itself. A tool pointed at the wrong thing returns a wrong answer
  wearing a right answer's clothes.* (Verified by execution 2026-08-24; fixed in PR #748.)
- **A guard's own off-switch.** The `childrensNotice` refusal (PR #756) disables itself when the
  document ships. That off-switch was mutation-tested — replace the derivation with a hardcoded
  `false` and the test must go red — because *the thing that turns a check off is the thing
  nobody tests.*
- **`has_all_required()`** can never return `True` for any user today, and nothing calls it, so
  nothing observes that.

Older members of the same family: an alert bound to a secret that never fires; a rate limiter
that fails open with no metric, so nobody can know what got through
(see the board's Redis fail-open ticket, **INI-102** — *not* INI-101).

## The near miss: a control exercised on the wrong axis

There is a second shape the three questions above do **not** catch, because it answers them
all correctly. A test that fires, passes legitimately, sits right next to the defect — and
cannot see it, because it asserts a different axis of the same behaviour.

PR #757, found 2026-08-24. A test named *"dismissing by the barrier is a NO, never a yes"*
asserted the **outcome** was fail-closed when a parent tapped the barrier. That was true, and it
had been passing honestly. The bug was that tapping the barrier mid-submit let the consent row
land while the outcome said `cancelled` — **the defect was in the side effect, not the outcome**,
so a test guarding the outcome could never have caught it however often it ran.

The fix replaced it with a test asserting the barrier does not resolve the gate **at all**, and
kept the old test's reasoning written above it rather than deleting it — the old assertion was
not wrong, it was narrow.

**So the extra question is: what does this control actually assert, as opposed to what does its
name suggest it covers?** The name is the risk; the body is the coverage; nothing keeps them in
sync.

Operationally, for any state-changing action: **list the axes it can move** — return value,
persisted row, network call, emitted event, navigation — and ask which of them the test actually
checks. **Fail-closed on one axis is not fail-closed.** The #757 test was fail-closed on the
outcome and silent on the row. A green test whose name describes the risk and whose body checks one
axis of it reads as coverage to everyone who does not open it. Closely related to the standing
rule about a PR that gives an existing signal a second meaning: the tests that defended the
consent close-code bug in July were each faithfully encoding the only meaning that had existed.

## Choosing a case to validate a fix

The same object appears one level up, in the case you pick to prove the repair works. A case
that the **wrong** implementation would also pass is a ritual, not a test.

> **Would the wrong implementation also pass this case?**
> If yes, it proves nothing. If no case in the data separates the right implementation from the
> wrong one, **that is the finding** — the fix cannot be validated yet, and saying so is the
> honest result.

Worked example, 2026-08-24. The `claude-review` fix keys the verdict on the run id.

PR **#744** carries a superseded `REQUEST_CHANGES` first and a current `APPROVE` last, so a
*recency-based* implementation gets it wrong. **#744 discriminates.**

PR **#757** does not, and the reason is worth following, because the first two explanations of
why were both wrong. It was initially described as a case recency passes **by luck**. Checked at
`15:33Z`, it had three `claude[bot]` comments: two stale verdicts (heads `cf7eff4c` and
`ac39ef38`) and a newest comment carrying **no verdict at all**, from the run still in flight
against the current head. So recency-over-verdicts returned a confident `APPROVE` about code no
longer under review, and plain recency landed on the no-verdict branch. **Its answer flips with
the timing of the fetch** — which makes it useless for validation in either direction, for a
different reason than the one first given.

The general lesson is not about recency. **A worked example decays**: #757's comment set changed
three times in one hour while people reasoned about it. Re-read the case before citing what it
proves.

## What to do about one

**Break it and watch.** A test that has never been red is the unfireable control one box down —
so prove the test can fail before trusting it green. Mutate the implementation, assert the test
goes red, restore.

🔴 **A control on the matcher is not a control on the corpus.** Handing a known-bad string
straight to the screening function proves only that the *matcher* speaks. It cannot catch a
collector that silently skipped files, or one pointed at the wrong tree — which is precisely the
failure that went undetected here for days. **Plant the known-positive at the top of the
pipeline and take it out again:** on 2026-08-24 a control theme dropped into the published tree
moved the result from `files=6 strings=911 sensitive=0` to `files=7 strings=913 sensitive=1`,
then was removed. That proves collection *and* screening. The harness's own four-item control
list, which passed throughout, could not have.

Generalised: **exercise the whole path the real input takes, not the last function in it.** Same
shape as reading a verdict rather than the check that reports it.

And when fixing one, check the fix is not another: **four separate times in one day the obvious
repair for a fail-open would have created a fail-open.** Flipping `exit 0` to `exit 1` would
have blocked 20 of 24 legitimate merges and been disabled within a day. A retry loop whose
exhaustion is indistinguishable from a genuine absence reintroduces the bug behind something
that looks handled.

See also: [[agent-working-model]].
