# LLM Token & Efficiency Optimizer

> A production-grade framework and Agent Skill to minimize LLM token consumption, latency, and API costs while maximizing reasoning quality and system throughput.

[![GitHub Stars](https://img.shields.io/github/stars/Henry-19/llm-token-optimizer?style=for-the-badge&logo=github)](https://github.com/Henry-19/llm-token-optimizer)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg?style=for-the-badge)](https://opensource.org/licenses/MIT)
[![skills.sh](https://img.shields.io/badge/skills.sh-Compatible-0a0a0a.svg?style=for-the-badge)](https://skills.sh)

## ⚡ Quick Install

```bash
npx skills add Henry-19/llm-token-optimizer -g
```

---

## 🚀 Key Highlights & Solved Trade-offs

| Strategy | Core Advantage | Traditional Limitations | Solved in this Framework |
| :--- | :--- | :--- | :--- |
| **Prompt Compression** *(LLMLingua)* | 50% - 80% input token reduction | Loss of critical tokens (numbers, code, negation logic) | **Preservation Masking:** Locks critical entities (NER, digits, condition words `NOT`, `ONLY`) before compression. |
| **Semantic Caching** *(GPTCache)* | 100% token savings on recurring queries (<20ms response) | Stale data & false positive cache hits on negated questions | **Polarity Check & Dynamic TTL:** Enforces polarity verification and strict data expiration policies. |
| **Structured Output** *(Outlines)* | Eliminates token waste on markdown chatter & syntax errors | Infinite generation loops when schema contradicts LLM capability | **Graceful Timeout & Schema Fallback:** Strict token limits and automatic raw text fallback. |
| **Intelligent Routing** *(RouteLLM)* | Up to 85% cost reduction by routing simple tasks to cheap models | Incorrect classification degrades response quality | **Dual-Threshold & Escalation:** Ambiguous queries (0.4-0.6 score) are automatically escalated to stronger models. |

---

## 🏗️ 4-Layer Architecture Pipeline

```text
                  [Incoming Request]
                           │
                           ▼
               ┌───────────────────────┐
               │ Layer 1: SemCache     │ ── (Hit) ──► Instant Return (0 Tokens)
               └───────────────────────┘
                           │ (Miss)
                           ▼
               ┌───────────────────────┐
               │ Layer 2: Model Router │ ──► Flash Tier (Fast/Cheap) vs Pro Tier (Deep Reasoning)
               └───────────────────────┘
                           │
                           ▼
               ┌───────────────────────┐
               │ Layer 3: Compressor   │ ──► Context pruning with entity preservation
               └───────────────────────┘
                           │
                           ▼
               ┌───────────────────────┐
               │ Layer 4: Structured   │ ──► Schema enforcement & zero-chatter output
               └───────────────────────┘
                           │
                           ▼
               [High-Efficiency Output]
```

---

## 📂 Repository Structure

```text
llm-token-optimizer/
├── SKILL.md             # Antigravity agent operational specification & runbook
├── README.md            # Architecture overview & documentation
└── examples/            # Production patterns for compression and routing
```

---

## 🤖 Antigravity Integration

To use this as an active skill in your Antigravity Agent environment:

1. Copy or link this folder into your agents directory:
   ```bash
   # Workspace-specific:
   cp -r llm-token-optimizer /path/to/project/.agents/skills/

   # Or Global:
   cp -r llm-token-optimizer ~/.gemini/config/skills/
   ```
2. The agent will automatically detect `SKILL.md` and apply token optimization strategies whenever designing prompts or building RAG pipelines.

---

## 🔒 Git Commit & Sole Author Policy

When committing and pushing changes in this repository:
- **Sole Author Integrity:** All commits and git push actions must record exactly **one single author** (`git config user.name` and `user.email`).
- **No Co-Authors Allowed:** Strictly prohibited from using `Co-authored-by:` git trailers or attributing multiple authors.

---

## 📄 License

MIT License. Feel free to use and modify in commercial and open-source projects.
