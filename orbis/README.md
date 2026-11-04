# Orbis

Orbis looks after a project while you're away. It runs on GitHub's computers, so your own computer can be off.

- **Weekly check-up.** Every Monday it looks for one real problem (security, a bug, outdated docs, unused packages) and opens one small PR, or nothing if all is well.
- **Urgent fixes.** When a production deploy fails, tests fail on the main branch, an API crashes or the site goes down, it opens an issue, writes a test that shows the bug, fixes it and opens a PR.
- **Health check.** Every 15 minutes it checks your site. If a new release broke it, it rolls back to the previous version on Vercel.
- **Reports on Telegram.** Outages, fixes ready for you, recoveries and a weekly report, with buttons that open GitHub.
- **Talk to it.** Comment `@orbis` on any PR or issue, for example `@orbis make this message shorter`.

It never merges, never pushes to your main branch and never touches production data, secrets or settings. You approve every change by merging its PR, or reject it by closing the PR. When you close one, leave a one-line reason so it learns.

## Install

### 1. Add the two files

Copy [`install/orbis.yml`](install/orbis.yml) and [`install/orbis-health.yml`](install/orbis-health.yml) into your repo's `.github/workflows/` folder. In both, change `site` to your app's address. In `orbis.yml`, change `[CI]` to the name of your test workflow.

### 2. Give it a brain (optional)

Add **one** of these as a repository secret (**Settings → Secrets and variables → Actions → New repository secret**):

| You have                           | Secret                                                                                      |
| ---------------------------------- | ------------------------------------------------------------------------------------------- |
| Claude Pro or Max                  | `CLAUDE_CODE_OAUTH_TOKEN`: run `claude setup-token` once and paste the token                |
| An Anthropic API key (pay per use) | `ANTHROPIC_API_KEY`: from console.anthropic.com                                             |
| Neither                            | Nothing. Orbis still checks, rolls back, opens issues and reports, but doesn't write fixes. |

With a brain, also install the Claude GitHub app on the repo: https://github.com/apps/claude

### 3. Telegram reports (optional)

1. In Telegram, message **@BotFather**, send `/newbot` and follow the steps. Keep the token it gives you.
2. Open your new bot, press **Start** and send it any message.
3. Open `https://api.telegram.org/bot<TOKEN>/getUpdates` in a browser. The number after `"chat":{"id":` is your chat ID.
4. Add the secrets `TELEGRAM_BOT_TOKEN` and `TELEGRAM_CHAT_ID`.

One bot can serve all your repos. Each message starts with the repo's name.

### 4. Automatic rollback (optional, Vercel only)

Add the secrets `VERCEL_TOKEN` (Vercel → Account Settings → Tokens), `VERCEL_ORG_ID` (Vercel → Team Settings → Team ID) and `VERCEL_PROJECT_ID` (Project → Settings → Project ID).

Orbis rolls back only when the live version is less than 2 hours old. After a rollback, Vercel pauses new releases until you press **Undo Rollback**.

### 5. Recommended

- **Protect your main branch.** In **Settings → Rules**, require a pull request and passing tests before merging. Orbis can then never change the main branch without you.
- **Write a `CLAUDE.md`** in your repo with the project's rules and the commands for its checks. Orbis reads it before every job.

### 6. Try it

- **Actions → Orbis health → Run workflow**, tick **rehearse**. Nothing changes; you get a Telegram message showing what Orbis would do in an outage.
- **Actions → Orbis → Run workflow** with the title empty runs a check-up now.

## Settings

Put these under `with:` in your files.

**`orbis.yml`**

| Setting                  | Default                     | What it does                                            |
| ------------------------ | --------------------------- | ------------------------------------------------------- |
| `site`                   | none                        | Your app's live address                                 |
| `setup`                  | install from the lockfile   | Commands that prepare the project so its checks can run |
| `node-version`           | `24`                        | Node.js version                                         |
| `production-environment` | `Production`                | Deploy environment that counts as production            |
| `reviewer`               | repo owner                  | Who reviews Orbis's PRs                                 |
| `tools`                  | pnpm, npm and yarn commands | Commands Orbis may run                                  |
| `model`                  | `claude-sonnet-5-5`         | Claude model                                            |
| `max-open-prs`           | `2`                         | Check-ups pause while this many Orbis PRs wait for you  |
| `max-incidents-per-day`  | `5`                         | After this, new incidents wait for you to say `@orbis`  |

**`orbis-health.yml`**

| Setting                    | Default  | What it does                              |
| -------------------------- | -------- | ----------------------------------------- |
| `site`                     | required | Your app's live address                   |
| `paths`                    | `/`      | Pages to check, separated by spaces       |
| `rollback-max-age-minutes` | `120`    | Only roll back releases younger than this |

## Sending alarms from your app

Your app can report its own errors. It starts the `orbis.yml` workflow with a `title` and `details`:

```
POST https://api.github.com/repos/OWNER/REPO/actions/workflows/orbis.yml/dispatches
Authorization: Bearer <fine-grained token with Actions: read and write on this repo>
{"ref": "main", "inputs": {"title": "API error: TypeError in /checkout", "details": "..."}}
```

One title means one issue. Repeats are ignored until that issue is closed. Repos are often public, so send the route and error type, never people's data.

## Versions

Projects use `@orbis-v1`. Changes that could break existing projects get a new label (`orbis-v2`), and each project moves when its owner is ready.
