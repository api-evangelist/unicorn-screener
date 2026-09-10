---
name: unicorn-screener-request-memo
description: Request a new asynchronous Unicorn Screener research memo for a startup and poll its screening status to completion.
api: Unicorn Screener API
generated: '2026-09-10'
method: generated
source: Grounded in operationIds from openapi/unicorn-screener-openapi.json; no operation invented.
operations:
- lookupStartup
- requestStartupMemo
- getScreeningStatus
---

# Request a startup research memo (async)

Use this skill to generate a fresh research memo when no existing result is
available. This consumes the caller's one free screening allowance, so only run
it with the user's explicit authorization.

## Steps

1. **Check for an existing memo first.** Call `lookupStartup` —
   `GET /api/agent/lookup?name=<exact name>`. If `found:true` and `analyzedAt`
   is recent enough, stop — no new screening is needed.

2. **Request the screening.** Call `requestStartupMemo` —
   `POST /api/request-report` with a JSON body:
   `startupName`, a real user-authorized non-disposable `email`, a stable
   persisted `fingerprint`, and `newsletter:false` unless the user explicitly
   opted in. This endpoint does not accept a payment token.
   - On `200`, read `success`, `message`, and the optional `slug` / `memoUrl`.
   - If no `slug` is returned, wait for the email link — acceptance is not
     completion.

3. **Poll status to a terminal state.** With the returned `slug`, call
   `getScreeningStatus` — `GET /api/screen-status?slug=<slug>` every 10 seconds
   with a bounded timeout.
   - `running` / `refreshing`: keep polling.
   - `moved`: follow `publishedSlug` and continue polling that slug.
   - `ready`: done — surface the memo (`memoUrl`).
   - `failed` / `refresh-failed`: terminal failure (a failed refresh preserves
     the previous memo).
   - `unknown` (`404`): stop.

## Rules (idempotency & quotas)

- **Never repeat the POST while polling** and never blindly retry a `500` — a
  memo may already be queued; duplicate POSTs waste the allowance and can create
  duplicate work (there is no Idempotency-Key header).
- One free screening per caller, enforced across email + fingerprint + cookie;
  shared IPs also have a daily cap. On `429`, read `reason`
  (`email|fingerprint|cookie|ip`), back off per `resetInSeconds`, and direct the
  user to the website for paid options ($9/report) — never rotate identity to
  evade limits.
- A screening request is **not reversible** — there is no cancel or undo — so
  confirm the target company before requesting.
