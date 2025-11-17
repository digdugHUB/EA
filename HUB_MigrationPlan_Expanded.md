# HUB International — AWS Migration & Modernization Plan
### Least Path of Resistance Strategy (RKGore Format)
Prepared for: scott.simpson1@hubinternational.com  
Prepared by: Douglas (Lead Architect) & Pavan (Cloud Engineer)

---

# 0. Glossary of Terms & Acronyms

| Acronym / Term | Meaning |
|----------------|---------|
| Legacy Admin Portal | MS Access → .NET 4.6 (Razor, Kendo UI, KnockoutJS, SQL stored procedures) |
| Modern Admin SPA | EmberJS SPA modules hosted on S3, integrated into legacy admin portal |
| Client Portal | .NET 4.6 Web API, EmberJS frontend, EF6 ORM |
| SPA | Single Page Application |
| EF6 | Entity Framework 6 ORM |
| WAF | AWS Web Application Firewall |
| Scribe | Process/Workflow Documentation Tool |
| Swagger | REST API (OpenAPI) documentation |
| RI / SP | Reserved Instances / Savings Plans |
| CE | AWS Cloud Economics |
| SecArch | Security Architecture |

---

# 1. Executive Overview

The enterprise currently operates three interdependent application stacks, each with unique modernization needs and technical debt.

## 1.1 Legacy Admin App (High Risk)

- Originally MS Access  
- Migrated to .NET 4.6 (architecture similar to .NET 3.5)  
- Razor Views with Kendo UI  
- KnockoutJS (deprecated)  
- Heavy SQL stored-procedure-driven logic  
- Minimal documentation

## 1.2 Modern Admin SPA Modules (Medium Risk)

- EmberJS SPAs hosted on S3  
- Integrated into legacy admin ecosystem  
- Backend still relies on legacy .NET APIs  
- Fragmented modernization effort

## 1.3 Client Portal (Moderate Risk)

- .NET 4.6 Web API  
- EmberJS frontend  
- EF6 ORM  
- Better structured but aging stack

---

# 2. Migration Phases

## 2.1 Phase 1 — Review (Discovery, Documentation, Visibility)

### 2.1.1 AWS Contract & Support

- Validate AWS billing ownership  
- CE to confirm Enterprise Support availability  
- CE to confirm Enterprise Discount Program eligibility  

### 2.1.2 Access Requirements

- Read-only IAM access across all AWS accounts  
- If delayed, request Trusted Advisor Report  

### 2.1.3 Documentation Requirements

#### Legacy Admin Portal

- Scribe workflow documentation  
- Logic dossiers  
- Full data dictionary in Markdown  
- Stored procedure mapping  
- Inventory of deprecated KnockoutJS modules  
- Razor/Kendo view mapping  

#### Modern Admin SPAs

- Full SPA inventory  
- API dependency mapping  
- Compatibility checks with legacy API endpoints  
- Update and align OpenAPI/Swagger documentation  

#### Client Portal

- Updated Swagger documentation  
- EF6 schema relationship mapping  
- Backend controller documentation  
- EmberJS architecture overview  

### 2.1.4 Architecture Documentation (Requested)

- Cloud infrastructure diagram  
- Application dependency diagram  
- API call graph  
- DR/Backup strategy  
- AWS OU and SCP structure  
- Region usage analysis  

---

## 2.2 Phase 2 — Change Plan (Security, Stability, Modernization)

### 2.2.1 Security Architecture

- Enable/verify root-access email recovery  
- Review penetration test findings  
- Review Black Duck (OSS/Vulnerability) results  
- Address deprecated libraries in frontend  
- Harden endpoints and SQL injection surfaces  
- Update WAF rulesets  
- Validate IAM role consistency  

---

# 3. Stack Complexity Overview (Expanded)

This section expands the stack analysis from a high-level overview into detailed architectural, operational, and risk dimensions.

---

## 3.1 Legacy Admin App — Deep Dive (High Complexity, High Risk)

The legacy admin stack is the oldest, least documented, and most fragile component of the ecosystem. It originated from MS Access, was lifted into a .NET 4.6 Web Application, and contains many years of accumulated logic, shortcuts, and technical debt.

### 3.1.1 Architectural Components

- .NET 4.6 application using pre-MVC design patterns  
- Razor Pages with inline logic  
- Kendo UI (legacy versions)  
- KnockoutJS (abandoned framework)  
- SQL Server as primary logic engine  
- Significant business logic inside stored procedures  
- Script-heavy frontend with synchronous calls  

### 3.1.2 Key Risks and Challenges

1. Logic buried in SQL  
   - Critical business logic lives in stored procedures.  
   - No standardized naming and limited documentation.  
   - Difficult to unit test or validate changes.

2. Deprecated frontend layers  
   - KnockoutJS is no longer supported.  
   - Kendo UI version is outdated, with potential security issues.  
   - Razor views mix UI and business logic.

3. Fragile build and deployment  
   - Manual EC2 deployments.  
   - No repeatable build artifacts.  
   - No CI/CD pipeline enforcement.

4. Outdated authentication/authorization  
   - Likely forms-based auth or custom cookie flows.  
   - No OIDC/OAuth2 federation.  
   - Limited MFA enforcement.

5. Browser compatibility issues  
   - Heavy reliance on jQuery-era DOM patterns.  
   - Global variables and side effects.  

### 3.1.3 Hidden Failure Points

- Circular dependencies between UI and stored procedures.  
- Hard-coded values for configuration and environment.  
- Use of SQL cursors and nested loops causing performance and locking issues.  
- Session state tied to IIS worker process, impacting scalability.  

### 3.1.4 Modernization Blockers

- Business logic must be extracted from SQL before a clean API layer can be built.  
- UI replacement requires a decoupled service layer.  
- Security posture of deprecated frameworks forces earlier change than desirable.  

### 3.1.5 Recommended Migration Strategy

- Freeze new feature development on the legacy admin app.  
- Map stored procedures to features and screens.  
- Build a new .NET 8 service layer that encapsulates business logic.  
- Shift the UI to React/Next.js or Ember LTS, backed by the new service layer.  
- Run legacy and new flows in parallel until confidence is achieved.  
- Decommission legacy pages and procedures incrementally.

---

## 3.2 Modern Admin SPA Modules — Deep Dive (Medium Complexity)

The modern admin SPA modules are an attempt to modernize, but they created a fragmented architecture with mixed paradigms.

### 3.2.1 Architectural Components

- EmberJS SPAs hosted on S3  
- Served via CloudFront  
- Backend still uses legacy .NET 4.6 APIs  
- Some SPAs rely on different Ember versions  
- Each SPA maintained as a separate codebase  

### 3.2.2 Strengths

- Better UX and performance than legacy pages.  
- Clearer client-side routing and state management.  
- Separation of concerns between API and UI (conceptually).  

### 3.2.3 Key Risks

1. Architectural fragmentation  
   - Multiple Ember versions in production.  
   - No shared component library or design system.  
   - APIs are not fully standardized.

2. Hard-coded endpoints  
   - Service URLs embedded in SPA code.  
   - Environment switching is fragile.

3. Lack of CI/CD and testing  
   - Manual or inconsistent deployment patterns.  
   - Limited unit/integration testing.

4. Limited observability  
   - No standardized error collection or tracing.  

### 3.2.4 Modernization Blockers

- Dependency on legacy backend responses.  
- SPA modernization is gated by API modernization.  

### 3.2.5 Recommended Migration Strategy

- Consolidate admin SPAs into a unified monorepo.  
- Introduce a shared design system and component library.  
- Rebuild backend APIs in .NET 8 with OpenAPI contracts.  
- Stand up CI/CD for build/test/deploy of SPAs.  
- Optionally migrate SPAs to Ember LTS or React.  

---

## 3.3 Client Portal — Deep Dive (Moderate Complexity)

The client portal is more modern and structured but still depends on aging frameworks and runtime versions.

### 3.3.1 Architectural Components

- .NET 4.6 Web API  
- EmberJS frontend  
- EF6 ORM with LINQ  
- Hosted on EC2 for backend and S3/CloudFront for frontend  
- Protected by AWS WAF  

### 3.3.2 Strengths

- Reasonably clean API layer.  
- ORM-based data access via EF6.  
- Clear separation of frontend and backend.  

### 3.3.3 Risks

1. .NET 4.6 end-of-support  
   - No new patches or performance improvements.  
   - Incompatible with some modern libraries and patterns.

2. EF6 performance and capability limits  
   - Limited async support.  
   - Less efficient query translation.

3. Ember version issues  
   - Older versions can be hard to maintain.  
   - May require major upgrades for ecosystem compatibility.

4. Deployment technical debt  
   - EC2-based hosting with manual scaling.  
   - No autoscaling or container orchestration.  

### 3.3.4 Modernization Blockers

- EF6 to EF Core migration requires careful refactor.  
- Business logic in controllers needs to be moved to separate services.  
- Ember upgrade requires coordinated change across routes, services, and models.  

### 3.3.5 Recommended Migration Strategy

- Lift-and-shift API to Windows containers as an interim step.  
- Incrementally refactor to .NET 8.  
- Upgrade data access from EF6 to EF Core 8.  
- Upgrade Ember frontend or migrate to React, using the same modern API.  
- Move hosting from EC2 to ECS Fargate for better scaling.  

---

## 3.4 Cross-Stack Complexity Summary

| Stack              | Risk Level | Key Issues                                                  | Modernization Need                      |
|--------------------|-----------:|-------------------------------------------------------------|-----------------------------------------|
| Legacy Admin App   |  Very High | SQL logic, deprecated JS, no CI/CD, security exposures     | Full rewrite and logic extraction       |
| Modern Admin SPA   |     Medium | Fragmented SPAs, inconsistent API contracts, version drift | Consolidation and new unified APIs      |
| Client Portal      |   Moderate | .NET 4.6, EF6, older Ember, manual scaling                 | Upgrade runtime, ORM, and frontend      |

---

## 3.5 Overall Modernization Priority Order

1. Legacy Admin App (highest risk and tech debt).  
2. Backend API layer (shared dependency across portals and SPAs).  
3. Admin SPA consolidation and modernization.  
4. Client Portal incremental upgrades.

---

## 3.6 SQL Stored Procedures — Complexity, Lift, and Cost to Decouple

A significant portion of the application’s business logic currently lives inside SQL Server stored procedures, rather than in a testable, versioned service layer. This has a direct impact on effort, risk, cost, and sequencing of the migration.

### 3.6.1 Current Use of Stored Procedures

- Business rules are implemented directly in stored procedures.  
- Stored procedures often:
  - Perform multi-step workflows (read, validate, transform, write).  
  - Contain conditional branching, loops, and cross-table logic.  
  - Implement domain rules (eligibility, pricing, status transitions, etc.).  
- Many procedures are:
  - Poorly or inconsistently named.  
  - Lightly documented or not documented at all.  
  - Reused by both the Legacy Admin App and the Client Portal.

This makes the database the de facto application server, which is not a scalable or maintainable pattern.

### 3.6.2 Why Stored Procedures Increase Complexity

1. Hidden business logic  
   - Logic is not visible at the API or application layer.  
   - Harder for new developers to reason about behavior.  
   - Requires DB-level expertise for every change.

2. Tight coupling to schema  
   - Procedures assume a specific table and column structure.  
   - Schema changes risk breaking multiple procedures.  
   - Refactoring becomes high-risk and time-consuming.

3. Limited testability  
   - No straightforward unit test harness.  
   - Testing often requires full data sets and manual validation.  
   - No CI/CD integration for logic changes, increasing regression risk.

4. Performance side effects  
   - Use of cursors, nested loops, and procedural logic inside SQL.  
   - Potential for long-running locks and deadlocks.  
   - Difficult performance troubleshooting and tuning.

5. Security and auditability  
   - Fine-grained permissions are harder to manage.  
   - Authorization cannot be easily centralized at the API layer.  
   - Hard to trace which feature or user triggered which DB behavior.

---

### 3.6.3 Decoupling Strategy — Phases

Decoupling logic from stored procedures should be treated as a multi-phase project:

#### Phase 1 — Inventory and Classification

- Identify all stored procedures used by:
  - Legacy Admin App  
  - Modern Admin SPAs  
  - Client Portal  
- Classify each stored procedure:
  - Read-only vs. read/write vs. workflow.  
  - Low, medium, or high complexity (branching, joins, loops).  
  - Critical-path vs. non-critical.

#### Phase 2 — Behavior Documentation

- For high- and medium-impact procedures:
  - Document input parameters and outputs.  
  - Document business rules, conditions, and error handling.  
  - Map each procedure to the UI screens/APIs that invoke it.

#### Phase 3 — Service-Layer Extraction

- Implement equivalent logic in a new .NET 8 service layer:
  - Use domain and application service patterns.  
  - Convert SQL branching logic into C# control flow.  
  - Reduce remaining stored procedures to pure data-access helpers (CRUD).

- Introduce integration tests comparing legacy SP behavior vs. new service layer behavior.

#### Phase 4 — Cutover and Decommission

- Gradually switch calls from:
  - UI/API → stored procedure  
  - To: UI/API → .NET 8 service layer → simpler queries  

- Run legacy stored procedures in read-only or shadow mode for a controlled period.  
- Decommission unused or fully replaced stored procedures.

---

### 3.6.4 Effort and Cost Drivers

The lift and cost of decoupling logic from stored procedures is driven by:

- Number of stored procedures in active use.  
- Complexity per procedure (branching, loops, joins, transactions).  
- Documentation quality.  
- Availability of subject matter experts (SMEs).  
- Required duration for parallel run and rollback safety.

---

### 3.6.5 T-Shirt Sizing for Procedure Migration

These are rough, planning-level buckets:

- Small (S)  
  - Simple lookups.  
  - 1–2 tables, no branching.  
  - Typical effort: 2–4 hours each.

- Medium (M)  
  - Multiple joins, basic conditional logic.  
  - Some data shaping.  
  - Typical effort: 1–2 days each.

- Large (L)  
  - Complex branching, loops, cross-table workflows.  
  - Business-critical use cases.  
  - Typical effort: 3–5 days each.

- Extra Large (XL)  
  - Orchestration of multi-step workflows across domains.  
  - Heavy usage and high business impact.  
  - Typical effort: 1–3 weeks including SME time and full regression testing.

---

### 3.6.6 Business Impact of Decoupling

Short-term impact:

- Increased project time and cost during migration.  
- Higher coordination load between DBAs, developers, and SMEs.  
- Need for staged releases and extensive regression testing.

Long-term benefits:

- Business logic becomes:
  - Testable.  
  - Version-controlled.  
  - Easier to understand and change.  

- Database becomes a persistence layer instead of an application server.  
- Easier to:
  - Introduce new UIs, integrations, and services.  
  - Scale horizontally at the application tier.  
  - Adopt cloud-native patterns and multi-region architectures.

---

### 3.6.7 Stored Procedure Decoupling — Before vs. After (Mermaid Diagram)

```mermaid
flowchart LR
    subgraph Before
        UI_B[UI / SPA]
        API_B[Legacy .NET API]
        DB_B[(SQL Server)]
        SP_B[[Stored Procedures (Logic + Data)]]

        UI_B --> API_B
        API_B --> SP_B
        SP_B --> DB_B
    end

    subgraph After
        UI_A[UI / SPA (Modern)]
        API_A[.NET 8 API Layer]
        SVC_A[Domain Service Layer]
        DB_A[(SQL Server)]
        SP_A[[Stored Procedures (Data Access Only)]]

        UI_A --> API_A
        API_A --> SVC_A
        SVC_A --> SP_A
        SP_A --> DB_A
    end
```

---

# 4. AWS Infrastructure Review

Current State:

- Primary region: us-east-1  
- Secondary regions used for historical tests  
- DNS hosted in Cloudflare  
- EC2 for backend  
- S3 + CloudFront for frontend  
- AWS WAF for protection  

Gaps:

- No multi-region DR  
- No multi-AZ standardization  
- No containerization  
- No CI/CD  
- No Terraform or other IaC tooling  

Modernization Path:

- Consolidate workloads into a single primary region.  
- Implement multi-AZ high availability.  
- Add containerization (ECS Fargate recommended).  
- Introduce Terraform for IaC.  
- Use GitHub Actions or AWS CodePipeline for CI/CD.  
- Unify monitoring with CloudWatch and OpenSearch.

---

# 5. DevOps Modernization

Current Gaps:

- Manual EC2 deployments  
- No automated build or test pipelines  
- No Docker/container artifacts  
- No environment parity  
- No version tagging or rollback capability  

Recommended Approach:

- Use GitHub Actions for build/test.  
- Use AWS CodePipeline for deployments.  
- Store artifacts and images in ECR.  
- Deploy workloads on ECS Fargate.  
- Use Terraform for environment consistency.  
- Introduce API Gateway to unify routing and observability.  

---

# 6. Security & Compliance

Priority Fixes:

- Patch .NET Framework dependencies.  
- Completely remove KnockoutJS.  
- Harden SQL stored procedure access and usage.  
- Enforce MFA and IAM role hygiene org-wide.  
- Implement WAF managed rule groups.  
- Evaluate AWS Shield Advanced.  
- Address penetration test findings.  
- Address Black Duck and other OSS vulnerability findings.

---

# 7. Action Items

## 7.1 Tammy’s Team Responsibilities

- Document admin portal via Scribe.  
- Document Lambda functions.  
- Update Swagger for all APIs.  
- Provide EF6 schema and ERD.  
- Provide complete data dictionary.  
- Provide Git repository access.

## 7.2 Douglas & Pavan Responsibilities

- Review penetration test and Black Duck findings.  
- Audit repositories and identify dead code.  
- Validate Cloudflare DNS access.  
- Map AWS region usage and recommend consolidation.  
- Begin modernization blueprint creation.  
- Recommend budget and timeline for staged implementation.

---

# 8. Final Summary

The current environment includes:

- Multiple legacy .NET systems.  
- Fragmented modernization efforts across SPAs.  
- Outdated frameworks and dependencies.  
- Significant reliance on stored procedures for business logic.  
- Incomplete or missing documentation.  
- Lack of containerization, CI/CD, and infrastructure as code.  
- Security exposures linked to aged components and patterns.

The proposed strategy delivers:

- A documentation-first approach.  
- A phased modernization path.  
- Migration from .NET 4.6 to .NET 8.  
- SPA consolidation and modernization.  
- Containerized AWS architecture on ECS Fargate.  
- Centralized security hardening and best practices.  
- Repeatable DevOps and Terraform-based infrastructure.  
- A clear path to decouple logic from stored procedures into a robust service layer.

This establishes a stable, secure, and future-proof ecosystem for HUB International.

---

# 9. Optional Future Deliverables (On Request)

- Architecture diagrams (additional Mermaid diagrams, Markdown-native).  
- RACI matrix for modernization and operations.  
- 30/60/90-day migration timeline.  
- Executive business-case slides (Markdown or PowerPoint-ready outline).  
- Fully packaged GitHub-ready documentation repository.

