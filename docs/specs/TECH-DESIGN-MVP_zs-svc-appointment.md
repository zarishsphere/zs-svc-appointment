# TECH-DESIGN-MVP — `zs-svc-appointment`

> **Document:** Technical Design (MVP) | **Version:** 1.0.0-mvp
> **Repository:** [https://github.com/zarishsphere/zs-svc-appointment](https://github.com/zarishsphere/zs-svc-appointment)
> **Layer:** Layer 2 — Backend Services | **Catalog #:** 45
> **Language:** Go 1.26.1 | **License:** Apache 2.0

---

## Technical Summary

**Scheduling — slot management, referrals, and appointment reminders.**

This document defines the **technical architecture, implementation design, complete repository tree, and acceptance criteria** for the MVP of `zs-svc-appointment`.

---

## Architecture Overview

```
Client Request
  → Traefik (API Gateway) — TLS, rate limit, auth header
  → SMART on FHIR middleware — JWT validation, scope check
  → chi v5 router
  → Handler — request parsing, validation
  → Service layer — business logic
  → Repository layer — pgx v5 PostgreSQL queries
  → FHIR AuditEvent writer (async)
  → NATS publisher (async)
  → Response
```

## Technology Stack

| Component | Library | Version |
|-----------|---------|---------|
| Language | Go | 1.26.1 |
| HTTP Router | chi | v5.2.1 |
| Database | PostgreSQL + pgx | 18.3 + v5.7.2 |
| Auth | go-oidc v3 + golang-jwt | v3 + v5 |
| Messaging | nats.go | v1.39 |
| Cache | valkey-go | latest |
| Logging | zerolog | latest |
| Config | viper | latest |
| Telemetry | OpenTelemetry | 1.40 |
| Metrics | prometheus/client_golang | latest |
| Migrations | golang-migrate | v4 |
| Testing | testify + testcontainers-go | latest |

## MVP API Endpoints

```
  POST   /fhir/R5/Appointment
  GET    /fhir/R5/Appointment/{id}
  PUT    /fhir/R5/Appointment/{id}
  GET    /fhir/R5/Appointment?params
  GET    /health
  GET    /metrics
```

## Database Schema

```sql
CREATE TABLE appointments (
    id        UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    fhir_id   TEXT NOT NULL,
    tenant_id TEXT NOT NULL,
    resource  JSONB NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW()
);
```



## Environment Variables (.env.example)

```bash
PORT=8009
DATABASE_URL=postgres://zarish:password@localhost:5432/zarishsphere?sslmode=disable
NATS_URL=nats://localhost:4222
VALKEY_URL=redis://localhost:6379
KEYCLOAK_URL=http://localhost:8443
KEYCLOAK_REALM=zarishsphere
KEYCLOAK_CLIENT_ID=zs-svc-appointment
OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317
LOG_LEVEL=info
TENANT_HEADER=X-Tenant-ID
```

## Dockerfile

```dockerfile
FROM golang:1.26-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -trimpath -ldflags="-w -s" -o /bin/server ./cmd/server/

FROM gcr.io/distroless/static-debian12:nonroot
COPY --from=builder /bin/server /server
EXPOSE 8009
ENTRYPOINT ["/server"]
```

## CI/CD Pipeline

```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with: {go-version-file: go.mod, cache: true}
      - run: go test ./... -race -count=1 -coverprofile=coverage.out
      - uses: golangci/golangci-lint-action@v6
      - uses: aquasecurity/trivy-action@master
        with: {scan-type: fs, severity: CRITICAL,HIGH, exit-code: '1'}
      - uses: codecov/codecov-action@v4
```

## MVP Complete Repository Tree

```
zs-svc-appointment/
├── README.md
├── LICENSE
├── go.mod
├── go.sum
├── Makefile
├── Dockerfile
├── .env.example
├── .gitignore
├── CHANGELOG.md
├── .github/
│   ├── CODEOWNERS
│   └── workflows/
│       ├── ci.yml
│       └── release.yml
├── cmd/
│   └── server/
│       └── main.go
├── internal/
│   ├── │   ├── api/handlers/
│   ├── │   │   └── appointment.go
│   ├── │   ├── service/
│   ├── │   │   └── appointment_service.go
│   ├── │   ├── repository/
│   ├── │   │   └── appointment_repo.go
│   ├── │   ├── model/
│   ├── │   │   └── appointment.go
│   ├── │   └── event/
│   ├── │       └── publisher.go
├── migrations/
│   ├── 001_create_appointment_table.sql
├── config/
│   ├── config.go
│   └── config.yaml
├── deploy/
│   ├── helm/
│   │   ├── Chart.yaml
│   │   ├── values.yaml
│   │   └── templates/
│   │       ├── deployment.yaml
│   │       ├── service.yaml
│   │       ├── serviceaccount.yaml
│   │       ├── hpa.yaml
│   │       └── configmap.yaml
│   └── k8s/
│       └── namespace.yaml
├── docs/
│   └── openapi.yaml
└── tests/
    ├── unit/
    └── integration/
        ├── suite_test.go
        └── (feature tests)
```

---


## Owners & Governance

| Role | GitHub Handle | Responsibility |
|------|--------------|----------------|
| Platform Lead | `@arwa-zarish` | Final approval, RFC votes |
| Technical Lead | `@code-and-brain` | Architecture, Go/TS review |
| DevOps Lead | `@DevOps-Ariful-Islam` | CI/CD, infra, deployment |
| Health Programs | `@BGD-Health-Program` | Clinical content, country programs |

**PR Policy:** All changes via Pull Request. Minimum 1 owner review. CI must pass. No direct commits to `main`.


---

## MVP Acceptance Checklist

- [ ] All MVP files exist in repository with real content (not placeholders)
- [ ] CI pipeline passes on `main` branch
- [ ] No secrets, credentials, or PHI committed
- [ ] README.md reflects current state with setup instructions
- [ ] CODEOWNERS file present
- [ ] All MVP functional requirements verified manually or via automated tests
- [ ] Linked to `CATALOGS.md` and `TODO.md` in `zs-docs-platform`

---

*This document is the authoritative MVP specification for `zs-svc-appointment`.*
*Changes require a Pull Request with at least 1 owner approval.*
