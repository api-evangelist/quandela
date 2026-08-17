---
name: Draw certified quantum randomness from Quandela Entropy (QRNG)
description: >-
  Fetch quantum random bytes or integers from Quandela's Entropy QRNG over HTTP,
  read the CHSH and min-entropy certification returned with every draw, and
  handle the entropy-pool-empty condition. The only synchronous operation on the
  Quandela Cloud API.
api: openapi/quandela-qrng-openapi.yml
generated: '2026-08-17'
method: generated
source: >-
  openapi/_original/quandela-cloud-openapi.json (QRNG tag,
  components.schemas JobBytes/JobInts/QrngBytesOutput/QrngIntsOutput), plus a
  live anonymous parameter-validation probe of
  https://api.cloud.quandela.com/qt/qrng/ints on 2026-08-17
operations:
  - get_qt_qrng_bytes
  - get_qt_qrng_ints
---

# Draw certified quantum randomness from Quandela Entropy

Base URL: `https://api.cloud.quandela.com/`
Auth: `Authorization: Bearer <Cloud Job Token>`

## Why this is different from every other operation on this API

Everything else on Quandela Cloud is submit-then-poll: you `POST` a job, get a
`job_id`, and poll for minutes or hours. **QRNG is synchronous.** Two `GET`
operations return values in the response body. If you need quantum randomness in
a request/response path, this is the one Quandela surface that fits.

It is also the only Quandela operation that returns its own **quality
attestation** alongside the data.

## The two operations

```
GET /qt/qrng/bytes    → QrngBytesOutput   (raw random bytes)
GET /qt/qrng/ints     → QrngIntsOutput    (random integers)
```

Both wrap a `jobs` array (required) of `JobBytes` / `JobInts`.

## Parameters — verified against the live endpoint

`GET /qt/qrng/ints` takes `count` and `type` as **query** parameters, both
required. Omitting them returns:

```
400 {"detail":{"query":{"count":["Missing data for required field."],
                        "type":["Missing data for required field."]}},
     "error":"Bad Request"}
```

`type` is a closed set. The live endpoint reports the permitted values itself:

```
400 {"detail":{"query":{"type":["Must be one of: uint8, uint16, uint32, uint64."]}},
     "error":"Bad Request"}
```

So: `type` ∈ `{uint8, uint16, uint32, uint64}`. (Sending `int8` is rejected —
unsigned only.)

```
GET /qt/qrng/ints?count=16&type=uint32
Authorization: Bearer <Cloud Job Token>
```

## Read the certification, not just the numbers

`JobInts` **requires** `chsh`, `id` and `min_entropy` alongside the optional
`ints`:

| Field | Meaning |
|---|---|
| `id` | draw identifier |
| `ints` | the random values |
| `chsh` | the measured CHSH (Bell-inequality) value for the draw |
| `min_entropy` | the certified min-entropy of the draw |

This is the point of a *quantum* RNG rather than a PRNG: the numbers arrive with
evidence of their own unpredictability. If you are using this for key material,
nonces, or anything auditable, **record `chsh` and `min_entropy` with the draw**
and check them against your own threshold. A draw whose min-entropy is below what
your use case requires should be discarded, not used — the API will hand you the
values either way; it does not gate on them for you.

## Handle the empty pool

The QRNG surface has an error the rest of the API does not:

```
404 {"detail":{},"error":"Entropy Pool empty"}
```

The hardware entropy pool is momentarily drained. This is transient and
expected under load — it is **not** a client error despite the `404`. Retry with
a short backoff, or request fewer bytes/ints. Do not treat it as a permanent
failure and do not fall back to a software PRNG silently: if certified randomness
was the requirement, degrading to `os.urandom` without saying so defeats the
purpose. Surface the condition to the caller.

Because there is no `Retry-After` header anywhere on this API, choose your own
interval — a few hundred milliseconds, then exponential backoff.

## Other errors

Standard vendor envelope, `{"detail": ..., "error": "..."}`:

| Status | `error` | Do |
|---|---|---|
| 400 | `Missing data for required field.` | supply both `count` and `type` |
| 400 | `Must be one of: uint8, uint16, uint32, uint64.` | fix `type` |
| 401 | `Invalid or expired job token` | mint/reopen a Cloud Job Token |
| 401 | `Not enough credits` | **quota, not auth** — stop, do not refresh the token |
| 403 | `No permission to use algorithm` | your offer excludes QRNG — escalate |
| 404 | `Entropy Pool empty` | transient — retry with backoff |

## Note on the product

The hosted endpoint is the cloud face of Quandela's **Entropy** quantum random
number generator, sold as hardware as well. If you need randomness without the
network round trip, the device is the other delivery form — see
<https://www.quandela.com/products-and-services/>.
