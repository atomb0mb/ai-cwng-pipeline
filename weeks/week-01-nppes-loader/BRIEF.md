# Week 1 brief: NPPES CSV → Postgres loader

## How to use this brief
1. Download an NPPES weekly file into `data/raw/` (see [data/README.md](../../data/README.md)).
2. Copy everything between the two lines below into your AI assistant, in one message.
3. Personalise the first bullet of **Context** before sending. Your background changes how the AI explains things.
4. Follow [the plan-first workflow](../../docs/ai-workflow.md). Read [RULES.md](RULES.md) for the rule tables with the reasoning behind them.
5. Don't open [FACILITATOR_NOTES.md](FACILITATOR_NOTES.md) until you've finished your plan and your log.

---

# Context
- I'm a **<your role and main languages, e.g. "backend engineer, mostly Java and SQL">** learning Python data pipelines. This is week 1 of an 8-week learning project: a Python pipeline that loads public US healthcare provider data (CMS NPPES) into Postgres. Later weeks add Airflow, Pydantic validation, LLM extraction, and evals, so keep the structure simple and easy to extend. I'm learning, so explain your reasoning in plain English as you go.
- This repo was created from a course template. Treat `docs/` and `weeks/` as read-only. Python 3.12, Postgres 16 in Docker Compose (or Podman with a compose provider), psycopg 3, pytest.
- Input: one NPPES weekly CSV I downloaded myself, located in `data/raw/`. It has ~330 columns, all quoted strings, and a header row.
- No other dependencies without asking me first.

# Goal
Load the first N rows of an NPPES file into a Postgres `providers` table so that rerunning the same file is always safe: no duplicates and no accidental overwrites with older data. Print a summary of what happened.

# Decisions (already made — don't revisit)
- Keep only these columns. Before writing any code, check these exact header names against the real file in `data/raw/`. If any differ, stop and show me the real names.
  - `NPI` → npi (primary key, text)
  - `Entity Type Code` → entity_type (nullable)
  - `Provider Organization Name (Legal Business Name)` → org_name
  - `Provider Last Name (Legal Name)` → last_name
  - `Provider First Name` → first_name
  - `Provider Business Practice Location Address State Name` → practice_state
  - `Provider Business Practice Location Address Postal Code` → practice_postal_code
  - `Healthcare Provider Taxonomy Code_1` → taxonomy_code
  - `Last Update Date` → last_update_date (date, nullable)
  - `NPI Deactivation Date` → deactivation_date (date, nullable)
- Empty strings become NULL.
- Dates in the file are MM/DD/YYYY.
- Use the stdlib `csv` module and a plain `@dataclass` for a record. No pandas, no ORM. Pydantic comes in week 2.
- `--limit` (default 1000) counts rows read from the file, before validation.
- Deduplicate within the file in Python before inserting, because Postgres `ON CONFLICT DO UPDATE` fails if one statement touches the same key twice.
- The whole load runs in one transaction. Any error rolls back everything.
- Read the connection string from the `DATABASE_URL` environment variable. Commit `.env.example`, never real credentials.
- Rejected rows are logged (count per reason, plus the first 10 examples) but not stored. A reject table comes in week 2.

# Behavior

## Row validation
| NPI | Last Update Date | Deactivation Date | Outcome | Reason |
|---|---|---|---|---|
| missing, or not exactly 10 digits | any | any | reject | invalid_npi |
| valid | not a valid MM/DD/YYYY date | any | reject | invalid_date |
| valid | any | not empty and not a valid MM/DD/YYYY date | reject | invalid_date |
| valid | empty | valid or empty | accept, last_update_date = NULL | — |
| valid | valid | valid or empty | accept | — |

## Duplicate NPIs within one file
| Situation | Outcome |
|---|---|
| NPI appears once | keep it |
| NPI appears more than once, dates differ | keep the row with the latest last_update_date (NULL counts as oldest); count each extra as duplicate_in_file |
| NPI appears more than once, same date | keep the first occurrence; count each extra as duplicate_in_file |

## Upsert into `providers`
| NPI already in table? | Incoming vs stored last_update_date | Outcome | Counted as |
|---|---|---|---|
| No | any | insert | inserted |
| Yes | incoming newer | update all columns | updated |
| Yes | equal or older | no change | unchanged |
| Yes | incoming NULL | no change | unchanged |
| Yes | stored NULL, incoming not NULL | update all columns | updated |

## Summary (printed at the end of every run)
Print: read, rejected (per reason), duplicate_in_file, inserted, updated, unchanged.
This must always hold: read = rejected + duplicate_in_file + inserted + updated + unchanged.

# Done when
- `docker compose up -d` (or `podman compose up -d`) starts Postgres, and the schema is created from `sql/001_create_providers.sql`.
- `python -m pipeline.load --file data/raw/<file>.csv --limit 1000` loads the data and prints the summary.
- Running the same command a second time prints inserted=0, updated=0, unchanged=(all valid rows).
- `pytest` passes, including:
  - Unit tests (no database): test_valid_npi_accepted, test_npi_wrong_length_rejected, test_npi_non_numeric_rejected, test_bad_update_date_rejected, test_bad_deactivation_date_rejected, test_empty_update_date_becomes_null, test_duplicate_keeps_latest_date, test_duplicate_same_date_keeps_first
  - Integration tests (real Postgres, separate `nppes_test` database, tables truncated between tests): test_first_load_inserts_all, test_rerun_same_file_changes_nothing, test_newer_date_overwrites, test_older_date_ignored, test_null_incoming_date_does_not_overwrite, test_null_stored_date_gets_updated, test_summary_counts_add_up, test_error_mid_load_rolls_back
- Tests use small fixture CSVs in `tests/fixtures/` with the real header names, not the large downloaded file. Start from the samples in `weeks/week-01-nppes-loader/fixtures/`.
- `README.md` is replaced with your project README covering setup, how to run, how to test, and both rule tables above. The course overview stays available in `docs/`. Add the setup, run and test commands to the "Project commands" section of `CLAUDE.md`.

# Out of scope (don't add these)
Airflow, outbound or delta files, a reject table, Pydantic, LLMs, cloud deployment, CI, pandas, async, parallelism, retries, a CLI framework (use argparse).

# Style
- Type hints everywhere. Plain functions. Classes only for the record dataclass.
- At most 2 levels of nesting. Functions under ~40 lines.
- Comments explain why, not what.
- Suggested layout: `src/pipeline/{parse.py, db.py, load.py}`, `sql/`, `tests/`, `tests/fixtures/`. Change it only if you explain why.

# Process
1. Restate the validation, duplicate, and upsert tables in your own words, including the rows where nothing happens.
2. Check the real CSV headers against my column list.
3. List your assumptions and open questions. "None" is suspicious here, so look hard.
4. Propose a numbered plan. For each step, list the files it touches and the tests that prove it. Suggested order:
   (1) project skeleton plus parse/validate/dedupe with unit tests, no database;
   (2) Docker Compose, schema, and database connection;
   (3) loader with upsert, counts, and the transaction;
   (4) integration tests;
   (5) README.
5. Wait for me to approve Step 1 only. After each step, show the test output, explain in 3–5 plain-English sentences what you built and why, then stop and wait for my approval before the next step.
6. If any command differs by OS, ask me which OS I'm on.

---
