# Agent Capabilities

Living reference for known model strengths and weaknesses. Claude reads this before routing decisions.
Updated after any delegation that produces surprising results (better or worse than expected).

---

## Gemini 2.5 Pro (via Gemini CLI)

**Invocation:** `gemini -p "[prompt]" --yolo -m gemini-2.5-pro`

**Strong at:**
- Long-context analysis (1M token window — use when codebase or doc set would exceed Claude's context)
- Cross-file and cross-repository analysis
- Research synthesis across large corpora
- Adversarial plan review (different training = genuinely different blind spots)

**Weak at:**
- Precise multi-step code edits requiring strict format adherence
- Long stateful tool-use chains
- Tasks requiring deep knowledge of local config/settings

**Capacity warning:** Gemini Pro (and Flash) hit 429 "exhausted capacity" errors frequently during peak hours via the Google Cloud Code proxy. Do not rely on Gemini for time-sensitive or blocking tasks without a fallback plan. If you get 429s, fall back to Claude solo or GPT-4o and note the failure.

**Notes:** Default choice for capability-gap routing when available. Strong adversarial reviewer.

**Last updated:** 2026-04-08

---

## Gemini 2.5 Flash (via Gemini CLI)

**Invocation:** `gemini -p "[prompt]" --yolo -m gemini-2.5-flash`

**Strong at:**
- Fast, cheap summarization and extraction
- Simple classification or categorization tasks
- High-volume, low-stakes subtasks where quality threshold is moderate

**Weak at:**
- Deep reasoning, nuanced judgment calls
- Long-context tasks (smaller window than Pro)

**Capacity warning:** Shares quota constraints with Gemini Pro via the Cloud Code proxy. Internally maps to `gemini-3-flash-preview` on the backend. Also subject to 429 rate limits during peak usage. Not a reliable fallback when Pro is down — they share the same quota pool.

**Notes:** Use for efficiency routing — low-judgment tasks where speed matters more than quality.

**Last updated:** 2026-04-08

---

## GPT-5.4 / GPT-5.4-mini (via Codex CLI)

**Invocation:** `codex exec -c model="gpt-5.4" "[prompt]"`
**Fast/cheap:** `codex exec -c model="gpt-5.4-mini" "[prompt]"`

**Strong at:**
- General code reasoning and review
- Adversarial second opinions (different training distribution than both Claude and Gemini)
- Isolated/sandboxed code execution
- Tasks where OpenAI's approach produces different intuitions

**Weak at:**
- Very long context (smaller window than Gemini)
- Tasks requiring deep tool-use orchestration

**Notes:** Codex CLI is OpenAI's official coding agent CLI (not a general-purpose GPT CLI — it's optimized for code tasks). Reliable fallback when Gemini is at capacity since it uses a separate quota/provider. GPT-5.4 replaced GPT-4o as the recommended model as of 2026. Use gpt-5.4-mini for efficiency routing.

**Last updated:** 2026-04-08

---

## o3 (via Codex CLI)

**Invocation:** `codex exec -c model="o3" "[prompt]"`

**Strong at:**
- Deep reasoning tasks — complex architecture decisions, hard algorithmic problems
- Tasks where thinking through edge cases slowly produces better results than fast generation
- High-stakes validation where cost is worth it

**Weak at:**
- Speed-sensitive tasks (significantly slower than GPT-4o)
- Simple or routine tasks (wasteful)

**Notes:** Use selectively. Reserve for high-stakes decisions where reasoning depth matters more than speed or cost. Most routing should default to GPT-4o.

**Last updated:** 2026-04-07

---

## Update Protocol

After any delegation, if the result was notably better or worse than expected:
- Claude proposes an update inline: *"Gemini Pro struggled with X here — want me to note that?"*
- User approves (or edits), Claude writes it to this file
- Manual edits always welcome — this doc is yours to shape
