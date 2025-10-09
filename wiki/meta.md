# Domain Model Discovery Process

**Purpose**: Document the iterative process of creating the Payment Receivables Platform domain model using Domain-Driven Design principles.

**Goal**: Create `project-ddd.md` through feedback loops, then convert to `project-ddd.yaml` once validated.

---

## Process Overview

### Phase 1: Research & Analysis
**Status**: Completed  
**Started**: 2025-10-09  
**Completed**: 2025-10-09

**Activities**:
1. ✅ Review vision.md - understand business domain and MVP scope
2. ✅ Study DDD reference materials:
   - `ddd-schema-definition.yaml` - YAML schema structure
   - `ddd-02-strategic-patterns.md` - Strategic design patterns
   - `ddd-06-ontological-taxonomy.md` - Pattern hierarchy
   - `ddd-schema-example.yaml` - Example implementation (Job Seeker app)
3. ✅ Formulate discovery questions
4. ✅ Iterate with user to clarify domain boundaries and concepts

**Key Outcomes**:
- Created discovery-questions.md with 40+ questions
- Clarified domain classification (Receivables = Core, Payment Processing = Supporting)
- Identified 4 domains and 8 bounded contexts

### Phase 2: Domain Identification
**Status**: Completed  
**Started**: 2025-10-09  
**Completed**: 2025-10-09

**Objectives**:
- Identify core, supporting, and generic domains
- Define strategic importance and investment strategy
- Map business capabilities to domains

### Phase 3: Bounded Context Definition
**Status**: Completed  
**Started**: 2025-10-09  
**Completed**: 2025-10-09

**Objectives**:
- Define bounded context boundaries
- Establish ubiquitous language for each context
- Identify team ownership
- Map context relationships

**Key Outcomes**:
- Defined 8 bounded contexts across 4 domains
- Created ubiquitous language glossaries for core contexts
- Mapped integration patterns (Partnership, Customer/Supplier, ACL, Conformist)
- Identified team ownership (Core Receivables, Integration, Platform teams)

### Phase 4: Tactical Modeling
**Status**: Initial Draft Completed  
**Started**: 2025-10-09  
**Completed**: 2025-10-09 (v0.1)

**Objectives**:
- Identify aggregates and aggregate roots
- Define entities and value objects
- Model domain services and repositories
- Define domain events

**Key Outcomes**:
- Identified 10+ aggregates with aggregate roots
- Documented domain events for each bounded context
- Defined integration points between contexts
- Documented key design decisions (small aggregates, eventual consistency)

**Status Note**: This is v0.1 - needs validation and refinement through feedback loops

### Phase 5: Documentation & Validation
**Status**: In Progress  
**Started**: 2025-10-09

**Objectives**:
- Create comprehensive project-ddd.md in markdown ✅
- Validate with stakeholders through feedback loops ⏳
- Refine based on feedback ⏳
- Convert to project-ddd.yaml format ⏳

**Deliverables**:
- ✅ project-ddd.md v0.1 created
- ⏳ Awaiting feedback loop 1 validation
- ⏳ project-ddd.yaml conversion pending

---

## Key Findings from Vision.md

### Business Domain: Payment Receivables Platform

**Core Value Proposition**:
- Two-sided platform connecting receivable customers (suppliers) with payors (buyers)
- Streamlines invoice approval and ACH payment collection
- Extends existing online banking for receivable customers
- New standalone portal for payors

### MVP Scope:
- ACH pre-approved debit (PAD) payments
- PDF invoice generation with customizable templates
- Invoice file upload capability
- Okta-based authentication with MFA
- User, permission, and approval rule management
- Amount-tier based approval workflows (single/dual approval)
- ACH returns management and resubmission

### Key User Roles:
1. **Receivable Customer** - Bank customer who supplies goods/services
2. **Payor** - Company that purchases from receivable customer
3. **Payor Admin** - Manages payor portal, users, and permissions
4. **Payor Approver** - Reviews and approves invoices

### Core Business Processes:
1. **Onboarding**: Receivable customer sets up payors and designates admin users
2. **User Management**: Payor admins configure users, permissions, approval rules
3. **Invoice Generation**: Create/upload invoices with PDF templates
4. **Invoice Approval**: Multi-step approval workflow based on amount tiers
5. **Payment Processing**: Generate ACH files for approved invoices
6. **Returns Management**: Handle ACH returns and resubmissions
7. **Account Management**: Payors manage Canadian bank accounts for debits

---

## DDD Reference Framework

### Strategic Patterns to Apply:
- **Domain Classification**: Core, Supporting, Generic
- **Bounded Contexts**: Define linguistic and organizational boundaries
- **Context Mapping**: Identify relationships between contexts
  - Partnership, Customer/Supplier, Anti-Corruption Layer, etc.

### Tactical Patterns to Consider:
- **Aggregates**: Consistency boundaries (prefer small)
- **Entities**: Objects with identity (Customer, Invoice, Payment)
- **Value Objects**: Immutable concepts (Money, Address, Email)
- **Domain Events**: Facts that happened (InvoiceApproved, PaymentSubmitted)
- **Repositories**: Persistence abstractions
- **Domain Services**: Operations spanning multiple aggregates
- **Application Services**: Use case orchestration

---

## Discovery Question Framework

### Strategic Questions (Domain & Context):
1. Domain boundaries and classification
2. Bounded context identification
3. Integration patterns between contexts
4. Team ownership and responsibilities

### Tactical Questions (Aggregates & Entities):
1. Core domain objects and their identities
2. Consistency boundaries (what must be updated together?)
3. Invariants and business rules
4. Value objects vs. entities
5. Domain events and cross-aggregate communication

### Process Questions (Workflow & Lifecycle):
1. Entity lifecycle states
2. State transitions and triggers
3. Validation rules
4. Integration points with external systems

---

## Feedback Loop Tracker

### Loop 1: Initial Discovery
**Status**: In Progress  
**Started**: 2025-10-09

**Questions Posed**: [To be documented]  
**Answers Received**: [To be documented]  
**Insights Gained**: [To be documented]  
**Decisions Made**: [To be documented]

### Loop 2: Refinement
**Status**: Pending

### Loop 3: Validation
**Status**: Pending

---

## Decision Log

### Decision 1: Domain Classification (2025-10-09)
**Context**: Determining which domains are Core vs Supporting, and what belongs in Receivables domain

**Options Considered**:
- Option A: Payment Processing as Core domain (differentiating capability)
- Option B: Payment Processing as Supporting domain (existing solution integration)
- Option C: Split Receivables into multiple separate domains

**Decision**: 
- **Payment Processing** = SUPPORTING domain (existing solution acts as supplier)
- **Receivables Management** = CORE domain (our competitive advantage)
- Receivables domain encompasses:
  - Invoicing
  - Payment Origination
  - Payor Profile Management
  - User Management

**Rationale**: 
- Payment processing (ACH) is commodity - we integrate with existing bank payment processor
- Our value is in the receivables workflow, not payment rails
- Keeping related capabilities (invoicing, payor, users) in one domain maintains cohesion
- Payment origination is different from payment processing - we originate, external system processes

**Impact**:
- Receivables becomes larger core domain with multiple bounded contexts
- Payment Processing bounded context uses Customer/Supplier or Conformist relationship pattern
- Investment strategy: Heavy in Receivables, minimal in Payment Processing integration

---

## Open Questions

1. How does the platform integrate with existing online banking systems?
2. What is the bank's core banking system boundary?
3. Is invoice generation truly part of MVP or separate concern?
4. How do we handle multi-currency in future (bounded context consideration)?
5. What are the compliance/audit requirements (separate context)?

---

## Resources

### Project Documents
- `wiki/vision.md` - Product vision and MVP scope
- `wiki/vision-reference.md` - Vision development guide

### DDD References
- `ddd/ddd-schema-definition.yaml` - YAML schema for DDD models
- `ddd/ddd-02-strategic-patterns.md` - Strategic design patterns
- `ddd/ddd-03-tactical-patterns.md` - Tactical design patterns
- `ddd/ddd-06-ontological-taxonomy.md` - Pattern hierarchy and decision trees
- `ddd/ddd-schema-example.yaml` - Example implementation

### Output
- `wiki/project-ddd.md` - Domain model in Markdown (work in progress)
- `wiki/project-ddd.yaml` - Final YAML model (future)

---

**Last Updated**: 2025-10-09  
**Next Action**: Formulate and present discovery questions
