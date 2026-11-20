# Trippi MCP

[Русский](README.md) · [Open Trippi](https://trippitroppa.app) · [Participant guide](https://trippitroppa.app/help/ai-assistants.html)

**Keep the trip in one place and act through your AI.** Trippi lets an assistant read permitted trip context, update cards and a step-by-step itinerary, discuss decisions, and verify the saved result on the board.

```text
https://trippitroppa.app/mcp
```

[Official MCP Registry entry](https://registry.modelcontextprotocol.io/v0.1/servers/app.trippitroppa%2Ftrippi/versions/latest)

This is the public integration kit for the hosted Trippi service: setup documentation, client examples, a portable agent skill, and MCP Registry metadata. It does not contain the full Trippi server or trip data. You do not need to clone this repository or run a local server to connect.

## Start in three steps

1. Sign in to [Trippi](https://trippitroppa.app). Create a trip or accept an organizer's invitation. Add dates, constraints and relevant files.
2. Add a remote MCP server using **Streamable HTTP** in your AI client. Enter the URL above and use OAuth. Sign in to your own Trippi account, select trips and permissions, and approve the connection.
3. Ask: “Read my trip, show the next steps and identify missing information. Do not change anything yet.” Then give a specific task and check the resulting board or Plan in Trippi.

See [client setup](docs/clients.md) if OAuth registration is unavailable. MCP support depends on the client and account/workspace permissions; a model subscription alone does not connect tools.

## What the assistant can do

- Read permitted cards, sources, extracted file text, board objects and discussions.
- Create or update cards, include them in the Plan, reorder steps and mark completion.
- Work with board objects, materials and comments, including resolved discussions.
- Manage invitations, Plan links and settings when the user's role permits it.
- Use configured assistant/provider actions. A listed action does not imply that every external integration is configured.

The server exposes 13 MCP tools. The workspace catalog currently contains 61 actions; `workspace_capabilities` returns the current schemas and role. Counts may change as the service evolves.

## Two roles, one permission model

| Role | Access |
|---|---|
| Organizer | Manages the trip, shared content, access and their own private records. |
| Participant | Reads permitted shared information, maintains their own private records and joins discussions. The server checks each object's permissions. |

Organizers do not automatically gain access to another person's private material. The AI acts as the signed-in user within the selected OAuth permissions. Connecting AI does not publish private cards.

Share the **Plan** when people only need the itinerary. Invite them to the **board** when they should help organize and discuss the trip. Creating a share link does not send it to anyone.

## Example requests

> Find trains for the dates in my trip. Check the route and stations against sources, save outward and return journeys as private Plan steps, and identify anything that is not confirmed.

> Read the arrangements and comments. Summarize unresolved decisions and suggest what to do next. Do not publish anything.

> Move this step to the following day, preserve the other details and read back the updated Plan.

A saved travel card is not a booking. Card search is not live web search. Corrections and exact dates in the conversation must be preserved.

## Portable agent instructions

Use [skills/trippi/SKILL.md](skills/trippi/SKILL.md) in a client that supports skills, or provide its text to your agent. It covers context, permissions, versions and readback. The skill does not grant access or replace the MCP connection.

To revoke access, open connection management from Help in Trippi and disconnect the client. Content already shared with the AI client may remain in that client's history.

## Compatibility

The hosted HTTPS OAuth/PKCE → read → write → readback → revoke workflow has been verified. This is not a claim that every ChatGPT, Claude or other client interface has been tested. Dynamic registration is available only when the live OAuth metadata advertises `registration_endpoint`; clients should follow that metadata.

[ChatGPT, Codex, Claude Code and other clients](docs/clients.md) · [Protocol and tools](docs/protocol.md)

The MIT license covers this integration kit. The hosted service requires a Trippi account and appropriate trip permissions.

## Forwarded bookings

In Trippi, open Settings → Connections to link your personal Telegram or enable the trip’s personal forwarding email address. Telegram `/trips` selects a trip and `/plan` reads its Plan. Forwarded text/email becomes a private review proposal. PDF/PNG/JPEG files sent to the personal bot chat (up to 10 MiB) become private cards with the original file; text PDFs are readable, images have no automatic OCR.

Once MCP is connected, ask your AI to find the forwarded materials, check dates and places, and add the same source-linked card to your private Plan without duplicates. Built-in Trippi AI is not required. A pasted prompt does not install MCP; initial connection and sign-in remain necessary. [Step-by-step help](https://trippitroppa.app/help/ai-assistants.html#materials).

## File already attached to your AI chat

With Trippi connected, attach the PDF/image and ask your AI to save the original to your trip and add the same card to Plan. File-input capable hosts can call `attach_chat_file`; other MCP clients can transfer actual bytes through `material.upload`. Refresh cached tools after upgrading. If the host only exposes extracted text and cannot transfer the original, the AI must say so and offer the Trippi or Telegram uploader. File names or summaries are not proof of a stored original. Maximum supported original size is 10 MiB; client limits may be lower.
