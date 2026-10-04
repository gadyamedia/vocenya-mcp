# Vocenya MCP servers

<!-- Badges. Uncomment each line once the listing it points to is live (see badges.md in the submission kit). -->

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![MCP Registry](https://img.shields.io/badge/MCP%20Registry-com.vocenya-1f3a2b)](https://registry.modelcontextprotocol.io/v0.1/servers?search=com.vocenya)
<!-- [![Vocenya Docs MCP connector](https://glama.ai/mcp/connectors/com.vocenya/docs/badges/score.svg)](https://glama.ai/mcp/connectors/com.vocenya/docs) -->
<!-- [![Smithery](SMITHERY_BADGE_URL)](SMITHERY_SERVER_URL) -->
<!-- [![MCP Market](MCPMARKET_BADGE_URL)](MCPMARKET_LISTING_URL) -->

Connect Claude, ChatGPT, Cursor, VS Code and other AI assistants to [Vocenya](https://vocenya.com), the AI receptionist for small businesses made by GH Business Solutions, and to its developer docs.

Vocenya answers phone calls and website chats 24/7, books appointments, captures leads, hands calls to trained people and places outbound AI calls with Do Not Call, consent and calling-hours checks. It runs three remote [Model Context Protocol](https://modelcontextprotocol.io) servers over Streamable HTTP. Nothing to install: point your client at a URL.

Full guide: **https://vocenya.com/docs/mcp**

| Server             | URL                                | Auth             | What it does                                                                                                              |
| ------------------ | ---------------------------------- | ---------------- | ------------------------------------------------------------------------------------------------------------------------- |
| Vocenya Docs       | `https://vocenya.com/mcp/docs`     | None             | Search the Vocenya developer docs and get every REST endpoint with code samples. Read-only.                               |
| Vocenya (platform) | `https://vocenya.com/mcp/platform` | OAuth or API key | One business account's calls, leads, bookings and chats; add leads, queue outbound AI calls, update the Do Not Call list. |
| Vocenya Site       | `https://vocenya.com/mcp/site`     | None             | Answers about Vocenya itself from vocenya.com: plans and prices, products, industries, locations, guides. Read-only.      |

The public servers allow 60 requests a minute per IP address. The platform server shares the API key's REST limit (120 requests a minute per key by default).

This repository holds the official MCP Registry entries, a Claude Code plugin and these instructions. The servers are hosted by Vocenya; their code is not here.

## Tools

### Vocenya Docs (no key, read-only)

| Tool             | What it does                                                                                                         |
| ---------------- | -------------------------------------------------------------------------------------------------------------------- |
| `search_docs`    | Search guides (authentication, webhooks, pagination, outbound calling rules, live chat, MCP) and every API endpoint. |
| `get_doc_page`   | One docs page as Markdown, for example `quickstart`, `webhooks` or `reference/leads`.                                |
| `list_endpoints` | REST endpoints with method, path, summary and required scope, grouped by resource.                                   |
| `get_endpoint`   | One endpoint in full: scope, parameters, request body, responses and code samples in cURL, Node, PHP and Python.     |

### Vocenya platform (OAuth or API key)

| Tool                  | Scope                                                 | What it does                                                                                       |
| --------------------- | ----------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| `list_calls`          | `calls:read`                                          | Calls handled by the AI, GH Live agents or the team, newest first.                                 |
| `get_call`            | `calls:read`                                          | One call with its leads; AI summary and transcript unless the account is in HIPAA mode.            |
| `list_leads`          | `leads:read`                                          | Leads from calls, chats and forms.                                                                 |
| `get_lead`            | `leads:read`                                          | One lead with contact details, reason, urgency and source.                                         |
| `create_lead`         | `leads:write` (+ `outbound:write` for an AI callback) | Add a lead from another system; optionally ask the AI to call back.                                |
| `list_bookings`       | `bookings:read`                                       | Appointments the AI booked into the business calendar.                                             |
| `list_chats`          | `chats:read`                                          | Website live chat conversations.                                                                   |
| `queue_outbound_call` | `outbound:write`                                      | Ask the AI to call someone who consented. Compliance is checked first; refusals return the reason. |
| `add_do_not_call`     | `dnc:write`                                           | Add a number to the business's Do Not Call list.                                                   |

Each tool checks its own scope (the API key's, or the scopes an owner approved when connecting with OAuth), so a key with only `leads:read` can read leads but cannot queue calls. Test keys (`vk_test_...`) work on separate sample data and never ring anyone. Accounts in HIPAA mode never return transcripts, AI summaries or chat messages through MCP.

### Vocenya Site (no key, read-only)

`get_pricing`, `get_service`, `search_industries`, `get_industry`, `list_locations`, `search_articles`, `get_article`, and the resource `vocenya://llms.txt`.

## Authentication

- **Docs and Site:** none.
- **Platform, OAuth (no key):** clients that sign in with OAuth (Claude Code, Claude connectors, ChatGPT developer mode and others) just need the URL. The client registers itself, then an account owner signs in to Vocenya and chooses the account, live or test data, and the permissions (the same scopes as API keys). Owners see and disconnect connected apps on the Developers page: https://vocenya.com/app/developers. The server supports OAuth 2.1 authorization code with PKCE (S256) and dynamic client registration.
- **Platform, API key:** `Authorization: Bearer vk_live_...` (or `vk_test_...` for test data). The account owner creates keys, each with its own scopes, on the same Developers page. Keep keys in environment variables; never paste them into a chat or commit them.

## Connect

### Claude Code

```bash
# Docs (no key)
claude mcp add --transport http vocenya-docs https://vocenya.com/mcp/docs

# Platform with OAuth (Claude Code opens the Vocenya sign-in)
claude mcp add --transport http vocenya https://vocenya.com/mcp/platform

# Or platform with an API key in an environment variable
claude mcp add --transport http vocenya https://vocenya.com/mcp/platform \
  --header "Authorization: Bearer $VOCENYA_API_KEY"
```

Or install the plugin from this repository, which adds the docs server and two skills (REST API and webhooks, website live chat install):

```bash
claude plugin marketplace add gadyamedia/vocenya-mcp
claude plugin install vocenya@vocenya
```

### Claude Desktop and claude.ai

Docs server: Settings > Connectors > Add custom connector, URL `https://vocenya.com/mcp/docs`, no authentication.

Platform server: add a custom connector with the URL `https://vocenya.com/mcp/platform` and sign in with your Vocenya owner account (OAuth).

Or, with an API key, through the `mcp-remote` bridge in `claude_desktop_config.json`:

```json
{
    "mcpServers": {
        "vocenya": {
            "command": "npx",
            "args": [
                "-y",
                "mcp-remote",
                "https://vocenya.com/mcp/platform",
                "--header",
                "Authorization:${VOCENYA_AUTH}"
            ],
            "env": {
                "VOCENYA_AUTH": "Bearer paste-your-key-here"
            }
        }
    }
}
```

### Cursor

`~/.cursor/mcp.json`:

```json
{
    "mcpServers": {
        "vocenya-docs": { "url": "https://vocenya.com/mcp/docs" },
        "vocenya": {
            "url": "https://vocenya.com/mcp/platform",
            "headers": { "Authorization": "Bearer ${env:VOCENYA_API_KEY}" }
        }
    }
}
```

### VS Code

`.vscode/mcp.json` (VS Code asks for the key once and stores it securely):

```json
{
    "inputs": [
        {
            "type": "promptString",
            "id": "vocenya-key",
            "description": "Vocenya API key (vk_live_... or vk_test_...)",
            "password": true
        }
    ],
    "servers": {
        "vocenya-docs": {
            "type": "http",
            "url": "https://vocenya.com/mcp/docs"
        },
        "vocenya": {
            "type": "http",
            "url": "https://vocenya.com/mcp/platform",
            "headers": { "Authorization": "Bearer ${input:vocenya-key}" }
        }
    }
}
```

### ChatGPT

Turn on developer mode, then add a custom connector with the URL `https://vocenya.com/mcp/docs` and no authentication. For your account data, add `https://vocenya.com/mcp/platform` with OAuth and sign in with your Vocenya owner account.

### Any other client

Any client that supports Streamable HTTP works. For the platform server, sign in with OAuth, or send `Authorization: Bearer <your key>`.

## Example prompts

- "Using the Vocenya docs, how do I verify a webhook signature? Show Node code."
- "Which scope do I need to create a lead, and what does the request body look like?"
- "Show the leads my AI receptionist captured this week." (platform)
- "What appointments did the AI book for next Monday?" (platform)
- "How much does Vocenya cost and how many AI minutes does each plan include?" (site)

## Official MCP Registry

The files in [`registry/`](registry) are the entries published to the [official MCP Registry](https://registry.modelcontextprotocol.io) under the `com.vocenya` namespace (`com.vocenya/docs`, `com.vocenya/platform`, `com.vocenya/site`).

## Links

- MCP guide: https://vocenya.com/docs/mcp
- Developer docs: https://vocenya.com/docs
- API reference: https://vocenya.com/docs/reference (OpenAPI: https://vocenya.com/docs/api.json)
- Test mode: https://vocenya.com/docs/test-mode
- Docs for LLMs: https://vocenya.com/docs/llms.txt and https://vocenya.com/docs/llms-full.txt
- Pricing: https://vocenya.com/pricing
- Privacy: https://vocenya.com/privacy · Terms: https://vocenya.com/terms

## Privacy

The docs and site servers are read-only and cannot see any customer account. The platform server sees only the one business account that owns the API key (or that an owner chose when connecting with OAuth), and only what the approved scopes allow. HIPAA mode is a paid add-on with a signed BAA; connecting an AI client does not make that client covered by the BAA, so only send patient data to AI tools you have your own agreements with. Full policy: https://vocenya.com/privacy

## Support, security and license

- Questions and corrections: open an issue, or email info@gh-businesssolutions.com
- Security reports: see [SECURITY.md](SECURITY.md)
- License: [MIT](LICENSE) for the contents of this repository. The Vocenya service itself is a hosted commercial product under the [Vocenya terms](https://vocenya.com/terms).

Copyright (c) 2026 DRAKB LLC dba GH Business Solutions.
