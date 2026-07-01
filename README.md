# Fayda Bot

Telegram bot for Fayda (Ethiopian National ID) self-service PDF download and print formatting.

## What's stubbed vs real

- **`app/services/fayda.py`** — OTP send/confirm against Fayda is STUBBED. In dev mode
  (no `FAYDA_VERIFY_BASE_URL` set) it accepts OTP `1234` and returns a fake profile.
  Replace with your real, authorized verification integration before going live.
- **`app/services/payment.py`** — Chapa integration. In dev mode (no `CHAPA_SECRET_KEY`)
  it returns a mock checkout URL and never actually charges anyone.

Everything else (menu, FSM flows, DB, PDF generation, rate limiting) is fully implemented.

## Local setup

1. Install Python 3.11+
2. `pip install -r requirements.txt`
3. Copy `.env.example` to `.env` and fill in `BOT_TOKEN` (from @BotFather) and
   `DATABASE_URL` (a local Postgres instance, or use a free Railway/Render Postgres)
4. Run: `python -m app.main`
5. Message your bot on Telegram — try `/start`, then PDF Download with FAN
   `1234567890` and OTP `1234` (dev mode)

## Deploying

### Render
- Push this repo to GitHub, then "New Web Service" on Render, or use `render.yaml`
  (Blueprint deploy) which also provisions the Postgres DB.
- Set the secret env vars (`BOT_TOKEN`, `FAYDA_*`, `CHAPA_*`) in the Render dashboard —
  they're marked `sync: false` in render.yaml so they're not committed.

### Railway
- `railway init`, then `railway up`. Add a Postgres plugin, set `DATABASE_URL`
  automatically via Railway's reference variables, and set the rest of the env vars
  in the Railway dashboard. Railway reads `Procfile` for the start command.

### Payment webhook URL
Whichever host you use, register `https://<your-domain>/payment/webhook` as the
success callback URL in your Chapa (or other gateway) dashboard.

## Safety notes baked into the design

- OTP must succeed before any ID data is generated — see `receive_otp` in each flow handler.
- Raw FAN numbers and OTP results are never persisted; only `fan_last4` is stored on
  an `Order` for support reference.
- OTP attempts are rate-limited per user per hour (`MAX_OTP_ATTEMPTS_PER_HOUR`) to
  prevent the bot being used to brute-force/lookup other people's FANs.
- The bot explicitly tells users to only verify their own ID before starting the flow.
