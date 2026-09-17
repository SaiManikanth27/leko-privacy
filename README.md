# LEKO

**Your money. Clearly.**

A personal finance app for iOS. Record what you spend, see where it
went, and once a month decide what to do with whatever is left.

Built by one person, mostly to find out whether it could be done.

The source is private; this page is here to explain what the app is and
how it works.

---

## What it is, and what it isn't

LEKO is an **expense tracker**, not a budgeting app. It shows where
money went; it never asks you to set a target and then tells you off
for missing it. Category percentages are the insight. There are no
budgets, no limits, no streaks, and nothing that grades your month.

That is a deliberate stance rather than a missing feature.

---

## What it does

- **Accounts** — cash, bank and credit cards, in USD or INR, kept in
  separate sections and never summed into one number. "How much do I
  have in this country" is the useful question.
- **Transactions** — expenses, income and transfers, with categories,
  merchants, notes and dates. Swipe to delete, with a five second undo.
- **Recurring** — bills and subscriptions that post themselves when due.
- **Income** — paycheck logging that records the work period each one
  covers, not just the day it landed.
- **Expenses** — separates what you have actually spent from what is
  merely scheduled, so a bill dated next week never counts as spent and
  never inflates a total.
- **Insights** — monthly totals and a category breakdown as a donut.
- **App lock** — optional Face ID / Touch ID / passcode on launch.
- **Bank sync** — imports accounts and transactions through Plaid. Built
  and tested, currently switched off; see below.

---

## How it is built

**Expo SDK 54 / React Native 0.81**, New Architecture enabled. iOS
first; the Android path exists but is untested.

**Supabase** for Postgres, auth and Edge Functions. Every table has
row-level security, so a signed-in user can only ever read and write
their own rows. Balance-affecting writes go through `SECURITY DEFINER`
RPCs so a transaction and the balance it moves change together or not
at all.

**No analytics, no advertising, no tracking, no third-party SDKs.** Not
a feature to be added later — there is nowhere for your data to go.

### Layout

```
App.js                 screen routing, session, pending-delete state
theme.js               design tokens — colours, type scale, radii
db.js                  every query and RPC the app makes
lib/                   supabase client, plaid, profile, app lock, haptics
screens/               one file per screen
components/            shared UI — cards, chips, rows, charts
supabase/schema.sql    tables, RLS policies, RPCs. Idempotent.
supabase/functions/    Edge Functions (Deno)
docs/design-brief.md   the design system and the decisions behind it
```

There is no navigation library. `App.js` holds a `screen` string and
renders one component — fine at this size, and one less dependency to
keep current.

---

## Bank sync

Plaid support is complete and tested against Sandbox, but ships turned
off behind `BANK_SYNC_ENABLED` in `lib/features.js`. Plaid Production
access is a separate approval process with an unpredictable timeline,
and an app that connects to real bank accounts also draws heavier App
Store review. Neither should delay releasing the manual app, which
stands on its own.

It runs through **Hosted Link** rather than Plaid's native SDK, which
keeps the app free of Plaid native code. Credentials never reach the
device: the Edge Functions hold the Plaid keys, and `plaid_items` grants
nothing to the `authenticated` role, so stored bank access tokens are
unreadable from the client by construction rather than by policy.

---

## Running it

```sh
npm install
npx expo start
```

Expo Go works for everything except Face ID, the app icon and bank
sync, all of which need a development build:

```sh
eas build --platform ios --profile development
npx expo start --dev-client
```

Adding a native module always means a new build. JavaScript changes
reload instantly; native code does not.

### Backend

`supabase/schema.sql` and `supabase/plaid_schema.sql` are both
idempotent — paste either into the Supabase SQL Editor and run. Edge
Functions in `supabase/functions/` are deployed from the Supabase
dashboard.

The Plaid functions expect `PLAID_CLIENT_ID`, `PLAID_SECRET` and
`PLAID_ENV` as Edge Function secrets. `SUPABASE_URL`,
`SUPABASE_ANON_KEY` and `SUPABASE_SERVICE_ROLE_KEY` are provided by the
Supabase runtime.

---

## Privacy

[Privacy policy](https://saimanikanth27.github.io/leko-privacy/)

Short version: your email is used to sign you in, your financial data is
what you type, nothing else is collected, and **Settings → Delete my
account** removes all of it immediately without emailing anyone to ask.
