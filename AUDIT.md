# IG-MONI audit and fixes

## Scope
This package is the existing Discord.js bot. It is not a Telegram migration. The audit covered the supplied JavaScript source, Mongoose schema calls, package manifest/lockfile, Dockerfile and deployment documentation.

## Fixes applied

1. **False ban classification:** missing or unparseable Instagram metadata, login redirects, challenge/checkpoint pages, HTTP 429, HTTP 5xx and other unexpected HTTP errors now produce `UNKNOWN` (`null` from the checker). They do not trigger ban/unban state changes. An explicit Instagram unavailable page or HTTP 404/410 is treated as unavailable/banned.
2. **HTTP 429 handling:** a global 15-minute cooldown is applied after HTTP 429. Further checks are skipped during cooldown rather than repeatedly hitting Instagram.
3. **Duplicate profile checks:** simultaneous checks for the same normalized username share one in-flight promise.
4. **Chromium load:** concurrency defaults to one and can be configured up to two with `MAX_CONCURRENT_CHECKS`. Chromium is recycled every six hours after active checks finish.
5. **Shutdown:** managed periodic intervals are stopped on SIGTERM/SIGINT; new check slots are not acquired during shutdown; Discord client and Chromium are closed.
6. **MongoDB startup:** startup now fails if all configured MongoDB URIs fail, instead of starting a partially initialized bot.
7. **Restore behavior:** paused accounts are not resumed; a temporarily unavailable Discord channel no longer causes its MongoDB watch record to be deleted.
8. **Mongoose history validation:** added the `verified` event to the Stat schema enum because the bot already writes that event.
9. **Deployment hygiene:** corrected `DISCORD_TOKEN` documentation, added `.env.example` and `.gitignore`, removed unused `axios` and `sqlite3` dependencies, removed the broken `build-exe` script reference (the referenced `build-exe.js` was absent), switched Docker installation to `npm ci`, and normalized lockfile registry URLs to npmjs.org.

## Validation performed

- `node --check` passed for every JavaScript source file.
- `npm run check` passed.
- `npm ci --omit=dev --ignore-scripts --dry-run --no-audit --no-fund` passed against the updated package manifest and lockfile.
- ZIP integrity check passed.

## Not verified in this environment

- A live Discord login, live MongoDB connection, Railway deployment, real Chromium launch against Instagram, or an actual Instagram status transition. These require the user's credentials and live external services.
- Instagram can still rate-limit or change its HTML. The bot handles these conditions as UNKNOWN, but cannot guarantee Instagram availability or eliminate HTTP 429.
- The current data model is designed around Discord channel/guild IDs and one destination per watched username. This release is not a multi-user Telegram service and does not claim 500–2,000-user Telegram readiness.

## Required Railway variables

- `DISCORD_TOKEN`
- `MONGODB_URI`

Optional variables are documented in `.env.example` and `README.md`.
