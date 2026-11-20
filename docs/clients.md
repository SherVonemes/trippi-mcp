# Connect your AI / Подключение AI

Server URL: `https://trippitroppa.app/mcp`  
Transport: **Streamable HTTP**  
Authentication: **OAuth authorization code + PKCE**

Each participant signs in to their own Trippi account. Select the intended trips and permissions on the Trippi consent screen. Do not copy another user's access token, browser cookies or login code.

## ChatGPT

1. Open the custom MCP connection flow in Plugins/Apps. Menu names and availability depend on the account and workspace.
2. Add a custom MCP server called **Trippi**, using the URL above and OAuth authentication.
3. If discovery offers dynamic client registration, let ChatGPT register itself. If the form instead asks for a predefined public OAuth client, use Client ID `trippi-chatgpt` and leave Client Secret empty.
4. Scan/review the tools, create the connection/plugin, then sign in to Trippi and select trips and permissions.
5. Enable or mention Trippi in the conversation. Ask it to read the trip before asking it to change anything.

No separate Trippi GPT subscription is required by this connection. Whether your ChatGPT account permits custom MCP servers is controlled by OpenAI and your workspace. [Official setup guide](https://help.openai.com/en/articles/12584461-connecting-custom-mcp-servers-in-chatgpt).

## Codex

For a current Codex CLI with remote OAuth support:

```sh
codex mcp add trippi --url https://trippitroppa.app/mcp
codex mcp login trippi
```

Complete the browser sign-in and consent. Start a session that loads the connection, then ask Codex to list permitted trips. [examples/codex.toml](../examples/codex.toml) is the equivalent endpoint configuration; merge it into existing settings rather than overwriting them. Commands were checked against the current CLI's help; this guide does not claim an interactive sign-in was completed in every client version.

## Claude Code

```sh
claude mcp add --transport http trippi https://trippitroppa.app/mcp
```

Run `/mcp` in Claude Code and complete authentication for Trippi. A project-scoped configuration example is in [examples/claude-code.mcp.json](../examples/claude-code.mcp.json). Review project configuration before trusting it. [Official MCP documentation](https://code.claude.com/docs/en/mcp).

## Other clients and models

Choose a client or agent harness that supports remote Streamable HTTP, OAuth discovery, PKCE and MCP tool calls. Enter the server URL, authenticate, and verify that `list_trips` and `workspace_capabilities` are available. DeepSeek or another model can use Trippi when its hosting application supplies these MCP capabilities; a plain model chat does not automatically have them.

The JSON example is specifically for Claude Code's configuration format. Other applications may use different keys; the common endpoint and authentication remain the same.

## Registration and troubleshooting

- The live authorization metadata determines whether automatic OAuth client registration is supported. If it contains `registration_endpoint`, let the client use it. Do not invent a client ID or callback.
- If the server does not advertise registration, that client needs a separately registered exact callback and Client ID. The ChatGPT Client ID is intended only for its registered callbacks.
- A `401` response before sign-in is expected. Reauthenticate after expiry or revoked access; a configured URL is not proof of a working connection.
- Missing trips usually mean the account, membership or consent selection differs. Check the account and selected trips in Trippi.
- A forbidden operation does not mean a broken connection. Ask for an allowed action or have the organizer adjust access deliberately.
- If an external provider/search action is unavailable, use an available search tool in the AI client or describe what is missing. Do not invent a completed lookup or booking.

For a useful first check, ask: “List the trips I authorized, identify my role in the selected trip, and read the current Plan. Do not change anything.”
