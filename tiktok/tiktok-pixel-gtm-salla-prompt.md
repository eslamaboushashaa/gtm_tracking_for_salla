# TikTok Pixel via GTM — Salla Stores — Reusable Prompt

Paste everything below the line into any LLM agent that has a **GTM MCP server** connected (a browser tool is optional but enables the live-store checks).
Fill in the INPUTS in Step 0. Nothing else changes between Salla stores.
(Claude Code users: install the skill in `skill/tiktok-pixel-gtm-salla/` instead — same workflow.)

---

You are setting up the TikTok Pixel on a Salla store through Google Tag Manager. Follow the workflow below step by step, in order. "The reference" means the REFERENCE section at the end of this prompt.


## WORKFLOW

## Overview

Every Salla store pushes the same dataLayer, so this build is fixed: one Custom HTML base tag plus six event tags on TikTok's official gallery template, each fed by exactly one Salla event. Nothing about events or field mapping needs asking — only the inputs below.

The Salla dataLayer, the shared `EA - Core` foundation, every GTM payload and the post-publish checklist live in the REFERENCE section at the end of this prompt. Read it before Step 1.

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

Follow the REFERENCE → **PROCEDURE**, Steps 1–10, in order:

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

---

# REFERENCE — TikTok Pixel via GTM on Salla

Salla dataLayer shapes were captured from live Salla stores in 2026; every ID and customer value below is fictional. Template IDs (`cvt_…`) are the values seen when this was written — each build reads the real one from the installed template.

## WHAT YOU ARE BUILDING (fixed spec)

**1 base tag + 6 event tags** (+1 conditional, Step 8c). Event tags use the installed TikTok gallery template (`tiktok/gtm-template-pixel`).

| Tag name | TikTok event | Fires on (Salla dataLayer event) | Data coverage |
|---|---|---|---|
| `EA - TikTok Pixel - Base Code` | *(loads `ttq`, fires Pageview)* | All Pages (`2147479553`), **priority 100** | ✅ |
| `EA - TikTok Pixel - ViewContent` | ViewContent | `Product Viewed` | ✅ full |
| `EA - TikTok Pixel - Search` | Search | `Products Searched` | ✅ search term |
| `EA - TikTok Pixel - AddToWishlist` | AddToWishlist | `Product Added to Wishlist` (success = true) | ⚠️ event only — no product data |
| `EA - TikTok Pixel - AddToCart` | AddToCart | `Product Added` | ✅ full |
| `EA - TikTok Pixel - InitiateCheckout` | InitiateCheckout | `checkout::started` | ⚠️ value + currency only |
| `EA - TikTok Pixel - Purchase` | **Purchase** (or CompletePayment — see below) | `Order Completed` | ✅ full |
| *(conditional)* `EA - TikTok Pixel - AddToCart - Bought Together` | AddToCart | `salla-bought-together::product.added` | only if the bought-together check fails |

> ⚠️ **TikTok's template does NOT load the pixel.** Its `validate()` fails with `"ttq not found"` unless `ttq` already exists. The Custom HTML base tag is mandatory.

Fixed decisions — do not re-litigate:
- **Purchase event name.** TikTok renamed `CompletePayment` → **`Purchase`** for web on 1 May 2025; legacy `CompletePayment` is auto-converted and supported indefinitely. Read the installed template's event dropdown (Step 3): use `Purchase` if offered, otherwise `CompletePayment`. Never type a value the dropdown doesn't contain.
- **Search term is sent twice:** in the template's `query` field (what the template natively forwards) **and** as an Additional Property `search_string` (the name TikTok's current docs use). Same value, so either naming is covered.
- **Advanced Matching in PLAIN mode** (`hash: "non-hashed"`) with `customer.email` and `customer.phone_with_plus`. TikTok hashes and normalises them itself. *Plain mode is right for Salla specifically:* hashed mode is only needed when a store sends a pre-hashed `external_id` (Zid does), and Salla has no `external_id`.
- **Manual contents**: `enhance_ecomm: false`. `single_multi_product: "multiple"` on ViewContent / AddToCart / Purchase; `"empty"` on Search / AddToWishlist / InitiateCheckout.
- Purchase value = `0.prices.total` (amount paid incl. shipping). `order_id` and `event_id` = `0.order_id`.
- AddToWishlist is sent **without** product data — never borrowed from `0.id` (fact #14).

---

## EVENT COVERAGE — why exactly these

Checked against TikTok's standard events & parameters article and its May-2025 updated standard events.

| TikTok standard event | Salla source | Verdict |
|---|---|---|
| Pageview | base code `ttq.page()` | ✅ Build |
| ViewContent | `Product Viewed` | ✅ Build |
| Search | `Products Searched` | ✅ Build — `query` + `search_string` |
| AddToWishlist | `Product Added to Wishlist` | ⚠️ Build — no product data in Salla's push |
| AddToCart | `Product Added` | ✅ Build |
| InitiateCheckout | `checkout::started` | ⚠️ Build — value/currency only |
| AddPaymentInfo | `payments::apple-pay.start-transaction` | ❌ Not built — Salla only pushes it for Apple Pay's "Pay now"; card and mada payments send nothing, so the event would count a fraction of payments and mislead optimisation |
| Purchase (formerly CompletePayment) | `Order Completed` | ✅ Build |
| CompleteRegistration | — | ❌ `auth::logged.in` is a login; signup vs login can't be told apart |
| Lead / SubmitForm | — | ❌ No lead event. `gtm.formSubmit` is generic GTM form noise |
| Contact | not in dataLayer | ➖ Optional extra via a `wa.me` link-click trigger — not built by default |
| PlaceAnOrder | — | ➖ Legacy, retiring 2027. Not built (a COD split on `0.payment_method` is possible later) |
| ClickButton | — | ➖ Legacy, retiring 2027 |
| Download, Schedule, StartTrial, Subscribe, SubmitApplication, CustomizeProduct, FindLocation, ApplicationApproval | — | ❌ Not applicable |

Salla events with **no** TikTok standard event — do not build: `Product Removed`, `Cart Viewed`, `salla-search::product.clicked`, `search::open`, `impressions`.

---

## SALLA dataLayer REFERENCE

The shape every Salla store pushes (IDs and values below are fictional). Build against it.

```js
// Product Viewed — ONE product, stored under the key "0"
dataLayer.push({
  0: { id: 100000001, name: "...", category: "Sample category",
       categories: [{ id: 500001, name: "..." }], price: 296, quantity: 1,
       variant: "", brand: "", currency: "SAR", sku: "", value: 296, url: "...", image_url: "..." },
  event: "Product Viewed"
});

// Product Added — NO currency; quantity is a STRING
dataLayer.push({
  0: { id: 100000001, name: "...", category: "...", price: 296, quantity: "1",
       cart_id: "<cart_id>", cart_item_id: "...", coupon: "", sku: "", url: "..." },
  event: "Product Added"
});

// Products Searched — the search itself. `total` = number of results
dataLayer.push({ event: "Products Searched", query: "عطر", total: 6 });

// search::open — the search box was opened. NOT a search
dataLayer.push({ event: "search::open" });

// salla-search::product.clicked — a click on a search result. Product sits under 0.product
dataLayer.push({
  0: { product: { id: "100000003", name: "...", price: 17, currency: "SAR",
                  category: { name: "Sample category" }, type: "service", /* ...60+ fields */ },
       position: 0 },
  event: "salla-search::product.clicked"
});

// Product Added to Wishlist — NO product data at all
dataLayer.push({ event: "Product Added to Wishlist", status: 201, success: true });

// salla-bought-together::product.added — "frequently bought together" widget.
// Product sits under 0.product (NOT 0.id). cartItem / cartId are undefined.
dataLayer.push({
  0: { product: { id: "100000004", name: "...", price: 9, currency: "SAR",
                  category: { name: "Sample category" }, sku: "", type: "product" /* ... */ },
       position: 3, cartItem: undefined, cartId: undefined, request_id: "<uuid>" },
  event: "salla-bought-together::product.added"
});

// removedFromCart — legacy UA-style duplicate of Product Removed (note quantity: 0). Never build on it
dataLayer.push({ event: "removedFromCart",
  ecommerce: { currencyCode: "SAR", remove: { products: [{ id: 100000005, price: 16, quantity: 0 /* ... */ }] } } });

// checkout::started — NO product data at all
dataLayer.push({ event: "checkout::started", checkout_type: "mini_checkout",
  checkout_id: "...", cart_id: "...", requires_shipping: true,
  value: 303, currency: "SAR", payment_methods: ["credit_card", "mada", "apple_pay"] });

// payments::apple-pay.start-transaction — Salla's Apple Pay "Pay now" button
dataLayer.push({ event: "payments::apple-pay.start-transaction",
  amount: 296, currency: "SAR", checkout_type: "buy_now", /* merchant config + callbacks */ });

// auth::logged.in
dataLayer.push({ event: "auth::logged.in", status: 200, success: true });

// purchase — Salla's legacy UA-style push. Fires FIRST on the thank-you page.
// The ONLY push that carries the customer object.
dataLayer.push({ event: "purchase",
  ecommerce: { currencyCode: "SAR",
    purchase: { actionField: { id: 200000001, total: 203, revenue: 203 }, products: [ /* ... */ ] } },
  store: { store: "sample-store", facebook_pixel_id: null, google_analytics_id: null, snapchat_pixel_id: null },
  customer: { isGuest: false, name: "<plain full name>",
    first_name: "<sha256>", last_name: "<sha256>",
    email: "customer@example.com", phone: "9665XXXXXXXX", phone_with_plus: "+9665XXXXXXXX",
    email_hashed: "<sha256>", phone_hashed: "<sha256>", phone_with_plus_hashed: "<sha256>",
    currency: "SAR" } });

// Order Completed — fires AFTER purchase. The ONLY purchase source to build on.
dataLayer.push({
  0: { order_id: "1000000001", reference_id: "200000001", cart_id: "...",
       total: 196, subtotal: 203, revenue: 196, shipping: 7, tax: 0, discount: 0, coupon: "",
       currency: "SAR", payment_method: "credit_card", event_id: "<uuid>",
       products: [{ product_id: 100000002, sku: null, name: "...", price: 196, quantity: 1, category: "..." }],
       prices: { total: "203.00", amount: "196", shipping_cost: "7", tax: "0.00", cod_cost: "0", currency: "SAR" } },
  event: "Order Completed"
});
```

`impressions` (product list pages) is described in the store's Tag Assistant inventory as `ecommerce.impressions[]` with `id, name, category, brand, price, position` — no raw sample exists. Confirm its shape in GTM Preview before relying on it.

**Facts that drive every Salla build:**
1. **Product and order data sit under the key `"0"`.** DLV paths are `0.id`, `0.price`, `0.order_id`, `0.prices.total`.
2. **Salla never clears `"0"` between pushes.** GTM merges each push into its data model, so a field from an earlier push survives into a later one unless overwritten. Example: `0.value` from `Product Viewed` is still there when `Product Added` fires. **Always compute from fields present in the current push** — add-to-cart value is `price × quantity`, never `0.value`.
3. `Product Added` has **no currency** and sends **quantity as a string** (`"1"`). Cast with `Number()`.
4. `checkout::started` carries **no product IDs or items**.
5. `payments::apple-pay.start-transaction` fires only for Apple Pay's "Pay now" (often `checkout_type: "buy_now"`). Card and mada payments produce **no payment-info event**. That is why no skill in this repo builds a payment-info event (AddPaymentInfo / ADD_BILLING): it would count only Apple Pay buyers.
6. **Three things look like a purchase — build on exactly one:** `Order Completed`. Never also `purchase` (legacy push) or `mobile::order.placed` (fired 27× in one test session).
7. The `customer` object arrives only in the `purchase` push, which fires *before* `Order Completed` on the same page, so its fields are still in the data model when Order Completed fires. On product/cart/search pages the customer fields are normally **undefined** — expected.
8. Salla sends email/phone **plain and hashed**. Verified by hashing sample values: `email_hashed` = SHA-256 of the lowercased email; `phone_hashed` = SHA-256 of the digits with country code and no `+` (`966…`); `phone_with_plus_hashed` = SHA-256 of `+966…`. `first_name`/`last_name` are **already SHA-256** with unverified normalisation (no plain version). `customer.name` is the plain full name — **never send it anywhere.**
9. Order **value** is `0.prices.total` — a **string** (`"203.00"`). `0.total` (196) excludes shipping; `0.subtotal` (203) is the fallback. `0.order_id` is a numeric string.
10. Salla has **no `external_id`** for customers.
11. `store.facebook_pixel_id` / `snapchat_pixel_id` / `google_analytics_id` reveal **Salla's native integrations**. Non-null = already tracked natively = double counting if GTM tags are added.
12. **Privacy — mandatory:** never create variables or tags reading `payments::apple-pay.authorized.init` (shipping contact, address, Apple Pay **payment token**). Never send `customer.name`, `shippingContact`, `shippingAddress` or card data.
13. **Search:** `Products Searched` is the search (`query` = term, `total` = result count). `search::open` only opens the box; `salla-search::product.clicked` is a click on a result. Salla's search is a live search box — `Products Searched` may fire more than once per visit as the shopper refines the term. Check the firing frequency in Preview (Step 2).
14. **Wishlist:** `Product Added to Wishlist` carries **no product ID, price or name**. Do **not** fill the gap from `0.id`: on a category page the heart icon would attach whichever product was pushed last — a wrong product is worse than none.
15. **Bought-together:** `salla-bought-together::product.added` stores the product under **`0.product`**. While it is in the data model, `0.id` still holds the *main* product from the page — never read `0.id` on this event. Whether Salla *also* pushes `Product Added` for these adds is unknown; see the **Bought-together check** below.
16. **Legacy duplicates** — `detail` (= Product Viewed), `addToCart` / `cart::item.added` (= Product Added), `removedFromCart` / `cart::item.deleted` (= Product Removed). Never build on them; they double count.
17. `salla-search::product.clicked` and the bought-together event also merge `0.product` and `0.position` into the data model — you'll see those extra keys in Preview on later events. Harmless to the `0.id` variables.

---

## SALLA FOUNDATION — shared by every skill in this repo (Meta, TikTok, Snap, ChatGPT)

All Salla skills in this repo use these **exact names**. Whichever runs first creates them in the folder **`EA - Core`**; the others **reuse** them. Create only what's missing — and only the items the skill you're running uses (see the "Used by" columns).

**Reuse by path, not by name.** Before creating a variable, look for any Data Layer Variable already reading the same path (older builds named them `ES - salla_*`, others may use `DLV - …`). If one exists, use its real name everywhere this document says `EA - Core - …`. Same for triggers: reuse any Custom Event trigger whose event name matches, whatever it is called.

### Data Layer Variables (`type: "v"`, `dataLayerVersion: 2`, `setDefaultValue: false`)

| Name | Path | Used by |
|---|---|---|
| `EA - Core - salla_product_id` | `0.id` | all |
| `EA - Core - salla_product_name` | `0.name` | all |
| `EA - Core - salla_product_category` | `0.category` | Meta, TikTok, Snap |
| `EA - Core - salla_product_price` | `0.price` | all |
| `EA - Core - salla_product_quantity` | `0.quantity` | all |
| `EA - Core - salla_item_currency` | `0.currency` | all |
| `EA - Core - salla_event_currency` | `currency` | all |
| `EA - Core - salla_checkout_value` | `value` | all |
| `EA - Core - salla_order_id` | `0.order_id` | all |
| `EA - Core - salla_order_prices_total` | `0.prices.total` | all |
| `EA - Core - salla_order_subtotal` | `0.subtotal` | all |
| `EA - Core - salla_order_products` | `0.products` | all |
| `EA - Core - salla_customer_email` | `customer.email` | Meta, TikTok, Snap |
| `EA - Core - salla_customer_phone` | `customer.phone` | Meta, Snap |
| `EA - Core - salla_customer_phone_plus` | `customer.phone_with_plus` | TikTok |
| `EA - Core - salla_customer_email_hashed` | `customer.email_hashed` | ChatGPT |
| `EA - Core - salla_customer_phone_hashed` | `customer.phone_hashed` | ChatGPT |
| `EA - Core - salla_search_query` | `query` | Meta, TikTok, Snap |
| `EA - Core - salla_success` | `success` | wishlist + login trigger filters |
| `EA - Core - salla_impressions` | `ecommerce.impressions` | Snap |
| *CONDITIONAL* `EA - Core - salla_bt_product_id` | `0.product.id` | bought-together only |
| *CONDITIONAL* `EA - Core - salla_bt_product_name` | `0.product.name` | bought-together only |
| *CONDITIONAL* `EA - Core - salla_bt_product_price` | `0.product.price` | bought-together only |
| *CONDITIONAL* `EA - Core - salla_bt_product_category` | `0.product.category.name` | bought-together only |

DLV JSON shape (`«COREFOLDER»` = the ID of the `EA - Core` folder; with `bulkUpdate` use its temporary ID such as `new_1`):
```json
{"name": "EA - Core - salla_product_id", "type": "v", "parentFolderId": "«COREFOLDER»",
 "notes": "Salla product ID from Product Viewed / Product Added (key 0).",
 "parameter": [
   {"type": "integer",  "key": "dataLayerVersion", "value": "2"},
   {"type": "boolean",  "key": "setDefaultValue",  "value": "false"},
   {"type": "template", "key": "name",             "value": "0.id"}]}
```

### Constant

```json
{"name": "EA - Core - store_currency", "type": "c", "parentFolderId": "«COREFOLDER»",
 "notes": "Store currency fallback. Salla's Product Added push carries no currency. Change only if this store does not sell in SAR.",
 "parameter": [{"type": "template", "key": "value", "value": "SAR"}]}
```

### Custom JavaScript Variables (`type: "jsm"`, key `javascript`, **ES5 only**)

`EA - Core - salla_currency`
```js
function() {
  var c = {{EA - Core - salla_item_currency}} || {{EA - Core - salla_event_currency}};
  if (c && c !== 'undefined') { return String(c); }
  return {{EA - Core - store_currency}};
}
```

`EA - Core - salla_product_ids`
```js
function() {
  var id = {{EA - Core - salla_product_id}};
  return id ? [String(id)] : undefined;
}
```

`EA - Core - salla_add_to_cart_value` — price × quantity, never the stale `0.value`
```js
function() {
  var price = Number({{EA - Core - salla_product_price}});
  var qty = Number({{EA - Core - salla_product_quantity}}) || 1;
  if (isNaN(price)) { return undefined; }
  return Math.round(price * qty * 100) / 100;
}
```

`EA - Core - salla_order_value` — amount paid incl. shipping
```js
function() {
  var v = Number({{EA - Core - salla_order_prices_total}});
  if (!v || isNaN(v)) { v = Number({{EA - Core - salla_order_subtotal}}); }
  if (!v || isNaN(v)) { return undefined; }
  return Math.round(v * 100) / 100;
}
```

`EA - Core - salla_order_item_ids`
```js
function() {
  var p = {{EA - Core - salla_order_products}};
  if (!p || !p.length) { return undefined; }
  var ids = [];
  for (var i = 0; i < p.length; i++) {
    var id = p[i] && (p[i].product_id || p[i].id);
    if (id) { ids.push(String(id)); }
  }
  return ids.length ? ids : undefined;
}
```

`EA - Core - salla_order_num_items`
```js
function() {
  var p = {{EA - Core - salla_order_products}};
  if (!p || !p.length) { return undefined; }
  var n = 0;
  for (var i = 0; i < p.length; i++) { n += Number(p[i] && p[i].quantity) || 1; }
  return n;
}
```

*CONDITIONAL* `EA - Core - salla_bt_product_ids`
```js
function() {
  var id = {{EA - Core - salla_bt_product_id}};
  return id ? [String(id)] : undefined;
}
```

### Triggers (Custom Event, **exact and case-sensitive** event names, folder `EA - Core`)

Name a new trigger `EA - Trigger - <dataLayer event name>`. Reuse an existing one with the same event name instead of creating it.

| Trigger name | Event name | Extra condition | Used by |
|---|---|---|---|
| `EA - Trigger - Product Viewed` | `Product Viewed` | — | all |
| `EA - Trigger - Product Added` | `Product Added` | — | all |
| `EA - Trigger - checkout::started` | `checkout::started` | — | all |
| `EA - Trigger - Order Completed` | `Order Completed` | `{{EA - Core - salla_order_id}}` matches RegEx `^\d+$` | all |
| `EA - Trigger - Products Searched` | `Products Searched` | — | Meta, TikTok, Snap |
| `EA - Trigger - Product Added to Wishlist` | `Product Added to Wishlist` | `{{EA - Core - salla_success}}` equals `true` | Meta, TikTok, Snap |
| `EA - Trigger - auth::logged.in` | `auth::logged.in` | `{{EA - Core - salla_success}}` equals `true` | Snap |
| `EA - Trigger - impressions` | `impressions` | — | Snap |
| *CONDITIONAL* `EA - Trigger - salla-bought-together::product.added` | `salla-bought-together::product.added` | — | only if the bought-together check fails |

Two filter patterns:

```json
{"name": "EA - Trigger - Order Completed", "type": "customEvent", "parentFolderId": "«COREFOLDER»",
 "notes": "The ONLY purchase trigger. Never add triggers on 'purchase' or 'mobile::order.placed' - they duplicate the same order. Digits-only regex because an undefined variable becomes the string 'undefined', which would pass a '.+' check.",
 "customEventFilter": [{"type": "equals", "parameter": [
   {"type": "template", "key": "arg0", "value": "{{_event}}"},
   {"type": "template", "key": "arg1", "value": "Order Completed"}]}],
 "filter": [{"type": "matchRegex", "parameter": [
   {"type": "template", "key": "arg0", "value": "{{EA - Core - salla_order_id}}"},
   {"type": "template", "key": "arg1", "value": "^\\d+$"}]}]}
```

```json
{"name": "EA - Trigger - Product Added to Wishlist", "type": "customEvent", "parentFolderId": "«COREFOLDER»",
 "notes": "Salla wishlist add. Filtered to success=true so failed attempts don't count. The push carries no product data.",
 "customEventFilter": [{"type": "equals", "parameter": [
   {"type": "template", "key": "arg0", "value": "{{_event}}"},
   {"type": "template", "key": "arg1", "value": "Product Added to Wishlist"}]}],
 "filter": [{"type": "equals", "parameter": [
   {"type": "template", "key": "arg0", "value": "{{EA - Core - salla_success}}"},
   {"type": "template", "key": "arg1", "value": "true"}]}]}
```
`auth::logged.in` uses the same `success = true` filter.

### Bought-together check (run once per store, in Step 2)

Salla's "frequently bought together" widget pushes `salla-bought-together::product.added`. It is not known whether Salla *also* pushes `Product Added` for that same add — it may differ by theme.

**Test:** open a product page that shows the widget, add a product from the widget, then read the GTM dataLayer array.
- **`Product Added` follows** with the widget product's ID → **build nothing extra.** The normal AddToCart already catches it; adding the widget event would double count.
- **No `Product Added` follows** → create the CONDITIONAL variables, CJS and trigger above, plus the platform's conditional tag (Step 8c).
- **The store has no widget** → skip.

Empty the test cart afterwards. If you can't run the browser test, say so in the report and leave the conditional items unbuilt.

---

## HARD RULES

1. **Never publish. Never create a container version.** Build, verify, report, stop.
2. **Never create a trigger or variable that already exists.** List first, reuse.
3. **Never rename a pre-existing tag, trigger or variable.**
4. **Naming:** tags `EA - TikTok Pixel - <TikTokEventName>`; platform variables `EA - TikTok Pixel - <snake_case>`; shared Salla variables `EA - Core - salla_<snake_case>`; new triggers `EA - Trigger - <dataLayer event name>`; folders `EA - TikTok Pixel` and `EA - Core`.
5. **Pixel ID lives only in the constant `EA - TikTok Pixel - pixel_id`** — including inside the base Custom HTML.
6. **Set `parentFolderId` at creation time** on every entity. TikTok items → `EA - TikTok Pixel`; Salla foundation → `EA - Core`.
7. Every entity gets a `notes` value.
8. Respect fact #12 (privacy). No exceptions.

---

## PROCEDURE

### Step 1 — Locate the container
`gtm_account list` → `gtm_container list` → `gtm_workspace list`. People give approximate names ("acme website under agency new" → account `Agency - new`, container `Acme - website`): one clear match → state it and continue; none or several → ask. Confirm the container's `publicId` (`GTM-XXXX`).

Create a dedicated workspace `EA - TikTok Pixel` and build there — never in the Default Workspace. If a write returns **"Workspace is already submitted"**, the client published meanwhile: re-run `gtm_workspace list` and continue in the new workspace.

### Step 2 — Live-site checks
In the browser tool, on the live store (skip this step if no browser tool is available, and list every unchecked item in the report):
1. **dataLayer name** — read `gtm.js?id=…&l=<name>`; confirm `Product Viewed` lands in the array GTM listens to. Mismatch → stop and tell the user.
2. **Existing TikTok install** — does `window.ttq` exist before GTM fires, or is there an `analytics.tiktok.com` script not loaded by GTM (Salla app / theme)? Yes → stop and warn the user: double counting.
3. **Search frequency** — type a two-word search slowly and count `Products Searched` pushes. Per-keystroke firing → report it.
4. **Bought-together check** (see the foundation section).

### Step 3 — Template tag type and Purchase event name
`gtm_template list` (huge — parse the saved file with a script). Find **TikTok Pixel** (owner `tiktok`, repo `gtm-template-pixel`). From `___INFO___` read `"id"` — currently **`cvt_MRQN8`**. From `___TEMPLATE_PARAMETERS___` read the `event` field's `selectItems`: **contains `Purchase` → use `Purchase`; otherwise `CompletePayment`.** Report which. Template missing → stop and tell the user.

### Step 4 — Audit
`gtm_trigger list`, `gtm_variable list`, `gtm_tag list` — **every page** (20 per page; read `totalPages`).

### Step 5 — Folders
`EA - TikTok Pixel`; `EA - Core` if missing.

### Step 6 — Salla foundation
Create what TikTok uses and is missing: DLVs marked *all* / *TikTok*, `EA - Core - salla_success`, the constant, the non-conditional Custom JS, and the triggers `Product Viewed`, `Product Added`, `checkout::started`, `Order Completed`, `Products Searched`, `Product Added to Wishlist`. Conditional items only per the bought-together check.

### Step 7 — TikTok variables (folder `EA - TikTok Pixel`)

`EA - TikTok Pixel - pixel_id` — Constant = pixel ID.

`EA - TikTok Pixel - product_contents`
```js
function() {
  var id = {{EA - Core - salla_product_id}};
  if (!id) { return undefined; }
  var c = {
    content_id: String(id),
    content_type: 'product',
    price: Number({{EA - Core - salla_product_price}}) || 0,
    quantity: Number({{EA - Core - salla_product_quantity}}) || 1
  };
  var n = {{EA - Core - salla_product_name}};
  if (n) { c.content_name = n; }
  var cat = {{EA - Core - salla_product_category}};
  if (cat) { c.content_category = cat; }
  return [c];
}
```

`EA - TikTok Pixel - order_contents`
```js
function() {
  var p = {{EA - Core - salla_order_products}};
  if (!p || !p.length) { return undefined; }
  var out = [];
  for (var i = 0; i < p.length; i++) {
    var it = p[i];
    var id = it && (it.product_id || it.id);
    if (!id) { continue; }
    var c = { content_id: String(id), content_type: 'product',
              price: Number(it.price) || 0, quantity: Number(it.quantity) || 1 };
    if (it.name) { c.content_name = it.name; }
    if (it.category) { c.content_category = it.category; }
    out.push(c);
  }
  return out.length ? out : undefined;
}
```

Both return `undefined` rather than `[]` when empty — an empty array tells TikTok "the cart is empty".

### Step 8 — Tags

In the tables below, *Trigger* names the Salla dataLayer event: use the trigger found or created for it in Step 4/6 (reused, or `EA - Trigger - <event>`), referenced by its trigger ID. Create everything in Steps 5–8 with one `gtm_workspace` `bulkUpdate` where the server supports it — temporary IDs (`new_1`, `new_2`, …) let new tags reference new triggers and folders in the same call.

**8a. Base tag** — `type: "html"`, `priority: {"type": "integer", "value": "100"}`, trigger `2147479553`, `supportDocumentWrite: false`, folder TikTok:

```html
<script>
!function (w, d, t) {
  w.TiktokAnalyticsObject=t;var ttq=w[t]=w[t]||[];ttq.methods=["page","track","identify","instances","debug","on","off","once","ready","alias","group","enableCookie","disableCookie","holdConsent","revokeConsent","grantConsent"],ttq.setAndDefer=function(t,e){t[e]=function(){t.push([e].concat(Array.prototype.slice.call(arguments,0)))}};for(var i=0;i<ttq.methods.length;i++)ttq.setAndDefer(ttq,ttq.methods[i]);ttq.instance=function(t){for(var e=ttq._i[t]||[],n=0;n<ttq.methods.length;n++)ttq.setAndDefer(e,ttq.methods[n]);return e},ttq.load=function(e,n){var r="https://analytics.tiktok.com/i18n/pixel/events.js",o=n&&n.partner;ttq._i=ttq._i||{},ttq._i[e]=[],ttq._i[e]._u=r,ttq._t=ttq._t||{},ttq._t[e]=+new Date,ttq._o=ttq._o||{},ttq._o[e]=n||{},n=document.createElement("script"),n.type="text/javascript",n.async=!0,n.src=r+"?sdkid="+e+"&lib="+t,e=document.getElementsByTagName("script")[0],e.parentNode.insertBefore(n,e)};

  ttq.load('{{EA - TikTok Pixel - pixel_id}}');
  ttq.page();
}(window, document, 'ttq');
</script>
```
Tell the user to diff this against TikTok Events Manager → Install Pixel → Install Code Manually; it's the standard published snippet, not pulled from the client's account.

**8b. Event tags** — `type` from Step 3, folder TikTok. Base block:

```json
{"type": "template", "key": "pixel_code",           "value": "{{EA - TikTok Pixel - pixel_id}}"},
{"type": "template", "key": "event",                "value": "<TikTokEvent>"},
{"type": "template", "key": "hash",                 "value": "non-hashed"},
{"type": "template", "key": "email",                "value": "{{EA - Core - salla_customer_email}}"},
{"type": "template", "key": "phone",                "value": "{{EA - Core - salla_customer_phone_plus}}"},
{"type": "boolean",  "key": "enhance_ecomm",        "value": "false"},
{"type": "template", "key": "single_multi_product", "value": "<multiple | empty>"},
{"type": "template", "key": "value",                "value": "<per tag>"},
{"type": "template", "key": "currency",             "value": "{{EA - Core - salla_currency}}"}
```
On **Search** and **AddToWishlist** omit the `value` and `currency` rows — there is no monetary value to send.

| Tag | `event` | Trigger | `single_multi_product` | `contents` | `value` | Extra |
|---|---|---|---|---|---|---|
| ViewContent | `ViewContent` | `Product Viewed` | `multiple` | `{{EA - TikTok Pixel - product_contents}}` | `{{EA - Core - salla_product_price}}` | — |
| Search | `Search` | `Products Searched` | `empty` | *(omit)* | *(omit)* | `query` = `{{EA - Core - salla_search_query}}` + Additional Property `search_string` (below) |
| AddToWishlist | `AddToWishlist` | `Product Added to Wishlist` | `empty` | *(omit)* | *(omit)* | — |
| AddToCart | `AddToCart` | `Product Added` | `multiple` | `{{EA - TikTok Pixel - product_contents}}` | `{{EA - Core - salla_add_to_cart_value}}` | — |
| InitiateCheckout | `InitiateCheckout` | `checkout::started` | `empty` | *(omit)* | `{{EA - Core - salla_checkout_value}}` | — |
| Purchase | from Step 3 | `Order Completed` | `multiple` | `{{EA - TikTok Pixel - order_contents}}` | `{{EA - Core - salla_order_value}}` | `order_id` = `{{EA - Core - salla_order_id}}`, `event_id` = `{{EA - Core - salla_order_id}}` |

Search's extra parameters:
```json
{"type": "template", "key": "query", "value": "{{EA - Core - salla_search_query}}"},
{"type": "list", "key": "custom_properties", "list": [
  {"type": "map", "map": [
    {"type": "template", "key": "key",   "value": "search_string"},
    {"type": "template", "key": "value", "value": "{{EA - Core - salla_search_query}}"}]}]}
```
The template forwards the `query` field as the `query` parameter, and every Additional Property row verbatim — so TikTok receives both `query` and `search_string`.

`event_id` is TikTok's dedup key against a future Events API integration; `order_id` keeps thank-you page reloads from inflating orders.

### Step 8c — CONDITIONAL: bought-together AddToCart
Only if the bought-together check showed **no** `Product Added`. Create `EA - TikTok Pixel - bt_contents` (folder TikTok):
```js
function() {
  var id = {{EA - Core - salla_bt_product_id}};
  if (!id) { return undefined; }
  var c = { content_id: String(id), content_type: 'product', quantity: 1,
            price: Number({{EA - Core - salla_bt_product_price}}) || 0 };
  var n = {{EA - Core - salla_bt_product_name}};
  if (n) { c.content_name = n; }
  var cat = {{EA - Core - salla_bt_product_category}};
  if (cat) { c.content_category = cat; }
  return [c];
}
```
Tag `EA - TikTok Pixel - AddToCart - Bought Together`: event `AddToCart`, trigger `salla-bought-together::product.added`, `single_multi_product` `multiple`, contents `{{EA - TikTok Pixel - bt_contents}}`, value `{{EA - Core - salla_bt_product_price}}`. Quantity assumed 1 — the widget push carries no added-quantity field.

### Step 9 — Verify
`gtm_folder action=entities` on both folders; 7 tags (8 with the conditional); every variable/trigger foldered; every `{{...}}` matches an existing variable. Paginate. Then run `gtm_workspace` `quickPreview` and search the (large) result for `compilerError`.

### Step 10 — Report and stop
IDs, template ID, which Purchase name you used and why, created vs reused, live-site check results (dataLayer name, existing install, search frequency, bought-together outcome), the partial events, the optional Contact extra, that the base snippet needs diffing, and whether tags were left paused for a missing pixel ID. **Do not publish.**

---

## CRITICAL GOTCHAS

1. **The template does not load the pixel.** Base Custom HTML at priority 100 is mandatory.
2. **Don't use the GA4 auto-ecommerce toggle.** `enhance_ecomm: true` reads `ecommerce.items`. Salla doesn't use GA4 ecommerce — product data sits under `"0"` — so the toggle would silently send empty events. It also disables the `query` and `single_multi_product` fields.
3. **Stale `"0"` data.** Compute values from fields in the current push (fact #2). On the bought-together event `0.id` is the main product, not the added one.
4. **One source per action** — `Order Completed` for purchase; never the legacy `addToCart` / `removedFromCart` / `detail` duplicates.
5. **Plain vs hashed is a per-store decision, not a house style.** Plain here because Salla has no pre-hashed `external_id`.
6. **Placeholder pixel IDs go live if published.** Pause tags until the real ID exists.
7. **Custom-named dataLayer** — check `l=` in Step 2.
8. **`update` replaces the whole entity** — send complete object + current `fingerprint`.
9. **Rate limit** — `429 Quota exceeded` is transient.
10. **Template list / workspace status overflow the token limit** — parse the saved file with a script.

---

## POST-PUBLISH CHECKLIST (for the person who publishes)

- GTM Preview: base tag fires before event tags; each event fires once.
- TikTok Pixel Helper: `contents` is a real array on product pages and the thank-you page.
- Search a word: Search carries both `query` and `search_string`, Arabic text intact.
- Heart a product on a category page: AddToWishlist fires with no product attached.
- Events Manager → Test Events: Purchase value = amount paid incl. shipping; `order_id` populated.
- Events Manager → event detail: email/phone show as matched on Purchase.
- No `[TikTok Pixel] Invalid pixel ID` in the console.
- Salla dashboard: no separate TikTok integration also enabled.

---

## Sources
- TikTok — Standard events & parameters — https://ads.tiktok.com/resources/help/article/standard-events-parameters?lang=en
- TikTok — How to adopt TikTok's updated standard events (CompletePayment → Purchase) — https://ads.tiktok.com/resources/help/article/how-to-adopt-tiktoks-updated-standard-events
- TikTok official GTM template — https://github.com/tiktok/gtm-template-pixel
