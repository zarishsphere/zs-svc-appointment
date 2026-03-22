# PRD — `zs-svc-appointment`

> **Document Class:** PRD | **Version:** 1.0.0 | **Status:** Bootstrapping
> **Repository:** [https://github.com/zarishsphere/zs-svc-appointment](https://github.com/zarishsphere/zs-svc-appointment)
> **Layer:** Layer 2A — Clinical Services | **Catalog #:** 45
> **License:** Apache 2.0 | **Governance:** RFC-0001

---

## 1. Overview

Scheduling, slot management, referrals, and waitlists.

---

## 2. Repository Metadata

- **Name:** `zs-svc-appointment`
- **Organization:** [https://github.com/zarishsphere](https://github.com/zarishsphere)
- **Language / Runtime:** Go 1.26.1
- **Port:** `8009`
- **Visibility:** Public
- **License:** Apache 2.0
- **Default Branch:** `main`
- **Branch Protection:** Required (2-owner review for critical paths)

---

## 3. Platform Context

This repository is part of the **ZarishSphere** sovereign digital health operating platform — a free, open-source, FHIR R5-native system for South and Southeast Asia.

**Non-negotiable constraints:**
- Zero cost — all tooling must use genuinely free tiers
- FHIR R5 native — all clinical data modelled as FHIR R5 resources
- Offline-first — must work without network connectivity
- No-coder friendly — GUI-first, template-driven
- Documentation as Code — all decisions in GitHub

---

## 4. Goals & Objectives

- Implement Appointment management as a FHIR R5-native Go microservice
- Expose OpenAPI 3.1 compliant REST endpoints
- Integrate with FHIR engine, NATS events, and Typesense search
- Enforce multi-tenancy and HIPAA audit logging

## 5. Functional Requirements

| ID | Requirement | Priority |
|----|------------|---------|
| F-01 | FHIR R5 resource CRUD operations | P0 |
| F-02 | SMART on FHIR 2.1 scope validation | P0 |
| F-03 | Multi-tenancy via tenant_id scoping | P0 |
| F-04 | FHIR AuditEvent for all PHI access | P0 |
| F-05 | OpenTelemetry instrumentation | P1 |
| F-06 | Prometheus metrics endpoint | P1 |
| F-07 | NATS JetStream event publishing | P1 |
| F-08 | Integration tests with testcontainers-go | P0 |
| F-09 | Slot availability management | P0 |
| F-10 | Reminder notification triggers | P2 |

## 6. Repository Tree


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
├── .github/
│   ├── CODEOWNERS
│   └── workflows/
│       └── ci.yml
├── cmd/
│   └── server/
│       └── main.go
├── internal/
│   ├── api/
│   │   ├── router.go
│   │   ├── middleware.go
│   │   └── handlers/
│   │       └── appointment.go
│   ├── service/
│   │   └── appointment.go          # Business logic
│   ├── repository/
│   │   └── appointment.go          # PostgreSQL queries (pgx v5)
│   ├── model/
│   │   └── appointment.go          # Domain models + FHIR mapping
│   └── event/
│       └── publisher.go               # NATS event publisher
├── migrations/
│   └── (SQL migrations)
├── config/
│   ├── config.go
│   └── config.yaml
├── deploy/
│   ├── helm/
│   │   ├── Chart.yaml
│   │   ├── values.yaml
│   │   └── templates/
│   └── k8s/
│       └── namespace.yaml
├── docs/
│   └── openapi.yaml                   # OpenAPI 3.1 spec
└── tests/
    ├── unit/
    └── integration/
        └── suite_test.go
```


## 7. Technical Stack

- Go 1.26.1, chi v5, pgx v5.7.2, zerolog, viper, go-oidc v3
- PostgreSQL 18.3 (JSONB), NATS JetStream, Valkey cache
- OpenTelemetry 1.40, Prometheus metrics
- testcontainers-go for integration tests
- Helm chart for Kubernetes deployment

### CI/CD (`.github/workflows/ci.yml`)

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
        with: { go-version-file: go.mod, cache: true }
      - run: go test ./... -race -coverprofile=coverage.out
      - uses: golangci/golangci-lint-action@v6
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: aquasecurity/trivy-action@master
        with: { scan-type: fs, severity: CRITICAL,HIGH }
```

## 9. Ownership & Governance

| Role | GitHub User |
|------|-------------|
| Platform Lead | `@arwa-zarish` |
| Technical Lead | `@code-and-brain` |
| DevOps Lead | `@DevOps-Ariful-Islam` |
| Health Programs | `@BGD-Health-Program` |

All changes go through Pull Request → 1+ owner review → CI pass → merge.
Breaking changes require RFC + ADR.

---

## 10. Definition of Done

- [ ] All listed files exist with content
- [ ] CI pipeline passes (test + lint + security)
- [ ] README.md reflects current state
- [ ] OpenAPI / AsyncAPI spec present (services only)
- [ ] At least 1 integration test using testcontainers-go (Go) or Playwright (UI)
- [ ] No secrets committed (GitGuardian verified)
- [ ] CODEOWNERS file present
- [ ] Linked to CATALOGS.md and TODO.md

---

*This PRD is the canonical source of truth for this repository's purpose, structure, and requirements.*
*Changes require a PR against this file with owner approval.*
