# 🏛️ VaultCore — Enterprise Metadata-Driven Document Management & Workflow Vault

> **Internship Project — Tofaş IT**

An enterprise-grade, metadata-driven Document Management System (DMS) and Workflow Automation platform inspired by M-Files architecture. Built from the ground up to replace rigid folder-based storage with an object-oriented, property-driven paradigm featuring runtime schema configuration, dynamic form rendering, materialized ACL permission engine, full-lifecycle versioning with check-out locking, full-text search with OCR, and enterprise workflow orchestration.

---

## ⚠️ Confidentiality Notice

This project was developed during my software engineering internship at Tofaş.

Due to corporate confidentiality, intellectual property, and internal security requirements, original production data, internal infrastructure URLs, proprietary credentials, and company-specific configurations are **strictly excluded**.

This repository serves as a **sanitized technical presentation and portfolio showcase** containing architecture documentation, engineering decisions, and selected user interface screenshots.

---

## 📌 Project Overview & Paradigm Shift

Traditional document management systems rely on hierarchical folder trees, which introduce deep operational inefficiencies in enterprise environments: documents are duplicated across department folders, access control becomes fragmented, and categorizing a document under multiple dimensions (e.g., Department, Brand, Year, Project) is impossible without redundancy.

**VaultCore eliminates rigid folder hierarchies entirely:**
* **No Folders, Only Objects & Properties:** Documents and business entities are defined strictly by their metadata attributes.
* **Saved Views as Dynamic Filters:** Virtual "folders" are dynamic, real-time database queries (`SavedView`) based on property criteria.
* **Runtime Schema Modeling:** Administrators can define new `ObjectType`, `ObjectClass`, `PropertyDefinition`, and `ValueList` entities directly from the web interface without writing a single line of backend code or running database migrations.
* **Materialized Access Control (ObjectAcl):** Security rules are evaluated upon document creation/modification and materialized into optimized lookup tables, eliminating heavy query-time permission evaluations.
* **Data-Driven Workflow Automation:** Business workflows, SLA deadlines, electronic signature checkpoints, and automatic task assignments are modeled as configurable state transitions rather than hardcoded conditional logic.

```text
┌─────────────────────────────────────────────────────────────────────────────┐
│                             VaultCore Workflow Cycle                        │
└─────────────────────────────────────────────────────────────────────────────┘
  1. Dynamic Schema Creation (Admin defines Classes & Properties via UI)
         ↓
  2. Document Intake & Metadata Indexing (Dynamic Form + SHA-256 CAS File Storage)
         ↓
  3. FTS & OCR Processing (PostgreSQL turkish_unaccent + Tesseract OCR Engine)
         ↓
  4. Materialized Security Assignment (ObjectAcl Engine & Rule Evaluation)
         ↓
  5. Workflow & Task Automation (State Transitions, SLA Tracking, Task Assignments)
         ↓
  6. Collaborative Governance (Check-Out Lock, Versioning, Append-Only Audit Trail)
```

---

# ✨ Key System Capabilities

### 🗂️ Metadata-Driven Architecture & Runtime Schema
* **Zero-Migration Schema Creation:** Define custom object types (Documents, Vehicles, Contracts, Suppliers) and classes on the fly.
* **Dynamic Property System:** Rich data types including Text, Integer, Decimal, DateTime, Boolean, Lookup (`ValueList`), and Multi-Select.
* **Dynamic Metadata Card:** The frontend dynamically constructs validation rules, dropdowns, and UI inputs at runtime based on the selected object class.

### 🔒 Materialized Security Engine & Dynamic ACL
* **Rule-Based Permissions:** Granular access rules based on user roles, group memberships, and document metadata properties.
* **Materialized `ObjectAcl`:** Permissions are pre-computed upon record changes, guaranteeing sub-millisecond query filtration even across hundreds of thousands of records.
* **Security & ACL Diagnostics:** Built-in analytical heatmaps, Allow/Deny distribution graphs, and user permission simulation tools for compliance teams.
* **Zero-Leakage Guarantee:** Unauthorized requests (`403 Forbidden`) return completely empty response bodies to prevent metadata enumeration.

### 🔄 State-Machine Workflow Engine & Automation
* **Visual Workflow Designer:** Interactive drag-and-drop workflow modeling with configurable states and directional transitions.
* **Automated Task Generation:** State transitions automatically assign tasks to specific users, groups, or department heads with SLA deadlines.
* **Electronic Approvals:** Critical state transitions require password re-authentication for cryptographic accountability.
* **Multi-User Resolution Policies:** Flexible task completion criteria (First-to-Complete vs. Consensus / All-Must-Complete).

### 📄 Version Control & Check-Out Locking
* **Immutable Version History:** Every document check-in creates an immutable snapshot (`ObjectVersion`); historical versions are never overwritten or deleted.
* **Check-Out Concurrency Lock:** Prevents edit collisions with TTL (Time-To-Live) expirations and administrative override capabilities.
* **Content-Addressable Storage (CAS):** File binaries are stored in isolated file storage indexed by their SHA-256 hash digest, guaranteeing deduplication and total path-traversal immunity.

### 🔍 Full-Text Search (FTS) & Tesseract OCR
* **PostgreSQL GIN Search:** Native Turkish full-text search using `turkish_unaccent` configuration and trigram indexing.
* **Optical Character Recognition (OCR):** Automated text extraction from scanned images and PDF files via background workers.
* **AI Metadata Suggestions:** Intelligent property value recommendations based on extracted document content.

### 🌐 Interoperability & Governance
* **WebDAV Endpoint:** Native file system mounting with Basic Authentication and lock management.
* **WOPI Host Integration:** Standardized interface for web-based Office document viewing and collaborative editing.
* **Append-Only Audit Trail:** Immutable event logging (`AuditEntry`) recording every read, write, transition, and download with JSON payload diffs.
* **User Delegation / Proxy System:** Automated task delegation during planned leaves or absences.

---

# 🖥️ Application Interface Walkthrough

## 1. Authentication & User Profile Management

### 🔐 Enterprise Login
The authentication portal features a modern glassmorphic interface with rate-limiting protection against brute-force attacks, multi-role authentication, and secure JWT token lifecycle management.

![Enterprise Login](screenshots/01_giris_ekrani.png)

---

### 👤 User Profile & Role Settings
Users can view their assigned organizational roles, view active security tokens, and update profile credentials.

![User Profile](screenshots/10_profilim.png)

---

### 🛡️ Task Delegation & Proxy Management
Users can set up temporary or scheduled delegations, transferring task-handling authority to a colleague during vacation or business travel while maintaining complete audit traceability.

![Task Delegation](screenshots/11_profilim_guvenlik_vekalet.png)

---

## 2. Executive Dashboard & Workflow Hub

### 📊 Executive Dashboard
The operational command center displays real-time KPI metrics, pending task counts, overdue SLA warnings, and dynamic distribution charts across all active business processes.

![Executive Dashboard](screenshots/02_gosterge_paneli.png)

---

### 🔀 Workflow Overview (L1 Hub)
A high-level catalog summarizing all active workflows across the enterprise, categorized by status badges, active document counts, and health indicators.

![Workflow Hub](screenshots/03_akislar_ana_sayfa.png)

---

### 📑 Contract & Financial Approval Workflow (L2 Process)
Detailed operational tracking of corporate agreements, purchasing contracts, and financial documents undergoing multi-tier review, legal checks, and executive sign-offs.

![Contract & Financial Approval](screenshots/06_akis_2_sozlesme_mali_onay.png)

---

### 🔧 Technical Service & Warranty Workflow (L2 Process)
Real-time tracking of technical fault reports, service quality inspections, warranty claims, and engineering escalations.

![Technical Service & Warranty](screenshots/07_akis_3_teknik_servis_garanti.png)

---

### 📥 My Open Tasks
A centralized inbox displaying all pending approval requests, document reviews, and assigned workflow steps with deadline countdowns and priority markers.

![My Open Tasks](screenshots/08_gorevlerim_acik_gorevler.png)

---

### ✅ Completed Task History
An auditable record of all previously processed and approved tasks, including completion timestamps, execution comments, and decision outcomes.

![Completed Tasks](screenshots/09_gorevlerim_tamamlanan_gorevler.png)

---

## 3. Object Explorer & Dynamic Document Intake

### 🗂️ Object Explorer (M-Files Virtual Layout)
The central exploration interface featuring dynamic saved-view hierarchies on the left, an interactive document data table in the center, and a live metadata & version inspection drawer on the right.

![Object Explorer](screenshots/12_nesne_gezgini.png)

---

### 📝 Dynamic Document Creation Form
When a user selects a document class, the form dynamically constructs required fields, validation constraints, date pickers, and lookup dropdowns based on the class's runtime schema.

![Dynamic Document Creation](screenshots/13_yeni_dokuman_olusturma_formu.png)

---

## 4. Administrative Structure & Dynamic Schema Management

### 🏷️ Vault Structure — Classes & Workflow Binding
Administrators configure document classes, associate them with default workflow state machines, and set operational rules without database migrations.

![Vault Classes](screenshots/15_admin_vault_yapisi_siniflar.png)

---

### 📦 Vault Structure — Object Types
Management of fundamental system entities (e.g., Documents, Contracts, Equipment, Customers) and their storage behavior.

![Object Types](screenshots/16_admin_vault_yapisi_nesne_turleri.png)

---

### ⚙️ Vault Structure — Property Definitions
Definition of system-wide metadata fields, configuring their underlying data types, default values, and lookup associations.

![Property Definitions](screenshots/17_admin_vault_yapisi_ozellik_tanimlari.png)

---

### 📋 Vault Structure — Value Lists
Centralized management of standard lookup lists (e.g., Departments, Brands, Document Types, Confidentiality Levels) consumed by metadata properties.

![Value Lists](screenshots/18_admin_vault_yapisi_deger_listeleri.png)

---

## 5. Security Engine, Dynamic ACL & Diagnostics

### 🛡️ Permission Rules Matrix
Granular matrix defining conditional access policies based on the intersection of user roles, user groups, and metadata attributes.

![Permission Rules Matrix](screenshots/19_admin_izin_kurallari.png)

---

### 🗺️ ACL Complexity & Heatmap Analytics
A visual security analysis tool displaying permission rule distribution, Allow/Deny policy ratios, and identifying the most complex security objects in the system.

![ACL Analytics Map](screenshots/26_admin_acl_analitik_haritasi.png)

---

### 🔬 ACL Diagnostics & User Access Evaluation
An administrative diagnostic simulator allowing security officers to select any user and inspect exactly how permission rules match against target documents.

![ACL Diagnostics](screenshots/27_admin_acl_tanilama_kullanici_sonucu.png)

---

## 6. Visual Workflow Designer & Step Automation

### 🎨 Visual Workflow Designer
An administrative workflow builder for designing state machine flows, connecting review steps, defining transition criteria, and setting automated triggers.

![Workflow Designer](screenshots/20_admin_is_akislari_tasarimcisi.png)

---

### ⏱️ Workflow Step Details & SLA Configuration
Configuration interface for individual workflow states, setting up automatic group/user assignments, mandatory comment flags, and SLA deadline durations.

![Workflow Step Details](screenshots/21_admin_is_akislari_adim_detaylari.png)

---

## 7. Enterprise Governance, Auditing & Identity

### 📜 System Audit Trail
An immutable, append-only security log recording every transaction across the platform (user logins, document reads, check-outs, metadata modifications, downloads, deletions).

![System Audit Trail](screenshots/22_admin_denetim_izi.png)

---

### 🔍 Audit Event Detail & Payload Inspection
A deep-dive inspection modal displaying raw JSON request payloads, before/after attribute diffs, client IP addresses, and user-agent strings.

![Audit Event Detail](screenshots/23_admin_denetim_izi_detay_modali.png)

---

### 👥 User Management
Administrative portal for provisioning user accounts, assigning global roles (Admin, ContentManager, StandardUser), and configuring organizational affiliations.

![User Management](screenshots/24_admin_kullanici_yonetimi.png)

---

### 🏢 Group & Department Management
Management of organizational units, cross-functional squads, and approval committees used for workflow routing and ACL rules.

![Group Management](screenshots/25_admin_grup_yonetimi.png)

---

# 🏗️ System Architecture

VaultCore is architected as a **Modular Monolith based on Clean Architecture principles**, ensuring strict separation of concerns, single-direction dependency flow, and domain isolation.

```text
┌─────────────────────────────────────────────────────────────────────────────┐
│                         Angular 20 Frontend (SPA)                           │
│                                                                             │
│   • Standalone Components            • Reactive Signals & State Management  │
│   • Dynamic Metadata Card Engine     • Role & Permission Route Guards       │
│   • M-Files Explorer Layout          • HTTP Auth & Error Interceptors       │
└──────────────────────────────────────┬──────────────────────────────────────┘
                                       │
                               HTTPS / REST / JWT
                                       │
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                          VaultCore.API Layer                                │
│                                                                             │
│   • Thin REST Controllers            • [RequireObjectPermission] ACL Filter │
│   • Global Exception Middleware      • Rate Limiting & Security Headers     │
│   • Swagger / OpenAPI Docs           • Dependency Injection Configuration   │
└──────────────────────────────────────┬──────────────────────────────────────┘
                                       │
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                      VaultCore.Application Layer                            │
│                                                                             │
│   • ObjectService & SearchService    • AclEngineService (Materialization)   │
│   • WorkflowService & TaskManager    • MetadataSuggestionService (AI)       │
│   • Fluent Validation & DTOs         • Interface Contracts & Abstractions   │
└──────────────────┬───────────────────────────────────────────┬──────────────┘
                   │                                           │
                   ▼                                           ▼
┌───────────────────────────────────────────┐ ┌───────────────────────────────┐
│             VaultCore.Core                │ │   VaultCore.Infrastructure    │
│                                           │ │                               │
│  • Domain Entities & Value Objects        │ │  • EF Core 9 / PostgreSQL 16  │
│  • Domain Enums (PermissionLevel, Types)  │ │  • SHA-256 CAS File Storage   │
│  • Domain Exceptions (No Framework Deps)  │ │  • Tesseract OCR & FTS Worker │
└───────────────────────────────────────────┘ │  • WebDAV & WOPI Providers    │
                                              │  • AclRecalculationWorker     │
                                              └───────────────┬───────────────┘
                                                              │
                                      ┌───────────────────────┴────────┐
                                      ▼                                ▼
                               ┌─────────────┐                  ┌─────────────┐
                               │ PostgreSQL  │                  │ CAS Storage │
                               │  Metadata,  │                  │  SHA-256    │
                               │  ACL & FTS  │                  │  Binaries   │
                               └─────────────┘                  └─────────────┘
```

---

# 🧱 Backend Clean Architecture Breakdown

The backend solution (`VaultCore.sln`) is organized into four strictly decoupled layers:

```text
backend/src/
├── VaultCore.Core/                    # Pure Domain Layer (Zero external dependencies)
│   ├── Entities/
│   │   ├── User.cs, Group.cs, UserGroup.cs
│   │   └── Vault/                    # ObjectType, ObjectClass, PropertyDefinition,
│   │                                 # ObjectInstance, ObjectVersion, PropertyValue,
│   │                                 # ObjectAcl, Workflow, WorkflowState, AuditEntry
│   ├── Enums/                        # PermissionLevel, PropertyDataType, RoleType
│   └── Exceptions/                   # DomainException hierarchy
│
├── VaultCore.Application/             # Business Logic & Orchestration
│   ├── Interfaces/                   # IObjectService, IAclEngineService, IWorkflowService...
│   ├── Services/                     # Core business logic implementations
│   ├── DTOs/                         # Request and Response contracts
│   ├── Validators/                   # Dynamic runtime schema validators
│   └── Extensions/                   # AddApplicationServices DI wiring
│
├── VaultCore.Infrastructure/          # External Concerns & Implementations
│   ├── Data/                         # VaultDbContext, EntityTypeConfigurations
│   ├── Storage/                      # SHA-256 Content-Addressable File Storage
│   ├── Services/                     # TesseractOcrService, WopiService, WebDavService
│   ├── Workers/                      # AclRecalculationWorker, TextExtractionWorker
│   └── Migrations/                   # PostgreSQL schema migrations
│
└── VaultCore.API/                     # Presentation & API Hosting
    ├── Controllers/                  # Thin HTTP endpoints (Auth, Objects, Admin, Workflow...)
    ├── Filters/                      # RequireObjectPermissionAttribute
    ├── Middlewares/                  # GlobalExceptionMiddleware, SecurityHeadersMiddleware
    └── Program.cs                    # Minimal hosting configuration
```

---

# 🌐 Frontend Architecture (Angular 20 Standalone)

The frontend is built using **Angular 20** utilizing modern standalone components and reactive architecture:

```text
frontend/src/app/
├── core/                              # Single-instance core services & security
│   ├── guards/                       # AuthGuard, RoleGuard (UX-level routing)
│   ├── interceptors/                 # AuthInterceptor (JWT injection), ErrorInterceptor
│   ├── services/                     # AuthService (Signal-based), ThemeService, ToastService
│   └── models/                       # TypeScript interfaces & API contracts
│
├── features/                          # Domain feature modules (Lazy-Loaded)
│   ├── auth/login/                   # Enterprise authentication screen
│   ├── vault/
│   │   ├── explorer/                 # M-Files layout: View Tree + Object Grid + Details
│   │   ├── objects/                  # Dynamic metadata card & dynamic property inputs
│   │   └── views/                    # SavedView query builder & virtual folders
│   └── admin/
│       ├── structure/                # Vault structure management (Classes, Types, Properties)
│       ├── workflows/                # Visual workflow canvas & step configuration
│       ├── permissions/              # Permission matrix & ACL diagnostic heatmap
│       ├── audit/                    # Audit log explorer & JSON diff modal
│       └── users/                    # User & group management console
│
└── shared/                            # Reusable UI components
    ├── components/shell/             # Navbar, sidebar, user status header
    └── ui/                           # Modal, badge, confirmation dialog, data-table
```

---

# 🛠️ Technology Stack

| Domain | Technology / Library | Version / Details |
| :--- | :--- | :--- |
| **Backend Framework** | .NET / ASP.NET Core Web API | .NET 9 (C# 13) |
| **Frontend Framework** | Angular | Angular 20 (Standalone Components, Signals) |
| **Database** | PostgreSQL | PostgreSQL 16 (`unaccent`, `pg_trgm`) |
| **ORM & Data Access** | Entity Framework Core | EF Core 9 (Npgsql provider) |
| **Authentication & AuthZ** | JWT Bearer Tokens | Role & Dynamic ACL Materialization |
| **Full-Text Search** | PostgreSQL FTS | `turkish_unaccent` custom text configuration |
| **OCR Text Extraction** | Tesseract OCR | Tesseract 5.x C# Wrapper |
| **File Storage** | Content-Addressable Storage (CAS) | SHA-256 hash-segmented physical storage |
| **Interoperability** | WebDAV & WOPI Host API | RFC 4918 WebDAV & Microsoft WOPI protocol |
| **API Documentation** | Swagger / OpenAPI | Swashbuckle ASP.NET Core |
| **Containerization** | Docker & Docker Compose | Multi-stage Dockerfiles + Compose orchestration |
| **Testing Frameworks** | xUnit, NSubstitute, Testcontainers | 315 Backend Tests + 112 Frontend Tests |

---

# 🗄️ Relational & Metadata Data Model

The data architecture bridges relational integrity with dynamic schema flexibility:

```text
┌──────────────────┐            ┌───────────────────┐
│    ObjectType    │◄───────────┤    ObjectClass    │
└────────┬─────────┘            └─────────┬─────────┘
         │                                │
         │                      ┌─────────┴──────────┐
         │                      │   ClassProperty    │
         │                      └─────────┬──────────┘
         │                                │
         │                      ┌─────────┴──────────┐            ┌─────────────────┐
         │                      │ PropertyDefinition │───────────►│    ValueList    │
         │                      └─────────┬──────────┘            └────────┬────────┘
         │                                │                                │
         ▼                                ▼                                ▼
┌──────────────────┐            ┌───────────────────┐            ┌─────────────────┐
│  ObjectInstance  │◄───────────┤   PropertyValue   │            │  ValueListItem  │
└────────┬─────────┘            └───────────────────┘            └─────────────────┘
         │
         ├──────────────────────┬──────────────────────┬──────────────────────┐
         ▼                      ▼                      ▼                      ▼
┌──────────────────┐   ┌──────────────────┐   ┌──────────────────┐   ┌──────────────────┐
│  ObjectVersion   │   │    ObjectAcl     │   │   WorkflowTask   │   │    AuditEntry    │
│ (PropertiesJson, │   │  (Materialized   │   │ (State, Assignee,│   │  (Append-Only    │
│  VersionNumber)  │   │   Permissions)   │   │   SLA Deadlines) │   │   JSON Payloads) │
└──────────────────┘   └──────────────────┘   └──────────────────┘   └──────────────────┘
```

### Core Schema Principles:
1. **`ObjectInstance` & `ObjectVersion`:** An instance represents the logical identity of the entity. Every check-in produces a new immutable `ObjectVersion` containing the snapshot state.
2. **`PropertyValue` & JSON Cache:** `PropertyValue` is the relational source of truth linked to each version. For ultra-fast queries, `ObjectVersion.PropertiesJson` (jsonb) is atomically derived during transaction commit.
3. **`ObjectAcl` (Materialized):** Access permissions are stored as direct `(ObjectId, UserId/GroupId, PermissionLevel)` tuples. Search queries execute a single indexed `JOIN` against `ObjectAcl` instead of dynamically recalculating permission rules for millions of rows.

---

# 🔒 Security Architecture & Governance

### 1. Zero Information Leakage (403 Enforcement)
When an unauthorized user requests an object, the API returns `403 Forbidden` with an empty payload. The object is not just filtered from lists; direct ID lookups reveal zero metadata.

### 2. Centralized `[RequireObjectPermission]` Filter
Controller endpoints enforce object-level ACLs declaratively using custom action filters, eliminating manual permission checks inside controller methods:
```csharp
[HttpGet("{id}")]
[RequireObjectPermission(PermissionLevel.Read)]
public async Task<IActionResult> GetObjectById(Guid id) { ... }
```

### 3. Path Traversal Immunity (Content-Addressable Storage)
Files are saved using the SHA-256 hash of their contents (e.g., `storage/ab/cd/abcdef123...`). Original filenames are stored as metadata only, making directory traversal attacks physically impossible.

### 4. Append-Only Audit Trail
The `AuditEntry` table tracks all system modifications. Database-level constraints ensure audit records can never be updated or deleted, providing guaranteed non-repudiation for regulatory compliance.

---

# 🧠 Engineering Challenges & Solutions

## 1. Dynamic Schema without Database Migrations
* **Challenge:** Traditional enterprise apps require database migrations (`ALTER TABLE`) whenever a business department needs a new document type or field.
* **Solution:** Implemented the `ObjectType` → `ObjectClass` → `PropertyDefinition` abstraction combined with a dual-storage strategy: normalized `PropertyValue` records for relational integrity and an auto-derived `jsonb` snapshot on `ObjectVersion` for indexed JSON queries.

---

## 2. Sub-Millisecond ACL Filtering on Large Datasets
* **Challenge:** Evaluating dynamic, attribute-based access rules (ABAC) during search queries causes unacceptable latency when scaling to 100,000+ objects.
* **Solution:** Created a **Materialized ACL Engine**. Permissions are evaluated once upon document save or rule modification and stored in the indexed `ObjectAcl` table. A background worker (`AclRecalculationWorker`) handles bulk recalculations asynchronously. Load tests confirmed instant query responses across 100,000+ records.

---

## 3. Data-Driven Workflow Engine
* **Challenge:** Avoid hardcoded conditional workflows (e.g., `if (status == "Approved")`) that require continuous code deployment for every process change.
* **Solution:** Built a pure state-machine engine where workflow states, transitions, mandatory comment requirements, electronic password confirmations, and automatic task assignments are defined as database records and edited via a visual designer.

---

## 4. Full-Text Search with Turkish Morphology & OCR
* **Challenge:** Standard SQL search fails on Turkish characters (İ/i, I/ı, Ğ/g) and cannot search inside scanned documents or PDFs.
* **Solution:** Configured PostgreSQL Full-Text Search with custom `turkish_unaccent` dictionary and GIN indexing, integrated with an asynchronous Tesseract OCR extraction pipeline for scanned files.

---

## 5. Comprehensive Test Architecture
* **Challenge:** Preventing authorization regressions across complex ACL and workflow branches.
* **Solution:** Developed an extensive automated testing suite:
  * **315 Backend Tests:** Unit tests with custom fakes/NSubstitute, and API integration tests using **Testcontainers PostgreSQL** + `WebApplicationFactory` to validate real HTTP authorization responses against isolated database containers.
  * **112 Frontend Tests:** Jasmine/Karma unit and smoke tests covering dynamic form generation, state signals, and routing guards.

---

# 📚 Professional Growth & Key Takeaways

Developing VaultCore provided comprehensive, hands-on experience in enterprise-grade software engineering:

* **Enterprise Architectural Patterns:** Deep practical mastery of Clean Architecture, Domain-Driven Design concepts, and Modular Monolith scalability.
* **Complex Data Modeling:** Designing runtime-extensible database schemas that maintain ACID guarantees while supporting dynamic metadata.
* **Security & Compliance:** Engineering materialized permission engines, electronic approval validations, and immutable audit logs.
* **Full-Stack Engineering:** Bridging a robust ASP.NET Core 9 backend with a modern Angular 20 reactive frontend utilizing Signals and dynamic component rendering.
* **Production Readiness:** Containerization with Docker Compose, automated database seeding, health checks, rate limiting, and structured logging.

---

## 📌 Project Summary

| Attribute | Specification |
| :--- | :--- |
| **Project Title** | VaultCore — Metadata-Driven Document Management & Workflow Vault |
| **Context** | Enterprise Software Engineering Internship — Tofaş IT |
| **Core Architecture** | Modular Monolith + Clean Architecture |
| **Frontend Stack** | Angular 20 (Standalone Components, TypeScript, SCSS) |
| **Backend Stack** | ASP.NET Core Web API (.NET 9, C# 13) |
| **Database** | PostgreSQL 16 (`unaccent`, `pg_trgm`, GIN Indexes) |
| **ORM** | Entity Framework Core 9 (Code-First & Migrations) |
| **Authentication & AuthZ** | JWT Bearer, Role-Based & Materialized Object ACL |
| **Search & Extraction** | PostgreSQL Full-Text Search + Tesseract OCR |
| **Interoperability** | WebDAV & Microsoft WOPI Host Protocol |
| **Automated Testing** | 315 Backend Tests (xUnit, Testcontainers) + 112 Frontend Tests (Karma) |
| **Containerization** | Docker & Docker Compose |

---

> **Note:** The interface screenshots and architectural diagrams in this document have been prepared strictly for technical portfolio and presentation purposes. No proprietary source code, internal credentials, or confidential corporate data is disclosed.
