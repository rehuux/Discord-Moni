# IG Monitor Bot V2

Discord bot for Instagram profile status and follower monitoring. Built with Node.js, Discord.js, Puppeteer and MongoDB.

## Requirements
- Node.js 20 or newer (Docker image uses Node 20)
- MongoDB connection string
- Discord bot token with the required intents enabled in the Discord Developer Portal
- Chromium (the supplied Dockerfile installs system Chromium)

## Environment variables
Create `.env` locally or configure these in Railway. Never commit `.env`.

```env
DISCORD_TOKEN=your_discord_bot_token
MONGODB_URI=mongodb+srv://...
# Optional
MONGODB_URI_BACKUP=
OWNER_ID_1=
OWNER_ID_2=
ALLOWED_USER_IDS=
CHECK_INTERVAL=90000
MAX_CONCURRENT_CHECKS=1
INSTAGRAM_COOLDOWN_MS=900000
CHECK_RETRY_DELAY_MS=5000
PUPPETEER_EXECUTABLE_PATH=/usr/bin/chromium
```

`DISCORD_TOKEN` is the variable read by the application. The old README incorrectly documented `DISCORD_BOT_TOKEN`.

## Install and run
```bash
npm ci
npm run check
npm start
```

## Docker / Railway
Use the repository Dockerfile. It installs Chromium and performs a lockfile-based `npm ci`. Set `DISCORD_TOKEN` and `MONGODB_URI` in Railway Variables. Keep one application replica unless you add distributed locking; running multiple replicas can duplicate monitoring jobs.

## Reliability notes
- Instagram HTTP 429 triggers a global cooldown (15 minutes by default).
- Login redirects, challenges, server errors and missing/unparseable profile metadata are treated as UNKNOWN and do not trigger ban/unban notifications.
- Same-username checks are deduplicated in-process; Chromium concurrency defaults to one.
- Chromium is recycled every six hours.
- MongoDB connection failure stops startup instead of starting a partially working bot.

Instagram may restrict automated requests. No scraper can guarantee uninterrupted access or eliminate upstream rate limits.

## Security
Do not commit `.env`, bot tokens, MongoDB credentials, cookies or browser profiles. Rotate any credentials that were previously exposed.

## License
MIT.
