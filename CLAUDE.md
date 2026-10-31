# CLAUDE.md

Working agreements for AI assistants in this repo. The learner is practising AI-assisted engineering, so how you work matters as much as what you produce. Course material lives in `docs/` and `weeks/`. Read the current week's `BRIEF.md` before planning anything.

## How we work
- **Plan before code.** Restate the rules in your own words, including the cases where nothing happens. Check real inputs (files, headers, installed tools) instead of assuming them. List assumptions and open questions ("none" is suspicious). Then propose numbered steps, each with the files it touches and the tests that prove it.
- **One step at a time.** After each step, show the test output, explain what you built and why in 3–5 plain-English sentences, then stop and wait for approval.
- **One branch and one PR per step.** CodeRabbit reviews every PR.
- **Explain in plain English.** Say why you chose an approach, and point out library behaviour that might surprise the learner.
- **Ask before** adding any dependency, changing the agreed layout, or running a command that differs by OS.
- **Verify, don't recall.** Check API, library and config details against docs or the real environment. Say so when something is unverified.

## Code style
- Type hints everywhere. Plain functions. Classes only where the week's brief allows them.
- At most 2 levels of nesting. Functions under ~40 lines.
- Comments explain why, not what.
- Standard library first. Don't use frameworks the brief hasn't introduced yet.

## Never
- Commit `data/raw/`, `.env`, credentials, or absolute local paths (paths can reveal usernames).
- Edit `docs/` or `weeks/` in a learner repo. They're course material. Notes go in `LEARNING_LOG.md`.
- Add anything listed in the week's "Out of scope" section.

## Project commands
<!-- Fill this in as the project grows: setup, run, test. -->
