---
name: Estimate then run a Quandela Quantum Toolbox algorithm
description: >-
  Use Quandela Cloud's five pre-built quantum primitives (Chemistry VQE, Custom
  VQE, CVaR VQE, Graph DSI, Graph Isomorphism) without writing a photonic
  circuit — price the workload with the paired /estimate operation first, then
  submit, poll and collect results.
api: openapi/quandela-quantum-toolbox-openapi.yml
generated: '2026-08-17'
method: generated
source: >-
  openapi/_original/quandela-quantum-toolbox-openapi.json,
  openapi/_original/quandela-cloud-openapi.json,
  sandbox/quandela-sandbox.yml, errors/quandela-problem-types.yml,
  rate-limits/quandela-rate-limits.yml
operations:
  - post_qt_chemistryvqe_estimate
  - post_qt_chemistryvqe
  - get_qt_chemistryvqe_by_job_id_status
  - get_qt_chemistryvqe_by_job_id_results
  - post_qt_customvqe_estimate
  - post_qt_customvqe
  - get_qt_customvqe_by_job_id_status
  - get_qt_customvqe_by_job_id_results
  - post_qt_cvarvqe_estimate
  - post_qt_cvarvqe
  - get_qt_cvarvqe_by_job_id_status
  - get_qt_cvarvqe_by_job_id_results
  - post_qt_graphDSI_estimate
  - post_qt_graphDSI
  - get_qt_graphDSI_by_job_id_status
  - get_qt_graphDSI_by_job_id_results
  - post_qt_graphIsomorphism_estimate
  - post_qt_graphIsomorphism
  - get_qt_graphIsomorphism_by_job_id_status
  - get_qt_graphIsomorphism_by_job_id_results
  - get_api_jobs_availability
---

# Run a Quandela Quantum Toolbox algorithm

Base URL: `https://api.cloud.quandela.com/`
Auth: `Authorization: Bearer <Cloud Job Token>`

## Why this surface, not `/api/jobs`

`POST /api/jobs` takes an **opaque, SDK-serialised** photonic circuit — you
effectively need `perceval-quandela` to build the body. The Quantum Toolbox
(`/qt/*`) is the opposite: each algorithm has fully typed request, status,
intermediate-result and result schemas in the OpenAPI, so **an agent can call it
directly over HTTP with no SDK.** If a Toolbox primitive fits the problem, prefer
it.

## The five algorithms

Every one follows the identical four-operation shape at `/qt/<algorithm>`:

| Algorithm | Path | Params schema | Results schema |
|---|---|---|---|
| Chemistry VQE | `/qt/chemistryvqe` | `ChemistryVqeParams` (contains `Atom[]`) | `ChemistryVqeResults` |
| Custom VQE | `/qt/customvqe` | `CustomVqeParams` | `CustomVqeResults` |
| CVaR VQE | `/qt/cvarvqe` | `CVarVqeParams` | `CVarVqeResults` |
| Graph DSI (dense subgraph) | `/qt/graphDSI` | `GraphDSIParams` | `GraphDSIResults` |
| Graph Isomorphism | `/qt/graphIsomorphism` | `GraphIsomorphismParams` | `GraphIsomorphismResults` |

## Step 1 — ALWAYS estimate first

This is the one genuinely agent-friendly affordance on this API and you should
never skip it. Each algorithm has a paired estimator that returns cost **without
running anything and without spending credits**:

```
POST /qt/<algorithm>/estimate
Authorization: Bearer <Cloud Job Token>
{ ...same params as the real call... }
```

Returns `EstimationResult`:

```
{ "nb_iterations": <int>, "nb_shots_per_job": <int>, "nb_total_shots": <int> }
```

Use `nb_total_shots` as your budget gate. If it exceeds what you are authorised
to spend, shrink the problem and estimate again — do not submit and hope. There
is no equivalent estimator for arbitrary Perceval jobs, so this is a reason to
stay on the Toolbox.

## Step 2 — Check your concurrency headroom

Toolbox jobs have their **own** ceiling, separate from Perceval jobs:

```
GET /api/jobs/availability
```

Read `max_running_qt_jobs` and `num_running_qt_jobs`. Exceeding it returns
`403 "Cannot have more than {MAX_QT_JOBS} running Quantum Toolbox jobs"` — note
that this ceiling is a **403**, while the Perceval waiting-job ceiling is a
**400**. There are no rate-limit headers to read.

Also check platform status anonymously at
`GET https://api.cloud.quandela.com/api/platforms/public` before committing.

## Step 3 — Submit

```
POST /qt/<algorithm>
Authorization: Bearer <Cloud Job Token>
{ ...params... }
```

Returns `QTJobId` → `{job_id}` (`job_id` is required in this schema, so it is
always present).

There is **no idempotency key** here either. A timed-out submit may or may not
have created a job, and unlike `POST /api/jobs` the Toolbox has no `process_id`
guard at all — so do not blind-retry. Poll instead: if you lost the `job_id`,
you cannot recover it over REST (there is no job list endpoint), so persist the
`job_id` the moment you receive it.

## Step 4 — Poll status

```
GET /qt/<algorithm>/{job_id}/status
```

Returns `<Algorithm>JobStatus`. The VQE algorithms additionally expose
`Intermediate<Algorithm>VqeResults`, so a status poll can carry the partial
optimisation trace — useful for showing convergence, and for deciding to cancel
early. Poll with exponential backoff; there is no `Retry-After` and no `429`.

## Step 5 — Collect results

```
GET /qt/<algorithm>/{job_id}/results
```

Returns `<Algorithm>JobResult` wrapping `<Algorithm>Results`.

## Choosing an algorithm

- **Chemistry VQE** — ground-state energy of a molecule. `ChemistryVqeParams`
  carries an `Atom[]` molecular specification.
- **Custom VQE** — bring your own cost Hamiltonian / ansatz.
- **CVaR VQE** — conditional-value-at-risk objective; the variant to reach for on
  combinatorial optimisation where you care about the tail.
- **Graph DSI** — dense subgraph identification.
- **Graph Isomorphism** — graph equivalence testing.

## Errors

Same vendor envelope as the rest of the API: `{"detail": ..., "error": "..."}`,
no machine-readable code, no RFC 9457. Toolbox-specific messages:

| Status | `error` contains | Do |
|---|---|---|
| 403 | `Cannot have more than {MAX_QT_JOBS} running` | Wait for a running QT job to finish |
| 403 | `No permission to use algorithm` | Your offer excludes it — escalate to a human |
| 400 | `Quantum Toolbox command '{command}' is not defined` | Wrong algorithm name |
| 400 | `Invalid iterator: must be a non-empty list` | Fix the params array |
| 401 | `Invalid or expired job token` | Mint/reopen the job token |
| 401 | `Not enough credits` | **Quota, not auth.** Stop; do not refresh the token |
| 422 | validation | Read `detail.<location>.<field>` |

Remember: credit exhaustion arrives as `401`, so never treat `401` as a signal to
refresh a credential without first reading the `error` string.
