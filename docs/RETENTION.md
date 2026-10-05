# Retention policy

**Filled by:** session 11. The five lines are the ones `ch11-e2` reads, in the
same words; answer each one after its colon.

STORED: nothing across questions; each `run` starts from the six corpus files only

WHY: the agent answers one question at a time from `data/corpus/`; it has no user session

CORRECTED BY: n/a; there is no stored session to clear

EXPIRES: n/a; cap is zero items

WE REFUSE TO REMEMBER: API keys, `.env` values, personal data, and any document text beyond the current question

## How the code enforces it

No memory store in `agent.py`. `test_memory_is_capped_reset_and_kept_per_user` stays skipped until a store exists.
