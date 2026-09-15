# Mawthooq — Offline-First Pharmacy ERP & POS

## Overview

**Mawthooq** is an integrated **ERP/POS** system specifically designed to manage pharmacy operations, built with an **Offline-First** philosophy to ensure efficient operation even during internet outages.

The primary goal of the system is not merely data recording, but to simulate the actual operational and financial reality of the pharmacy with the highest possible level of accuracy, preventing errors and inconsistencies before they reach the database while maintaining inventory precision and financial integrity across every transaction.

---

## Architectural Philosophy

The project is built entirely in accordance with the following principles:

- **Clean Architecture** — Strict separation between layers (Domain, Application, Infrastructure, API) with a clear dependency direction towards the Domain.
- **Domain-Driven Design (DDD)** — Business logic is entirely encapsulated within the Domain Layer, and Aggregates serve as the sole consistency boundaries allowed to mutate their own state.
- **CQRS** — Complete separation between the write path (Commands) and the read path (Queries), each having an independent execution model optimized for its purpose.
- **Event-Based Persistence** — Aggregates do not generate SQL directly; instead, they record Domain Events describing actual changes that occurred, while the Infrastructure translates them into explicit SQL operations (without Entity Framework Change Tracking or automatic object comparison).
- **Optimistic Concurrency** via RowVersion to protect business data from conflicting updates.
- **Explicit over Implicit** — No implicit or magic behavior; every persistence operation and architectural decision is explicit and traceable.

---

## Layered Architecture (Layers)

```
API
 ↓
Application   (Use Case Orchestration — CQRS, Transactions, Validation)
 ↓
Domain        (Business Rules, Aggregates, Invariants, Domain Events)
 ↑
Infrastructure (SQL Execution, Repositories, Unit of Work, External Integrations)
```

- **Domain** knows nothing about databases, Infrastructure, or HTTP.
- **Application** orchestrates Use Cases only, without any business rules.
- **Infrastructure** implements technical details (explicit SQL, Unit of Work, Repositories) and relies on contracts defined by the Application.
- **API** deals solely with HTTP, Validation, and Authentication/Authorization.

---

## Business Capability Model

The system is divided into clearly bounded Bounded Contexts, each owning a single business responsibility and a single source of truth:

| Module | Responsibility |
|---|---|
| **Product** | The commercial definition of what is sold (brand name, packaging, sales units, barcode) |
| **Medicine** | The medical identity of the pharmaceutical item (composition, dosage form, therapeutic classification) |
| **Inventory** | Physical inventory within the branch (batches, quantities, FEFO allocation, pricing) |
| **Purchase** | Purchasing operations (Purchase Orders and Goods Receipts) |
| **Sales** | Sales operations and returns (Sales Orders, Sales Returns, Payments) |
| **Customer / Supplier** | Commercial identity of external transacting parties |
| **Branch** | The organizational unit within which operations are executed |
| **Active Ingredient / Manufacturer / Therapeutic Class** | Shared reference catalogs reused across the system |
| **Financial** | A layer for recording the financial impact of completed operational processes (not a full accounting system) |

Each module protects its own rules, and no module's data is directly modified by another; inter-module communication occurs strictly via **Domain Events / Integration Events**.

---

## Key Architectural Decisions

- **Separation of Commercial Identity and Medical Identity**: `Product` answers "What are we selling?", while `Medicine` answers "What is this medicine?" — two distinct entities, each with its own Aggregate.
- **Inventory as the Single Source of Truth for Physical Stock**: No other module directly modifies quantities; allocation follows the **FEFO** (First Expired, First Out) strategy.
- **Preserving Historical Truth**: Completed documents (sales invoices, medical records at the time of the event) are never overwritten when reference data changes later; reliance is placed on Snapshots and Identity Preservation.
- **Financial as a Recording Layer, Not a Full Accounting System**: Receives completed operational events and logs their financial impact only, laying the groundwork for future integration with a full accounting system.
- **Idempotency** to protect critical operations (completing a sale, receiving goods) from duplication caused by retries or network interruptions.

---

## Tech Stack

- **.NET / C#**
- **ASP.NET Core**
- **ADO.NET** (Explicit SQL without ORM Change Tracking)
- **SQL Server**
- **xUnit** for testing
- Development Environment: **VS Code** on **Linux**

---

## Knowledge Governance (AI-Governed Knowledge Base)

The project maintains a fully documented Knowledge Base governed by high-priority **Constitutions** (Project Identity, AI Governance, Knowledge Model, Architecture Principles, Documentation Standards, Decision Governance, Terminology), ensuring:

- A single source of truth for every architectural decision.
- A **No Assumptions** policy — any unverified information is explicitly stated as unknown, assuming nothing.
- Full decision traceability via **ADRs** (Architecture Decision Records).
- Complete consistency in terminology (**Ubiquitous Language**) across all documentation and code layers.

---

## Project Status

The project is under active, continuous development, built incrementally module by module while strictly adhering to Aggregate boundaries and architectural dependency rules in every new addition.
