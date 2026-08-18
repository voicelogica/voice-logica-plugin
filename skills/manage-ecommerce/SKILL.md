---
name: manage-ecommerce
description: Connect WooCommerce, Shopify, OpenCart, Magento, PrestaShop, Megasoft, or a custom shop API and configure the shop / eshop_integration agent tool. Use when the user wants product lookup, stock, or order data on a voice agent.
---

# Manage Voice Logica e-commerce

Supported shops: **WooCommerce**, **Shopify**, **OpenCart**, **Magento**, **PrestaShop**, **Megasoft**, **custom shop API**.

## 3-layer rule

1. Connect the shop integration on the company (API key / OAuth / custom URL).
2. Enable the shop tool on the agent (`list_agent_tools` / `update_agent_tool`). Read the live name — often `eshop_integration`, or a shop-specific name such as `woocommerce` / `shopify` / `opencart` / `magento` / `megasoft` / `custom_api`. **PrestaShop** has no default voice-agent tool; use workflows / MCP and do not promise a voice tool unless `list_agent_tools` shows one.
3. Test with a Call ID. Confirm the tool ran and the agent used the result.

Never paste live shop API tokens into chat. Guide the user to paste them into the Voice Logica form.

## `eshop_integration` fields

When this tool is present, configure:

- `url` — shop / catalog endpoint the tool should call
- `interval` — how often the catalog syncs
- `aiEnabled` — whether the agent may call the shop tool live
- `keyOfId` — which product identifier the agent should ask for and send

`keyOfId` values:

- `mpn` — manufacturer part number
- `ean` — barcode / EAN
- `skroutzId` — Skroutz product id
- `groupCode` — shop group / family code
- `none` — no single id; the agent searches by name or other fields

Pick `keyOfId` from how callers actually identify products. A parts desk usually wants `mpn` or `ean`. A consumer shop often wants `none` or `ean`.

## Per-shop setup (company connection)

**WooCommerce.** Store URL (http vs https matters), name, REST consumer key/secret. Enable order items, shipping carrier, estimated delivery, customer phone/email. Phone lookup often fails on landline vs mobile — enable **phone normalization**; fall back to order ID. Missing shipping in the reply is usually the carrier field not enabled, or Woo has no shipping method.

**Shopify.** Two methods:

- **New / OAuth:** shop URL + authenticate. Simpler. Basic order lookup. May lack "Update draft order".
- **Legacy:** Admin API token from Shopify → Apps → Develop apps → Voice Logica → Configuration. Token displays **once**. If lost, regenerate (old token dies) and paste immediately into Voice Logica. Needed for full features including Update draft order.

Ask which method they use when "missing features" comes up. Uninstall requires reconnect. App Store billing may disable in-app plan changes.

**OpenCart.** Store URL, API user/key, dialect (`dreamvention` is the documented value). Wrong dialect is a common connect failure.

**Magento.** Store URL; OAuth1 keys **or** bearer. Confirm the auth mode matches their Adobe Commerce setup.

**PrestaShop.** Store URL, webservice key, language id. Mainly workflows / MCP.

**Megasoft (PRISMA Win).** siteKey, user, password, optional baseUrl + order defaults.

**Custom API.** Base URL + header name/value (+ label). Good for one-off REST backends. Failures: missing fields, wrong auth header, TLS.

## Common jobs

**"The agent cannot find products / orders."**
Check the shop is connected, the agent tool is on, `aiEnabled` is true (when that field exists), and `keyOfId` matches what callers say. Then get a Call ID and read `toolCalls`. For orders: confirm the order exists, try order ID vs phone, check phone format.

**"We changed shops / URLs."**
Update `url` on the tool. Do not leave the old host in the prompt. Retest with a Call ID.

**"Sync feels stale."**
Check `interval` and whether the last sync finished. Do not lower interval as a first guess if the tool never fired.

## After a change

Say which shop, which agent, `keyOfId`, and `aiEnabled` (if used). Ask for a test Call ID that asks about a real SKU or order.
