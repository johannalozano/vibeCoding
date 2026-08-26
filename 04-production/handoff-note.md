# Engineering Handoff Note

> Module 4 · Production Specs. Open the black box, make the build legible to an engineer.

## What this is

This is a front-end-only, clickable prototype of a **Daily Action Queue** for a Customer Success / revenue rep. It replaces the morning shadow-spreadsheet ritual with one ranked, cross-object todo list covering deals, onboardings, support escalations, and follow-ups, with one-tap actions for common updates. The app is built with TanStack Start, React 19, and Tailwind CSS v4. State lives entirely in memory; there is no backend, authentication, or persistence. The two implemented screens are **Daily Action Queue** (`/`) and **Email** (`/email?item=<id>`). Everything else—including sidebar data, breadcrumbs, the fetch layer, and email sending—is mocked to validate the interaction model before committing to a real CRM integration.

## Architecture (plain language)

- **Frontend:** TanStack Start v1 with React 19, Radix primitives, shadcn/ui components, Tailwind CSS v4, and file-based routing. Thin route files live in `src/routes/`, while screen logic and UI live in feature folders.
- **Backend / data:** There is no backend in this prototype. Queue items, contacts, email addresses, threads, and fetch behavior are mocked. State is held in a shared in-memory store at `src/store/crm-store.ts`.
- **Key flows:** Queue items are ranked by urgency → committed date → account value. Users can search and filter, then take inline actions such as logging calls, adding notes, updating stages, snoozing, completing, or moving into the email flow. Email composition derives context from the selected queue item and supports simulated sending or a real `mailto:` handoff.

### Frontend details

- **Framework:** TanStack Start v1 with file-based routing and Vite 8.
- **UI library:** React 19 + Radix primitives + shadcn/ui components under `src/components/ui/`.
- **Styling:** Tailwind CSS v4 via `src/styles.css`; neo-brutalist editorial theme with Archivo headings and JetBrains Mono for data.
- **Routing:** File-based.
  - `/` → `DailyActionQueueScreen`
  - `/email` → `EmailScreen`, which reads the `?item=` search parameter
- **State:** A single hand-rolled in-memory store at `src/store/crm-store.ts`, subscribed to with `useSyncExternalStore`. It holds queue items and the sent-email log. Both screens read from and write to this store.

### Backend / data details

There is **no backend** in this prototype.

- `src/features/daily-action-queue/data/queue-model.ts` — domain types and pure functions for ranking, urgency, value formatting, and rationale.
- `src/features/daily-action-queue/data/queue-items.ts` — mocked seed dataset (`SEED_ITEMS`) with relative dates so the queue remains populated across urgency buckets.
- `src/features/email/data/email-templates.ts` — derived mock contact addresses, subject/body builders, templates, and fake thread history.
- `src/lib/mock-fetch.ts` — simulates network delay and controllable failures for loading/error states.

### Key flows in detail

#### 1. Daily queue flow

- Seed items are ranked by `urgency → committed date → account value`.
- Users can filter by search text, When, Work type, and Stage.
- Inline actions—call, note, stage, snooze, complete, and email—patch the item in the shared store and append to an in-memory activity log.
- Selecting **Email** navigates to `/email?item=<id>` with the relevant contact and context pre-filled.

#### 2. Email flow

- `EmailScreen` reads `itemId` from the URL.
- It looks up the item in the shared store and derives recipient, subject, and body.
- Users can select a template, edit the message, and either simulate sending or open a real `mailto:` link.
- Simulated sends append to the store's `sent` array and update the originating item's `lastTouch`.

#### 3. Shell flow

- `CrmSidebar`, `CrmBreadcrumb`, `LoadingState`, and `NoDataState` wrap both screens.
- A fetch-health toggle in the sidebar allows QA to force the **No data** state and test retry behavior.

## What's solid vs. what's duct tape

| Area | State | Notes |
|---|---|---|
| Feature folder structure | solid | Components, data, and routes are grouped by screen: `daily-action-queue`, `email`, and `crm-shell`. |
| Pure domain logic | solid | Ranking, urgency, formatting, and rationale are deterministic functions with no side effects. |
| Shared state contract | solid | `useCrmState` plus `patchItem` / `recordSentEmail` provide a minimal but consistent shared-store pattern. |
| Route structure | solid | Route files are thin metadata wrappers; UI lives inside feature components. |
| Loading / error UX | solid | `mockFetch` provides controllable delay, failure, retry, and empty-data states. |
| Design consistency | solid | Screens share the same shell, typography, signal colors, and component primitives. |
| Persistence | rough | Refreshing the browser resets all state, including sent emails and completed items. |
| Authentication | rough | There is no login, session management, role-based access, or permissions model. |
| Email | rough | “Send” only writes to memory; `mailto:` is the only real external handoff. |
| CRM integration | rough | The prototype does not call Salesforce, HubSpot, or any other real CRM API. |
| Data | rough | Contacts, threads, email addresses, and queue items are derived from mocked seed data. |
| Automated tests | rough | The prototype has been verified manually and through build/typecheck only. |
| Audit / undo | rough | Completed or snoozed items can be changed back during the session, but there is no durable audit history. |

## Risks & assumptions for the team

| Risk | Impact | Mitigation before production |
|---|---|---|
| Ranking algorithm is unvalidated | Reps may not trust the order and may keep using their own spreadsheet | Run user research on priority signals and test alternative ranking variants |
| In-memory state is lost on refresh | Data decay and unusable production behavior | Replace the store with a database-backed model and optimistic UI |
| Mock fetch does not represent real latency | Performance assumptions may be wrong | Instrument real endpoints and introduce production loading/caching strategies |
| Email is only `mailto:` / simulated send | No deliverability, tracking, compliance, or reply capture | Integrate a real email provider and add consent/audit logging |
| No authentication | Cannot support real multi-user usage | Add authentication, authorization, and row-level security |
| Narrow IC-focused scope | Manager, reporting, reassignment, and bulk workflows are missing | Validate the IC workflow first, then expand based on evidence |

### Key assumptions

- The primary user is an individual contributor who owns a book of accounts end-to-end.
- A deterministic rank based on urgency, committed date, and account value is sufficient to replace the shadow spreadsheet.
- One-tap, low-friction updates will improve CRM hygiene more effectively than multi-field forms.
- Email context can be derived from the queue item without requiring a full contact record.
- A shared `committed` concept can eventually be mapped across deals, onboarding, support, and follow-up objects.
- Users will trust a ranked queue if the system explains why each item appears where it does.

## How to run it

### Prerequisites

- Node.js
- `bun` or `npm`
- This repo checked out locally

### Install dependencies

```bash
bun install
# or
npm install
```

### Development server

```bash
bun run dev
# or
npm run dev
```

The app runs at:

```text
http://localhost:8080
```

### Build

```bash
bun run build
# or
npm run build
```

### Lint / format

```bash
bun run lint
bun run format
```

### Useful paths for the next engineer

- Routes: `src/routes/index.tsx`, `src/routes/email.tsx`
- Screens:
  - `src/features/daily-action-queue/components/DailyActionQueueScreen.tsx`
  - `src/features/email/components/EmailScreen.tsx`
- Domain data:
  - `src/features/daily-action-queue/data/queue-model.ts`
  - `src/features/daily-action-queue/data/queue-items.ts`
  - `src/features/email/data/email-templates.ts`
- Shared state: `src/store/crm-store.ts`
- Shell: `src/features/crm-shell/components/`
- Mock fetch: `src/lib/mock-fetch.ts`

---

*This handoff reflects the prototype as currently shipped. Any productionization work should begin with the Risks & assumptions and What's solid vs. what's duct tape sections.*
