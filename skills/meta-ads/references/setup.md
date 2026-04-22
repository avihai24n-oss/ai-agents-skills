# One-time Meta API setup

Read this when the user has no credentials yet, or when `auth_check.py` fails.

This is the only manual part of the skill. Walk the user through it patiently — Meta's developer console is not famously clear and they may have questions. None of these steps can be automated (they require human clicks in Meta's web UI and decisions only the user can make).

Plan on 15–25 minutes the first time, mostly waiting for Meta's UI.

## Step 0: Which path?

**Path A — System User token (never expires)**
- Your ads run inside a Meta Business Manager you own.
- The ad accounts you want to analyze are **owned by that BM**.
- You want a token you don't have to rotate.

**Path B — Long-lived user token (60 days)**
- You boost posts **from the Instagram app** (these usually create a personal ad account *outside* any Business Manager).
- Your ad account is attached to your **personal Facebook profile**, not a BM.
- You want to see every ad account your personal FB user can access.

**Not sure? Start with Path B.** A user token sees everything a System User token can see *plus* your personal ad accounts. You can always switch to Path A later if you move all your ad activity into a Business Manager.

Both paths share these one-time prerequisites: **a Meta developer app with the Marketing API product enabled.** Do steps 1–2, then jump to Path A or Path B.

---

## Step 1: Create a Meta developer app (both paths)

1. Go to <https://developers.facebook.com/apps>.
2. Click **Create App**.
3. Use case: **"Other"** → Next. App type: **"Business"** → Next.
4. App name: something descriptive for you (only you see it). Contact email: yours. If you have a Business Manager, select it.
5. Create the app.

## Step 2: Add the Marketing API product (both paths)

1. Inside your new app dashboard, scroll to "Add products to your app".
2. Find **Marketing API**, click **Set up**.
3. The dashboard now shows Marketing API permissions. You're done with this step.

Find your **App ID** (top of app dashboard) and **App Secret** (Settings → Basic → Show). You'll need the App Secret **only** for Path B token refresh. Path A users can skip noting the secret.

---

## Path A: System User token (recommended for BM-owned ads)

### A.1 — Make sure your ad account is in a Business Manager

1. Go to <https://business.facebook.com/settings>. If you've never created a BM, create one now ("Create Account" at <https://business.facebook.com>).
2. Business Settings → **Accounts** → **Ad Accounts**. If the ad account you want isn't listed, click **Add** → "Add an ad account" (if you own it) and enter its ID.
3. Note your **Business ID** (Business Info, top-right, or the URL).

If Meta refuses to let you add a personal ad account to the BM, your account is probably locked to your personal profile — switch to Path B.

### A.2 — Create a System User and grant it access

1. Business Settings → **Users** → **System Users** → **Add**.
2. Name it something like `claude-skill-admin`. Role: **Admin** (required for writes; use Employee for read-only).
3. Click the system user you just created → **Add Assets**.
4. **Ad Accounts** tab → check each account you want the skill to access → grant **"Manage campaigns"** (both read + write) or "Read performance" (read only). Save.
5. **Apps** tab → check the app from Step 1 → **"Manage app"** or "Develop app". Save.

### A.3 — Generate the System User token

1. Still on the System User page → **Generate New Token**.
2. **App:** the one from Step 1.
3. **Permissions:** check `ads_read`, `ads_management`, `business_management`.
4. **Token expiration: Never** (this is the whole point of System User tokens).
5. Generate. **Copy immediately** — Meta shows it once. Lose it and you regenerate.

### A.4 — Find your ad account ID

In Ads Manager, the URL contains `act=1234567890`. That number is your ad account ID. The format Meta expects is `act_1234567890`.

### A.5 — Write the .env file

Copy `assets/env.template` to `.env` in the directory where you'll run the skill:

```
META_ACCESS_TOKEN=EAAxxxxxxxxxx_THE_SYSTEM_USER_TOKEN
META_AD_ACCOUNT_ID=act_1234567890
META_API_VERSION=v25.0
```

You can leave `META_APP_ID` and `META_APP_SECRET` blank for Path A.

### A.6 — Verify

```bash
pip install -r requirements.txt    # once per machine
python scripts/auth_check.py
python scripts/list_accounts.py --with-recent-spend
```

First command should print `"ok": true` and list visible ad accounts with your identity as the System User. Second command tells you which accounts are actually running spend.

---

## Path B: Long-lived user token (for personal ad accounts / Instagram boosts)

### B.1 — Get a short-lived user token from Graph Explorer

1. Go to <https://developers.facebook.com/tools/explorer> while logged into the **personal FB account** that owns the ad activity you want to see.
2. Top-right **Meta App** dropdown: select the app from Step 1.
3. **User or Page** dropdown: **User Token**.
4. Click **"Permissions"**. Check: `ads_read`, `ads_management`, `business_management`, `pages_show_list`. Close the permissions popup.
5. Click **"Generate Access Token"**. Accept the OAuth popup (it may ask which Pages to grant access — pick the ones connected to your IG boosts, at minimum).
6. The token appears in the big text field at the top. Copy it. **This token lasts ~1–2 hours** — you're about to exchange it.

### B.2 — Exchange for a 60-day long-lived token

Put your App ID and App Secret (from Step 2) into `.env`:

```
META_APP_ID=1234567890
META_APP_SECRET=abcdef1234567890abcdef1234567890
```

Then run:

```bash
pip install -r requirements.txt    # once per machine
python scripts/exchange_token.py --write-env
# Paste the short-lived token when prompted (input is hidden).
```

This:
- Calls Meta's `oauth/access_token` exchange endpoint.
- Writes the resulting 60-day token into `META_ACCESS_TOKEN` in your `.env`.
- Prints the expiration (~59.5 days in seconds).

The short-lived token is now useless — delete it from wherever you pasted it.

### B.3 — Discover your ad accounts

```bash
python scripts/auth_check.py
python scripts/list_accounts.py --with-recent-spend
```

`list_accounts.py` returns every ad account your personal FB user can see, sorted by last-30-day spend then lifetime spend. Pick the one with the real activity you care about.

### B.4 — Pin the default ad account

Add to `.env`:

```
META_AD_ACCOUNT_ID=act_1234567890
```

Now scripts default to that account without needing `--account-id` on every call.

### B.5 — 60-day rotation

Long-lived user tokens expire after ~60 days. Set a calendar reminder. When it expires, repeat B.1 then B.2 — the same exchange works as long as the app ID/secret are still valid.

---

## App Review (only if needed)

For your **own** ad accounts, the app stays in "Development mode" and works fine — no App Review needed. App Review is only required if *other* Meta users (not just you) need to use this app to access *their* ad accounts.

If a client wants this skill against their accounts, they should run their own setup with their own app and System User. Don't share tokens between clients.

---

## What to put in the user's password manager

Save these somewhere safe (1Password / similar):
- App ID
- App Secret (only if using Path B)
- System User Token (Path A) or long-lived user token (Path B)
- Business Manager ID (Path A)
- Ad Account ID(s)

If any token leaks, anyone with it can spend money on your ads and/or read your ad data. Treat it like a credit card.

---

## Troubleshooting

See `references/troubleshooting.md` for the full list. The top three hits:

- **`auth_check.py` returns `ok: true` but `ad_accounts_visible: 0`** — token is valid but can't see accounts. For Path A: System User needs access to each account (Business Settings → System Users → Add Assets). For Path B: your personal FB user doesn't own any ad accounts — make sure you're logged into the right FB account at Graph Explorer.

- **`(#3018) start date cannot be beyond 37 months from the current date`** — Meta's insights API only serves data from the last 37 months. Campaigns older than that are invisible via API. Check Ads Manager archived reports UI instead.

- **`ProxyError: Tunnel connection failed: 403`** — your execution environment can't reach `graph.facebook.com`. Run the scripts from the user's host machine instead of a sandbox.
