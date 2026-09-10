---
name: unicorn-screener-lookup-startup
description: Look up an existing Unicorn Screener startup score and summary for free, resolving the company identity first when the name is ambiguous.
api: Unicorn Screener API
generated: '2026-09-10'
method: generated
source: Grounded in operationIds from openapi/unicorn-screener-openapi.json; no operation invented.
operations:
- findStartupSuggestions
- lookupStartup
---

# Look up a startup score (free)

Use this skill to retrieve an existing unicorn-potential score and summary for a
startup without launching a new (allowance-consuming) screening. The API is keyless.

## Steps

1. **Resolve the identity if the name is ambiguous.** Call
   `findStartupSuggestions` — `GET /api/autocomplete?q=<fragment>` (2-60 chars).
   Pick the candidate whose `domain` matches the company you mean. An empty
   `results` array does not prove the company is absent — it can also mean an
   invalid length or throttling.

2. **Fetch the existing result.** Call `lookupStartup` —
   `GET /api/agent/lookup?name=<exact name>`. The name is normalized to a slug,
   not fuzzy-matched, so pass the exact name (use the suggestion's `name`).
   - On `found:true`, read `score` (0-100), `classification`, `summary`,
     `website`, `hq`, and `analyzedAt`.
   - On `found:false`, do **not** treat it as a negative assessment — it only
     means no memo has been generated yet.

3. **Respect freshness and caching.** Check `analyzedAt`; cache the response and
   avoid re-requesting the same company repeatedly.

## Rules

- Rate limit: 60 lookups/hour/IP. On `429`, back off and honor `resetInSeconds`
  when present; never rotate IP to evade the limit.
- This is read-only research and consumes no screening allowance.
- Scores support research and are not investment advice.
