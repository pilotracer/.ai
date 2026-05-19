# Observability Specification — AC Billing System

**Status:** Draft · 2026-05-17
**Owner:** Tech lead
**Pairs with:** `DOCS_TECH_STACK.md` §5 (OTel → backend TBD), fiscal SPEC §14, FEATURE_STANDARD §6, CONVENTIONS §7

---

## 1. Purpose

Single catalog for **metrics, traces, logs, and SLOs** so features do not invent incompatible names. Implementation uses OpenTelemetry SDK → collector → backend (Datadog / Grafana Cloud / self-hosted — **TODO** in stack doc).

## 2. Global conventions

### 2.1 Metric names

- Prefix: `acb_`
- Pattern: `acb_<context>_<verb>_total` (Counter), `acb_<context>_<verb>_seconds` (Histogram)
- Labels: `tenant_id` (low-cardinality slug or id per backend limits), `result` ∈ `success|failure`, plus domain labels documented per context.
- **Never** label metrics with `clave`, email, or tax id.

### 2.2 Traces

- Span name: `<context>.<use_case>` (e.g. `fiscal.submit`, `commercial.confirm`).
- Required attributes: `tenant_id`, `correlation_id`, `user_id` (if authenticated).
- W3C `traceparent` propagated: browser → API → Celery → Hacienda HTTP client.

### 2.3 Logs (structlog / pino)

Mandatory fields: `ts`, `level`, `event`, `tenant_id`, `correlation_id`.

Optional: `user_id`, `doc_id`, `clave`, `duration_ms`, `http_status`.

Forbidden: PII bodies, JWT, keys, full Hacienda JSON envelope (CONVENTIONS §7).

### 2.4 Correlation ID

- Generated at API edge (UUID v4) if not supplied.
- Returned to client as `X-Correlation-Id`.
- Stored on `fiscal_events`, `audit_log`, commercial documents at issue time.

---

## 3. Platform metrics

| Metric | Type | Labels |
|--------|------|--------|
| `acb_http_request_total` | Counter | `method`, `route`, `status_class` |
| `acb_http_request_seconds` | Histogram | `method`, `route` |
| `acb_celery_task_total` | Counter | `task_name`, `result` |
| `acb_celery_task_seconds` | Histogram | `task_name` |
| `acb_db_pool_in_use` | Gauge | — |

---

## 4. Context metrics

### 4.1 Fiscal (from fiscal SPEC §14 — canonical)

`acb_fiscal_draft_total`, `acb_fiscal_sign_total`, `acb_fiscal_sign_seconds`, `acb_fiscal_submit_total`, `acb_fiscal_submit_seconds`, `acb_fiscal_callback_total`, `acb_fiscal_poll_total`, `acb_fiscal_contingency_queue_depth`, `acb_fiscal_contingency_oldest_seconds`, `acb_fiscal_token_refresh_total`.

**SLOs (initial):**

| SLO | Target |
|-----|--------|
| p95 sign latency | ≤ 500 ms |
| p95 submit latency | ≤ 2 s |
| median Submitted → Accepted | ≤ 60 s |
| callback success rate | ≥ 90% |

### 4.2 Master data

| Metric | Type | Labels |
|--------|------|--------|
| `acb_master_data_party_write_total` | Counter | `result` |
| `acb_master_data_item_write_total` | Counter | `result` |
| `acb_catalog_sync_total` | Counter | `resource` (cabys\|fx\|ex\|ae), `result` |
| `acb_catalog_sync_seconds` | Histogram | `resource` |
| `acb_catalog_sync_rate_limited_total` | Counter | `resource` |

### 4.3 Commercial

| Metric | Type | Labels |
|--------|------|--------|
| `acb_commercial_document_create_total` | Counter | `doc_type`, `result` |
| `acb_commercial_confirm_total` | Counter | `doc_type`, `result` |
| `acb_commercial_issue_total` | Counter | `doc_type`, `result` |
| `acb_commercial_confirm_seconds` | Histogram | `doc_type` |

### 4.4 Identity / auth

| Metric | Type | Labels |
|--------|------|--------|
| `acb_auth_login_total` | Counter | `result` |
| `acb_auth_step_up_total` | Counter | `result` |

---

## 5. Log event catalog (minimum)

| Event name | Level | When |
|------------|-------|------|
| `http.request.completed` | info | API request done |
| `fiscal.drafted` | info | fiscal R1 |
| `fiscal.signed` | info | post-sign |
| `fiscal.submitted` | info | 201 from Hacienda |
| `fiscal.callback.received` | info | callback handler |
| `fiscal.poll.completed` | info | poll job |
| `fiscal.rejected` | warn | terminal rejection |
| `fiscal.signature_failed` | error | engineering alert |
| `commercial.confirmed` | info | |
| `commercial.issued` | info | |
| `catalog.sync.failed` | warn | retry scheduled |
| `security.callback_token_invalid` | warn | possible attack |

---

## 6. Dashboards (staging + prod)

| Dashboard | Panels |
|-----------|--------|
| Fiscal health | SLOs, queue depth, submit error rate, token refresh failures |
| Hacienda dependency | 429/5xx rate, rate-limit headers, poll backlog |
| Tenant activity | documents issued/hour (no PII) |
| Catalog sync | CABYS/FX job success |

---

## 7. Alerting (initial)

| Alert | Condition | Severity |
|-------|-----------|----------|
| Fiscal submit error spike | `rate(acb_fiscal_submit_total{result="failure"}[5m])` > threshold | page |
| Contingency queue aging | `acb_fiscal_contingency_oldest_seconds` > CPA-defined SLA (TBD ADR 010) | page |
| Token refresh failures | `rate(acb_fiscal_token_refresh_total{result="failure"}[15m])` > 0 | ticket |
| Catalog sync stuck | no `acb_catalog_sync_total` success in 24h | ticket |

---

## 8. Environments

| Env | Sampling | Backend |
|-----|----------|---------|
| local-dev | 100% traces | console / optional local stack |
| ci | metrics only or 10% traces | discarded |
| staging | 100% | TBD |
| prod | 10% traces, 100% metrics/logs | TBD |

---

## 9. Residual verification

- Backend vendor not pinned (stack TODO).
- Fiscal metric names match fiscal SPEC §14 (verified by cross-read).
