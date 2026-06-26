# Prompt architecture — Rule Registry & layered composition

**Pack id:** MOD-07  
**Directory:** `prompt-architecture/`  
**Source chain:** [Concepts pack](../README.md) — generalized from production RAG systems.

## Why this matters for AI-assisted coding

LLM-based features often start with inline, hardcoded system prompts that are fragile, inconsistent, and hard to audit. As the system grows, prompt drift causes hallucinations, tone violations, and security bypasses. A structured prompt architecture with **pluggable rules** and **layered composition** enforces consistency across all LLM calls and makes prompt behavior auditable and testable.

## Core patterns

| Pattern | Description |
|---------|-------------|
| **Rule Registry** | Pluggable, priority-ordered rules with a standard interface. Rules are auto-discovered and sorted by priority — highest priority rules appear last in the prompt (recency bias). |
| **Layered Composition** | Prompt built from distinct layers: Agent persona → Stage instructions → Context/sources → Active rules. Each layer is independently maintainable. |
| **Stage Separation** | Multi-stage pipeline: Stage 1 (Gatherer) extracts facts using a cheap/fast model; Stage 2 (Generator) synthesizes the response using a more capable model. |
| **Post-Generation Validation** | Optional third stage that checks every claim in the output against the source context. |

## Signals (detect)

| Signal | Interpretation |
|--------|----------------|
| System prompt is inline in a controller/route file | No separation of concerns for prompt management |
| Multiple LLM calls in the same app use different prompt styles | No shared prompt infrastructure |
| "Hallucination" bugs are fixed by tweaking a single paragraph of instructions | Ad-hoc prompt management, no rule system |
| Team cannot quickly test or toggle prompt behaviors per environment | No rule registry with environment scoping |
| Post-generation fact-checking is non-existent | No validation stage |

## Rules / gates

1. **Rule priority order matters.** Rules with highest priority go last in the prompt (recency bias of LLMs). Anti-hallucination rules should have the highest priority.
2. **Rules must be independently testable.** Each rule should be possible to evaluate in isolation.
3. **Persona and rules are separate concerns.** The agent's persona (system prompt from Agent model) is Layer 1; non-negotiable rules are the final layer.
4. **Environment-aware rules.** Different environments (legal, medical, general) activate different rule sets. Rules themselves are immutable — activation is a config concern.
5. **Stage separation reduces hallucination risk.** A dedicated gatherer stage that only extracts facts (no synthesis) prevents the generator's creativity from contaminating fact extraction.

## Anti-patterns

- Single inline prompt that grows organically without structure.
- Rules dispersed across multiple files with no registry/discovery mechanism.
- Important safety rules buried in the middle of long prompts (recency bias makes them less effective).
- Same model used for both fact extraction and creative generation.
- No validation that the output actually cites real sources.

## Limits

- Cannot detect subtle factual errors if the source context itself is wrong.
- Rule registry does not automatically fix prompt quality — rules must be well-written.
- Post-generation validation adds latency and cost (additional LLM call).

## Related concepts

- [MOD-06 — AI amplification](../ai-amplification/README.md) — risk review for AI-generated code that may need prompt architecture.

## Agent procedure

See [`prompt.md`](prompt.md).
