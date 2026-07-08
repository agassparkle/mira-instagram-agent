# Mira — Instagram Growth Agent

*Architecture inspired by [Nova](https://github.com/sharbelxyz/nova-youtube-agent), Sharbel's YouTube growth agent.*

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

## Every metric Mira can pull

The full data surface, live-verified against the API (July 2026) — including
several metrics Meta's own documentation tables omit.

**Your account — the daily pulse**
- Views & reach, each **split into followers vs non-followers** (your discovery rate)
- Profile views, website clicks, profile-button taps (call / email / bio link)
- Likes, comments, shares, saves, replies, reposts, total interactions, accounts engaged
- New follows & unfollows per day, follower count history
- **When your followers are online**, hour by hour — posting times from data, not folklore

**Your audience — who they actually are**
- Three demographic lenses, compared: who **follows** you, who **engages**, and who you **reach** — by age, gender, country, city, over selectable timeframes (works from 100 followers)

**Every post & Reel**
- Views, reach, likes, comments, saves, shares, reposts
- **Follows gained from the post** — which content converts viewers into followers
- Profile visits and **bio-link clicks** driven by the post — the money metric
- Reels: average & total watch time, and **skip rate** — the % of viewers gone in the first 3 seconds, Instagram grading your hook directly

**Stories** *(insights expire 24h after posting — Mira pulls before they vanish)*
- Views, reach, replies, link clicks
- **Exit points** — taps-away vs swipes-forward vs rewinds, i.e. exactly where people bail

**Tagged content**
- Posts where *others* tag you — collabs, customer posts, social proof

**Competitors** *(full setup)*
- Followers, posting cadence, per-post likes & comments, **Reel view counts** — real outlier detection across your niche

**Niche research** *(full setup)*
- Top & recent posts for any hashtag (Meta caps this at 30 hashtags per week)

**Comments** *(full setup)*
- Full text of comments on your posts — sentiment, themes, and the questions your audience keeps asking

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
Install this skill: https://github.com/agassparkle/mira-instagram-agent
```

**Option B — manual:**

```bash
git clone https://github.com/agassparkle/mira-instagram-agent.git ~/skills/mira-instagram-agent
```

Then tell your agent: **"Install Mira"**. She runs a privacy gate first (what
she'll read, where it's stored, what she'll never do), then a ~10-question
onboarding, then walks the API setup step by step and verifies every stage
with a live call before moving on.

## Upgrading an existing install

Your personal files (`config.md`, `.env`, `memory/`) are gitignored — upgrades
never touch them. Onboarding answers, tokens, and everything Mira has learned
survive every update.

- **Installed with git** (both install options above): `cd` into the skill
  directory and `git pull`. Done.
- **Installer copied files / no `.git` there:** tell your agent:
  *"Update the Mira skill from https://github.com/agassparkle/mira-instagram-agent —
  replace SKILL.md, setup/, config-template.md, example-config.md and README.md,
  but do not touch config.md, .env, memory/ or references/."*
- Then start a fresh session so the updated skill actually loads.

If you hand-edited SKILL.md locally (custom systems, personal tweaks), a pull
may conflict — re-apply your customizations on top, or keep them in a separate
file the skill references.

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
