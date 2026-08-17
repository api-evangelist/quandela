---
name: Mint, rotate, audit and revoke Quandela Cloud Job Tokens
description: >-
  Manage the credential lifecycle on Quandela Cloud — generate a labelled,
  time-boxed Cloud Job Token per workload, list and inspect tokens, adjust
  priority, read the per-platform consumption ledger, then revoke, reopen or
  delete. Includes the deprecated singular paths to avoid.
api: openapi/quandela-perceval-job-token-openapi.yml
generated: '2026-08-17'
method: generated
source: >-
  openapi/_original/quandela-cloud-openapi.json,
  authentication/quandela-authentication.yml,
  lifecycle/quandela-lifecycle.yml, errors/quandela-problem-types.yml
operations:
  - post_api_tokens
  - get_api_tokens
  - get_api_tokens_by_token_id
  - put_api_tokens_by_token_id
  - post_api_tokens_revoke
  - post_api_tokens_reopen
  - post_api_tokens_delete_by_ids
  - post_api_tokens_delete
  - get_api_tokens_usage_report
  - post_api_auth_tokens
  - get_api_auth_tokens
  - get_api_auth_tokens_by_token_id
  - put_api_auth_tokens_by_token_id
  - post_api_auth_tokens_revoke
  - post_api_auth_tokens_reopen
  - post_api_auth_tokens_delete_by_ids
---

# Manage Quandela Cloud Job Tokens

Base URL: `https://api.cloud.quandela.com/`

## The two credentials

One `BearerAuth` scheme, two different credentials — the contract does not
distinguish them, so get this right first:

| Credential | Where it comes from | What it is for | 401 message |
|---|---|---|---|
| **Account access token** | interactive login at `account.quandela.com` | managing tokens (the operations in this skill) | `Authentication and authorization failed: token not found` |
| **Cloud Job Token** | `POST /api/tokens` | submitting jobs, `/qt/*`, QRNG | `Invalid or expired job token` |

Acquiring the account access token is **not** documented publicly —
`account.quandela.com` serves no OIDC or OAuth discovery document, so that step
needs a human at a browser. Everything after it is fully self-service.

## Why rotate at all

This API has no scopes and no permissions model: a Cloud Job Token is
all-or-nothing within the account's commercial offer. The only blast-radius
control available to you is **token granularity** — mint a short-lived, labelled,
low-priority token per workload and revoke it afterwards. Because revoke,
reopen and delete are all API operations, an agent can do this without human
involvement, which makes per-run tokens genuinely practical here.

## Mint a token

`post_api_tokens`:

```
POST /api/tokens
Authorization: Bearer <account access token>
{
  "label": "agent-run-2026-08-17-a",
  "priority": 0,
  "duration": 3600,
  "is_explorer_token": false,
  "user_id": "<optional, for delegated minting>"
}
```

Returns `TokenGenerateResponse` → `{token, token_id, duration}`. `token` is
shown here and used as the bearer on the job surface; the list projection later
exposes only `key`, so **capture `token` now**.

Constraints, all enforced with `400`:

- `label` must be unique — `"Label already exists: {label}"`.
- `priority` cannot exceed your own user priority —
  `"Can't assign priority higher than user priority"`.
- An unknown `user_id` returns `"Unknown user"`.

Do **not** set `is_explorer_token: true` for an agent workload: explorer tokens
are rejected outright on some operations with
`401 "Authentication failed: explorer tokens are not allowed"`.

## List and inspect

`get_api_tokens`:

```
GET /api/tokens?limit=50&offset=0&include_expired=false&include_revoked=false
```

Returns `TokensListUnitResponse[]` → `id`, `key`, `label`, `priority`,
`user_id`, `count`, `created_date`, `expiration_date`, `last_used_time`,
`is_revoked`. This is the API's **only** paginated collection — limit/offset,
no cursor.

`get_api_tokens_by_token_id` — `GET /api/tokens/{token_id}` — returns the fuller
`Token` object: adds `email`, `company_id`, `company_name`, `valid`,
`is_apikey`, `is_explorer_token`, `user_priority` and `last_used_duration`.

Use `last_used_time` and `count` to find stale tokens worth revoking.

## Update

`put_api_tokens_by_token_id`:

```
PUT /api/tokens/{token_id}
{"label": "<new label>", "priority": <int>}
```

Only `label` and `priority` are mutable. You cannot extend a token's expiry —
mint a new one instead.

## Audit consumption

`get_api_tokens_usage_report`:

```
GET /api/tokens/usage/report?token_ids=<id,id>&start_time=<ts>&end_time=<ts>&type=<...>
```

Returns `TokenUsageResponse` → `{token_id, total_jobs, usage}` where each
`TokenUsageUnit` is **broken down by platform**: `platform_name`, `nb_jobs`,
`duration_seconds`, `credits`, `free_credits`.

This is the closest thing Quandela publishes to a billing API, and it is the only
way to answer "how much has this agent spent, and on which QPU" — the pricing
page publishes no rates, so consumption is knowable but cost is not.
`free_credits` being separate from `credits` is how you tell whether you are
still inside the free allocation.

## Revoke, reopen, delete

```
POST /api/tokens/revoke          # body: RevokeToken {token}   — required
POST /api/tokens/reopen          # bring a revoked token back
POST /api/tokens/delete-by-ids   # body: ListTokenIds — permanent
```

Revocation is **reversible** via reopen, which is unusual and worth knowing: a
revoked token is suspended, not destroyed. Use `delete-by-ids` when you want it
gone for good.

Guards:

- `400 "Can't not revoke"` — token is not in a revocable state.
- `400 "Tokens not found | Can't delete the current active token"` — you cannot
  delete the token you are presently authenticating with.
- `403 "You do not have permission to delete this token"` — it belongs to
  another user.

`post_api_tokens_delete` (`POST /api/tokens/delete`, body `DeleteTokensByKey`)
deletes by key rather than id — same guards.

## Do not use the deprecated paths

Ten operations in this contract are marked `deprecated: true`. On the token
surface, prefer the plural form:

| Deprecated | Use instead |
|---|---|
| `POST /api/auth/token/generate` | `POST /api/auth/tokens` |
| `POST /api/auth/token/revoke` | `POST /api/auth/tokens/revoke` |
| `GET /api/auth/token/{token_id}` | `GET /api/auth/tokens/{token_id}` |
| `POST /api/auth/token/delete` | `POST /api/auth/tokens/delete-by-ids` |
| `POST /api/auth/tokens/delete` | `POST /api/auth/tokens/delete-by-ids` |

Quandela publishes **no sunset dates and no deprecation policy**, and sends no
`Sunset` or `Deprecation` response headers — the `deprecated` flag in the spec is
the only warning you get. Treat these as removable without notice.

Note the two parallel families: `/api/tokens*` (tag *Api - Perceval Job Token*)
and `/api/auth/tokens*` (tag *Api - Job Token*). They overlap heavily and neither
is documented as canonical. This skill uses `/api/tokens*`; the `/api/auth/tokens*`
equivalents are listed in the frontmatter if you need them.
