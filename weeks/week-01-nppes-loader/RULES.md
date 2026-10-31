# Week 1 rules, with the reasoning

These are the same tables as in [BRIEF.md](BRIEF.md), plus *why* each rule exists. Keep this open while you review your AI's plan and code.

## The big idea: idempotency
A load is **idempotent** if running it twice leaves the same result as running it once. Data pipelines get rerun all the time: a job fails halfway, someone reruns yesterday's file by mistake, a scheduler retries. Every rule below exists so that a rerun is boring.

## Row validation
| NPI | Last Update Date | Deactivation Date | Outcome | Reason |
|---|---|---|---|---|
| missing, or not exactly 10 digits | any | any | reject | invalid_npi |
| valid | not a valid MM/DD/YYYY date | any | reject | invalid_date |
| valid | any | not empty and not a valid MM/DD/YYYY date | reject | invalid_date |
| valid | empty | valid or empty | accept, last_update_date = NULL | — |
| valid | valid | valid or empty | accept | — |

**Why reject instead of fixing?** A loader that quietly "repairs" bad data hides upstream problems. Rejecting with a counted reason makes data-quality problems visible.

**Why does an empty Last Update Date pass?** Missing isn't the same as wrong. The row is still useful, and the upsert rules below decide how much to trust it.

## Duplicate NPIs within one file
| Situation | Outcome |
|---|---|
| NPI appears once | keep it |
| NPI appears more than once, dates differ | keep the row with the latest last_update_date (NULL counts as oldest); count each extra as duplicate_in_file |
| NPI appears more than once, same date | keep the first occurrence; count each extra as duplicate_in_file |

**Why deduplicate in Python first?** A single Postgres `INSERT ... ON CONFLICT DO UPDATE` statement errors out if it tries to change the same row twice. Deduplicating first also makes the winner a deliberate rule rather than an accident of row order.

## Upsert into `providers`
| NPI already in table? | Incoming vs stored last_update_date | Outcome | Counted as |
|---|---|---|---|
| No | any | insert | inserted |
| Yes | incoming newer | update all columns | updated |
| Yes | equal or older | no change | unchanged |
| Yes | incoming NULL | no change | unchanged |
| Yes | stored NULL, incoming not NULL | update all columns | updated |

**Why compare dates instead of just overwriting?** Files can arrive out of order. Without the date check, loading last week's file after this week's would roll the data back.

**Why does "equal" count as unchanged?** It's what makes a rerun of the same file a no-op, which is idempotency again.

## Summary invariant
```
read = rejected + duplicate_in_file + inserted + updated + unchanged
```
**Why?** It proves every row went *somewhere*. If the numbers don't add up, a row was silently dropped or double-counted, and that's a bug even when the data looks fine.
