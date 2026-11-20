---
name: llm-token-optimizer
description: >-
  Comprehensive guide and operational runbook for optimizing LLM token consumption, latency, and cost
  while maximizing model productivity and reasoning quality. Use this skill when designing prompts,
  implementing RAG/agent workflows, integrating semantic caching, configuring structured outputs,
  or routing tasks across model tiers.
---

# LLM Token & Efficiency Optimizer (SKILL.md)

This skill provides an enterprise-grade architecture and standardized operational procedures to minimize LLM token consumption, reduce API costs, and accelerate inference throughput while systematically mitigating the inherent limitations of isolated optimization techniques.

---

## 1. 4-Layer Optimization Pipeline

Every incoming user or system prompt request passes sequentially through four optimization checkpoints:

```text
[User / System Request]
          │
          ▼
┌─────────────────────────┐
│ Layer 1: Semantic Cache │ ─── (Hit: Similarity >= Threshold & Valid TTL) ───► Instant Return (0 Tokens)
└─────────────────────────┘
          │ (Miss)
          ▼
┌─────────────────────────┐
│ Layer 2: Model Routing  │ ─── Evaluate task complexity and risk profile
└─────────────────────────┘
          │
          ├── Routine / Standard ────► Route to Fast Tier (Flash / Mini / 8B)
          └── Complex / Reasoning ───► Route to Strong Tier (Pro / Sonnet / 70B)
          │
          ▼
┌─────────────────────────┐
│ Layer 3: Context & Comp │ ─── Compress RAG documents, extract salient entities,
└─────────────────────────┘     strip boilerplate, markup, and low-information tokens
          │
          ▼
┌─────────────────────────┐
│ Layer 4: Structured Out │ ─── Enforce strict JSON Schema / grammars; eliminate conversational fluff
└─────────────────────────┘
          │
          ▼
[High-Quality Minimal-Token Output]
```

---

## 2. Core Trade-offs & Limitation Mitigations

| Strategy | Primary Benefit | Inherent Limitation | Systematic Mitigation in this Skill |
| :--- | :--- | :--- | :--- |
| **Prompt Compression** *(LLMLingua)* | Reduces input context tokens by 50% - 80%; expands effective context capacity. | Risk of pruning critical numbers, code snippets, legal entities, or negation logic. | **Preservation Masking:** Pin and protect critical entities (named entities, digits, equations, conditional operators `NOT`, `ONLY`, `BEFORE`) prior to compression. Only compress decorative background prose. |
| **Semantic Caching** *(GPTCache)* | 100% token savings on repeated or semantically equivalent queries; response latency < 20ms. | Serving stale data, or false positive cache hits on questions containing subtle negation words. | **Strict Polarity Filtering + Dynamic TTL:** Enforce polarity checks (detecting words like `not`, `without`, `except`) before confirming a cache hit. Attach TTLs keyed to underlying data volatility. |
| **Structured Output** *(Outlines/Guidance)* | Eliminates token waste on markdown chatter and conversational filler; ensures 0 parse retries. | Risk of infinite generation loops when schema constraints contradict model reasoning capabilities. | **Graceful Timeout & Schema Fallback:** Configure explicit `max_tokens`; keep schemas as simple as possible; provide automatic fallback to raw text parsing on timeout. |
| **Intelligent Routing** *(RouteLLM/LiteLLM)* | Slashes API costs by 70% - 85% by delegating simple tasks to low-cost models. | Router misclassifications can send complex prompts to weak models, degrading output quality. | **Dual-Threshold & Escalation Policy:** Queries falling into the ambiguity zone (score 0.4 - 0.6) automatically escalate to the stronger model. If a smaller model yields low confidence or formatting errors, automatically escalate. |

---

## 3. Token-Saving Prompt Engineering Rules

When constructing prompts or designing agent instruction files, strictly enforce the following rules:

1. **Eliminate Politeness and Conversational Fluff:**
   * ❌ *Avoid:* "Please be so kind as to read through the following text carefully and if possible let me know..."
   * ✅ *Use:* "Task: Extract security vulnerabilities from the code below. Format: concise bullet points."

2. **Use Concise Delimiters and Markdown:**
   * Use triple backticks (```) or compact XML tags (`<ctx>`, `<req>`) rather than verbose prose transitions.

3. **Enforce Zero-Chatter Output:**
   * Always append explicit output formatting directives:
     `Return raw JSON only. No markdown formatting, no explanations, no prefix/suffix.`

4. **Minimal Few-Shot Examples:**
   * Rather than 5 lengthy examples, provide 1–2 ultra-concise edge-case examples that illustrate the exact boundary constraints.

---

## 4. Git Commit & Push Attribution Policy (Sole Author Enforcement)

When developing, updating, or pushing skills, documentation, or code within this repository:

1. **Sole Author Requirement:**
   * Every git commit and push operation MUST record strictly **one single author**.
   * The author identity must match the repository owner / primary developer configured in `git config user.name` and `git config user.email`.

2. **Absolute Prohibition of Co-Authors:**
   * **STRICTLY PROHIBITED:** Appending `Co-authored-by:` trailers, co-author metadata, or multiple author signatures in git commit messages or git push payloads.
   * Never credit secondary automated agent personas or auxiliary users as git co-authors.

3. **Clean Commit Messages:**
   * Keep commit messages focused, atomic, and free of extraneous authorship trailers or automated tool signatures.

---

## 5. Production Runbook (Python Workflow)

### A. Context Compressor with Preservation Masking
```python
def compress_context_safely(long_text: str, protected_terms: list[str]) -> str:
    """
    Compresses background context while preserving essential entities (numbers, IDs, keywords).
    """
    # 1. Mask and isolate sensitive / critical tokens
    placeholders = {}
    for i, term in enumerate(protected_terms):
        tag = f"__PROTECTED_{i}__"
        placeholders[tag] = term
        long_text = long_text.replace(term, tag)

    # 2. Apply compression logic (or invoke a lightweight compressor / line trimmer)
    compressed = " ".join([line.strip() for line in long_text.splitlines() if line.strip()])

    # 3. Restore protected entities verbatim
    for tag, original in placeholders.items():
        compressed = compressed.replace(tag, original)
        
    return compressed
```

### B. Task Router with Escalation Policy
```python
def route_model(prompt: str, task_type: str) -> str:
    """
    Routes requests to optimal model tier based on task complexity and prompt size.
    """
    # Tasks requiring multi-step logical deduction, architectural design, or security audits
    HEAVY_TASKS = {"architecture_design", "security_audit", "complex_reasoning"}
    
    if task_type in HEAVY_TASKS or len(prompt) > 8000:
        return "pro-tier"  # e.g., Claude 3.5 Sonnet / Gemini Pro
    
    # Simple classification, data extraction, translation, or basic summarization
    return "flash-tier"    # e.g., Gemini Flash / GPT-4o-mini
```

---

## 6. Pre-Request Optimization Checklist

Before dispatching any prompt request to an LLM:
- [ ] Has this query or semantic equivalent hit the Cache? Was polarity checked?
- [ ] Does this task genuinely require a flagship tier model, or is a Flash tier sufficient?
- [ ] Have extraneous HTML tags, CSS, repeated headers, and whitespace been pruned from RAG documents?
- [ ] Has a zero-chatter directive been attached to eliminate conversational filler?
- [ ] Has `max_output_tokens` been configured to prevent infinite hallucination loops?
- [ ] If committing/pushing changes, is the commit configured with exactly one sole author (no `Co-authored-by`)?
