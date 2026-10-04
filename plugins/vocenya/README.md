# Vocenya plugin for Claude Code

Adds the public Vocenya Docs MCP server and two skills to Claude Code, for developers building on Vocenya, the AI receptionist for small businesses (https://vocenya.com).

## What it adds

- **MCP server `vocenya-docs`** at `https://vocenya.com/mcp/docs` (Streamable HTTP, no key, read-only): `search_docs`, `get_doc_page`, `list_endpoints`, `get_endpoint`. Claude uses it to look up real endpoints, scopes, parameters and code samples instead of guessing.
- **Skill `vocenya-api`**: how to authenticate with an organization API key, pick scopes, call the REST API, verify webhook signatures and stay within rate limits.
- **Skill `vocenya-chat-install`**: how to add the Vocenya website live chat to a site, per framework, with Content Security Policy sources and the JavaScript API.

The skills are the same files Vocenya publishes at https://vocenya.com/.well-known/agent-skills/index.json.

## Install

```bash
claude plugin marketplace add gadyamedia/vocenya-mcp
claude plugin install vocenya@vocenya
```

## Your own account data (optional)

This plugin does not connect to a business account. To let Claude read your calls, leads, bookings and chats, add the platform server yourself. Without a key, Claude Code opens the Vocenya sign-in (OAuth) and an account owner approves access:

```bash
claude mcp add --transport http vocenya https://vocenya.com/mcp/platform
```

Or use an API key from https://vocenya.com/app/developers:

```bash
claude mcp add --transport http vocenya https://vocenya.com/mcp/platform \
  --header "Authorization: Bearer $VOCENYA_API_KEY"
```

Choose test data on the consent screen, or use a `vk_test_` key, to try it on sample data first. Full guide: https://vocenya.com/docs/mcp

## Support

info@gh-businesssolutions.com · Privacy: https://vocenya.com/privacy · Terms: https://vocenya.com/terms

## License

MIT. See `LICENSE`. The license covers the files in this plugin (configuration and skill text). The Vocenya service, API and MCP servers themselves are a hosted commercial product covered by the Vocenya terms of service.
