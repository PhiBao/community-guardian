# Community Guardian

**An autonomous community manager that protects a creator's income and keeps their Discord community welcoming — without the false positives of keyword bots.**

A persistent AI agent ("Mind") on the Minds by Animoca Brands platform, built for creators who run Discord communities (streamers, YouTubers, musicians, makers).

**Track:** Creative Minds Jam #1 — Track C: Moderation & Community Assistance

---

## The problem (why this matters to creators)

Every day, creator Discords get hit with the same scams: fake "free Nitro" giveaways, phishing links dressed as merch drops, bots promising "mod payouts." Each one costs the creator in two ways at once — **real money** (audience members get phished) and **audience trust** (a community that gets scammed stops trusting the creator).

The creator's options today are both bad:

- **No moderation** → scams and spam run free while they sleep, stream, or make content.
- **Keyword-bot moderation** (MEE6, Wick, AutoMod) → the bot "catches" scams but also mutes and warns loyal regulars for saying "hate" or joking about scams. It **rage-baits the fans who keep the community alive.**

**The job:** *"Keep my community safe from scams without making me babysit chat — and without enraging my regulars the way keyword bots do."*

## The product

Community Guardian is a persistent Mind that:

1. **Learns the server** — rules, channel purposes, the owner's norms, and who the trusted regulars are.
2. **Monitors** — watches configured channels and *reads what people actually say* (Message Content intent).
3. **Understands context before acting** — evaluates each message against conversation context (replies, sarcasm, in-jokes), member history, and server norms. Never acts on a single keyword.
4. **Acts on a reversible-first escalation ladder:**
   - Strike 1: private DM warning (names what and why)
   - Strike 2: 10-minute timeout
   - Strike 3+: 1–24h timeout by severity
   - Critical (scam floods, doxxing, hate): immediate timeout + steward alert
   - NEVER bans, deletes, or acts irreversibly without the steward
5. **Remembers and improves** — keeps member histories and an incident log; escalates repeat offenders automatically; learns from every steward override.

### Why it wins where keyword bots lose

| Signal | Keyword bot (MEE6/Wick) | Community Guardian |
|---|---|---|
| `"i hate this update lmao"` (regular joking) | **Mutes a loyal fan** | Recognizes the regular + context → **does nothing** |
| `"free nitro here totally-not-a-scam.example.com"` (scam) | Flags keyword / link | Sees the scam pattern → **DM warning, then timeout on repeat** |
| Same scammer returns next day | No memory | **Remembers the strike → escalates automatically** |
| Owner says "that was a test" | Static | **Clears strikes, logs the lesson, changes behavior** |

The second and third rows are the ones that cost creators money. The first and fourth are the ones that keep regulars loyal.

## What's in this repo

| Path | Contents |
|---|---|
| `docs/TECHNICAL.md` | Architecture, threat model, the Mind's memory model, and escalation ladder |
| `docs/DEMO.md` | The four live demo scenes and how to reproduce them |
| `manifest.md` | The skill manifest (paste-to-Mind artifact) used to create the Guardian |
| `LICENSE` | MIT |

## How it works (2-minute overview)

Community Guardian runs as a persistent Mind on the Minds platform. The owner installs their own Discord bot (by sending its token to the Mind in chat), and the Mind calls the Discord REST API directly — reading channels, messages, members, and roles, and taking reversible moderation actions (DM warnings, timeouts, role changes) through its own HTTP execution. The Mind never exposes the token; it uses it as a private credential.

The demo (`docs/DEMO.md`) shows four live scenes on a real Discord server:

1. **Scam burst** → detected, reversible timeout, calm public note
2. **In-joke** (`"i hate this update so much lmao"`) → correctly ignored — keyword-bot bait
3. **Repeat scammer** → Strike 1 DM, then Strike 2 10-minute timeout on repeat
4. **Steward override** → strikes cleared, behavior changed permanently

Beyond the live loop, the Guardian can email the steward a digest of incidents and recommended actions — the "check your phone, here's what your community manager did while you slept" moment (see `docs/TECHNICAL.md` § Operational notes).

## Getting started

1. Create a Mind on [build.hellominds.ai](https://build.hellominds.ai) (or via CLI).
2. Paste `manifest.md` to the Mind; reply "Build it".
3. Create a Discord bot in the [Developer Portal](https://discord.com/developers/applications) with `GUILD_MEMBERS` + `Message Content` intents enabled.
4. Send the bot token to the Mind in chat (it stores it as a private tenet; never echoed).
5. Tell the Mind your server id and channel purposes.

See `docs/TECHNICAL.md` for the full architecture and security model.

## Security

- The Discord bot token is stored as a Mind-held tenet and never echoed or logged. See `docs/TECHNICAL.md` § Security for the full threat model.
- Reversible-first moderation: no bans, deletions, or irreversible actions without explicit steward approval per incident.

## Built on

- [Minds by Animoca Brands](https://build.hellominds.ai) — persistent AI agent platform
- Discord REST API v10
- RFC 2606 reserved test domains for safe demo scenes

## License

MIT — see `LICENSE`.
