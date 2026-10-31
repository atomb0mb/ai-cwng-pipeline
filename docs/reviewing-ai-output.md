# Reviewing AI output

AI-written code usually *looks* right. These checks are for the ways it's actually wrong. Work through them on every PR.

## Behaviour
- [ ] **Walk the rule table row by row.** For each row, find the line of code that implements it and the test that proves it. A row with no test is a row that isn't really implemented.
- [ ] **Check the "nothing happens" rows.** Are there tests that prove an update *didn't* happen, or that a record was *left alone*?
- [ ] **Precedence.** When two rules could both apply (for example, two different reasons to reject), which wins, and is that what the spec says?
- [ ] **Invariants.** If the spec says counts must add up, is that tested on input where every bucket is non-zero?

## Edge inputs
- [ ] Empty strings, whitespace-only strings, missing columns, extra columns.
- [ ] Values that are *almost* valid: off by one character, wrong separator, leading zeros dropped, impossible dates such as February 30.
- [ ] Unicode and encodings: byte-order marks, non-ASCII characters, line breaks inside quoted fields.
- [ ] **Library leniency.** Standard functions often accept more than your rule allows. For example, Python's `int()` accepts `" 42 "` and `"4_2"`. Whenever validation uses a library call, ask: "what else does this accept?" Then try it in a REPL.

## Tests
- [ ] **Would the test fail if the code were wrong?** Break the code on purpose (flip a `>` to `>=`, delete a line) and run the tests. If nothing fails, the test isn't testing that behaviour.
- [ ] Tests check **behaviour** (what goes in, what comes out, what's in the database), not implementation details.
- [ ] Test data is small, readable, and lives in fixtures, not in a 300-column string inside the test.
- [ ] Tests that touch a database clean up, and **can't run against a non-test database by accident**.

## Data and transactions
- [ ] Where does the transaction start and end? If the run fails halfway, what's left in the database?
- [ ] Is rerunning the same input safe (idempotent)?
- [ ] After a failure, does the program report counts it then rolled back? That would be lying to the user.

## Scope and dependencies
- [ ] Any new dependency the brief didn't approve? Check `pyproject.toml`.
- [ ] Anything from the "Out of scope" list sneaking in, such as retries, parallelism, or a CLI framework?
- [ ] Does the code follow the style rules (nesting depth, function length, type hints)?

## Claims and environment
- [ ] Did the AI *verify* the input format, or recall it from memory?
- [ ] Did it claim something about a library or database feature? Check the docs for the version you're actually using.
- [ ] Do the commands work on **your** OS and with **your** tools, not just the ones the AI assumed?

## Public repo hygiene
- [ ] No credentials, no data files, no absolute paths, and no personal emails in the diff or the PR description.
