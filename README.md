# Dar Al Tharwah Members — Webflow Cloud Membership Case Study

I built a membership and gated-content layer for [Dar Al Tharwah](https://daraltharwa.com/) without rebuilding its Webflow website or adding a separate membership SaaS.

The production code is private. This repository documents the problem, the architecture I chose, the trade-offs, and the parts of the result that can be verified publicly.

[Read the case study](CASE_STUDY.md) · [View the architecture](docs/ARCHITECTURE.md)

## The problem

Dar Al Tharwah already had a bilingual Webflow site, CMS, design system, and publishing workflow. It needed registration, authentication, member profiles, protected downloads, consultation requests, and a clean path to CRM/email integrations.

Moving the whole site to a conventional application stack would have duplicated working design and content systems. A membership plugin would have added another subscription and another platform boundary. Keeping authorization in browser scripts was not acceptable.

I kept Webflow as the site and added a proper application boundary on Webflow Cloud.

## What I built

- Registration and login with email/password and Google
- Member bootstrap and profile management
- Public newsletter and authenticated consultation flows
- Server-authorized gated downloads
- English/Arabic and LTR/RTL interactive states
- Durable, idempotent events for future CRM/email processing
- Separate staging and production release paths

## Architecture

~~~mermaid
flowchart TD
    W["Webflow<br/>Pages, CMS, design system"]
    C["Code Components<br/>Interactive member UI"]
    A["Webflow Cloud<br/>Next.js APIs and business rules"]
    I["Clerk<br/>Identity and sessions"]
    D["D1 + Drizzle<br/>Application data"]
    O["Integration outbox<br/>CRM and email events"]

    W --> C
    C --> A
    A --> I
    A --> D
    A --> O
~~~

The boundary is intentional:

| Layer | Responsibility |
| --- | --- |
| Webflow | Pages, content, visual system, responsive behavior, EN/AR and RTL |
| Code Components | Accessible interactive UI inside Webflow-authored surfaces |
| Webflow Cloud / Next.js | Validation, authorization, business rules and protected delivery |
| Clerk | Registration, verification, recovery and sessions |
| D1 / Drizzle | Members, submissions, content, enrollments, rate limits and outbox records |
| Integration outbox | Durable handoff to asynchronous CRM/email consumers |

The browser presents state. It never grants access. Protected content is authorized again on the server and streamed without exposing its source reference.

## Why I chose this approach

It preserved the parts of Webflow that were already working well while moving identity, data, and access control to the server.

I also kept third-party integrations out of the request path. Registration, forms, and enrollment do not wait for a CRM or email provider. They write an idempotent outbox event that a separate consumer can process safely.

Performance work focused on fewer requests and fewer database round trips: coalesced reads, single-flight bootstrap, indexed joined queries, atomic rate-limit decisions, and batched domain/outbox writes.

## Live evidence

The production site is public:

- [Dar Al Tharwah](https://daraltharwa.com/)
- [Books library](https://daraltharwa.com/books)
- [Bingo — registered-member download state](https://daraltharwa.com/books/bingo-the-path-to-wealth)
- [The Wealth Seeker — registered-member download state](https://daraltharwa.com/books/the-wealth-seeker)

The private repository contains the implementation, tests, migrations, CI history, and release evidence. It remains the source of truth.

## Results

- The membership MVP is live in production.
- The existing Webflow design and editorial workflow stayed intact.
- Gated content is enforced by server APIs rather than hidden UI.
- The product supports public and authenticated workflows in English and Arabic.
- No separate membership SaaS subscription was introduced.

Dar Al Tharwah has a business audience of more than 40,000 people. That is business context, not a claim of 40,000 registered members or 40,000 concurrent users.

At launch, the project operated within the existing Webflow plan and avoided a separate membership-platform subscription. Webflow Cloud usage limits and possible overages still apply; I do not claim zero marginal cost at every future load level.

## Repository scope

This is a portfolio case study, not an open-source copy of production. It contains no production code, credentials, customer data, environment values, or private operational configuration.

For the full decision record, read [CASE_STUDY.md](CASE_STUDY.md). For system boundaries and data flows, see [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).
