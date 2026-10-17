---
name: snap-pixel-gtm-salla
description: Use when asked to install, set up or audit the Snapchat (Snap) Pixel or Snap conversion events through Google Tag Manager on a Salla store, using a GTM MCP server.
---

# Snap Pixel via GTM — Salla Stores

## Overview

Every Salla store pushes the same dataLayer, so this build is fixed: nine Snap Pixel tags on Snap's official "Snap Pixel" gallery template, each fed by exactly one Salla event. Snap is the only platform here with standard events for list views and logins, so it uses two Salla events the others don't. Nothing about events or field mapping needs asking — only the inputs below.

The Salla dataLayer, the shared `EA - Core` foundation, every GTM payload and the post-publish checklist live in `reference.md` (the REFERENCE section below if this is a pasted prompt). Read it before Step 1.

## Step 0 — Inputs gate

```
GTM account ............ <name or ID>
GTM container .......... <name or GTM-XXXXXXX>
Snap Pixel ID .......... <UUID, from Snapchat Ads Manager → Events Manager>
```

If any is missing: ask for it and **stop**. No GTM reads, no browsing.

## What gets built

| Tag | Snap `event_type` | Salla dataLayer event | Data |
|---|---|---|---|
| `EA - Snap Pixel - PAGE_VIEW` | PAGE_VIEW | All Pages (`2147479553`) — also initialises the pixel | — |
| `EA - Snap Pixel - LIST_VIEW` | LIST_VIEW | `impressions` | product IDs on list pages |
| `EA - Snap Pixel - VIEW_CONTENT` | VIEW_CONTENT | `Product Viewed` | full |
| `EA - Snap Pixel - SEARCH` | SEARCH | `Products Searched` | search term |
| `EA - Snap Pixel - ADD_TO_WISHLIST` | ADD_TO_WISHLIST | `Product Added to Wishlist` (success = true) | event only — Salla sends no product |
| `EA - Snap Pixel - ADD_CART` | ADD_CART | `Product Added` | full |
| `EA - Snap Pixel - START_CHECKOUT` | START_CHECKOUT | `checkout::started` | value + currency only |
| `EA - Snap Pixel - PURCHASE` | PURCHASE | `Order Completed` | full, `transaction_id` + `client_dedup_id` = order ID |
| `EA - Snap Pixel - LOGIN` | LOGIN | `auth::logged.in` (success = true) | — |

Plus, only if the store needs it: `EA - Snap Pixel - ADD_CART - Bought Together` (reference: bought-together check).

Fixed decisions — do not re-litigate:
- **Snap's template loads the pixel itself** (`snaptr('init')`); no base Custom HTML tag.
- `item_ids` is a real **array** of Salla product IDs, never a comma-joined string.
- Advanced Matching sends **plain** `user_email` = `customer.email` and `user_phone_number` = `customer.phone`; Snap normalises and hashes them. Name and `external_id` fields stay empty — Salla has no plain name and no external ID.
- Purchase value = `0.prices.total`, the amount paid including shipping.
- ADD_TO_WISHLIST carries no product data. Never fill it from `0.id`.
- **No payment-info event.** Salla pushes `payments::apple-pay.start-transaction` only for Apple Pay's "Pay now"; card and mada payments send nothing. ADD_BILLING built on it would count a fraction of payments, so it is deliberately not built.
- **Never set `gateway_url`.**

## Workflow

Follow `reference.md` → **PROCEDURE**, Steps 1–10, in order:

1. Locate the container and create the workspace `EA - Snap Pixel` (Step 1).
2. Live-site checks when a browser tool exists: dataLayer name, native Salla Snap integration (`store.snapchat_pixel_id`) or an `sc-static.net` script not loaded by GTM, search frequency, the `impressions` shape on a category page, bought-together check (Step 2). A native integration means double counting — stop and warn.
3. Read the Snap Pixel template's tag type from `___INFO___` (Step 3).
4. Audit triggers, variables and tags — every page (Step 4).
5. Folders → `EA - Core` foundation (create only what is missing) → Snap variables → tags (Steps 5–8).
6. Verify, compile with `quickPreview`, report, stop (Steps 9–10).

## Hard rules

1. **Never publish and never create a container version.** Build, verify, report, stop.
2. **Reuse before creating.** A Custom Event trigger for the same Salla event, or a Data Layer Variable on the same path, is reused whatever it is called (older builds use `ES - salla_*`).
3. **Never rename, edit or delete anything that already existed.**
4. **Naming:** tags `EA - Snap Pixel - <SNAP_EVENT_TYPE>` (Snap's own constants); Snap variables `EA - Snap Pixel - <snake_case>`; shared variables `EA - Core - salla_<snake_case>`; new triggers `EA - Trigger - <Salla event>`.
5. **The Pixel ID lives only in `EA - Snap Pixel - pixel_id`.**
6. Every entity gets a folder and a `notes` value at creation time.
7. Never read `payments::apple-pay.authorized.init` or send `customer.name`.

## Common mistakes

| Mistake | What actually happens |
|---|---|
| Setting `gateway_url` | Pins the script URL against template updates and implies a Conversions API Gateway that doesn't exist. Leave it unset; the UI showing a default is cosmetic. |
| Adding `uuid_c1` / `_scid` | Server-side (Conversions API) parameter. The browser pixel sets `_scid` itself. |
| `item_ids` as `"1,2,3"` | Must be an array. Use the ID variables. |
| Purchase also on `purchase` or `mobile::order.placed` | Every order counted two or more times. `Order Completed` is the only source. |
| ADD_CART price from `0.value` | Left over from `Product Viewed`. Use `EA - Core - salla_add_to_cart_value`. |
| Trusting LIST_VIEW without checking `impressions` | No raw sample of that push exists. Confirm `ecommerce.impressions[].id` in Preview or the browser first. |
| `update` with a partial object | `update` replaces the entity. Send the full object plus the current `fingerprint`. |
| Reading `gtm_template list` or `quickPreview` into context | Both are 100k+ characters. Save and search the file. |

## Report

Container, workspace and folder IDs · template ID used · created versus reused · live-site check results (or which checks could not run), including the `impressions` shape · the partial events and why · that `gateway_url` was left untouched · "nothing is published". Then stop.
