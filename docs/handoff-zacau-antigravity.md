# Handoff: build the `zacau-antigravity` plugin

**Date:** 2026-11-23
**Target repo:** https://github.com/HR-Monster/zacau-antigravity (public, empty, no default branch yet)
**Parent issue:** HR-Monster/Zacau-Frontend#243. The research comment with the full context is at https://github.com/HR-Monster/Zacau-Frontend/issues/243#issuecomment-7094649154

## Goal

Package Zacau's remote MCP server as a **Google Antigravity plugin**. Users then get setup without hand-editing a config file:

1. **Now:** one terminal command, `agy plugin install https://github.com/HR-Monster/zacau-antigravity`
2. **After Google approves it:** a one-click install from Antigravity's **Marketplace** (Customizations › Marketplace › **+**)

Antigravity has **no deep link or URL scheme** for installing MCP servers. A plugin is the only route to one-click setup.

## What exists already

- **Frontend (Zacau-Frontend, branch `feat/issue-243`, commits `ea7e04e` and `070054a`, not pushed yet):** Settings › Integrations › "View setup guide" has an **Antigravity** tab. Its main button, "Copy config for Antigravity", copies:
  ```json
  { "mcpServers": { "zacau": { "serverUrl": "<MCP URL>" } } }
  ```
  - Built in `src/lib/mcp.js` (`ANTIGRAVITY_CONFIG`) and `src/components/settings/ConnectAiClientCards.jsx` (`ConnectAntigravityCard`).
  - The card shows Connected, last activity and Disconnect from `GET/DELETE /api/auth/ai-connections`.
- **Backend MCP server (zacau-backend, project `Zacau.Mcp.Remote`):** Streamable HTTP at `/mcp`. OAuth 2.1 with PKCE and dynamic client registration (DCR) at `/oauth/register`.
  - DCR accepts **any** redirect URI the client registers (`McpRemotePublicClientRegistry.RegisterDynamicClientAsync`), so Antigravity's `https://antigravity.google/oauth-callback` should be accepted.
  - **Not tested end to end yet.**

## MCP URLs (from Zacau-Frontend `.github/workflows/`)

| Env | URL |
|---|---|
| prod | `https://zacau-mcp-epg2bhgvcefhd0ad.southeastasia-01.azurewebsites.net/mcp` |
| dev | `https://zacau-mcp-dev-gjdwemepdyf0hzaj.southeastasia-01.azurewebsites.net/mcp` |

- **Use prod in the plugin.** A plugin holds one fixed URL, so dev and staging users keep the copy-config card.
- **Ask the user** whether a custom domain (e.g. `mcp.zacau.…`) is planned before publishing. The Azure hostname would be baked into the Marketplace listing.

## Plugin spec (from https://antigravity.google/docs/build-plugins/, read 2026-11-23)

Minimum layout:
```
plugin.json          # required (or .antigravity-plugin/plugin.json, which takes precedence)
mcp_config.json      # the MCP server
README.md            # required for Marketplace
assets/logo.png      # bundled logo, required for Marketplace
```

`plugin.json` fields needed for the Marketplace:
- `name`
- `displayName`
- `description`
- `version`
- `logo` (relative path to the bundled asset)
- `suggestedPrompts`
- `author`

Optional: `repository`, described as a "Public `https://` source code repository URL"; set it to the repo URL.

`mcp_config.json`:
```json
{ "mcpServers": { "zacau": { "serverUrl": "https://zacau-mcp-epg2bhgvcefhd0ad.southeastasia-01.azurewebsites.net/mcp" } } }
```

Rules:
- **Transport:** only Streamable HTTP is supported. Use the key `serverUrl` (`url` also works); **never** `httpUrl`.
- **No `oauth` block:** don't add `oauth.clientId`. Antigravity resolves credentials in this order: manual `oauth.clientId`, then CIMD (Client ID Metadata Documents), then DCR. Our server supports DCR.
- **No headers or secrets:** sign-in happens through Zacau OAuth in the browser.

**Naming:** Antigravity shows servers bundled in a plugin as `<plugin-name>_<server-key>`.
- With `name: "zacau"` and key `zacau`, the server shows as `zacau_zacau`.
- Pick something cleaner, e.g. key `tasks` (giving `zacau_tasks`). **Confirm with the user.**

**Validate:** `agy plugin validate --strict .` must report **zero errors and no unresolved warnings**. Run it if `agy` is installed; if not, ask the user to run it, e.g. `! agy plugin validate --strict .`.

## Assets

- **Logo:** `Zacau-Frontend/public/zacau-logo.png` (470 KB). Consider a smaller square copy for `assets/logo.png`. `public/favicon.svg` is also available.
- **Suggested prompts:** keep them close to the frontend's Claude prompt, "What do I have to do today in Zacau?". For example:
  - "What do I have to do today in Zacau?"
  - "Create a Zacau task to review the PR tomorrow"
  - "Summarize my overdue Zacau tasks"

  Check them against the backend's actual MCP tool list, in `Zacau.Mcp.Remote/McpRemoteApplication.cs`.

## README contents

- what Zacau is and what the plugin does
- install: CLI command, Marketplace (once listed), and the manual `mcp_config.json` fallback
- sign-in: Antigravity App › Customizations › Installed › MCP Servers › **Authenticate** › sign in to Zacau in the browser › paste the code into **Enter authorization code** › **Submit**
- how to disconnect: Sign out in Antigravity, or Zacau › Settings › Integrations › Antigravity › Disconnect
- privacy policy and support contact links. **Ask the user for these URLs.**

## Steps

1. Scaffold the files above in `HR-Monster/zacau-antigravity`. The repo has no branch yet, so the first commit creates `main`.
2. Validate with `agy plugin validate --strict`.
3. Test end to end:
   - `agy plugin install https://github.com/HR-Monster/zacau-antigravity` (or a local path)
   - Authenticate and sign in
   - Ask a suggested prompt
   - Check that Zacau-Frontend › Settings › Integrations › Antigravity shows **Connected**
4. If Connected doesn't show, that's a classification issue (see risks below), not a plugin bug. Note what Antigravity registered as `client_name` and report it.
5. Commit, then push. Push and anything public need the user's OK.
6. The **user** submits Google's partner intake form at https://forms.gle/2EX5RFYPoJe1UgxR9. It's a multi-page Google Form; only page 1 (Email) could be read, so the later questions are unknown. Prepare the answers (repo URL, description, logo, contact) for the user rather than submitting it yourself.

## Follow-up in Zacau-Frontend (after the repo is live)

- **Now:** in the Antigravity card, add the `agy plugin install https://github.com/HR-Monster/zacau-antigravity` command as a copyable option next to "Copy config for Antigravity". New text needs keys in all 7 locales under `src/i18n/locales/*/settings.json` → `integrations.clients.antigravity` (rule in `AGENTS.md`).
- **After Marketplace approval:** change the main button for prod to a Marketplace instruction or link. Dev and staging keep copy-config.

## Risks and open questions

- **OAuth not tested with Antigravity.** One public report (the mowteng server, Nov 2026) says `serverUrl` with DCR sign-in worked on macOS. Google's docs only show a fixed `clientId` example.
- **The Connected badge depends on the OAuth client name.** The backend's `AiConnectionsService.Classify` only knows `claude`, `chatgpt` and `other`. The frontend's `resolveAiClient` in `src/components/settings/AiClientCard.jsx` maps `clientName` matching `/antigravity/i` to Antigravity.
  - If Antigravity registers a different name, the badge stays "Not connected".
  - Fix: match the redirect URI `antigravity.google/oauth-callback` in the backend classifier, adding `"antigravity"` as an app value.
- **Marketplace review time is unknown.** Google publishes no timeline or criteria (the catalog is "curated"). Recent submissions were still pending after about 5 days. Plan for several weeks.
- **Plugin `name` is effectively permanent** once listed. Confirm it with the user.

## Sources

- https://antigravity.google/docs/build-plugins/
- https://antigravity.google/docs/plugins/
- https://antigravity.google/docs/mcp/
- https://discuss.ai.google.dev/t/mcp-store-listing-request-mowteng-remote-mcp-server-oauth/202564
- https://github.com/palmier-io/palmier-pro/issues/484 (no Antigravity deep link)
