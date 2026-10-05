# ADR 0001: the shape of one run

**Filled by:** session 10, for the choice you measured in session 8 (chain,
loop or graph, and the model calls each one cost). The four fields are the
ones `ch10-e2` reads.

- Status: accepted
- Date: 2026-10-05

## Context

Each practice question is one retrieve, then zero or one model call. Traces
show `[llm_call] attempt 1` or `refused without an LLM call`. A loop would
add extra calls per question without a measured gain on this corpus.

## Decision (`decision`)

We keep the chain in `agent.py`: retrieve → (optional) one model call → check citations.

## Options considered (`options_considered`)

1. Chain (taken): retrieve, at most one `complete`, then citation check.
2. Loop (turned down): extra model/tool rounds until a budget runs out.

## Why not the other option (`why_not`)

A loop costs more model calls per question. On the practice traces a passing
grounded answer already uses one call; refusals use zero. Extra rounds would
spend Nebius credits without a measured lift on `claim_support`.

## What would reverse it (`reverses_it`)

When more than 2 model calls are needed in 10 of 10 practice questions
(`uv run bootcamp final trace` shows `attempt 2` or a second `llm_call`)
and the practice score is still below 30% with every critical FAIL.
