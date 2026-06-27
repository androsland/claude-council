# Council

A Claude Code skill that pressure-tests a real decision by running it through
five Claude advisors with deliberately different thinking styles. They answer
independently, peer-review each other anonymously, and a chairman synthesizes a
single verdict that **preserves disagreement instead of averaging it away**.

> Inspired by Andrej Karpathy's "LLM Council" idea — but honest about one thing
> that most clones bury: **this is a single-model technique.**

## What it actually does (and doesn't)

The original LLM Council idea gets its power from *model diversity* — different
models (GPT, Gemini, Claude…) have uncorrelated blind spots, so they catch each
other's mistakes. This skill runs entirely inside Claude Code with **one model in
five roles**. That means:

- ✅ It fights **sycophancy** (Claude validating your first answer), **anchoring**
  (your framing steering the result), and surfaces tradeoffs and hidden
  assumptions you hadn't named.
- ❌ It does **not** give you uncorrelated cross-model judgment. All five seats
  share the same blind spots. When they all agree, that can be shared model bias,
  not truth — so the chairman is required to flag unanimity as a *warning*, not a
  guarantee.

If you want true cross-model corroboration, you need to wire in other providers'
APIs/CLIs — which this skill deliberately does not do, so it needs no API keys.

## The five advisors

| Advisor | Job |
|---|---|
| **Contrarian** | Build the strongest kill-case. Find the fatal flaw, don't be fair. |
| **First-Principles** | Ignore the stated options. What's actually being solved? |
| **Expansionist** | Assume downside is handled — where's the asymmetric upside? |
| **Outsider** | Zero domain context. Name what's confusing or assumed. |
| **Executor** | What would you do Monday? Cheapest reversible first step? |

Each advisor commits a **confidence score** and a **falsifier** ("what would
change my mind"). The chairman produces: convergence (with bias caveat), live
disagreements (preserved), blind spots, a recommendation with its key assumption,
a cheapest-reversible first action, and a **tripwire** — the earliest signal that
you chose wrong.

## See it in action

[**EXAMPLE.md**](./EXAMPLE.md) is a real, unedited run on "rewrite the monolith
into microservices, or modularize in place?" — all five advisors, the anonymized
peer-review round, and the chairman synthesis (including how it flags the
five-way agreement as *possible shared bias* rather than corroboration).

## Cost

A full run is ~11+ model invocations (5 advisors → peer review → chairman) and a
few minutes. For smaller calls the skill offers **lite mode**: 3 advisors, no
peer-review round, ~half the cost. It won't silently run the expensive path on a
marginal question.

## Triggers

Say **"council this"**, "pressure-test this", "stress-test this", "war room
this", or "debate this". It also fires on real decisions phrased like "should I X
or Y", "which option", "I'm torn between" — but **not** on factual lookups,
yes/no questions, or trivial choices.

## Install

**Recommended — via the [skills.sh](https://www.skills.sh) CLI** (works with Claude
Code, Codex, Cursor, Cline, and 15+ other agents):

```bash
npx skills add androsland/claude-council
```

**Or manually for Claude Code:**

```bash
git clone https://github.com/androsland/claude-council ~/.claude/skills/council
```

Either way, the skill auto-invokes on the triggers above, or you can run it
explicitly with `/council`.

## Credits

- The **LLM Council** pattern is Andrej Karpathy's idea.
- The earlier [`tenfoldmarc/llm-council-skill`](https://github.com/tenfoldmarc/llm-council-skill)
  popularized it as a Claude Code skill and inspired this rewrite. The main
  difference here is honesty about the single-model limitation: agreement among
  five Claude personas is treated as a possible shared-bias signal, not as
  cross-model corroboration.

## License

MIT — see [LICENSE](LICENSE).
