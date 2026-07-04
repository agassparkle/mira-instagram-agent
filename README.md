# Mira — Instagram Growth Agent - Totally inspired by sharbelxyz nova-youtube-agent

Mira is an AI agent that runs your Instagram content strategy end-to-end:
competitor scanning, account analysis, idea generation, Reels scripts, comment
intelligence, performance tracking, and a feedback loop that makes her smarter
the longer you use her.

The name is a star — and in Spanish, *mira* means **"look!"**, which is what
every Instagram post is trying to make someone do. (Don't like it? Rename your
agent during onboarding or anytime after — Mira is just the default persona.)

Runs on any agent harness that can load a markdown skill and call HTTP APIs
(OpenClaw, Hermes Agent, Claude Code, and similar). Built on the **official
Instagram Graph API** — no scrapers, no cookie hacks, no risk to your account.

---

## What Mira does

| System | What it does |
|---|---|
| 🔍 Competitor scan | Finds breakout Reels across your niche (real view counts via the official API) and extracts the transferable pattern |
| 📊 Account analysis | Pulls your real metrics and separates what drives *reach* from what drives *follows* |
| 💡 Idea generation | Interviews you first, mines your comments for recurring questions, grounds every idea in your real experience |
| 🎬 Content writing | Reels scripts (2-second hook rule), carousels, captions, hashtags — in your voice |
| 💬 Comment intelligence | Reads your comments for sentiment and themes; every recurring question becomes a post idea |
| 📈 Performance logging | Pulls each post's numbers itself — views, saves, shares, follows, watch time |
| 🔄 Feedback loop | Logs every approval and rejection with reasons — never repeats a rejected angle |
| 🧠 Learning loop | Reads all memory before every session; gets sharper the longer she runs |

## What Mira never does

- **Never posts, comments, DMs, or follows.** She prepares; you publish.
- **Never touches private data.** Competitors = public professional accounts
  only. Your tokens and config stay in local, gitignored files.
- **Never displays your tokens** — including through the two side channels
  Meta itself leaks them through (documented and defended against; see the
  setup guide's gotchas G11/G18).

---

## Two setup levels

| | Quick (~5 min) | ⭐ Full (~15 min) |
|---|---|---|
| Facebook Page needed | No | Yes (can stay empty — it's plumbing) |
| Your own stats (growth, views, reach, watch time) | ✅ | ✅ |
| Competitor scanning with Reel view counts | — | ✅ |
| Reading your comments (sentiment, question mining) | — | ✅ |

Start quick if you want; upgrading later takes ~10 minutes and keeps
everything you've built. Mira walks you through either path conversationally,
with clickable links for every step — including Instagram's account-conversion
privacy notice *before* you convert, not after.

## Requirements

- An Instagram account (Mira guides the free switch to a Creator account)
- A Facebook/Meta account (needed for Meta's developer portal on both paths)
- An agent harness (OpenClaw, Hermes Agent, Claude Code, ...) with any capable model

## Install

**Option A — point your agent at this repo:**

```
Install this skill: https://github.com/<owner>/mira-instagram-agent
```

**Option B — manual:**

```bash
git clone https://github.com/<owner>/mira-instagram-agent.git ~/skills/mira-instagram-agent
```

Then tell your agent: **"Install Mira"**. She runs a privacy gate first (what
she'll read, where it's stored, what she'll never do), then a ~10-question
onboarding, then walks the API setup step by step and verifies every stage
with a live call before moving on.

## Why the setup guide is worth the repo alone

`setup/graph-api-setup.md` is a live-tested field manual for the Instagram
Graph API with **19 documented gotchas** that Meta's own docs won't tell you,
including:

- the Page-creation rate limit that punishes retries (G1)
- "permissions granted" while zero assets are actually opted in (G14)
- Explorer tokens silently dying within the hour, and the exchange endpoint
  that fails while the UI button works (G17)
- the API leaking your own token back at you in paging URLs and error
  messages (G11, G18)
- and the definitive experiment showing Instagram-Login tokens **cannot be
  revoked** while an app stays authorized (G19) — which is why Mira's
  secret-handling rules are hard rules

## File structure

```
mira-instagram-agent/
├── README.md                  ← you are here
├── SKILL.md                   ← Mira's full instructions (all systems)
├── config-template.md         ← template; Mira fills config.md during onboarding
├── example-config.md          ← a filled example for reference
├── setup/
│   └── graph-api-setup.md     ← the field manual (19 gotchas, link directory)
├── memory/                    ← Mira's memory (gitignored, stays local)
└── references/                ← accumulated playbooks (grows as you use her)

.env                           ← your tokens (gitignored, created at setup)
config.md                      ← your profile (gitignored, created at onboarding)
```

## Privacy

`config.md`, `.env`, and everything in `memory/` are gitignored. Your personal
data, tokens, and Mira's knowledge about your account never leave your machine.

## Credits

Architecture inspired by [Nova](https://github.com/sharbelxyz/nova-youtube-agent),
Sharbel's YouTube growth agent — the onboarding → config → memory → feedback-loop
pattern. Mira is an independent implementation for Instagram with an
API-verified data layer.

## License

MIT — free to use, modify, and share.
