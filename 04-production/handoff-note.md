# Engineering Handoff Note

> Module 4 · Production Specs. Open the black box, make the build legible to an engineer.

## What this is

This is a front-end-only prototype of a **Daily Action Queue** for Customer Success / revenue reps. It combines deals, onboarding tasks, support escalations, and follow-ups into one ranked list so users can decide what to do next without maintaining a separate spreadsheet. The prototype includes a Daily Action Queue and an Email workflow, with inline actions such as logging calls, adding notes, changing stages, snoozing, completing, and composing emails. It is built with TanStack Start, React 19, and Tailwind CSS v4. All data and state are mocked and stored in memory; there is currently no backend, authentication, persistence, or real CRM/email integration.

## Architecture (plain language)

- **Frontend:** TanStack Start with React 19, Tailwind CSS v4, Radix/shadcn UI components, and file-based routing. The main screens are the Daily Action Queue (`/`) and Email (`/email?item=<id>`). Shared client state is managed through a lightweight in-memory store using `useSyncExternalStore`.
- **Backend / data:** There is no real backend. Queue items, contacts, email threads, and addresses come from mocked seed data. Ranking, urgency, formatting, and priority rationale are calculated with deterministic frontend functions. A mock fetch utility simulates network loading and failure states.
- **Key flows:** Queue items are ranked by urgency → committed date → account value. Users can search/filter the queue and perform inline actions that update the shared store. Selecting Email passes the queue item through the URL, pre-fills the relevant contact and context, and allows the user to simulate sending or hand the message off through `mailto:`.

## What's solid vs. what's duct tape

| Area | State | Notes |
|---|---|---|
| Feature architecture | solid | Screens, domain logic, data, routes, and shell components are separated into clear feature folders. |
| Ranking logic | solid | Ranking, urgency, formatting, and rationale are deterministic pure functions with no side effects. |
| Shared state | solid | Both screens use one consistent in-memory store and immediately reflect queue/email updates. |
| Loading & error UX | solid | Loading, failure, retry, and empty-data behaviors are implemented and controllable for testing. |
| Persistence | rough | All state resets on refresh; there is no database or durable activity history. |
| CRM integration | rough | Queue records, contacts, sidebar data, and CRM objects are mocked rather than connected to Salesforce, HubSpot, or another CRM. |
| Email | rough | Sending is simulated. `mailto:` is the only real external handoff; there is no inbox sync, deliverability, reply capture, or tracking. |
| Authentication | rough | There are no users, sessions, permissions, roles, or row-level access controls. |
| Automated testing | rough | The prototype has been validated manually and through build/typechecking, but does not yet have an automated test suite. |

## Risks & assumptions for the team

The biggest product risk is that the current priority model—urgency, committed date, then account value—has not yet been validated with real users. If reps disagree with the ranking, they may continue using their own spreadsheet despite the improved interface. The build also assumes that a single `committed` date can be consistently derived from real CRM objects, even though deals, support tickets, onboarding tasks, and follow-ups may each use different fields and semantics. Before production, the team will also need to define what actions such as **Snooze** and **Complete** change in the source CRM, replace the in-memory store with persistent data, implement authentication and permissions, and decide whether email becomes a full send-and-sync integration or remains a compose-and-handoff workflow.

## How to run it

```bash
# Install dependencies
bun install
# or
npm install

# Start development server
bun run dev
# or
npm run dev

# App runs at:
# http://localhost:8080

# Production build
bun run build
# or
npm run build

# Lint / format
bun run lint
bun run format
```
