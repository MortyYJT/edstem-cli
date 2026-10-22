<p align="center"><a href="https://unicorn.tuuhub.com"><img src=".github/assets/hero.jpg" alt="edstem-cli: Ed Discussion from your terminal, scripts and AI agent, part of unicorn" width="100%"></a></p>

# edstem-cli

**Give your AI agent access to Ed Discussion.**

Let it catch up on the forum, search threads, read lessons, download slides and attachments, and post when you ask. One package, three ways in: a CLI, a local MCP server, and a hosted **remote MCP server** for Claude, ChatGPT and other web AI.

[![npm version](https://img.shields.io/npm/v/edstem-cli?logo=npm)](https://www.npmjs.com/package/edstem-cli)
[![CI](https://github.com/bunizao/edstem-cli/actions/workflows/ci.yml/badge.svg)](https://github.com/bunizao/edstem-cli/actions/workflows/ci.yml)
[![Node.js 20+](https://img.shields.io/badge/Node.js-20%2B-339933?logo=nodedotjs&logoColor=white)](https://nodejs.org/)
[![MIT License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

> 🦄 **Part of [unicorn](https://unicorn.tuuhub.com).** Live tools answer what is there now; unicorn answers what changed.
>
> 📖 **[Read the docs → unicorn.tuuhub.com/docs/edstem](https://unicorn.tuuhub.com/docs/edstem)**

| Use it from | How |
| --- | --- |
| Claude, ChatGPT or any web AI | Add `https://edstem.tuuhub.com/mcp` as a custom connector ([remote MCP](#remote-mcp)) |
| Claude Code, Codex or another local agent | `edstem-mcp` ([local MCP](#local-mcp)) or the CLI plus its [agent skill](#agent-skill) |
| Your terminal and scripts | `npm install -g edstem-cli` ([quick start](#quick-start)) |

## Quick start

```bash
npm install -g edstem-cli
edstem auth login
edstem units
edstem threads UNIT --unread --limit 20
```

`edstem auth login` opens the Ed token page for you and works out your Ed region.

## Quick links

- [Install and sign in](#install-and-sign-in)
- [Study Boooooooooost](#study-boooooooooost)
- [Remote and local MCP](#mcp-servers)
- [Developer and agent reference](#for-developers-and-agents)

## Install and sign in

### Install

```bash
npm install -g edstem-cli
edstem --version
```

This installs two binaries: `edstem` (the CLI) and `edstem-mcp` (a local stdio MCP server).

### Sign in

```bash
edstem auth login
edstem auth status
```

Login opens Ed's API token page in your browser, reads the token without echoing it, and finds which Ed site accepts it (`edstem.org`, `us.edstem.org` or `eu.edstem.org`). It saves both to `~/.config/edstem-cli/token` (mode `0600`), and the CLI and `edstem-mcp` use them from then on. If verification fails, login lets you retry, enter another token, pick another region, or cancel without touching saved credentials.

For scripts and CI, skip the file:

```bash
export EDSTEM_TOKEN="your-token"
export EDSTEM_REGION="us"   # au (edstem.org, default), us, or eu

# Or save it non-interactively
printf '%s\n' "your-token" | edstem auth login --region us --token-stdin
```

`EDSTEM_TOKEN` wins over the saved token, and login warns when an environment variable overrides what it saves. `edstem auth logout --yes` removes the saved credentials.

### Update

```bash
edstem update --check
edstem update --yes
```

## Study Boooooooooost

Ask your agent in plain language or run the matching command:

| Student request | CLI command |
| --- | --- |
| “What did I miss on the forum?” | `edstem threads UNIT --unread` |
| “Any unanswered questions this week?” | `edstem threads UNIT --unanswered --since 7d` |
| “Find threads about the assignment deadline.” | `edstem threads search UNIT assignment deadline` |
| “Read thread 42 for me.” | `edstem threads read UNIT#42` |
| “What lessons are left in week 2?” | `edstem lessons UNIT --module "Week 2"` |
| “Download the slides from this lesson.” | `edstem files get LESSON --dest ./slides` |

`UNIT` is the numeric Ed course ID or the course code exactly as Ed shows it; `edstem units` lists both. If two enrolments share a code, use the ID from `edstem units --archived`. Threads are addressed as `UNIT#42` (unit plus thread number).

Commands follow `edstem <noun> [verb] [args] [flags]`. `courses` and `projects` are aliases for `units`, and the verb can be left out when the arguments are unambiguous. `read` always prints Markdown and never changes anything in Ed.

```bash
edstem threads show UNIT#42
edstem lessons show LESSON
edstem lessons read LESSON
edstem slides read SLIDE
edstem slides show SLIDE --section questions
edstem activity UNIT
```

### Filter threads and lessons

```bash
edstem threads UNIT --category "Assignments" --subcategory "A1"
edstem threads UNIT --unanswered --since 7d --limit 20
edstem threads search UNIT assignment deadline --since 2026-09-01
edstem lessons UNIT --module "Week" --status all
```

- `--since` takes an ISO date or datetime, or a relative offset (`12h`, `7d`, `2w`).
- `threads search` matches every word, case-insensitively, in titles and bodies, and takes the same filters as listing.
- Ed applies none of these filters itself, so `--limit` pages through Ed until enough threads match (up to ten requests).
- Lesson filters are case-insensitive. `--module` takes an ID or part of a name; an invalid value lists the ones the unit has.

### Catch up since your last visit

```bash
edstem threads UNIT --unread --limit 20     # what Ed reports as unseen
edstem threads UNIT --since last --limit 100  # new since your last catch-up
```

`--since last` keeps a local cursor per unit in `~/.config/edstem-cli/state.json`. The first run looks back seven days. The cursor only moves after a complete, unfiltered listing, so a truncated or filtered run never makes you miss threads. Listing never marks anything read in Ed.

### Download lesson and thread files

```bash
edstem files list LESSON
edstem files get LESSON --dest ./slides
edstem files get LESSON --slide SLIDE --dest ./slides
edstem files get thread:UNIT#42 --dest ./attachments
```

Lesson targets collect PDF slides and Ed-hosted files embedded in lesson content. `thread:` targets collect attachments from the post, answers and nested comments. Existing files are kept unless you pass `--force`.

### Export a forum

```bash
edstem threads export UNIT --dest ./archive
edstem threads export UNIT --dest ./archive --since 2026-07-01 --no-files
```

The archive is plain Markdown: `index.md`, one file per thread under `threads/`, attachments under `files/` with links rewritten to local paths, and a `manifest.json`. A rerun skips threads already exported, so an interrupted export resumes; `--force` rewrites them. Use one destination per unit.

### Write to Ed

Every write prints a plan and asks `y/N`. Scripts pass `--yes`; `--dry-run` shows the plan without sending anything.

```bash
# Thread state: star/unstar, watch/unwatch, upvote/unvote, mark-read/mark-unread
edstem threads star UNIT#42 --yes
edstem threads upvote UNIT#42 --comment 88991 --yes
edstem threads mark-read UNIT --all --yes

# Lessons and slides
edstem lessons mark-read UNIT Pre-Reading --yes
edstem slides submit SLIDE --question 991 --choice 2 --yes
```

State actions skip the write when the thread is already in the requested state. Marking a whole unit always needs `--all`.

### Post threads and replies

Posting publishes under your own name and cannot be undone from this tool. Request formats have been checked against mocked Ed responses only, so preview with `--dry-run` first; it prints the Ed XML that will be sent.

```bash
edstem threads send UNIT --title "Week 3 question" --body "Does **Q4** need induction?" --dry-run
edstem threads send UNIT --title "Week 3 question" --body-file ./post.md --type question --yes
edstem replies send UNIT#42 --body "Fixed by reinstalling." --yes
edstem replies send UNIT#42 --to 88991 --as comment --body-file - --yes
```

The body is Markdown (headings, code, bold, italic, links, lists). With no body flag, a terminal opens `$EDITOR`. `--private` posts to staff only; `--anonymous` hides your name from other students.

## MCP servers

### Remote MCP

`https://edstem.tuuhub.com/mcp` is a hosted Streamable HTTP server with OAuth. Add it as a custom connector in Claude, ChatGPT or any client that speaks remote MCP, then sign in with your Ed token. Client-by-client steps: [unicorn.tuuhub.com/docs/mcp](https://unicorn.tuuhub.com/docs/mcp).

Prefer your own server? [Deploy your own Worker](#deploy-your-own-worker).

### Local MCP

`edstem-mcp` uses the same saved token and region as the CLI:

```json
{
  "mcpServers": {
    "edstem": {
      "command": "edstem-mcp"
    }
  }
}
```

Unit arguments follow the CLI rule, so tools accept a course code directly. The posting tools `create_thread` and `reply_thread` are off by default; set `EDSTEM_ALLOW_POSTING=1` to enable them. Thread state actions are CLI-only.

### Interactive views

In hosts that support [MCP Apps](https://github.com/modelcontextprotocol/ext-apps) (Claude and ChatGPT among them), four tools render a view in the chat instead of a wall of text:

| Tool | Shows |
| --- | --- |
| `show_forum_catchup` | Unread announcements and new or updated threads, expandable in place. |
| `show_thread_activity` | New threads per week by category, with each week's busiest threads. |
| `show_lesson_progress` | Completed lessons per released module, and what is left. |
| `show_lesson_guide` | A step-through guide for one lesson plus a short practice quiz. The lesson's own Ed quiz is never answered. |

Ask for plain text and the assistant uses the `list_*` and `read_*` tools instead. `EDSTEM_WIDGETS=0` removes the `show_*` tools from `edstem-mcp`.

### Deploy your own Worker

[![Deploy to Cloudflare](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/bunizao/edstem-cli)

The button deploys `src/worker.ts` to your Cloudflare account, serving `https://<worker-host>/mcp` and `/healthz`. It has no database or storage: clients send an Ed token with each request and the Worker validates it without keeping it. The [MCP setup guide](MCP_SETUP.md) covers manual deployment, Worker variables and connection steps for ChatGPT, Codex and Claude.

## For developers and agents

### Command and output contract

```bash
edstem commands --json
```

prints the full command tree, including aliases, positionals, enum values and which commands write.

- Output is a table in a terminal and JSON in a pipe; force one with `--json`, `--yaml` or `--table`.
- `--fields a,b` selects fields; `-o, --output FILE` writes to a file.
- In a terminal, a command missing its unit asks with a picker; pipes, `--json` and agent shells get a usage error instead.
- Errors are printed once on stderr. Reads retry on rate limits; writes never retry.

| Exit code | Meaning |
| --- | --- |
| 0 | Success |
| 2 | Usage error |
| 3 | Authentication error |
| 4 | Unit, thread, lesson or slide not found |
| 5 | Ed rejected the request |
| 130 | Cancelled |

### Agent skill

```bash
npx skills add https://github.com/bunizao/edstem-cli
```

[`SKILL.md`](SKILL.md) is generated from `edstem commands --json` and the MCP tool catalog with `edstem skills generate`; CI rejects drift.

### Configuration

| Variable | Purpose |
| --- | --- |
| `EDSTEM_TOKEN` | Ed API token; overrides the saved one. |
| `EDSTEM_REGION` | `au` (edstem.org), `us` or `eu`; overrides the saved region. Defaults to `au`. |
| `EDSTEM_BASE_URL` | Override the Ed API endpoint. |
| `EDSTEM_CONFIG` | Use another config file instead of `~/.config/edstem-cli/config.yaml`. |
| `EDSTEM_ALLOW_POSTING` | `1` enables posting tools in `edstem-mcp`. |
| `EDSTEM_WIDGETS` | `0` removes the `show_*` tools from `edstem-mcp`. |

`config.yaml` can tune read retries with `rateLimit.maxRetries` (default `3`) and `rateLimit.retryBaseDelay` (seconds, default `1.0`).

### Build from source

```bash
npm ci
npm run check
npm test
npm run build
npm run pack:check
```

## 🦄 Part of unicorn

unicorn is one project in two layers. The live tools answer what is there now; unicorn answers what changed.

| Project | Layer | What it does |
| --- | --- | --- |
| [🦄 unicorn](https://github.com/TuuHub/unicorn) | Memory | A Cloudflare Worker on your own account. Reads Moodle, Ed, Canvas, Gmail and feeds every hour, remembers what each said, and tells your AI agent what changed. Remote MCP. |
| [moodle-cli](https://github.com/bunizao/moodle-cli) | Live | Moodle: units, deadlines, grades, forums, files, submissions. CLI, local and remote MCP. |
| **[edstem-cli](https://github.com/bunizao/edstem-cli)** (you are here) | **Live** | **Ed Discussion: units, threads, lessons, files, posting. CLI, local and remote MCP.** |
| [ontrack](https://github.com/bunizao/ontrack-cli) | Live | OnTrack / Doubtfire: units, tasks, chats, submissions. CLI only. |

The three CLIs share one command contract through [@bunizao/cli-kit](https://github.com/bunizao/cli-kit).

📖 **Docs:** [this project](https://unicorn.tuuhub.com/docs/edstem) · [all projects](https://unicorn.tuuhub.com/docs) · [CLIs overview](https://unicorn.tuuhub.com/cli) · [MCP clients](https://unicorn.tuuhub.com/docs/mcp)

## License

[MIT](LICENSE). See [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md) for migrated remote MCP attribution.
