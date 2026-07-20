# Agent Capabilities

Living reference for known model strengths and weaknesses. Claude reads this before routing decisions.
Updated after any delegation that produces surprising results (better or worse than expected).

**Last checked:** 2026-07-20

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

**Capacity warning:** Gemini Pro (and Flash) hit 429 "exhausted capacity" errors frequently during peak hours via the Google Cloud Code proxy. Do not rely on Gemini for time-sensitive or blocking tasks without a fallback plan. If you get 429s, fall back to Claude solo or gpt-5.6-terra and note the failure.

**Notes:** Default choice for capability-gap routing when available. Strong adversarial reviewer. Gemini 3.5 Pro exists but is not yet publicly available (as of 2026-07-20, was targeted for July 17 but has slipped; enterprise Vertex preview only).

**Last updated:** 2026-07-20

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

**Notes:** Use for efficiency routing — low-judgment tasks where speed matters more than quality. Gemini 3.5 Flash is now available (see section below) and is the preferred Flash-tier model for new tasks.

**Last updated:** 2026-04-08

---

## Gemini 3.5 Flash (via Gemini CLI)

**Invocation:** `gemini -p "[prompt]" --yolo -m gemini-3.5-flash`

**Strong at:**
- Fast, cheap summarization and extraction at near-Pro quality
- High-volume, low-stakes subtasks
- Parallel agentic execution (pro-level coding proficiency at flash-tier cost)

**Weak at:**
- Deep reasoning tasks (no "Deep Think" mode — that's reserved for 3.5 Pro)
- Tasks requiring Gemini 3.5 Pro's expected 2M token context (3.5 Flash has 1M)

**Capacity warning:** Subject to the same 429 quota constraints as other Gemini models via the Cloud Code proxy. There are also known compatibility issues in some CLI versions (GitHub issues #27258, #27789) — if `gemini-3.5-flash` returns errors, fall back to `gemini-2.5-flash`.

**Notes:** Released May 19, 2026. Delivers near-Pro intelligence at Flash-tier cost and speed. Preferred over Gemini 2.5 Flash for new efficiency-routing tasks when available. Source: [Google Gemini 3.5 blog](https://blog.google/innovation-and-ai/models-and-research/gemini-models/gemini-3-5/)

**Last updated:** 2026-07-20

---

## GPT-5.4 / GPT-5.4-mini (via Codex CLI)

**Invocation:** `codex exec -m gpt-5.4 "[prompt]"`
**Fast/cheap:** `codex exec -m gpt-5.4-mini "[prompt]"`

**Strong at:**
- General code reasoning and review
- Adversarial second opinions (different training distribution than both Claude and Gemini)
- Isolated/sandboxed code execution
- Tasks where OpenAI's approach produces different intuitions

**Weak at:**
- Very long context (smaller window than Gemini)
- Tasks requiring deep tool-use orchestration

**Notes:** Codex CLI is OpenAI's official coding agent CLI (not a general-purpose GPT CLI — it's optimized for code tasks). Reliable fallback when Gemini is at capacity since it uses a separate quota/provider. GPT-5.4 is still active and not deprecated as of 2026-07-20. GPT-5.6 (Sol/Terra/Luna) is now available and preferred for new tasks — see section below. Use gpt-5.4-mini for efficiency routing when GPT-5.6 Luna isn't available. Invocation syntax updated from `-c model=` to `-m` flag.

**Last updated:** 2026-07-20

---

## GPT-5.6 Sol / Terra / Luna (via Codex CLI)

**Flagship:** `codex exec -m gpt-5.6-sol "[prompt]"`
**Balanced (recommended default):** `codex exec -m gpt-5.6-terra "[prompt]"`
**Fast/cheap:** `codex exec -m gpt-5.6-luna "[prompt]"`

**Strong at:**
- All GPT-5.4 strengths plus improved reasoning and agentic workflows
- Sol: deep reasoning, cross-repo refactors, hard algorithmic problems
- Terra: everyday coding tasks, reviews, moderate-complexity work
- Luna: completions, batch tasks, light CI, high-volume subtasks

**Weak at:**
- Very long context (still smaller window than Gemini)
- Tasks requiring deep tool-use orchestration

**Notes:** Released July 2026. Three-tier model family — Sol (flagship, deepest reasoning), Terra (balanced everyday workhorse), Luna (fastest and cheapest). `gpt-5.6` bare alias routes to Sol. Terra is recommended as the new default for general Codex routing (beats GPT-5.5 at ~half Sol's price). Sources: [OpenAI GPT-5.6 announcement](https://openai.com/index/gpt-5-6/), [Developer guide](https://www.developersdigest.tech/blog/gpt-5-6-sol-terra-luna-developer-guide)

**Last updated:** 2026-07-20

---

## o3 (via Codex CLI)

**Invocation:** `codex exec -m o3 "[prompt]"`

**Strong at:**
- Deep reasoning tasks — complex architecture decisions, hard algorithmic problems
- Tasks where thinking through edge cases slowly produces better results than fast generation
- High-stakes validation where cost is worth it

**Weak at:**
- Speed-sensitive tasks (significantly slower than standard models)
- Simple or routine tasks (wasteful)

**Notes:** Use selectively. Reserve for high-stakes decisions where reasoning depth matters more than speed or cost. Most routing should default to gpt-5.6-terra; use Sol or o3 only when reasoning depth is the bottleneck. Invocation syntax updated from `-c model=` to `-m` flag.

**Last updated:** 2026-07-20

---

## Update Protocol

After any delegation, if the result was notably better or worse than expected:
- Claude proposes an update inline: *"Gemini Pro struggled with X here — want me to note that?"*
- User approves (or edits), Claude writes it to this file
- Manual edits always welcome — this doc is yours to shape
