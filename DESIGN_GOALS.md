# agent-router — Design Goals

This document captures the original intent and explicit scope of the agent-router skill.
It exists to prevent intent creep during future iterations. When modifying SKILL.md, check here first.

## Why This Skill Exists

Claude is the primary orchestrator on this machine, but Gemini CLI and Codex CLI are available and genuinely capable of things Claude handles less well. This skill gives Claude systematic awareness of when and how to use them — instead of always proceeding solo.

## Use Cases (Priority Order)

1. **Validation & Adversarial Check** — Get a second opinion from a model with different training before committing to major decisions. Different models have different blind spots. Two perspectives beat one.

2. **Capability Gaps** — Route to models that are concretely better at specific task types (Gemini's 1M context, Codex's sandbox). The most strategically important use case because it includes *ongoing discovery*: as we learn what other models do better, we update `agent-capabilities.md` and the routing improves.

3. **Efficiency** — Route low-judgment subtasks to faster/cheaper models so Claude stays focused on high-judgment work.

## What Success Looks Like

- Claude proactively routes to a better-suited model when confidence is high, with a brief notification
- Claude asks before routing when unsure
- After any delegation, Claude proposes a capability doc update if the result was surprising
- The capability doc grows more accurate over time through use
- User can explicitly invoke `/gemini` or `/gpt` and get clean, immediate delegation

## Explicit Non-Goals

- **Automated benchmarking** — we discover capabilities through real use, not synthetic tests
- **Parallel workstreams** — Claude's Agent tool handles that natively; don't route here for parallelism
- **Direct API calls** — only installed CLIs (gemini, codex); no raw HTTP
- **Cross-agent state sharing** — sub-agents work from shared cwd; no explicit state handoff
- **Replacing Claude** — routing supplements Claude's judgment, never replaces it

## Scope Gate

If a proposed change to SKILL.md would:
- Add benchmarking or periodic probing logic → out of scope
- Add parallel task distribution → out of scope (use Agent tool)
- Expand to non-CLI model access → out of scope
- Reduce the capability doc update feedback loop → pushback required

If unsure whether a change is in scope, read this file first.
