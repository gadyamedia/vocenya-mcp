---
name: vocenya-front-desk-review
description: 'Run a daily front-desk review of a business''s Vocenya account: summarize yesterday''s calls, chats and new leads, flag urgent or unhappy callers, and draft follow-ups for the team, without contacting anyone unless the owner approves.'
---

# Vocenya front-desk review

Use this when a business owner wants a regular summary of what their Vocenya AI receptionist handled, or asks "what happened on the phones yesterday?". It uses the `vocenya` MCP server (https://vocenya.com/mcp/platform), connected to one business account by OAuth or an API key.

## Before you start

- The owner connects the account by signing in with OAuth, or gives an API key from https://vocenya.com/app/developers. For a review, read scopes are enough: `calls:read`, `leads:read`, `chats:read` and `bookings:read`.
- If a tool answers `missing_scope`, tell the owner which scope to add. Do not ask for write scopes just in case.
- Accounts in HIPAA mode never return transcripts, AI summaries or chat messages. Report counts and times only, and do not try to work around it.

## The review

1. List calls, chats, new leads and bookings since the last review (default: the previous calendar day in the business's time zone).
2. Group them: new leads to call back, bookings made or changed, questions the receptionist could not answer, calls that were transferred or missed, and anything that sounded urgent or unhappy.
3. For each lead to call back, write one line: name, number, what they wanted, and when they asked to be called.
4. List the questions the receptionist could not answer, so the owner can add the answers in Vocenya under Knowledge.
5. Draft follow-up notes for the team. Keep them as drafts.

Keep the summary short and plain, most important first. Link each call or chat with the URL the tools return.

## Approval boundary

Never do these without the owner's explicit approval in the conversation, every time:

- queue an outbound call (`outbound:write`), even to a lead who asked for a callback;
- add or remove a number on the Do Not Call list (`dnc:write`);
- create or change leads (`leads:write`) or send a chat reply (`chats:write`);
- contact a customer in any other way.

Outbound calls Vocenya places still follow its own consent, Do Not Call and calling-hours rules: https://vocenya.com/docs/outbound-calls

## Links

- MCP setup, including Grok and Grok Bot: https://vocenya.com/docs/mcp
- REST API reference: https://vocenya.com/docs/reference
