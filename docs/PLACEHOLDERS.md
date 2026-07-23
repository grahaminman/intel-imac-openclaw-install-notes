# Placeholders

Use these consistently when adapting the guide or generating screenshots.

| Placeholder | Example replacement | Safe to commit? | Notes |
|---|---|---|---|
| `<YOUR_NAME>` | `Alex` | Yes, if fictional/generic | Prefer a fake name in public docs |
| `<ASSISTANT_NAME>` | `Atlas` | Yes | Prefer a generic example name in shared docs |
| `<ASSISTANT_EMOJI>` | `🛠️` | Yes | Optional |
| `<MAC_USERNAME>` | `alex` | Maybe | Real usernames can identify a machine/account |
| `<WORKSPACE_PATH>` | `/Users/alex/OpenClaw/workspace` | Maybe | Keep generic in public docs; home-folder layout only |
| `<TIMEZONE>` | `Europe/London` | Yes | Low sensitivity |
| `<DEFAULT_MODEL>` | `xai/grok-example` | Yes | Prefer example ids in shared docs; confirm real model ids on your machine |
| `<TELEGRAM_BOT_USERNAME>` | `my_openclaw_bot` | Yes | Public username |
| `<TELEGRAM_USER_ID>` | `123456789` | Prefer no | Personal identifier |
| `<DISCORD_BOT_NAME>` | `OpenClawHelper` | Yes | |
| `<DISCORD_USER_ID>` | `123456789012345678` | Prefer no | Personal identifier |
| `<DISCORD_SERVER_ID>` | `123456789012345678` | Prefer no | Server identifier |
| `<GATEWAY_PORT>` | `18789` | Yes | Confirm locally |
| `<MC_PORT>` | `3000` | Yes | Confirm locally |

## Never placeholders — always secrets

Do not put these into the repo at all:

- Telegram bot token
- Discord bot token
- OpenClaw gateway token
- Model provider API keys / OAuth refresh material
- Dashboard passwords
- Email provider API keys

Store secrets in a password manager and in OpenClaw’s local secret storage only.
