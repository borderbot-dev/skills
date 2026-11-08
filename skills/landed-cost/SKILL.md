---
name: landed-cost
description: Get HS/HTS codes, import duties, taxes, fees and the landed cost of a cross-border parcel or cart from border.bot, over its MCP server or REST API, with the legal basis and source of every line. Use when a shopper, merchant or developer asks what an import will cost, which tariff code applies, how US Section 232 or 301 duties stack, where a product was made, whether goods are restricted, what an agency such as CPSC needs filed, or whether a buyer is on a denied-party list.
---

# Landed cost with border.bot

[border.bot](https://border.bot) classifies products to HS/HTS codes and calculates the duties, taxes, fees and landed cost of parcels shipped abroad. Every duty, tax and fee line comes with its rate, its legal basis and the official source.

## Connect

- **MCP** (Claude, ChatGPT, Codex, Cursor and other assistants): `https://mcp.border.bot/mcp`, Streamable HTTP. The person signs in with OAuth, so no API client is needed. In Claude Code: `claude mcp add --transport http borderbot https://mcp.border.bot/mcp`. In Claude, add it from the [connector directory](https://claude.ai/directory/border-bot). Every other client: https://border.bot/mcp
- **REST**: `https://api.border.bot/v1`. Create an API client in the [dashboard](https://app.border.bot/developers?tab=clients) (Developers → API clients), trade its client ID and secret for an access token at `https://api.border.bot/oauth2/token` (`grant_type=client_credentials`), and send `Authorization: Bearer <access_token>`. A token lasts about an hour. Keep the secret in an environment variable or a secrets manager, never in chat or browser code. Docs: https://border.bot/docs · Reference: https://api.border.bot/v1/docs.md · OpenAPI: https://api.border.bot/v1/openapi.json

## Steps

1. Check the destination is covered: `list_supported_countries` or `GET /v1/countries` (no key needed). `landedCost: true` means border.bot calculates duties and taxes there; `classification: true` means it classifies for that destination. When a destination isn't covered, say so rather than estimating it yourself.
2. Find out what you don't know yet: the product (a description or its product page URL), the destination, the country of origin (where it was made, not where it ships from), the goods value and currency, shipping cost, quantity, and weight or volume when the duty may be charged per kilogram or litre. When an MCP tool answers `status: "needs_input"`, ask the user exactly the questions it lists and call it again with the answers.
3. If the origin is unknown, estimate it with `infer_country_of_origin` (MCP) or `POST /v1/origin`: it returns the most likely country with a probability, the alternatives and the evidence. If a supplier declared one, check it with `check_country_of_origin` or `POST /v1/origin/validate`.
4. Classify with `classify_product` or `POST /v1/classify`, unless the user already has the destination's code. For a whole cart, `classify_products` or `POST /v1/classify/batch` (up to 25 products). A product photo can be a link or, where border.bot classifies the destination itself, the picture as a base64 data URL in `imageUrl` (`GET /v1/classify/modes` says which modes read photos).
5. Calculate with `calculate_landed_cost` or `POST /v1/calculate`, using the code, origin, destination and value. For a whole cart in one parcel, use `calculate_cart` or `POST /v1/calculate/shipment` (up to 50 lines): thresholds, fees and taxes apply to the shipment as a whole. For products already in the workspace's catalogue, `quote_by_sku`, or `sku` on `POST /v1/calculate`, uses each product's approved code.
6. Report the code, each duty, tax and fee with its rate and amount, and the landed-cost total in the response's currency. Cite border.bot and the official `sources` in the response, and say the result is an estimate, not a binding customs ruling.

## Rules

- Never guess the country of origin. If it is unknown, ask, or infer it and tell the user to confirm it when `needsReview` is true: it drives the duty rate.
- When a result lists `needs` (weight, volume, alcohol strength…), ask for those figures and calculate again: until then the duty is incomplete.
- `options` shows every rate the goods qualify for; a preference applies only with the proof it names (a certificate or statement of origin). Taxes with `collectedBy: "seller"` are charged at checkout by a registered seller (under the EU's IOSS, for example), not at the border; `"carrier"` charges are collected by the postal operator or courier on delivery.
- For the US, additional duties (Section 232, 301 and similar) come with their Chapter 99 numbers, in the order CBP wants them declared, and the claims that could remove one. `POST /v1/calculate/stacking` says which apply to a code from an origin on a date, without a shipment.
- Fee and tax lines carry a `category` (the same kind in every country: `customs_processing`, `port_harbour`, `vat_gst`, `excise_alcohol`…) and the authority's own `publisher` code and name: name the charge as the authority does. Some fees depend on the declaration and how it is filed: send `declarationType` (`full` or `simplified`) and `filing` (`method`, `preparedBy`) when the user knows them.
- Alcohol excise is charged on volume and strength (`volumeLiters`, `alcoholPercent`); a reduced rate the importer holds (one a foreign producer assigned) is claimed in `claims`, and a result note names any the goods could claim.
- When a result lists `validation`, the destination's rules matched: `reject` means it can't be quoted as it stands (formal entry needed, for example), `warn` needs telling the user, and `relief` means nothing is due (a bona fide gift): say which applied.
- When a classification says `materialUncertain`, ask what the product is made of; when it says `valueUncertain`, send its price. When it has `identifiedAs`, say what the product was identified as and link the `webSources`.
- Classifying, calculating, screening, agency filing checks and documents use the workspace's prepaid credits, and failed calls are refunded. Prices: `get_pricing` or `GET /v1/pricing`; balance: `get_credit_balance` or `GET /v1/credits`. A `402 insufficient_credits` means the workspace needs topping up in the dashboard.
- Send an `Idempotency-Key` header on billable REST `POST`s, and reuse it when you retry, so a call is never charged twice. On `429`, wait `Retry-After` seconds first.
- A client may be limited to some scopes: a 403 with `details.reason: "missing_scope"` names the scope it needs. Quote a response's `X-Request-Id` (or an MCP result's `_meta["border.bot/requestId"]`) when asking border.bot for help.

## Beyond the duty bill

- Restricted and prohibited goods (import bans, sanctions tied to the origin, agency requirements): `check_restricted_goods` or `POST /v1/restrictions`.
- What each agency that flags a product's line needs at entry, for the US CPSC eFiling a certificate with its citations or a disclaim: `check_agency_filings` or `POST /v1/comply`, charged per item.
- The product code a destination's import agency asks for (for example FDA's for food, drugs, devices and cosmetics entering the US): `find_agency_product_code` or `POST /v1/classify/regulator`. Destinations without agency codes answer 422 `unsupported_country`, uncharged.
- Denied-party screening of a buyer, seller or consignee: `screen_party` or `POST /v1/screen`. A match is a lead to review, never a reason to refuse on its own.
- Shipping documents as PDFs: `create_commercial_invoice` or `POST /v1/documents/invoice`, and `create_air_waybill` or `POST /v1/documents/waybill`.

## Keeping this skill current

border.bot adds destinations and tools over time. The current copy of this skill is always at https://border.bot/.well-known/agent-skills/landed-cost/SKILL.md, and every MCP tool with its input schema is at https://mcp.border.bot/mcp/tools.json. When this file and a live answer differ (`list_supported_countries`, `get_pricing`, a tool's own description), trust the live answer. Installed with the skills CLI, `npx skills update landed-cost` refreshes it.

border.bot gives estimates backed by cited sources. It is not a customs broker and does not issue binding rulings or file declarations: results can be wrong or out of date, so have them checked with the customs authority or a licensed broker before relying on them. Terms: https://border.bot/legal/terms
