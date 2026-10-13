# Get SMS Code

> Let your AI agent buy a temporary phone number, wait for the SMS, and read the verification code.

Part of [Voxisim MCP](README.md). Connection, account creation and payments are described in the [main README](README.md); this page covers the Get SMS Code product itself. Full docs: [voxisim.com/docs/get-sms-code](https://voxisim.com/docs/get-sms-code).

## What your agent can do

```
you   › Get me a WhatsApp number and wait for the code.
agent → sms_activation_list_countries   {"country": "GB", "price": "0.42", "stock": 118}
agent → sms_activation_buy_number       {"id": "act_8f3k2", "number": "+447700900123"}
agent → sms_activation_get_code         {"codes": ["847291"], "done": true}
```

800+ services · 200+ countries · pay per number, never a subscription.

The numbers you buy here are private: they are not shared with other users. The Voxisim apps and website also offer free public numbers, which are shared and can expose their inbox history to others; those are not part of this MCP server.

## Activation or rental

| | SMS Activation | SMS Rental |
|---|---|---|
| Lifetime | About 20 minutes | A day or more |
| You get | One verification code | Multiple SMS messages |
| Best for | Account verification, OTP flows, short agent tasks | Sessions, longer tests, ongoing access |
| Tools | 5 | 9 |

If you need one code and then you are done, take an activation: it is the cheapest per code. If the number has to keep receiving messages for a while, take a rental. When in doubt, pick the shorter one.

## How it works

1. **Discover.** List services, then countries. Stock and exact USD prices come back live. Browsing is public and needs no account.
2. **Buy one number.** Pay from your prepaid balance or with x402, and keep the returned `id`. Everything after the purchase hangs off it.
3. **Get the code.** Poll every few seconds until a code arrives (`codes` is non-empty or `done` is true). The code is the one-time verification code the service sent to your number.
4. **Done or refunded.** Use the code right away. If no SMS arrives before the activation expires, you are refunded automatically.

With a rental the flow is the same, except you also pick an interval in step 2, keep receiving messages for the whole rental in step 3, and can cancel or extend it in step 4.

Before you start, connect the server and set up payment: see [Quick start](README.md#quick-start) and [Payments](README.md#payments).

## Tools

### SMS Activation (one-time number, one code)

| Tool | What it does |
|---|---|
| `sms_activation_list_services` | List services and their cheapest live options |
| `sms_activation_list_countries` | List countries with live stock and exact USD prices |
| `sms_activation_buy_number` | Buy a one-time number and return the activation `id` |
| `sms_activation_get_code` | Read status and received codes for an activation |
| `sms_activation_list_numbers` | List active or archived activations |

### SMS Rental (number for a day or more, many messages)

| Tool | What it does |
|---|---|
| `sms_rental_list_services` | List rentable services |
| `sms_rental_list_countries` | List countries for a service |
| `sms_rental_list_intervals` | List rental intervals with exact prices |
| `sms_rental_buy_number` | Rent a number for one of the offered intervals |
| `sms_rental_get_codes` | Retrieve received SMS; pass `last_id` to get only new messages |
| `sms_rental_list_numbers` | List active or archived rentals |
| `sms_rental_cancel` | Cancel an active rental (refund if inside the eligible window) |
| `sms_rental_list_extend_options` | List extension durations and prices |
| `sms_rental_extend` | Extend an active rental |

Full parameters and response fields: [MCP tools for activation](https://voxisim.com/docs/mcp.md#sms_activation), [MCP tools for rental](https://voxisim.com/docs/mcp.md#sms_rental), and the REST references [activation](https://voxisim.com/docs/api.md#sms_activation) and [rental](https://voxisim.com/docs/api.md#sms_rental). The OpenAPI contract is at [openapi.json](https://api.voxisim.com/openapi.json) and in [`voxisim-openapi`](https://github.com/voxisim/voxisim-openapi).

Funding the account has its own tool, `topup_balance`; see [Payments](README.md#payments).

## Try asking your agent

- "Find the cheapest live country for WhatsApp, buy one activation, and wait for the verification code."
- "Show me the available countries and exact current prices for a Telegram activation before buying anything."
- "Check my active SMS activations and report any code that has arrived."
- "Compare the live rental intervals for WhatsApp in the United States and show me the exact prices."
- "List my active rentals and extend the one expiring soonest."

## Refunds

- **Activation:** if no SMS arrives before the roughly 20-minute expiry, the refund is automatic. Prepaid balance is credited back, and an unsettled x402 authorization is released. No ticket, no claim.
- **Rental:** cancelling inside the eligible window refunds the rental. Outside that window it runs to the end of its term, so cancel as soon as you are done.

## REST quickstart

Prefer plain HTTP to MCP? The same account and balance work over REST. The example below uses SMS Activation. SMS Rental has the same shape under `/v1/sms/rental/...`, with an extra `hours` field when you buy.

**1. Discover (public, no account needed)**

```bash
curl 'https://api.voxisim.com/v1/sms/activation/services'
curl 'https://api.voxisim.com/v1/sms/activation/countries?service_id=SERVICE_ID'
```

**2. Buy a number with your prepaid key**

```bash
# once: create an account and get a vxs_ key
curl -X POST https://api.voxisim.com/v1/signup \
  -H 'content-type: application/json' \
  -d '{"label":"my-app"}'

curl -X POST https://api.voxisim.com/v1/sms/activation/numbers \
  -H 'authorization: Bearer YOUR_VXS_API_KEY' \
  -H 'content-type: application/json' \
  -d '{"service_id":"SERVICE_ID","country_id":COUNTRY_ID}'
```

To pay with x402 instead, send the same request without the `authorization` header, sign one entry from the `accepts` list in the `402` response, and repeat the request with `-H 'X-PAYMENT: BASE64_SIGNED_AUTHORIZATION'`. Over MCP, call the buy tool again with the signed `payment`.

**3. Poll for the code**

```bash
curl 'https://api.voxisim.com/v1/sms/activation/numbers/YOUR_ACTIVATION_ID' \
  -H 'authorization: Bearer YOUR_VXS_API_KEY'
```

| Method | Route | What it does |
|---|---|---|
| GET | `/v1/sms/activation/services` | List services and cheapest live options |
| GET | `/v1/sms/activation/countries` | List countries, stock and exact prices |
| POST | `/v1/sms/activation/numbers` | Buy a one-time number |
| GET | `/v1/sms/activation/numbers/{id}` | Read status and received codes |
| GET | `/v1/sms/activation/numbers` | List active or archived activations |

The full rental routes and every parameter are in the [OpenAPI spec](https://github.com/voxisim/voxisim-openapi).

## Notes for agent builders

- **Prices and stock are live.** Call the countries (and, for rentals, intervals) tools right before buying. Never hardcode a country because it was cheapest last week.
- **Keep the id.** Everything after the purchase hangs off the activation or rental `id`.
- **Poll every few seconds** until `codes` is non-empty or `done` is true. An activation catches one code only; buy another for a second attempt.
- **Rentals:** poll with `last_id` to avoid re-reading messages. The `hours` you send must match an interval currently on offer.
- **Branch on typed errors:**
  - `invalid_argument`: service, country or interval is stale. Re-run discovery and send a valid combination.
  - `payment_required`: expected on the x402 path. Sign one requirement and repeat the identical request.
  - `number_unavailable` or a conflict: that option just sold out. Discover again and pick another instead of replaying the buy.
  - `provider_unavailable` or rate limiting: back off and honour the retry hint. Immediate retries make it worse.

## Not building an integration?

The **GetSmsCode** apps for [iOS](https://apps.apple.com/us/app/get-sms-code-verify/id6476933273) and [Android](https://play.google.com/store/apps/details?id=app.getsmscode) do the same thing with an interface: same inventory, same prices, and the same balance as the API.
