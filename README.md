# my-final-assignment

A research assistant that answers developer questions from the six documents in
`data/corpus/`, cites the document it used, and refuses when those files do not support the question.

![check](https://github.com/kb-dev28/my-final-assignment/actions/workflows/check.yml/badge.svg)

## The problem

People ask coding-assistant questions and get fluent answers with no way to
check the source. If the corpus is silent, the model still invents. This repo
is the agent in the middle: retrieve, maybe one model call, then a citation
check or a refusal.

## Demo

Two runs, pasted exactly as the commands printed them. Never an edited one.
`trace` prints every step the agent took, then the answer.

### One supported answer

```bash
uv run bootcamp final trace "How does chunking work in RAG?"
```

```text
[retrieve] top_k=2 -> [('rag-basics', 1), ('rag-basics', 2)]
[llm_call] attempt 1: 285 chars
[decision] answered with citations ['rag-basics']

answer: Chunking in RAG splits documents into passages small enough to be individually relevant, and it is better to respect paragraph boundaries rather than cut at a fixed character count mid-sentence.
citations: ['rag-basics']
confidence: 1.0
needs_human_review: False
```

### One refusal

```bash
uv run bootcamp final trace "What is the capital city of Mongolia?"
```

```text
[decision] weak evidence; refused without an LLM call

answer: I don't know based on the provided corpus.
citations: []
confidence: 0.0
needs_human_review: True
```

## Architecture

One run is a **chain**: retrieve → refuse if the best chunk score is `< 3.0` →
otherwise one model call → drop citations the retriever did not return.
Cost: 0 or 1 model call. Decision and reversal trigger:
[docs/adr/0001-run-shape.md](docs/adr/0001-run-shape.md).

## Measured results

Every number here comes from a command in this table, run on this commit. Say
which model produced it: CI has no keys, so a CI number is always the offline
fake model's.

| What | Command | Model | Result |
|---|---|---|---|
| Contract tests | `uv run pytest` | fake | `5 passed, 1 skipped, 3 xfailed` (after the rank-1 regression test) |
| Practice grader | `uv run bootcamp final grade` | openai / Qwen/Qwen3-235B-A22B-Instruct-2507 | Before `3/10 (30%)`; after rank-1 fix `4/10 (40%)`; after `top_k=2` `5/10 (50%)` still NOT YET |
| Evaluation, before and after | see [docs/EVAL_REPORT.md](docs/EVAL_REPORT.md) | same Qwen | 30% → 40% on the threshold; `fa-09` PASS; `fa-05`/`fa-07` still FAIL |

## The honest limitation

Rank 1 of [docs/ISSUES.md](docs/ISSUES.md) (weak retrieval still calling the
model) is fixed. Rank 2 remains: `fa-05` and `fa-07` cite an extra document.
Next step: keep `top_k=3` if `top_k=2` drops `prompt-injection` (`citation_recall`
failed on fa-05 in the 5/10 run), and filter citations to the allowed retrieved id
instead of shrinking k.

## How to run it

```bash
git clone https://github.com/kb-dev28/my-final-assignment && cd my-final-assignment && uv sync && uv run pytest
```

No key needed: without a `.env` it runs on the offline fake model. For a real
model, copy `.env.example` to `.env`, fill in your provider, and
`uv sync --extra openai`.

To hand in the final assignment, commit and push, then run
`uv run bootcamp final submit --github <you>`. It runs the practice set
first, then answers the final questions and opens the pull request.
`--dry-run` shows the bundle without handing anything in.

## Rollback

A rollback is going back to a named snapshot of this repo (a **git tag**), not
a Jupyter trick. After tagging `v0.1.0`:

```text
git checkout v0.1.0 && uv run pytest
```

The previous agent answers again inside 2 minutes. Create the tag with
`git tag v0.1.0 && git push --tags` (do not copy the course example `v0.3.1`).

---

| Path | What it is |
|---|---|
| `agent.py` | The agent: `YourAgent`, the class the tests, `trace` and the grader run |
| `tests/test_contract.py` | The capstone contract, as tests (`uv run pytest -k refusal`, `-k injection`, ...) |
| `data/corpus/` | The six source documents, versioned; nothing here writes to them |
| `docs/EVAL_REPORT.md` | Numbers you produced, before and after, with the command behind each |
| `docs/SKILL.md` | A skill another assistant can load (session 10) |
| `docs/adr/0001-run-shape.md` | The architecture decision and what would reverse it (session 10) |
| `docs/RETENTION.md` | What a session remembers, and what it refuses to (session 11) |
| `docs/ISSUES.md` | The ranked issue list (session 9, kept until 14) |

Built during the Dev3Pack AI Engineering bootcamp, on the course package at
commit `81a918144aeae97db58c871c1f4e2be68cdd1bc5` of https://github.com/Gecko-Academy/dev3pack-cohort-2026-09.
