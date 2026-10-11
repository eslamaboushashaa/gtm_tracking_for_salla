# ChatGPT Ads (OpenAI) Pixel via GTM — Salla Stores — Reusable Prompt

Paste everything below the line into any LLM agent that has a **GTM MCP server** connected.
Fill in the INPUTS in Step 0. Nothing else changes between Salla stores.
(Claude Code users: install the skill in `skill/chatgpt-ads-pixel-gtm-salla/` instead — same workflow.)

---

You are setting up the ChatGPT Ads (OpenAI) measurement pixel on a Salla store through Google Tag Manager. Follow the workflow below step by step, in order. "The reference" means the REFERENCE section at the end of this prompt.


## WORKFLOW


## Overview

Every Salla store pushes the same dataLayer, so the build is fixed: one base tag and four funnel events, each fed by exactly one Salla event. The work is to find the container, reuse what it already has, build in a separate workspace, verify, and stop. **Never publish.**

The Salla dataLayer, the OpenAI pixel rules and all tag code live in the REFERENCE section at the end of this prompt. Read it before Step 2.

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

Read the REFERENCE section. If you have web access, also open `https://developers.openai.com/ads/measurement-pixel` and `https://developers.openai.com/ads/supported-events`. Where the live docs differ from the reference, the live docs win — say what changed.

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

---

# REFERENCE — ChatGPT Ads (OpenAI) pixel on Salla

Pixel facts from `developers.openai.com/ads/measurement-pixel` and `/ads/supported-events`, read 2026-10-11. Salla dataLayer shapes captured from a live Salla store in 2026 (IDs and customer data below are fictional). The tag code was built and tested in GTM Preview on a live Salla store on 2026-10-11.

There is no official GTM template for this pixel — every tag is Custom HTML.

## 1. OpenAI pixel rules

- `oaiq("init", { pixelId, user })` — `pixelId` required. `debug: true` logs SDK activity to the browser console; testing only.
- `oaiq("measure", eventName, eventData, options)` — `eventData.type` must match the event. `options.event_id` is the deduplication ID.
- Dedup key = pixel ID + event name + `event_id`. The first event wins, so an `event_id` also protects against a thank-you page reload.
- `oaiq("consent", false)` before `init` blocks measurement; `oaiq("consent", true)` after opt-in. Default is granted. Blocked events are not replayed.
- Automatic: stores the `oppref` click parameter in the `__oppref` cookie (30 days), sets `__obref` (365 days), adds `source_url` and a timestamp.
- CSP, if the store sets one: `script-src https://bzrcdn.openai.com`; `connect-src https://bzr.openai.com https://bzrcdn.openai.com`; `img-src https://bzr.openai.com`.

Events used on Salla — all have `type: "contents"`, with optional `amount`, `currency`, `contents[]`:

| Event | Meaning |
|---|---|
| `page_viewed` | page load |
| `contents_viewed` | product viewed |
| `items_added` | added to cart |
| `checkout_started` | checkout begins |
| `order_created` | purchase completed |

`contents[]` item — **only these fields**: `id` (string), `name`, `content_type` (`"product"`), `quantity` (integer), `amount` (price of one unit, integer minor units), `currency`.

**Amount rules**
- Integer in the currency's ISO 4217 minor unit: 296 SAR → `29600`; 5 KWD → `5000` (3 decimals); 500 JPY → `500` (0 decimals).
- Event `amount` = value of the whole event. Item `amount` = price of **one** unit; `quantity` is sent separately.
- `amount` present → `currency` required.

**User data** — in `init` only, never in `measure`. SHA-256, 64-character lowercase hex, never raw:

| Field | Normalise before hashing |
|---|---|
| `email_sha256` | trim, lowercase |
| `phone_number_sha256` | digits with country code, no `+`, no spaces or punctuation, no leading zeros |

To add user data later in the page, call `init` again with the full `user` object.

## 2. Salla dataLayer

Product and order data sit under the **numeric key `"0"`**, so Data Layer Variable paths start with `0.` (Version 2).

```js
// Product page
{ event: "Product Viewed",
  0: { id: 100000001, name: "Sample product", category: "Sample category",
       price: 296, quantity: 1, currency: "SAR", value: 296, sku: "", url: "https://…" } }

// Add to cart — no currency, quantity is a STRING, and 0.value is left over from Product Viewed
{ event: "Product Added",
  0: { id: 100000001, name: "Sample product", price: 296, quantity: "1",
       cart_id: "…", cart_item_id: "…", coupon: "" } }

// Checkout — top-level fields, no products
{ event: "checkout::started", checkout_type: "mini_checkout", checkout_id: "…", cart_id: "…",
  value: 303, currency: "SAR", payment_methods: ["credit_card", "mada", "apple_pay"] }

// Thank-you page, pushed first — the only place the customer object exists
{ event: "purchase",
  ecommerce: { purchase: { actionField: { id: 200000001, revenue: 203 }, products: [ /* … */ ] } },
  customer: { isGuest: false, name: "…", email: "…", phone: "9665XXXXXXXX", phone_with_plus: "+9665XXXXXXXX",
              email_hashed: "<64 hex>", phone_hashed: "<64 hex>", phone_with_plus_hashed: "<64 hex>",
              first_name: "<64 hex>", last_name: "<64 hex>" } }

// Thank-you page, pushed next — the purchase source
{ event: "Order Completed",
  0: { order_id: "1000000001", reference_id: "200000001", total: 196, currency: "SAR",
       products: [{ product_id: 100000001, name: "Sample product", price: 196, quantity: 1, category: "…" }],
       prices: { total: "203.00", shipping_cost: "7", amount: "196" },
       subtotal: 203, shipping: 7, tax: 0, discount: 0, coupon: "", payment_method: "credit_card",
       event_id: "<uuid>" } }
```

Facts that decide the build:

1. **Stale data.** Salla never clears key `"0"` between pushes and GTM merges objects, so a field missing from the new push keeps its old value. `0.value` on `Product Added` is the product-page value. Always compute add-to-cart value as `0.price × 0.quantity`.
2. **`Product Added` has no currency.** Use the currency variable with the store-currency fallback.
3. **`checkout::started` has no products.** Send amount and currency only.
4. **One purchase source: `Order Completed`.** `purchase` and `mobile::order.placed` describe the same order; a tag on either double-counts.
5. **Purchase value = `0.prices.total`** — the amount paid including shipping (a string, `"203.00"`). `0.total` is products only. Fallback: `0.subtotal`.
6. **Customer data exists only in the `purchase` push**, which arrives just before `Order Completed`. Verified by hashing sample values: `customer.email_hashed` = SHA-256 of the lowercased email, and `customer.phone_hashed` = SHA-256 of the digits with country code and no plus. Both match OpenAI's normalisation and are passed through unchanged. `phone_with_plus_hashed` does **not** match.
7. `customer.first_name` / `last_name` are hashes whose normalisation is unverified — not sent. `customer.name`, `email`, `phone` are raw — never read them for this pixel.
8. **Never read `payments::apple-pay.authorized.init`.** It carries the shipping contact and the Apple Pay payment token.
9. Legacy duplicates of the events above — `detail`, `addToCart`, `cart::item.added`, `removedFromCart` — are never used.

## 3. Shared Salla variables and triggers (folder `EA - Core`)

Create only what the container does not already have (Step 2 of the workflow). If a variable on the same path already exists under another name, use that name in sections 4–5.

Data Layer Variables, Version 2, no default value:

| Name | Path |
|---|---|
| `EA - Core - salla_product_id` | `0.id` |
| `EA - Core - salla_product_name` | `0.name` |
| `EA - Core - salla_product_price` | `0.price` |
| `EA - Core - salla_product_quantity` | `0.quantity` |
| `EA - Core - salla_item_currency` | `0.currency` |
| `EA - Core - salla_event_currency` | `currency` |
| `EA - Core - salla_checkout_value` | `value` |
| `EA - Core - salla_order_id` | `0.order_id` |
| `EA - Core - salla_order_prices_total` | `0.prices.total` |
| `EA - Core - salla_order_subtotal` | `0.subtotal` |
| `EA - Core - salla_order_products` | `0.products` |
| `EA - Core - salla_customer_email_hashed` | `customer.email_hashed` |
| `EA - Core - salla_customer_phone_hashed` | `customer.phone_hashed` |

Constant `EA - Core - store_currency` = `SAR` (change only if the store sells in another currency).

Custom JavaScript `EA - Core - salla_currency`:

```js
function() {
  var c = {{EA - Core - salla_item_currency}} || {{EA - Core - salla_event_currency}};
  if (c && c !== 'undefined') { return String(c); }
  return {{EA - Core - store_currency}};
}
```

Custom JavaScript `EA - Core - salla_order_value`:

```js
function() {
  var v = Number({{EA - Core - salla_order_prices_total}});
  if (!v || isNaN(v)) { v = Number({{EA - Core - salla_order_subtotal}}); }
  if (!v || isNaN(v)) { return undefined; }
  return Math.round(v * 100) / 100;
}
```

Triggers — type Custom Event, event name **equals**, case-sensitive:

| Name (if it has to be created) | Event name | Extra condition |
|---|---|---|
| `EA - Trigger - Product Viewed` | `Product Viewed` | — |
| `EA - Trigger - Product Added` | `Product Added` | — |
| `EA - Trigger - checkout::started` | `checkout::started` | — |
| `EA - Trigger - Order Completed` | `Order Completed` | `{{EA - Core - salla_order_id}}` matches RegEx `^\d+$` |

## 4. OpenAI variables (folder `EA - OpenAI`) — ES5 only

Constant `EA - OpenAI - pixel_id` = the Pixel ID.

`EA - OpenAI - minor_unit_factor`:

```js
function() {
  var c = String({{EA - Core - salla_currency}} || '').toUpperCase();
  var three = {BHD: 1, IQD: 1, JOD: 1, KWD: 1, LYD: 1, OMR: 1, TND: 1};
  var zero = {BIF: 1, CLP: 1, DJF: 1, GNF: 1, JPY: 1, KMF: 1, KRW: 1, PYG: 1, RWF: 1, UGX: 1, VND: 1, VUV: 1, XAF: 1, XOF: 1, XPF: 1};
  if (three[c]) { return 1000; }
  if (zero[c]) { return 1; }
  return 100;
}
```

`EA - OpenAI - product_amount` (price × quantity; never reads `0.value`):

```js
function() {
  var price = Number({{EA - Core - salla_product_price}});
  var qty = parseInt({{EA - Core - salla_product_quantity}}, 10) || 1;
  if (isNaN(price) || price <= 0) { return undefined; }
  return Math.round(price * qty * {{EA - OpenAI - minor_unit_factor}});
}
```

`EA - OpenAI - product_contents`:

```js
function() {
  var id = {{EA - Core - salla_product_id}};
  if (!id) { return undefined; }
  var item = {
    id: String(id),
    content_type: 'product',
    quantity: parseInt({{EA - Core - salla_product_quantity}}, 10) || 1
  };
  var name = {{EA - Core - salla_product_name}};
  if (name) { item.name = String(name); }
  var price = Number({{EA - Core - salla_product_price}});
  if (!isNaN(price) && price > 0) {
    item.amount = Math.round(price * {{EA - OpenAI - minor_unit_factor}});
    item.currency = {{EA - Core - salla_currency}};
  }
  return [item];
}
```

`EA - OpenAI - checkout_amount`:

```js
function() {
  var v = Number({{EA - Core - salla_checkout_value}});
  if (isNaN(v) || v <= 0) { return undefined; }
  return Math.round(v * {{EA - OpenAI - minor_unit_factor}});
}
```

`EA - OpenAI - order_amount`:

```js
function() {
  var v = Number({{EA - Core - salla_order_value}});
  if (isNaN(v) || v <= 0) { return undefined; }
  return Math.round(v * {{EA - OpenAI - minor_unit_factor}});
}
```

`EA - OpenAI - order_contents`:

```js
function() {
  var p = {{EA - Core - salla_order_products}};
  if (!p || !p.length) { return undefined; }
  var factor = {{EA - OpenAI - minor_unit_factor}};
  var currency = {{EA - Core - salla_currency}};
  var out = [];
  for (var i = 0; i < p.length; i++) {
    var it = p[i];
    var id = it && (it.product_id || it.id);
    if (!id) { continue; }
    var item = {
      id: String(id),
      content_type: 'product',
      quantity: parseInt(it.quantity, 10) || 1
    };
    if (it.name) { item.name = String(it.name); }
    var price = Number(it.price);
    if (!isNaN(price) && price > 0) {
      item.amount = Math.round(price * factor);
      item.currency = currency;
    }
    out.push(item);
  }
  return out.length ? out : undefined;
}
```

`EA - OpenAI - user_data` (hashes only; `undefined` when none is valid):

```js
function() {
  var re = /^[a-f0-9]{64}$/;
  var out = {};
  var email = {{EA - Core - salla_customer_email_hashed}};
  var phone = {{EA - Core - salla_customer_phone_hashed}};
  if (typeof email === 'string' && re.test(email)) { out.email_sha256 = email; }
  if (typeof phone === 'string' && re.test(phone)) { out.phone_number_sha256 = phone; }
  return (out.email_sha256 || out.phone_number_sha256) ? out : undefined;
}
```

## 5. Tags (folder `EA - OpenAI`) — Custom HTML, `supportDocumentWrite` false

GTM variables are written bare inside `<script>` — no quotes around `{{…}}`.

### 5.1 `EA - OpenAI - init + page_viewed`

Trigger: All Pages (`2147479553`). Firing option: once per page.

```html
<script>
  (function (w, d, s, u) {
    if (w.oaiq) return;
    var q = function () { q.q.push(arguments); };
    q.q = [];
    w.oaiq = q;
    var js = d.createElement(s);
    js.async = true;
    js.src = u;
    var f = d.getElementsByTagName(s)[0];
    f.parentNode.insertBefore(js, f);
  })(window, document, "script", "https://bzrcdn.openai.com/sdk/oaiq.min.js");

  (function () {
    var config = { pixelId: {{EA - OpenAI - pixel_id}} };
    var user = {{EA - OpenAI - user_data}};
    if (user) { config.user = user; }
    window.oaiq("init", config);
    window.oaiq("measure", "page_viewed", { type: "contents" });
  })();
</script>
```

### 5.2 `EA - OpenAI - contents_viewed` and `EA - OpenAI - items_added`

Triggers: `Product Viewed` and `Product Added`. Same body; only the event name in the last call changes (`"contents_viewed"` / `"items_added"`). Once per event, setup tag = 5.1.

```html
<script>
  (function () {
    if (!window.oaiq) return;
    var data = { type: "contents" };
    var amount = {{EA - OpenAI - product_amount}};
    var contents = {{EA - OpenAI - product_contents}};
    if (amount) {
      data.amount = amount;
      data.currency = {{EA - Core - salla_currency}};
    }
    if (contents) { data.contents = contents; }
    window.oaiq("measure", "contents_viewed", data);
  })();
</script>
```

### 5.3 `EA - OpenAI - checkout_started`

Trigger: `checkout::started`. Once per event, setup tag = 5.1. No `contents`.

```html
<script>
  (function () {
    if (!window.oaiq) return;
    var data = { type: "contents" };
    var amount = {{EA - OpenAI - checkout_amount}};
    if (amount) {
      data.amount = amount;
      data.currency = {{EA - Core - salla_currency}};
    }
    window.oaiq("measure", "checkout_started", data);
  })();
</script>
```

### 5.4 `EA - OpenAI - order_created`

Trigger: `Order Completed` only. Once per event, setup tag = 5.1. Calls `init` again so the hashed email/phone are attached even if the base tag ran before the customer data arrived.

```html
<script>
  (function () {
    if (!window.oaiq) return;
    var user = {{EA - OpenAI - user_data}};
    if (user) {
      window.oaiq("init", { pixelId: {{EA - OpenAI - pixel_id}}, user: user });
    }
    var data = { type: "contents" };
    var amount = {{EA - OpenAI - order_amount}};
    var contents = {{EA - OpenAI - order_contents}};
    if (amount) {
      data.amount = amount;
      data.currency = {{EA - Core - salla_currency}};
    }
    if (contents) { data.contents = contents; }
    var orderId = {{EA - Core - salla_order_id}};
    if (orderId) {
      window.oaiq("measure", "order_created", data, { event_id: String(orderId) });
    } else {
      window.oaiq("measure", "order_created", data);
    }
  })();
</script>
```

### 5.5 GTM API shapes (for a GTM MCP server that takes raw entities)

```json
{ "variable": { "variableId": "new_2", "name": "EA - Core - salla_product_id", "type": "v", "parentFolderId": "new_1",
  "parameter": [ { "type": "integer", "key": "dataLayerVersion", "value": "2" },
                 { "type": "boolean", "key": "setDefaultValue", "value": "false" },
                 { "type": "template", "key": "name", "value": "0.id" } ] } }

{ "variable": { "name": "EA - OpenAI - pixel_id", "type": "c",
  "parameter": [ { "type": "template", "key": "value", "value": "<PIXEL ID>" } ] } }

{ "variable": { "name": "EA - OpenAI - order_amount", "type": "jsm",
  "parameter": [ { "type": "template", "key": "javascript", "value": "function() { … }" } ] } }

{ "trigger": { "name": "EA - Trigger - Order Completed", "type": "customEvent",
  "customEventFilter": [ { "type": "equals", "parameter": [
      { "type": "template", "key": "arg0", "value": "{{_event}}" },
      { "type": "template", "key": "arg1", "value": "Order Completed" } ] } ],
  "filter": [ { "type": "matchRegex", "parameter": [
      { "type": "template", "key": "arg0", "value": "{{EA - Core - salla_order_id}}" },
      { "type": "template", "key": "arg1", "value": "^\\d+$" } ] } ] } }

{ "tag": { "name": "EA - OpenAI - order_created", "type": "html", "tagFiringOption": "oncePerEvent",
  "firingTriggerId": [ "<trigger id>" ],
  "setupTag": [ { "tagName": "EA - OpenAI - init + page_viewed", "stopOnSetupFailure": true } ],
  "parameter": [ { "type": "template", "key": "html", "value": "<script>…</script>" },
                 { "type": "boolean", "key": "supportDocumentWrite", "value": "false" } ] } }
```

Base tag: `"tagFiringOption": "oncePerLoad"`, `"firingTriggerId": ["2147479553"]`.

## 6. Verification checklist

- Workspace compiles: no `compilerError` in the quick preview.
- The Pixel ID appears only in `EA - OpenAI - pixel_id`.
- Five tags, each with exactly one trigger; `order_created` only on `Order Completed`.
- The four event tags have the base tag as setup tag.
- Every `amount` comes from an `EA - OpenAI - *_amount` variable or the per-unit conversion inside a contents variable — no raw Salla value is sent as an amount.
- Every `measure` that sends `amount` also sends `currency`.
- No tag or variable reads `customer.email`, `customer.phone`, `customer.phone_with_plus`, `customer.name`, or anything under `payments::apple-pay.authorized.init`.
- No `debug: true`.
- Worked example for the samples in section 2:
  - `contents_viewed` → `{ type: "contents", amount: 29600, currency: "SAR", contents: [{ id: "100000001", content_type: "product", quantity: 1, name: "Sample product", amount: 29600, currency: "SAR" }] }`
  - `checkout_started` → `{ type: "contents", amount: 30300, currency: "SAR" }`
  - `order_created` → `{ type: "contents", amount: 20300, currency: "SAR", contents: [{ id: "100000001", content_type: "product", quantity: 1, name: "Sample product", amount: 19600, currency: "SAR" }] }` with options `{ event_id: "1000000001" }`

## 7. Not built, and why

| Salla event | Reason |
|---|---|
| `Products Searched`, `Product Added to Wishlist`, `auth::logged.in`, `Cart Viewed`, `Product Removed` | OpenAI has no standard event for search, wishlist, login, view cart or remove from cart. |
| `payments::apple-pay.start-transaction` | No standard payment-info event; `checkout_started` already covers the flow. |
| First and last name hashes | Salla's normalisation is unverified. |

If the user asks for one of these, build it as a `custom` event:

```js
window.oaiq("measure", "custom", { type: "custom" }, { custom_event_name: "search" });
```

`custom_event_name`: 1–64 characters, lowercase letters, digits, `_` or `-`, starting and ending with a letter or digit, and not a standard event name. Note that `Product Added to Wishlist` carries no product data and `Products Searched` can fire more than once per search.

**Consent.** Salla stores normally run without a consent banner, so the tags carry no consent checks. If the store has one, call `oaiq("consent", false)` before `init` and `oaiq("consent", true)` after opt-in, and say so in the report.
