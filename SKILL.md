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

### Gemini (auto) — default; system picks best available model
```bash
gemini \
  -p "[role framing + task + relevant context]" \
  --yolo
```
Omit `-m` entirely — `auto` is the configured default in `~/.gemini/settings.json`. This routes to the best Gemini 3 model available without hitting Pro capacity limits as aggressively.

### Gemini Pro — when you explicitly need the most capable model
```bash
gemini \
  -p "[role framing + task + relevant context]" \
  --yolo \
  -m gemini-3-pro-preview
```
Use only when task clearly requires Pro reasoning. Expect 429s during peak hours.

**Codex model pins:** the ChatGPT-account login rejects `-c model="gpt-5.4"` ("not supported when using Codex with a ChatGPT account" — re-confirmed 2026-10-10). Omit `-c model=` and use the CLI default from `~/.codex/config.toml` (`gpt-6-astra` as of 2026-10-10). Other pins below are UNVERIFIED on this account.

### Codex default — general reasoning, code review, adversarial second opinion
```bash
codex exec --skip-git-repo-check "[role framing + task + relevant context]"
```

### Codex (sandboxed) — untrusted or destructive execution
```bash
codex exec --sandbox read-only "[task]"
```

### Codex image generation — illustrated images with accurate text
```bash
codex exec --skip-git-repo-check "Use your image generation tool to create ONE image: [style]. Text must appear EXACTLY as written: [copy]. Copy the image into the current working directory as [name].png and print its path."
```
Attach a reference image with `-i <file>` **after** the prompt and redirect `< /dev/null` — `-i` takes multiple values, so a prompt placed after it is read as a second image path and Codex hangs on "Reading prompt from stdin..." (hit 2026-10-10). A reference image plus "change ONLY the text" kept the art nearly identical. Run from the output directory. About 1 minute per image; 4 parallel runs worked. Originals also land in `~/.codex/generated_images/`. Verify every line of text by viewing the output.

### o3 — deep reasoning, high-stakes decisions (use selectively — slow and expensive; UNVERIFIED on the ChatGPT account)
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

## Failure Handling

Sub-agents can fail. Handle gracefully:

- **429 / capacity exhausted** → Gemini Pro and Flash share a quota pool via the Cloud Code proxy. If Pro is down, Flash is often down too. Fall back to GPT-4o (different quota) or proceed Claude-solo. Always tell the user: *"Gemini is at capacity — proceeding solo / falling back to GPT-4o."*
- **Explicit request fails** → Still tell the user. Don't silently continue as if delegation happened.
- **Fallback priority:** Gemini Pro → GPT-4o → Claude solo. Never silently eat a failure.

Note capacity failures in the capability doc — they're signal.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Silently delegating | Always notify before invoking |
| Using wrong model flag | Copy from templates above — don't guess flags |
| Passing full file contents in prompt | Pass paths instead; sub-agent reads files from cwd |
| Raw-dumping sub-agent output | Synthesize; flag divergences on adversarial tasks |
| Routing for parallel work | Use Claude's Agent tool for parallelism, not this skill |
| Skipping cap doc update when surprised | The update loop is the point — don't skip it |
| Assuming Flash works when Pro is down | They share quota — treat both as unavailable together |
| Silently eating sub-agent failures | Always surface failures to the user with the fallback taken |
