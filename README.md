[README.md](https://github.com/user-attachments/files/33405086/README.md)
Voxisim MCP

> Give your AI agent real phone numbers, SMS codes and connectivity. One hosted MCP server, nothing to install, and the agent can pay for what it buys.

[Voxisim](https://voxisim.com) is a privacy and connectivity platform: virtual phone numbers, eSIM, global IP telephony, and security tools. Every product can be used over a REST API or handed to an AI agent over MCP, with the same account and the same balance.

[Docs](https://voxisim.com/docs) · [OpenAPI spec](https://github.com/voxisim/voxisim-openapi) · [llms.txt](https://voxisim.com/llms.txt) · [Support](mailto:support@voxisim.com)

## Products

| Product | What it is for | Status | Guide |
|---|---|---|---|
| **Get SMS Code** | Buy a temporary phone number (one-time or rental) and receive SMS verification codes from 800+ services in 200+ countries. | Available | [get-sms-code.md](get-sms-code.md) · [Docs](https://voxisim.com/docs/get-sms-code) |
| **Second Phone** | A persistent second line with calls and messages, programmable like the rest of the platform. | Coming soon | Docs coming soon |
| **eSIM** | Travel data eSIMs for a country or a global region, installed from an LPA code, with usage and top-ups over the API. | Coming soon | Docs coming soon |
| **AI Phone Translator** | Two-way call translation with AI, so you can speak with anyone in any language. | Coming soon | Docs coming soon |

Prefer an app to an API? **GetSmsCode** for [iOS](https://apps.apple.com/us/app/get-sms-code-verify/id6476933273) and [Android](https://play.google.com/store/apps/details?id=app.getsmscode) uses the same inventory, prices and balance.

## Quick start

The server is hosted. There is no package to install, no local process and no container.

**Endpoint:** `https://api.voxisim.com/mcp` (Streamable HTTP, OAuth or bearer key)

### Claude (web and desktop)

Settings → Connectors → add a custom connector named `Voxisim` → paste the endpoint → Connect and finish the browser sign-in. Then enable Voxisim from "Search and tools" in your chat.

### Claude Code

```bash
claude mcp add --transport http --scope user \
  voxisim https://api.voxisim.com/mcp
```

Run `/mcp` in Claude Code and follow the browser authentication prompt.

### Codex (app, CLI, IDE)

`~/.codex/config.toml`:

```toml
[mcp_servers.voxisim]
url = "https://api.voxisim.com/mcp"
# Authenticate once from a terminal:
# codex mcp login voxisim
```

Restart the client and use `/mcp` to confirm the connection.

### Cursor

`~/.cursor/mcp.json` (or `.cursor/mcp.json` for one project):

```json
{
  "mcpServers": {
    "voxisim": {
      "url": "https://api.voxisim.com/mcp"
    }
  }
}
```

Open Settings → Tools & MCP and authenticate Voxisim.

## Account and API key

Browsing prices and availability is public and needs no account. You need credentials only when you buy.

There are three ways to get access. Pick one.

### Option 1: OAuth (easiest)

Connect your MCP client and approve the browser sign-in. No key to copy.

### Option 2: Create an account with your email

Create a free account with your email address on the [docs page](https://voxisim.com/docs). A sign-in link brings you back with an API key that starts with `vxs_`. The same key works for REST and MCP.

### Option 3: Create an account with one API request

If you prefer not to use OAuth, or you are setting things up from a script or an agent, create an account and get a key with a single request:

```bash
curl -X POST https://api.voxisim.com/mcp/register \
  -H 'content-type: application/json' \
  -d '{"label":"my-mcp-client"}'
```

`label` is a name for the key, so you can tell your keys apart. The response contains your new `vxs_` key. It is shown only once, so copy it right away.

Then send the key as a bearer token with every MCP request:

```
Authorization: Bearer vxs_YOUR_KEY
```

For example, in Claude Code:

```bash
claude mcp add --transport http --scope user \
  --header "Authorization: Bearer vxs_YOUR_KEY" \
  voxisim https://api.voxisim.com/mcp
```

Or in a client that takes a JSON config, such as Cursor:

```json
{
  "mcpServers": {
    "voxisim": {
      "url": "https://api.voxisim.com/mcp",
      "headers": {
        "Authorization": "Bearer vxs_YOUR_KEY"
      }
    }
  }
}
```

### About keys

Keys are stored hashed and shown only once, so save yours in a secret manager. Generating a new key replaces the current one immediately, and anything still using the old key stops working.

## Payments

Two ways to pay. The purchase request is identical either way, and MCP costs the same as REST.

### Prepaid balance

Best when you buy often.

1. Create an account and get your `vxs_` key (see above).
2. Add funds with **Add funds** in your account on [voxisim.com](https://voxisim.com). Card payments are processed by Stripe and are always made by you manually; cards are never charged automatically. For fully autonomous purchases by an agent, use x402. You can also top up with stablecoins over x402: over REST call `POST /v1/balance/topup` with `{"amount_usd": "5.00"}` and go through the same `402` → sign → retry loop, or over MCP use the `topup_balance` tool. If you have a Voxisim voucher code, redeem it with `POST /v1/vouchers/claim`.
3. Buy with your key. Each purchase is deducted from the balance.

A number bought over MCP can be read over REST and the other way round: both use the same account and the same balance.

### Accountless x402

Best for a one-off autonomous purchase, with no account or key.

[x402](https://www.x402.org) is an open payment protocol built on the HTTP `402 Payment Required` status. Your agent needs a wallet that can sign payments in one of the supported assets:

| Network | Asset |
|---|---|
| Solana | USDC |
| Solana | USDT |
| Base | USDC |

The list may grow. The `402` response always shows what is accepted at that moment.

1. Send the purchase with no credentials. The server answers `payment_required` with the payment options it accepts.
2. Your agent signs one of the offered requirements from its wallet.
3. Repeat the identical request with the signed authorization. Over REST, send it in the `X-PAYMENT` header (base64). Over MCP, call the same buy tool again with the signed `payment`.
4. The first payment creates an account tied to your wallet and returns its API key once as `receipt_token`. Save it: without it you cannot read back what you bought.

Refund rules depend on the product; see its guide.

## Security

- Keep `vxs_` keys and x402 `receipt_token` values in a secret manager, never in prompts, source control or logs.
- Use a separate key per agent so you can revoke one without touching the others, and keep only the balance an agent should be able to spend.
- Report vulnerabilities to support@voxisim.com.

## Resources

- Docs: [all Voxisim products](https://voxisim.com/docs) · [Get SMS Code](https://voxisim.com/docs/get-sms-code)
- REST API reference: [docs/api.md](https://voxisim.com/docs/api.md) · OpenAPI contract: [openapi.json](https://api.voxisim.com/openapi.json) · repo: [voxisim-openapi](https://github.com/voxisim/voxisim-openapi)
- MCP reference (connection, authentication, tool schemas): [docs/mcp.md](https://voxisim.com/docs/mcp.md)
- For LLMs and agents: [llms.txt](https://voxisim.com/llms.txt) is a machine-readable index, and [llms-full.txt](https://voxisim.com/llms-full.txt) is the full self-contained REST and MCP documentation
- Legal: [Terms of Service](https://voxisim.com/terms) · [Privacy Policy](https://voxisim.com/privacy)
- Support: support@voxisim.com

## License

MIT, see [LICENSE](LICENSE). Use of the Voxisim service is governed by the [Terms of Service](https://voxisim.com/terms).

The Voxisim name and logo are trademarks of Voxisim and are not covered by this license.
