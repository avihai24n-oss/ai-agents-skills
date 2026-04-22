# Write Actions: Safety Protocol

This file is **mandatory reading** before calling `pause_ad.py`, `update_budget.py`, or `duplicate_ad.py`. These scripts touch real money. A misread analysis or a confidently-wrong recommendation can pause a winning ad, 10x a losing budget, or duplicate the wrong creative across an account. There's no "undo" button — Meta will execute the change instantly and you'll be reconstructing the previous state from `effective_status` history if it goes wrong.

## The five non-negotiable rules

### 1. Explicit confirmation in chat for every single write

A previous "go ahead" or "yes do whatever you think" in the conversation does NOT count. Each individual write call needs its own confirmation message. The user has to actively re-affirm. This is friction by design — the friction is the safety feature.

What confirmation looks like, minimally:
- The user explicitly names the action (`yes pause it`, `confirm`, `כן תשנה`, `בצע`)
- It's the user's most recent message before the write call
- It's not just "ok" or "👍" in response to something else — it has to be unambiguous about the specific action

If you're unsure whether a message is a confirmation, treat it as not. Ask again.

### 2. Surface the impact before asking

Don't just say "should I pause ad 12345?" Say:

> Pause ad **`Summer_Launch_v3`** (ID `120214567890`)?
> - Spent ₪47.20 over the last 7 days
> - 0.8% CTR (vs your account avg 1.4%)
> - 12 link clicks, 0 conversions
> - Ad set still has 3 other ads delivering, so the ad set won't go dark
>
> Confirm with `yes` to proceed.

The user can only consent to what they understand. If you can't explain the impact in 3-5 lines, you don't understand it well enough to recommend the action.

### 3. One action per confirmation, by default

If the analysis suggests pausing 12 ads, don't bundle them into one mega-confirmation. Either:
- Walk through them one at a time (slower but safest), OR
- Present a numbered list of all 12 and explicitly ask for "pause all 12" or "pause #1, #3, #7" — accept partial selections

Never assume a previous batch confirmation extends to a new batch. If the user said "pause those 3 fatigued ads" and you found 5 more later in the conversation, the new 5 need their own confirmation.

### 4. Always dry-run when the script supports it

Every write script accepts `--dry-run`. Use it first when:
- This is the first write of the session
- The change is large (>50% budget swing, bulk pause, deep_copy duplications)
- You're acting on data you fetched more than 30 minutes ago (it might be stale)
- The user is testing the skill and hasn't done a write before

The dry-run output shows exactly what would change. Read it back to the user and ask for the real confirmation.

### 5. Never chain writes without intermediate verification

If the plan is "duplicate the winning ad, then pause the original" — don't run both as a single batch. Run the duplicate, fetch the new ID, confirm to the user that the duplicate is created and in the expected state, *then* ask to pause the original. Mid-chain failures are the worst kind: you can end up with the original paused and the duplicate broken, leaving the ad set with nothing delivering.

## Per-script safety notes

### `pause_ad.py`

- Status `PAUSED` stops delivery immediately. Spending stops within minutes (not seconds — there's a small in-flight auction tail).
- Status `ACTIVE` resumes. The ad re-enters the learning phase if it had been paused for >7 days.
- Pausing a campaign also stops all ad sets and ads under it; pausing an ad set stops only its ads.
- A paused ad in an active ad set with active sibling ads has no effect on overall delivery — Meta just shifts spend to the siblings. This is usually desirable.
- A paused ad in an ad set where it was the *only* active ad means the ad set goes dark. Surface this to the user before pausing.

### `update_budget.py`

- The 2x cap per call is a hard rule. To go from ₪50/day to ₪200/day, do it in steps: ₪50 → ₪100 → ₪175 → ₪200, with confirmations between. Override with `--allow-large-increase` only if the user explicitly says "yes I understand the risk, do the full 4x".
- Any change >20% triggers Meta's learning-phase reset. The script's output includes a `warning` field when this applies. Mention it to the user — they should expect 3-7 days of unstable performance after a big budget move.
- Decreases >50% can drive Meta to pause the ad set automatically because the budget is "too low to deliver". Watch for `effective_status: CAMPAIGN_PAUSED` in subsequent fetches.
- Lifetime budgets and daily budgets are mutually exclusive on a given object. The script auto-detects which is set; if both or neither are returned, it refuses to act.
- For campaign budget optimization (CBO) campaigns, budget lives at the campaign level, not the ad set. The script handles this correctly because it queries the object first, but be aware: passing an ad set ID for a CBO campaign will fail with a clear error.

### `duplicate_ad.py`

- Default `--status PAUSED` is the right default. Only override if the user explicitly wants the duplicate to go live immediately.
- `deep_copy=true` (used for campaigns) duplicates the entire tree — campaign, ad sets, ads, creatives. For a campaign with 5 ad sets and 20 ads, that's 26 new objects. Tell the user the count before duplicating.
- Duplicating an ad set into a different campaign (`--target-parent-id`) requires the target campaign to have a compatible objective. Mismatched objectives → API error.
- Naming: Meta's default suffix is " - Copy". Use `--rename` to keep the account organized — naming chaos compounds fast across iterations.
- Duplicate ≠ fresh creative. A duplicated ad is the same ad, just a new ID. It enters its own learning phase but the *creative* is identical. If the original is fatigued because of creative burnout, duplicating won't help — only new creative will.

## When something goes wrong

If you executed a write and the user wants to undo it:

- **Pause → Active rollback:** call `pause_ad.py --status ACTIVE` on the same ID. Note: if the ad was paused for >7 days, it re-enters learning.
- **Budget rollback:** call `update_budget.py` with the previous value. Each script's success output includes the previous value (`status_before`, `before_minor`) — capture these into the conversation context immediately so they're available for rollback.
- **Duplicate rollback:** the duplicate has a new ID returned in the output. To "undo," delete or pause the new copy. Deleting requires `delete()` on the meta_client — there's no bundled script because permanent deletion is risky; do it via the Meta UI unless the user really wants programmatic delete.

## What this skill will NOT help with

- **Bulk operations across accounts.** Each call is scoped to one ad account at a time. Multi-account bulk = multi-tenant work, wrong skill.
- **Creative editing or upload.** Out of scope for v1. Meta's creative endpoints have a learning curve and a bigger blast radius.
- **Audience creation or modification.** Same reason.
- **Pixel / CAPI event manipulation.** Different API entirely (Conversions API), different auth, different mental model.
- **Scheduled writes.** This skill is on-demand. If the user wants "pause all ads with frequency >4 every Monday morning," that belongs in a scheduler (cron, a scheduled-tasks integration, etc.), not inside this skill.

## Scope note: personal-scale vs. agency / enterprise

This skill is built for personal-scale ad management — one operator, one set of credentials, a handful of ad accounts. At that scale individual writes matter and confirmations are easy.

If this ever gets pointed at multi-tenant use (an agency managing dozens of client Business Managers, or an in-house team acting on shared client accounts), the rules in this file are insufficient. You'd need per-client credential storage, an audit log of every write with operator identity, role-based access, and probably a four-eyes approval flow for anything above a threshold. Don't try to retrofit this skill — build a separate multi-tenant version. Each account owner should run their own setup with their own Meta app and their own System User token.
