---
name: Submit a Perceval job to a Quandela QPU and collect the result
description: >-
  Mint a Cloud Job Token, check that the target platform and your account
  actually have capacity, submit a photonic-circuit job to Quandela Cloud, poll
  it to completion, and retrieve the result — with the credit-safety and
  retry rules this API's error semantics demand.
api: openapi/quandela-perceval-job-openapi.yml
generated: '2026-08-17'
method: generated
source: >-
  openapi/_original/quandela-cloud-openapi.json (operationIds derived in
  overlays/quandela-cloud-overlay.yaml), conventions/quandela-conventions.yml,
  errors/quandela-problem-types.yml, rate-limits/quandela-rate-limits.yml,
  sandbox/quandela-sandbox.yml
operations:
  - post_api_tokens
  - get_api_jobs_availability
  - post_api_jobs
  - get_api_jobs_by_job_id_status
  - get_api_jobs_by_job_id_result
  - get_api_jobs_by_job_id_data
  - post_api_jobs_by_job_id_cancel
  - post_api_jobs_by_job_id_rerun
  - post_api_tokens_revoke
---

# Submit a Perceval job to Quandela Cloud

Base URL: `https://api.cloud.quandela.com/`

## Before you start — read this or you will waste credits

Four facts about this API change how you must behave:

1. **There is no idempotency.** No `Idempotency-Key` exists. If `POST /api/jobs`
   times out, you do **not** know whether the job was created. A blind retry can
   consume credits twice. Set a `process_id` on every submission (see step 4) so
   a duplicate is *rejected* rather than silently duplicated.
2. **`401` does not always mean "bad token".** Quandela returns `401` for credit
   and quota exhaustion as well as for authentication failure. An agent that
   handles `401` by refreshing its credential will loop forever on
   `"Not enough credits"`. Always branch on the `error` string.
3. **`sim:` vs `qpu:` is the only guard.** The prefix on `platform_name` is what
   separates a free-ish simulator rehearsal from real hardware that spends
   credits. Nothing in the contract stops you sending `qpu:` when you meant
   `sim:`. Rehearse on `sim:` first.
4. **The QPUs are frequently unavailable.** On 2026-08-17 both published QPUs
   reported `unreachable` and `maintenance`. Check availability first (step 2).

## 1. Get a Cloud Job Token

Two credentials exist on one `BearerAuth` scheme. Token *administration* wants
the account access token from `account.quandela.com`; job *execution* wants a
**Cloud Job Token**. Mint one with `post_api_tokens`:

```
POST /api/tokens
Authorization: Bearer <account access token>
{"label": "agent-run-2026-08-17", "priority": 0, "duration": 3600}
```

Returns `TokenGenerateResponse` → `{token, token_id, duration}`. Use `token` as
the bearer on every step below.

Give every run its own labelled, short-lived token and revoke it at the end
(step 8). Labels must be unique — a collision returns `400 "Label already
exists"`. You cannot request a `priority` above your own user priority.

## 2. Check capacity before you submit

Two pre-flight checks, and both matter:

**Platform status** — anonymous, no token needed:

```
GET https://api.cloud.quandela.com/api/platforms/public
```

Returns each platform's `name`, `status`, a `maintenance` window and daily
`availability` percentages. Do not submit to a platform whose `status` is
`maintenance` or `unreachable`. (This endpoint is real and public but is *not*
in the OpenAPI.)

**Your own slots** — `get_api_jobs_availability`:

```
GET /api/jobs/availability
Authorization: Bearer <cloud job token>
```

Returns `max_concurrent_jobs` / `num_concurrent_jobs`,
`max_jobs_in_queue` / `num_jobs_in_queue`,
`max_running_qt_jobs` / `num_running_qt_jobs`. This is the **only** place the
`{MAX_WAITING_JOBS}` and `{MAX_QT_JOBS}` ceilings become concrete numbers —
there are no `RateLimit-*` headers on this API. If you are at the ceiling, wait
or cancel a queued job rather than submitting and eating a `400`.

## 3. Choose a platform

Address it by `platform_name` (namespaced string) or `platform_id` (UUID).
Supply exactly one, or you get `400 "Require platform_id or platform_name"`.

- `sim:...` — simulator/emulator. Use this to rehearse. Documented example:
  `sim:slos`.
- `qpu:...` — real hardware, spends credits. Observed: `qpu:belenos`,
  `qpu:ascella`.

## 4. Submit the job

`post_api_jobs`:

```
POST /api/jobs
Authorization: Bearer <cloud job token>
{
  "job_name": "<your label>",
  "payload":  { ... },
  "platform_name": "sim:slos",
  "max_shots": <int>,
  "max_duration": <seconds>,
  "pcvl_version": "1.2.4",
  "process_id": "<your own unique id>",
  "job_group_name": "<optional grouping key>"
}
```

Required: `job_name`, `payload`. The `payload` is an **opaque object serialised
by the Perceval SDK** — the OpenAPI does not describe its structure, so build it
with `perceval-quandela` (PyPI, 1.2.4, 2026-07-02) rather than by hand.

Budget guards you should always set:

- `max_shots` — per-platform ceiling; over it → `400 "Max shots must less than or equal to {shots_limit}"`.
- `max_duration` — seconds. Must be non-zero and ≤ `864000`; also subject to a
  per-platform ceiling.
- `process_id` — your idempotency *substitute*. A reused id returns
  `400 "process_id is existed and assigned to another user job"`. That is a
  rejection, **not** a replay of the original response — so on a timeout, re-send
  with the *same* `process_id`: a `400` on `process_id` tells you the first
  submission landed, and you should then find the job rather than create another.

Returns `JobId` → `{job_id}`.

## 5. Poll for completion

There are no webhooks, no callbacks and no event stream on this API. Polling is
the only mechanism. `get_api_jobs_by_job_id_status`:

```
GET /api/jobs/{job_id}/status
Authorization: Bearer <cloud job token>
```

Returns `JobStatusResponse` → `status`, `status_message`, `progress`,
`progress_message`, `creation_datetime`, `start_time`, `duration`, `shots`,
`failure_code`, `last_intermediate_results`.

Poll with exponential backoff. There is no `Retry-After` header to obey and no
`429`, so choose your own interval — start around 5 s and back off; quantum jobs
queue behind hardware availability and can sit for a long time.
`last_intermediate_results` lets you show partial progress before the job ends.

## 6. Fetch the result

`get_api_jobs_by_job_id_result`:

```
GET /api/jobs/{job_id}/result
```

Returns `JobResult` → `{job_id, results, results_type, intermediate_results,
duration, shots}`. Branch on `results_type` to interpret `results`.

For the full submission record — `command`, `payload`, `pcvl_version`,
`platform_name`, `token_label`, `rerun_from` — use
`get_api_jobs_by_job_id_data` (`GET /api/jobs/{job_id}/data`).

## 7. Cancel or rerun

- `post_api_jobs_by_job_id_cancel` — `POST /api/jobs/{job_id}/cancel`. Returns
  `400 "Can not cancel job"` if the job is not in a cancellable state. Cancel a
  *queued* job to free a slot when you hit the waiting-job ceiling.
- `post_api_jobs_by_job_id_rerun` — `POST /api/jobs/{job_id}/rerun`. The new job
  records the original in `rerun_from`.

## 8. Clean up

`post_api_tokens_revoke` — revoke the run's token when finished. It can be
brought back with `post_api_tokens_reopen` if needed, and hard-deleted with
`post_api_tokens_delete_by_ids`. Consumption for the token is available from
`get_api_tokens_usage_report`, broken down by platform with `credits` and
`free_credits` separated.

## Error handling rules

Errors are **not** RFC 9457. Every failure is
`{"detail": <object>, "error": "<english sentence>"}` with no machine-readable
code, so you must string-match on `error`.

| Status | `error` contains | What it means | Do |
|---|---|---|---|
| 401 | `token not found` | No/invalid account token | Re-authenticate at account.quandela.com |
| 401 | `Invalid or expired job token` | Job token dead | Reopen or mint a new one |
| 401 | `explorer tokens are not allowed` | Wrong token tier | Use a non-explorer token |
| 401 | `Not enough credits` | **Quota, not auth** | Stop. Do **not** refresh the token. Top up or shrink the job |
| 401 | `all remaining credits are reserved` | Credits locked by queued jobs | Cancel a queued job or wait |
| 401 | `Not enough {platform_type} time left` | Enterprise budget spent | Escalate to a human |
| 400 | `Cannot create more than` | Waiting-job ceiling | Wait or cancel; re-check `/api/jobs/availability` |
| 400 | `Max shots`/`Max duration` | Budget guard exceeded | Lower the value |
| 400 | `Create job disabled, Platform in` | Platform down/retired | Pick another platform |
| 400 | `process_id is existed` | Duplicate submission | The prior submit landed — find that job, do not resubmit |
| 403 | `No permission to use platform` / `is locked` | Offer does not include it | Escalate to a human |
| 404 | `Not found job` | Bad `job_id` | Check the id |
| 422 | validation | Field error | Read `detail.<location>.<field>` |

No `429` exists anywhere on this API, and no operation declares a `5xx`
response — treat any `5xx` as undocumented and retry with backoff.
