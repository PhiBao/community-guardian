# Community Guardian — Technical Design

## 1. System overview

Community Guardian is a persistent AI agent ("Mind") running on the Minds by Animoca Brands platform. It acts as an **autonomous community manager** for a solo/indie creator's Discord server — protecting the creator's income and audience trust from scams, while keeping loyal regulars happy (no keyword-bot false positives).

**Creator-economy fit (Track C: Moderation & Community Assistance).** Creator Discords are a prime target for scams that directly hit creator income and trust: fake "free Nitro" giveaways, phishing links disguised as merch drops, "mod payout" bots. A creator cannot watch chat 24/7 — they stream, film, and make content. Keyword-bot moderation is the only existing defense, and it fails on two axes at once: it misses context-aware scams, and it mutes loyal regulars over in-jokes. Community Guardian solves the specific problem of *detecting scams that look like normal chat, without punishing the normal chat.*

```
Discord API v10  ◄──HTTP/HTTPS──►  Community Guardian Mind  ◄──chat──►  Steward
   (channels/messages/members/roles)      (Minds platform)        (Discord/email/CLI)
                                                │
                                                └── memory: server_norms, member_profiles,
                                                     incident_log, escalation_state
```

The Mind interacts with Discord via its own HTTP execution tool (`HTTP_Execute`), using a bot token it holds as a private tenet. It does **not** use a pre-packaged app connector — this was a deliberate architectural decision after the platform's Discord app layer was found to return `401 Unauthorized` on every tool (see §7).

## 2. The Mind's memory model

The Guardian persists four memory blocks, updated every session:

| Block | Contents |
|---|---|
| `server_norms` | Rules, channel purposes, allowed/forbidden topics, the steward's stance on grey areas |
| `member_profiles` | Per member: strike count (0–3), last incident, known positive behaviors, known in-jokes |
| `incident_log` | Timestamp, channel, member, context, action, steward verdict, lesson learned |
| `escalation_state` | Who is in timeout and until when, pending steward approvals |

The memory is what enables **continuity**: a repeat offender is recognized across sessions, and a steward's correction is applied to future judgments.

## 3. Escalation ladder

Reversible-first. Each step is the minimum that keeps the community safe and can be undone.

| Step | Trigger | Action |
|---|---|---|
| Strike 1 | First offense, moderate severity | Private DM warning (specific: what, why) |
| Strike 2 | Repeat offender, or severity warrants | 10-minute timeout (`communication_disabled_until`) |
| Strike 3+ | Repeat after Strike 2 | 1–24h timeout by severity |
| Critical | Scam flood, doxxing, hate, sexual content | Immediate timeout + steward alert within minutes |
| Never | — | Permanent ban, message deletion, >24h timeout without per-incident steward approval |

**What counts as harm (the only things acted on):** spam floods, scam/phishing links, doxxing, hate speech, harassment, sexual content, coordinated disruption.

**What never triggers action alone:** profanity, criticism, disagreement, a single keyword.

## 4. Context awareness (the differentiator)

The Guardian evaluates each message against:
- **Author identity + history** — known regular vs new account; strike count; known in-jokes
- **Conversation context** — is this a reply? sarcasm? an in-joke among regulars?
- **Pattern** — single message vs flood vs coordinated burst
- **Content substance** — does it target a person/group, or is it a communal gripe?
- **Channel fit** — social channel vs audit/reporting channel

Verified in demo scene 2: a message containing "hate" (`"i hate this update so much lmao who greenlit this"`) was correctly left alone because it was playful vent from a known regular — while the same account's scam template (scene 3) was escalated. A keyword bot fails exactly this test.

## 5. Recognition layer

The Guardian learns recognition rules from steward feedback. Example (demo scene 4): after the steward revealed that `.example.com` scam-messages were a staged test, the Guardian recorded that RFC 2606 reserved test domains (`.example.com`, `.test`, `.invalid`, `.localhost`) are overwhelmingly staged content, and now pauses before non-reversible escalation on test-domain signals — while real-domain spam is escalated normally.

This is a **learning loop**: steward override → lesson tenet → changed future behavior.

## 6. Capabilities on the Discord API

Reads (verified live, HTTP 200):
- `GET /guilds/{g}/channels`
- `GET /channels/{c}/messages`
- `GET /guilds/{g}/members`
- `GET /guilds/{g}/roles`

Writes (verified live):
- `POST /channels/{c}/messages` (public note)
- `POST /users/{u}/channels` + message (private DM warning)
- `PATCH /guilds/{g}/members/{u}` with `communication_disabled_until` (timeout)
- `PUT` / `DELETE /guilds/{g}/members/{u}/roles/{r}` (role add/remove)

Bot intents required: `GUILD_MEMBERS` and `Message Content` (both privileged, enabled in the Developer Portal). Without `Message Content`, Discord strips `content` from messages the bot doesn't @mention — moderation is impossible.

## 7. Security review

### Threat model

| Asset | Exposure | Mitigation |
|---|---|---|
| Discord bot token | Held as a Mind tenet | Never echoed in chat; not logged; `Authorization: Bot <token>` header only |
| Member PII | Incident log / member profiles | Only stored in Mind memory; never exposed outside server + steward email |
| Moderation actions | Discord API | Reversible-first ladder; no ban/delete in v1; every action logged |
| Steward account | Discord | Timeout/role ops only; owner cannot be banned/deleted |

### Design decisions

1. **Token as tenet, not in code.** The bot token is delivered once via chat and stored as a private memory tenet with highest salience. It is never committed, never echoed, and never appears in logs. Demo/diagnostic messages instruct the Mind to redact it.
2. **No destructive actions.** The manifest hard-forbids permanent bans and message deletion in v1. Critical incidents time out the member and alert the steward for a human decision.
3. **Reversible actions only.** Timeouts auto-expire; role changes are reversible. The cost of a wrong call is bounded (5–10 minute timeout, zero residual state), which makes false positives cheap — a deliberate property.
4. **Recognized platform bug, documented workaround.** The platform's Discord app connector returned `401 Unauthorized` on all 13 tools despite completed OAuth and a valid token. The documented official path (send your own bot token to the Mind) is used instead. See `bug-report-discord-401.md` for the full report filed with the platform team.

## 8. Operational notes

- **Steward email digest (designed capability).** On critical incidents or after an incident batch, the Guardian composes an email digest to the steward: who did what, what was done, and a recommended next step (e.g. "repeat scammer — recommend ban"). This is the "your community manager handled it while you slept" moment — it converts autonomous action into creator trust, and is the primary escalation channel for anything the ladder won't touch without approval. (Not part of the live demo loop; included in the manifest as the designed flow.)
- Monitoring cadence is configurable (default 300s). Each observation cycle costs ~6.8 cognition credits, so cadence balances responsiveness against runtime budget.
- On-demand scans can be triggered by the steward at any time and don't wait for the cadence tick.
- The Mind's processing loop has occasional platform-side latency/unresponsiveness; `minds mind disable/enable` and a re-ping are the documented recovery path.

## 9. Viability & scalability (why creators adopt this)

- **Crowded category, empty niche.** Keyword moderation bots (MEE6, Wick, AutoMod) are ubiquitous but universally disliked for false positives. No mainstream tool combines *context understanding, member memory, and steward-driven learning* — exactly the three things creators say they want from a moderator.
- **Monetization path.** Free tier moderates one server (the creator's own); paid tier adds multiple servers, multi-platform (Telegram, X), and the email digest. The per-incident value is concrete and measurable (scams stopped, regulars unharmed), so willingness to pay is tied to a visible ROI.
- **Distribution.** Discord is where creator communities live; the bot installs via the standard OAuth + token flow already documented by the platform. A creator sees value in the first incident handled.
- **Why it compounds.** Every incident and every steward override improves the memory. The longer it runs, the better it understands the specific community — a moat that grows with usage and that no fresh keyword bot can match.

## 10. Validation metrics (for the demo + doc)

- **Autonomy:** incidents handled without steward intervention (verified: flood → timeout, scam → DM + timeout)
- **False-positive rate:** steward overrides / actions taken (verified: override on staged test → lesson learned)
- **Context vs keyword:** a "hate" message ignored, a scam template escalated (verified)
- **Memory reuse:** repeat offender detected via stored strike history (verified: Strike 1 → Strike 2)
- **Time-to-response vs human baseline:** autonomous detection in seconds via cadence/on-demand scans, vs a creator who may be offline for hours
