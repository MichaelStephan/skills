# Bluebox skills

Agent skills that connect your coding agent to [Bluebox](https://bluebox.ai), the observability agent for AI-native development. With them, your agent checks production before it plans a change, picks up a Bluebox investigation's root cause, and adds OpenTelemetry to your services.

The skills work with Claude Code, Cursor, GitHub Copilot, Codex, OpenCode, Windsurf, Kiro, and other agents that read the [Agent Skills](https://agentskills.io) format.

## Skills

| Skill | What it does |
|---|---|
| [bluebox-overview](skills/bluebox-overview/SKILL.md) | Explains Bluebox to your agent and which Bluebox skill fits a request. |
| [bluebox-production-query](skills/bluebox-production-query/SKILL.md) | Gets production context (error rates, latency, callers, active problems) before your agent designs or ships a change. |
| [bluebox-investigation-context](skills/bluebox-investigation-context/SKILL.md) | Loads the root cause and evidence of the Bluebox investigation that filed a GitHub issue, so your agent builds the fix on it. |
| [bluebox-otel-instrumentation](skills/bluebox-otel-instrumentation/SKILL.md) | Adds OpenTelemetry to a service or repository and sends its telemetry to Bluebox. |
| [bluebox-otel-instrumentation-guided](skills/bluebox-otel-instrumentation-guided/SKILL.md) | The same instrumentation as a step-by-step flow. Install it together with `bluebox-otel-instrumentation`. |

## Before you start

The skills call the `bluebox` CLI, so you need a Bluebox account and the CLI. The [Bluebox docs](https://docs.bluebox.ai) show how to install it.

**If you use `bluebox setup`, you already have these skills.** It installs them into your agents and keeps them up to date. You don't need this repository.

## Install

In your project, run:

```bash
npx skills add bluebox-hq/skills
```

It asks which skills to install and for which agents. To choose without the questions, for example in a script or when your coding agent runs the install, name them:

```bash
npx skills add bluebox-hq/skills --skill '*' --agent claude-code cursor
```

Name one or more skills after `--skill` to install only those; `bluebox-otel-instrumentation-guided` needs `bluebox-otel-instrumentation` too. Add `-g` to install for your user instead of the current project. Prefer `--agent` over `--all`: `--all` installs for every agent `skills` supports and adds a skills folder for each to your project.

## Plugins

Claude Code, Cursor, Google Antigravity and Kiro can install Bluebox as one plugin instead. The plugin also connects your agent to the [Bluebox MCP server](https://docs.bluebox.ai/mcp-server). You sign in to Bluebox in your browser the first time the agent connects, so you don't need an access token.

Install the skills with one tool only. If you use the plugin, don't also install the skills with `npx skills` or `bluebox setup`, or your agent loads each skill twice.

### Claude Code

```bash
claude plugin marketplace add bluebox-hq/skills
claude plugin install bluebox@bluebox
```

To sign in, run `/mcp` in Claude Code, select `plugin:bluebox:bluebox` and log in to Bluebox. The plugin updates when this repository changes.

### Cursor

Clone this repository and link it into your local plugins folder:

```bash
git clone https://github.com/bluebox-hq/skills.git ~/bluebox-skills
ln -s ~/bluebox-skills ~/.cursor/plugins/local/bluebox
```

Reload Cursor, then sign in to the `bluebox` MCP server under **Settings → MCP**. To update, run `git pull` in the clone.

### Google Antigravity

```bash
agy plugin install https://github.com/bluebox-hq/skills
```

Antigravity opens the Bluebox sign-in when it first connects to the `bluebox` MCP server.

### Kiro

Kiro installs Bluebox as a power: the MCP server plus a guide for the agent ([POWER.md](POWER.md)). Open the **Powers** panel, select **Add Custom Power**, choose **GitHub repository** and enter `https://github.com/bluebox-hq/skills`. Kiro opens the Bluebox sign-in when it first connects. The power doesn't include the skills. To add them, run `npx skills add bluebox-hq/skills --skill '*' --agent kiro`.

## Update and remove

```bash
npx skills update
npx skills remove bluebox-production-query
```

Install each skill with one tool only. To switch to `bluebox setup`, first remove the copies you installed here (add `-g` for a user install), then run `bluebox setup`:

```bash
npx skills remove bluebox-investigation-context bluebox-otel-instrumentation bluebox-otel-instrumentation-guided bluebox-overview bluebox-production-query
```

Name the skills instead of using `--all`, which removes every skill in your agents' skills folders, not only these.

## License

[Apache-2.0](LICENSE)
