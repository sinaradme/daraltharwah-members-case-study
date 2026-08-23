# Architecture

This document describes the public, high-level architecture of Dar Al Tharwah Members. It intentionally excludes source code, credentials, environment values, provider identifiers, customer data, internal hostnames, and private operational procedures.

## Architectural intent

The system extends an existing Webflow site instead of replacing it. Webflow remains the presentation and publishing layer, while trust-sensitive application behavior runs through Webflow Cloud.

The governing rule is simple:

> Webflow owns presentation. Webflow Cloud owns application behavior. Clerk owns identity. D1 owns application data. The integration outbox owns the handoff to external systems.

## System context

~~~mermaid
flowchart TD
    U["Visitor or member"]
    W["Webflow site<br/>Pages, CMS, design system, Code Components"]
    A["Webflow Cloud application<br/>Mounted Next.js APIs and server services"]
    I["Clerk<br/>Identity and sessions"]
    D["D1 + Drizzle<br/>Application data"]
    O["Integration outbox<br/>CRM and email events"]

    U --> W
    W --> A
    A --> I
    A --> D
    A --> O
~~~

Browser components call same-site application paths. Webflow Cloud route handlers authenticate, validate, and delegate to server-side domain services. Those services use Clerk when identity is required, read or write D1 for application state, and create durable integration events when downstream work is needed.

## 1. Webflow layer

### Responsibilities

Webflow owns:

- Public page structure and content
- CMS-authored content
- Design variables, classes, typography, spacing, and themes
- Responsive breakpoints and page composition
- Navigation and shared visual components
- English and Arabic presentation
- LTR and RTL behavior
- Placement and configuration of member-facing Code Components

### Interactive enhancement

Reusable Code Components and controlled browser runtimes add application-aware behavior inside Webflow-authored surfaces. Current product capabilities include:

- Member authentication UI
- Authenticated navigation status
- Member profile and dashboard states
- Course/download gates
- Newsletter form
- Consultation form

Components own interactive presentation and accessible state. They do not become an authorization boundary.

### Boundary

Webflow does not store credentials, authorize protected content, or make trusted identity decisions. Hiding a component or changing client state never grants access.

## 2. Webflow Cloud application layer

### Runtime model

A mounted Next.js application runs on Webflow Cloud. Its API routes provide the trust boundary between the public browser and protected product behavior.

The application layer owns:

- HTTP route handling
- Shared API response contracts
- Authentication checks
- Origin validation for mutations
- Input parsing, normalization, and validation
- Member bootstrap and profile rules
- Newsletter and consultation workflows
- Gated-content state and enrollment rules
- Authorized delivery proxying
- Server-side Turnstile verification
- Application rate limiting
- D1 data access
- Integration-event creation

Routes remain thin. Business and security rules are delegated to server-side services so transport, domain logic, and persistence have clear responsibilities.

### API characteristics

At a public level, the application exposes three categories of behavior:

| Category | Examples | Trust requirement |
| --- | --- | --- |
| Public/system | Service health, safe runtime configuration | No member data |
| Public mutation | Newsletter submission | Origin, validation, rate limit, honeypot, Turnstile |
| Authenticated member | Bootstrap, account, resources, access, enrollment, delivery | Authoritative server identity; Origin validation for mutations |

Member-specific responses remain private and non-cacheable by shared infrastructure. Short-lived browser-memory request coalescing is used only to reduce duplicate reads within the active experience and is invalidated by identity changes or mutations.

## 3. Authentication layer

Clerk owns:

- Registration and login
- Email verification
- Google sign-in
- Password handling
- Account recovery
- Sessions
- Authentication abuse controls

The server derives the current identity from Clerk and maps it to one application member record.

The application never persists:

- Passwords
- Verification codes
- Cookies
- JWTs
- Session tokens
- Full provider identity objects

Browser-supplied user identifiers, email addresses, roles, or enrollment claims are not authoritative.

## 4. Database layer

Webflow Cloud D1 stores application-owned state. Drizzle provides the typed schema and controlled migration history.

High-level data domains include:

| Domain | Purpose |
| --- | --- |
| Members | Application profile and identity mapping |
| Content items | Registry of gateable courses and downloads |
| Enrollments | Authoritative member-to-content relationship |
| Form submissions | Newsletter and consultation records |
| Rate limits | Hashed fixed-window counters |
| Integration outbox | Durable, deduplicated external-work events |

### Data rules

- Authentication secrets and session material do not belong in D1.
- CAPTCHA tokens are verified and discarded.
- Raw rate-limit subjects are not stored.
- Applied migrations are immutable; changes use ordered, additive migrations.
- Content delivery references remain server-side.
- Related writes are batched where they must be ordered and completed together.

### Query and write strategy

The system reduces database round trips through indexed joined reads and related-write batches. Examples documented by the production project include:

- One indexed read for account state
- One indexed joined read for resources, including the empty state
- One indexed joined read for content access
- One atomic rate-limit statement
- Batches for accepted form + outbox, member + initial events, and enrollment + outbox writes

These budgets are regression-tested at the contract level. They are not presented as a substitute for live regional latency monitoring.

## 5. CRM and integration layer

Member-facing operations do not call a CRM or email provider synchronously.

Instead, domain operations create versioned integration events with a unique deduplication key. Current event families cover member creation, welcome requests, form submissions, and enrollment creation.

~~~mermaid
sequenceDiagram
    participant M as Member
    participant A as Application
    participant D as D1
    participant O as Outbox consumer
    participant C as CRM or email

    M->>A: Submit accepted operation
    A->>D: Store domain record and outbox event
    D-->>A: Persisted
    A-->>M: Success
    O->>D: Claim pending event
    O->>C: Send idempotent operation
    O->>D: Mark completed or retry
~~~

This pattern prevents third-party availability from controlling registration, form submission, or enrollment persistence.

The durable outbox producer is implemented. A production-grade downstream consumer is a documented extension point and should add authenticated scheduling, bounded retries, failure review, metrics, reconciliation, and safe logging.

## Core data flows

### Registration and member bootstrap

1. Clerk completes registration, verification, and session creation.
2. The browser requests member initialization.
3. The application derives the authoritative Clerk identity.
4. The server creates or synchronizes the application member.
5. Deduplicated welcome/CRM-ready events are recorded with the member operation.

### Newsletter

1. A public Webflow component submits email, explicit consent, locale, approved attribution, honeypot value, and a Turnstile token.
2. The application validates the Origin and input, applies rate limiting, and verifies Turnstile server-side.
3. The form submission and its integration event are persisted.
4. The browser receives a safe success response; the CAPTCHA token is not stored.

Newsletter submission does not create a member account.

### Consultation

1. The component confirms an authenticated experience and submits the consultation request.
2. The server independently derives the member identity and requires an initialized member.
3. Origin, input, rate limit, honeypot, and Turnstile checks are applied.
4. The form and outbox event are persisted.

The browser cannot replace canonical member identity with submitted contact values.

### Gated resource delivery

1. The component asks the application for an access state using a stable content key.
2. The server resolves the member, active content, and enrollment.
3. The browser never receives the underlying delivery reference.
4. A delivery request is authorized again.
5. The application accepts only same-application or approved HTTPS destinations, revalidates redirects, and streams the resource with private response controls.
6. The enrollment/resource relationship is recorded when required.

## Security boundaries

~~~mermaid
flowchart LR
    B["Untrusted browser"]
    T["Trusted application boundary"]
    P["Managed providers and persistence"]

    B -->|"Validated requests"| T
    T -->|"Server-side identity, data, verification"| P
~~~

The main controls are:

- Server-derived identity
- Exact trusted-Origin checks for state changes
- Cross-site request rejection
- Server-side type, length, value, path, and consent validation
- Server-verified Turnstile
- Hashed rate-limit subjects
- Safe return-path and delivery-destination allowlists
- Revalidation of delivery redirects
- Private/no-store handling for member data
- No secrets in browser code, source documentation, or logs
- No client-side authorization decisions

## Performance and scalability model

The architecture favors fewer calls over shared caching of private member data:

- Simultaneous identical member reads are coalesced in the browser.
- Member bootstrap is single-flight.
- Safe public runtime configuration is short-lived and tab-scoped.
- Member reads remain private.
- Joined queries resolve content and resources without query chains.
- Related writes use D1 batches.
- External calls use bounded wait times.
- Integration delivery is asynchronous and idempotent.

This design supports the product context of more than 40,000 users. The public case study does not publish or imply a specific concurrent-load benchmark.

## Deployment separation

The production system maintains separate staging and production application mounts and follows a feature-to-preview-to-production release flow.

At a high level:

1. A focused change is developed outside the protected branches.
2. Automated application and Code Component checks run.
3. The exact revision is deployed and tested in staging.
4. English/Arabic, LTR/RTL, responsive, authentication, accessibility, loading, error, and success states are checked as relevant.
5. The accepted revision is promoted to production.
6. Rollback uses a new reviewed change; migration history is not rewritten.

Private branch names, runtime identifiers, environment values, and operational credentials are intentionally omitted from this portfolio repository.

## Deliberate limitations

The current MVP does not claim to provide:

- Payments or subscriptions
- Certificates, quizzes, or an advanced LMS
- Complex organizational roles
- A full admin portal
- Private-object storage delivery
- A completed downstream CRM/email consumer

These are extension decisions rather than hidden launch gaps. Each would require its own product policy, security, privacy, data, operations, and rollback design.
