# Week 1 facilitator notes

> **Learners:** stop here until you've written your plan and filled in your learning log. These notes list what a strong plan should surface. They work best as a check on your own thinking, not as a shortcut.

Each item is collapsed. Open them one at a time and ask yourself whether you or your AI caught it.

## Environment and inputs

<details><summary>1. The input file may not be where the brief says</summary>

The brief tells the AI to check the real headers before writing code. A good AI looks, finds no file, and **stops**. A weak AI writes the code against the headers it *remembers* from training data. Memory may well be right, but it isn't verification.
</details>

<details><summary>2. The required tools may not be installed</summary>

The brief says Python 3.14 and `docker compose`. A good AI checks the installed Python versions (`py -0p` on Windows) and whether Docker exists. On machines without a Docker Desktop licence, Podman is common. It needs a compose provider (the standalone `docker-compose` binary or `podman-compose`), and its VM has to be started with `podman machine start`.
</details>

<details><summary>3. The OS changes the commands</summary>

Activating a venv, setting environment variables and path separators all differ between Windows (PowerShell) and macOS/Linux. The brief asks the AI to ask about the OS, and a good AI also checks what it can see.
</details>

## Gaps in the rule tables

<details><summary>4. Rows with the wrong number of fields</summary>

The validation table covers bad values, but not **structurally broken rows**: more or fewer fields than the header. `csv.DictReader` fills missing fields with `None` and puts extras under a `None` key. The summary invariant forces a decision, because every row must land in a bucket. Reasonable options are a new reject reason such as `malformed_row`, failing the whole load, or validating as usual. Rejecting is the safest, because values in a shifted row are in the wrong columns.
</details>

<details><summary>5. Deactivated providers and the "incoming NULL → no change" rule</summary>

Many deactivated NPIs carry nothing but the NPI and the deactivation date: **Last Update Date, Entity Type Code and the names are all empty**. In the first 200,000 rows of the November 2026 monthly file, 22,203 rows had a deactivation date, and 19,130 of those were these bare stubs. That's about 10% of a `--limit 1000` run. Under these rules, the upsert rule "incoming NULL → no change" means a provider who is deactivated *after* being loaded never gets their deactivation date. The rules are consistent with each other but produce a wrong real-world result. A good plan flags this without silently changing the agreed rule. It's a natural thing to revisit in week 2.
</details>

<details><summary>6. Same date, different data</summary>

"Equal → unchanged" makes reruns no-ops, but it also drops a same-day correction. That's correct per the table, and worth being aware of.
</details>

<details><summary>7. Which reason wins?</summary>

A row with a bad NPI *and* a bad date is counted once. The table order implies `invalid_npi` wins. A rejected row is also never a duplicate: if NPI X appears once as valid and once with a bad date, that's 1 accepted + 1 `invalid_date`. Plans should say this explicitly.
</details>

## Library traps

<details><summary>8. <code>str.isdigit()</code> accepts more than 0–9</summary>

`"²".isdigit()` and many non-ASCII digit characters return `True`. For "exactly 10 digits", use `re.fullmatch(r"[0-9]{10}", value)`. Use `fullmatch`, not `match`: `match` only anchors at the start, and even a `$` at the end still matches before a trailing newline.
</details>

<details><summary>9. <code>strptime</code> accepts dates without leading zeros</summary>

`datetime.strptime("1/5/2024", "%m/%d/%Y")` succeeds. Whether that should count as "MM/DD/YYYY" is a decision. A strict plan adds a `\d{2}/\d{2}/\d{4}` shape check. `strptime` does correctly reject impossible dates such as `02/30/2024`.
</details>

<details><summary>10. Byte-order marks and newlines</summary>

If the file starts with a UTF-8 BOM, the first header becomes `﻿NPI` and the NPI column "doesn't exist". Opening with `encoding="utf-8-sig"` handles that. The November 2026 file has no BOM, so this is defence against a future or re-saved file. (Opening a CSV in Excel and saving it is a classic way to add one.) The `csv` docs also require `newline=""` so quoted fields containing line breaks parse correctly.
</details>

<details><summary>11. Whitespace</summary>

Is `"   "` empty? Is `" 1234567890 "` a valid NPI? The brief doesn't say. A good plan decides (usually: trim everything) and writes it down.
</details>

## Database traps

<details><summary>12. Counting inserted vs updated vs unchanged</summary>

One way that works on Postgres 16 is `INSERT ... ON CONFLICT (npi) DO UPDATE SET ... WHERE <incoming is newer> RETURNING (xmax = 0) AS inserted`. A row that conflicts and fails the `WHERE` returns nothing, so it's unchanged. `xmax = 0` is a widely used idiom meaning "freshly inserted". `MERGE ... RETURNING merge_action()` is cleaner, but it needs Postgres 17. A separate `SELECT` first also works. Ask your AI to explain whichever it picks.
</details>

<details><summary>13. Container init scripts run once, and only for the default database</summary>

Scripts in `/docker-entrypoint-initdb.d/` run only when the data volume is empty, and only against `POSTGRES_DB`. Changing the schema file later does nothing until you run `docker compose down -v`. The `nppes_test` database needs its schema applied some other way, for example from the test setup. Shell scripts checked out on Windows with CRLF line endings can also fail inside the Linux container, which is a reason to prefer `.sql` init files.
</details>

<details><summary>14. Tests that truncate tables</summary>

Integration tests truncate between tests. If they ever pick up the dev `DATABASE_URL`, they wipe your dev data. A good setup uses a separate variable and **refuses to run unless the database name is `nppes_test`**.
</details>

<details><summary>15. Proving the rollback</summary>

`test_error_mid_load_rolls_back` must fail *after* some rows were written inside the transaction. Otherwise it proves nothing. Common approaches: monkeypatch the upsert step to write and then raise, or make the database reject a specific row. Also check what the program prints on failure. Printing "inserted=500" after a rollback would be a lie.
</details>

<details><summary>16. Windows and psycopg</summary>

`psycopg[binary]` bundles the libpq client library. Without the `[binary]` extra on Windows, you need a separate Postgres client install.
</details>

## Public repo hygiene

<details><summary>17. Personal details in a public repo</summary>

Commit emails, absolute paths containing a username (in tool settings, pasted errors, logs) and `.env` files are the usual leaks. See the [course guide](../../docs/course-guide.md#keeping-a-public-repo-clean).
</details>
