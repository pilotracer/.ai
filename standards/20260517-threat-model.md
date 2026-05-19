# Threat Model — AC Billing System

**Status:** initial · 2026-05-17. Revisit on every release train, on every new bounded context, and immediately on any security-relevant incident.
**Method:** STRIDE-per-boundary, augmented with explicit per-tenant blast-radius analysis (a leak that crosses tenants is treated as a category of harm, not a per-CIA classification).
**Audience:** tech lead, security reviewer, and any future external auditor.
**Pairs with:** `.ai/standards/20260517-data-classification.md`, `.work/features/fiscal-pipeline/20260517-SPEC.md` §15, ADR 003/004/005.

This is not a pen-test report. It is the design-time enumeration of how the system can fail under adversarial pressure and what stops each failure.

---

## 1. Assets (highest value first)

| ID | Asset | Why it matters | Classification |
|----|-------|----------------|----------------|
| A1 | Tenant signing material (P12 / ATV cryptographic key + PIN) | Lets the holder forge legally-valid comprobantes in the tenant's name. | `credential` |
| A2 | Tenant Hacienda OAuth credentials (ATV-issued username + password) | Lets the holder submit comprobantes as the tenant. | `credential` |
| A3 | Signed XML artifacts (issued and Hacienda response) | Legal evidence of what was issued and accepted. Loss = audit failure. | `financial` + integrity-critical |
| A4 | Event log (`fiscal_events`) | Sole source of truth for what happened. Tampering = legal exposure. | `operational` integrity-critical |
| A5 | Customer master data (parties, items, prices) | Aggregated PII + commercial-sensitive. | `pii` + `financial` |
| A6 | Tenant DB content (the entire schema) | Cross-tenant leak from here is the worst-case multi-tenant failure. | `pii` + `financial` |
| A7 | API access tokens (5-min Hacienda JWT, cached) | Short-lived but lets the holder submit during the window. | `credential` |
| A8 | Operator dashboard sessions | Privileged operations include contingency activation, credential rotation. | `credential` |
| A9 | Callback URL token | If guessed, allows forged callbacks injecting `Accepted`/`Rejected` events. | `credential` |
| A10 | Backup snapshots (DB + S3) | A leaked backup carries everything above. | inherits |

## 2. Trust boundaries

```
[Customer browser] ─── HTTPS ──▶ [CloudFront] ─── HTTPS ──▶ [ALB + WAF] ─── HTTPS ──▶ [FastAPI in EKS]
                                                                                            │
                                                                                            ├─ Redis (broker/cache)
                                                                                            ├─ Postgres (RDS)
                                                                                            ├─ Secrets Manager / KMS
                                                                                            └─ Redis ── Celery ──▶ [fiscal-worker in EKS]
                                                                                                                          │
                                                                                                                          ├─ KMS (Decrypt; only here)
                                                                                                                          ├─ S3 (signing material + artifacts)
                                                                                                                          ├─ HTTPS ▶ Hacienda IdP
                                                                                                                          ├─ HTTPS ▶ Hacienda recepcion/v1
                                                                                                                          └─ HTTPS ◀ Hacienda callback (back through ALB)
```

Boundaries (the lines that matter):

- **B1** Public Internet ↔ CloudFront (browsers and Hacienda).
- **B2** CloudFront ↔ ALB (AWS-internal; still external from VPC's view).
- **B3** ALB ↔ FastAPI process.
- **B4** FastAPI ↔ Postgres / Redis / Secrets Manager.
- **B5** Celery broker ↔ fiscal-worker.
- **B6** fiscal-worker ↔ KMS.
- **B7** fiscal-worker ↔ S3.
- **B8** fiscal-worker ↔ Hacienda IdP + API.
- **B9** Per-tenant logical boundary inside the database (RLS-enforced, ADR 004).
- **B10** Operator (human) ↔ dashboard.
- **B11** Developer (human) ↔ source repo + CI + cloud console.

## 3. STRIDE per boundary (asset-focused)

Notation: `S/T/R/I/D/E` = Spoofing / Tampering / Repudiation / Information disclosure / Denial of service / Elevation of privilege.

### 3.1 B1 — Public Internet ↔ CloudFront

| Risk | Class | Asset | Mitigation |
|------|-------|-------|------------|
| Forged callback claiming to be Hacienda | S, T, E | A4, A9 | Callback path includes per-tenant `callback_token` (256-bit, KMS-encrypted at rest). Handler verifies (tenant, token); resolves comprobante by `(tenant_id, clave)`; never trusts the body alone (R8 + R10 of fiscal SPEC). Unknown tokens → 403 + security log. |
| Credential stuffing on operator login | S | A8 | WAF rate-limit + per-account lockout + MFA mandatory for any role that includes `tenant.fiscal.contingency.manage` or `tenant.fiscal.credentials.manage`. |
| DDoS on the customer-facing dashboard | D | A6 (indirect) | CloudFront + WAF; ALB scaling; per-IP rate limits. |
| Reflected XSS / CSRF in dashboard | T, E | A8 | Next.js with strict CSP, SameSite cookies, double-submit CSRF tokens on state-changing requests. |

### 3.2 B3 — ALB ↔ FastAPI

| Risk | Class | Asset | Mitigation |
|------|-------|-------|------------|
| JWT forgery / replay against internal API | S, E | A8 | Tokens signed by Keycloak with RS256; `kid` rotation; `aud` and `iss` validated; short TTL (5 min) on access tokens with refresh. |
| Missing `app.tenant_id` on a DB session | E | A6 | SQLAlchemy session middleware refuses to issue tenant-scoped queries without `app.tenant_id`. Integration test enforces. |
| Mass assignment via Pydantic input | T | A5, A6 | All input models declare `model_config = ConfigDict(extra='forbid')`. CI lint guards. |
| Path traversal in S3 presigned URL generation | I | A1, A3 | S3 keys derived from server-side IDs only; user-supplied filenames are stored separately and never used as the key suffix. |

### 3.3 B4 — FastAPI ↔ Postgres / Secrets / Redis

| Risk | Class | Asset | Mitigation |
|------|-------|-------|------------|
| SQL injection | T, I | A6 | SQLAlchemy parameterized only; raw SQL only in reporting `Read` repositories, with no user-supplied identifiers. |
| Postgres role abuse | E | A4 | Application role has no `BYPASSRLS`; only one ops role does, and its use is audited via PgAudit. |
| Redis-as-broker poisoning (task body tampering) | T | A1, A4 | Celery signed payloads (`task_serializer='json' + message_signing`) with a per-cluster HMAC key in Secrets Manager. |
| Snapshot/backup exfiltration | I | A10 | RDS snapshots are KMS-encrypted; access requires a separate IAM role; cross-region replication of S3 backup bucket is in a different AWS account. |

### 3.4 B6 — fiscal-worker ↔ KMS (highest-risk boundary)

| Risk | Class | Asset | Mitigation |
|------|-------|-------|------------|
| API role gains `kms:Decrypt` on a tenant CMK | E | A1, A2 | IAM policy explicitly denies; Terraform CI check; production CMK key policies whitelist the worker role only. |
| Worker decrypts the wrong tenant's CMK | I, T | A1 across tenants | Worker is single-tenant per task; Celery task routing partitions by tenant; per-task IAM session via STS `AssumeRole` with a tenant-scoped policy (`Condition: ${aws:RequestTag/tenant}` matches). |
| Compromised worker pod exfiltrates current decrypted material | I | A1 | Material is wiped after each signing (`bytearray` zeroed); pod's outbound network ACL allows only Hacienda hosts + KMS + S3; egress goes through a NAT with logged flow. |
| KMS key rotation breaks signing | D | A1 | Rotation is enabled; alias resolves dynamically; integration test exercises rotation in `local-dev` via LocalStack. |

### 3.5 B8 — fiscal-worker ↔ Hacienda

| Risk | Class | Asset | Mitigation |
|------|-------|-------|------------|
| MITM on token endpoint | S, I | A2, A7 | TLS verification mandatory; client refuses any `http://` host even though Hacienda metadata returns `http://` (doc 02 §3.2 caveat). |
| Token leakage in logs | I | A7 | CONVENTIONS §7 + redact helper. CI grep for token-shaped strings. |
| Rate-limit triggers IP ban | D | A6 (operations) | Per-tenant token bucket inside the worker; cluster-level guard via global counter in Redis; `X-Ratelimit-Reset` honoured. |
| Replay of stale signed comprobante | T | A4 | `clave` uniqueness (R2) means Hacienda will reject duplicates with `X-Error-Cause: ya fue recibido`. Pipeline maps this to `HaciendaRejection` and does not retry. |

### 3.6 B9 — Per-tenant logical boundary in the DB

| Risk | Class | Asset | Mitigation |
|------|-------|-------|------------|
| Cross-tenant join via missing `tenant_id` filter | I | A5, A6 | Schema-per-tenant + RLS + denormalized `tenant_id` column on every table (ADR 004); SQLAlchemy session middleware sets `app.tenant_id`; CI checks no migration creates a tenant table without these. |
| Operator queries with `BYPASSRLS` accidentally exposing data | I | A5, A6 | `BYPASSRLS` granted to one ops role; usage audited; standard ops scripts under `apis/scripts/ops/` use the application role explicitly. |
| Backup restore into the wrong tenant's schema | T, I | A6 | Restore tooling refuses to write to a schema whose `tenant_id` does not match the manifest in the backup. |

### 3.7 B10 — Operator ↔ dashboard

| Risk | Class | Asset | Mitigation |
|------|-------|-------|------------|
| Privilege escalation by re-using a session | E | A8 | Privileged endpoints (`contingency:*`, `fiscal-credentials/rotate`) require step-up auth (re-MFA within 60 s). |
| Insider rogue admin | R, T | All | Every privileged action emits an `audit_log` row with `actor_id`, `correlation_id`, `payload_hash`. Logs are append-only via Postgres trigger (no UPDATE/DELETE granted on `audit_log`). |
| Phishing of operator credentials | S | A8 | MFA mandatory; session bound to UA/IP family (warns on change); SSO-only path planned post-GA. |

### 3.8 B11 — Developer ↔ source / CI / cloud

| Risk | Class | Asset | Mitigation |
|------|-------|-------|------------|
| Secret committed to repo | I | A1, A2 | `gitleaks` pre-commit + CI; PR template asks for an explicit "no secrets" checkbox. |
| Malicious or compromised dependency | T, E | All | `pip-audit`, `npm audit`, Dependabot, signed commits required for `main`, two-reviewer rule on security-sensitive paths (CONVENTIONS §12). |
| Cloud-console standing access | E | All | Console access via SSO + just-in-time AWS IAM Identity Center role assumption with audit log. No long-lived IAM users for humans. |
| CI runner compromise produces a tainted image | T | All | OIDC-based push to ECR with no static credentials; signed images (cosign) verified at deploy. |

## 4. Cross-cutting risks (not boundary-specific)

| Risk | Description | Mitigation |
|------|-------------|------------|
| **Cross-tenant key reuse** | A bug routes one tenant's signing material to another tenant's worker task. | I4 in fiscal SPEC + single-tenant Celery queue partitioning + per-task IAM session bound to `tenant_id`. |
| **Lost callback secret** | Tenant's `callback_token` leaks via logs or backups. | Tokens are encrypted at rest; never logged (CI grep); rotated through `fiscal-credentials/rotate`. |
| **Time drift** | Worker clock skews; signed XML carries wrong `fecha`; Hacienda rejects. | NTP via AWS Time Sync; integration test asserts time helper produces server-tz-aware output regardless of pod tz. |
| **Annex version drift** | Hacienda publishes v4.5; our XSDs reject correctly-formed v4.5 documents during overlap. | Doc 02 §8 maintenance process; release train owns annex bumps; staging runs both validators during the 30-day overlap. |
| **Supply-chain compromise of `xmlsec` / libxmlsec1** | Compromised crypto library forges signatures. | Pin versions in `DOCS_TECH_STACK.md`; mirror wheels through a private index; cross-verify signatures in contract tests using a second library (`signxml`). |
| **Disposable test tenants leak into prod** | A `local-dev` tenant accidentally has real Hacienda credentials. | Provisioning script refuses to attach real-IdP credentials to a tenant tagged `env != prod`; CI rejects any seed file referencing the prod IdP host. |

## 5. Out of scope of this model

- **DDoS economics:** mitigated by CloudFront / WAF / ALB scaling; cost modelling lives in ops, not here.
- **Physical security of AWS data centres:** assumed (AWS responsibility under shared-responsibility model).
- **Costa Rica regulatory interpretation of an incident:** owned by CPA (ADR 009).

## 6. Top 10 to fix before GA

Ranked by impact × likelihood as judged from current design:

1. Confirm IAM policy denies `kms:Decrypt` for the API role (Terraform check) — B6.
2. Implement per-tenant Celery routing and `AssumeRole` for signing tasks — B6.
3. Implement `callback_token` rotation endpoint — B1/A9.
4. Implement MFA + step-up auth on privileged dashboard endpoints — B10/A8.
5. Wire `gitleaks` + `pip-audit` + Dependabot in CI — B11.
6. Implement RLS on every tenant table from migration #1 (foundational; not retrofittable) — B9.
7. Wire `audit_log` append-only Postgres trigger + revocation of UPDATE/DELETE on the role — B10.
8. Implement Celery message signing — B4/B5.
9. Implement signed-artifact SHA-256 verification on read — A3/A4.
10. Run a tabletop incident exercise (data leak + Hacienda rejection storm) before first paying customer — operational.

## 7. Review cadence and triggers

- Every release train: scan for new boundaries and new assets. Update this document.
- Every new bounded context: add a STRIDE row block.
- Every new external dependency: add a supply-chain risk row.
- Every security-relevant incident: file a post-mortem and update this model. Old versions are kept under `.work/plans/archives/threat-model/` (per ADR convention "never edit a Decided artifact in place").
