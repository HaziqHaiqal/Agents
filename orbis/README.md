# Orbis

Orbis looks after a project while you're away. It runs on GitHub's computers, so your own computer can be off. It works with most languages, hosts and AI services.

- **Weekly check-up.** Every Monday it looks for one real problem (security, a bug, outdated docs, unused packages) and opens one small PR, or nothing if all is well.
- **Urgent fixes.** When a production deploy fails, tests fail on the main branch, an API crashes or the site goes down, it opens an issue, writes a test that shows the bug, fixes it and opens a PR.
- **Health check.** Every 15 minutes it checks your site. If a new release broke it, it can roll back to the previous version.
- **Reports on Telegram.** Outages, fixes ready for you, recoveries and a weekly report, with buttons that open GitHub.
- **Talk to it.** Comment `@orbis` on any PR or issue, for example `@orbis make this message shorter`.

It never merges, never pushes to your main branch and never touches production data, secrets or settings. You approve every change by merging its PR, or reject it by closing the PR. When you close one, leave a one-line reason so it learns.

## Install

Secrets below go in your repo under **Settings → Secrets and variables → Actions → New repository secret**.

### 1. Add the two files

Copy [`install/orbis.yml`](install/orbis.yml) and [`install/orbis-health.yml`](install/orbis-health.yml) into your repo's `.github/workflows/` folder. In both, change `site` to your app's address. In `orbis.yml`, change `[CI]` to the names of the workflows that test or deploy your project. Orbis steps in when one of them fails on the main branch.

### 2. Choose an AI (optional)

Set `ai` and `model` in `orbis.yml` and add the key as a secret.

| Service                     | `ai`                                 | `model`                                | Secret                                                                                      |
| --------------------------- | ------------------------------------ | -------------------------------------- | ------------------------------------------------------------------------------------------- |
| Anthropic plan (Pro or Max) | `anthropic`                          | empty for the default, or a model name | `AI_PLAN_TOKEN`: run `claude setup-token` once and paste the token                          |
| Anthropic API               | `anthropic`                          | empty for the default, or a model name | `AI_API_KEY` from console.anthropic.com                                                     |
| Google Gemini               | `gemini`                             | for example `gemini-3.6-flash`         | `AI_API_KEY` from aistudio.google.com                                                       |
| OpenAI                      | `openai`                             | a model name from platform.openai.com  | `AI_API_KEY`, an API key (a ChatGPT plan doesn't come with one)                             |
| DeepSeek                    | `https://api.deepseek.com/anthropic` | for example `deepseek-v4-pro`          | `AI_API_KEY`                                                                                |
| Ollama on your own server   | `http://your-server:11434`           | a model you've pulled                  | `AI_API_KEY`: any text, Ollama ignores it                                                   |
| None                        |                                      |                                        | Nothing. Orbis still checks, rolls back, opens issues and reports, but doesn't write fixes. |

Other services work too:

- **Services that accept Anthropic-format requests**, such as Kimi, GLM and Ollama's cloud. If their docs give an `ANTHROPIC_BASE_URL`, put that address in `ai`.
- **Everything else** goes through LiteLLM, a translator Orbis starts inside each run. Use its provider name as `ai`, as with `gemini` and `openai` above. Others include `mistral`, `groq` and `xai`; see the [list of providers](https://docs.litellm.ai/docs/providers).

Orbis is tested with Anthropic and Gemini. Other services usually work, but smaller models often struggle with long, multi-step work like finding a bug, testing it and opening a PR. When a run fails, Orbis reports that it couldn't finish, and nothing else is affected.

### 3. Let it open PRs

Orbis opens PRs through a GitHub app. PRs opened by an app start your tests; PRs opened by GitHub's built-in token don't.

**Your own app (recommended).** PRs and comments show as made by your app, for example **orbis[bot]**. You create it once and can install it on all your repos.

1. On GitHub, go to your profile picture → **Settings → Developer settings → GitHub Apps → New GitHub App**.
2. **GitHub App name:** `Orbis`. Names are unique across GitHub, so if it's taken, add something, like `Orbis Lee`. **Homepage URL:** your repo's address. Under **Webhook**, untick **Active**.
3. Under **Repository permissions**, set **Contents**, **Issues** and **Pull requests** to **Read and write**, and **Actions** to **Read-only**.
4. Press **Create GitHub App**. On the next page, copy the **Client ID**, then under **Private keys** press **Generate a private key**. A `.pem` file downloads.
5. In the left menu, open **Install App**, press **Install**, choose **Only select repositories** and pick your repo.
6. Add two secrets: `ORBIS_APP_CLIENT_ID` (the Client ID) and `ORBIS_APP_PRIVATE_KEY` (open the `.pem` file in a text editor and paste all of it).

**Ready-made app (quicker).** Install https://github.com/apps/claude on your repo and skip the secrets. PRs then show as made by that app.

### 4. Telegram reports (optional)

1. In Telegram, message **@BotFather**, send `/newbot` and follow the steps. Keep the token it gives you.
2. Open your new bot, press **Start** and send it any message.
3. Open `https://api.telegram.org/bot<TOKEN>/getUpdates` in a browser. The number after `"chat":{"id":` is your chat ID.
4. Add the secrets `TELEGRAM_BOT_TOKEN` and `TELEGRAM_CHAT_ID`.

One bot can serve all your repos. Each message starts with the repo's name.

### 5. Automatic rollback (optional)

**On Vercel**, add the secrets `VERCEL_TOKEN` (Vercel → Account Settings → Tokens), `VERCEL_ORG_ID` (Vercel → Team Settings → Team ID) and `VERCEL_PROJECT_ID` (Project → Settings → Project ID). Orbis rolls back only when the live version is less than 2 hours old. After a rollback, Vercel pauses new releases until you press **Undo Rollback**.

**On any other host**, set `rollback-command` in `orbis-health.yml` to the command that puts the previous version live, and add the token it needs as the secret `DEPLOY_TOKEN`. The command can use it as `$DEPLOY_TOKEN`. For example, on Heroku: `HEROKU_API_KEY=$DEPLOY_TOKEN npx heroku rollback --app my-app`. Check your host's docs for its command. It runs once at the start of every outage, whatever the cause, so use it only if going back one version is always safe for your app.

Without either, Orbis still opens an issue and reports the outage, and you roll back by hand.

### 6. Recommended

- **Protect your main branch.** In **Settings → Rules**, require a pull request and passing tests before merging. Orbis can then never change the main branch without you.
- **Write an `AGENTS.md`** in your repo with the project's rules and the commands for its checks. Many AI coding tools read this file. Orbis reads it (or a `CLAUDE.md`) before every job.

### 7. Try it

- **Actions → Orbis health → Run workflow**, tick **rehearse**. Nothing changes; you get a Telegram message showing what Orbis would do in an outage.
- **Actions → Orbis → Run workflow** with the title empty runs a check-up now.

## Settings

Put these under `with:` in your files.

**`orbis.yml`**

| Setting                  | Default                                     | What it does                                                           |
| ------------------------ | ------------------------------------------- | ---------------------------------------------------------------------- |
| `site`                   | none                                        | Your app's live address                                                |
| `ai`                     | `anthropic`                                 | Which AI service to use (see step 2)                                   |
| `model`                  | the service's default                       | Model name at that service. Required for services other than Anthropic |
| `setup`                  | install from the usual files                | Commands that prepare the project so its checks can run                |
| `node-version`           | latest LTS                                  | Node.js version for JavaScript projects                                |
| `production-environment` | `Production`                                | Deploy environment that counts as production                           |
| `reviewer`               | repo owner                                  | Who reviews Orbis's PRs                                                |
| `tools`                  | build and test commands of common languages | Commands Orbis may run. Setting it replaces the whole list             |
| `max-open-prs`           | `2`                                         | Check-ups pause while this many Orbis PRs wait for you                 |
| `max-incidents-per-day`  | `5`                                         | After this, new incidents wait for you to say `@orbis`                 |

**`orbis-health.yml`**

| Setting                    | Default  | What it does                                         |
| -------------------------- | -------- | ---------------------------------------------------- |
| `site`                     | required | Your app's live address                              |
| `paths`                    | `/`      | Pages to check, separated by spaces                  |
| `rollback-command`         | none     | Command that rolls back on hosts other than Vercel   |
| `rollback-max-age-minutes` | `120`    | On Vercel, only roll back releases younger than this |

## Other languages and hosts

**Languages.** Orbis installs JavaScript (npm, pnpm, yarn, bun), Python (pip, uv, poetry), Go, Rust, Ruby and PHP projects by itself from their usual files. GitHub's computers also come with Java, Maven, Gradle and .NET. For anything else, put the install commands in `setup`. If your checks need a command that isn't in `tools`, add it, for example `Bash(./scripts/test.sh:*)`.

**Failed deploys.** Orbis hears about them when your host reports deploys to GitHub. Vercel, Netlify and many others do, and they appear under **Deployments** on your repo's page. Set `production-environment` to the name shown there. If you deploy from a GitHub workflow instead, add that workflow's name to the `[CI]` list in `orbis.yml`.

**Not a website?** For a library, a command-line tool or a phone app, leave out `orbis-health.yml`. Everything else works.

## Sending alarms from your app

Your app can report its own errors, in any language that can send a web request. It starts the `orbis.yml` workflow with a `title` and `details`:

```sh
curl -X POST https://api.github.com/repos/OWNER/REPO/actions/workflows/orbis.yml/dispatches \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"ref": "main", "inputs": {"title": "API error: TypeError in /checkout", "details": "..."}}'
```

`$TOKEN` is a fine-grained GitHub token with **Actions: Read and write** on this repo only. One title means one issue. Repeats are ignored until that issue is closed. Repos are often public, so send the route and error type, never people's data.

## Versions

Projects use `@orbis-v1`. Changes that could break existing projects get a new label (`orbis-v2`), and each project moves when its owner is ready.
