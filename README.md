# PX9V Landing Provisioner — updated template

This package contains a Telegram landing/store bot that provisions isolated PX9V bot instances from the supplied `9V-main.zip` template.

## Landing flow

`/start` → **BUY NEW BOT** → send access/create key → send customer bot token → send Admin ID → bot is provisioned and started.

If the access key was created with Premium enabled, the new PX9V instance receives a one-time PX9V-compatible Premium Key. The key uses the same `premium_keys` schema and redemption flow as the supplied PX9V source.

## Admin

`/admin` opens the landing admin panel.

**CREATE TOKEN** asks for:
1. USER / bot-creation count
2. Whether a Premium Key should be included
3. Premium KEY USER limit
4. Premium KEY REVENUE limit
5. Premium KEY TIME in days

The access key can therefore provision exactly the configured number of customer bots.

## Revoke

A customer can revoke their own bot with:

- `/revoke @botusername`
- `/revoke ACCESS_TOKEN`

Revoking an access token also revokes/stops every bot created from that access token.

The landing admin can revoke any bot/token.

## Important environment variables

```text
LANDING_BOT_TOKEN=...
LANDING_ADMIN_ID=...
LANDING_DB_PATH=/data/landing_bot.db
INSTANCE_ROOT=/data/instances
TEMPLATE_ROOT=px9v_template
MASTER_INSTANCE_KEY=<44-character URL-safe Fernet key>
REVOKE_DELETE_FILES=true
```

Generate `MASTER_INSTANCE_KEY` without installing cryptography on the phone:

```bash
python -c "import secrets,base64; print(base64.urlsafe_b64encode(secrets.token_bytes(32)).decode())"
```

## Railway

Add a Railway Volume mounted at `/data`. This keeps the landing DB and customer bot instances across redeployments.

Start command:

```text
python main.py
```

## PX9V template / QR handling

The customer bot template is the newly supplied `9V-main.zip`, not the previous template.

Its `requirements.txt` includes `qrcode[pil]>=8.0`. Additionally, the QR code handler has an HTTP QR fallback, so the customer bot does not show the old `QR module is missing on the server` error merely because the optional local QR package is unavailable.


## New son-bot permissions and themes

- Every ID in `LANDING_ADMIN_ID` and the optional comma-separated `LANDING_ADMIN_IDS` is passed into each provisioned son bot as `FATHER_ADMIN_IDS`.
- Those father-bot administrators receive `/padmin` and the same permanent-admin permissions inside every son bot. The customer-selected Admin ID remains a permanent admin too.
- Son-bot `/admin` now includes **CHANGE THEME** and **RESTRICT BOT**.
- Theme 1 preserves the existing layout. Theme 2 displays premium plans as one button per row and menu buttons in pairs (Menu 1 | Menu 2, then Menu 3, etc.).
- `/buttoncolor [ID] [R/G/B]` controls premium-button colors; `/menucolor [ID] [R/G/B]` controls menu-button colors in either selected theme.

## Protected content limitation

**RESTRICT BOT** enables Telegram's `protect_content` default for bot-sent messages after the son bot is restarted. This restricts forwarding and saving for supported messages. Telegram bots cannot prevent screenshots or screen recordings, and this feature must not be described as doing so.
