# Instagram Graph API Setup — Field Manual

This is not boilerplate. Every step and every gotcha below was hit live during
a real end-to-end setup (July 2026) with a real account, including the failure
modes. Mira follows this file when walking a user through data setup.

Two paths (the user picks via the choice screen in SKILL.md):

- **QUICK** — "Instagram API with Instagram Login". No Facebook Page. Own-account
  data only (profile, media, insights). Comments endpoint returns nothing on
  this path (verified — see G12).
- **FULL** — "Instagram API with Facebook Login". Needs a Facebook Page linked
  to the Instagram account. Adds Business Discovery (competitor data incl.
  Reel view counts) and comment reading.

A Facebook/Meta account is required for BOTH paths (the developer portal login).
The Page is only needed for FULL.

Wizard behavior rules:
- One step at a time; verify each stage with a live API call before moving on.
- Always give clickable links (directory at the bottom); build `{app-id}` links
  as soon as the app exists.
- Tokens go straight into the gitignored `.env` — never into chat.
- When pointing at a Meta/Instagram menu, give all known name variants — Meta
  A/B-tests labels.

---

## Stage A — Instagram account → Creator

1. If the account is Personal: show Instagram's own conversion notice VERBATIM
   and get acknowledgment first (see SKILL.md privacy gate + onboarding). The
   sharp edge: **pending follow requests on private accounts are auto-approved
   the moment the account converts.** Users must know before, not after.
2. Instagram app (phone): Settings → Account type → switch to **Creator**
   (Business also works; Creator fits individuals). Free, reversible.
   Mobile-only step — no web URL exists.

## Stage B — Facebook Page (FULL path only)

Preferred route (verified working even while the web form was rate-limited):
- **Create the Page from inside Instagram:** phone app → Profile → Edit
  profile → the "Page" / "Facebook" row → Connect → log into Facebook when
  prompted (expected, legitimate) → accept when it offers to **create a new
  Facebook Page**. Creates AND links in one flow.

Alternate routes:
- Web: https://www.facebook.com/pages/create → choose **"Public Page"** —
  NEVER "Upgrade your existing profile" (that converts the personal profile;
  it is not a real Page and does not work with the API). Name 5+ characters,
  category "Digital creator" is fine, skip every optional prompt. Then link
  from the Instagram app: Edit profile → Page → connect.
- Facebook mobile app: Menu → Pages → Create.
- Meta Business Suite: https://business.facebook.com → Settings → Pages → Add.

Notes:
- The Page can stay empty forever. It's plumbing.
- Linking is MOBILE-APP-ONLY: Instagram web's Edit profile has no Page row.
- Do NOT confuse the Page link with Accounts Center profile-sharing
  (cross-posting to a Facebook *profile*). Accounts Center does not satisfy
  the API requirement.

## Stage C — Meta developer app

1. https://developers.facebook.com → log in → first visit asks to register as
   a developer (email/phone confirm).
2. Create app: https://developers.facebook.com/apps/creation/
3. "Add use cases" screen: select **"Manage messaging & content on Instagram"**
   (both API flavors live inside this single use case — see Stage D2).
4. Business portfolio screen: keep **"I don't want to connect a business
   portfolio yet."** Portfolio/verification is only for published apps serving
   third parties. Say this explicitly — the wording makes people think they
   need a registered company. They don't. Dev mode + own account = full access.
5. Requirements screen: "No requirements identified" is the expected result.
6. Overview → Create app (password confirm).
7. Ignore "Become a Tech Provider" and everything under Publish. The app stays
   in dev mode forever.

## Stage D — QUICK path: Instagram-Login setup

Dashboard → "Customize the Manage messaging & content on Instagram use case"
(or Use cases → Customize) → **API setup with Instagram login**:

1. The panel shows the Instagram app name / app ID / **app secret**. Never
   show app ID and secret together; same secrecy rule as tokens.
2. Section 1 → **Add all required permissions**.
3. Left panel → **Permissions and features** → also enable
   **`instagram_business_manage_insights`** (VERIFIED present on this flavor;
   the welcome banner's "switch to Facebook login for insights" is stale — it
   applies to hashtags and other-account data). Standard Access suffices in
   dev mode.
4. Section 2 → **Add account** → Instagram login opens → at the consent
   screen set the toggles: insights **ON**; messages, publishing, and comments
   **OFF** (comments return nothing on this flavor anyway — G12).
   If "Insufficient Developer Role" appears → G9.
5. **Generate token** → user pastes it into `.env` as `IG_ACCESS_TOKEN=...`
   (never into chat). This flavor's tokens are already long-lived.
6. Section 3 (webhooks) and section 4 (business login): SKIP — published-app
   features.

## Stage D2 — FULL path: Facebook-Login setup (adds Business Discovery)

Same app, same use case — no second app needed. Use cases → Customize → left
panel → **API setup with Facebook login**:

1. **Add required content permissions** (the content bundle: `instagram_basic`,
   `pages_read_engagement`, `pages_show_list`, plus `instagram_content_publishing`
   and `business_management` which Mira never uses). SKIP the messaging bundle.
2. `instagram_manage_insights` is NOT in the content bundle: left panel →
   **Permissions and features** → find and Add it (the plain one — the
   `instagram_business_*` names are the other flavor's family; G13 note).
3. Token via Graph API Explorer:
   https://developers.facebook.com/tools/explorer/{app-id}/ →
   **Get User Access Token** → select permissions: `instagram_basic`,
   `instagram_manage_insights`, `pages_show_list`, `pages_read_engagement`
   (+ `instagram_manage_comments` if comment features are wanted) →
   **Generate Access Token**.
4. In the OAuth popup: choose **"Opt in to current Pages only"**, tick ONLY
   the linked Page — confirm it reads "1 asset selected" (G14) — and tick the
   Instagram account on its screen.
5. Token → `.env` as `FB_ACCESS_TOKEN=...`. App secret (for the exchange) →
   `.env` as `FB_APP_SECRET=...` from
   https://developers.facebook.com/apps/{app-id}/settings/basic/
6. **Exchange for the 60-day token IMMEDIATELY — Explorer tokens die within
   ~1–2 hours** (G17). Lead with the UI route for fresh apps:
   https://developers.facebook.com/tools/debug/accesstoken/ → paste token →
   Debug → **"Extend Access Token"** (bottom) → replace `FB_ACCESS_TOKEN` in
   `.env` with the extended token. The API exchange
   (`GET /oauth/access_token?grant_type=fb_exchange_token&client_id={app-id}&client_secret={secret}&fb_exchange_token={token}`)
   is the automated route for later refreshes but returned persistent
   "transient" code-2 errors for a fresh app (0 for 12 over 36 minutes, with a
   verifiably correct secret) while the button worked first try.

## Stage E — verification calls (run before onboarding continues)

QUICK (host `graph.instagram.com`, `IG_ACCESS_TOKEN`):
- `/v23.0/me?fields=user_id,username,followers_count,media_count`
- `/v23.0/me/media?fields=id,media_type,timestamp,like_count,comments_count`
- `/v23.0/me/insights?metric=views,reach&period=day&metric_type=total_value`

FULL (host `graph.facebook.com`, `FB_ACCESS_TOKEN`):
- `debug_token` → confirm validity, ~60-day expiry, and granular_scopes
  targeting exactly one Page + one IG account
- `/{page-id}?fields=name,instagram_business_account` → the IG user ID
- one Business Discovery call on a known professional account, e.g.
  `/{ig-user-id}?fields=business_discovery.username(natgeo){followers_count,media{like_count,comments_count,view_count}}`
  — confirms competitor access AND view_count (present on VIDEO media).

Always `curl -g` (G16). Always strip `paging` before showing results (G11).
Never show raw errors (G18).

## Stage F — token lifecycle & repair

- Long-lived tokens ≈ 60 days. Mira records the issue date, checks expiry
  every session (`debug_token`), and walks the refresh when <10 days remain.
- Rotation/revocation is a REPAIR path, not setup — and on the Instagram-Login
  flavor it barely exists (G19). Prevention (G11/G18 sanitization) is the
  actual defense.

---

## Direct links directory

Rule: always hand the user a clickable URL. Fill `{app-id}` once the app exists.

Meta developer portal:
- Create app: https://developers.facebook.com/apps/creation/
- Dashboard: https://developers.facebook.com/apps/{app-id}/dashboard/
- Use cases: https://developers.facebook.com/apps/{app-id}/use_cases/
- App settings → Basic (app secret): https://developers.facebook.com/apps/{app-id}/settings/basic/
- App roles (tester invites): https://developers.facebook.com/apps/{app-id}/roles/roles/
- Graph API Explorer (app preselected): https://developers.facebook.com/tools/explorer/{app-id}/
- Access Token Debugger (+ Extend button): https://developers.facebook.com/tools/debug/accesstoken/

Instagram:
- Apps & websites / Website permissions (tester invites, app access):
  https://www.instagram.com/accounts/manage_access/

Facebook:
- Your Pages: https://www.facebook.com/pages/
- Create a Page: https://www.facebook.com/pages/create/
- Business integrations (full-path authorizations live HERE, not in "Apps and
  websites"): https://www.facebook.com/settings?tab=business_tools
- Meta Business Suite: https://business.facebook.com

Mobile-only (no URL — prose + "pick up your phone"):
- Instagram → Creator conversion (Settings → Account type)
- Instagram → Edit profile → Page connect/create

---

## Gotchas log (all hit or verified live)

- **G1 — Page creation rate limit.** Generic "ensure you are following Page
  policies" error; every retry COUNTS and extends the cooldown until it
  becomes "too many Pages recently." Short names (≤4 chars) can trigger the
  first rejection. Branch: stop retrying, check https://www.facebook.com/pages/
  in case an attempt silently succeeded, wait (30 min–24 h), use the alternate
  routes in Stage B. The policy link in the error is a generic catch-all —
  don't send users to read it.
- **G2 — "Public Page" vs "Upgrade your existing profile".** Only Public Page
  works with the API. The upgrade option converts the personal profile and is
  a dead end here.
- **G3 — Page linking is mobile-only.** Desktop Instagram web lacks the
  option entirely.
- **G4 — Accounts Center ≠ Page link.** Two near-identical "connect Facebook"
  features; only the Page connection counts for the API.
- **G5 — Business-portfolio wording scares users** into thinking company
  verification is required. In dev mode with your own account, it isn't.
- **G6 — "Extra steps required" dialog** when adding the Instagram use case
  refers to publishing requirements. Irrelevant in dev mode; proceed.
- **G7 — Stale insights banner.** The Instagram API panel suggests switching
  to Facebook login for insights; `instagram_business_manage_insights` is in
  fact available on Instagram Login (verified on Permissions and features).
- **G8 — QUICK→FULL upgrade is exactly two steps** (Page + Facebook-Login
  token). Nothing else changes; config and memory carry over.
- **G9 — "Insufficient Developer Role"** on Add account/Generate token. The
  IG account must be an app tester, and it's a two-sided dance the error
  doesn't explain: App roles → Roles → Add people → "Instagram Tester" → IG
  username, THEN accept on the Instagram side
  (https://www.instagram.com/accounts/manage_access/ — labeled "Website
  permissions" or "Apps and websites" depending on version → Tester invites →
  Accept). Retry after both.
- **G10 — Instagram's consent screen supports least privilege** (verified).
  Toggling OFF messages/comments/publishing while keeping profile+insights
  works fine. Tell users which toggles to flip — a token that can't post or
  DM is structurally safer.
- **G11 — the API leaks the token in paging URLs.** `paging.next`/`previous`
  embed the full access_token. Strip `paging` from anything displayed or
  logged. Non-negotiable.
- **G12 — comments are FULL-path only (verified by fresh-post test).** The
  Facebook-Login token returns full comment text + usernames, including other
  users' comments, even in dev mode. The Instagram-Login token returns
  `{"data": []}` for the SAME media — even for the app user's own fresh
  comment, with the permission granted. Also: comments from before the
  professional conversion appear unavailable on both flavors.
- **G13 — Graph API Explorer only lists permissions the app is configured
  for.** Before Stage D2, the Facebook-Login permission family isn't
  selectable at all. Enable the use case's Facebook-login setup first;
  otherwise the Explorer strands the user.
- **G14 — permissions granted ≠ assets opted in.** If no Page is ticked in the
  OAuth popup ("0 assets selected"), every permission still shows "granted"
  while `/me/accounts` returns an EMPTY list with no error. Diagnostic:
  `debug_token` → `granular_scopes` → per-permission `target_ids`; empty
  targets = failed opt-in. Fix: remove the authorization (Facebook →
  **Business integrations**, NOT "Apps and websites") and re-consent.
- **G15 — `/me/accounts` can return empty even when correctly scoped**
  (listing lag after re-auth). Don't depend on it: take the Page ID from
  `debug_token` granular_scopes and query `/{page-id}` directly.
- **G16 — curl globbing.** Business Discovery field syntax uses `{braces}`;
  curl parses them as globs and fails silently. Always `-g`/`--globoff`.
- **G17 — Explorer tokens are SHORT-LIVED (~1–2 h).** Exchange immediately.
  For fresh apps, the API exchange endpoint can return persistent "transient"
  code-2 errors (observed: 12/12 failures over 36 min with a verified-correct
  secret) while the Access Token Debugger's **"Extend Access Token"** button
  works first try. Lead with the button at setup; keep the API call for
  automated refreshes later. (Instagram-Login tokens from the use-case panel
  are already long-lived.)
- **G18 — Meta error messages echo the full token** ("Malformed access token
  EAAX..."). Second leak channel after G11. Never display raw error responses;
  summarize.
- **G19 — DEFINITIVE: Instagram-Login tokens cannot be rotated while the app
  stays authorized.** Tested every mechanism against a deliberately exposed
  token: generating a new token → old survives (they coexist). Instagram app
  secret reset → ALL tokens survive. Removing the app at manage_access → all
  tokens die instantly ✓ … but re-authorizing the same app **revives every
  previously issued token**, including the exposed one. Token validity is
  simply "is this app currently authorized for this user." A truly
  compromised setup must remove the authorization and migrate to a NEW app
  (new app ID = old tokens permanently orphaned). Partial mitigations:
  re-consent with narrowed toggles (old tokens inherit the CURRENT grant) and
  the ~60-day expiry. The real defense is never leaking tokens: G11 + G18.
- **G20 — .env appends glue onto the last line.** If `.env` lacks a trailing
  newline, appending a new `KEY=value` produces `OLDVALUEKEY=value` on one
  line. Symptoms are misleading: "Invalid OAuth access token" / "Malformed
  access token" / "(#200) Provide valid app ID" — nothing points at the file.
  Hit twice in live use. Verify the file ends with a newline before appending;
  after any append, sanity-check keys and value lengths (never print values).
- **G21 — token permanence (verified live 2026-07-08).** Instagram-Login
  tokens self-refresh via
  `graph.instagram.com/refresh_access_token?grant_type=ig_refresh_token&access_token={current}`
  → fresh 60-day token, agent-driven; the quick tier needs no human token
  maintenance, ever. Facebook flavor: the Page token derived via
  `/{page-id}?fields=access_token` from a long-lived user token has **no hard
  expiry** (debug_token expires_at = 0) and serves insights, Business
  Discovery, AND comments — all three verified live. Remaining clock:
  `data_access_expires_at` (~90 days) → quarterly re-auth (new user token →
  re-derive Page token). Zero-clock permanence exists only via Business
  Manager system-user tokens (requires a business portfolio; optional
  advanced path).
