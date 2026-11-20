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

The workspace catalog covers cards, Plan, board objects, comments, files, sources, invitations, sharing, settings and configured integrations. Use its current schemas instead of hard-coding an old action list.

Read context pages until `nextCursor` is null. Check coverage and extraction status: a file being present does not mean its text was extracted. Refresh the object/version before a write. Follow each action's retry rules, then read back the changed object in the canonical service. Report partial completion explicitly.

The MCP Registry record identifies this integration as `app.trippitroppa/trippi`. Registry metadata points to the hosted endpoint; it does not distribute a standalone server binary or certify each AI client's behavior.

## Forwarded booking workflow

Discover the current catalog before using these actions. `read_workspace` → `intake.pending` lists visible pending email, Telegram and web proposals without contacting the email provider. It accepts `params.limit` (1–50, default 20) and `params.cursor`; follow `nextCursor` until null. `total` is the full visible pending count. Pages also have a 60 KB soft bound (an individual item is never dropped). A changed listing invalidates the cursor with `workspace_intake_changed`; restart discovery. Already accepted cards and saved file materials are available through the normal workspace context.

Supported PDF/PNG/JPEG files sent to a linked personal Telegram chat are imported as a private source-linked document card plus original `card_materials` file, up to 10 MiB. They have no pending proposal and are initially outside Plan. Use `get_workspace_context` for available extracted pages, `card.get`/`material.list` for current state, and `material.download` for an explicit original-file read. A personal AI can update the existing card's schedule and full Plan tuple with `card.update` after the user's instruction; **do not confirm a proposal or create a duplicate card for this path**. PNG/JPEG remain `image_unread`; the server does not promise OCR. This file-import flow applies to the personal chat, not group attachments. Forwarded email and saved/forwarded Telegram text follow the pending-proposal path instead.

Read `proposal.get`, `source.get`, then explicitly use `source.email_attachment_text` for a private email source. This is a read action with `workspace:read`; its underlying canonical endpoint requires POST with an empty body and normal session protections. Each PDF attachment returns its own extraction status and pages. Provider access, parsing and extraction limits remain enforced. PNG/JPEG and scanned PDFs without extractable text are not silently treated as read. Legacy Telegram captions whose file was not imported are reported as unavailable in context coverage.

To include a reviewed booking in Plan using a personal AI, confirm the proposal without `schedule`/`scheduleProof`, read the source-linked resulting card, then update that card with the user's explicit schedule and the complete Plan tuple. These are two writes: a successful confirmation followed by a failed Plan update leaves a valid saved card to resume from. Reuse the same confirmation request ID for an identical retry, and inspect current card state before retrying an uncertain update. No built-in Trippi AI configuration or allowance is required for this manual edit workflow. The separate atomic scheduled-confirmation endpoint requires a proof issued by the built-in assistant; never invent or reuse an unrelated proof.

The [portable skill](../skills/trippi/SKILL.md) describes permissions, incomplete attachment handling and recovery. Private forwarding never automatically publishes documents to other participants. Shared publication is an explicit separate operation.
