# Engineering Handbook 2026 - Organization Engineering Standards

```text
====================================================================================
                    ENGINEERING HANDBOOK 2026
               Organization Engineering Standards
                         Version 1.0
====================================================================================

PURPOSE
-------
This handbook defines the engineering standards, architecture principles, development
workflow, quality gates, security requirements, testing strategy, repository
structure, deployment process, and operational guidelines for all software projects.

Engineering Goals
-----------------
• Simplicity          • Reliability         • Obsvervability
• Scalability         • Performance         • Automation
• Maintainability     • Testability         • Documentation
• Security            • Consistency         • Excellence
```

---

## 1. Engineering Principles

### Follow:
* **SOLID**: Single Responsibility, Open-Closed, Liskov Substitution, Interface Segregation, Dependency Inversion.
* **DRY**: Don't Repeat Yourself (abstract code where reuse is logical; avoid copy-pasting blocks).
* **KISS**: Keep It Simple, Stupid (prefer straightforward, readable code over clever optimizations).
* **YAGNI**: You Aren't Gonna Need It (do not write code for future specs that aren't approved yet).
* **Separation of Concerns**: Separate rendering, query states, services, and business calculations.
* **High Cohesion & Low Coupling**: Keep related operations close; minimize dependency intersections.
* **Composition over Inheritance**: Use functional plugins or interfaces instead of subclassing models.
* **Fail Fast**: Throw clear validation/input errors early rather than passing invalid state down.
* **Defensive Programming**: Validate types, parameters, boundaries, and handles at every boundary.
* **Immutable Data**: Protect states from side effects by returning new copies of data arrays/objects.
* **Idempotent Operations**: Ensure repeated calls yield the same system state safely.
* **Least Privilege**: Grant components, systems, and tokens only the access parameters required.
* **Principle of Least Surprise**: Code must behave predictably; use explicit naming and standard patterns.

### Avoid:
* **God Classes / God Components**: Massive classes or UI files containing thousands of lines of logic.
* **Circular Dependencies**: Module A imports Module B, which imports Module A.
* **Duplicate Code**: Similar routines duplicated in multiple routes without abstraction.
* **Business Logic inside UI**: Calculations, data transformations, or validations written inside components.
* **Tight Coupling**: Direct dependencies on concrete implementations instead of interfaces/abstract keys.
* **Premature Optimization**: Refactoring code for scale before actual bottleneck data is measured.
* **Hidden Side Effects**: Mutating outer variables or structures silently inside a helper function.
* **Hardcoded Secrets**: Inscribing passwords, JWT keys, URLs, or client keys directly in repositories.

---

## 2. Project Lifecycle

```text
  Idea / Requirement Discovery
               ↓
      Architecture Design
               ↓
      Repository Creation
               ↓
    Development & Formatting
               ↓
    Local Lint / Secret Scan
               ↓
      Testing & Coverage
               ↓
     Code & Security Review
               ↓
         CI/CD Checks
               ↓
       Staging Deployment
               ↓
     Production Deployment
               ↓
     Monitoring & Alerting
               ↓
          Maintenance
               ↓
          Retirement
```

---

## 3. Repository Strategy

### Monorepo
* **Use when**: Working on multiple applications that share utility packages, shared UI components, shared backend API interfaces, and are built/released by a single engineering team.

### Polyrepo
* **Use when**: Service deployments are independent, release cycles are decoupled, microservices are owned by distinct teams, or codebase scales past single-repository compiler capacities.

### Mandatory Repository Files:
Each repository must contain the following standard files before production approval:
1. `README.md` — Setup guides, tech stack details, build scripts, and local developer manual.
2. `LICENSE` — Copywrite permissions.
3. `CONTRIBUTING.md` — Coding guidelines, branching strategy, and PR expectations.
4. `CODEOWNERS` — Defines PR review assignees automatically.
5. `SECURITY.md` — Policy for reporting security vulnerabilities.
6. `CHANGELOG.md` — Log of changes added in each release tag.
7. `.gitignore` — Prevents tracking build folders, secrets, and caches.
8. `.editorconfig` — Standardizes editor settings (indentation, tabs, line endings).
9. `.prettierrc` — Code style rules.
10. `eslint.config.js` — JavaScript/TypeScript linter rules.
11. `Dockerfile` / `docker-compose.yml` — Containerization configs.
12. `.env.example` — Template env variables without real values.
13. `architecture_diagram.png` — Structural overview of the database and components.

---

## 4. Standard Project Structure

```text
├── docs/                   # ADRs, runbooks, schemas, and design documents
├── src/                    # Source files
├── tests/                  # Integration and E2E mock suites
├── scripts/                # CI check scripts, audits, and deployment tasks
├── config/                 # Static environment configurations
├── assets/                 # Uncompiled images, videos, and fonts
├── public/                 # Static assets served directly
├── docker/                 # Container files (Dockerfiles, entrypoints)
├── .github/                # GitHub Actions workflows and templates
├── .env.example            # Environment variables template
└── README.md               # Repository readme documentation
```

### Frontend-scoped Directories (under `src/`):
* `app/` — Root configurations, route provider setups, and app bootstrap logic.
* `components/` — Global presentation UI elements (atomic, generic elements like buttons/modals).
* `features/` — High-cohesion domain modules (combining state, components, and hooks for a specific domain).
* `hooks/` — Custom query and local utility hook managers.
* `services/` — Pure API client fetch methods.
* `api/` — Low-level interceptors, HTTP configurations, and client setup.
* `stores/` — Global client state engines (Zustand, Redux, etc.).
* `routes/` — Page routes (mapped to TanStack file routing).
* `layouts/` — Standard layouts (Sidebar, Header, Main content views).
* `providers/` — React Context bindings (theme providers, query clients, router instances).
* `types/` — Standard typescript interfaces/types declarations.
* `utils/` — Math, string formatting, date handlers, and helpers.
* `constants/` — Static lists, route paths, and configurations.
* `styles/` — Global styles, tailwind configurations, and variable sheets.

### Backend-scoped Directories (under `src/` or repository root):
* `cmd/` — Entrypoint files for application run commands.
* `internal/` — Code restricted to the package (cannot be imported by external projects).
* `api/` — OpenAPI specs, routers, and path mappings.
* `handlers/` — Layer handling HTTP/gRPC requests, calling service methods, and returning status codes.
* `services/` — Business logic calculations, workflows, and transaction layers.
* `repositories/` — Pure database actions (SQL queries, ORM calls).
* `domain/` — Core business models and rules.
* `entities/` — Database table mappings.
* `dto/` — Data Transfer Objects validating incoming and outgoing request formats.
* `middleware/` — Auth filters, logging, CORS, rate limiting, and metrics.
* `config/` — Environment variables parsing and configuration loads.
* `database/` — DB connection clients, pools, and seeding tasks.
* `migrations/` — Incremental SQL change schemas.
* `jobs/` — Cron configurations, routines, and schedulers.
* `events/` — Pub/Sub message definitions (Kafka, RabbitMQ, etc.).
* `workers/` — Message queue processors.

---

## 5. Architectural Isolation & Layer Rules

```text
┌────────────────────────────────────────────────────────┐
│                   Presentation Layer                   │
└───────────┬────────────────────────────────────────────┘
            │ (Dependency flow goes INWARD only)
            ▼
┌────────────────────────────────────────────────────────┐
│                     Business Layer                     │
└───────────┬────────────────────────────────────────────┘
            │
            ▼
┌────────────────────────────────────────────────────────┐
│                      Domain Layer                      │
└───────────┬────────────────────────────────────────────┘
            │
            ▼
┌────────────────────────────────────────────────────────┐
│                Data / Repository Layer                 │
└───────────┬────────────────────────────────────────────┘
            │
            ▼
┌────────────────────────────────────────────────────────┐
│                  Infrastructure Layer                  │
└────────────────────────────────────────────────────────┘
```

### Dependency Rules:
1. **No UI Database Queries**: UI components must never access databases directly.
2. **No Business Logic in UI**: Components should only display values, bind handlers, and display layouts.
3. **Decoupled API Contracts**: UI views must not depend on service configurations; changes to services or hooks must not break UI rendering.
4. **Isolated Database Access**: Database operations are restricted to repositories.
5. **Thin Controllers**: Controllers (or handlers) must only parse variables, call the correct service methods, and return status codes.
6. **Domain Boundary Rules**: Business policies belong inside the Domain layer, completely decoupled from specific frameworks (e.g. clean architecture rules).

---

## 6. API Standards

### REST Method Naming
* **`GET /users`** — Retrieves list of users (supports pagination, filtering, sorting).
* **`POST /users`** — Creates a new user record.
* **`GET /users/{id}`** — Retrieves details of a specific user.
* **`PATCH /users/{id}`** — Modifies specific properties of a user record.
* **`DELETE /users/{id}`** — Performs deletion (soft/hard) of a user record.

### Required Actions:
* **Versioning**: Prefix all API paths (e.g., `/api/v1/users`).
* **Input Validation**: Validate every payload at the controller level using DTO decorators or validation schemas (e.g., Zod, Pydantic).
* **HTTP Status Codes**: Use exact response codes (`200 OK`, `201 Created`, `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `422 Unprocessable Entity`, `500 Internal Error`).
* **List Support**: Enforce pagination (`limit`/`offset`), filters, and sorting parameters on all list endpoints.
* **Rate Limiting**: Defend all API endpoints with rate-limiting middlewares.
* **OpenAPI Documentation**: Automatically generate API documentation (Swagger/ReDoc).
* **Error Masking**: Never expose internal database errors or stack traces to the public response; map them to generic error messages.

---

## 7. Database Standards

* **Naming Conventions**: Use `snake_case` naming and pluralized table names (e.g., `user_roles`).
* **Keys**: Prefer `UUIDv4` primary keys. Foreign key relationships are mandatory for data consistency.
* **Performance**: Every query filter column must have a matching database index.
* **Deletions**: Do not delete data directly; use a `deleted_at` soft-delete column to preserve audit history.
* **Audit Columns**: Every table must contain audit columns: `created_at`, `updated_at`, `created_by`, `updated_by`.
* **Database Migrations**: Database changes must be migration-based. Direct modifications to production databases are strictly prohibited.

---

## 8. Frontend Engineering Standards

* **Feature-First Architecture**: Group code by features (e.g., `features/auth/`, `features/dashboard/`) rather than putting everything in generic folders.
* **Reusable Components**: Abstract UI inputs, dropdowns, and layouts into highly reusable components.
* **Lazy Loading & Code Splitting**: Lazy-load all page routes. Avoid bundling the entire application into a single load file.
* **Accessibility (a11y)**: Enforce standard accessibility attributes (ARIA labels, keyboard controls, semantic markup).
* **Server State Control**: Use React Query (TanStack Query) to manage all caching, loading states, and remote syncs. Keep local client states (Zustand) minimal.
* **Form Validation**: Validate all inputs at the UI level (using tools like React Hook Form + Zod) before sending API requests.
* **Error Control**: Wrap page modules in Error Boundaries and React Suspense/Skeleton templates.

---

## 9. Backend Engineering Standards

* **Layer Isolation**: Follow Clean Architecture rules. Controllers call Services, which fetch from Repositories.
* **DTO Mapping**: Mapping database entities directly to API responses is prohibited. Map database results to clean Data Transfer Objects first to avoid leaking structural info.
* **Robust Auth**: Protect API resources behind standard JWT/OAuth authorization schemas. Enforce Role-Based Access Control (RBAC) validations on all handlers.
* **Background Tasks**: Offload heavy calculations, email notifications, and data updates to asynchronous message queues (RabbitMQ, Kafka, Celery).
* **Health Checks**: Implement `/health` and `/ready` endpoints for monitoring.
* **Logs**: Write structured log streams (JSON formatting) to support parsing tools (e.g., Datadog, ELK).
* **Graceful Shutdown**: Intercept termination signals (`SIGTERM`, `SIGINT`) to finish running queries and close database pools cleanly before exiting.

---

## 10. Security Mandates

* **Secrets Management**: Storing API keys, server passwords, database credentials, or secret variables inside source code is strictly prohibited. Load values at runtime from Environment variables.
* **Transport Encryption**: Enforce HTTPS and TLS 1.3 across all environments.
* **Encryption**: Encrypt sensitive data in databases (AES-256) and hash passwords using secure hashing algorithms (bcrypt, Argon2).
* **CSRF & XSS Prevention**: Enforce CSRF token validation and sanitize input values before rendering them in the DOM.
* **SQL Injection Prevention**: Use ORMs or parameterized queries; never concatenate strings to run database statements.
* **Scanning**: Run automated scanners (e.g., Gitleaks, Dependabot) before merging code to identify dependency issues and prevent secret leaks.

---

## 11. Testing Strategy

```text
             ▲
            / \
           /   \      E2E Tests (<10% - Critical user paths)
          / E2E \
         /───────\
        /  INTEG  \   Integration Tests (~30% - API routes, queries)
       /───────────\
      /    UNIT     \ Unit Tests (>60% - Functions, UI components)
     /───────────────\
```

* **Coverage Gate**: Code coverage target is **>80%** (verified by Vitest or Jest on every merge pull request).
* **Unit Testing**: Focus on isolating logic blocks. Mock external resources completely.
* **Integration Testing**: Test interactions between routes, database queries, and third-party APIs.
* **Contract Testing**: Ensure backend API changes do not break frontend expectations (using tools like Pact).
* **Performance Testing**: Run stress tests (e.g., using k6) to identify load capacities and memory leaks.

---

## 12. DevOps & Deployments

* **Containerization**: Package applications in standard, lightweight Docker containers.
* **Infrastructure as Code (IaC)**: Build infrastructure using declarative tools (Terraform, CloudFormation).
* **Deployment Strategies**:
  * **Blue-Green**: Minimize downtime by maintaining two identical production environments.
  * **Canary**: Deploy updates to a small subset of users (e.g., 5%) first to verify stability.
* **Feature Flags**: Control feature availability dynamically in production using Feature flags.
* **Rollbacks**: If key error metrics spike after deployment, trigger automatic rollbacks.

---

## 13. Observability

* **Structured Logs**: Enforce JSON logging formats containing unified tracing identifiers (`correlation_id`).
* **System Metrics**: Monitor standard resource metrics (CPU, Memory, Disk, network IO).
* **Request Metrics**: Track request metrics: Latency, Error Rate, and Throughput.
* **Distributed Tracing**: Trace request journeys across services using tracing headers (OpenTelemetry).
* **Alerting Policies**: Configure critical alerts based on error thresholds, latency degradation, and service downtime.

---

## 14. Documentation Standards

* **ADR (Architecture Decision Records)**: Document all major technology, architectural, and design choices using standard ADR files.
* **API Documentation**: Maintain interactive OpenAPI specifications (Swagger).
* **Onboarding Guides**: Create standard step-by-step setup guides to help new engineers configure their local environments.
* **Runbooks**: Document operational workflows to handle system errors, server outages, and database rollbacks.

---

## 15. Code Quality Checklist

```text
┌────────────────────────┐
│      Pre-Commit        │ -> Branch Check, Secrets Scanning, lint-staged (Linter + Formatter)
└──────────┬─────────────┘
           ▼
┌────────────────────────┐
│      Pre-Push          │ -> Type Check, Unit Tests, Security Audits
└──────────┬─────────────┘
           ▼
┌────────────────────────┐
│      CI Pipeline       │ -> Build, Integration Tests, Package, Staging Smoke Test
└────────────────────────┘
```

Every code change must pass:
1. **Linting Check**: ESLint validations with zero errors.
2. **Formatting**: Automatic formatting using Prettier rules.
3. **Type Checking**: Clean compiler builds (`tsc --noEmit`).
4. **Vitest Unit Tests**: All tests pass.
5. **Coverage Gate**: Code coverage stays above 80%.
6. **Vulnerability Audit**: High or critical dependencies checked and updated.

---

## 16. Git Workflow

* **Main Branch (`main`)**: The production-ready branch. Direct push is blocked.
* **Develop Branch (`develop`)**: The target integration branch for feature branches.
* **Branch Naming**:
  * `feature/feat-name` — Adding a new feature.
  * `bugfix/bug-desc` — Fixing an issue.
  * `hotfix/issue-desc` — Emergency production fix.
* **Conventional Commits**: Commit messages must follow standard conventions:
  * `feat(scope): message` — For new features.
  * `fix(scope): message` — For bug fixes.
  * `docs(scope): message` — For documentation updates.
  * `refactor(scope): message` — For code updates.
* **Pull Request Requirements**:
  * Block direct pushes to protected branches.
  * Require at least one peer approval.
  * Require successful CI checks before merging.

---

## 17. CI/CD Pipeline Stages

```text
    Local Developer Branch
               ↓
    Pull Request Opened to main
               ↓
      CI Stage 1: Checkout Code
               ↓
      CI Stage 2: Install Dependecies
               ↓
      CI Stage 3: Run Linter & Formatter
               ↓
      CI Stage 4: Run Type Checker
               ↓
      CI Stage 5: Run Unit & Integration Tests
               ↓
      CI Stage 6: Security & Dependency Scan
               ↓
      CI Stage 7: Compile Build Bundle
               ↓
     CD Stage 1: Push Build to Registry
               ↓
     CD Stage 2: Deploy to Staging Environment
               ↓
     CD Stage 3: Run Automated Smoke Tests
               ↓
    Manual / Gated Production Release
               ↓
    CD Stage 4: Deploy to Production
               ↓
    CD Stage 5: Post-Deployment Smoke Test
```

---

## 18. Performance Standards

* **Lazy Loading**: Enforce dynamic imports for components that aren't visible on initial page load.
* **Code Splitting**: Split JS bundles into page-specific chunks to speed up page loads.
* **Caching**: Set up correct caching policies (Redis, browser cache headers) for static files and common query requests.
* **Bundle Budget**: Set maximum limits for initial JS bundles (e.g. <300 KB). Run bundler analysis during build steps.
* **Optimizations**: Enforce asset compression (Gzip/Brotli) and utilize next-gen image formats (WebP).

---

## 19. Engineering Checklist (Definition of Ready)

```text
[ ] Requirements are complete and approved.
[ ] Architecture design has been reviewed and approved.
[ ] Git repository has been created.
[ ] Documentation is prepared.
[ ] CI/CD workflows are configured.
[ ] Linters and formatters are configured and active.
[ ] Type check configurations are active.
[ ] Unit test setup is complete.
[ ] Security checks and secrets scanning are configured.
[ ] Observability, logging, and error handling are configured.
[ ] Performance, accessibility, and compatibility reviews are complete.
[ ] Peer review is complete.
[ ] Production readiness checks are complete.
```

---

## 20. Definition of Done (DoD)

A task is considered **Done** only when:
```text
[ ] Feature implementation is complete.
[ ] All automated tests pass successfully.
[ ] Code coverage meets the target gate (>80%).
[ ] Peer review has been approved.
[ ] Documentation (README, ADRs, Swagger) is updated.
[ ] Secrets scanning has checked out clean.
[ ] Code performance has been verified.
[ ] Logging, metrics, and observability tracers are active.
[ ] CI/CD pipeline builds successfully.
[ ] Deployment to Staging is complete.
[ ] Automated smoke tests pass on Staging.
[ ] Deployment to Production is complete and verified.
```

---

```text
====================================================================================
ENGINEERING MANTRA
====================================================================================

Build software that is simple to understand, easy to maintain, secure by default,
well-tested, observable in production, and scalable as the team and business grow.

Every repository should be consistent.

Every engineer should follow the same standards.

Automation should replace manual work wherever possible.

Quality is built into the process—not inspected at the end.

====================================================================================
END OF ENGINEERING HANDBOOK
====================================================================================
```
