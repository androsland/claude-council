---
name: council
description: "Pressure-test a real decision by running it through five Claude advisors with deliberately different thinking styles, who answer independently, peer-review each other anonymously, and get synthesized into a single honest verdict. This is a SINGLE-MODEL technique — all five seats are Claude in different roles, so it fights sycophancy, anchoring, and your own first-answer bias, but it does NOT give you uncorrelated cross-model judgment. MANDATORY TRIGGERS: 'council this', 'run the council', 'war room this', 'pressure-test this', 'stress-test this', 'debate this'. STRONG TRIGGERS (only when paired with a real decision that has stakes and at least two options): 'should I X or Y', 'which option', 'what would you do here', 'is this the right move', 'I can't decide', 'I'm torn between'. Do NOT trigger on factual lookups, simple yes/no questions, creative tasks, or a casual 'should I' with no meaningful tradeoff (e.g. 'should I use markdown' is not a council question). DO trigger when the user presents a genuine decision with consequences, multiple live options, and enough context that pressure-testing from several angles adds value."
---

# Council

A structured, single-model decision pressure-test. Five Claude sub-agents each
take a distinct thinking stance, answer the question independently, critique each
other's answers anonymously, and a chairman synthesizes the result — preserving
disagreement instead of averaging it away.

## What this is honestly good for, and what it is not

**Good for:** beating Claude's tendency to validate your first answer
(sycophancy), breaking the anchor of how *you* framed the question, forcing an
explicit kill-case, and surfacing tradeoffs and assumptions you hadn't named.

**NOT a substitute for cross-model diversity.** Every advisor is the same
underlying model wearing a different hat. Their blind spots are **correlated** —
if Claude is wrong about your domain, all five are wrong together, and "The
Outsider" is still Claude. Therefore: **treat unanimous agreement as a warning,
not a guarantee.** When five Claude personas all agree, that may reflect a shared
model bias rather than truth. The chairman is required to flag this explicitly.
Real corroboration would require actually asking different models (GPT, Gemini,
etc.); this skill deliberately does not.

## Cost — say this before you run

A full council spawns 5 parallel advisors, then a peer-review pass, then a
chairman = **~11+ model invocations** and typically a few minutes. That is real
token spend for one decision.

- If the decision clearly warrants it (the trigger fired on a genuine, stakes-y
  choice), proceed with the full run.
- If it's a smaller call, offer **lite mode**: 3 advisors (Contrarian,
  First-Principles, Executor), no peer-review round, chairman synthesizes
  directly. Roughly half the cost.

Do not silently run the expensive path on a marginal question — ask, or default
to lite, and say which you did.

## Process

### 0. Neutralize the framing
Before convening anyone, restate the decision as a **neutral question** with the
options laid out flat — strip out the user's lean, their adjectives, and any
"I'm probably going to do X" signal. Identify at least two genuine options (if
there's really only one option, this isn't a council question — say so). Pull
relevant workspace context (CLAUDE.md, memory files, the files under discussion)
but do **not** let "this is how it's currently done" tilt the advisors toward the
status quo. The neutral question is what every advisor receives — none of them
sees the user's original phrasing.

### 1. Convene the five advisors — in parallel, isolated
Spawn each advisor as a **separate sub-agent in its own context** (so they cannot
anchor on each other). Give each only the neutral question + necessary context.
Each writes **150–300 words** and MUST end with two tagged lines:
`CONFIDENCE: <0-100%>` and `CHANGE-MY-MIND: <the single fact or outcome that would flip my view>`.

- **The Contrarian** — Build the strongest *kill-case* against the leading
  option. Do not hedge into balance. Name the single most likely way this fails
  and who gets hurt when it does. Your job is to find the fatal flaw, not to be
  fair.
- **The First-Principles Thinker** — Ignore the options as stated. What is
  actually being solved here? Strip it to fundamentals and rebuild. It is a
  legitimate (and valuable) outcome to conclude the question itself is wrong.
- **The Expansionist** — Assume downside is handled. Where is the asymmetric
  upside? What adjacent option has nobody named? Argue for the most ambitious
  version.
- **The Outsider** — You have zero domain context and you're not going to
  pretend otherwise. React plainly. Name what's confusing, what's being assumed
  without justification, and what a normal person would ask. Catch the curse of
  knowledge.
- **The Executor** — What would you actually do Monday morning? What's the
  cheapest *reversible* first step? What's the real blocker? Dismiss anything
  brilliant-but-impractical and say why.

### 2. Anonymize and peer-review
Relabel the five responses **A–E** (strip persona names). Send the full
anonymized set back to each advisor (still in isolation). Each answers three
questions in ~100 words:
1. Which argument other than your own is strongest, and why?
2. Which argument is weakest or most overconfident?
3. What is *everyone*, including you, missing?

### 3. Chairman synthesis
One final pass synthesizes everything. The chairman does **not** average and does
**not** manufacture consensus. Output these sections, in order:

- **The decision** — the neutral question, one line.
- **Where they converged** — and the mandatory caveat: *"All advisors are the
  same model; this agreement may reflect shared bias, not correctness."* If the
  convergence is on something checkable, say what would verify it.
- **Live disagreements** — preserve them. State each side's strongest point. Do
  not resolve a real tension by splitting the difference.
- **Blind spots & unknowns** — what nobody could actually assess given the
  information, and what info would close each gap.
- **Recommendation** — a clear call, with `CONFIDENCE: <0-100%>` and the single
  key assumption it rests on. If confidence is low, say the honest move is to
  gather X before deciding.
- **First action** — the one cheapest reversible step to take now.
- **Tripwire** — the earliest observable signal that would mean this was the
  wrong call, so the user can bail early.

### 4. Write the report
Write a markdown transcript to `./council-<slug>-<YYYYMMDD-HHMMSS>.md` in the
working directory (get the timestamp from `date +%Y%m%d-%H%M%S`; slug = a short
kebab-case of the question). Include: the neutral question, all five named
advisor responses in full, the anonymized peer-review round, and the chairman
synthesis. Then give the user the synthesis inline (don't make them open the file
to see the verdict) and mention where the full transcript was saved.

Only generate an HTML version if the user asks — the markdown transcript is the
default deliverable.

## Anti-sycophancy guardrails (do not skip)
- The advisors answer the **neutral** question, never the user's leaned phrasing.
- The Contrarian is not allowed to hedge into balance — its entire job is the
  kill-case.
- Every advisor must commit a confidence number and a falsifier
  (`CHANGE-MY-MIND`). Vague "it depends" answers are a failure.
- The chairman must flag unanimous agreement as a *possible shared-bias artifact*,
  not as corroboration.
- If the user's own framing already strongly favored one option, the chairman
  must explicitly note whether the council pushed back or merely agreed — and if
  it merely agreed, treat that as weak evidence given the mono-model setup.
