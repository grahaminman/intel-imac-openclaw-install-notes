# OpenClaw Setup Guide — Intel iMac (Generic)

**Audience:** someone setting up OpenClaw who is not deeply technical  
**Goal:** a working local OpenClaw “home base” with file-based persistent memory, Telegram, and Discord  
**Style:** complete, careful, low drama — stop and verify at each stage

| Field | Value |
|---|---|
| **Status** | Generic shareable draft |
| **Hardware focus** | Intel iMac class machines |
| **OS focus** | Official Apple macOS on a supported Intel iMac preferred; OpenCore possible but not taught here |
| **Repo working title** | `intel-imac-openclaw-install` (rename before public launch) |

---

## Disclaimer — please read

### Testing status

**This guide has only been tested on a 2015 Intel iMac with 32 GB RAM.**

It may work on other Intel iMacs and nearby macOS versions, but that is **not guaranteed**.

Not validated as a general matrix across:

- every Intel iMac year and GPU/CPU config
- Apple silicon Macs
- every macOS version
- managed/school/work Macs with locked admin rights
- every future OpenClaw release

OpenClaw CLI commands, model names, and package versions change. If a command in this guide differs from current upstream docs, prefer the current upstream behaviour and treat this as a practical sequence.

### No warranty

This documentation is provided as-is. You are responsible for your own backups, accounts, spend/token usage, and security choices.

### Security rule

Never paste live bot tokens, API keys, gateway tokens, or passwords into Discord, email, screenshots, git commits, or shared copies of notes. Keep secrets in a password manager.

---

## Placeholders used in this guide

Replace these with your own values. Explanatory notes are included where a value is easy to get wrong.

| Placeholder | Replace with | Notes |
|---|---|---|
| `<YOUR_NAME>` | Your name or preferred address | Used in `USER.md` so the assistant knows who you are |
| `<ASSISTANT_NAME>` | The name you give your agent | Example only: `Atlas`, `Nova`, `Helper` — pick yours |
| `<ASSISTANT_EMOJI>` | Optional emoji | Example: `🔍` |
| `<MAC_USERNAME>` | macOS account short name | Finder path is often `/Users/<MAC_USERNAME>` |
| `<WORKSPACE_PATH>` | Folder OpenClaw should use as home | Use: `/Users/<MAC_USERNAME>/OpenClaw/workspace` |
| `<TIMEZONE>` | Your IANA timezone | Examples: `Europe/London`, `America/New_York` |
| `<DEFAULT_MODEL>` | Your chosen cloud model id | Example shape: `xai/grok-...` — use whatever `openclaw models status` currently lists |
| `<TELEGRAM_BOT_USERNAME>` | Bot username from BotFather | Public username is fine to record; **token is secret** |
| `<TELEGRAM_USER_ID>` | Your numeric Telegram user id | Needed for DM allowlisting |
| `<DISCORD_BOT_NAME>` | Discord bot display/name | Not the token |
| `<DISCORD_USER_ID>` | Your numeric Discord user id | Needed for allowlisting |
| `<DISCORD_SERVER_ID>` | Your private server id | Developer mode → copy server id |
| `<GATEWAY_PORT>` | Local OpenClaw gateway port | Common default shape: `18789` — confirm on your install |
| `<MC_PORT>` | Optional Mission Control port | Common example: `3000` |

**Never commit real values for tokens.** IDs are less sensitive than tokens, but this guide still treats personal IDs as private to you.

---

## 0. What you are building

When finished, you should have:

1. **OpenClaw Gateway** running in the background on this Mac  
2. A **workspace folder** of Markdown files that is OpenClaw’s real long-term memory  
3. Chat access via **Telegram** and/or **Discord**  
4. Model access via a cloud provider (this guide uses **xAI / Grok** as the example)  
5. **Optional:** **Obsidian** opened on that same workspace folder (nicer browsing for *you*; not required for memory to work)  
6. **Optional / recommended when the Mac can handle it:** **Ollama** for local embeddings and a simple local model fallback  
7. **Optional:** a local operator dashboard such as **Mission Control**

### Mental model

```text
You (Telegram / Discord / browser)
        │
        ▼
 OpenClaw Gateway  (always-on local service)
        │
        ├── reads workspace memory files  (this is persistent memory)
        ├── talks to your cloud model provider
        └── optional local tools (Ollama, scripts)
```

**Important:** the AI does not magically “remember.”  
OpenClaw memory is **plain files in the workspace** (`MEMORY.md`, `memory/…`, and related notes). If something matters, write it down.

**Obsidian is optional.** It does not create OpenClaw memory. It is a free app that can open the same folder so *you* can browse, search, and link notes more comfortably. OpenClaw works without it.

---

## 1. Read this before you start

### 1.1 Honest expectations

- Budget **half a day to a full day**, not 20 minutes.
- Most pain is front-loaded: Apple developer tools, Homebrew, Node, accounts.
- Once the foundation works, Telegram/Discord/memory are much easier.
- If a step fails, **stop**. Do not stack five half-broken installs.
- **Troubleshooting help during install:** on the tested setup, [Grok.com](https://grok.com) was used to debug install errors (paste the exact command + full error text; never paste tokens/passwords). Once OpenClaw itself is working, you can switch to asking your local OpenClaw assistant to help troubleshoot the remaining stages.

### 1.2 What this guide deliberately avoids

- **OpenCore Legacy Patcher as a requirement for this guide**  
  The install this guide is based on **can use OpenCore and work well**, but if you are not experienced at setting up machines it is **simpler to install without it** (official macOS on a supported Intel iMac).  
  If you **do** use OpenCore, expect **extra steps** — especially around **storage volumes**, boot/mount reliability, and keeping large workspace/model disks available after reboot. This guide does not walk through OpenCore install or storage patching.  
  Do **not** install OpenCore just because you are following this OpenClaw guide.
- Exposing OpenClaw to the public internet on day one
- Fancy multi-agent factories on day one
- Paid extras you do not need yet

### 1.3 Mac login: local account is enough

You do **not** need an Apple ID to follow this guide.

A normal **local macOS user account** is the intended baseline:

- full control of the machine
- no Apple Account sign-in required for OpenClaw, Homebrew, Node, Telegram, or Discord setup
- you can add an Apple ID later if you ever want App Store, iCloud, or similar services

If macOS offers Apple Account / iCloud during first boot, you can skip those and continue with a local account only.

### 1.4 Accounts that are actually useful for this guide

Create these only as needed, and store credentials privately:

| Account | Required for this guide? | Why |
|---|---|---|
| Local Mac user account | Yes | day-to-day admin on the iMac |
| xAI / Grok account | Yes, for the cloud-model example below | chat model provider |
| Telegram account | If you want Telegram access | talk to the bot |
| Discord account + a private server you control | If you want Discord access | channel chat with the bot |
| Apple ID | **No / optional later** | only if you later want App Store, iCloud, or Apple’s developer download site |
| Dedicated agent email provider | Optional later | skip on day one if overwhelmed |

Also decide:

- `<MAC_USERNAME>` (local account short name)

### 1.5 Paths used in this guide

Use one simple layout on the Mac's main user volume:

```text
/Users/<MAC_USERNAME>/OpenClaw/workspace
/Users/<MAC_USERNAME>/.openclaw/     (created automatically)
```

In shorthand many commands use:

```text
~/OpenClaw/workspace
~/.openclaw/
```

Set:

```text
<WORKSPACE_PATH> = /Users/<MAC_USERNAME>/OpenClaw/workspace
```

### 1.6 How to use Terminal safely

1. Open **Terminal** (Spotlight: `Cmd + Space`, type `Terminal`, Enter).
2. Copy **one command at a time**.
3. Paste with `Cmd + V`, then press **Return**.
4. If macOS asks for your password, type it (nothing will appear as you type — that is normal) and press Return. If you mistype it, the Mac will usually say the password was incorrect and let you try again. For many Terminal admin prompts (`sudo`), you typically get about **3 tries** before that command gives up — just run the same command again and re-enter the password carefully.
5. If something goes wrong and Terminal shows an error (often in red, or text starting with `error:` / `fatal:`):
   - **Stop.** Do not keep running more install steps.
   - Select **all the text from the command you typed down through the full error message** (not just one line).
   - Copy it (`Cmd + C`).
   - Paste the whole thing into an AI chat and ask for help fixing that exact step.
   - Before OpenClaw is working, use [Grok.com](https://grok.com) (that is what was used on the tested install).
   - After OpenClaw is working, you can ask your OpenClaw assistant instead.
   - Never include passwords, bot tokens, or API keys in what you paste.

---

## 2. Stage checklist (do in order)

| Stage | Name | Required? | Done when… |
|---|---|---|---|
| A | Mac basics + updates | Yes | macOS updated, disk space OK |
| B | Xcode Command Line Tools | Yes | `xcode-select -p` works |
| C | Homebrew | Yes | `brew --version` works |
| D | Node.js 22+ | Yes | `node -v` shows v22 or newer |
| E | OpenClaw install + gateway | Yes | gateway status is running |
| F | Workspace + identity files | Yes | core `.md` files exist |
| G | Cloud model login | Yes | short chat reply works |
| H | Persistent memory (workspace files) | Yes | memory write/read test passes |
| H-opt | Obsidian (browse the same folder) | Optional | vault opens on workspace if you want it |
| I | Telegram | Recommended | DM reply works |
| J | Discord | Recommended | channel reply works |
| K | Ollama (local models / fallback) | Recommended when hardware allows | Ollama runs, or you consciously skipped it |
| L | Cost control | Recommended | no expensive surprise loop |
| M | Mission Control | Optional | local browser UI works |
| N | Extras | Optional later | skip until core is solid |

---

## 3. Stage A — Prepare the Intel iMac

### A1. Confirm hardware/OS basics

Apple menu → **About This Mac**

Record for your notes:

- iMac model/year
- memory (RAM)
- macOS name/version

This guide’s only formal test claim remains: **2015 Intel iMac / 32 GB RAM**.

### A2. Prefer stock macOS

If the machine only runs a modern OS because of unofficial patching tools, this guide may not match your reality. Prefer an official OS path for fewer surprises.

### A3. Update macOS

System Settings → **General** → **Software Update**  
Install updates, restart if asked, then continue.

### A4. Check free disk space

Aim for **40 GB free minimum**, more if you will store local AI models.

### A5. Keep the Mac awake while installing

Plug into power. Optionally reduce sleep while you work so long installs are not interrupted.

### A6. Create your folders

```bash
mkdir -p "$HOME/OpenClaw/workspace"
mkdir -p "$HOME/OpenClaw/projects"
ls "$HOME/OpenClaw"
```

Everything for this guide lives under your home folder on the main Mac volume.

### A7. Checkpoint

- [ ] macOS updated  
- [ ] Enough free space  
- [ ] `<WORKSPACE_PATH>` exists  

---

## 4. Stage B — Apple Command Line Tools

Homebrew and many dev tools need Apple’s **Command Line Tools**.

### B1. Try the normal installer first

```bash
xcode-select --install
```

- A macOS window should appear  
- Click **Install**  
- Agree to the license  
- Wait (can take a while)

### B2. If the popup never appears, fails, or loops

This is a common obstacle. Stay on the local-account path if you can.

**Try these first (usually no Apple ID):**

1. Reboot and run `xcode-select --install` again
2. Confirm the Mac has working internet
3. Run:

```bash
sudo xcode-select --reset
xcode-select --install
```

**Manual package download (optional fallback only):**

Apple’s browser download site (`developer.apple.com/download/all`) typically asks for an **Apple ID**.  
You do **not** need that for the normal path above.

Use the manual site only if the popup path is completely stuck **and** you are willing to create/use an Apple ID later:

1. Open Safari and sign in at:  
   <https://developer.apple.com/download/all/>
2. Search for: **Command Line Tools for Xcode**
3. Download a package compatible with **your macOS version**
4. Open the `.dmg` → run the `.pkg` → finish the installer
5. Restart Terminal

If you are deliberately keeping the Mac free of Apple ID / Apple Account ties, stop at the local `xcode-select --install` path and get help with that error rather than creating an Apple ID just for this step.

### B3. Verify

```bash
xcode-select -p
git --version
clang --version
```

Expected:

- a developer tools path such as `/Library/Developer/CommandLineTools`
- `git` and `clang` print versions (not “command not found”)

### B4. Common failures

| Symptom | What to do |
|---|---|
| `xcode-select: error: no developer tools...` | rerun install or manual `.pkg` route |
| Installer stuck at “Finding software” | reboot, retry; if still stuck use manual download |
| License not accepted | run: `sudo xcodebuild -license accept` then retry |
| Wrong/old tools after OS change | `sudo xcode-select --reset` then reinstall CLT |

### B5. Checkpoint

- [ ] `xcode-select -p` works  
- [ ] `git --version` works  

**Do not install Homebrew until this stage is green.**

---

## 5. Stage C — Homebrew

Homebrew is a package manager. On Intel Macs it commonly lives under `/usr/local`.

### C1. Official install

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

- Read what it says before confirming  
- Enter your Mac password if prompted  
- Let it finish fully

### C2. If the one-liner fails

Try in order:

1. Confirm Command Line Tools again (Stage B)
2. Retry in a new Terminal window after reboot
3. Check network access in Safari
4. Follow the current official manual instructions:  
   <https://docs.brew.sh/Installation>  
   (use official docs only)

### C3. Put `brew` on your PATH (Intel Mac)

```bash
which brew || ls /usr/local/bin/brew
brew --version
```

If `brew` is installed but `command not found`:

```bash
echo 'eval "$(/usr/local/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/usr/local/bin/brew shellenv)"
brew --version
```

Close Terminal, reopen it, run `brew --version` again.

> **Note:** Apple silicon Homebrew paths differ (`/opt/homebrew`). This guide is Intel-first.

### C4. First Homebrew health pass

```bash
brew update
brew doctor
```

Warnings can be normal. A broken `brew --version` is not.

### C5. Older Intel iMac notes

- First installs can be slow. Leave them alone.
- Prefer prebuilt packages when Homebrew offers them.
- Do not install OpenCore to “fix” Homebrew.

### C6. Checkpoint

- [ ] `brew --version` works in a **new** Terminal window  
- [ ] `brew update` completes  

---

## 6. Stage D — Node.js

OpenClaw needs a current Node. Aim for **Node 22 or newer**.

### D1. Install Node via Homebrew

```bash
brew install node
node -v
npm -v
```

### D2. If Homebrew Node is unsuitable

Use the official macOS installer from <https://nodejs.org/>  
Prefer current **LTS** unless you know you need another track.

Reopen Terminal and recheck `node -v`.

### D3. Checkpoint

- [ ] `node -v` ≥ 22  
- [ ] `npm -v` works  

---

## 7. Stage E — Install OpenClaw + start the gateway

### E1. Install the OpenClaw CLI

```bash
npm install -g openclaw
openclaw --version
```

If `openclaw: command not found` after a successful install:

```bash
npm config get prefix
ls "$(npm config get prefix)/bin/openclaw"
export PATH="$(npm config get prefix)/bin:$PATH"
echo 'export PATH="$(npm config get prefix)/bin:$PATH"' >> ~/.zprofile
```

Reopen Terminal and try `openclaw --version` again.

### E2. Onboard / setup

```bash
openclaw onboard
```

If that command is unavailable in your version:

```bash
openclaw setup
openclaw configure
```

Prefer:

- **Local gateway**
- **Loopback / localhost only**
- Token auth enabled
- Workspace path = `<WORKSPACE_PATH>`

### E3. Start and check the gateway

```bash
openclaw gateway status
openclaw doctor
openclaw status
```

Healthy signs:

- gateway **running**
- local probe OK on something like `ws://127.0.0.1:<GATEWAY_PORT>`
- no critical doctor failures

Open the control UI:

```bash
openclaw dashboard
```

Prefer `http://127.0.0.1:<GATEWAY_PORT>` style URLs over fancy hostnames.

### E4. If the gateway will not start

```bash
openclaw doctor
openclaw doctor --fix
openclaw logs --follow
```

Stop and fix before continuing.

### E5. Checkpoint

- [ ] `openclaw --version` works  
- [ ] gateway running  
- [ ] dashboard/control UI opens locally  

---

## 8. Stage F — Workspace files (your “brain on disk”)

### F1. Confirm workspace path

```bash
ls "<WORKSPACE_PATH>"
openclaw status
```

If OpenClaw is pointed somewhere else, use onboarding/configure/docs for your version to point it at `<WORKSPACE_PATH>`.

### F2. Create the minimum identity/memory files

```bash
cd "<WORKSPACE_PATH>"
mkdir -p memory second-brain templates directives skills
touch AGENTS.md SOUL.md IDENTITY.md USER.md MEMORY.md TOOLS.md TODO.md HEARTBEAT.md Home.md
```

| File | Purpose |
|---|---|
| `AGENTS.md` | operating rules for the assistant |
| `SOUL.md` | persona / tone |
| `IDENTITY.md` | short name + vibe |
| `USER.md` | about you |
| `MEMORY.md` | curated long-term memory |
| `TOOLS.md` | machine-local notes (paths, IDs) |
| `TODO.md` | ideas and open loops |
| `memory/` | daily raw notes |

### F3. Starter content with placeholders

**`USER.md`**

```markdown
# USER.md

- Name: <YOUR_NAME>
- Timezone: <TIMEZONE>
- What to call them: <YOUR_NAME>
- Notes: Learning OpenClaw; wants a reliable assistant with memory.
```

**`IDENTITY.md`**

```markdown
# IDENTITY.md

- Name: <ASSISTANT_NAME>
- Emoji: <ASSISTANT_EMOJI>
- Vibe: Practical, clear, helpful co-pilot
```

**`SOUL.md`** (short version is fine)

```markdown
# SOUL.md

I am a capable local assistant named <ASSISTANT_NAME>.
I write important facts into workspace files so they survive restarts.
I ask before sending emails, public posts, or anything external I am unsure about.
```

**`MEMORY.md`**

```markdown
# MEMORY.md

## Who we are
- Assistant: <ASSISTANT_NAME>
- Human: <YOUR_NAME>

## Active priorities
1. Finish OpenClaw setup
2. Keep useful notes in this workspace

## Standing preferences
- Write important decisions into files
- Private stays private
```

**`TOOLS.md`**

```markdown
# TOOLS.md

## Paths
- Workspace: <WORKSPACE_PATH>
- OpenClaw state: ~/.openclaw/

## Accounts to fill in later
- Telegram bot username: <TELEGRAM_BOT_USERNAME>
- Discord bot name: <DISCORD_BOT_NAME>
- Default model: <DEFAULT_MODEL>
```

**`AGENTS.md`** (short version)

```markdown
# AGENTS.md

## Memory
- Write important decisions, preferences, and setup changes into files
- Daily raw notes go in memory/YYYY-MM-DD.md
- Long-term distilled facts go in MEMORY.md
- Machine-specific paths/IDs go in TOOLS.md
- Do not rely on chat history alone

## Safety
- Ask before external actions (email, public posts)
- Never put secrets into git or shared chats
```

**First daily note:**

```bash
DATE=$(date +%Y-%m-%d)
cat > "<WORKSPACE_PATH>/memory/$DATE.md" <<EOF
# $DATE

## Session Log
- Started OpenClaw setup on Intel iMac

## Decisions Made
- Using workspace files for persistent memory

## Things to Remember
- Do stages in order; verify before moving on
EOF
```

### F4. Checkpoint

- [ ] workspace path is intentional and stable  
- [ ] core markdown files exist with your placeholders replaced  
- [ ] `memory/YYYY-MM-DD.md` exists for today  

---

## 9. Stage G — Connect a cloud model (xAI / Grok example)

This guide uses xAI as a concrete example because it is a common OpenClaw setup path. Other providers may work; follow current OpenClaw provider docs if you choose differently.

### G1. Login

```bash
openclaw models auth login --provider xai
openclaw models status
```

Complete the browser/OAuth flow.

Set a default only after you see valid model ids:

```bash
openclaw models set <DEFAULT_MODEL>
```

If unsure, use whatever current model id `openclaw models status` recommends/lists.

### G2. First real reply test

In the OpenClaw dashboard/webchat send:

```text
Reply with exactly: setup-ok
```

### G3. Known obstacle: OAuth can expire

Symptoms:

- doctor warns about expiring auth  
- model calls fail after earlier success  

Fix:

```bash
openclaw models auth login --provider xai
# if stuck:
openclaw models auth login --provider xai --force
```

### G4. Checkpoint

- [ ] models auth looks healthy  
- [ ] one successful chat reply  

---

## 10. Stage H — Persistent memory (workspace files; Obsidian optional)

### H1. How memory actually works

OpenClaw’s persistent memory is **built in**: Markdown files in the workspace folder. That is the source of truth.

Typical pattern:

1. You (or the assistant) write notes into workspace files  
2. Next session, OpenClaw loads key bootstrap files (for example `AGENTS.md`, `SOUL.md`, `USER.md`, `MEMORY.md`)  
3. Daily notes live under `memory/YYYY-MM-DD.md`  
4. Optional search tools can help find older notes later  

If it is only said in chat and never written down, it may be forgotten.

You can open those files with **any** editor (TextEdit, VS Code, Finder preview). That is enough for a working system.

### H2. Recommended starter folder structure

```text
<WORKSPACE_PATH>/
  AGENTS.md
  SOUL.md
  IDENTITY.md
  USER.md
  MEMORY.md
  TOOLS.md
  Home.md
  memory/
    YYYY-MM-DD.md
  second-brain/
    README.md
    concepts/
    projects/
    journal/
  templates/
  directives/
  skills/
```

Simple `Home.md`:

```markdown
# Home

## Daily
- Notes under `memory/`

## Long-term
- See `MEMORY.md`

## Projects
- Add notes under `second-brain/` as you go
```

### H3. Memory test (required)

In OpenClaw chat:

```text
Please add to MEMORY.md that my test phrase is "blue kettle".
Then tell me where you wrote it.
```

Start a fresh session/surface and ask:

```text
What is my test phrase?
```

If it answers from the file, persistent memory is working. **You do not need Obsidian for this test.**

### H4. Optional — Obsidian (human-friendly browser for the same files)

Install Obsidian only if you want a nicer way to browse and link notes.

**What Obsidian is good for**

- Comfortable reading/editing of the workspace  
- Graph view and `[[wiki links]]` if you use them  
- Daily-note plugins and other note-taking extras  

**What Obsidian is not**

- Not required for OpenClaw to remember anything  
- Not a second memory database  
- Not a substitute for writing facts into workspace files  

If you want it:

1. Download from the official site: <https://obsidian.md>  
2. Install the app  
3. **Open folder as vault** → choose `<WORKSPACE_PATH>`

One workspace = one vault. Avoid creating a second unrelated vault for the same notes.

Optional CLI check (skip if unavailable):

```bash
obsidian version
```

Later optional extras (plugins, extra search tools) are nice-to-have only.

### H5. Checkpoint

**Required**

- [ ] `MEMORY.md` and `memory/` exist and are in use  
- [ ] memory write/read test passed  

**Optional**

- [ ] Obsidian opens `<WORKSPACE_PATH>` if you chose to install it  

---

## 11. Stage I — Telegram bot

### I1. Create the bot

1. In Telegram, talk to **@BotFather**
2. `/newbot`
3. Choose a name and username → record `<TELEGRAM_BOT_USERNAME>`
4. Copy the **bot token** into your password manager only

### I2. Find your Telegram user id

Obtain your numeric id and store it as `<TELEGRAM_USER_ID>`.

### I3. Configure OpenClaw

Use onboarding/config UI if available, or supported OpenClaw config/secrets commands for your version.

Target shape:

- Telegram enabled  
- **DM policy = allowlist**  
- allowlist includes only `<TELEGRAM_USER_ID>`  
- bot token stored in OpenClaw secrets storage  

Then:

```bash
openclaw channels status --probe
openclaw gateway status
```

Restart gateway if the CLI tells you to.

### I4. Test

1. DM your bot on Telegram  
2. Send `ping` or `Say hello`  
3. Confirm a reply  

### I5. Common Telegram failures

| Problem | Fix |
|---|---|
| Bot exists but never replies | token wrong; gateway not running; channel disabled |
| Replies to strangers | tighten allowlist immediately |
| Works in webchat, not Telegram | channel config/probe; restart gateway |
| Token leaked in chat | revoke/regenerate at BotFather |

### I6. Checkpoint

- [ ] DM reply works  
- [ ] allowlist locked to you  
- [ ] bot username recorded in `TOOLS.md` (**not** the token)  

---

## 12. Stage J — Discord bot

Discord has more moving parts than Telegram. Go slowly.

### J1. Create the application

1. Open <https://discord.com/developers/applications>
2. **New Application**
3. Create a **Bot** → record `<DISCORD_BOT_NAME>`
4. Copy the bot token to your password manager only
5. Under **Privileged Gateway Intents**, enable at least:
   - **Message Content Intent** (required for normal channel text)
   - **Server Members Intent** (recommended)
6. Save changes

### J2. Invite the bot to your private server

OAuth2 → URL Generator:

- Scopes: `bot` (and `applications.commands` if offered/needed)
- Permissions minimum:
  - View Channels
  - Send Messages
  - Read Message History
  - Embed Links
  - Attach Files
  - Add Reactions

Invite only into a server **you control**. Record `<DISCORD_SERVER_ID>`.

### J3. Create simple channels

Suggested starter channels:

- `#general`
- `#set-up`
- `#research`

### J4. Configure OpenClaw Discord

Target shape:

- Discord enabled  
- DM policy allowlist to `<DISCORD_USER_ID>`  
- guild/server allowlisted with `<DISCORD_SERVER_ID>`  
- your user allowed  
- for a private personal server, disabling require-mention can be convenient  
- bot token in secrets storage  

Then:

```bash
openclaw channels status --probe
```

### J5. Test in order

1. In the server channel: `@<DISCORD_BOT_NAME> ping`  
2. Then plain: `ping`  
3. Confirm replies in the **channel itself**

### J6. Common Discord obstacles

#### Message Content Intent off
**Symptom:** bot online, ignores normal channel messages  
**Fix:** enable Message Content Intent → save → restart gateway

#### Allowlist too strict / empty
**Symptom:** silence  
**Fix:** ensure server id and user id are actually allowlisted

#### Missing channel permissions
**Symptom:** bot present but cannot speak  
**Fix:** channel permissions / bot role hierarchy

#### Checking the wrong surface
Discord channel talks may live in separate sessions from webchat.  
Judge success in the Discord channel.

#### Status text like content-limited
On smaller bots this can still be normal if the intent is enabled and channel replies work. Prove it with a ping test.

### J7. Record non-secret IDs in `TOOLS.md`

```markdown
## Discord
- Server id: <DISCORD_SERVER_ID>
- Bot name: <DISCORD_BOT_NAME>
- My user id: <DISCORD_USER_ID>
- Key channels: #general, #set-up
```

### J8. Checkpoint

- [ ] intents enabled  
- [ ] bot in server with send permissions  
- [ ] mention ping works  
- [ ] plain ping works if you disabled require-mention  

---

## 13. Stage K — Ollama (recommended when the Mac can handle it)

You can run a solid OpenClaw setup **without** Ollama. Cloud chat + workspace files already work.

Ollama is still worth understanding, because it unlocks useful **local** capabilities when the machine is strong enough.

### K0. Why Ollama is valuable (and when to skip it)

**Good reasons to install it**

1. **Better memory search** — a small local **embeddings** model helps OpenClaw find older notes by meaning, not only exact words  
2. **Failover / degraded mode** — if your cloud AI login, API, or network fails, a small local chat model can still let the assistant reply in a limited way  
3. **Cheap simple jobs** — light local tasks (some heartbeat-style checks, short summaries, routine classification) can use a small local model so you are not always spending cloud tokens  

**When it may not be possible or not a good idea**

- Low free disk (local models can be hundreds of MB to many GB each)  
- Older / low-RAM Intel iMacs that become unusably slow when a chat model is loaded  
- You only want the smallest working cloud setup first  
- You are still fighting gateway / auth / Telegram / Discord issues — finish those before adding local models  

**Rough hardware guidance (practical, not a hard rule)**

| Machine class | Sensible Ollama plan |
|---|---|
| ~8 GB RAM | Often skip local chat models; embeddings-only may still be tight — prioritise cloud + files |
| ~16 GB RAM | Embeddings usually fine; keep any local chat model **small** |
| ~32 GB+ RAM (like the tested install) | Embeddings + a small local chat fallback is realistic |

Always prefer a **small** local chat model for fallback. Large “smart” local models on older Intel Macs are often slow and frustrating.

If you skip Ollama now, that is a valid choice. Come back later when the core system is stable.

### K1. Install Ollama (if proceeding)

Options:

1. Official macOS app from <https://ollama.com>  
2. or `brew install ollama` if that is current/supported on your machine

Check:

```bash
ollama --version
```

### K2. Model storage (defaults are fine)

Leave Ollama on its default location under your home/main volume.  
You do **not** need a separate `/Volumes/...` models disk for this guide.

### K3. Pull an embeddings model first (best first local win)

```bash
ollama pull nomic-embed-text
ollama list
```

This is usually the highest-value, lowest-drama local piece for memory search.

### K4. Optional — small local chat model for failover / simple jobs

Only after embeddings work, and only if RAM/disk feel comfortable:

```bash
# Example only — pick a currently small model name from Ollama’s model library
ollama pull <SMALL_LOCAL_CHAT_MODEL>
ollama list
```

Then, using current OpenClaw docs/config for your version:

- keep your **cloud model as the normal default**  
- configure the small local model as a **fallback** and/or for light jobs if your OpenClaw version supports model failover / routing  
- do **not** expect a tiny local model to match full cloud quality  

Good uses for a small local model:

- “Cloud is down, can you still answer simply?”  
- short heartbeat-style checks  
- tiny summarise / classify tasks  

Poor uses on older Intel hardware:

- long creative writing as the daily driver  
- huge models that thrash disk and freeze the Mac  

### K5. Checkpoint

If you installed Ollama:

- [ ] `ollama list` works  
- [ ] embeddings model present  
- [ ] optional small chat model only if the Mac stays responsive  
- [ ] cloud model remains your normal default  
- [ ] notes recorded in `TOOLS.md`  

If you skipped Ollama:

- [ ] conscious skip (disk/RAM/priority) written in your notes  
- [ ] core cloud chat + workspace memory still verified  

---

## 14. Stage L — Cost control

Background automation can spend money quietly.

### L1. Disable aggressive heartbeat while learning

Use OpenClaw config/UI for your version to turn off frequent cloud heartbeats, or set the interval off / zero if supported.

Verify:

```bash
openclaw status
```

### L2. Optional: one daily check instead

A safer learning policy:

- one light daily job  
- local timezone `<TIMEZONE>`  
- only notify you if something needs attention  

```bash
openclaw cron list
```

### L3. Checkpoint

- [ ] no surprise high-frequency cloud loop  
- [ ] you understand what is scheduled  

---

## 15. Stage M — Mission Control (optional local dashboard)

Only do this after Telegram or Discord already works.

Upstream project example: [builderz-labs/mission-control](https://github.com/builderz-labs/mission-control)

> Mission Control quality and setup details can change quickly. If this stage fights you, skip it. Core OpenClaw does not depend on it.

### M1. Prerequisites

```bash
node -v    # 22+
npm install -g pnpm
pnpm -v
```

### M2. Install sketch

```bash
git clone --depth 1 https://github.com/builderz-labs/mission-control.git "$HOME/OpenClaw/mission-control"
cd "$HOME/OpenClaw/mission-control"
bash scripts/generate-env.sh .env
```

Manually edit `.env`:

- set strong local auth values  
- do not leave critical auth lines commented  
- point at local OpenClaw gateway `127.0.0.1` and `<GATEWAY_PORT>`  
- use the same gateway token OpenClaw uses  
- prefer state-dir style paths over double-nested home paths  

Example shape:

```env
OPENCLAW_STATE_DIR=/Users/<MAC_USERNAME>/.openclaw
OPENCLAW_HOME=/Users/<MAC_USERNAME>
PORT=<MC_PORT>
```

Then:

```bash
chmod 600 .env
pnpm install
pnpm rebuild better-sqlite3
pnpm build
```

### M3. macOS bind obstacle

On macOS, some apps bind to the machine hostname instead of localhost. Browsers then fail with “can’t connect.”

Mitigation pattern:

```bash
export HOSTNAME=127.0.0.1
export MC_BIND_HOST=127.0.0.1
export PORT=<MC_PORT>
```

Open:

```text
http://127.0.0.1:<MC_PORT>
```

### M4. Verify

```bash
curl -sS "http://127.0.0.1:<MC_PORT>/health"
```

### M5. Checkpoint (if installed)

- [ ] health endpoint OK on loopback  
- [ ] login works  
- [ ] gateway shows online  

---

## 16. Optional later upgrades

Do not block “done” on these:

| Upgrade | What it gives you | When to add |
|---|---|---|
| Voice memo transcription | speech notes become text | after chat surfaces are stable |
| Local long-audio transcription | meeting-length files on-device | if you regularly record long audio |
| Dedicated agent email | assistant-owned inbox | when email identity matters |
| Extra search tooling | better note retrieval | after you have lots of notes |
| Business/newsletter tooling | product workflows | only if needed |

---

## 17. Final “you are done” checklist

### Minimum viable setup

- [ ] Intel iMac on official macOS, no OpenCore required by this guide  
- [ ] Command Line Tools installed  
- [ ] Homebrew works  
- [ ] Node 22+ works  
- [ ] OpenClaw installed  
- [ ] Gateway running on localhost  
- [ ] Workspace files created with your placeholders replaced  
- [ ] Cloud model auth works; one successful chat  
- [ ] Memory write/read test passed (workspace files — Obsidian not required)  
- [ ] Telegram DM works **or** Discord channel works (both is better)  
- [ ] Secrets stored privately  
- [ ] Background jobs not quietly draining money  

### Strong setup

Everything above, plus:

- [ ] Both Telegram and Discord  
- [ ] Daily note habit / `MEMORY.md` habit  
- [ ] Obsidian opened on the workspace if you want visual browsing  
- [ ] Ollama when hardware allows: embeddings + optional small local failover model  
- [ ] Optional dashboard on loopback only  
- [ ] Basic cron/jobs understood  

---

## 18. Day-2 operations

```bash
# Is the system awake?
openclaw status
openclaw gateway status
openclaw channels status --probe
openclaw doctor
openclaw models status

# Refresh model login if replies die
openclaw models auth login --provider xai

# Logs when something is weird
openclaw logs --follow
```

### If the Mac rebooted

1. Wait a minute  
2. Run `openclaw gateway status`  
3. Start/restart the gateway service if needed  
4. Test Telegram or Discord again  

### If free space runs out

OpenClaw workspace files are small at first; local models can use more disk.  
Check Apple menu → About This Mac → Storage, free some space, then retry.

---

## 19. Obstacle playbook

### Homebrew install script failed
1. Fix Command Line Tools first  
2. Reboot  
3. Retry official installer  
4. Only then use official manual install docs  

### `brew` works once, then vanishes in a new Terminal
PATH not set in `~/.zprofile`. Re-do Stage C3.

### `openclaw` command not found after npm install
Node bin directory not on PATH. Re-do Stage E1 path fix.

### Gateway running but no model replies
Re-auth the model provider. Check `openclaw models status`.

### Discord bot is online and mute
Message Content Intent, allowlists, channel permissions — in that order.

### It forgot everything
Either the workspace path changed, or the fact was never written into files.

### Browser can’t open a local dashboard
Force loopback (`127.0.0.1`), confirm the process is listening, avoid fancy hostnames first.

### Someone told you to install OpenCore for this
Not required for this guide. The reference-style install can work with OpenCore, but unsupported OS patching adds complexity (including storage management). Prefer stock macOS on supported hardware unless you already know OpenCore well.

---

## 20. Security baseline

- Keep OpenClaw gateway on **localhost/loopback** unless you later harden remote access on purpose  
- Keep any dashboard on **127.0.0.1** only at first  
- Telegram/Discord allowlists = **you only** at first  
- Tokens live in a password manager / OpenClaw secrets storage  
- Never commit tokens to GitHub  
- If a token is pasted into chat, **rotate it**  
- Be careful what private memory is visible in shared group chats  

Useful commands when available:

```bash
openclaw security audit
openclaw security audit --deep
```

---

## 21. Suggested one-day schedule

### Morning
- Stages A–D (Mac, CLT, Homebrew, Node)
- Stage E (OpenClaw gateway)

### Midday
- Stages F–G (workspace + model auth)
- Stage H (memory write/read test; Obsidian only if you want it)

### Afternoon
- Stage I (Telegram)
- Stage J (Discord)
- Stage L (cost control)

### Only if energy remains
- Stage K (Ollama — embeddings first; small local failover only if the Mac stays happy)
- Stage M (Mission Control)

If you only finish through one chat surface plus memory files, that is already a real system.

---

## 22. Help request template

Before asking for help, capture:

1. Which **stage letter** failed  
2. The exact command you ran  
3. The full error text  
4. Outputs of:

```bash
sw_vers
uname -m
sysctl -n hw.memsize
xcode-select -p
brew --version
node -v
openclaw --version
openclaw gateway status
openclaw doctor
```

Do **not** include tokens or passwords.

**Where to ask:** before OpenClaw works, use [Grok.com](https://grok.com) (or similar) with the template above — that is what was used on the tested install. After OpenClaw is up, ask your OpenClaw assistant in webchat/Telegram/Discord instead.

Optional useful context:

- iMac model/year  
- RAM amount  
- roughly how much free disk you have  

---

## 23. Short version

1. Update macOS  
2. Install Apple Command Line Tools  
3. Install Homebrew  
4. Install Node 22+  
5. Install OpenClaw and start gateway  
6. Create workspace identity/memory files  
7. Login cloud model provider  
8. Prove memory with a write/read test (Obsidian optional)  
9. Connect Telegram  
10. Connect Discord  
11. Turn off expensive background loops  
12. Optionally add Ollama (embeddings + small local failover if hardware allows)  
13. Write everything important into files  

**Prefer no OpenCore unless you already know it. No public exposure. Verify after every stage.**

---

## 24. Maintainer note for this repository

- Working GitHub name: `intel-imac-openclaw-install`  
- Keep the repo **private** until reviewed  
- Rename before public release for searchability  
- Keep examples generic; never commit live tokens or personal server inventories  
- Formal test claim remains: **2015 Intel iMac / 32 GB RAM**

---

_End of guide._
