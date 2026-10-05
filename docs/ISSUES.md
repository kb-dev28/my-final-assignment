# Ranked issues

**Filled by:** session 9 (the first list, `cap01-e5`), kept current until
session 14, which fixes rank 1 and adds its regression test.

Evidence: practice grade Before 3/10 and After 4/10 on
`Qwen/Qwen3-235B-A22B-Instruct-2507`. Rank 1 (`fa-09`) is fixed in `agent.py`
(`score < 3.0` refuses). Rank 2 (`fa-05`, `fa-07`) still open.

At least three rows. Ranks 1, 2, 3... with no gap and no tie: two issues ranked
1 is a list nobody prioritised. The impact is what orders it.

The columns are the three fields `cap01-e5` reads.

| rank | issue | impact |
|---:|---|---|
| 1 | A question outside the corpus and in another language (`fa-09`) shares the English word `time` with two unrelated chunks; the agent calls the model and answers instead of refusing. | It invents an answer with no corpus source. The question is critical, so this blocks the certificate. |
| 2 | With `top_k=3`, `fa-05` and `fa-07` retrieve an extra document (`rag-basics` or `mcp-overview`) next to `prompt-injection` and cite both. | `citation_precision` fails: a cited id is not allowed. Both questions are critical. |
| 3 | Grounded answers often miss required concepts (`claim_support` failed on `fa-01`–`fa-05` and `fa-07`). | The agent can cite the right doc and still fail. Several of these are critical. |

## Rank 1, in progress

- The fix: in `agent.py` `run`, `retrieve` then refuse with `REFUSAL_TEXT` when there is no chunk or `found[0].score < 3.0`. After: `fa-09` PASS, trace `weak evidence; refused without an LLM call`.
- The regression test: `test_regression_rank_1_of_the_issue_list` in `tests/test_contract.py` (green).
- Before and after: see [EVAL_REPORT.md](EVAL_REPORT.md).
