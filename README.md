# AI Pipeline Course

Learn AI-assisted data engineering by building a real pipeline, one reviewed step at a time.

Over eight weeks you build a Python pipeline that loads public US healthcare provider data (CMS NPPES) into Postgres, then grow it with validation, orchestration, LLM extraction and evals. You write the spec and review every change. An AI coding assistant writes most of the code. CodeRabbit reviews every pull request.

The skill you're really practising is the one AI-related roles depend on: **turning a fuzzy goal into a precise spec, directing an AI through it, and catching what it gets wrong.**

> **This is a template repository.** Don't work in it directly. Click **Use this template** to create your own copy, and do the exercises there.

## Who it's for
Working engineers, from any language background, who want hands-on practice with Python data pipelines and AI-assisted development. You should be comfortable with SQL, Git and a terminal. You don't need prior experience with Python data tooling.

## Quick start
1. Click **Use this template**, then **Create a new repository**. A name like `ai-pipeline-<yourname>` works well. Public is fine, as long as you never commit data, credentials or local paths ([why](docs/course-guide.md#keeping-a-public-repo-clean)).
2. Clone your new repo and install the [prerequisites](docs/course-guide.md#prerequisites).
3. Install CodeRabbit on your repo: [docs/coderabbit-poc.md](docs/coderabbit-poc.md).
4. Read [docs/ai-workflow.md](docs/ai-workflow.md). It describes the plan-first loop you'll use every week.
5. Open [weeks/week-01-nppes-loader/BRIEF.md](weeks/week-01-nppes-loader/BRIEF.md), personalise its Context section, and give it to your AI assistant.
6. Work one step per branch and pull request. Record what you learn in [LEARNING_LOG.md](LEARNING_LOG.md), and save the prompts you used in [prompts/](prompts/).

## Roadmap
| Week | Topic | Status |
|---|---|---|
| 1 | [NPPES CSV → Postgres loader](weeks/week-01-nppes-loader/BRIEF.md): stdlib `csv`, dataclass, idempotent upsert, one transaction, pytest | Available |
| 2 | Pydantic validation and a reject table | Planned |
| Later | Airflow orchestration | Planned |
| Later | LLM extraction | Planned |
| Later | Evals | Planned |

Each week builds on the same codebase, so the structure starts simple and is designed to be extended.

## Repository layout
```
README.md               This file. Replace it with your project README in week 1, step 5.
CLAUDE.md               Short working agreements for Claude Code (or any AI assistant).
LEARNING_LOG.md         Your log: what the AI got wrong, what you caught, what you learned.
.coderabbit.yaml        CodeRabbit review rules.
.github/                Pull request template.
docs/                   Course guide, AI workflow, reviewing AI output, CodeRabbit POC.  Read-only.
weeks/week-NN-*/        Brief, rules, fixtures and facilitator notes for each week.      Read-only.
prompts/                The prompts you actually used, saved per step.
data/raw/               Downloaded NPPES files (git-ignored, never committed).
```
Your code (`src/`, `tests/`, `sql/`, and so on) lives at the root of your own repo. Treat `docs/` and `weeks/` as read-only course material. Put your own notes in `LEARNING_LOG.md`, not in those folders.

## Docs
- [Course guide](docs/course-guide.md): prerequisites, setup, weekly rhythm, branch and PR flow, keeping a public repo clean, pulling template updates.
- [AI workflow](docs/ai-workflow.md): the plan-first loop, and why each step exists.
- [Reviewing AI output](docs/reviewing-ai-output.md): a checklist for code and tests an AI wrote.
- [CodeRabbit POC](docs/coderabbit-poc.md): setup, commands, and how to evaluate it.

## License
MIT, see [LICENSE](LICENSE). NPPES data is published by CMS and isn't redistributed here. Download it yourself, as described in [data/README.md](data/README.md).
