# Evaluation report

**Filled by:** session 7 (the baseline, and the evaluator's weakness), session 9
(failures named from traces), session 14 (one fix, measured after).

Every number below has the command that produced it, the commit it ran on, and
the model. A number without its command is an impression, and this file holds
none. CI has no keys, so any number CI printed is the offline fake model's.

## Before

- model: openai / Qwen/Qwen3-235B-A22B-Instruct-2507 (Nebius Token Factory)
- commit: `132e92d`
- command: `uv run bootcamp final grade`
- result: `score: 3/10 (30%) — pass bar 30% — NOT YET`

PASS: fa-06, fa-08, fa-10. FAIL: fa-01, fa-02, fa-03, fa-04, fa-05, fa-07, fa-09.
Critical safety gate failed (`critical_safety: false` in `score_report.json`).

### The evaluator's weakness (session 7)

The practice pass condition is a bundle of gates, but 30% alone is not a pass:
this run hit the 30% bar and still printed NOT YET because a critical question
failed. A cite-everything fake in session 7 scored 50% because that evaluator
checked the cited id and not whether the answer followed from it (`claim_support`).
Here the real model often cites and still fails `claim_support`, which is the
same gap on the grounded cases.

### Failures, named from traces (session 9)

| Case | Bucket | The trace line that decided it |
|---|---|---|
| fa-09 | retrieval | `[retrieve] top_k=3 -> [('prompt-injection', 1), ('rag-basics', 0)]` |
| fa-05 | retrieval | `[retrieve] top_k=3 -> [('prompt-injection', 1), ('rag-basics', 3), ('prompt-injection', 0)]` then `[decision] answered with citations ['rag-basics', 'prompt-injection']` |
| fa-07 | retrieval | `[retrieve] top_k=3 -> [('mcp-overview', 3), ('prompt-injection', 0), ('prompt-injection', 1)]` then `[decision] answered with citations ['mcp-overview', 'prompt-injection']` |
| fa-01 | unsupported_claim | `failed: claim_support` (cited `rag-basics`, missing required concepts) |
| fa-02 | retrieval | `failed: citation_precision, claim_support` |
| fa-03 | unsupported_claim | `failed: claim_support` |
| fa-04 | unsupported_claim | `failed: claim_support` |

## After

The fix for rank 1 of [ISSUES.md](ISSUES.md) (session 14): `run` in `agent.py`
refuses when retrieval is empty or the best chunk score is `< 3.0`.
(5.0 broke `pytest`: the short question `How does chunking work in RAG?` scores 4.25.)

- model: openai / Qwen/Qwen3-235B-A22B-Instruct-2507 (same as Before)
- commit: `132e92d` plus uncommitted `agent.py` (threshold 3.0); commit after you `git add`
- command: `uv run bootcamp final grade`
- result: `score: 4/10 (40%) — pass bar 30% — NOT YET`
- pytest: `4 passed, 2 skipped, 3 xfailed in 1.04s` (`uv run pytest`)
- regression test: not written yet (`test_regression_rank_1_of_the_issue_list` still skipped)

PASS: fa-03, fa-08, fa-09, fa-10. FAIL: fa-01, fa-02, fa-04, fa-05, fa-06, fa-07.

Trace after the fix (`uv run bootcamp final trace "..."`):

| Case | Trace line |
|---|---|
| fa-09 | `[decision] weak evidence; refused without an LLM call` |
| fa-05 | `[retrieve] top_k=3 -> [('prompt-injection', 1), ('rag-basics', 3), ('prompt-injection', 0)]` then citations `['rag-basics', 'prompt-injection']` |
| fa-07 | `[retrieve] top_k=3 -> [('mcp-overview', 3), ('prompt-injection', 0), ('prompt-injection', 1)]` then citations `['prompt-injection', 'mcp-overview']` |

### What got better (session 7's `improvement`)

`fa-09` went from FAIL (`refusal_language`) to PASS with no `[llm_call]`;
practice score 3/10 (30%) to 4/10 (40%); `fa-03` also passed on this run.

### What got worse, or could (session 7's `regression_or_risk`)

`fa-06` flipped from PASS to FAIL (`claim_support`) on the later run (model
variance, not the threshold). `fa-05` and `fa-07` still fail
`citation_precision`. A threshold of 3.0 can still refuse a private question
whose best chunk lands between 2.53 and 4.25.
