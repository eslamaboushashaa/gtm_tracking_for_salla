---
name: chatgpt-ads-pixel-gtm-salla
description: Use when asked to install, set up or audit the ChatGPT Ads / OpenAI measurement pixel (oaiq) or ChatGPT conversion events through Google Tag Manager on a Salla store, using a GTM MCP server.
---

# ChatGPT Ads (OpenAI) Pixel via GTM — Salla Stores

## Overview

Every Salla store pushes the same dataLayer, so the build is fixed: one base tag and four funnel events, each fed by exactly one Salla event. The work is to find the container, reuse what it already has, build in a separate workspace, verify, and stop. **Never publish.**

The Salla dataLayer, the OpenAI pixel rules and all tag code live in `reference.md` (the REFERENCE section below if this is a pasted prompt). Read it before Step 2.

| OpenAI event | Salla dataLayer event | Sends |
|---|---|---|
| `page_viewed` | every page load | — |
| `contents_viewed` | `Product Viewed` | product, amount, currency |
| `items_added` | `Product Added` | product, amount (price × quantity), currency |
| `checkout_started` | `checkout::started` | amount, currency (Salla sends no products) |
| `order_created` | `Order Completed` | products, amount paid, currency, `event_id` = order ID, hashed email/phone |

## Step 0 — Inputs gate

```
GTM account + container  <account name> / <container name>
OpenAI Pixel ID ........ <from Ads Manager → Conversions>
Store URL (optional) ... <https://...>
```

- Account/container missing → ask and **stop**.
- Pixel ID missing → ask once. If the user says to build without it, store the literal `REPLACE_WITH_OPENAI_PIXEL_ID` in the constant and say so first in the report.

Resolve the container by listing accounts, then the containers in that account. Names given by people are approximate ("acme website under agency new" → account `Agency - new`, container `Acme - website`). One clear match → state it and continue. Zero or several plausible matches → ask. If a store URL and a browser tool are available, confirm the container's public ID (`GTM-XXXX`) is the one loaded on the store.

## Step 1 — Load the reference

Read `reference.md`. If you have web access, also open `https://developers.openai.com/ads/measurement-pixel` and `https://developers.openai.com/ads/supported-events`. Where the live docs differ from the reference, the live docs win — say what changed.

## Step 2 — Inventory the container, reuse first

List workspaces, folders, triggers, variables and tags. Tag and template lists can be very large: save them and search for `"name"`, `"type"`, `firingTriggerId`.

- **Triggers**: for each of the four Salla events, look for a Custom Event trigger whose event filter equals the exact name (`Product Viewed`, `Product Added`, `checkout::started`, `Order Completed`). If one exists, reuse its ID whatever it is called. Create only the missing ones.
- **Variables**: for each dataLayer path in reference section 3, look for a Data Layer Variable on that same path (older builds name them `ES - salla_*`). If one exists, use its real name inside the templates in place of the `EA - Core - …` name. Create only the missing ones.
- An existing OpenAI tag (`oaiq` in a Custom HTML tag) means the pixel is already there: report it and ask before adding anything.
- Never edit, rename or delete anything that was already in the container.

## Step 3 — Build (GTM MCP)

1. Create a dedicated workspace `EA - OpenAI Pixel`. Never build in the default workspace.
2. One atomic `bulkUpdate`, using temporary IDs (`new_1`, `new_2`, …) so entities can reference each other:
   - Folder `EA - Core` (if missing) — the shared Salla variables and triggers that were not already in the container.
   - Folder `EA - OpenAI` — everything OpenAI-specific.
   - Constant `EA - OpenAI - pixel_id` = the Pixel ID. **The ID appears nowhere else.**
   - The variables and five Custom HTML tags from reference sections 3–5.

| Entity | Name |
|---|---|
| Tags | `EA - OpenAI - init + page_viewed`, `EA - OpenAI - contents_viewed`, `EA - OpenAI - items_added`, `EA - OpenAI - checkout_started`, `EA - OpenAI - order_created` |
| OpenAI variables | `EA - OpenAI - pixel_id`, `EA - OpenAI - minor_unit_factor`, `EA - OpenAI - product_amount`, … |
| Shared Salla variables | `EA - Core - salla_product_id`, `EA - Core - salla_currency`, … |
| New triggers (only if none existed) | `EA - Trigger - <Salla event name>` |
| Folders / workspace | `EA - Core`, `EA - OpenAI` / `EA - OpenAI Pixel` |

Tag settings:
- `EA - OpenAI - init + page_viewed`: trigger All Pages (built-in ID `2147479553`), firing option **once per page**.
- The four event tags: firing option **once per event**, and the base tag as **setup tag** with `stopOnSetupFailure: true`.
- Put a `notes` line on every entity saying what it fires on and why.

## Step 4 — Verify, then stop

1. `quickPreview` the workspace and search the result for `compilerError`. The result is large; search it, do not read it whole.
2. Re-read the five tags and check each against the checklist in reference section 6.
3. **Do not create a version. Do not publish.** Publishing is the user's call, every time.

If the user wants to test by hand, add `debug: true` to **both** `init` calls (base tag and `order_created` tag), tell them the logs appear in the browser console, and remove it again when they finish. Never hand over or publish a tag with `debug: true`.

## Step 5 — Report

Container and workspace · the five tags with the trigger each uses (reused or new) · variables reused vs created · whether the Pixel ID is real or a placeholder · what was not built and why (reference section 7) · the Preview checklist: product page → add to cart → checkout → one test order, checking that amounts are in minor units and `order_created` carries the order ID · "nothing is published".

## Common mistakes

| Mistake | Fix |
|---|---|
| Sending `amount: 296` for 296 SAR | Integer **minor units**: `29600`. Use the amount variables. |
| Purchase tag also on `purchase` or `mobile::order.placed` | `Order Completed` is the only purchase source. The others double-count. |
| Reading `0.value` for add to cart | It is left over from `Product Viewed`. Use price × quantity. |
| No currency on `Product Added` | That push has none. Use `EA - Core - salla_currency`, which falls back to the store currency. |
| Raw `customer.email` / `customer.phone` / `customer.name` in a tag | Only `customer.email_hashed` and `customer.phone_hashed`, in `init`, never in `measure`. |
| `customer.phone_with_plus_hashed` | Wrong normalisation for OpenAI. Use `customer.phone_hashed`. |
| Any variable on `payments::apple-pay.authorized.init` | That push holds an address and a payment token. Never read it. |
| `"{{EA - OpenAI - pixel_id}}"` in quotes inside Custom HTML | Write `{{EA - OpenAI - pixel_id}}` bare. |
| ES6 in GTM (`const`, arrow functions, template strings) | ES5 only in Custom HTML and Custom JS variables. |
| `amount` without `currency` | Currency is required whenever amount is sent. |
| Extra fields in `contents[]` (category, brand, url, image) | Only `id`, `name`, `content_type`, `quantity`, `amount`, `currency`. |
| New trigger that duplicates an existing one | Reuse the existing trigger ID. |
| `debug: true` left in a tag | Remove it before handing over. |
| Publishing because "it compiled" | Stop at the workspace. Ask. |
