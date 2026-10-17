# Meta (Facebook) Pixel via GTM — Salla Stores — Reusable Prompt

Paste everything below the line into any LLM agent that has a **GTM MCP server** connected (a browser tool is optional but enables the live-store checks).
Fill in the INPUTS in Step 0. Nothing else changes between Salla stores.
(Claude Code users: install the skill in `skill/meta-pixel-gtm-salla/` instead — same workflow.)

---

You are setting up the Meta (Facebook) Pixel on a Salla store through Google Tag Manager. Follow the workflow below step by step, in order. "The reference" means the REFERENCE section at the end of this prompt.


## WORKFLOW

## Overview

Every Salla store pushes the same dataLayer, so this build is fixed: seven Meta Pixel tags on the facebook "Meta Pixel" gallery template, each fed by exactly one Salla event. Nothing about events or field mapping needs asking — only the inputs below.

The Salla dataLayer, the shared `EA - Core` foundation, every GTM payload and the post-publish checklist live in the REFERENCE section at the end of this prompt. Read it before Step 1.

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
| `EA - Meta Pixel - Purchase` | Purchase | `Order Completed` | full, `eventId` = order ID |

Plus, only if the store needs it: `EA - Meta Pixel - AddToCart - Bought Together` (reference: bought-together check).

Fixed decisions — do not re-litigate:
- `content_ids` / `contents[].id` = Salla's numeric product ID (`0.id`, `product_id` on orders). Flag that it must match the catalog's retailer IDs.
- Purchase value = `0.prices.total`, the amount paid including shipping. Never `0.total` (products only).
- Advanced Matching sends the **plain** `customer.email` and `customer.phone`; `fbq` hashes them itself.
- `optInMetaCAPI` = `false` and `consent` = `true` on every tag.
- AddToWishlist carries no product data. Never fill it from `0.id`.
- **No payment-info event.** Salla pushes `payments::apple-pay.start-transaction` only for Apple Pay's "Pay now"; card and mada payments send nothing. AddPaymentInfo built on it would count a fraction of payments, so it is deliberately not built.

## Workflow

Follow the REFERENCE → **PROCEDURE**, Steps 1–10, in order:

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

---

# REFERENCE — Meta Pixel via GTM on Salla

Salla dataLayer shapes were captured from live Salla stores in 2026; every ID and customer value below is fictional. Template IDs (`cvt_…`) are the values seen when this was written — each build reads the real one from the installed template.

## WHAT YOU ARE BUILDING (fixed spec)

**7 tags** on the installed facebook "Meta Pixel" gallery template (+1 conditional, see Step 8c):

| Tag name | Meta event | Fires on (Salla dataLayer event) | Data coverage |
|---|---|---|---|
| `EA - Meta Pixel - PageView` | PageView | All Pages (built-in `2147479553`) | ✅ |
| `EA - Meta Pixel - ViewContent` | ViewContent | `Product Viewed` | ✅ full |
| `EA - Meta Pixel - Search` | Search | `Products Searched` | ✅ search term (no result IDs) |
| `EA - Meta Pixel - AddToWishlist` | AddToWishlist | `Product Added to Wishlist` (success = true) | ⚠️ event only — **Salla sends no product data** |
| `EA - Meta Pixel - AddToCart` | AddToCart | `Product Added` | ✅ full |
| `EA - Meta Pixel - InitiateCheckout` | InitiateCheckout | `checkout::started` | ⚠️ value + currency only — no product IDs in this push |
| `EA - Meta Pixel - Purchase` | Purchase | `Order Completed` | ✅ full |
| *(conditional)* `EA - Meta Pixel - AddToCart - Bought Together` | AddToCart | `salla-bought-together::product.added` | only if the bought-together check fails |

Fixed decisions — do not re-litigate:
- `content_ids` / `contents[].id` = Salla's numeric **product ID** (`0.id` on product events, `product_id` on orders). Flag that it must match the Meta catalog's retailer IDs.
- Purchase **value = `0.prices.total`** — the amount the customer paid, incl. shipping (203 in the sample, not the 196 in `0.total`).
- Advanced Matching uses the **plain** `customer.email` and `customer.phone`; Meta hashes them itself.
- `optInMetaCAPI` = `false` on every tag. `consent` = `true`.
- Purchase `eventId` = `0.order_id` — stable across thank-you page reloads and identical to the order ID every other platform gets.
- AddToWishlist is sent **without** product data. Never borrow `0.id` from the data model for it (fact #14).

---

## EVENT COVERAGE — why exactly these

Checked against Meta's official standard events reference. For each standard event: *does Salla's dataLayer carry a real user action for it?*

| Meta standard event | Salla source | Verdict |
|---|---|---|
| PageView | All Pages | ✅ Build |
| ViewContent | `Product Viewed` | ✅ Build — id, name, category, price, currency |
| Search | `Products Searched` | ✅ Build — `search_string` = `query`. Salla doesn't push the result product IDs, so Advantage+ catalog ads can't use Search |
| AddToWishlist | `Product Added to Wishlist` | ⚠️ Build — the push is only `{status: 201, success: true}`; all Meta properties are optional for this event, so it is valid without them |
| AddToCart | `Product Added` | ✅ Build — no currency in the push (fallback handles it), quantity is a string |
| InitiateCheckout | `checkout::started` | ⚠️ Build — value/currency only |
| AddPaymentInfo | `payments::apple-pay.start-transaction` | ❌ Not built — Salla only pushes it for Apple Pay's "Pay now"; card and mada payments send nothing, so the event would count a fraction of payments and mislead optimisation |
| Purchase | `Order Completed` | ✅ Build |
| CompleteRegistration | — | ❌ `auth::logged.in` is a **login**; Salla's OTP flow doesn't distinguish new signups |
| Lead | — | ❌ No lead-capture event |
| Contact | not in dataLayer | ➖ Optional extra: Just Links trigger on `wa.me` clicks. Not built by default — mention it |
| CustomizeProduct, Donate, FindLocation, Schedule, StartTrial, SubmitApplication, Subscribe | — | ❌ Not applicable to a Salla store |

Salla events with **no** Meta standard event — do not build: `Product Removed`, `Cart Viewed`, `salla-search::product.clicked` (click on a search result), `search::open` (search box opened), `impressions`.

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
4. **Naming:** tags `EA - Meta Pixel - <MetaEventName>`; platform variables `EA - Meta Pixel - <snake_case>`; shared Salla variables `EA - Core - salla_<snake_case>`; new triggers `EA - Trigger - <dataLayer event name>`; folders `EA - Meta Pixel` and `EA - Core`.
5. **Pixel ID lives only in the constant `EA - Meta Pixel - pixel_id`.**
6. **Set `parentFolderId` at creation time** on every tag, variable and trigger. Meta-specific → `EA - Meta Pixel`; Salla foundation → `EA - Core`.
7. Every entity gets a `notes` value.
8. Respect fact #12 (privacy). No exceptions.

---

## PROCEDURE

### Step 1 — Locate the container
`gtm_account list` → `gtm_container list` → `gtm_workspace list`. People give approximate names ("acme website under agency new" → account `Agency - new`, container `Acme - website`): one clear match → state it and continue; none or several → ask. Confirm the container's `publicId` (`GTM-XXXX`).

Create a dedicated workspace `EA - Meta Pixel` and build there — never in the Default Workspace. If a write returns **"Workspace is already submitted"**, the client published meanwhile: re-run `gtm_workspace list` and continue in the new workspace.

### Step 2 — Live-site checks
In the browser tool, on the live store (skip this step if no browser tool is available, and list every unchecked item in the report):
1. **dataLayer name.** Read the GTM script `src`: `gtm.js?id=GTM-XXXX&l=<name>`. If `l=` is present, GTM listens to `window[<name>]`. Confirm Salla's `Product Viewed` lands in that same array. Different array → **stop and tell the user** — no trigger would fire.
2. **Native Salla pixel.** Read `store.facebook_pixel_id` (thank-you page sample, or ask the user). **Not null → stop and warn the user**: Salla's own Meta integration is on and GTM tags would double-count.
3. **Theme-injected pixel.** Does `fbq` exist before GTM fires?
4. **Search frequency.** Type a two-word search slowly and count `Products Searched` pushes. If it fires on every keystroke, report it — it inflates Search counts.
5. **Bought-together check** (see the foundation section).

### Step 3 — Template tag type
`gtm_template list` (≈100k+ chars, lands in a file — **parse with a script, never read into context**). Find **Meta Pixel** (owner `facebook`), read `___INFO___` → `"id"`. Currently **`cvt_5RM3Q`**. Never guess `cvt_<containerId>_<templateId>`. Not installed → stop and tell the user.

### Step 4 — Audit
`gtm_trigger list`, `gtm_variable list`, `gtm_tag list` — **paginate**: 20 per page; read `totalPages` and fetch every page.

### Step 5 — Folders
`EA - Meta Pixel`; `EA - Core` if missing.

### Step 6 — Salla foundation
Create what Meta uses from the foundation and is missing: all DLVs marked *all* / *Meta*, `EA - Core - salla_success`, the constant, the non-conditional Custom JS, and the triggers `Product Viewed`, `Product Added`, `checkout::started`, `Order Completed`, `Products Searched`, `Product Added to Wishlist`. Conditional items only per the bought-together check.

### Step 7 — Meta variables (folder `EA - Meta Pixel`)

`EA - Meta Pixel - pixel_id` — Constant = pixel ID.

`EA - Meta Pixel - product_contents`
```js
function() {
  var id = {{EA - Core - salla_product_id}};
  if (!id) { return undefined; }
  return [{
    id: String(id),
    quantity: Number({{EA - Core - salla_product_quantity}}) || 1,
    item_price: Number({{EA - Core - salla_product_price}}) || 0
  }];
}
```

`EA - Meta Pixel - order_contents`
```js
function() {
  var p = {{EA - Core - salla_order_products}};
  if (!p || !p.length) { return undefined; }
  var out = [];
  for (var i = 0; i < p.length; i++) {
    var it = p[i];
    var id = it && (it.product_id || it.id);
    if (!id) { continue; }
    out.push({ id: String(id), quantity: Number(it.quantity) || 1, item_price: Number(it.price) || 0 });
  }
  return out.length ? out : undefined;
}
```

### Step 8 — Tags

In the tables below, *Trigger* names the Salla dataLayer event: use the trigger found or created for it in Step 4/6 (reused, or `EA - Trigger - <event>`), referenced by its trigger ID. Create everything in Steps 5–8 with one `gtm_workspace` `bulkUpdate` where the server supports it — temporary IDs (`new_1`, `new_2`, …) let new tags reference new triggers and folders in the same call.

Every tag: `type` = template ID from Step 3, `parentFolderId` = Meta folder. Base parameter block:

```json
{"type": "template", "key": "pixelId",           "value": "{{EA - Meta Pixel - pixel_id}}"},
{"type": "boolean",  "key": "enhancedEcommerce", "value": "false"},
{"type": "boolean",  "key": "useGA4Ecommerce",   "value": "false"},
{"type": "template", "key": "eventName",         "value": "standard"},
{"type": "template", "key": "standardEventName", "value": "<MetaEventName>"},
{"type": "template", "key": "consent",           "value": "true"},
{"type": "boolean",  "key": "advancedMatching",  "value": "true"},
{"type": "list", "key": "advancedMatchingList", "list": [
  {"type": "map", "map": [
    {"type": "template", "key": "name",  "value": "em"},
    {"type": "template", "key": "value", "value": "{{EA - Core - salla_customer_email}}"}]},
  {"type": "map", "map": [
    {"type": "template", "key": "name",  "value": "ph"},
    {"type": "template", "key": "value", "value": "{{EA - Core - salla_customer_phone}}"}]}
]},
{"type": "template", "key": "objectPropertiesFromVariable", "value": "false"},
{"type": "list", "key": "objectPropertyList", "list": [ /* per tag */ ]},
{"type": "boolean", "key": "optInMetaCAPI",     "value": "false"},
{"type": "boolean", "key": "dpoLDU",            "value": "false"},
{"type": "boolean", "key": "disableAutoConfig", "value": "false"},
{"type": "boolean", "key": "disablePushState",  "value": "false"}
```

Each `objectPropertyList` row:
```json
{"type": "map", "map": [
  {"type": "template", "key": "name",  "value": "content_ids"},
  {"type": "template", "key": "value", "value": "{{EA - Core - salla_product_ids}}"}]}
```

| Tag | `standardEventName` | Trigger | Object properties |
|---|---|---|---|
| PageView | `PageView` | `2147479553` | *(omit `objectPropertyList`)* |
| ViewContent | `ViewContent` | `Product Viewed` | content_type=`product`, content_ids=`{{EA - Core - salla_product_ids}}`, contents=`{{EA - Meta Pixel - product_contents}}`, content_name=`{{EA - Core - salla_product_name}}`, content_category=`{{EA - Core - salla_product_category}}`, currency=`{{EA - Core - salla_currency}}`, value=`{{EA - Core - salla_product_price}}` |
| Search | `Search` | `Products Searched` | search_string=`{{EA - Core - salla_search_query}}` |
| AddToWishlist | `AddToWishlist` | `Product Added to Wishlist` | *(omit `objectPropertyList` — no product data exists)* |
| AddToCart | `AddToCart` | `Product Added` | content_type=`product`, content_ids=`{{EA - Core - salla_product_ids}}`, contents=`{{EA - Meta Pixel - product_contents}}`, content_name=`{{EA - Core - salla_product_name}}`, currency=`{{EA - Core - salla_currency}}`, value=`{{EA - Core - salla_add_to_cart_value}}` |
| InitiateCheckout | `InitiateCheckout` | `checkout::started` | currency=`{{EA - Core - salla_currency}}`, value=`{{EA - Core - salla_checkout_value}}` |
| Purchase | `Purchase` | `Order Completed` | content_type=`product`, content_ids=`{{EA - Core - salla_order_item_ids}}`, contents=`{{EA - Meta Pixel - order_contents}}`, num_items=`{{EA - Core - salla_order_num_items}}`, currency=`{{EA - Core - salla_currency}}`, value=`{{EA - Core - salla_order_value}}` |

Purchase only — extra top-level parameter:
```json
{"type": "template", "key": "eventId", "value": "{{EA - Core - salla_order_id}}"}
```

Notes to put on the partial tags:
- Search: *"search_string only. Salla doesn't push the result product IDs, so Advantage+ catalog ads can't use this event."*
- AddToWishlist: *"Salla's wishlist push has no product data. Deliberately NOT enriched from 0.id - on category pages that would attach the wrong product."*
- InitiateCheckout: *"checkout::started carries no product IDs; content_ids/contents deliberately absent."*

### Step 8c — CONDITIONAL: bought-together AddToCart
Only if the bought-together check showed **no** `Product Added`. Create `EA - Meta Pixel - bt_contents` (folder Meta):
```js
function() {
  var id = {{EA - Core - salla_bt_product_id}};
  if (!id) { return undefined; }
  return [{ id: String(id), quantity: 1, item_price: Number({{EA - Core - salla_bt_product_price}}) || 0 }];
}
```
Tag `EA - Meta Pixel - AddToCart - Bought Together`, `standardEventName` `AddToCart`, trigger `salla-bought-together::product.added`, properties: content_type=`product`, content_ids=`{{EA - Core - salla_bt_product_ids}}`, contents=`{{EA - Meta Pixel - bt_contents}}`, content_name=`{{EA - Core - salla_bt_product_name}}`, currency=`{{EA - Core - salla_currency}}`, value=`{{EA - Core - salla_bt_product_price}}`. Quantity is assumed 1 — the widget push has no added-quantity field (`0.product.quantity` is stock, not cart quantity).

### Step 9 — Verify
`gtm_folder action=entities` on both folders. Confirm 7 tags (8 with the conditional) and every variable/trigger is foldered. Check **every `{{...}}` string in every tag matches a variable that exists** (GTM resolves by name and fails silently). Paginate. Then run `gtm_workspace` `quickPreview` and search the (large) result for `compilerError`.

### Step 10 — Report and stop
State: IDs used, template ID, created vs reused, live-site check results (dataLayer name, native pixel, search frequency, bought-together outcome), the partial events and why, the optional `Contact` (WhatsApp click) extra, and that `content_ids` must be verified against the catalog. **Do not publish.**

---

## CRITICAL GOTCHAS

1. **Stale `"0"` data (fact #2).** Values not in the current push survive from the previous one. Compute; don't trust leftovers. On the bought-together event, `0.id` is the wrong product.
2. **One source per action.** `Order Completed` for purchase; `Product Viewed` / `Product Added` — never the legacy `detail` / `addToCart` / `removedFromCart` duplicates.
3. **Custom-named dataLayer.** Check the `l=` parameter in Step 2. A mismatch means zero events and no error anywhere.
4. **Native Salla pixel (fact #11).** Non-null `store.facebook_pixel_id` = double counting.
5. **"Duplicate Pixel ID" console warning is normal.** The Meta template re-runs `fbq('init')` on each tag to refresh advanced matching. Confirm with `fbq.getState().pixels` — one entry means one pixel.
6. **`optInMetaCAPI` defaults to `true`** — it authorises Meta to set up a CAPI integration on the client's behalf. Always `false`.
7. **`update` replaces the whole entity** — send the complete object plus current `fingerprint`.
8. **GTM resolves variables by name.** Renaming doesn't update `{{...}}` references.
9. **Rate limit:** ~5+ writes/minute returns `429 Quota exceeded`. Transient — wait and retry.
10. **Browser verification:** tracking requests are often POST/`fetch`; Resource Timing doesn't show their bodies. Use the Meta Pixel Helper or hook `fetch`/`sendBeacon`.

---

## POST-PUBLISH CHECKLIST (for the person who publishes)

- GTM Preview: each event fires once; `content_ids` is a real array on product pages and the thank-you page.
- Add a product with quantity 3: AddToCart value = 3 × price.
- Search a word: one Search event with the right `search_string` (Arabic text intact).
- Heart a product on a category page: AddToWishlist fires with **no** product attached.
- Meta Pixel Helper: no warnings beyond the expected duplicate-init notice.
- Events Manager → Test Events: Purchase value = amount paid incl. shipping; currency SAR.
- Reload the thank-you page: Meta should dedupe on `eventId` = order ID.
- Events Manager → Purchase → Advanced Matching shows email/phone **matched**.
- Commerce Manager → Catalog: one `content_ids` value resolves to a product.
- Salla dashboard → Integrations: Meta Pixel field empty.

---

## Sources
- Meta Pixel standard events reference — https://developers.facebook.com/docs/meta-pixel/reference
- Meta Business Help: standard events specifications — https://www.facebook.com/business/help/402791146561655
