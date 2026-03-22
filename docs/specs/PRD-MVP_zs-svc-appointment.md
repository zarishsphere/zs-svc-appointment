# PRD-MVP — `zs-svc-appointment`

> **Document:** Product Requirements (MVP) | **Version:** 1.0.0-mvp
> **Repository:** [https://github.com/zarishsphere/zs-svc-appointment](https://github.com/zarishsphere/zs-svc-appointment)
> **Layer:** Layer 2 — Backend Services | **Catalog #:** 45
> **Language:** Go 1.26.1 | **License:** Apache 2.0

---

## Executive Summary

**Scheduling — slot management, referrals, and appointment reminders.**

This document defines the **Minimum Viable Product (MVP)** scope for `zs-svc-appointment` within the ZarishSphere sovereign digital health platform. It covers what must be built first, acceptance criteria, user stories, and the complete repository file structure.


### Platform Non-Negotiables (apply to every repository)

| Constraint | Rule |
|-----------|------|
| **Zero Cost** | All tooling, hosting, and services must use genuinely free tiers |
| **Open Source** | Apache 2.0 license; all code public |
| **FHIR R5 Native** | All clinical data modelled as FHIR R5 resources |
| **Offline-First** | Must function without network connectivity |
| **No-Coder Friendly** | GUI-first, template-driven, automatable |
| **Documentation as Code** | All decisions in GitHub via RFC/ADR |
| **Multi-tenant** | tenant_id scoping on all data operations |
| **HIPAA/GDPR** | AuditEvent on all PHI access; field-level encryption |

---

## Problem Statement

No-shows and scheduling conflicts are major causes of inefficiency in resource-constrained settings.

## MVP Goals

1. Implement the core appointment FHIR R5 resource operations (Create, Read, Update, Search)
2. Enforce multi-tenancy, HIPAA audit logging, and SMART on FHIR 2.1 auth
3. Publish domain events to NATS JetStream
4. Pass CI (test, lint, security scan) on every commit
5. Deploy via Helm chart to Kubernetes

## MVP User Stories

- As a clinician, I can manage appointment records via FHIR R5 API.
- As a system, all PHI access is audited with FHIR AuditEvent.
- As a program manager, I can query data filtered by my facility tenant.

## MVP Functional Requirements

| ID | Requirement | Acceptance Criteria | Priority |
|----|------------|---------------------|---------|
| M-01 | FHIR R5 Appointment CRUD | POST/GET/PUT return correct FHIR responses | P0 |
| M-02 | SMART on FHIR auth enforced | 401 on missing token; 403 on wrong scope | P0 |
| M-03 | Multi-tenancy via tenant_id | Cross-tenant queries return empty Bundle | P0 |
| M-04 | FHIR AuditEvent on PHI access | AuditEvent row written on read/write | P0 |
| M-05 | Integration tests pass | testcontainers-go suite passes in CI | P0 |
| M-06 | Prometheus metrics exposed | Metrics visible at /metrics | P1 |

## Out of Scope for MVP

- Bulk export ($export operation)
- Advanced analytics / reporting
- Cross-tenant data sharing
- External system integrations (post-MVP)

## MVP Complete Repository Tree

```
zs-svc-appointment/
├── README.md                              # Service overview, local setup, API reference
├── LICENSE                                # Apache 2.0
├── go.mod                                 # Module: github.com/zarishsphere/zs-svc-appointment
├── go.sum
├── Makefile                               # build test lint run docker migrate-up migrate-down
├── Dockerfile                             # Multi-stage distroless build
├── .env.example                           # All required env vars documented
├── .gitignore
├── CHANGELOG.md                           # Semantic-release changelog
├── .github/
│   ├── CODEOWNERS                         # @arwa-zarish @code-and-brain
│   └── workflows/
│       ├── ci.yml                         # test + lint + trivy + codeql
│       └── release.yml                    # GHCR image push on semver tag
├── cmd/
│   └── server/
│       └── main.go                        # Wire deps, start HTTP server on :8009
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
│   ├── config.go                          # Viper config struct with validation
│   └── config.yaml                        # Default values
├── deploy/
│   ├── helm/
│   │   ├── Chart.yaml                     # name: zs-svc-appointment, version: 0.1.0
│   │   ├── values.yaml                    # replicas, image, resources, env
│   │   └── templates/
│   │       ├── deployment.yaml
│   │       ├── service.yaml               # ClusterIP, port 8009
│   │       ├── serviceaccount.yaml
│   │       ├── hpa.yaml                   # min:1 max:5, cpu:70%
│   │       ├── configmap.yaml
│   │       └── _helpers.tpl
│   └── k8s/
│       └── namespace.yaml
├── docs/
│   └── openapi.yaml                       # OpenAPI 3.1 full specification
└── tests/
    ├── unit/
    │   └── (unit test files)
    └── integration/
        ├── suite_test.go                  # testcontainers-go: PostgreSQL + NATS
        └── (feature integration tests)
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
