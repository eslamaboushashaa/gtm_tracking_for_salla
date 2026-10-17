# Snapchat (Snap) Pixel via GTM — Salla

A workflow that lets an LLM agent install the Snapchat (Snap) Pixel and its conversion events on **any Salla store** through Google Tag Manager.

Every Salla store pushes the same dataLayer, so the build is fixed. The agent finds the container, reuses the triggers and variables it already has, builds the tags in a separate GTM workspace, verifies them, and stops. It never publishes.

| Snap Pixel event | Salla dataLayer event |
|---|---|
| PAGE_VIEW | every page load |
| LIST_VIEW | `impressions` |
| VIEW_CONTENT | `Product Viewed` |
| SEARCH | `Products Searched` |
| ADD_TO_WISHLIST | `Product Added to Wishlist` |
| ADD_CART | `Product Added` |
| START_CHECKOUT | `checkout::started` |
| PURCHASE | `Order Completed` |
| LOGIN | `auth::logged.in` |

## What is here

| Path | Use it when |
|---|---|
| [`snap-pixel-gtm-salla-prompt.md`](snap-pixel-gtm-salla-prompt.md) | You use any LLM agent. One self-contained file: paste it in and fill the inputs. |
| [`skill/snap-pixel-gtm-salla/`](skill/snap-pixel-gtm-salla/) | You use Claude Code. `SKILL.md` is the workflow, `reference.md` holds the Salla dataLayer, the shared foundation and every GTM payload. |

Both contain the same workflow. The prompt is the two skill files joined together.

## Requirements

- A GTM MCP server connected to the agent, with edit access to the container.
- GTM already installed on the Salla store.
- The Snap Inc. "Snap Pixel" template installed in the container from the Community Template Gallery. The agent stops and asks if it is missing.
- Optional: a browser tool. With one, the agent also checks the live store for a custom dataLayer name, a pixel Salla or the theme already loads, search behaviour and the bought-together widget.

## Inputs the agent asks for

1. GTM account (name or ID)
2. GTM container (name or `GTM-…` ID)
3. Snap Pixel ID (UUID)

## Install the skill (Claude Code)

```bash
cp -r skill/snap-pixel-gtm-salla ~/.claude/skills/
```

Then ask Claude Code to set up the Snap Pixel for a Salla store.

## What the workflow gets right

- Purchases fire only on `Order Completed` — never also on Salla's `purchase` or `mobile::order.placed` pushes, which describe the same order.
- Purchase value is the amount paid including shipping (`0.prices.total`).
- Add-to-cart value is price × quantity, because Salla leaves a stale `value` in the dataLayer.
- Wishlist events are sent without product data, because Salla's push has none — never with a guessed product.
- Nothing reads the customer's name or the Apple Pay payment token.
- No payment-info event: Salla only signals it for Apple Pay, so it would count a fraction of payments.
- `item_ids` is always a real array of product IDs.
- `gateway_url` is never set, so no Conversions API Gateway is implied.
- Purchase carries `transaction_id` and `client_dedup_id` = order ID.
- Names follow `EA - Snap Pixel - <event>`; the Pixel ID lives only in the constant `EA - Snap Pixel - pixel_id`.
- Work happens in a workspace named `EA - Snap Pixel`. Publishing is always left to you.

## After the agent finishes

1. Open GTM Preview on the `EA - Snap Pixel` workspace.
2. Walk through a search, a wishlist heart, product page → add to cart → checkout → one test order.
3. Work through the post-publish checklist at the end of `reference.md`, then publish yourself.
