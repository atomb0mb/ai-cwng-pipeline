# Course guide

This guide covers everything you need before and around the weekly exercises. The exercises themselves are in [`weeks/`](../weeks/).

## How the course works
Each week has a **brief**: a precise spec you hand to an AI coding assistant. You don't write the spec from scratch in week 1, because a good one is provided. What you do is drive the AI through it the way a senior engineer would:

1. The AI restates the rules, checks the real inputs, lists assumptions, and proposes a plan.
2. You challenge the plan until it's right, then approve **one step**.
3. The AI builds that step on its own branch, and you open a pull request.
4. You, CodeRabbit and (optionally) a second AI review the PR. You merge only once you understand every line.
5. You log what the AI got wrong and what you caught.

The details are in [ai-workflow.md](ai-workflow.md).

## Prerequisites
| Tool | Version | Notes |
|---|---|---|
| Python | The version the week's brief names (3.14 for week 1) | Check with `py -0p` on Windows, or `python3 --version` on macOS/Linux. |
| Git | Any recent version | |
| Container runtime | Docker Desktop, **or** Podman with a compose provider | With Podman, also install the standalone `docker-compose` binary, then use `podman machine start` and `podman compose up -d`. Check your employer's Docker Desktop licensing if you're on a work machine. |
| GitHub account | | A personal account is simplest for installing CodeRabbit. |
| AI coding assistant | | The course is written for Claude Code (it reads [`CLAUDE.md`](../CLAUDE.md)), but any capable assistant works. |
| CodeRabbit | | Free GitHub app for public repos (check current pricing). See [coderabbit-poc.md](coderabbit-poc.md). |

## Setup
1. Create your repo from this template (**Use this template**), then clone it.
2. Set a repo-local commit email so your real address stays private (see [below](#keeping-a-public-repo-clean)).
3. Download the week's input data into `data/raw/` (see [data/README.md](../data/README.md)).
4. Create a virtual environment. This is the one command that differs by OS:
   | OS | Create | Activate |
   |---|---|---|
   | Windows (PowerShell) | `py -3.14 -m venv .venv` | `.venv\Scripts\Activate.ps1` |
   | macOS / Linux | `python3.14 -m venv .venv` | `source .venv/bin/activate` |

   *Why a venv?* It plays the same role as a per-project `packages` folder in .NET or `node_modules` in JavaScript. Each project gets its own isolated set of dependencies.
5. Install CodeRabbit on your repo.

## Weekly rhythm
| When | What |
|---|---|
| Start | Read the brief. Personalise its Context section. Give it to your AI and get a plan. |
| Plan review | Compare the plan with the brief, line by line. Push back on vague assumptions. Save the prompt to `prompts/`. |
| Each step | Branch, build, test, open a PR, review it, merge. Add an entry to `LEARNING_LOG.md`. |
| End | Check your log against the week's `FACILITATOR_NOTES.md`. What did you miss? What did you catch that the notes don't mention? |

## Branch and PR flow
```bash
git switch -c week-01/step-1-parse        # one branch per step
# ... AI builds the step, tests pass ...
git add -A && git commit -m "Week 1 step 1: parse, validate, dedupe"
git push -u origin week-01/step-1-parse
gh pr create --fill                        # or open the PR on github.com
```
CodeRabbit comments on the PR within a few minutes. Reply to its comments as you would to a colleague's: fix the issue, or explain why it doesn't apply.

## Keeping a public repo clean
A public repo is visible to everyone, including future employers. There are several easy ways to leak information:

| Leak | Prevention |
|---|---|
| **Your email in commits.** Every commit records an author email. | Turn on *Keep my email addresses private* at github.com/settings/emails, then in your repo run `git config user.email "<id>+<username>@users.noreply.github.com"` (no `--global`, so it applies to this repo only). |
| **Your username in paths.** Error messages and some tool settings contain absolute paths such as `C:\Users\<you>\...` or `/home/<you>/...`. | Use relative paths in committed files. Check pasted terminal output before committing. `.claude/settings.local.json` is already git-ignored. |
| **Credentials.** | Only `.env.example` (with fake values) is committed. `.env` is git-ignored. |
| **Data.** NPPES files are public but huge. | `data/raw/` is git-ignored. Tests use the small fixtures. |

Before pushing, search for anything personal: `git grep -n -i "<your username>"`.

## Pulling template updates
Repos created from a template don't receive updates automatically. *Use this template* copies the files into a brand-new history that has no link to the template's. Connect the two once, then pull updates whenever you like.

**One-time setup:**
```bash
git remote add template https://github.com/atomb0mb/ai-pipeline-template.git
git fetch template
git log template/main --oneline --format="%h %cs %s"   # find the commit dated just before you created your repo
git merge -s ours --allow-unrelated-histories <that-commit> -m "Link history to template"
```
*Why `-s ours`?* It records "I already have everything up to this template commit" without changing any of your files. Without it, Git has no common starting point, treats every file as added on both sides, and reports a conflict for each file the template changed later.

**Each time the template has new content:**
```bash
git fetch template
git merge template/main
```
Because you never edit `docs/` or `weeks/`, these merges should be conflict-free. The exceptions are `README.md` and `CLAUDE.md`, which you own after week 1. For those, keep your version and copy in anything new from the template by hand.

## For facilitators
Each week has a `FACILITATOR_NOTES.md` that lists the gaps and traps a strong plan should surface. Learners should open it only **after** writing their own plan and log. In a group, compare logs: the most useful discussion comes from what one person's AI missed and another person's caught.
