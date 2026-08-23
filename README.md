# Dar Al Tharwah Members

A production case study in extending a Webflow website with a secure, scalable membership application—without rebuilding the site or adding a paid membership platform.

[View the live website](https://daraltharwa.com/) · [Read the full case study](CASE_STUDY.md) · [Explore the architecture](docs/ARCHITECTURE.md)

> This repository is a public portfolio representation. It contains no production source code, credentials, customer data, or private operational configuration.

## What this project is

Dar Al Tharwah Members adds registration, authentication, member profiles, gated resources, newsletter capture, consultation requests, and CRM-ready integration events to an existing bilingual Webflow website.

The production MVP is live. The engineering challenge was not simply to add login screens; it was to introduce a custom application layer while preserving Webflow as the visual, editorial, and responsive source of truth.

## The problem solved

Webflow provided the right publishing and design environment, but its native capabilities did not cover the required membership and application workflows. Replatforming would have discarded an established design system and content workflow. Subscription membership tools would have introduced recurring cost, vendor constraints, and another operational dependency.

The chosen approach extended the existing Webflow investment:

- Webflow remains responsible for pages, content, styles, responsive behavior, English/Arabic presentation, and RTL.
- Reusable Webflow Code Components add interactive member experiences.
- A mounted Next.js application on Webflow Cloud handles APIs and business rules.
- Clerk provides identity, verification, recovery, and session security.
- Webflow Cloud D1 and Drizzle store application-owned data.
- A durable outbox records idempotent CRM/email events without making user requests depend on external vendors.

## Why the architecture is interesting

This is a hybrid no-code + custom-engineering system with explicit responsibility boundaries. It avoids both extremes: forcing complex application logic into the visual layer, or replacing the Webflow site with a separate frontend.

The result supports an audience of more than 40,000 users while adding no recurring platform subscription beyond the existing premium Webflow plan. That scale statement describes the product context and intended operating audience; this public repository does not claim a 40,000-user concurrent load test.

## Technology overview

| Layer | Responsibility |
| --- | --- |
| Webflow | Pages, CMS/content, design system, responsive behavior, localization, RTL |
| Webflow Code Components | Accessible interactive member UI inside Webflow-authored surfaces |
| Webflow Cloud / Next.js | APIs, validation, authorization, business rules, content delivery |
| Clerk | Registration, authentication, verification, recovery, sessions |
| Webflow Cloud D1 + Drizzle | Members, submissions, content, enrollments, rate limits, outbox |
| Integration outbox | Durable, idempotent CRM/email work for asynchronous consumers |
| Turnstile | Server-verified abuse protection for forms |

## Demonstrated product and engineering outcomes

- Kept the live Webflow site and design workflow intact.
- Delivered a production membership MVP with email/password and Google sign-in.
- Added public newsletter and authenticated consultation workflows.
- Enforced gated-content access and delivery on the server.
- Preserved bilingual English/Arabic, LTR/RTL, responsive, and accessible behavior.
- Reduced avoidable browser and database work through request coalescing, indexed joined reads, atomic rate-limit decisions, and batched related writes.
- Avoided an additional membership-plugin or membership-SaaS subscription.
- Established a clean integration boundary for future CRM/email processing.

## Scope of this repository

This public repository documents the problem, decisions, trade-offs, architecture, and verified outcomes. The private production repository remains the source of truth and is intentionally not mirrored here.

For the complete narrative, see [CASE_STUDY.md](CASE_STUDY.md). For system boundaries and data flows, see [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).
