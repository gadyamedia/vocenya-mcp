---
name: vocenya-api
description: 'Use the Vocenya REST API, webhooks and platform MCP server for a business account: authenticate with an organization API key, pick scopes, call the endpoints, verify webhook signatures and stay within rate limits.'
---

# Vocenya API

The Vocenya REST API and platform MCP server give access to one business’s Vocenya account. Full reference: https://vocenya.com/docs/reference (Markdown: https://vocenya.com/docs/reference.md; OpenAPI: https://vocenya.com/docs/api.json).

## Authenticate

- Ask the business owner for an organization API key. They create it on the Developers page of the Vocenya portal (https://vocenya.com/app/developers) and choose its scopes. MCP clients that sign in with OAuth can instead connect to the platform MCP server and have an owner approve the same scopes. Details: https://vocenya.com/auth.md
- Send it on every request: `Authorization: Bearer vk_live_...`
- 401 means the key is missing, invalid, expired or revoked. 403 with `missing_scope` means the key needs another scope: tell the owner which one.

## Scopes

- `calls:read`: Read calls
- `leads:read`: Read leads
- `leads:write`: Create leads
- `bookings:read`: Read bookings
- `outbound:write`: Queue outbound AI calls and check their status
- `dnc:read`: Read the Do Not Call list
- `dnc:write`: Add to and remove from the Do Not Call list
- `chats:read`: Read website chats
- `chats:write`: Reply to and close website chats
- `webhooks:manage`: Manage webhooks

## Endpoints

Base URL: https://vocenya.com/api/public/v1

- `GET /api/public/v1/me` (any valid key)
- `GET /api/public/v1/calls` (scope `calls:read`)
- `GET /api/public/v1/calls/{call}` (scope `calls:read`)
- `GET /api/public/v1/leads` (scope `leads:read`)
- `GET /api/public/v1/leads/{lead}` (scope `leads:read`)
- `POST /api/public/v1/leads` (scope `leads:write`)
- `GET /api/public/v1/bookings` (scope `bookings:read`)
- `GET /api/public/v1/bookings/{booking}` (scope `bookings:read`)
- `GET /api/public/v1/outbound-calls` (scope `outbound:write`)
- `POST /api/public/v1/outbound-calls` (scope `outbound:write`)
- `GET /api/public/v1/outbound-calls/{outboundCall}` (scope `outbound:write`)
- `GET /api/public/v1/do-not-call` (scope `dnc:read`)
- `POST /api/public/v1/do-not-call` (scope `dnc:write`)
- `DELETE /api/public/v1/do-not-call/{entry}` (scope `dnc:write`)
- `GET /api/public/v1/chats` (scope `chats:read`)
- `GET /api/public/v1/chats/{chat}` (scope `chats:read`)
- `POST /api/public/v1/chats/{chat}/messages` (scope `chats:write`)
- `POST /api/public/v1/chats/{chat}/close` (scope `chats:write`)
- `GET /api/public/v1/webhook-endpoints` (scope `webhooks:manage`)
- `POST /api/public/v1/webhook-endpoints` (scope `webhooks:manage`)
- `DELETE /api/public/v1/webhook-endpoints/{webhookEndpoint}` (scope `webhooks:manage`)
- `POST /api/public/v1/webhook-endpoints/{webhookEndpoint}/test` (scope `webhooks:manage`)

Lists are newest first and cursor paginated: pass `per_page` (1 to 100) and the previous page’s `meta.next_cursor` as `cursor`. Times are ISO 8601. Errors look like `{"message": "...", "code": "..."}`; validation errors (422) add `errors`.

Outbound calls go through the same checks as calls started in the app: consent, the Do Not Call list and local calling hours. A refused call comes back with a reason code. Never try to work around a refusal.

Website live chat: `POST /chats/{chat}/messages` with `{"body": "..."}` replies as the team (the visitor sees it at once, and a chat the AI is answering is taken over); `POST /chats/{chat}/close` ends the chat. Both need `chats:write`; replies are limited to 30 a minute per key. A closed chat answers `409` with `chat_closed`. To answer chats from your own system, subscribe to `chat.message.created` and skip messages with `via_api: true` (your own replies).

## Webhooks

Events: `lead.created`, `call.completed`, `booking.created`, `chat.handed_off`, `outbound_call.completed`, `chat.started`, `chat.message.created`, `chat.lead_captured`, `chat.handoff_requested`, `chat.closed`.

Each delivery is signed in the `Vocenya-Signature: t=<timestamp>,v1=<hex>` header: HMAC-SHA256 of `<timestamp>.<raw body>` with the endpoint secret. Reject signatures older than 300 seconds. Respond with a 2xx within 10 seconds; failures are retried with exponential backoff up to 8 attempts. Use the event `id` (also in the `Vocenya-Delivery` header) to ignore duplicates.

## Rate limits

120 requests a minute per key by default, shared with the platform MCP server. Over the limit: `429` with `Retry-After` (seconds).

## MCP

- Platform server (streamable HTTP, same key and scopes): https://vocenya.com/mcp/platform. Tools: list_calls, get_call, list_leads, get_lead, create_lead, list_bookings, queue_outbound_call, add_do_not_call, list_chats.
- Site server (public, read-only, no key): https://vocenya.com/mcp/site.
- Docs server (public, read-only, no key): https://vocenya.com/mcp/docs. Tools: search_docs, get_doc_page, list_endpoints, get_endpoint.

## HIPAA mode

Accounts in HIPAA mode never return transcripts, AI summaries, chat message text or lead reasons and details, and their webhooks carry ids only. Call recordings are never available through the API. Do not ask the user to work around this.
