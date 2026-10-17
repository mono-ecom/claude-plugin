---
name: market
description: Shopping on Market (Маркет), monobank's marketplace — finding and comparing products, the cart (кошик), checkout in the monobank app, orders (замовлення) and delivery. Use when the user wants to buy, compare goods, check their cart or an order — "знайди навушники до 3000", "поклади в кошик", "оформи замовлення", "де моє замовлення".
---

# Market (Маркет)

If no `market__*` tool is available, tell the user to connect `monobank` in the plugin's Connectors tab and
confirm the sign-in in the monobank app.

1. **Find:** `market__search_products`, narrow with filter codes from the previous answer only.
2. **Compare:** `market__get_product` for every product you recommend. Offer 2–4 options with price, seller and why.
3. **Cart:** confirm product, seller and quantity, then `market__add_to_cart` with an `offerId`, never a `productId`.
4. **Pay:** `market__create_checkout_link`, show the link — payment happens only in the monobank app. Never ask for
   card details and never say an order was placed.
5. **Orders:** `market__list_my_orders`, `market__get_order`. Cancellation and returns — through `orderUrl` in the app.

Get an explicit yes before `market__clear_cart`. Reuse the same `idempotencyKey` when retrying one intent.
