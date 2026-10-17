# gtm_tracking_for_salla

Prompts and skills that let an LLM agent set up ad-platform tracking through Google Tag Manager on **Salla** stores.

| Platform | Folder | Skill |
|---|---|---|
| ChatGPT Ads (OpenAI pixel) | [`chatgpt/`](chatgpt/) | `chatgpt-ads-pixel-gtm-salla` |
| Meta (Facebook) Pixel | [`meta/`](meta/) | `meta-pixel-gtm-salla` |
| Snapchat (Snap) Pixel | [`snapchat/`](snapchat/) | `snap-pixel-gtm-salla` |
| TikTok Pixel | [`tiktok/`](tiktok/) | `tiktok-pixel-gtm-salla` |

Each folder holds a paste-and-run prompt for any LLM agent and the same workflow packaged as a Claude Code skill (`SKILL.md` + `reference.md`).

## How they fit together

- **One shared foundation.** All four use the same Salla dataLayer variables and triggers, named `EA - Core - salla_…` and `EA - Trigger - <Salla event>`, in a folder `EA - Core`. Whichever runs first creates them; the rest reuse them. Run them in any order on the same container.
- **Reuse, never duplicate.** Existing triggers and variables for the same Salla event or dataLayer path are reused whatever they are called.
- **Nothing is published.** Each build happens in its own workspace and stops for you to test in GTM Preview.

## Install every skill (Claude Code)

```bash
for d in */skill/*/; do cp -r "$d" ~/.claude/skills/; done
```

## Conventions

- Tags `EA - <Platform> - <event>`, with the platform's own event name. Change the `EA` prefix to your own if you fork this.
- Pixel IDs live only in a Constant variable, never hardcoded in a tag.
- One purchase source on every platform: Salla's `Order Completed`, valued at the amount paid including shipping, so the platforms agree with each other.

## Notes

- All IDs, product data and customer values in these files are fictional.
- Template IDs (`cvt_…`) are the values seen when these were written; each build reads the real one from the installed template.
- Platform behaviour changes. Check the result in each platform's own test-events tool before relying on the data.
