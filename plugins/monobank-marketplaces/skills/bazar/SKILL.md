---
name: bazar
description: Running a ФОП seller's Bazar (Базар) advertisements (оголошення) through the monobank connector — what is happening with sales, price offers (торг), auctions (аукціон), orders to ship and payouts (виплата); changing price, title or description; adding photos; creating a new advertisement from photos; deleting one. Use whenever a seller talks about their Bazar listings — "які в мене оголошення", "що з моїми продажами", "є торги?", "підніми ціну на кросівках", "зміни опис", "додай фото", "хочу продати", "створи оголошення", "видали оголошення". Not for buying: shopping on Market (Маркет) is the market skill.
---

# Bazar (Базар) — the user as a seller

The `bazar__*` tools of the monobank connector act only on the signed-in seller's own advertisements. Each tool's
description has the details; this is how the pieces fit together.

## Before anything: is the connector there?

If no `bazar__*` tool is available, the connector is not connected yet. Tell the user, in their language:

1. In Claude: **Customize → Connectors** (or the plugin's **Connectors** tab), find **monobank**, press **Connect**.
2. The sign-in page asks for the phone number of their monobank account.
3. A push arrives in the monobank app with a code (`XX-XX-XX`); they type it on the page and allow access.

If the tools are there but a call says the account has no Bazar seller profile, the seller has to open Bazar in
the mono app first; nothing here creates one.

## "How are my sales?" — overview

`bazar__list_my_advertisements` returns the whole list in one call, each item with views, favourites (обране),
stock, a price offer (торг) waiting, an auction, an order to ship, a payout.

- Lead with `REQUIRES_ACTION` items (a торг waiting for the seller, an order to ship), then `REQUIRES_ATTENTION`,
  then the rest in one short summary. Do not dump every field.
- Deadlines (`offerExpiresAt`, `shipmentDueAt`, `paymentDueAt`, `payoutScheduledAt`) are compared with `asOf`
  from the answer, not with your own idea of "now". Say them as "до завтра 14:00", not as raw timestamps.
- No `nextCursor` means the list is complete — say so, do not offer to load more.
- Answering a торг, shipping an order, and stock are done in the app; say so instead of reaching for a `market__`
  tool, which is the other marketplace.

## Changing an advertisement

Always the same three steps:

1. **Find it.** `bazar__find_my_advertisement` with the user's words (`query`). Several matches → show the titles
   (and photos, when the answer carries them) and ask which one. Never guess.
2. **Show and confirm.** Show the current value and the new one, and get a yes. These overwrite live data.
   - Price: pass `expectedCurrentPriceUah` with the price you showed, so a change made in the app meanwhile is not
     overwritten.
   - Description: `newDescription` replaces the whole text (10–1000 chars). For "додай, що є доставка" write the
     full new text yourself from the current one and show it.
   - Title: 3–100 chars.
3. **Apply and report** by the result: `APPLIED` — done; `PROCESSING` — accepted, still being applied or moderated,
   re-read with `bazar__find_my_advertisement` in a few seconds and do not send the same change again;
   `UNCHANGED` — it already was so; `NOT_APPLIED` — refused, pass on the reason.

An edit is refused while a торг waits for an answer or an auction has bids — tell the seller to deal with it in the
app first. Titles and descriptions go through moderation.

## Photos

A photo travels in two calls, because the file goes from your own environment, not through the connector:

1. `bazar__create_photo_upload_link` → run its `uploadCommand` with `<FILE_PATH>` replaced by the file's path. The
   last line is the HTTP status; 200 means stored.
2. `bazar__add_advertisement_photo` with the `photoUploadId` (existing advertisement), or pass the ids to
   `bazar__create_advertisement_draft` (new one).

What to know:
- JPEG, PNG or WebP, at most 1.5 MB, at most 9 photos per advertisement. The link lives 10 minutes and works once.
- The file must be in this conversation (attached by the user) — you cannot reach their phone's gallery. If the
  photos are only on the phone, say the advertisement or photo has to be added in the mono app.
- **Status 000, "CONNECT tunnel failed" or a proxy refusal** means your environment's network does not allow the
  upload host. Retrying will not help. Tell the user to open Claude **Settings → Capabilities**, and under code
  execution network access add the host from `uploadUrl` — `ai.monoecom.tech` — to the allowed domains (on a Team
  or Enterprise plan an owner does it in Organization settings). Then ask for a fresh link and run it again.
- A 429 with an empty body is pacing: wait a few seconds and run the same command again.
- Removing or reordering photos of an existing advertisement is not possible here — the app does it.

## Creating a new advertisement

Four tools, because the seller is asked between them:

1. Upload every photo (above), in the order buyers should see them; the first is the main one.
2. `bazar__create_advertisement_draft` with `photoUploadIds`, the price, and `brief` — **the user's own words**
   about the item: what it is, condition, size, defects. Ask for what is missing; never invent facts.
3. If the status is `ATTRIBUTES_PENDING`, ask the user each question in `pendingQuestions` and send the answers with
   `bazar__answer_advertisement_draft_questions`. Short values ("42", "чорний"). Leave out what the user does not
   know — never fill in a guess.
4. At `READY_TO_PUBLISH` show the title, description and price exactly as they will appear. Publish with
   `bazar__publish_advertisement_draft` only after a yes, then give the `sharingUrl`.

`CLIENT_PAYLOAD_PROCESSING` / `POST_CONTENT_PROCESSING` → wait a few seconds and read with
`bazar__get_advertisement_draft`. One unfinished draft per seller: continue it, or discard it
(`discardUnfinishedDraft`) only after telling the user what will be thrown away. In a new conversation,
`bazar__get_advertisement_draft` without an id finds the unfinished one.

## Deleting

Deletion is permanent. Find the advertisement, show that exact one (title, price), and call
`bazar__delete_advertisement` only after an explicit yes for it. Never delete several on one general "yes".

## Always

- Every mutating call takes an `idempotencyKey`: a fresh UUID per user intent, the same one when retrying it.
- A refused call comes with a sentence written for you — pass it on in the user's language. A generic error: retry
  once with the same key, then say Bazar is not answering and suggest the app.
- Answer the seller in their language (usually Ukrainian), with prices in грн.
