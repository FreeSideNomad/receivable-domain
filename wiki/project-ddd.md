# Payment Receivables Platform - Domain Model

**Version**: 0.1 (Draft)  
**Last Updated**: 2025-10-09  
**Status**: Initial Discovery - Feedback Loop 1

---

## System Overview

**System ID**: `sys_payment_receivables_platform`  
**System Name**: Payment Receivables Platform  
**Description**: Two-sided platform connecting bank customers (receivable customers) with their business customers (payors) to streamline invoice approval and ACH payment collection.

---

## Domains

### Domain 1: Receivables Management (CORE)

**Domain ID**: `dom_receivables_management`  
**Type**: Core Domain  
**Strategic Importance**: Critical  

**Description**:
The heart of our competitive advantage. Manages the complete lifecycle of receivables from invoice creation through approval workflows to payment origination. Encompasses payor relationships and user management for the approval portal.

**Bounded Contexts**:
- Invoicing
- Payment Origination
- Payor Management
- Approval Workflow

**Investment Strategy**:
Best team, rigorous DDD practices, continuous refinement, heavy testing. This is where we differentiate from competitors.

**Key Differentiators**:
- Two-sided platform approach (receivable + payor perspectives)
- Flexible approval workflows with amount-tier rules
- Seamless integration with bank's existing online banking
- Invoice generation with customizable templates

**Notes**:
This domain contains our core business logic. We must maintain high code quality, comprehensive domain events, and strong consistency within aggregates.

---

### Domain 2: Payment Processing (SUPPORTING)

**Domain ID**: `dom_payment_processing`  
**Type**: Supporting Domain  
**Strategic Importance**: Important  

**Description**:
Integration with existing bank payment processor for ACH file generation and processing. Handles payment submission, status tracking, and returns management. The actual payment processing is done by an external system.

**Bounded Contexts**:
- Payment Gateway Integration

**Investment Strategy**:
Adequate quality, standard integration patterns, minimal custom logic. Focus on reliable integration with existing payment processor.

**Supplier**:
Existing bank payment processing system (acts as upstream supplier)

**Notes**:
This is commodity functionality. We originate payments, but delegate actual processing to proven payment infrastructure. Use Anti-Corruption Layer or Customer/Supplier pattern.

---

### Domain 3: Identity & Access Management (GENERIC)

**Domain ID**: `dom_identity_access`  
**Type**: Generic Domain  
**Strategic Importance**: Standard  

**Description**:
User authentication, authorization, and session management. Leverages Okta for authentication with MFA. Manages user profiles, roles, and permissions.

**Bounded Contexts**:
- Identity & Access

**Investment Strategy**:
Use third-party service (Okta), minimal custom implementation, standard OAuth/OIDC patterns.

**Notes**:
Okta handles authentication. We manage authorization rules and permission mappings. Consider complete outsourcing if Okta can handle our authorization needs.

---

### Domain 4: Banking Integration (SUPPORTING)

**Domain ID**: `dom_banking_integration`  
**Type**: Supporting Domain  
**Strategic Importance**: Important  

**Description**:
Integration with bank's existing online banking platform. Provides seamless experience for receivable customers by extending existing banking interface.

**Bounded Contexts**:
- Online Banking Integration

**Investment Strategy**:
Adequate quality, focus on seamless user experience, standard integration patterns.

**Supplier**:
Bank's core online banking system (upstream)

**Notes**:
Receivable customer UI is an extension of existing banking. We must conform to their model or use ACL to protect our domain model.

---

## Bounded Contexts (CORE DOMAIN)

### BC1: Invoicing Context

**Context ID**: `bc_invoicing`  
**Domain**: Receivables Management (Core)  
**Team Ownership**: Core Receivables Team  

**Description**:
Manages invoice lifecycle from creation/upload through sending to payors. Handles invoice generation using customizable PDF templates, invoice metadata, line items, and status tracking.

**Responsibilities**:
- Create invoices from templates or upload files
- Manage invoice templates (customization, branding)
- Track invoice status (draft, sent, pending approval, approved, submitted)
- Send invoices to payors
- Manage invoice-payor relationships

**Ubiquitous Language**:

| Term | Definition | Examples |
|------|------------|----------|
| **Invoice** | A document requesting payment from a payor for goods/services | INV-2024-001 for $5,000 |
| **Invoice Template** | Customizable PDF template for invoice generation | Company branded template with logo |
| **Invoice Line Item** | Individual item or service on an invoice | "Consulting Services - 40 hours @ $150/hr" |
| **Invoice Status** | Current state in invoice lifecycle | Draft, Sent, Pending Approval, Approved, Rejected, Submitted for Payment, Paid |
| **Due Date** | Date by which payment is expected | Net 30 days from invoice date |
| **Receivable Customer** | Bank customer who issues invoices | ABC Corp (supplier) |

**Aggregates**:
- Invoice Aggregate (root: Invoice entity)
- Invoice Template Aggregate (root: InvoiceTemplate entity)

**Key Domain Events**:
- InvoiceCreated
- InvoiceSentToPayor
- InvoiceApproved
- InvoiceRejected
- InvoiceSubmittedForPayment
- InvoicePaid

**Integration Points**:
- **→ Approval Workflow Context**: Send invoice for approval
- **→ Payment Origination Context**: Approved invoices ready for payment
- **→ Payor Management Context**: Invoice sent to specific payor

---

### BC2: Approval Workflow Context

**Context ID**: `bc_approval_workflow`  
**Domain**: Receivables Management (Core)  
**Team Ownership**: Core Receivables Team  

**Description**:
Manages configurable approval workflows based on amount tiers. Handles approval rules, approval chains (single/dual approver), approval actions, and notification triggers.

**Responsibilities**:
- Configure approval rules per payor (amount tiers)
- Route invoices through approval chains
- Track approval status and history
- Enforce approval policies (e.g., second approver must differ from first)
- Trigger notifications when approvals are needed

**Ubiquitous Language**:

| Term | Definition | Examples |
|------|------------|----------|
| **Approval Rule** | Configuration defining who approves based on amount | "Invoices > $10K require dual approval" |
| **Amount Tier** | Threshold ranges for approval requirements | Tier 1: $0-$5K, Tier 2: $5K-$25K, Tier 3: >$25K |
| **Approver** | Payor user authorized to approve invoices | John Smith - Primary Approver for Tier 2 |
| **Approval Chain** | Sequence of required approvals | Primary → Secondary for high-value invoices |
| **Approval Action** | Approve or Reject decision | Approved by John Smith on 2024-10-09 |
| **Approval Status** | Current state of approval process | Pending Primary Approval, Awaiting Second Approval, Approved, Rejected |

**Aggregates**:
- Approval Rule Aggregate (root: ApprovalRule entity)
- Invoice Approval Aggregate (root: InvoiceApproval entity)

**Key Domain Events**:
- ApprovalRuleConfigured
- ApprovalRequired
- InvoiceApprovedByPrimaryApprover
- InvoiceApprovedBySecondaryApprover
- InvoiceRejectedByApprover
- ApprovalChainCompleted

**Integration Points**:
- **← Invoicing Context**: Receives invoices to approve
- **→ Invoicing Context**: Updates invoice status after approval
- **→ Payment Origination Context**: Notify when approval complete
- **← Payor Management Context**: Uses approval rules configured per payor

---

### BC3: Payment Origination Context

**Context ID**: `bc_payment_origination`  
**Domain**: Receivables Management (Core)  
**Team Ownership**: Core Receivables Team  

**Description**:
Originates payments from approved invoices. Manages payment creation, batching of invoices into payment files, payment submission preparation, and coordination with payment processing system.

**Responsibilities**:
- Create payments from approved invoices
- Select and batch invoices for payment
- Generate payment instructions (ACH file metadata)
- Track payment submission status
- Handle payment state transitions (originated → submitted → settled → failed)
- Coordinate payment resubmission for returns

**Ubiquitous Language**:

| Term | Definition | Examples |
|------|------------|----------|
| **Payment** | Instruction to debit payor account for approved invoice(s) | Payment-2024-1001 for Invoice INV-2024-001 |
| **Payment Batch** | Group of invoices selected for payment at same time | Batch of 15 invoices due this week |
| **Payment Instruction** | Metadata for ACH file generation | Payor account, amount, effective date |
| **Payment Status** | Current state of payment | Originated, Submitted, Processing, Settled, Returned, Failed |
| **Payment Originator** | Receivable customer who originates payment | ABC Corp requesting payment from XYZ Inc |
| **Effective Date** | Date payment should be debited | 2024-10-15 |

**Aggregates**:
- Payment Aggregate (root: Payment entity)
- Payment Batch Aggregate (root: PaymentBatch entity)

**Key Domain Events**:
- PaymentOriginated
- PaymentBatchCreated
- PaymentSubmittedForProcessing
- PaymentSettled
- PaymentReturned
- PaymentFailed
- PaymentResubmitted

**Integration Points**:
- **← Approval Workflow Context**: Receives approved invoices
- **← Invoicing Context**: References invoice details
- **→ Payment Gateway Integration Context**: Submits payment for processing
- **← Payment Gateway Integration Context**: Receives payment status updates

---

### BC4: Payor Management Context

**Context ID**: `bc_payor_management`  
**Domain**: Receivables Management (Core)  
**Team Ownership**: Core Receivables Team  

**Description**:
Manages payor organizations, their profiles, bank account information, and relationships with receivable customers. Handles payor onboarding, account management, and configuration.

**Responsibilities**:
- Onboard new payors
- Manage payor profile information
- Manage payor bank accounts (for debit authorization)
- Track payor-receivable customer relationships
- Store payor preferences and settings
- Manage payor organization metadata

**Ubiquitous Language**:

| Term | Definition | Examples |
|------|------------|----------|
| **Payor** | Organization that purchases goods/services and approves invoices | XYZ Inc (buyer) |
| **Payor Profile** | Organization details and configuration | Company name, address, tax ID, settings |
| **Payor Admin** | User who manages payor organization and settings | Jane Doe - Admin for XYZ Inc |
| **Bank Account** | Canadian bank account used for payment debits | Account at TD Bank - *****1234 |
| **Account Verification** | Process to validate bank account ownership | Micro-deposit verification |
| **Payor Relationship** | Link between receivable customer and payor | ABC Corp supplies to XYZ Inc |

**Aggregates**:
- Payor Aggregate (root: Payor entity)
  - Contains: PayorProfile value object, BankAccount entities
- PayorRelationship Aggregate (root: PayorRelationship entity)

**Key Domain Events**:
- PayorOnboarded
- PayorProfileUpdated
- BankAccountAdded
- BankAccountVerified
- BankAccountRemoved
- PayorRelationshipEstablished
- PayorAdminDesignated

**Integration Points**:
- **→ Identity & Access Context**: Create payor admin users
- **← Invoicing Context**: Invoices sent to payors
- **→ Approval Workflow Context**: Provides approval rules per payor
- **→ Payment Origination Context**: Provides bank account for payment

---

### BC5: User Management Context

**Context ID**: `bc_user_management`  
**Domain**: Receivables Management (Core)  
**Team Ownership**: Core Receivables Team  

**Description**:
Manages payor portal users, their roles, permissions, and access controls. Handles user provisioning, role assignment, permission configuration, and user lifecycle within payor organizations.

**Responsibilities**:
- Provision payor users (admins, approvers, viewers)
- Assign roles and permissions
- Manage user profiles
- Track user activity
- Configure permission levels
- Handle user deactivation/reactivation

**Ubiquitous Language**:

| Term | Definition | Examples |
|------|------------|----------|
| **Payor User** | Individual with access to payor portal | John Smith at XYZ Inc |
| **Role** | Named set of permissions | Admin, Primary Approver, Secondary Approver, Viewer |
| **Permission** | Specific access right | Approve Invoices, Manage Users, View Reports |
| **User Profile** | Individual user information | Name, email, phone, preferences |
| **User Status** | Current state of user account | Active, Inactive, Pending Registration |
| **Permission Level** | Scope of user's access | Organization-wide, Department-specific |

**Aggregates**:
- PayorUser Aggregate (root: PayorUser entity)
- Role Aggregate (root: Role entity)

**Key Domain Events**:
- PayorUserProvisioned
- PayorUserActivated
- RoleAssigned
- RoleRevoked
- PermissionGranted
- PermissionRevoked
- UserProfileUpdated
- UserDeactivated

**Integration Points**:
- **→ Identity & Access Context**: Coordinate with Okta for authentication
- **← Payor Management Context**: Users belong to payors
- **→ Approval Workflow Context**: Users act as approvers

---

## Bounded Contexts (SUPPORTING DOMAINS)

### BC6: Payment Gateway Integration Context

**Context ID**: `bc_payment_gateway`  
**Domain**: Payment Processing (Supporting)  
**Team Ownership**: Integration Team  

**Description**:
Anti-corruption layer for existing bank payment processor. Translates our payment domain model to external system's ACH processing model. Handles file generation, submission, status polling, and return processing.

**Responsibilities**:
- Generate ACH files in required format
- Submit files to payment processor
- Poll for payment status updates
- Process ACH return notifications
- Translate external codes to domain model
- Handle retry logic and error scenarios

**Ubiquitous Language**:

| Term | Definition | Examples |
|------|------------|----------|
| **ACH File** | NACHA-formatted file for batch payment processing | ACH file with 50 payment records |
| **Payment Processor** | External system that processes payments | Bank's payment processing platform |
| **Return Code** | Standard ACH return reason code | R01 - Insufficient Funds, R02 - Account Closed |
| **Settlement** | Confirmation that payment was processed | Settlement report from processor |
| **Processing Status** | Status from external system | Accepted, Processing, Settled, Returned |

**Integration Pattern**: Anti-Corruption Layer + Customer/Supplier

**Aggregates**:
- ACH File Aggregate (root: ACHFile entity)

**Key Domain Events**:
- ACHFileGenerated
- ACHFileSubmitted
- PaymentProcessingStatusReceived
- ACHReturnReceived

**Integration Points**:
- **← Payment Origination Context**: Receives payment instructions
- **→ Payment Origination Context**: Sends status updates
- **→ External Payment Processor**: Submits ACH files

**Translation Map**:
```
Our Model → External System
Payment → ACH Payment Record
PaymentStatus.Submitted → ProcessingStatus.InProcess
PaymentStatus.Settled → ProcessingStatus.Completed
ACH Return R01 → PaymentStatus.ReturnedInsufficientFunds
```

---

### BC7: Online Banking Integration Context

**Context ID**: `bc_online_banking`  
**Domain**: Banking Integration (Supporting)  
**Team Ownership**: Integration Team  

**Description**:
Integration layer with bank's existing online banking platform. Provides seamless UI extension for receivable customers. Handles SSO, session management, and customer context.

**Responsibilities**:
- Integrate with online banking UI framework
- Handle SSO from banking platform
- Synchronize customer context
- Provide consistent UX with banking platform
- Handle navigation between banking and receivables

**Ubiquitous Language**:

| Term | Definition | Examples |
|------|------------|----------|
| **Banking Customer** | Bank customer using online banking | Receivable customer logged into banking |
| **Customer Context** | Session information from banking system | Customer ID, account info, permissions |
| **UI Extension** | Our interface embedded in banking platform | Receivables tab in online banking |
| **SSO Token** | Authentication token from banking | SAML assertion or JWT |

**Integration Pattern**: Conformist or Anti-Corruption Layer

**Integration Points**:
- **← Bank Online Banking System**: Receives customer context
- **→ Invoicing Context**: Maps banking customer to receivable customer
- **→ Identity & Access Context**: Coordinates authentication

---

### BC8: Identity & Access Context

**Context ID**: `bc_identity_access`  
**Domain**: Identity & Access Management (Generic)  
**Team Ownership**: Platform Team  

**Description**:
Manages authentication via Okta and authorization for payor portal users. Handles MFA, session management, and token validation.

**Responsibilities**:
- Authenticate users via Okta
- Enforce MFA requirements
- Validate access tokens
- Manage sessions
- Coordinate with User Management for authorization

**Ubiquitous Language**:

| Term | Definition | Examples |
|------|------------|----------|
| **Authentication** | Verifying user identity | Okta login with MFA |
| **Authorization** | Determining user permissions | Can this user approve invoices? |
| **MFA** | Multi-factor authentication | SMS code, authenticator app |
| **Access Token** | JWT token with user claims | OAuth2 access token |

**Integration Pattern**: Open Host Service (Okta provides)

**Integration Points**:
- **→ User Management Context**: Sync user profiles
- **→ Okta**: Delegate authentication
- **→ Approval Workflow Context**: Validate approver identity

---

## Context Map

### Relationships

#### Within Core Domain (Receivables Management)

**Invoicing → Approval Workflow**
- **Type**: Partnership
- **Pattern**: Domain events + shared understanding
- **Integration**: InvoiceSentToPayor event triggers approval workflow

**Approval Workflow → Invoicing**
- **Type**: Partnership
- **Pattern**: Domain events
- **Integration**: InvoiceApproved event updates invoice status

**Approval Workflow → Payment Origination**
- **Type**: Partnership
- **Pattern**: Domain events
- **Integration**: ApprovalChainCompleted event enables payment origination

**Invoicing → Payor Management**
- **Type**: Customer/Supplier
- **Pattern**: Reference by ID
- **Integration**: Invoice references Payor by ID, queries for details

**Payor Management → User Management**
- **Type**: Partnership
- **Pattern**: Aggregate references
- **Integration**: Payor contains users, shared lifecycle

**User Management → Approval Workflow**
- **Type**: Customer/Supplier
- **Pattern**: Reference by ID
- **Integration**: Approver references User, validates permissions

---

#### Core to Supporting Domains

**Payment Origination → Payment Gateway (ACL)**
- **Type**: Customer/Supplier
- **Pattern**: Anti-Corruption Layer
- **Integration**: Payment Origination sends commands, ACL translates to ACH format
- **Reason for ACL**: Protect our domain model from external payment processor's model

**Invoicing → Online Banking**
- **Type**: Conformist
- **Pattern**: Adapt to banking platform
- **Integration**: UI embedded in banking, uses their session/styling
- **Reason for Conformist**: Banking platform is upstream, we extend their UX

**User Management → Identity & Access**
- **Type**: Customer/Supplier
- **Pattern**: Coordinate authentication with authorization
- **Integration**: User Management stores roles/permissions, Identity validates identity

---

## Key Design Decisions

### Decision: Small Aggregates

**Rationale**: Following Vaughn Vernon's guidance, we prefer small aggregates to:
- Improve performance (smaller transactions)
- Reduce contention (fewer locks)
- Enable better scalability
- Simplify consistency boundaries

**Examples**:
- Invoice is separate aggregate from Payment (reference by ID)
- Payor is separate from its approvals (reference by ID)
- User is separate from their approval actions (reference by ID)

### Decision: Eventual Consistency Between Aggregates

**Rationale**: Use domain events for cross-aggregate updates:
- Invoicing publishes InvoiceApproved
- Payment Origination listens and creates Payment
- No distributed transaction needed

### Decision: Anti-Corruption Layer for Payment Processor

**Rationale**: 
- External ACH processor has different model (batch-oriented, NACHA format)
- We want clean domain model (payment-oriented)
- ACL translates between models
- Protects us from changes in processor

### Decision: Partnership Pattern Within Core Domain

**Rationale**:
- Same team owns all Core contexts
- Shared understanding of ubiquitous language
- Can evolve together
- Use domain events for decoupling

---

## Open Questions

### Q1: Invoice Template Management
**Question**: Is Invoice Template management complex enough for separate bounded context?  
**Current**: Part of Invoicing Context  
**Consider**: Separate Template Management Context if templates become sophisticated (versioning, approvals, marketplace)

### Q2: Notification System
**Question**: Should notifications be a separate bounded context?  
**Current**: Implied in Approval Workflow (triggers notifications)  
**Consider**: Generic Notifications Context if we need sophisticated rules (preferences, channels, scheduling)

### Q3: Reporting & Analytics
**Question**: Is there a Reporting bounded context?  
**Current**: Not defined  
**Consider**: Separate read model context using CQRS if reporting needs differ from operational model

### Q4: Audit & Compliance
**Question**: Should audit trail be separate context?  
**Current**: Implied in domain events  
**Consider**: Dedicated Audit Context if compliance requires specific audit model

### Q5: Multi-Currency Future
**Question**: How will multi-currency affect contexts?  
**Current**: CAD only  
**Consider**: May need Currency Management value objects, forex rate handling

---

## Next Steps

### Validation Needed
1. Review bounded context boundaries with domain experts
2. Validate ubiquitous language terms
3. Confirm aggregate boundaries (especially Invoice, Payment, Payor)
4. Clarify Banking Integration pattern (Conformist vs ACL)

### Detailed Modeling
1. Define entity attributes and value objects for each aggregate
2. Document business rules and invariants
3. Map complete domain event flows
4. Define repository interfaces
5. Specify application services (use cases)

### Technical Validation
1. Assess existing payment processor capabilities
2. Review online banking integration constraints
3. Validate Okta configuration options
4. Consider data migration from existing systems

---

**Version History**:
- v0.1 (2025-10-09): Initial domain model based on vision.md and domain classification decision
