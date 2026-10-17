---
name: tiktok-pixel-gtm-salla
description: Use when asked to install, set up or audit the TikTok Pixel or TikTok conversion events through Google Tag Manager on a Salla store, using a GTM MCP server.
---

# TikTok Pixel via GTM — Salla Stores

## Overview

Every Salla store pushes the same dataLayer, so this build is fixed: one Custom HTML base tag plus six event tags on TikTok's official gallery template, each fed by exactly one Salla event. Nothing about events or field mapping needs asking — only the inputs below.

The Salla dataLayer, the shared `EA - Core` foundation, every GTM payload and the post-publish checklist live in `reference.md` (the REFERENCE section below if this is a pasted prompt). Read it before Step 1.

## Step 0 — Inputs gate

```
GTM account ............ <name or ID>
GTM container .......... <name or GTM-XXXXXXX>
TikTok Pixel ID ........ <20 characters, e.g. C4XXXXXXXXXXXXXXXXXX>
```

Account or container missing: ask and **stop**. Pixel ID missing: ask once; if the user says to build anyway, build everything with every TikTok tag **paused** and say so first in the report — a live base tag with a fake ID runs `ttq.load('<garbage>')` on every page.

## What gets built

| Tag | TikTok event | Salla dataLayer event | Data |
|---|---|---|---|
| `EA - TikTok Pixel - Base Code` | loads `ttq`, sends Pageview | All Pages (`2147479553`), priority 100 | — |
| `EA - TikTok Pixel - ViewContent` | ViewContent | `Product Viewed` | full |
| `EA - TikTok Pixel - Search` | Search | `Products Searched` | search term (`query` + `search_string`) |
| `EA - TikTok Pixel - AddToWishlist` | AddToWishlist | `Product Added to Wishlist` (success = true) | event only — Salla sends no product |
| `EA - TikTok Pixel - AddToCart` | AddToCart | `Product Added` | full |
| `EA - TikTok Pixel - InitiateCheckout` | InitiateCheckout | `checkout::started` | value + currency only |
| `EA - TikTok Pixel - Purchase` | Purchase (or CompletePayment) | `Order Completed` | full, `order_id` + `event_id` = order ID |

Plus, only if the store needs it: `EA - TikTok Pixel - AddToCart - Bought Together` (reference: bought-together check).

Fixed decisions — do not re-litigate:
- **TikTok's template does not load the pixel.** The Custom HTML base tag is mandatory.
- **Purchase event name**: TikTok renamed CompletePayment → Purchase for web on 1 May 2025 (legacy name still accepted). Use `Purchase` if the installed template's event dropdown offers it, otherwise `CompletePayment`. Never type a value the dropdown doesn't contain.
- Manual contents: `enhance_ecomm: false`. Salla is not GA4-shaped; the auto-ecommerce toggle would send empty events.
- Advanced Matching in **plain** mode (`hash: "non-hashed"`) with `customer.email` and `customer.phone_with_plus`; TikTok hashes them itself.
- Purchase value = `0.prices.total`, the amount paid including shipping.
- AddToWishlist carries no product data. Never fill it from `0.id`.
- **No payment-info event.** Salla pushes `payments::apple-pay.start-transaction` only for Apple Pay's "Pay now"; card and mada payments send nothing. AddPaymentInfo built on it would count a fraction of payments, so it is deliberately not built.

## Workflow

Follow `reference.md` → **PROCEDURE**, Steps 1–10, in order:

1. Locate the container and create the workspace `EA - TikTok Pixel` (Step 1).
2. Live-site checks when a browser tool exists: dataLayer name, an existing `ttq` or `analytics.tiktok.com` script not loaded by GTM, search frequency, bought-together check (Step 2). An existing install means double counting — stop and warn.
3. Read the TikTok template's tag type from `___INFO___` and its Purchase event name from the `event` dropdown (Step 3).
4. Audit triggers, variables and tags — every page (Step 4).
5. Folders → `EA - Core` foundation (create only what is missing) → TikTok variables → base tag → event tags (Steps 5–8).
6. Verify, compile with `quickPreview`, report, stop (Steps 9–10).

## Hard rules

1. **Never publish and never create a container version.** Build, verify, report, stop.
2. **Reuse before creating.** A Custom Event trigger for the same Salla event, or a Data Layer Variable on the same path, is reused whatever it is called (older builds use `ES - salla_*`).
3. **Never rename, edit or delete anything that already existed.**
4. **Naming:** tags `EA - TikTok Pixel - <TikTokEventName>`; TikTok variables `EA - TikTok Pixel - <snake_case>`; shared variables `EA - Core - salla_<snake_case>`; new triggers `EA - Trigger - <Salla event>`.
5. **The Pixel ID lives only in `EA - TikTok Pixel - pixel_id`** — including inside the base Custom HTML.
6. Every entity gets a folder and a `notes` value at creation time.
7. Never read `payments::apple-pay.authorized.init` or send `customer.name`.

## Common mistakes

| Mistake | What actually happens |
|---|---|
| No base tag ("the template will load it") | The template's `validate()` fails with `ttq not found`; nothing is sent. |
| `enhance_ecomm: true` | Reads `ecommerce.items`, which Salla doesn't have — empty events, and the `query` / `single_multi_product` fields are disabled. |
| `contents` returning `[]` | Tells TikTok the cart is empty. The variables return `undefined` instead. |
| Purchase also on `purchase` or `mobile::order.placed` | Every order counted two or more times. `Order Completed` is the only source. |
| AddToCart value from `0.value` | Left over from `Product Viewed`. Use `EA - Core - salla_add_to_cart_value`. |
| Hashed mode with plain values (or the reverse) | Match rate collapses silently. Salla has no pre-hashed `external_id`, so plain mode is right here. |
| Publishing with a placeholder Pixel ID | Every page loads a broken pixel. Pause the tags until the real ID exists. |
| `update` with a partial object | `update` replaces the entity. Send the full object plus the current `fingerprint`. |
| Reading `gtm_template list` or `quickPreview` into context | Both are 100k+ characters. Save and search the file. |

## Report

Container, workspace and folder IDs · template ID and the Purchase event name used (and why) · created versus reused · live-site check results (or which checks could not run) · the partial events and why · that the base snippet should be diffed against TikTok Events Manager → Install Code Manually · whether tags were left paused for a missing Pixel ID · "nothing is published". Then stop.
