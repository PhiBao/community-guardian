# Community Guardian — Demo Script & Reproduction Guide

## What to prepare

A Discord server plus the Minds chat, showing four live moderation scenes
that tell one story: **"protecting a creator's community from scams,
without enraging the regulars."**

Use RFC 2606 test domains (`.example.com`, `.test`, `.invalid`) for any links — they
are reserved, non-routable, and safe. Never post a real site.

---

### Scene 1 — Scam burst → detected + reversible timeout (0:00–0:30)

**The story: a scammer hits the server while the creator is busy.**

1. From a **test account**, post a burst of scam-like messages into `#general`
   (6–8 rapid messages: empty posts, "free nitro" promos, or a link flood).
2. In the Minds chat, trigger an on-demand scan: `"Guardian, scan #general now"`.
3. Show the Guardian's assessment: it identifies the **scam-flood pattern**, names the
   risk (phishing / free-item scam), and applies a **reversible short timeout** (e.g.
   300s) via `communication_disabled_until`.
4. Show a calm public note posted in Discord by the bot.
5. On-screen caption: **"Scam burst detected. Reversible action. No ban, no delete."**

> Verified live: `_harukiii.` flood at 18:47Z → 5-min timeout applied (HTTP 200,
> `communication_disabled_until` set) + public note posted.

### Scene 2 — In-joke → correctly ignored (0:30–1:00)

**The story: the differentiator — the same community banter a keyword bot would punish.**

1. From a **known regular** account, post banter containing a keyword-bot trigger word.
   Example: `"i hate this update so much lmao who greenlit this"`.
2. Trigger a scan.
3. Show the Guardian's judgment: it names the keyword trap (`hate`), notes the author
   is a known regular with positive history, and **takes no action**.
4. On-screen caption: **"Context-aware. A keyword bot would mute this fan."**

> Verified live: `"i hate this update so much lmao who greenlit this"` → no action,
> logged as playful vent. A keyword-bot would have warned/muted the member.

### Scene 3 — Repeat scammer → memory-based escalation (1:00–1:30)

**The story: the scammer returns — the Guardian remembers and escalates.**

1. From the same test account, post a scam-template message using a test-domain link:
   `"yo free nitro here totally-not-a-scam.example.com/nitro for everyone!! dm me fast"`.
2. Trigger a scan → the Guardian applies **Strike 1** (private DM warning, names what
   and why).
3. Post the same template again.
4. Trigger a scan → the Guardian **remembers the prior strike** and applies
   **Strike 2: 10-minute timeout** (HTTP 200, verified on Discord).
5. On-screen caption: **"Remembers. Escalates repeat scammers automatically."**

> Verified live: DM warning at 19:48Z (strike 1), 10-min timeout at 19:54Z (strike 2,
> `communication_disabled_until: 20:04:02Z` confirmed on Discord).

### Scene 4 — Steward override → learning (1:30–2:00)

**The story: the creator is in control — the Guardian learns from every correction.**

1. Tell the Guardian the staged test was not real spam and to clear the strikes.
2. Show it reset strikes to 0 and record a lesson (RFC 2606 test-domain recognition).
3. On-screen caption: **"Learns from the steward. No permanent record."**

> Verified live: strikes 2 → 0, lesson tenet written, behavior change confirmed for
> future test-domain links.

---

## What the demo proves (maps to judging criteria)

| Judging criterion | Where it's proven |
|---|---|
| Minds Integration Depth | The Mind is the product — it reads, judges, acts, and remembers |
| Persistence (memory / continuity / autonomous follow-up) | Scene 3 (remembers strikes across sessions) + scenes 1/3 (acts without prompting) |
| Creator-economy problem fit | Scam protection = protecting creator income + audience trust (Track C) |
| Innovation | Scenes 2 vs 3: context distinguishes a regular's joke from a real scam — keyword bots can't |
| Execution & completeness | Live, verified actions on a real Discord server |

---

## Reproduction checklist

- [ ] Minds account + enabled Mind (Gang.Guardian)
- [ ] Discord bot with `GUILD_MEMBERS` + `Message Content` intents, in the server
- [ ] Bot token delivered to the Mind (stored as tenet; never echoed)
- [ ] Mind config: `active_guild_id`, `watchlist`, `channel_purposes`
- [ ] `manifest.md` pasted; Mind replied "Build it"
- [ ] Two Discord accounts: the regular account + the staged-test account
- [ ] Test-domain links only (`.example.com` / `.test` / `.invalid`)
