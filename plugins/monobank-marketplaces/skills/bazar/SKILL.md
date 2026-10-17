---
name: bazar
description: A seller's Bazar (Базар) advertisements (оголошення) — sales, price offers (торг), auctions, orders to ship, payouts; changing price, title or description; photos; creating and deleting advertisements. Use when a seller talks about their Bazar listings — "що з моїми продажами", "підніми ціну на кросівках", "додай фото", "хочу продати", "видали оголошення".
---

# Bazar (Базар)

If no `bazar__*` tool is available, tell the user to connect `monobank` in the plugin's Connectors tab and
confirm the sign-in in the monobank app.

- **Overview:** `bazar__list_my_advertisements`. Lead with what needs the seller's action, summarise the rest.
- **Change:** find with `bazar__find_my_advertisement` (ask if several match), show the current and new value, get
  a yes, then update.
- **Photos:** `bazar__create_photo_upload_link`, run its command on a file the user attached, then use the
  `photoUploadId`. Photos only on the user's phone — they need the mono app.
- **New advertisement:** upload photos → `bazar__create_advertisement_draft` with the user's own words → ask
  Bazar's questions → show the result → publish only after a yes.
- **Delete:** permanent. Show that exact advertisement and get a yes for it.

Never invent facts about an item. Reuse the same `idempotencyKey` when retrying one intent.
