# Portfolio Repository Guide

## Purpose

This repository is a public portfolio representation of the Dar Al Tharwah Members project.

Its audience is recruiters, hiring managers, technical reviewers, product leaders, and future maintainers who need to understand the problem, architecture, engineering decisions, trade-offs, and business impact without accessing the private production system.

This repository is documentation-only. It is not a deployable copy, an open-source edition, a backup, or a sanitized fork of production.

## Source of truth

The private production repository is the only source of truth for:

- Current behavior
- Source code
- Tests
- Database schema and migrations
- Deployment and rollback procedures
- Environment configuration
- Provider configuration
- Security controls
- Operational status
- Production and staging revisions

The public case-study repository may summarize verified facts from the private documentation, but it must never override or compete with the production documentation.

If the two repositories appear inconsistent, do not “fix” production from this repository. Re-review the current private documentation, confirm the intended public claim, and update only the case study unless the owner separately authorizes production work.

## Non-negotiable separation

Never copy, expose, or reconstruct production implementation details here.

Do not add:

- Production source code or substantial implementation excerpts
- Database schemas, migration files, seed data, or record exports
- Customer, member, form, analytics, or CRM data
- Credentials, secrets, tokens, cookies, keys, or signed URLs
- Environment-variable values
- Internal hostnames, binding identifiers, account identifiers, or provider project IDs
- Complete allowlists, origin lists, delivery destinations, or private file references
- Production logs, stack traces containing private context, or incident evidence
- Screenshots containing personal data, admin controls, configuration panels, or private URLs
- Private issue, pull-request, commit, workflow, or deployment details
- Vendor contracts, pricing documents, or unsupported savings calculations
- A copied directory tree detailed enough to reconstruct private implementation
- Claims that cannot be traced to current production documentation or owner-provided business context

Never change the private repository's visibility or mirror its Git history into this repository.

## What may be improved

Future updates should focus on presentation and explanation:

- Refine the README as a concise portfolio landing page
- Improve the full case-study narrative
- Clarify engineering trade-offs and decision rationale
- Add high-level Mermaid diagrams
- Add carefully redacted product screenshots
- Add architecture or flow illustrations that reveal no private details
- Improve accessibility, link structure, navigation, and document consistency
- Add a public changelog for case-study documentation
- Update verified results or business context after reviewing current sources
- Improve wording for recruiter, engineering, and product audiences

Updates should remain documentation-first. Do not turn this repository into a demo application unless the owner explicitly changes its purpose and defines a safe, independent implementation.

## Claim standard

Every public statement must fit one of these categories:

1. **Verified production behavior** — supported by current private production documentation or release evidence.
2. **Owner-provided business context** — clearly framed without inventing technical proof.
3. **Architectural rationale** — an explanation of a documented decision or trade-off.
4. **Deliberate limitation or roadmap item** — clearly identified as not currently implemented.

Use precise language:

- Say “designed for an audience of more than 40,000 users,” not “load-tested at 40,000 concurrent users,” unless a verified test exists.
- Say “CRM-ready integration events are written to a durable outbox,” not “fully synchronized CRM,” while the downstream consumer remains an extension point.
- Say “no additional recurring platform subscription beyond the existing premium Webflow plan,” not a currency savings amount unless a defensible comparison is documented.
- Say “server-authorized gated delivery,” not “unbreakable” or “perfectly secure.”
- Say “production MVP is live and owner-confirmed,” not that every future roadmap capability is live.

Avoid vague terms such as revolutionary, enterprise-grade, seamless, cutting-edge, infinitely scalable, or military-grade. Prefer observable behavior, clear ownership, and stated limits.

## Screenshot and diagram rules

### Screenshots

Before adding any screenshot:

1. Confirm the page is public or use a purpose-built demo account and approved test data.
2. Remove names, email addresses, phone numbers, account IDs, tokens, URLs containing identifiers, and browser extension details.
3. Crop out admin panels, provider dashboards, environment configuration, deployment metadata, and unrelated tabs.
4. Verify both English and Arabic presentation when the screenshot is intended to demonstrate localization.
5. Add useful alt text that explains the product state, not decorative appearance.
6. Store only the final redacted asset.

If safe redaction cannot be proven, do not publish the screenshot.

### Diagrams

Keep diagrams at the responsibility-boundary level. It is safe to show:

- Webflow presentation
- Webflow Cloud application
- Clerk identity
- D1 application data
- Integration outbox and an abstract downstream consumer

Do not show private hostnames, account IDs, route mounts, table columns containing personal data, secret names or values, provider dashboards, or detailed network topology.

## Safe update workflow for future agents

1. Read this guide before changing the repository.
2. Confirm that the task concerns the public portfolio repository.
3. Read the current private production documentation without modifying production.
4. Identify the exact facts needed for the public update.
5. Separate verified current behavior from roadmap items.
6. Draft the minimum public explanation needed.
7. Review the draft for confidential information and unsupported claims.
8. Update documentation, screenshots, diagrams, or explanations only.
9. Check every link and Mermaid diagram.
10. Confirm the private production repository was not modified.
11. Report changed files and the evidence used for public claims.

If a requested update requires production source code, secrets, customer data, repository visibility changes, or uncertain confidential details, stop and ask the owner. Do not attempt to sanitize sensitive material through guesswork.

## Repository structure

- **README.md** — concise landing page and navigation
- **CASE_STUDY.md** — complete problem, decision, architecture, impact, and learning narrative
- **docs/ARCHITECTURE.md** — public high-level system explanation
- **docs/PORTFOLIO_GUIDE.md** — this maintenance and confidentiality contract

Additional public documentation should have a defined audience and should be linked from the README or case study. Avoid duplicate files that create competing versions of the same story.

## Review checklist

Before merging any future update, confirm:

- [ ] Only the public case-study repository was changed.
- [ ] The private production repository remains private and unmodified.
- [ ] No production code or confidential implementation detail was copied.
- [ ] No secret, credential, personal data, or private identifier is present.
- [ ] Current behavior and future work are clearly separated.
- [ ] Scale and cost claims use precise, supportable wording.
- [ ] CRM/email capability is not overstated.
- [ ] Screenshots are fully redacted and approved.
- [ ] Diagrams remain high-level.
- [ ] README, case study, architecture, and guide do not contradict one another.
- [ ] Links render correctly on GitHub.
