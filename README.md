# Smartlead campaign preparation for AI assistants

Inspect campaigns and explicitly create an empty campaign for manual configuration.

[Start locally](#get-started) · [Connect your assistant](#connect-your-assistant) · [Available tools](#available-tools) · [Hosted PipMCP setup](#use-pipmcp-for-production)

## Built for useful workflows

- **Build a Smartlead brief.** List campaigns visible to your API key. Make that provider response available to your assistant before it proposes a next action.
- **Use Smartlead from your existing tools.** Run `smartlead_list_campaigns` from the example CLI or expose it to a stdio MCP client using the same account configuration.
- **Add a reviewed Smartlead action.** Create an empty campaign; no leads, sequences, sender accounts or launch action are included. Review the input first; execution requires operator opt-in and per-call approval.

## Why use this package?

Get a focused Smartlead MCP integration without assembling request mappings and a stdio transport yourself. The package provides documented input schemas, provider authentication handling and an example CLI, with the same tools available to your assistant.

Reads are available independently of writes. Approved changes use operator opt-in and a per-call idempotency key; uncertain writes are not silently replayed.

## Get started

**Requirements:** Node.js 22 or newer, npm, and access to the provider account you intend to use.

```sh
git clone https://github.com/jannikco/smartlead-mcp.git
cd smartlead-mcp
npm ci --ignore-scripts
npm test
npm start
```

`npm start` shows the tool inventory and workflow offline. It does not authenticate or call the provider. The tests use provider fixtures and stdio transport; they do not modify your account.

### Configure your provider account

The current release connects directly to the provider with **your own credentials**. It does not require a PipMCP key for local execution. Configure the environment of the process or MCP client that runs the server; `.env` files are not loaded automatically.

| Environment variable | What to supply |
| --- | --- |
| `SMARTLEAD_API_KEY` | Your provider API key. The provider requires it in the HTTPS request query. |

Use your own Smartlead API key. The documented provider protocol requires api_key in the HTTPS query; avoid request URL logging. This connector does not expose launch or lead-upload actions.

[Provider credential/reference documentation](https://raw.githubusercontent.com/Smartlead-Public/docs/main/guides/campaign-setup.mdx). Keep credentials out of commits, issue reports and shared client configuration.

### Run your first provider read

After configuring the environment, run `smartlead_list_campaigns`:

```sh
npm start -- --call smartlead_list_campaigns '{}'
```

Replace sample account, record, file and project IDs with values from your authorized account. This command makes a real provider request and returns its result for you to inspect.

## Available tools

| Capability | MCP tool | Effect | Required inputs |
| --- | --- | --- | --- |
| List campaigns visible to your API key. | `smartlead_list_campaigns` | Read | None |
| Create an empty campaign; no leads, sequences, sender accounts or launch action are included. | `smartlead_create_campaign` | Approved write | `name` |

Tool schemas and bindings are in [connector.json](./connector.json). These are the selected operations this package ships.

### A practical tool sequence

1. Use `smartlead_list_campaigns` — List campaigns visible to your API key.
2. Use `smartlead_create_campaign` — Create an empty campaign; no leads, sequences, sender accounts or launch action are included.

## Connect your assistant

Add this stdio server to an MCP-compatible client. Replace the absolute path and credential placeholders. Use your client’s secret/environment controls instead of committing this configuration.

```json
{
  "mcpServers": {
    "smartlead-mcp": {
      "command": "node",
      "args": [
        "/absolute/path/smartlead-mcp/server.mjs"
      ],
      "env": {
        "SMARTLEAD_API_KEY": "YOUR_SMARTLEAD_API_KEY"
      }
    }
  }
}
```

`node server.mjs` starts the MCP server; `npm start` runs the example CLI. The server reserves stdout for MCP messages. If the client does not inherit your shell environment, pass the required variables explicitly.

## Review changes before running writes

The write tool is disabled by default. Enable `CONNECTOR_ALLOW_WRITES=true` in the server process and include this approval object in the **specific** write call after reviewing its arguments:

```json
{
  "_approval": {
    "approved": true,
    "idempotencyKey": "unique-reviewed-request-id"
  }
}
```

This object is added alongside the tool’s own required fields; it is not a complete write request. Use a new idempotency key for a different action. If an outcome is uncertain, inspect the provider before trying again. Creating a record may trigger existing provider automations even when this package does not send a message itself.

## Use PipMCP for production

The local package is useful when you want to run these selected Smartlead operations in your own process. PipMCP provides a hosted place to configure provider connections, publish tools, issue endpoint access tokens, apply server-side policy and approvals, and inspect execution/usage records. Provider credentials and hosted execution are configured in that service.

**[Import Smartlead into PipMCP](https://pipmcp.com/github/start?integration=openapi-to-mcp&utm_source=github&utm_campaign=github-smartlead-mcp&utm_content=readme)**

This provider is not listed as a ready managed template in the current availability snapshot (2026-11-08). The link opens the OpenAPI importer; it does not activate the local tools automatically.

Use [selected-openapi.json](./selected-openapi.json) as a starting point for manual import. Configure the account hostname, authentication and provider-specific requirements, then review and publish the resulting tools in PipMCP.

Local credentials are not uploaded by this repository. A hosted endpoint uses its own PipMCP endpoint access token and published tool inventory; the local and hosted credentials are separate.

## Scope and validation

- The installed package has HTTP request fixtures and MCP stdio `tools/call` tests.
- Fixture success checks client behavior; it does not establish provider authentication, paid-plan access or live account execution.
- No live provider execution is claimed for this package by these tests.
- Hosted tool names, schemas, permissions and availability can differ from this local package. A queued or accepted job/message is not proof of completion or delivery. Provider API plans, account rules and limits apply.
- See [integration.json](./integration.json) for operation-scoped evidence and the provider-specific limits above.

## FAQ

**Do I need a Smartlead account?**

You need the provider account, token and permissions required by these operations. Provider charges and API limits remain with that account.

**Does this version require a PipMCP API key?**

This package uses your provider credentials for local execution. To use the hosted service, configure the provider connection in PipMCP and use the endpoint access token issued there.

**Why is a tool missing or a request rejected?**

This repository exposes the selected tools above. Check the required environment variables, provider scopes, account/region requirements and input IDs first. Hosted endpoints have their own published inventory and may exclude tools by role or policy. Do not retry a failed write without checking its outcome.

**Is this an official provider SDK or a hosted-service guarantee?**

This is an independently maintained MCP integration. Its fixtures and source references do not imply provider endorsement, unrestricted API access, or identical hosted behavior.

## Guides and related workflows

- [Smartlead integration guide](https://jannikco.github.io/pipmcp-connectors/integrations/smartlead/)
- [Smartlead workflow and tool sequence](https://jannikco.github.io/pipmcp-connectors/workflows/smartlead/)
- [More integration workflows](https://jannikco.github.io/pipmcp-connectors/workflows/)
- [Typed LeadAdapter API client](https://github.com/jannikco/linkedin-typescript-sdk)
- [LeadAdapter Python API client](https://github.com/jannikco/linkedin-python-sdk)
- [LinkedIn API quickstart](https://github.com/jannikco/linkedin-api)
- [LinkedIn people and post search](https://github.com/jannikco/linkedin-search-api)
- [LinkedIn post engagers](https://github.com/jannikco/linkedin-post-engagers)
- [LeadAdapter LinkedIn MCP server](https://github.com/jannikco/linkedin-mcp)
- [OpenAPI to MCP import starter](https://github.com/jannikco/openapi-to-mcp)
- [Swagger to MCP import starter](https://github.com/jannikco/swagger-to-mcp)
- [Smartlead API and credential reference](https://raw.githubusercontent.com/Smartlead-Public/docs/main/guides/campaign-setup.mdx)

Maintained by [jannikco](https://github.com/jannikco). MIT for this repository’s code; dependencies retain their licenses. [Report a reproducible issue](https://github.com/jannikco/smartlead-mcp/issues) with redacted input and error details. For credentials or security issues, use [private contact](https://pipmcp.com/contact).
