# Backend/hosting stack research

> This is the first file under `docs/research/` — this is a brand-new repo with no prior
> research-notes convention, so this establishes one: one Markdown file per research ticket,
> named for the topic, findings cited to primary sources, ending in a recommendation.

Resolves the research question on [issue #3](../../../../issues/3) (child of the wayfinder map,
[issue #1](../../../../issues/1)).

## Requirements recap

From [CONTEXT.md](../../CONTEXT.md) and issue #1: multi-tenant with fully isolated data per
Account (an invited person's isolated space — no cross-Account visibility); passwordless
magic-link auth; a PWA frontend; a weekly scheduled job that recalculates each Account's Calorie
Target from their Weight Trend; low/no cost at personal-family scale (a handful of invited users,
not a commercial product).

## Candidates compared

### Supabase

- **Magic-link auth**: Native. Supabase Auth ships passwordless email login via Magic Link (and
  OTP) out of the box — "Supabase Auth offers two passwordless login methods that use the user's
  email address: Magic Link... OTP." No third-party auth library needed.
  Source: https://supabase.com/docs/guides/auth/auth-email-passwordless
- **Scheduled/background jobs**: Native. **Supabase Cron** is a Postgres module for scheduling
  recurring jobs with cron syntax, runnable "anywhere from every second to once a year," either
  running SQL/DB functions directly or making an HTTP call (e.g. to invoke an Edge Function).
  Source: https://supabase.com/docs/guides/cron — a weekly Calorie Target recalculation is a
  one-line `cron.schedule(...)` call or a dashboard entry. (The `pg_cron` extension page now
  redirects to this Cron product docs page: https://supabase.com/docs/guides/database/extensions/pg_cron)
- **Postgres + per-tenant isolation**: It's Postgres, and Supabase's own docs recommend Postgres
  Row Level Security (RLS) as the mechanism: "When you need granular authorization rules, nothing
  beats Postgres's Row Level Security (RLS)... Supabase allows convenient and secure data access
  from the browser" using RLS policies scoped by `auth.uid()`.
  Source: https://supabase.com/docs/guides/database/postgres/row-level-security — this maps
  directly onto "fully isolated data per Account": one `account_id` column + one RLS policy per
  table, enforced by Postgres itself, not application code.
- **Cost at personal-family scale**: Free plan is $0/month: unlimited API requests, **50,000
  monthly active users**, 500 MB database, 5 GB egress, 1 GB file storage, 200 concurrent
  Realtime connections, 500K Edge Function invocations. The only catch: "Free projects are paused
  after 1 week of inactivity. Limit of 2 active projects." Pro is $25/month if/when the free
  limits are outgrown. Source: https://supabase.com/pricing — at a handful of family users, the
  free tier's 50,000 MAU and 500 MB ceiling are effectively unreachable; the only practical
  friction is the pause-after-a-week-idle behavior (a scheduled cron ping or just regular app
  usage prevents this).
- **Setup effort**: Low. One hosted product covers Postgres + Auth + Cron + (optionally) storage
  and edge functions — no separate services to wire together, and RLS + magic link are both
  documented, first-class features rather than assembled from parts.

### Firebase

- **Magic-link auth**: Native. Firebase Authentication supports "Email Link (Passwordless)
  Sign-in" directly, documented at
  https://firebase.google.com/docs/auth/web/email-link-auth?hl=en — sends a sign-in link to the
  user's email, no password.
- **Scheduled/background jobs**: Native, via Cloud Functions' `onSchedule` (2nd gen), backed by
  Cloud Scheduler under the hood: "Both Unix Crontab and App Engine syntax are supported by Cloud
  Scheduler." Source:
  https://firebase.google.com/docs/functions/schedule-functions?hl=en&gen=2nd — a weekly
  recalculation job is a standard scheduled Cloud Function.
- **Postgres or equivalent with per-tenant isolation**: **No native Postgres.** Firebase's
  primary database is Firestore, a NoSQL document store organized as collections and
  (sub)collections, not relational tables — "The best way to store messages in this scenario is
  by using subcollections... A subcollection is a collection associated with a specific document."
  Source: https://firebase.google.com/docs/firestore/data-model?hl=en. Per-tenant isolation is
  possible (Firestore Security Rules keyed on `request.auth.uid`, or a subcollection-per-Account
  layout) but it is rules-based document filtering, not RLS on a relational schema, and it's a
  materially different data model than "Postgres or equivalent relational storage" — nutrient
  targets, food entries, and weight-trend aggregation are naturally relational/tabular, which
  Firestore does not model natively. Google does offer a real Postgres via Cloud SQL through
  Firebase's "SQL Connect," but it's a separate paid product layered on top (3-month free trial,
  then Cloud SQL pricing "starting as low as $9.37/month" after the trial) — source:
  https://firebase.google.com/pricing (Firebase Pricing page, SQL Connect row) — so a genuinely
  relational, RLS-isolated Firebase stack isn't really "free."
- **Cost at personal-family scale**: Spark (free) plan: Firestore gives 1 GiB stored data, 50,000
  document reads/day, 20,000 writes/day, 20,000 deletes/day, 10 GiB outbound transfer/month
  (source: https://cloud.google.com/firestore/pricing). Cloud Functions (2nd gen, billed as Cloud
  Run under the hood) has its own free tier — "CPU - First 180,000 vCPU-seconds free per month...
  RAM - First 360,000 GiB-seconds free per month... Requests - 2 million requests free" (source:
  https://cloud.google.com/run/pricing) — but **using scheduled Cloud Functions at all requires
  the Blaze (pay-as-you-go) plan**, which needs a billing account attached even though the free
  quotas still apply on top of Blaze. So Firebase can run at $0 in practice for this app's tiny
  scale, but only after enabling billing on a card, unlike Supabase/Neon where the free tier needs
  no card at all.
- **Setup effort**: Medium-high for this specific app. Auth and scheduled functions are easy, but
  the data model mismatch (NoSQL vs. the app's clearly relational Nutrient Target/Food
  Entry/Weight Trend structure) means either fighting Firestore's document model or paying for
  Cloud SQL — added complexity a solo non-expert builder doesn't need.

### Self-hosted-style: Postgres (Neon) + Next.js on Vercel

This is the realistic shape of "self-hosted Postgres + Node/Next.js stack" today: Vercel no
longer offers a first-party Postgres product — its storage docs point to Blob/Global Config plus
a **Marketplace** of database providers (Neon, Supabase, etc.) installed as integrations:
`vercel install neon` (source: https://vercel.com/docs/storage). So this candidate is really
**Next.js on Vercel + Neon Postgres + Auth.js + a transactional-email provider**, assembled from
separate best-of-breed pieces rather than one platform.

- **Magic-link auth**: Not native to Vercel or Neon — requires **Auth.js** (formerly NextAuth)
  with its Email provider. Auth.js's own docs are explicit about the two extra pieces this needs:
  a database adapter ("a database is required for passwordless login to work as verification
  tokens need to be stored") and an SMTP-capable mailer via Nodemailer ("Auth.js does not include
  `nodemailer` as a dependency, so you'll need to install it yourself... You will need access to
  an SMTP server"). Source: https://authjs.dev/getting-started/authentication/email. In practice
  that SMTP server is a service like Resend.
- **Scheduled/background jobs**: Native to Vercel via **Cron Jobs**, available on every plan,
  including Hobby: "Cron Jobs are available on all plans." Source:
  https://vercel.com/docs/cron-jobs. The catch on the free Hobby plan: "Hobby accounts are limited
  to cron jobs that run once per day... Timing precision: Vercel cannot assure a timely cron job
  invocation... a cron job configured as `0 1 * * *` will trigger anywhere between 1:00 am and
  1:59 am." Source: https://vercel.com/docs/cron-jobs/usage-and-pricing. A once-a-week job with
  loose timing precision is exactly what this app needs, so the Hobby-plan restriction is not a
  real constraint here.
- **Postgres + per-tenant isolation**: Real Postgres (Neon is standard Postgres under the hood),
  so RLS is available exactly as documented by PostgreSQL itself: "tables can have row security
  policies that restrict, on a per-user basis, which rows can be returned by normal queries or
  inserted, updated, or deleted." Source:
  https://www.postgresql.org/docs/current/ddl-rowsecurity.html. Same RLS-per-Account pattern as
  the Supabase option, just self-assembled instead of built in.
- **Cost at personal-family scale**:
  - *Vercel Hobby*: $0/month, but the docs are explicit it's "for personal, non-commercial use"
    (source: https://vercel.com/pricing FAQ) — which this app is.
  - *Neon Free plan*: $0/month — "100 projects, 10 branches per project, 100 CU-hours of compute
    per project per month, autoscaling up to 2 CU (≈8 GB RAM), 0.5 GB of storage per project, and
    5 GB of public network transfer per month... Scale to zero is always enabled (computes
    suspend after 5 minutes of inactivity)." Source: https://neon.com/docs/introduction/plans. At
    a handful of family users this is very unlikely to be exceeded; if it is, Launch-tier compute
    is $0.106/CU-hour and storage $0.35/GB-month (same source) — Neon's own worked example puts
    "light usage" at about $2.31/month.
  - *Email for magic links*: needs an SMTP/API provider. Resend's Free plan allows a 100
    email/day sending limit with 1 custom domain (source: https://resend.com/pricing comparison
    table) — more than enough for a handful of Accounts logging in occasionally.
  - Net: $0/month realistically, assembled from three separate free tiers instead of one.
- **Setup effort**: Highest of the three. Nothing is wrong per se, but it's four services to
  provision and wire together (Vercel + Neon + Auth.js + Resend/SMTP), each with its own
  dashboard, environment variables, and failure modes — more moving parts for a solo,
  non-expert builder than Supabase's single integrated product.

### Other options considered

- **Railway**: Good general app host (Postgres databases, cron jobs — "For short-lived tasks that
  complete quickly and exit properly, such as a daily database backup," source:
  https://docs.railway.com/reference/cron-jobs) but no built-in auth product, so magic-link auth
  would still need Auth.js + a mailer, same as the Vercel/Neon option, with less generous
  free-tier framing: "Start with a 30-day free trial with $5 credits, then $1 per month... Up to
  1 vCPU / 0.5 GB RAM per service." Source: https://railway.com/pricing. After the trial there's
  no perpetual full-featured free tier the way Supabase/Neon/Vercel Hobby offer — it becomes a
  small recurring cost. Not a clear improvement over Vercel+Neon for this app, so not pursued
  further.
- **Fly.io**: Similar story — good general compute host, but Postgres is now split into a
  "Managed Postgres" paid product and an explicitly unsupported "Fly Postgres (Unmanaged)" DIY
  option ("We are not able to provide support or guidance for unmanaged Postgres," source:
  https://fly.io/docs/postgres/), no built-in auth, and only a resource-limited free trial rather
  than an ongoing free tier (source: https://fly.io/docs/about/free-trial/). More ops burden than
  this project needs.

## Summary comparison

| | Magic-link auth | Scheduled jobs | Relational + tenant isolation | Cost at family scale | Setup effort |
|---|---|---|---|---|---|
| **Supabase** | Native (Auth) | Native (Supabase Cron) | Native Postgres + RLS | $0, no card required | Low — one product |
| **Firebase** | Native (Auth) | Native (Cloud Functions `onSchedule`) | No — Firestore is NoSQL; real Postgres is a separate paid Cloud SQL add-on | $0 achievable, but requires enabling Blaze billing (card on file) | Medium-high — data-model mismatch |
| **Vercel + Neon (+ Auth.js + Resend)** | Via Auth.js + SMTP mailer, not native | Native (Vercel Cron Jobs) | Native Postgres + RLS | $0, assembled from 3 free tiers | Highest — 4 services to wire up |
| Railway | Via Auth.js + SMTP, not native | Native (cron jobs) | Native Postgres + RLS | Free trial only, then ~$1+/mo | Medium |
| Fly.io | Via Auth.js + SMTP, not native | Would need to build/host it | Postgres available but unmanaged-DIY or paid Managed Postgres | Free trial only, no ongoing free tier | High |

## Recommendation

**Supabase.** It's the only candidate where every one of this app's requirements — passwordless
magic-link auth, a weekly scheduled job, Postgres with per-tenant RLS isolation, and a genuinely
$0 free tier that needs no credit card — is a native, documented, first-class feature of a single
product, rather than something assembled from two or more separate services. For a solo,
non-expert builder shipping a personal/family-scale PWA, that collapses setup to "one account,
one project" instead of coordinating auth, database, cron, and email providers separately (the
Vercel+Neon path) or fighting a NoSQL data model that doesn't match this app's clearly relational
domain (Firebase/Firestore). The free-tier limits (50,000 MAU, 500 MB DB, 2 active projects,
pause-after-a-week-idle) are all well beyond what a handful of invited Accounts will ever
approach, and the only mitigation needed — keeping the project from idling a full week — is
trivially satisfied by the weekly Cron job itself hitting the database.

If Supabase ever stopped fitting (e.g. wanting Firestore's realtime multi-device sync model, or
wanting to fully own every layer of the stack), the Vercel + Neon + Auth.js combination is the
credible fallback — same underlying Postgres/RLS story, still $0 at this scale, just more parts
to own.
