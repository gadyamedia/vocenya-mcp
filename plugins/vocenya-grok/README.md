# Vocenya plugin for Grok Bot

Connects a Grok Bot to Vocenya, the AI receptionist for small businesses (https://vocenya.com), so the Bot can review what the receptionist handled and draft follow-ups for the team.

## What it adds

- **MCP server `vocenya`** at `https://vocenya.com/mcp/platform` (Streamable HTTP, OAuth or API key): one business account's calls, leads, bookings and chats; adding leads, queuing outbound AI calls and updating the Do Not Call list need write scopes.
- **MCP server `vocenya-docs`** at `https://vocenya.com/mcp/docs` (no key, read-only): the Vocenya developer docs.
- **Skill `vocenya-front-desk-review`**: a daily review of calls, chats, leads and bookings, with an approval boundary: the Bot never calls, texts or changes the Do Not Call list without the owner's approval.

## Install

Custom MCP servers in Grok are available on Grok Business and Enterprise; a team admin may need to allow them. Give your Bot this repository's plugin folder (`plugins/vocenya-grok`), or add `https://vocenya.com/mcp/platform` as a custom MCP connector. Sign in with OAuth and choose the account and permissions, or use an API key with read scopes only (`calls:read`, `leads:read`, `chats:read`, `bookings:read`) until you want the Bot to act.

A good Bot description: "Own the daily front-desk review for our business. Each morning, summarize yesterday's calls, chats and new leads from Vocenya, flag anyone who sounded urgent or unhappy, and draft follow-up notes for the team. Never queue an outbound call, add to the Do Not Call list or contact a customer without approval."

Full guide: https://vocenya.com/docs/mcp#grok-and-grok-bot

## Support

info@gh-businesssolutions.com · Privacy: https://vocenya.com/privacy · Terms: https://vocenya.com/terms

## License

MIT
