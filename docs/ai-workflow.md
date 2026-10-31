# The plan-first AI workflow

AI coding assistants are fast and confident, and those are exactly the two traits that let mistakes slip through. This workflow slows down the parts where mistakes are cheap to catch (the spec and the plan), so the fast part (writing code) builds the right thing.

```
Brief → Restate → Verify inputs → Assumptions & questions → Plan → [Build one step → Review → Log] × N
```

## 1. Brief
A precise spec: goal, decisions already made, behaviour as **rule tables**, a definition of done, what's out of scope, and the style rules. Each week provides one.

**Why rule tables?** Prose like "don't overwrite newer data" hides edge cases. A table forces you to fill in every row: what if the stored value is NULL? What if both dates are equal? Empty cells show up as questions you still have to answer.

## 2. Restate
Ask the AI to restate every rule in its own words, **including the rows where nothing happens**.

**Why:** if the AI misunderstands a rule, the restatement shows it before any code exists. The "nothing happens" rows matter most, because that's where silent bugs live: a record that should have been left alone gets overwritten.

## 3. Verify inputs
The AI checks reality instead of assuming it: the real file's headers and encoding, the installed tool versions, whether the database or container runtime exists.

**Why:** an AI will happily write code against the column names it *remembers*. The spec says "Python 3.12", but is it installed? The spec says `docker compose`, but is Docker there? These checks take seconds and save hours.

## 4. Assumptions and open questions
The AI lists every assumption it's making and every question the spec doesn't answer.

**Why:** "None" is almost always wrong. Look especially for:
- Inputs the rule tables don't cover (malformed data, empty values, odd encodings).
- Places where a library is more lenient than your rule.
- Rules that are internally consistent but produce a surprising result in a real-world scenario.

Answer the blocking questions. Write the rest down as known limitations.

## 5. Plan
Numbered steps. For each step: the files it touches and the tests that prove it.

**Why:** small steps keep each PR reviewable in one sitting. Naming the tests up front means "done" is defined before the code exists, so the AI can't quietly redefine it.

## 6. Build one step
Approve **only the next step**. The AI builds it, runs the tests, shows the output, and explains in 3–5 plain sentences what it built and why. Then it stops.

**Why stop?** Every step is a checkpoint where you can still change direction cheaply. An AI that runs all five steps at once hands you 1,000 lines to review, and you won't review them properly.

## 7. Review
Review the PR in three passes:
1. **You:** use the [reviewing AI output checklist](reviewing-ai-output.md).
2. **CodeRabbit:** automatic on every PR. See [coderabbit-poc.md](coderabbit-poc.md).
3. **A second AI (optional):** for example, Claude Code's `/code-review`, started in a fresh session so it doesn't carry over the builder's assumptions.

Merge only when you can explain every line to someone else.

## 8. Log
Add an entry to [`LEARNING_LOG.md`](../LEARNING_LOG.md): what the AI proposed, what you changed, what each reviewer caught, and what you learned. Save the prompt you used in [`prompts/`](../prompts/).

**Why:** the log is the real output of this course. Months from now it will show you, or a hiring manager, how you think when working with AI.

## Prompting tips
- Paste the brief whole. Don't summarise it, because summaries drop exactly the details that matter.
- When you disagree with the AI, say why. "No" teaches it nothing for the rest of the session. "No, because the file can contain a byte-order mark" does.
- Ask "what would make this fail?" after every step.
- If the AI says something confidently about a library or API, ask it to show you the docs or a quick experiment.
