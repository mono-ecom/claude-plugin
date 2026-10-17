---
name: market
description: Shopping on Market (Маркет), monobank's marketplace, through the monobank connector — finding and comparing products, the cart (кошик), the hand-off to checkout in the monobank app, and order history (замовлення) with delivery and tracking (ТТН). Use whenever the user wants to buy, choose or compare goods, check their cart, pay, or asks where an order is — "знайди навушники до 3000", "який пилосос взяти", "поклади в кошик", "оформи замовлення", "де моє замовлення", "коли приїде посилка". Not for selling: a ФОП managing their own advertisements (оголошення) is the bazar skill.
---

# Market (Маркет) — the user as a buyer

The `market__*` tools of the monobank connector act on the user's own Market account: the same cart and orders the
monobank app shows. Each tool's description has the details; this is how the pieces fit together.

## Before anything: is the connector there?

If no `market__*` tool is available, the connector is not connected yet. Do not improvise with web search. Tell
the user, in their language, how to connect it:

1. In Claude: **Customize → Connectors** (or the plugin's **Connectors** tab), find **monobank**, press **Connect**.
2. The sign-in page asks for the phone number of their monobank account.
3. A push arrives in the monobank app with a code (`XX-XX-XX`); they type it on the page and allow access.

Then ask them to come back to the conversation. Only a monobank client with the app can sign in.

## Choosing a product

1. `market__search_products` with the user's own words in `query`, and their budget in `priceFromUah` /
   `priceToUah`. A result is a preview.
2. Narrow by calling again with filter and category **codes taken from the previous answer** (`{"brand": ["apple"]}`).
   Never invent a code. Refine with filters rather than firing many searches in parallel — Market limits the rate
   per buyer.
3. `market__get_product` for each product you are about to recommend: description, characteristics, every seller's
   offer, delivery options, warranty. Compare on that, not on the search preview.
4. Present a short choice: 2–4 options, price, seller, rating, what differs between them, and why you would pick
   one. Prices are whole hryvnias (грн).

A product has several sellers' offers. What goes into the cart is an **offer** (`offerId` from `market__get_product`,
or `bestOfferId` from search) — never a `productId`.

## The cart (кошик)

- Confirm the product, seller and quantity with the user before `market__add_to_cart`.
- `quantity` in `market__add_to_cart` is **added** to what is already there; `market__set_cart_item_quantity` sets
  the **total** (0 removes the line). Read the returned cart — Market may fit the quantity to the offer's limits.
- `market__clear_cart` also removes what the user added in the app themselves: show the cart and get an explicit
  yes first.
- Every mutating call takes an `idempotencyKey`: a fresh UUID per user intent, and **the same one** when retrying
  that intent after an error or timeout, so a retry does not add twice.

## Paying

Payment, delivery address and recipient are chosen **only in the monobank app** — nothing here pays.

1. `market__get_cart` and show what will be paid and the total before delivery.
2. `market__create_checkout_link` and show `checkoutUrl` as a link: open it on the phone with the monobank app.
3. If checkout sends them to the app's main screen, their Market account is not set up yet — give `cartUrl`; the
   cart's own checkout button finishes the setup.
4. Mention `linesLeftOut` if it is not zero: those lines stay in the cart unpaid.

Never ask for card numbers or any payment details, and never claim an order was placed — only the app knows that.
After the user says they paid, `market__list_my_orders` shows the new orders.

## Orders (замовлення)

- `market__list_my_orders` for "what did I buy / where is it"; `market__get_order` with `orderNumber` for delivery,
  the Nova Post branch, pick-up deadline, tracking number (ТТН), payment and return status.
- One order is one product from one seller: buying three things makes three orders.
- Cancelling and returns work only in the app: show `orderUrl` and tell the user to open it on the phone.
  `returnAllowed` / `returnAllowedUntil` say whether a return is still possible.

## When something fails

A refused call comes with a sentence written for you — pass it on in the user's language and follow what it says
about retrying. A generic error means retry once with the same `idempotencyKey`; if it fails again, tell the user
Market is not answering right now and suggest the app.
