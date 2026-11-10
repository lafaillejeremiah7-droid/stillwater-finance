# Stillwater Private Finance — local-first PWA

## Status
Working static-source PWA with no build step or paid dependencies. Core automated tests pass. Browser integration, Safari installation, offline service worker, and encrypted backup round-trip have NOT been verified on an actual iPhone. No live deployment was performed.

## Hosting and installation
Host the entire directory unchanged on an HTTPS static host (e.g. GitHub Pages, Cloudflare Pages, Netlify static deploy). The host serves HTML, JS, CSS, SVG and manifest only; imported financial records are kept in browser IndexedDB. Do not commit real exports or backups. Do not use a third-party script, analytics or CDN. This app contains no external network calls. On iPhone: open the HTTPS address in Safari > Share > Add to Home Screen > Add. Launch from Home Screen, open Settings and import JSON. Open online at least once to cache offline assets. `file://` opening does not enable PWA service workers or reliable IndexedDB behavior. A hosting provider can observe visits and IP addresses even though financial records are not transmitted.

## Import schema version 1
JSON root: `{ "schemaVersion": 1, "accounts": [...], "transactions": [...], "netWorthHistory": [...] }`.
Account: `{ "id":"stable-id", "name":"Checking", "kind":"checking", "balanceCents":12345, "asOf":"2026-11-10" }`. Kinds: checking, savings, cash, investment, credit, loan, other-asset, other-liability. Liability balances are positive amounts owed. Balances are snapshots, not recalculated from imported transactions. Latest account `asOf` wins for matching IDs.
Transaction: `{ "id":"stable-id", "accountId":"stable-account-id", "date":"2026-11-10", "merchant":"Example", "category":"Groceries", "type":"expense", "amountCents":1234 }`. Types: income, expense, transfer. All amounts nonnegative integer cents. Use income for refunds; currently refunds are included in income rather than netted against original category spending. For a transfer between accounts, use transfer entries in both accounts, not income/expense. Do not count card payments as expenses twice. `netWorthHistory` is an optional list of `{ "date":"YYYY-MM-DD", "amountCents":12345 }` snapshots. Import doesn't infer or manufacture historical values.
CSV: header `id,accountId,date,merchant,category,type,amountCents` (optional notes). CSV requires account IDs that already exist in the app. RFC-style quoted commas and escaped quotes supported. Stable IDs and account/date/merchant/type/amount fingerprints suppress duplicates. Similar but genuinely separate same-day purchases may be treated as duplicates; review imports. There is no automated transaction reconciliation.

## ChatGPT Finances export prompt
"Use my authorized Finances connection to retrieve available account balances and transaction records. Produce ONLY a downloadable UTF-8 JSON file conforming to Stillwater Finance schemaVersion 1, with arrays `accounts`, `transactions`, and `netWorthHistory`. Do not fabricate missing fields, transactions, balances, or history. All money values must be integer US-dollar cents, with liabilities positive; dates ISO YYYY-MM-DD. Assign stable account IDs and stable transaction IDs based on actual provider identifiers when available. Use `kind` checking/savings/cash/investment/credit/loan/other-asset/other-liability. Classify transaction `type` as income, expense or transfer. Explicitly exclude transfers from income and expense. Include only verified historical net-worth snapshots. If source data cannot be reliably classified or required fields are missing, omit the record and separately report the omissions in your chat response; do not invent placeholders. Do not claim you have exported records that your connected tool cannot retrieve. Never put my banking credentials or authentication tokens in the file. If a downloadable JSON artifact cannot be created, provide the JSON in a code block for me to save locally."

## Backup and recovery
Settings > Download encrypted backup > enter a unique password of at least 12 characters > save `.swbackup` in a secure place. Backup uses browser Web Crypto AES-256-GCM with random 96-bit nonce and PBKDF2-SHA-256 (310,000 iterations; 128-bit random salt). Password never persisted. Settings > Restore backup > select file > enter password > confirm replacement. There is no recovery if the password is lost. Encrypted backups protect confidentiality and tampering of backup contents; password strength matters. The encrypted backup contains account and transaction records, budgets, goals, category rules and historical snapshots. Restore REPLACES local data, not merges.

## Security and limitations
- Imported financial data resides in IndexedDB; IndexedDB is NOT application-level encrypted. iOS device encryption, passcode and OS sandboxing offer protections, but unlocked-device/browser access and device compromise remain risks.
- No Face ID gate is implemented; web biometrics are not a drop-in local vault unlock.
- Offline service worker caches static app files; data stays in IndexedDB. Safari can evict website storage or the user can clear it. Back up frequently.
- No server sync, no direct ChatGPT plugin access, no bank credential collection, no trackers or external scripts.
- JSON/CSV parsing checks data shapes, limits sizes, validates dates and escapes rendered text. It is not a complete adversarial security audit.
- Recurring bill detection, upcoming bill forecasts, investment price sync, robust refund accounting, cross-account transfer pairing, multi-currency support, transaction pagination and charting-library integration are NOT implemented. The chart uses built-in Canvas rather than an external chart library to avoid dependencies.
- Manual transactions do not automatically alter account snapshot balances. Update account balance separately to match a current statement.
- There is no automatic net-worth history collection; historical points must be imported.
- A financial export file is sensitive even when the PWA is local-first. Handle raw JSON/CSV exports securely.
- For updates, deploy all static files together. Service worker currently caches its app shell and uses cache-first reads; a version bump in `sw.js` CACHE constant and reopening online are required to ensure new assets replace old cached assets.

## Test commands
`node test.js` validates account arithmetic, income/expense/transfer treatment, duplicate detection, CSV parsing, invalid date, malformed amount and merchant category rules. JS syntax checked with `node --check`. Test coverage is partial, not a full audit. Test on real iPhone before using it as your only financial record.
