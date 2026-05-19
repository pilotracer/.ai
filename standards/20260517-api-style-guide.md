# API Style Guide — AC Billing System

**Status:** binding once `apis/` lands · 2026-05-17
**Audience:** backend and dashboard engineers
**Pairs with:** `20260517-CONVENTIONS.md` §6, `20260517-observability-spec.md`, FEATURE_STANDARD §6

---

## 1. Base URL and versioning

| Env | Public API base | Dashboard BFF |
|-----|-----------------|---------------|
| local-dev | `http://localhost:8000/api/v1` | `http://localhost:3000` → proxies `/api/v1` |
| staging/prod | `https://{tenant-subdomain}.acb.example.com/api/v1` | Same origin |

- Version in path only: `/api/v1/...`. No `Accept-Version` header in v1.
- Breaking changes require `/api/v2` and an ADR.

---

## 2. Authentication and tenant context

- **Operator/dashboard:** Keycloak-issued JWT (`Authorization: Bearer`).
- **Hacienda callback:** unauthenticated path; validated via `callback_token` in path (fiscal SPEC §6).
- **Internal context → fiscal:** service token or signed internal header (defined when workers call API).

**Required headers (authenticated routes):**

| Header | Required | Notes |
|--------|----------|-------|
| `Authorization` | Yes | Bearer JWT |
| `X-Correlation-Id` | Optional on request | Server generates UUID v4 if absent; echoed on response |
| `Idempotency-Key` | On mutating POST that create side effects | UUID v4; 24 h replay window |
| `Accept-Language` | Optional | `es-CR`, `en-US`, … per ADR 008 |

**Tenant resolution:** JWT claim `tenant_id` (UUID). Middleware sets Postgres `app.tenant_id` before any handler runs. Cross-tenant access → `403`.

---

## 3. Resource naming

- Plural nouns, kebab-case: `/commercial-documents`, `/master-data/parties`, `/fiscal-documents`.
- Actions use **custom methods** with colon suffix on the resource id:
  - `POST /commercial-documents/{id}:confirm`
  - `POST /commercial-documents/{id}:issue`
  - `POST /tenants/{tenant_id}/contingency:activate`
- Collection filters via query string, not new paths: `?state=Draft&doc_type=TiqueteElectronico`.

**Identifiers in paths:**

| Type | Format |
|------|--------|
| Internal id | UUID |
| Hacienda `clave` | Only on dedicated routes: `/fiscal-documents/by-clave/{clave}` |

---

## 4. Request and response bodies

- **Content-Type:** `application/json; charset=utf-8` unless file upload (presigned S3 flow).
- **Field naming:** `snake_case` in JSON (matches Python/Pydantic).
- **Money:** string decimal in JSON (`"1234.56"`), never float. Scale per CONVENTIONS §4.
- **Timestamps:** ISO-8601 UTC in API (`2026-05-17T15:00:00Z`); Hacienda-bound fields use `cr_iso` wrapper when exposed.
- **Extra fields:** Pydantic `extra='forbid'` on all input models.

**Envelope (list responses):**

```json
{
  "data": [ ... ],
  "meta": {
    "page": 1,
    "page_size": 50,
    "total_count": 123,
    "has_next": true
  }
}
```

Single-resource responses return the resource object directly (no `data` wrapper).

---

## 5. Pagination

**Offset pagination (default v1):**

| Query | Default | Max |
|-------|---------|-----|
| `page` | 1 | — |
| `page_size` | 50 | 200 |

Response `meta` as above.

**Cursor pagination (optional for high-volume fiscal lists v1.1):**

- Request: `?cursor={opaque}&limit=50`
- Response: `meta.next_cursor`, `meta.has_next`

---

## 6. Sorting and filtering

- Sort: `?sort=-created_at` (prefix `-` = descending). Allow-list per route in OpenAPI.
- Filters: explicit query params (`state`, `doc_type`, `from`, `to`); no generic `filter=` DSL in v1.

---

## 7. Errors — RFC 7807 Problem Details

All non-2xx responses use `application/problem+json`:

```json
{
  "type": "https://acb.example.com/problems/commercial/not-draft",
  "title": "Document is not in Draft state",
  "status": 409,
  "detail": "Only Draft documents can be edited.",
  "instance": "/api/v1/commercial-documents/3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "code": "NotDraft",
  "retryable": false,
  "errors": [
    { "field": "commercial_state", "message": "Expected Draft, got Confirmed" }
  ]
}
```

| Field | Required | Notes |
|-------|----------|-------|
| `type` | Yes | Stable URI; never changes meaning |
| `title` | Yes | Short English for logs |
| `status` | Yes | HTTP status |
| `detail` | Yes | Safe for UI; no PII, no raw Hacienda JSON |
| `instance` | Yes | Request path |
| `code` | Yes | Machine enum matching `DomainError.code` |
| `retryable` | When applicable | Client may retry with backoff |
| `errors` | Optional | Field-level validation (422) |

**Status code map (minimum):**

| Code | Use |
|------|-----|
| 400 | Malformed JSON / invalid query |
| 401 | Missing/invalid JWT |
| 403 | RBAC or tenant mismatch |
| 404 | Resource not found (do not leak cross-tenant existence) |
| 409 | State conflict (`NotDraft`, `DuplicateClave`) |
| 422 | Validation failed |
| 429 | Rate limit (include `Retry-After`) |
| 502/503 | Upstream (Hacienda) — `retryable: true` |

**Hacienda errors:** map to problem+json with `code: HaciendaRejection` or `HaciendaUnavailable`; include `x_error_cause` in `detail` only when Hacienda provided it (fiscal R14).

---

## 8. Idempotency

Applies to: `POST` that creates resources or triggers fiscal issue/submit.

```
Idempotency-Key: 7c9e6679-7425-40de-944b-e07fc1f90ae7
```

- Server stores `(tenant_id, idempotency_key, route, request_hash)` → response snapshot for **24 h**.
- Replay with same key + same body → **same status + body** as first success.
- Same key + different body → `409` `IdempotencyKeyConflict`.
- Missing key on required routes → `400`.

Routes requiring idempotency in v1:

- `POST /commercial-documents`
- `POST /commercial-documents/{id}:issue`
- `POST /internal/fiscal/sign-and-submit/{id}` (internal)

---

## 9. Long-running operations

Pattern: **202 Accepted** + job resource.

```http
POST /commercial-documents/{id}:issue
→ 202 Accepted
Location: /api/v1/jobs/{job_id}
```

```json
{ "job_id": "…", "status": "pending", "poll_url": "/api/v1/jobs/{job_id}" }
```

Job GET returns `status: pending|running|succeeded|failed` and `result` on success. UI polls every 2 s with backoff to 10 s.

---

## 10. OpenAPI and client generation

- FastAPI auto-generates OpenAPI 3.1 at `/api/v1/openapi.json`.
- Dashboard TypeScript client generated in CI from OpenAPI (`openapi-typescript` or equivalent) — pin in dashboard bootstrap.
- Every route documents: auth scope, idempotency, possible `code` values.

---

## 11. Rate limiting (application layer)

| Scope | Limit | Response |
|-------|-------|----------|
| Per IP (unauthenticated callback) | 60/min | 429 |
| Per tenant API | 300/min | 429 + `Retry-After` |
| Per user login | WAF + Keycloak | — |

Distinct from Hacienda quotas (catalog sync only).

---

## 12. CORS and security headers

- Dashboard same-origin in prod; CORS allow-list for `local-dev` only.
- Responses include: `Strict-Transport-Security`, `X-Content-Type-Options: nosniff`, `Cache-Control: no-store` on authenticated routes.

---

## 13. Examples

**Create draft commercial document:**

```http
POST /api/v1/commercial-documents
Authorization: Bearer …
Idempotency-Key: …
Content-Type: application/json

{
  "doc_type": "TiqueteElectronico",
  "branch_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "party_id": null
}
```

**List with filters:**

```http
GET /api/v1/commercial-documents?state=FiscalPending&page=1&page_size=20&sort=-created_at
```

---

## 14. Residual verification

- Aligns with CONVENTIONS §6 (RFC 7807) and fiscal/commercial SPECs.
- No runtime API exists yet.
