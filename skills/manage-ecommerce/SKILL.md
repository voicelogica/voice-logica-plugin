---
name: manage-ecommerce
description: Connect WooCommerce, Shopify, OpenCart, Magento, PrestaShop, Megasoft, or a custom shop API and configure the eshop_integration agent tool. Use when the user wants product lookup, stock, or order data on a voice agent.
---

# Manage Voice Logica e-commerce

Supported shops: **WooCommerce**, **Shopify**, **OpenCart**, **Magento**, **PrestaShop**, **Megasoft**, **custom shop API**.

## 3-layer rule

1. Connect the shop integration on the company (API key / OAuth / custom URL).
2. Enable `eshop_integration` on the agent (`list_agent_tools` / `update_agent_tool`).
3. Test with a Call ID. Confirm the tool ran and the agent used the result.

Never paste live shop API tokens into chat.

## `eshop_integration` fields

Configure these on the agent tool:

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

## Common jobs

**"The agent cannot find products."**
Check the shop is connected, `eshop_integration` is on, `aiEnabled` is true, and `keyOfId` matches what callers say. Then get a Call ID and read `toolCalls`.

**"We changed shops / URLs."**
Update `url` on the tool. Do not leave the old host in the prompt. Retest with a Call ID.

**"Sync feels stale."**
Check `interval` and whether the last sync finished. Do not lower interval as a first guess if the tool never fired.

## After a change

Say which shop, which agent, `keyOfId`, and `aiEnabled`. Ask for a test Call ID that asks about a real SKU.
