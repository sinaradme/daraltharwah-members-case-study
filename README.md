# Dar Al Tharwah Members

A case study of designing and building a custom membership experience for an existing English and Arabic Webflow website.

**My role:** product design, Webflow design and development, application architecture, and implementation.

[Read the case study](CASE_STUDY.md) · [Explore the architecture](docs/ARCHITECTURE.md) · [Visit the live website](https://daraltharwa.com/)

## The project

Dar Al Tharwah needed accounts, personal resources, protected downloads, and course progress connected to its public educational website.

I extended the existing site with a Webflow Cloud application. Webflow continues to manage pages, CMS content, and presentation; the application handles member data and trusted access decisions.

## What was delivered

- Email/password and Google sign-in, account verification, and recovery
- Member profiles and a personal resource dashboard
- Server-authorized courses and download delivery
- Course enrollment and lesson progress
- Public newsletter and authenticated consultation workflows
- English and Arabic interfaces with right-to-left support

## Why this architecture

Webflow owns publishing and presentation, Clerk owns identity, and a Next.js application on Webflow Cloud owns application rules. D1 stores the product's profiles, resources, enrollments, and progress.

This separation preserves the site's publishing workflow while providing a foundation for member services. Performance work reduces duplicate reads, database round trips, and unnecessary sequential waits while retaining server-side authorization.

## Outcome and scope

The membership MVP is live. It adds member capabilities to the existing website without a separate membership SaaS subscription at launch.

CRM integration remains in development. Payments, subscriptions, certificates, advanced assessments, and a full administration portal are outside the current MVP. Course playback uses unlisted YouTube videos, whose links can be shared.

The case study describes verified capabilities and architectural decisions. It makes no claim of measured conversion growth, quantified savings, or benchmarked latency improvements.

## Explore

- [Case study](CASE_STUDY.md): challenge, role, approach, decisions, outcome, and lessons
- [Architecture](docs/ARCHITECTURE.md): system responsibilities, data flows, and trade-offs
- [Portfolio guide](docs/PORTFOLIO_GUIDE.md): editorial and confidentiality requirements
- [Live books collection](https://daraltharwa.com/books): public resource entry points

This is a documentation-only portfolio repository. Production implementation and operational records remain private.
