# Dar Al Tharwah Members
## Extending a Webflow website with a custom membership experience

Dar Al Tharwah needed a member experience connected to its existing educational website: account access, personal resources, protected downloads, and course progress. I designed and implemented a membership layer using Webflow, Webflow Cloud, Next.js, Clerk, and D1.

The resulting production MVP brings those capabilities into the existing English and Arabic website while preserving Webflow as the team's publishing and design environment.

## My role

I was responsible for product design, Webflow design and development, application architecture, and implementation of the custom membership layer. My work connected the member journeys and interface states to the identity, data, and access rules behind them.

## The challenge

The public website already had its own content structure and visual language. Adding membership introduced a different set of requirements: visitors needed to become members, members needed a personal dashboard, and access to protected resources needed to depend on trusted account and enrollment data.

The challenge was to make these journeys part of one website while keeping content management practical. Moving all presentation into a separate application would have changed the publishing workflow. Putting access decisions in browser components would have weakened the protection of member resources.

The business also had an audience of more than 40,000 people. That context made a maintainable foundation important, although it did not establish a registered-member count or concurrent-user requirement.

## The approach

I separated responsibilities around the work each platform was suited to own:

- **Webflow** manages public pages, CMS content, the visual system, and English/Arabic presentation.
- **Webflow Cloud and Next.js** handle application rules, validated requests, member state, and protected delivery.
- **Clerk** manages identity, authentication, verification, and sessions.
- **D1 and Drizzle** manage application profiles, resources, enrollments, and progress.

This structure allowed the public site and membership features to evolve together. Reusable components connect Webflow-authored pages to the application layer, while trusted decisions remain on the server.

## The member experience

### From visitor to member

Members can register and sign in with email and password or Google. Clerk handles verification, recovery, and sessions. The application then initializes the member profile and stores the information needed by the product.

Profile completion requires a first and last name; phone and country are optional. This keeps the initial account requirements focused while supporting later profile updates.

### A personal place for resources

The dashboard gives members a place to return to their resources. Course enrollment and lesson completion connect access with progress, allowing the experience to continue across sessions.

The interface supports English and Arabic, including right-to-left presentation. Webflow remains responsible for visual composition, with interactive components handling account, loading, error, and success states.

### Access that follows product rules

Protected courses and downloads use server-derived identity and enrollment checks. A hidden button or a browser-supplied account claim cannot grant access.

Downloads are authorized again when delivered, and their underlying source references stay on the server. Authorized course viewers receive the playback information they need. The use of unlisted YouTube videos supports the current course experience, with the understood limitation that those links can be shared.

### Forms with different purposes

The newsletter remains available to public visitors. Consultation requests require an authenticated, initialized member. Both workflows validate submissions and apply consent and abuse-prevention controls appropriate to their purpose.

These distinctions let the site support audience growth and member services without treating every visitor as an account holder.

## Key decisions and trade-offs

### Keep identity separate from application data

Using Clerk for identity avoided introducing custom password and session management. D1 stores the profile and learning relationships owned by the application. This makes the boundary between signing in and receiving product access explicit.

### Reduce unnecessary work in the member journey

Performance work focused on reducing duplicate requests, database round trips, and sequential waits. Identical browser reads are coalesced, joined database reads resolve related resource information, and independent server work can run concurrently where authorization rules allow it.

Public syllabus metadata also avoids unnecessary identity processing. Protected member operations retain their authentication checks. These decisions reduce avoidable work without relying on shared caching of private member data; a numerical latency improvement would require a separate benchmark.

### Keep external integrations off the critical path

Accepted member and form operations record durable integration events alongside application data. This creates a foundation for downstream services without making registration or submission depend on a third-party response.

CRM integration remains work in progress. Its completion and production activation are a next phase, rather than an outcome of this MVP.

### Define a focused MVP

The first version concentrates on membership, resources, gated learning, and forms. Payments, subscriptions, certificates, advanced assessments, and a full administration portal are outside its current scope.

That scope kept the architecture aligned with the immediate member experience and left larger product decisions for later phases.

## Outcome

The membership MVP is live on [Dar Al Tharwah](https://daraltharwa.com/). The existing website now supports account access, profiles, a member dashboard, protected resources, enrollment, and lesson progress within its English and Arabic experience.

The project retained Webflow's publishing workflow and introduced a custom application foundation for member services. No separate membership SaaS subscription was introduced at launch.

The outcome is a delivered product capability and a maintainable separation of responsibilities. Conversion gains, quantified cost savings, and measured latency improvements are not established results of this case study.

Public entry points include the [books collection](https://daraltharwa.com/books) and resource pages such as [Bingo: The Path to Wealth](https://daraltharwa.com/books/bingo-the-path-to-wealth) and [The Wealth Seeker](https://daraltharwa.com/books/the-wealth-seeker).

## What I learned

A membership experience works best when its interface and access rules are designed together. Account status, profile readiness, enrollment, and progress each represent a different product state, and making those distinctions explicit helps both the interface and the application remain understandable.

The platform boundary was equally important. Keeping publishing in Webflow, identity in Clerk, and application rules in Webflow Cloud gave each part a clear responsibility. Performance improvements could then target unnecessary work without changing those boundaries.

A focused first release also creates a clearer path forward. The current foundation supports future integration work, while the case study can describe the value already delivered without presenting planned capabilities as completed results.

For a deeper technical explanation, see the [architecture document](docs/ARCHITECTURE.md).
