# monobank marketplaces

Connects Claude to monobank's two marketplaces through the `monobank` remote MCP connector
(`https://ai.monoecom.tech/mcp`), and teaches Claude how to use it well.

- **Market (Маркет)** — the user as a buyer: search and compare products, manage the cart, get a link that opens
  checkout in the monobank app (payment happens only there), and follow orders, delivery and Nova Post tracking.
- **Bazar (Базар)** — the user as a ФОП seller: see every advertisement with its views, price offers, auctions,
  orders to ship and payouts; change price, title or description; add photos; create and publish a new
  advertisement from photos; delete one.

## Getting started

Install the plugin, open its Connectors tab and press Connect next to `monobank`. Sign in with the phone number of
your monobank account and confirm with the code from the push in the monobank app. Then just ask: "що з моїми
продажами на Базарі?" or "знайди навушники до 3000 на Маркеті".

Uploading photos to Bazar runs a command in Claude's code-execution environment, so `ai.monoecom.tech` has to be
among the allowed domains in Settings → Capabilities. Everything else works without it.

## Skills

- `market` — the buying flow from search to checkout link, and order questions.
- `bazar` — the seller's overview, editing, photos, creating and deleting advertisements.
