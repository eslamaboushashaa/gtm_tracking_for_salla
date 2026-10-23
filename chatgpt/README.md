# ChatGPT Ads (OpenAI) pixel via GTM — Salla

A workflow that lets an LLM agent install the ChatGPT Ads measurement pixel and its conversion events on **any Salla store** through Google Tag Manager.

Every Salla store pushes the same dataLayer, so the build is fixed. The agent finds the container, reuses the triggers and variables it already has, builds five tags in a separate GTM workspace, verifies them, and stops. It never publishes.

| OpenAI event | Salla dataLayer event |
|---|---|
| `page_viewed` | every page load |
| `contents_viewed` | `Product Viewed` |
| `items_added` | `Product Added` |
| `checkout_started` | `checkout::started` |
| `order_created` | `Order Completed` |

## What is here

| Path | Use it when |
|---|---|
| [`chatgpt-ads-pixel-gtm-salla-prompt.md`](chatgpt-ads-pixel-gtm-salla-prompt.md) | You use any LLM agent. One self-contained file: paste it in and fill the inputs. |
| [`skill/chatgpt-ads-pixel-gtm-salla/`](skill/chatgpt-ads-pixel-gtm-salla/) | You use Claude Code. `SKILL.md` is the workflow, `reference.md` holds the Salla dataLayer, the pixel rules and the tag code. |

Both contain the same workflow. The prompt is the two skill files joined together.

## Requirements

- A GTM MCP server connected to the agent, with edit access to the container.
- GTM already installed on the Salla store.

## Inputs the agent asks for

1. GTM account name and container name
2. OpenAI Pixel ID (Ads Manager → Conversions)
3. Store URL (optional — used to confirm the container is the one on the store)

## Start in one message

Connect your agent to a GTM MCP server first, then send it one line.

**ChatGPT** (installs the skill once; open a new chat afterwards, because skills load when a chat starts):

```text
Help me install the following skill into my personal ChatGPT skill directory: https://github.com/eslamaboushashaa/gtm_tracking_for_salla/tree/main/chatgpt/skill/chatgpt-ads-pixel-gtm-salla
```

If your ChatGPT has no Skills, paste [`chatgpt-ads-pixel-gtm-salla-prompt.md`](chatgpt-ads-pixel-gtm-salla-prompt.md) into the chat instead: same workflow in one file.

**Claude Code** (installs the skill once):

```text
Install the skill from https://github.com/eslamaboushashaa/gtm_tracking_for_salla/tree/main/chatgpt/skill/chatgpt-ads-pixel-gtm-salla into ~/.claude/skills/
```

Or copy it yourself from a clone of this repo:

```bash
cp -r skill/chatgpt-ads-pixel-gtm-salla ~/.claude/skills/
```

Then ask it to set up the ChatGPT Ads pixel for your Salla store and give it the three inputs listed above.

## What the workflow gets right

- Amounts are sent as integers in minor units (296 SAR → `29600`), as OpenAI requires.
- Purchases fire only on `Order Completed`, with the order ID as `event_id`, so they are never double-counted.
- Purchase value is the amount paid including shipping.
- Add-to-cart value is price × quantity, because Salla leaves a stale `value` in the dataLayer.
- Customer matching uses only the SHA-256 email and phone that Salla already provides. No raw personal data is read.
- Names follow `EA - <Platform> - <event>`; the Pixel ID lives only in the constant `EA - OpenAI - pixel_id`.
- Work happens in a workspace named `EA - OpenAI Pixel`. Publishing is always left to you.

## After the agent finishes

1. Open GTM Preview on the `EA - OpenAI Pixel` workspace.
2. Walk through product page → add to cart → checkout → one test order.
3. Publish yourself when the events look right.

## Source docs

- https://developers.openai.com/ads/measurement-pixel
- https://developers.openai.com/ads/supported-events
