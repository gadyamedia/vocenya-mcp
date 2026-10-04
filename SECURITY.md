# Security policy

## Reporting a vulnerability

Email **info@gh-businesssolutions.com** with "Security" in the subject line. Please include:

- what you found and where (server URL, tool name, file in this repository)
- steps to reproduce
- what an attacker could do with it

Please do not open a public GitHub issue for security problems, and do not include real customer data, API keys or caller details in your report. If you need sample data, use a `vk_test_` key, which only ever sees test data.

We will acknowledge your report and keep you updated while we fix it.

## Scope

- The hosted MCP servers: `https://vocenya.com/mcp/docs`, `https://vocenya.com/mcp/platform` and `https://vocenya.com/mcp/site`
- The Claude Code plugin and the registry files in this repository

The Vocenya application itself is closed source. Reports about it are welcome at the same address.

## Keys

Never commit a Vocenya API key. If you think a key has leaked, revoke it on the Developers page of the Vocenya portal (https://vocenya.com/app/developers) and create a new one.
