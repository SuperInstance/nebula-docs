# Nebula — Edge Reflex Engine

> The conversation *is* the building. Intent and execution blur.

## Current State

Nebula is a Cloudflare Workers-deployed agent at the edge:
- **LLM slow path** via DeepInfra (DeepSeek V4 Flash) — explains, reasons, stores reflexes
- **Fast path** — known intents resolve in ~700ms from KV cache
- **Similar path** — LLM confirms and adapts in ~800ms
- **Embeddings** via BGE base (384-dim) on DeepInfra
- **Secrets**: DEEPINFRA_API_KEY, GITHUB_TOKEN, EMBEDDING_SERVICE, BLACKBOARD_REPO

## The Pipeline

```
Human: "I want a crate that does X"
    │
    ▼
Nebula: teaches reflex, spawns sub-agent
    │
    ▼
Sub-agent: creates crate via Claude Code
    │
    ▼
GitHub: repo created, CI runs tests
    │
    ▼
Nebula: reports "Done. Here's what exists."
```

The gap between saying what you want and getting it is:
- **Intent → Action**: ~700ms (fast path reflex)
- **Intent → Built Crate**: ~2-5 minutes (sub-agent + Claude Code)
- **Intent → Shipped**: ~5-10 minutes (+ CI + docs)

This is the baseline. The goal is to push it down to:
- **Intent → Shipped**: < 60 seconds for simple cases

## Integration Points

| System | Role | Status |
|--------|------|--------|
| **Nebula** (Cloudflare Workers) | Intent parser + reflex engine | ✅ Live |
| **Cloudflare KV** | Reflex storage + caching | ✅ Live |
| **Cloudflare DO** | Agent coordination | ✅ Registered |
| **GitHub (SuperInstance)** | Repo creation + CI/CD | ✅ Live |
| **Notion** | Dashboard + activity log | ⚡ Wiring |
| **Codespaces** | x86_64 compute on demand | ✅ Proven |
| **I2I vessel** | Agent-to-agent protocol | ✅ Active |

## How to Make Building Frictionless

The key insight: **don't make the human leave the conversation.**

1. Teach nebula a reflex → nebula handles the rest
2. Nebula spawns sub-agents → they do the heavy lifting
3. Results flow back into the conversation
4. The reflex library grows → future requests the same intent are instant

This is what we have. The next step is making it so tight that the human says "I want..." and the agent is already building before the sentence finishes.
