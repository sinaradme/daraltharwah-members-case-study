# Dar Al Tharwah Members — Webflow Cloud Case Study

I built the member layer for Dar Al Tharwah as an extension of its existing Webflow site. The production application is live; its code and operational details remain private.

My role covered product design, Webflow design and development, and the architecture and implementation of the custom application layer on Webflow Cloud.

My goal was not to force Webflow to behave like a backend, and it was not to replace a site that already worked. I kept presentation in Webflow and moved identity, data, authorization, and delivery into a mounted application on Webflow Cloud.

## 1. Project Overview

The member experience includes:

- Email/password and Google registration and login
- Member initialization and profile management
- A public newsletter form
- An authenticated consultation form
- Gated courses and downloads
- Server-authorized file delivery
- Durable integration events for CRM/email processing

Webflow still owns the pages, CMS, design system, responsive behavior, English/Arabic content, and RTL presentation. The application adds the stateful and security-sensitive behavior around it.

## 2. Business Challenge

Dar Al Tharwah needed membership functionality, but the larger problem was architectural.

The site already had a working Webflow design and publishing workflow. Rebuilding it in Next.js would have duplicated the frontend and made content work harder. A membership plugin would have introduced another subscription, vendor model, and integration surface. Browser-only membership logic would not provide trustworthy authorization or a durable data model.

I needed a solution that kept Webflow useful while adding a real application boundary.

## 3. Constraints

I worked within six constraints:

- Webflow had to remain the visual and editorial source of truth.
- The custom layer could not create a second site shell.
- Passwords, verification, recovery, and sessions needed a dedicated identity provider.
- Protected content had to be authorized on the server.
- English/Arabic, LTR/RTL, responsive and accessible states had to remain consistent.
- Production and staging needed a controlled release path.

I also wanted to avoid a separate membership SaaS subscription. That influenced the architecture, but it did not justify weakening identity or access control.

## 4. Why Webflow + Webflow Cloud

Webflow already handled the work it was good at: page composition, CMS content, styles, responsive behavior, and localization.

Webflow Cloud supplied the missing application layer. I could mount Next.js APIs alongside the site, keep browser requests on the same product surface, and handle business rules on the server.

This split avoided a replatforming project:

| Layer | What I kept there |
| --- | --- |
| Webflow | Content, layout, visual system, breakpoints, EN/AR and RTL |
| Code Components | Interactive UI inside Webflow pages |
| Webflow Cloud | APIs, validation, authorization and delivery |
| Clerk | Identity and sessions |
| D1 / Drizzle | Application-owned data |
| Integration outbox | Durable external-work events |

The trade-off is coordination across several surfaces. I addressed that with explicit ownership, shared API contracts, tests, staging, and release documentation.

## 5. Solution Architecture

A normal request moves through these layers:

~~~text
Webflow page
  -> Code Component or browser runtime
  -> mounted Next.js API on Webflow Cloud
  -> server domain/security service
  -> Clerk and/or D1
  -> optional integration-outbox event
~~~

The browser receives state and renders the correct experience. It does not decide whether a member can access content.

For a gated download, the server resolves the current member, content record, and enrollment. The delivery route authorizes the request again, validates the destination, follows only approved redirects, and streams the file without returning the underlying delivery reference.

For forms, the server validates the Origin and payload, applies rate limiting, verifies Turnstile, persists the submission, and writes its integration event.

The detailed public architecture is in [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## 6. Technical Decisions

### Clerk owns identity

I did not build password or session handling. Clerk owns registration, verification, recovery, Google sign-in, and sessions. The server maps the authenticated identity to one internal member.

D1 never stores passwords, verification codes, cookies, JWTs, session tokens, CAPTCHA tokens, or full Clerk objects.

### D1 stores application data

D1 contains members, content records, enrollments, form submissions, rate-limit counters, and integration events. Drizzle provides a typed schema and ordered migration history.

I used joined reads and batches where they reduced round trips without weakening privacy or correctness.

### Authorization stays on the server

A hidden button is not access control. Code Components can show signed-out, locked, enrollment, or ready states, but the server remains authoritative.

The content source is not returned to the browser. Delivery is checked again at request time.

### Integrations use an outbox

Registration, forms, and enrollment should not fail because a CRM or email provider is unavailable.

The application writes versioned, deduplicated outbox events with the related domain operation. A separate worker can claim and process them with retries.

The producer is implemented. The downstream CRM/email consumer remains an extension point; I do not present it as a completed real-time sync.

### Performance has explicit budgets

I focused on avoidable work:

- Identical simultaneous member reads are coalesced.
- Member bootstrap and runtime configuration are single-flight.
- Content access and resource lists use indexed joined reads.
- Rate limiting uses one atomic statement.
- Related domain and outbox writes are batched.
- External verification and content-provider waits are bounded.

The private project documents measurable reductions, including two concurrent normal GETs becoming one browser request and content access moving from three D1 statements to one joined query.

## 7. Security Approach

The main security rules are straightforward:

- Identity comes from Clerk on the server, not from browser-supplied user IDs or roles.
- State-changing requests require an approved exact Origin.
- Untrusted values are parsed, normalized, bounded, and validated server-side.
- Turnstile is verified on the server and its token is discarded.
- Rate-limit subjects are hashed rather than stored raw.
- Member data remains private and is not shared-cached.
- Delivery targets and redirects are allowlisted.
- Browser state never grants access.
- Secrets remain outside source code and documentation.

The public repository intentionally omits credentials, environment values, provider identifiers, host allowlists, customer records, and private deployment details.

## 8. Scalability Considerations

Dar Al Tharwah has a business audience of more than 40,000 people. I designed the member layer with that audience in mind.

This is not a claim of 40,000 registered members, 40,000 concurrent sessions, or a published load test.

The relevant architecture choices are stateless routes, indexed reads, idempotent bootstrap, request coalescing, batched writes, bounded provider calls, and asynchronous integration delivery. These reduce avoidable load and keep vendor latency out of core user operations.

Operational monitoring, outbox processing, rate-limit retention, and private-object delivery remain explicit extension areas.

## 9. Cost Optimization Impact

The project avoided a separate membership plugin or membership SaaS subscription. At launch, it operated within the Webflow plan already used by the site.

The saving is therefore specific: no additional recurring membership-platform subscription and no replatforming project.

I do not publish a currency savings estimate because there is no defensible vendor comparison in the production documentation. I also do not claim that future Webflow Cloud usage is free; plan limits and usage-based overages may apply as traffic grows.

## 10. Results and Business Value

The production MVP is live.

Publicly verifiable examples include:

- [Dar Al Tharwah](https://daraltharwa.com/)
- [Books library](https://daraltharwa.com/books)
- [Bingo registered-member gate](https://daraltharwa.com/books/bingo-the-path-to-wealth)
- [The Wealth Seeker registered-member gate](https://daraltharwa.com/books/the-wealth-seeker)

The result is practical:

- The Webflow site and editorial workflow stayed in place.
- Registration, login, profiles, forms, and gated downloads are working in production.
- Protected content is enforced by server APIs.
- The member experience supports English/Arabic and LTR/RTL.
- CRM/email work has a durable integration boundary.
- The business did not add a separate membership SaaS subscription.

Implementation details, tests, migrations, CI checks, and release history are verified in the private production repository.

## 11. Key Learnings

- A Webflow limitation does not automatically require a rebuild.
- A hybrid system works when every layer has a clear owner.
- UI state and authorization must remain separate.
- External integrations should not sit in the critical user-request path.
- Performance improvements are more credible when expressed as request and query budgets.
- A public case study can show engineering judgment without exposing private code.

## Evidence and maintenance

The private production repository is the source of truth. This case study must be reviewed whenever production changes materially affect architecture, identity, data ownership, security, delivery, integration boundaries, scalability, cost, or verified results.

If a public claim cannot be verified against the current production state or owner-provided business context, it must be narrowed or removed.
