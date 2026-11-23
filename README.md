# Zacau for Antigravity

[Zacau](https://zacau.com) is a task and project workspace. This plugin connects Google Antigravity to Zacau's remote MCP server, so the agent can read and update your Zacau tasks, projects and captures.

You sign in with your Zacau account through OAuth in the browser. The plugin holds no API keys or secrets.

## What you can do

- See what's on your plate: today's tasks, your open tasks, overdue work, and a summary
- Create, update, assign and comment on tasks, and change their status
- Browse and manage projects and workspaces
- Capture quick notes and triage them into tasks
- Check workspace stats, velocity and team reports
- Sync tasks with Google Calendar

Try:

- "What do I have to do today in Zacau?"
- "Create a Zacau task to review the PR tomorrow"
- "Summarize my overdue Zacau tasks"

## Install

### Marketplace

Not listed yet. Once it's approved: Antigravity › **Customizations** › **Marketplace** › search "Zacau" › **+**.

### CLI

```sh
agy plugin install https://github.com/HR-Monster/zacau-antigravity
```

### Manual config (no plugin)

Add this to your Antigravity MCP config:

```json
{
  "mcpServers": {
    "zacau": {
      "serverUrl": "https://zacau-mcp-epg2bhgvcefhd0ad.southeastasia-01.azurewebsites.net/mcp"
    }
  }
}
```

Zacau › **Settings** › **Integrations** › **Antigravity** has a "Copy config" button that copies the same snippet, with the right URL for your environment.

## Sign in

**CLI (`agy`):** run `agy`, open `/plugin`, expand **zacau** under **Installed**, and choose **Sign in**. Then sign in to Zacau in the browser.

**Antigravity App:**

1. Antigravity › **Customizations** › **Installed** › **MCP Servers** › `zacau_tasks` › **Authenticate**
2. Sign in to Zacau in the browser window that opens.
3. Copy the code, paste it into **Enter authorization code** in Antigravity, then **Submit**.

## Disconnect

Use either:

- Antigravity: sign out of the `zacau_tasks` server, or uninstall the plugin with `agy plugin uninstall zacau`.
- Zacau › **Settings** › **Integrations** › **Antigravity** › **Disconnect**. This revokes Antigravity's access.

## Privacy and support

- Privacy policy: https://zacau.com/privacy
- Support: [admin@zacau.com](mailto:admin@zacau.com)
