# Intel iMac OpenClaw Install

> **How these notes were made:** written from a real install, using OpenClaw’s persistent memory plus my own notes. Some of this was drafted with OpenClaw + Grok 4.5. **It may not be fully accurate.** Treat it as a practical walkthrough that will be corrected as I keep using and testing it — not as perfect documentation.

Private working title for a **generic, shareable** OpenClaw setup guide aimed at less-technical users on **Intel iMacs**.

> **Repo name is temporary.** Change it to something more searchable before making the repository public.

| Field | Value |
|---|---|
| **Status** | Private draft |
| **Audience** | People setting up OpenClaw on an Intel iMac without deep Terminal experience |
| **Primary guide** | [`docs/INTEL_IMAC_OPENCLAW_SETUP_GUIDE.md`](docs/INTEL_IMAC_OPENCLAW_SETUP_GUIDE.md) |
| **Tested on** | 2015 Intel iMac, Quad Core, 32 GB RAM — see disclaimer below |

## Disclaimer — testing status

**This guide has only been tested on a 2015 Intel iMac, Quad Core, with 32 GB RAM.**

It has **not** been broadly validated across:

- every Intel iMac year/config
- Apple silicon Macs
- every macOS version OpenClaw may support
- every network or corporate-managed Mac environment

Commands, package versions, model names, and OpenClaw CLI flags change over time. Treat this as a practical walkthrough, not a vendor warranty.

If you are on a different machine and something fails, stop at that stage and capture the error before continuing.

## What you get

A staged install path for:

1. Mac prerequisites (Command Line Tools; Homebrew/Node via installer or rescue path)
2. OpenClaw local gateway
3. Workspace files used as **persistent memory**
4. Optional Obsidian as a browser for that memory
5. Telegram bot access
6. Discord bot access
7. Optional Ollama (embeddings / simple local failover when hardware allows)
8. Optional local operator dashboard (Mission Control)
9. Cost-control notes so background jobs do not surprise you

## What this is not

- Not an official OpenClaw manual
- Not a hardened multi-user production deployment guide
- Not a guide that requires OpenCore Legacy Patcher
- Not a guide that requires an Apple ID (local Mac account is enough; Apple ID is optional later)
- Not a dump of anyone’s private tokens, server IDs, or personal notes

## Placeholders

The guide uses placeholders such as:

| Placeholder | Meaning |
|---|---|
| `<YOUR_NAME>` | The human user’s display name |
| `<ASSISTANT_NAME>` | What you call the agent |
| `<MAC_USERNAME>` | macOS short account name |
| `<WORKSPACE_PATH>` | Folder OpenClaw should treat as home |
| `<TELEGRAM_USER_ID>` | Numeric Telegram allowlist id |
| `<DISCORD_USER_ID>` | Numeric Discord allowlist id |

Replace placeholders with your own values. Do **not** commit real tokens.

## Quick start

1. Read the disclaimer above.
2. Open [`docs/INTEL_IMAC_OPENCLAW_SETUP_GUIDE.md`](docs/INTEL_IMAC_OPENCLAW_SETUP_GUIDE.md).
3. Work **one stage at a time**.
4. Do not skip verification checkpoints.
5. Keep secrets in a password manager, never in git.

## Security

- Keep the OpenClaw gateway on **localhost / loopback** unless you later harden remote access on purpose.
- Allowlist only your own Telegram/Discord IDs at first.
- If a token is pasted into chat, rotate it.
- This repository should stay **private** until reviewed for public release and renamed.

## Licence

See [`LICENSE`](LICENSE). Documentation is provided as-is, without warranty.

## Maintainer notes

- Working title / repo slug: `intel-imac-openclaw-install`
- Rename before public launch to improve discoverability
- Keep this repo free of live credentials and machine-local private history
