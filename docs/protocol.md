# Protocol and tool workflow

Trippi is a hosted, authenticated MCP server at `https://trippitroppa.app/mcp`.

- Protected resource metadata: `https://trippitroppa.app/.well-known/oauth-protected-resource/mcp`
- Authorization server metadata: `https://trippitroppa.app/.well-known/oauth-authorization-server`
- OAuth authorization code with S256 PKCE, resource binding, and user consent for selected trips.
- Identity scopes: `openid`, `offline_access`.
- Resource scopes: `trips:read`, `cards:write`, `workspace:read`, `workspace:write`.
- Refresh credentials are handled by the authenticated client. Never embed bearer tokens in shared config files.

Use discovery for authorization, token and client-registration endpoints. Do not assume every deployment supports the same registration mode.

## Current tool surface

| Tool | Purpose |
|---|---|
| `list_trips` | List trips selected for this connection. |
| `workspace_capabilities` | Read the current role, action catalog, input schemas and retry rules. |
| `get_workspace_context` | Read permitted shared and personal context, with pagination and coverage. |
| `read_workspace` | Execute a named read from the catalog. |
| `act_on_workspace` | Execute a named action from the catalog, subject to current permissions. |
| `get_trip_context`, `list_cards`, `get_item`, `search_cards`, `get_schedule` | Specialized compatible reads of personal cards. |
| `create_card`, `update_card`, `get_operation` | Compatible personal-card writes and operation receipts. |

The workspace catalog currently has 61 actions, covering cards, Plan, board objects, comments, files, sources, invitations, sharing, settings and configured integrations. Use its current schemas instead of hard-coding an old action list.

Read context pages until `nextCursor` is null. Check coverage and extraction status: a file being present does not mean its text was extracted. Refresh the object/version before a write. Follow each action's retry rules, then read back the changed object in the canonical service. Report partial completion explicitly.

The MCP Registry record identifies this integration as `app.trippitroppa/trippi`. Registry metadata points to the hosted endpoint; it does not distribute a standalone server binary or certify each AI client's behavior.
