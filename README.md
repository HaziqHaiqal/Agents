# Agents

AI agents that look after projects. Each one is a shared GitHub workflow: a project adds a couple of small files, and the agent runs on GitHub's computers, so it works even when nobody's computer is on. They work with most languages and hosts, and with the AI service you choose.

| Agent                    | What it does                                                                                                                                                                                                  |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Orbis](orbis/README.md) | Keeps a project healthy. Weekly check-ups, urgent fixes when a deploy, API or test fails, automatic rollback when the site goes down, and reports on Telegram. Every change comes as a PR for you to approve. |
