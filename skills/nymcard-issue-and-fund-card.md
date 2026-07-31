---
name: Issue and fund an nCore card
description: Create a cardholder, issue a card against a card product, and load funds so the card is ready to transact.
api: NymCard nCore
auth: apiKey header `apikey`
operations: [createUser, listCardProducts, createCard, getCardAccounts, loadAccountFunds, getAccount]
source: https://docs.nymcard.com/get-started/quick-tutorial
---

# Issue and fund an nCore card

Operating instructions for an agent using the NymCard nCore API to take a new
cardholder from zero to a funded, transaction-ready card. Every step maps to a
real published operationId.

## Auth & conventions
- Send the static API key in the `apikey` request header on every call. All
  traffic is HTTPS/TLS only.
- All POST requests are idempotent: set an `x-nymos-idempotency-key` header and
  reuse it verbatim on retries so a timed-out request is never double-applied.
- Base URL is version-pinned in the path: `https://api.nymcard.com/v1/...`.
- List endpoints page with `limit` + `after` (cursor); read `paging`/`after`/
  `has more` from the response.

## Steps
1. **Create the user** — `POST /users` (`createUser`) with the cardholder's
   details (e.g. `first_name`, `last_name`). Save the returned `id`.
2. **Pick a card product** — `GET /cardproducts` (`listCardProducts`). A card
   product is the template that defines currency, PIN, and authorization
   behavior. Save the chosen product `id`.
3. **Issue the card** — `POST /cards` (`createCard`) with `user_id`,
   `card_type` (`VIRTUAL` or `PHYSICAL`), and `card_product_id`. Save the
   card `id`.
4. **Get the card account** — `GET /cards/{id}/accounts` (`getCardAccounts`).
   An account is auto-created and linked unless the card product sets
   `link_account_to_card: false`. Save the account `id`.
5. **Load funds** — `POST /accounts/{id}:loadfunds` (`loadAccountFunds`) with
   `currency` and `amount` to move money from the program funding account into
   the card account.
6. **Confirm balance** — `GET /accounts/{id}` (`getAccount`) and verify the
   available balance before the card is used.

## Error handling
- Standard HTTP status codes apply (401 auth, 403 forbidden, 404 not found,
  409 duplicate, 429 rate-limited, 5xx platform) — see
  `errors/nymcard-problem-types.yml`.
- Transaction/authorization outcomes carry a numeric `status_code` (e.g. `0000`
  approved, `1806` card not active) — see `errors/nymcard-decline-codes.yml`.
- Subscribe to webhooks (`TRANSACTION`, `CARD_STATUS_CHANGE`,
  `ACCOUNT_STATUS_CHANGE`) for async state — see `asyncapi/nymcard-webhooks.yml`.
