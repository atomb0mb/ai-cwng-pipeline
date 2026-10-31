# CodeRabbit proof of concept

[CodeRabbit](https://www.coderabbit.ai/) is an AI code reviewer that comments on pull requests. In this course it's the second reviewer on every PR, after you. The question this POC answers is: **what does an AI reviewer catch that you don't, what does it miss, and how noisy is it?**

*Facts below were checked against CodeRabbit's docs in October 2026. Plans and limits change, so recheck [docs.coderabbit.ai](https://docs.coderabbit.ai/) before relying on them.*

## Setup (personal GitHub account)
1. Go to [app.coderabbit.ai/login](https://app.coderabbit.ai/login) and choose **Login with GitHub**.
2. Select your personal account, then **Only select repositories**, and pick your course repo.
3. Click **Install & Authorize**.

To change the repos later: GitHub **Settings → Integrations → Applications → Installed GitHub Apps → CodeRabbit → Configure**.

**Plans:** CodeRabbit's open-source plan covers public repositories with no paid subscription, at a lower rate limit. According to their plans page, **public repos with fewer than 10 stars must trigger reviews manually**. If no review appears on your PR, comment `@coderabbitai review`.

## How it's configured here
- [`.coderabbit.yaml`](../.coderabbit.yaml) at the repo root holds the review profile, the paths to skip, extra instructions per path, and the Ruff linter. CodeRabbit uses the version of the file **on the PR's branch**.
- [`CLAUDE.md`](../CLAUDE.md) is read automatically as coding guidelines, so the style rules are written once and shared by Claude Code and CodeRabbit.
- To see the configuration CodeRabbit actually resolved, comment `@coderabbitai configuration` on a PR.

## Commands you'll use
Write each command as plain text at the start of a PR comment, one command per comment.

| Command | What it does |
|---|---|
| `@coderabbitai review` | Reviews new commits (incremental). Use this on low-star repos to start a review. |
| `@coderabbitai full review` | Reviews the whole PR from scratch. |
| `@coderabbitai summary` | Regenerates the PR summary. |
| `@coderabbitai resolve` | Resolves CodeRabbit's threads. As a reply, it resolves just that thread. |
| `@coderabbitai pause` / `resume` | Stops or restarts automatic reviews on this PR. |
| `@coderabbitai configuration` | Shows the resolved configuration. |
| `@coderabbitai help` | Lists all commands. |

You can also reply to any CodeRabbit comment in plain English, for example "why is this a problem?" or "this is intentional because…". Pushing back on a reviewer is a skill too.

## Evaluating it
For each PR, add rows to the table in your `LEARNING_LOG.md` entry:

| Finding | Found by (you / CodeRabbit / other AI) | Real issue? (yes / no / debatable) | Severity (bug / risk / style) | Acted on? |
|---|---|---|---|---|

At the end of week 1, answer these questions:
1. **Catch rate:** how many real issues did CodeRabbit find that you didn't?
2. **Misses:** which real issues did it miss? Were they rule-level (business logic) or code-level?
3. **Noise:** what share of its comments were false positives or nitpicks?
4. **Guidelines:** did it enforce the `CLAUDE.md` rules (nesting depth, function length, no unapproved dependencies)?
5. **Cost of attention:** how long did triaging its comments take, compared with the value they gave?
6. **Verdict:** would you keep it on a real team, and with which `profile` setting?

## Tips
- Keep PRs small. One step per PR gives the reviewer, and you, a fair chance.
- Don't merge just because CodeRabbit approved. It doesn't know your rule tables unless the code, the tests or the instructions spell them out.
- A finding you rejected, and why, is a great `LEARNING_LOG.md` entry.
