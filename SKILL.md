---
name: agent-router
description: Use when a task may be better handled by Gemini or Codex — large context, adversarial review, sandboxed execution, or efficiency routing. Also governs when to suggest delegation vs. proceed solo.
---

# Agent Router

## Overview

Claude is the primary orchestrator. Gemini CLI and Codex CLI are available for tasks where they're concretely better. This skill governs when to route, which model to use, and how to invoke them correctly.

**Always read `~/.claude/agent-capabilities.md` before routing.** It contains the live model capability index.

## Trigger Logic

```
Task arrives
  ↓
Read ~/.claude/agent-capabilities.md
Does this task match a known delegation pattern?
  ↓
HIGH confidence match  →  Delegate autonomously + notify user (see below)
PARTIAL / unsure       →  Ask: "This might suit [model] — route it?"
NO match               →  Proceed solo; propose cap doc update if outcome is notable
EXPLICIT request       →  User says "use Gemini/GPT" → skip check, delegate immediately
```

**High confidence** = task type is clearly listed in the capability doc and Claude has enough context to be certain the match holds.

## Notification (for autonomous routing)

Always tell the user before invoking a sub-agent:
> "Routing to [model] — [one-line reason]. I'll report back."

Never silently delegate. The user must know when another model is involved.

## Invocation Templates

### Gemini 2.5 Pro — long context, adversarial review, research synthesis
```bash
gemini \
  -p "[role framing + task + relevant context]" \
  --yolo \
  -m gemini-2.5-pro
```

### Gemini 2.5 Flash — fast, cheap, low-stakes subtasks
```bash
gemini \
  -p "[task]" \
  --yolo \
  -m gemini-2.5-flash
```

### GPT-4o — general reasoning, code review, adversarial second opinion
```bash
codex exec -c model="gpt-4o" "[role framing + task + relevant context]"
```

### GPT-4o (sandboxed) — untrusted or destructive execution
```bash
codex exec -c model="gpt-4o" --sandbox "[task]"
```

### o3 — deep reasoning, high-stakes decisions (use selectively — slow and expensive)
```bash
codex exec -c model="o3" "[task]"
```

## Context Packaging

When constructing the sub-agent prompt:
1. **Role frame first** — "You are an adversarial reviewer. Find the strongest arguments against this plan." or "You are analyzing a large codebase for X."
2. **Task description** — clear, specific, bounded
3. **Relevant context** — file paths the sub-agent should read (it inherits cwd), or inline content if small
4. **Output format** — tell it how to structure the response if it matters

Sub-agents inherit the current working directory — they can read files directly. Pass paths, not full file contents, for large files.

## Result Integration

Do NOT raw-dump sub-agent output. Always synthesize:
- Summarize what the sub-agent found
- For adversarial review: explicitly compare its findings against your own reasoning; flag divergences
- For capability-gap tasks: report the result as if it were your own work (with attribution)
- If the output was incomplete or malformed, say so rather than passing garbage back

## Capability Doc Updates

After every delegation, assess whether the result matched expectations:

- **Notably better than expected** → propose update: *"Gemini Pro handled X better than I expected — want me to note that in the capability doc?"*
- **Notably worse or surprising** → propose update: *"Gemini Pro struggled with Y here — want me to note that?"*
- **As expected** → no update needed
- User approves or edits, then Claude writes to `~/.claude/agent-capabilities.md`

**Update threshold:** Propose an update if the sub-agent's output materially changed your recommendation, surfaced something you missed, or failed in a way that would change which model you'd route to next time. Don't update for minor quality differences.

**On consensus in adversarial review:** If the sub-agent reaches the same conclusion you did, that's useful signal — note it briefly ("Gemini agreed with my assessment") and treat it as higher confidence to proceed. Consensus is a valid outcome; don't discount it.

This feedback loop is how the system improves over time.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Silently delegating | Always notify before invoking |
| Using wrong model flag | Copy from templates above — don't guess flags |
| Passing full file contents in prompt | Pass paths instead; sub-agent reads files from cwd |
| Raw-dumping sub-agent output | Synthesize; flag divergences on adversarial tasks |
| Routing for parallel work | Use Claude's Agent tool for parallelism, not this skill |
| Skipping cap doc update when surprised | The update loop is the point — don't skip it |
