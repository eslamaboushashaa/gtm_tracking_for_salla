# REFERENCE — Snap Pixel via GTM on Salla

Salla dataLayer shapes were captured from live Salla stores in 2026; every ID and customer value below is fictional. Template IDs (`cvt_…`) are the values seen when this was written — each build reads the real one from the installed template.

## WHAT YOU ARE BUILDING (fixed spec)

**10 tags** on the installed Snapchat "Snap Pixel" gallery template (+1 conditional, Step 8c):

| Tag name | Snap `event_type` | Fires on (Salla dataLayer event) | Data coverage |
|---|---|---|---|
| `EA - Snap Pixel - PAGE_VIEW` | `PAGE_VIEW` | All Pages (`2147479553`) | ✅ |
| `EA - Snap Pixel - LIST_VIEW` | `LIST_VIEW` | `impressions` | ✅ **Snap-only** |
| `EA - Snap Pixel - VIEW_CONTENT` | `VIEW_CONTENT` | `Product Viewed` | ✅ full |
| `EA - Snap Pixel - SEARCH` | `SEARCH` | `Products Searched` | ✅ search term |
| `EA - Snap Pixel - ADD_TO_WISHLIST` | `ADD_TO_WISHLIST` | `Product Added to Wishlist` (success = true) | ⚠️ event only — no product data |
| `EA - Snap Pixel - ADD_CART` | `ADD_CART` | `Product Added` | ✅ full |
| `EA - Snap Pixel - START_CHECKOUT` | `START_CHECKOUT` | `checkout::started` | ⚠️ value + currency only |
| `EA - Snap Pixel - ADD_BILLING` | `ADD_BILLING` | `payments::apple-pay.start-transaction` | ⚠️ Apple Pay only |
| `EA - Snap Pixel - PURCHASE` | `PURCHASE` | `Order Completed` | ✅ full |
| `EA - Snap Pixel - LOGIN` | `LOGIN` | `auth::logged.in` (success = true) | ✅ **Snap-only** |
| *(conditional)* `EA - Snap Pixel - ADD_CART - Bought Together` | `ADD_CART` | `salla-bought-together::product.added` | only if the bought-together check fails |

**Snap's template loads the pixel itself** (injects `scevent.min.js`, calls `snaptr('init')`) — unlike TikTok, no base Custom HTML tag. The PAGE_VIEW tag does the init.

Fixed decisions — do not re-litigate:
- `item_ids` is a real **array** of Salla product IDs, never a comma-joined string.
- Advanced Matching uses **plain** `user_email` = `customer.email` and `user_phone_number` = `customer.phone`. Snap normalises and hashes them. Salla has no plain first/last name (only hashes) and no `external_id` — leave those fields empty.
- Purchase value = `0.prices.total` (amount paid incl. shipping). `transaction_id` and `client_dedup_id` = `0.order_id`.
- ADD_TO_WISHLIST is sent **without** product data — never borrowed from `0.id` (fact #14).
- **Never set `gateway_url`.** See Gotcha #2.

---

## EVENT COVERAGE — why exactly these

Event list taken from the Snap Pixel GTM template's own `event_type` options (the authoritative list of what GTM can send) and Snap's standard-event documentation.

| Snap standard event | Salla source | Verdict |
|---|---|---|
| PAGE_VIEW | All Pages | ✅ Build |
| LIST_VIEW | `impressions` (`ecommerce.impressions`) | ✅ Build — product IDs on category/list pages |
| VIEW_CONTENT | `Product Viewed` | ✅ Build |
| SEARCH | `Products Searched` | ✅ Build — `search_string` = `query` |
| ADD_TO_WISHLIST | `Product Added to Wishlist` | ⚠️ Build — no product data in Salla's push |
| ADD_CART | `Product Added` | ✅ Build |
| START_CHECKOUT | `checkout::started` | ⚠️ Build — value/currency only |
| ADD_BILLING | `payments::apple-pay.start-transaction` | ⚠️ Build — Apple Pay only |
| PURCHASE | `Order Completed` | ✅ Build |
| LOGIN | `auth::logged.in` | ✅ Build — filtered to `success: true` |
| SIGN_UP | — | ❌ Salla's OTP flow can't distinguish a signup from a login |
| SAVE, SHARE, RATE, INVITE, RESERVE, SUBSCRIBE, START_TRIAL | — | ❌ No matching user action in Salla |
| APP_INSTALL, APP_OPEN, LEVEL_COMPLETE, COMPLETE_TUTORIAL, ACHIEVEMENT_UNLOCKED, SPENT_CREDITS, AD_CLICK, AD_VIEW | — | ❌ App/game events, not web |
| CUSTOM_EVENT_1–5 | — | ➖ Available later (e.g. WhatsApp click) — not built by default |

Salla events with **no** Snap standard event — do not build: `Product Removed`, `Cart Viewed`, `salla-search::product.clicked`, `search::open`.

**About `impressions`:** no raw push sample exists — confirm the `ecommerce.impressions[]` shape in GTM Preview on a category page before trusting LIST_VIEW. It fires on every list page, so expect high volume.

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
5. `payments::apple-pay.start-transaction` fires only for Apple Pay's "Pay now" (often `checkout_type: "buy_now"`). Card and mada payments produce **no payment-info event**.
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
| `EA - Core - salla_payment_amount` | `amount` | Meta, TikTok, Snap |
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
| `EA - Trigger - payments::apple-pay.start-transaction` | `payments::apple-pay.start-transaction` | — | Meta, TikTok, Snap |
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
4. **Naming:** tags `EA - Snap Pixel - <SNAP_EVENT_TYPE>` (Snap's own constants); platform variables `EA - Snap Pixel - <snake_case>`; shared Salla variables `EA - Core - salla_<snake_case>`; new triggers `EA - Trigger - <dataLayer event name>`; folders `EA - Snap Pixel` and `EA - Core`.
5. **Pixel ID lives only in the constant `EA - Snap Pixel - pixel_id`.**
6. **Set `parentFolderId` at creation time** on every entity. Snap items → `EA - Snap Pixel`; Salla foundation → `EA - Core`.
7. Every entity gets a `notes` value.
8. Respect fact #12 (privacy). No exceptions.

---

## PROCEDURE

### Step 1 — Locate the container
`gtm_account list` → `gtm_container list` → `gtm_workspace list`. People give approximate names ("acme website under agency new" → account `Agency - new`, container `Acme - website`): one clear match → state it and continue; none or several → ask. Confirm the container's `publicId` (`GTM-XXXX`).

Create a dedicated workspace `EA - Snap Pixel` and build there — never in the Default Workspace. If a write returns **"Workspace is already submitted"**, the client published meanwhile: re-run `gtm_workspace list` and continue in the new workspace.

### Step 2 — Live-site checks
In the browser tool, on the live store (skip this step if no browser tool is available, and list every unchecked item in the report):
1. **dataLayer name** — read `gtm.js?id=…&l=<name>`; confirm `Product Viewed` lands in that array. Mismatch → stop.
2. **Native Salla Snap pixel** — read `store.snapchat_pixel_id` (thank-you page sample, or ask the user). **Not null → stop and warn the user**: Salla is already tracking Snap. Also check for an `sc-static.net` / `tr.snapchat.com` script not loaded by GTM.
3. **Search frequency** — type a two-word search slowly and count `Products Searched` pushes. Per-keystroke firing → report it.
4. **`impressions` shape** — on a category page, read one `impressions` push and confirm `ecommerce.impressions[].id` exists. If the shape differs, adapt `EA - Snap Pixel - list_item_ids` and say so.
5. **Bought-together check** (see the foundation section).

### Step 3 — Template tag type
`gtm_template list` (huge — parse the saved file). Find **Snap Pixel** (gallery owner `Snapchat`). `___INFO___` → `"id"` — currently **`cvt_K4VXG`**. Not installed → stop and tell the user (Templates → Search Gallery → "Snap Pixel" by Snap Inc.).

### Step 4 — Audit
List triggers, variables and tags — **every page** (20 per page).

### Step 5 — Folders
`EA - Snap Pixel`; `EA - Core` if missing.

### Step 6 — Salla foundation
Create what Snap uses and is missing: DLVs marked *all* / *Snap*, `EA - Core - salla_success`, `EA - Core - salla_impressions`, the constant, the non-conditional Custom JS, and **all nine** non-conditional triggers (Snap is the only skill that uses `auth::logged.in` and `impressions`). Conditional items only per the bought-together check.

### Step 7 — Snap variables (folder `EA - Snap Pixel`)

`EA - Snap Pixel - pixel_id` — Constant = pixel ID.

`EA - Snap Pixel - product_quantity` — Salla sends quantity as a string on Product Added
```js
function() {
  return Number({{EA - Core - salla_product_quantity}}) || 1;
}
```

`EA - Snap Pixel - list_item_ids` — capped at 50
```js
function() {
  var imp = {{EA - Core - salla_impressions}};
  if (!imp || !imp.length) { return undefined; }
  var ids = [];
  for (var i = 0; i < imp.length && ids.length < 50; i++) {
    var id = imp[i] && imp[i].id;
    if (id) { ids.push(String(id)); }
  }
  return ids.length ? ids : undefined;
}
```

`EA - Snap Pixel - payment_info_available`
```js
function() {
  return 1;
}
```

### Step 8 — Tags

In the tables below, *Trigger* names the Salla dataLayer event: use the trigger found or created for it in Step 4/6 (reused, or `EA - Trigger - <event>`), referenced by its trigger ID. Create everything in Steps 5–8 with one `gtm_workspace` `bulkUpdate` where the server supports it — temporary IDs (`new_1`, `new_2`, …) let new tags reference new triggers and folders in the same call.

`type` from Step 3, folder Snap. Base block on every tag:
```json
{"type": "template", "key": "pixel_id",          "value": "{{EA - Snap Pixel - pixel_id}}"},
{"type": "template", "key": "event_type",        "value": "<SNAP_EVENT>"},
{"type": "template", "key": "user_email",        "value": "{{EA - Core - salla_customer_email}}"},
{"type": "template", "key": "user_phone_number", "value": "{{EA - Core - salla_customer_phone}}"},
{"type": "boolean",  "key": "enable_console_logging", "value": "false"}
```

| Tag | `event_type` | Trigger | Extra parameters |
|---|---|---|---|
| PAGE_VIEW | `PAGE_VIEW` | `2147479553` | — |
| LIST_VIEW | `LIST_VIEW` | `impressions` | `item_ids`=`{{EA - Snap Pixel - list_item_ids}}`, `currency`=`{{EA - Core - salla_currency}}` |
| VIEW_CONTENT | `VIEW_CONTENT` | `Product Viewed` | `item_ids`=`{{EA - Core - salla_product_ids}}`, `item_category`=`{{EA - Core - salla_product_category}}`, `description`=`{{EA - Core - salla_product_name}}`, `price`=`{{EA - Core - salla_product_price}}`, `number_items`=`{{EA - Snap Pixel - product_quantity}}`, `currency`=`{{EA - Core - salla_currency}}` |
| SEARCH | `SEARCH` | `Products Searched` | `search_string`=`{{EA - Core - salla_search_query}}` |
| ADD_TO_WISHLIST | `ADD_TO_WISHLIST` | `Product Added to Wishlist` | — *(no product data exists)* |
| ADD_CART | `ADD_CART` | `Product Added` | `item_ids`=`{{EA - Core - salla_product_ids}}`, `item_category`=`{{EA - Core - salla_product_category}}`, `description`=`{{EA - Core - salla_product_name}}`, `price`=`{{EA - Core - salla_add_to_cart_value}}`, `number_items`=`{{EA - Snap Pixel - product_quantity}}`, `currency`=`{{EA - Core - salla_currency}}` |
| START_CHECKOUT | `START_CHECKOUT` | `checkout::started` | `price`=`{{EA - Core - salla_checkout_value}}`, `currency`=`{{EA - Core - salla_currency}}` |
| ADD_BILLING | `ADD_BILLING` | `payments::apple-pay.start-transaction` | `price`=`{{EA - Core - salla_payment_amount}}`, `currency`=`{{EA - Core - salla_currency}}`, `payment_info_available`=`{{EA - Snap Pixel - payment_info_available}}` |
| PURCHASE | `PURCHASE` | `Order Completed` | `item_ids`=`{{EA - Core - salla_order_item_ids}}`, `price`=`{{EA - Core - salla_order_value}}`, `number_items`=`{{EA - Core - salla_order_num_items}}`, `currency`=`{{EA - Core - salla_currency}}`, `transaction_id`=`{{EA - Core - salla_order_id}}`, `client_dedup_id`=`{{EA - Core - salla_order_id}}` |
| LOGIN | `LOGIN` | `auth::logged.in` | — |

`description` carries the product name because Snap has no `content_name` field.

### Step 8c — CONDITIONAL: bought-together ADD_CART
Only if the bought-together check showed **no** `Product Added`. Tag `EA - Snap Pixel - ADD_CART - Bought Together`: `event_type` `ADD_CART`, trigger `salla-bought-together::product.added`, `item_ids`=`{{EA - Core - salla_bt_product_ids}}`, `item_category`=`{{EA - Core - salla_bt_product_category}}`, `description`=`{{EA - Core - salla_bt_product_name}}`, `price`=`{{EA - Core - salla_bt_product_price}}`, `number_items`=`1`, `currency`=`{{EA - Core - salla_currency}}`. Quantity is 1 because the widget push carries no added-quantity field.

### Step 9 — Verify
`gtm_folder action=entities` on both folders; 10 tags (11 with the conditional); every variable/trigger foldered; every `{{...}}` matches an existing variable. Paginate. Then run `gtm_workspace` `quickPreview` and search the (large) result for `compilerError`.

### Step 10 — Report and stop
IDs, template ID, created vs reused, live-site check results (dataLayer name, `store.snapchat_pixel_id`, search frequency, `impressions` shape, bought-together outcome), the partial events, and that `gateway_url` was left untouched. **Do not publish.**

---

## CRITICAL GOTCHAS

1. **Stale `"0"` data** — compute values from fields in the current push (fact #2). ADD_CART uses `price × quantity`, not `0.value`; the bought-together event uses `0.product.*`, never `0.id`.
2. **Never set `gateway_url`.** It's labelled "CAPI Gateway Script URL" and has `defaultValue: "https://sc-static.net/scevent.min.js"` built in. Setting it pins the URL against template updates and falsely implies a Conversions API Gateway exists. GTM's UI repopulates the default when cleared, so it *looks* set when nothing is stored — cosmetic.
3. **`uuid_c1` / `_scid` is not a pixel parameter.** Snap lists it under the Conversions API (server-side). The browser pixel writes `_scid` itself; the template has no `uuid_c1` field.
4. **One source per action** — `Order Completed` for purchase; never the legacy `addToCart` / `removedFromCart` / `detail` duplicates.
5. **Custom-named dataLayer** — check `l=` in Step 2.
6. **`update` replaces the whole entity** — send complete object + current `fingerprint`.
7. **Rate limit** — `429` is transient.
8. **Template list / workspace status overflow the token limit** — parse the saved file.

---

## POST-PUBLISH CHECKLIST (for the person who publishes)

- GTM Preview: each event fires once; `item_ids` is a real array on product, list and thank-you pages.
- Add quantity 3: ADD_CART `price` = 3 × unit price, `number_items` = 3.
- Search a word: SEARCH carries `search_string`, Arabic text intact.
- Heart a product on a category page: ADD_TO_WISHLIST fires with no product attached.
- Snap Pixel Helper: no errors; PURCHASE carries `transaction_id`.
- Snapchat Events Manager → Test Events: LIST_VIEW, SEARCH and LOGIN arrive.
- Salla dashboard: Snap integration field empty.

---

## Sources
- Snap Pixel GTM template (event_type option list) — https://github.com/Snapchat/snapchat-google-tag-manager
- Snap Business Help — Pixel Helper glossary (standard events) — https://businesshelp.snapchat.com/s/article/pixel-helper-glossary?language=en_US
- Snap Business Help — Directly implement the pixel — https://businesshelp.snapchat.com/s/article/pixel-direct-implementation?language=en_US
- Snap Marketing API — Conversion parameters (`uuid_c1` is server-side) — https://marketingapi.snapchat.com/docs/conversion.html#conversion-parameters
