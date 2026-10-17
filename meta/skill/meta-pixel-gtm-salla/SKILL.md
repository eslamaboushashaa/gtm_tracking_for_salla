---
name: meta-pixel-gtm-salla
description: Use when asked to install, set up or audit the Meta (Facebook) Pixel or Meta conversion events through Google Tag Manager on a Salla store, using a GTM MCP server.
---

# Meta Pixel via GTM — Salla Stores

## Overview

Every Salla store pushes the same dataLayer, so this build is fixed: eight Meta Pixel tags on the facebook "Meta Pixel" gallery template, each fed by exactly one Salla event. Nothing about events or field mapping needs asking — only the inputs below.

The Salla dataLayer, the shared `EA - Core` foundation, every GTM payload and the post-publish checklist live in `reference.md` (the REFERENCE section below if this is a pasted prompt). Read it before Step 1.

## Step 0 — Inputs gate

```
GTM account ............ <name or ID>
GTM container .......... <name or GTM-XXXXXXX>
Meta Pixel ID .......... <15–16 digits, from Events Manager>
```

If any is missing: ask for it and **stop**. No GTM reads, no browsing.

## What gets built

| Tag | Meta event | Salla dataLayer event | Data |
|---|---|---|---|
| `EA - Meta Pixel - PageView` | PageView | All Pages (`2147479553`) | — |
| `EA - Meta Pixel - ViewContent` | ViewContent | `Product Viewed` | full |
| `EA - Meta Pixel - Search` | Search | `Products Searched` | search term only |
| `EA - Meta Pixel - AddToWishlist` | AddToWishlist | `Product Added to Wishlist` (success = true) | event only — Salla sends no product |
| `EA - Meta Pixel - AddToCart` | AddToCart | `Product Added` | full |
| `EA - Meta Pixel - InitiateCheckout` | InitiateCheckout | `checkout::started` | value + currency only |
| `EA - Meta Pixel - AddPaymentInfo` | AddPaymentInfo | `payments::apple-pay.start-transaction` | Apple Pay only |
| `EA - Meta Pixel - Purchase` | Purchase | `Order Completed` | full, `eventId` = order ID |

Plus, only if the store needs it: `EA - Meta Pixel - AddToCart - Bought Together` (reference: bought-together check).

Fixed decisions — do not re-litigate:
- `content_ids` / `contents[].id` = Salla's numeric product ID (`0.id`, `product_id` on orders). Flag that it must match the catalog's retailer IDs.
- Purchase value = `0.prices.total`, the amount paid including shipping. Never `0.total` (products only).
- Advanced Matching sends the **plain** `customer.email` and `customer.phone`; `fbq` hashes them itself.
- `optInMetaCAPI` = `false` and `consent` = `true` on every tag.
- AddToWishlist carries no product data. Never fill it from `0.id`.

## Workflow

Follow `reference.md` → **PROCEDURE**, Steps 1–10, in order:

1. Locate the container and create the workspace `EA - Meta Pixel` (Step 1).
2. Live-site checks when a browser tool exists: dataLayer name, native Salla Meta integration (`store.facebook_pixel_id`), theme-injected `fbq`, search frequency, bought-together check (Step 2). A native integration or existing `fbq` means double counting — stop and warn.
3. Read the Meta Pixel template's real tag type from `___INFO___` (Step 3).
4. Audit triggers, variables and tags — every page (Step 4).
5. Folders → `EA - Core` foundation (create only what is missing) → Meta variables → tags (Steps 5–8).
6. Verify, compile with `quickPreview`, report, stop (Steps 9–10).

## Hard rules

1. **Never publish and never create a container version.** Build, verify, report, stop.
2. **Reuse before creating.** A Custom Event trigger for the same Salla event, or a Data Layer Variable on the same path, is reused whatever it is called (older builds use `ES - salla_*`).
3. **Never rename, edit or delete anything that already existed.** GTM resolves variables by name; a rename silently breaks every `{{...}}`.
4. **Naming:** tags `EA - Meta Pixel - <MetaEventName>`; Meta variables `EA - Meta Pixel - <snake_case>`; shared variables `EA - Core - salla_<snake_case>`; new triggers `EA - Trigger - <Salla event>`.
5. **The Pixel ID lives only in `EA - Meta Pixel - pixel_id`.**
6. Every entity gets a folder and a `notes` value at creation time.
7. Never read `payments::apple-pay.authorized.init` or send `customer.name`.

## Common mistakes

| Mistake | What actually happens |
|---|---|
| Tag `type` guessed as `cvt_<containerId>_<templateId>` | `400 Unknown entity type`. The type is the template's own ID from `___INFO___` (currently `cvt_5RM3Q`). |
| Purchase also on `purchase` or `mobile::order.placed` | Every order counted two or more times. `Order Completed` is the only source. |
| AddToCart value from `0.value` | It is left over from `Product Viewed`. Use `EA - Core - salla_add_to_cart_value` (price × quantity). |
| AddToCart currency from the push | `Product Added` has none. `EA - Core - salla_currency` falls back to the store currency. |
| Leaving `optInMetaCAPI` at its default | It defaults to `true` and authorises Meta to set up a Conversions API integration for the client. |
| Sending `customer.email_hashed` to `em` | Send the plain value; `fbq` normalises and hashes it. |
| `update` with a partial object | `update` replaces the entity; omitted fields are deleted. Send the full object plus the current `fingerprint`. |
| Reading `gtm_template list` or `quickPreview` into context | Both are 100k+ characters. Save and search the file. |
| Treating `429 Quota exceeded` as a bad payload | Per-minute API limit. Wait and retry. |

## Report

Container, workspace and folder IDs · template ID used · created versus reused · live-site check results (or which checks could not run) · the partial events and why · the optional WhatsApp `Contact` extra · that `content_ids` must be checked against the catalog · "nothing is published". Then stop.
