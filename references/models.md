# Model Facts — Volatile Reference

Model-specific facts (IDs, parameters, behavioral quirks) live HERE, not in SKILL.md. SKILL.md holds evergreen technique guidance; this file holds everything that rots.

## Refresh Protocol

Each vendor section carries a `last-verified` date and source. **Before asserting any fact from a section older than 60 days, re-verify it**:

- **Anthropic / Claude** → the `claude-api` skill (authoritative, ships current model IDs and params)
- **Libraries / dev-tool APIs** → context7
- **Everything else** → web search: `"<vendor> current models <month> <year>"`

After verifying, update the section and its `last-verified` date. Never present an expired fact as current — say "as of <date>" or verify first. Sections marked `UNVERIFIED` were carried from upstream v1.6.0 and have never been independently checked.

---

## Anthropic / Claude

`last-verified: 2026-06-10` · source: claude-api skill (cache 2026-05-26)

| Model | ID | Context | Output | Notes |
|---|---|---|---|---|
| Claude Fable 5 | `claude-fable-5` | 1M | 128K | Top tier, above Opus. $10/$50 per MTok |
| Claude Opus 4.8 | `claude-opus-4-8` | 1M | 128K | Current Opus. $5/$25 |
| Claude Opus 4.7 | `claude-opus-4-7` | 1M | 128K | Previous gen |
| Claude Sonnet 4.6 | `claude-sonnet-4-6` | 1M | 64K | Speed/intelligence balance. $3/$15 |
| Claude Haiku 4.5 | `claude-haiku-4-5` | 200K | 64K | Fast/cheap. $1/$5 |

**API params (matter when the prompt is for API/scripted use):**
- Fable 5 / Opus 4.8 / 4.7: adaptive thinking ONLY — `thinking: {type: "adaptive"}`. `budget_tokens` returns 400. `temperature` / `top_p` / `top_k` removed (400). Fable 5 additionally 400s on explicit `thinking: {type: "disabled"}` — omit the param instead.
- Last-assistant-turn prefills return 400 on Fable 5 and the entire 4.6+ family — use `output_config.format` (structured outputs) instead.
- Effort levels: `low | medium | high | xhigh | max`. `xhigh` is the Claude Code default; start at `high` for API work and sweep.
- `thinking.display` defaults to `"omitted"` on 4.7+ — reasoning text streams empty unless set to `"summarized"`.

**Behavioral (Opus 4.8 / Fable 5 era — affects prompt wording):**
- Literal instruction following: missing context = narrow output, not a smart guess. Front-load everything in one turn.
- 4.8 **narrates more** than 4.7 (interim updates, long wrap-ups). Remove "summarize after every N steps" scaffolding; if too chatty, add a silence-default: *"Default to silence between tool calls. Only write text when you find something, change direction, or hit a blocker — one sentence each."*
- 4.8 is **more deliberate — asks more often**. Add: *"For minor choices (naming, defaults, equivalent approaches), pick a reasonable option and note it rather than asking. For scope changes or destructive actions, still ask first."*
- 4.8 **under-reaches for tools, subagents, search, and memory**. State trigger conditions explicitly — in the system prompt AND each tool's description: *"Call this when…"*. For research: *"For questions where current information would change the answer, search before answering rather than answering from memory."*
- Do NOT add "think step by step" or thinking budgets — adaptive thinking calibrates itself. To influence depth: "Think carefully before responding" (more) / "Prioritize responding quickly" (less).
- Opus 4.x over-engineers — add "Only make changes directly requested. Do not add features or refactor beyond what was asked."

---

## OpenAI / ChatGPT

`last-verified: 2026-06-10` · source: web (openai.com release notes, TechCrunch)

- **All active ChatGPT models are GPT-5 family** (consolidation 2026-02-13). The GPT-4 series and the entire o-series are retired from ChatGPT (o3 fully retires 2026-08-26; there is no o4).
- Current lineup: **GPT-5.5 Instant** (default, released May 2026, low-latency), **GPT-5.4 Thinking** (paid reasoning), **GPT-5.4 Pro** (research-grade). GPT-5.2 and GPT-5.3-Codex are sunset.
- If the user says "o3" / "o4-mini": route them to GPT-5.4 Thinking and apply reasoning-model rules.
- **GPT-5.x Instant**: smallest prompt that achieves the goal; explicit output contract (format, length, "done"); dense structured instructions are fine; constrain verbosity explicitly when needed.
- **GPT-5.x Thinking / Pro**: treat as reasoning-native — SHORT clean instructions, NO CoT or reasoning scaffolding (degrades output), zero-shot first, state what you want and what done looks like, keep system prompts lean.

---

## Google / Gemini

`last-verified: 2026-06-10` · source: web (blog.google, ai.google.dev)

- Current: **Gemini 3.1 Pro** (`gemini-3.1-pro-preview`). `gemini-3-pro-preview` was discontinued 2026-03-26 — don't target it. (Gemini 3.5 Pro rumored, not shipped as of verification.)
- 1M context, strong multimodal (text/audio/image/video/PDF/repos) — leverage for document-heavy prompts.
- `thinking_level` parameter controls reasoning depth (relative guideline, not token guarantee) — set "high" for complex tasks. `media_resolution` trades fine-detail reading vs token cost.
- Tuned concise and direct; **may guess when information is missing** — for grounded tasks always add: "Base your response only on the provided context. Do not extrapolate. If information is missing, say so."
- Still prone to hallucinated citations — add "Cite only sources you are certain of. If uncertain, say [uncertain]."
- Can drift from strict formats — use explicit format locks with a labelled example.

---

## DeepSeek

`last-verified: 2026-06-10` · source: web (api-docs.deepseek.com, MIT Tech Review)

- Current: **DeepSeek V4** (launched 2026-04-24). `deepseek-v4-pro` = larger, coding/complex agents; `deepseek-v4-flash` = faster/cheaper.
- **R1 is superseded** — V4's optional thinking mode covers the R1 use case. Legacy API names `deepseek-chat` and `deepseek-reasoner` are fully retired 2026-07-24; use the v4 IDs.
- V4 thinking mode is reasoning-native: short clean instructions, no CoT. Unlike R1, **V4 supports tool calls inside thinking mode**.
- Over OpenRouter, verify the provider route honors thinking/tool settings (some providers silently disable tools — check before relying on agentic prompts).

---

## Moonshot / Kimi

`last-verified: 2026-06-10` · source: web (HuggingFace, moonshotai GitHub)

- **Kimi K2.5** (Jan 2026): 1T-param MoE (32B active), native multimodal, strong agentic/tool use and structured output. OpenRouter ID `moonshotai/kimi-k2.5`. Responds well to explicit role assignment + clear output format specs.
- **Kimi K2.6** (Apr 2026): open-weight 1T, ties GPT-5.5 on SWE-Bench Pro. Caveat: community-verified tool-call leak bug on sparse toolsets (hermes-agent #24534) — for tool/agent prompts on small toolsets, prefer K2.5 or re-verify before shipping.

---

## Alibaba / Qwen

`last-verified: 2026-06-10` · source: web

- **Qwen3** family (0.6B–235B MoE, 256K ctx, Apache 2.0): two modes — thinking mode = reasoning-native (short instructions, no CoT, no scaffolding); non-thinking = full structure, explicit format, role assignment.
- **Qwen 3.5 / 3.6** are newer with stronger coding; same two-mode prompting rules apply.
- **Qwen2.5 instruct / qwen2.5-coder** still widely deployed locally (Ollama): excellent instruction-following and JSON output; clear system-prompt role; shorter focused prompts beat long complex ones.

---

## MiniMax

`UNVERIFIED — carried from upstream v1.6.0; re-verify before asserting`

- M2.7: OpenAI-compatible API, 1M context, strong instruction following / structured output. M2.5-highspeed: 204K context, latency-optimized.
- Temperature must be in (0, 1] — above 1 fails. May emit `<think>` tags — add "Output only the final answer, no reasoning tags." if unwanted.
- Function calling: OpenAI-style tool definitions.

---

## Local / Ollama

Evergreen guidance (model-dependent — ALWAYS ask which model is running):

- System prompt is the highest-leverage knob — include it so the user can set it in their Modelfile.
- Shorter, flatter prompts; local models lose coherence with deep nesting.
- Temperature 0.1 for coding/deterministic, 0.7–0.8 creative.
- Coding: qwen2.5-coder / a coder variant, not general-chat models.

---

## Image / Video / Voice / 3D Tools

Version claims for these tools (Midjourney `--v`, SD checkpoints, Sora/Runway/Kling capabilities) are `UNVERIFIED — carried from upstream v1.6.0`. Technique guidance in `references/tools-longtail.md` is largely evergreen; verify version flags via web search before asserting them.

---

## Reasoning-Native List (for the hard rule)

Never add CoT / "think step by step" / reasoning scaffolding to: **GPT-5.4 Thinking & Pro · DeepSeek V4 thinking mode · Qwen3+ thinking mode · Claude adaptive thinking (4.6+/Fable) · legacy o-series & R1** (retired, but users may still target them). They reason internally; CoT degrades output.
