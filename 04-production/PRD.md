# PRD — Daily Action Queue (CRM prototype)

Status: prototype-derived PRD. Everything described below is reverse-engineered from a clickable front-end prototype with **no backend**. Sections flag mocked vs. real explicitly. Product assumptions that the prototype cannot validate are labeled as such rather than treated as established facts.

## Problem

Revenue and customer-success teams keep a "shadow spreadsheet" next to the CRM because the CRM tells them what *exists* (records, pipelines, tickets) but not what to *do next today*. Work spans deals, onboardings, support escalations and follow-ups, each living in a different object with a different stage model, so the rep rebuilds a single prioritized list by hand every morning. Logging the outcome afterwards costs a multi-field form, so it often doesn't happen and the data decays.

**Prototype-supported hypothesis:** if the CRM presents one cross-object list ranked by urgency, committed date and account value, states *why* each item is on the list, and lets the rep record the outcome inline with no required fields, reps should need less manual prioritization and should be more likely to keep touch data current. The prototype demonstrates the interaction feasibility of this approach: the queue is presented on one screen, rank and rationale are visible, and every routine update (call, note, stage, snooze, complete, email) is reachable from the item card. Whether this actually replaces the shadow spreadsheet or improves CRM hygiene remains a user-validation question.

## Users & jobs

**Primary user:** an individual contributor who owns a book of accounts end-to-end — Customer Success Manager / account executive hybrid — working a queue of 10–40 open commitments across deal, onboarding, support and follow-up work.

**Job to be done:** "When I start my day, I want one ranked list of the commitments that are due or slipping across all my accounts, so I can work top-down and record what happened without leaving the list or filling out a form."

Supporting jobs:
- "Show me why this is at the top so I can trust the order."
- "Let me reach the contact by email with the context already written."
- "Let me push something out of today without it disappearing."

**Secondary users (not designed for yet):** manager reviewing team load; ops owner configuring the priority rule.

### Confidence & validation status

- **High confidence — prototype behavior:** unified queue structure, visible rank/rationale, filtering, inline actions, session activity updates, deep-linking, and queue/email state propagation are directly demonstrated by the prototype.
- **Medium confidence — workflow fit:** the JTBD and hybrid CSM/AE persona are coherent with the prototype, but still need confirmation with real users and real workload sizes.
- **Lower confidence — business rules:** the ranking inputs, source of `committed`, snooze/completion semantics, email integration model, and final success metrics are intentionally unresolved and should not be treated as validated.


## Scope

**In scope**
- Single cross-object daily queue ranked deterministically by urgency bucket, committed date, then account value, with visible priority rationale.
- Counters for remaining / overdue / done.
- Search plus When, Work type and Stage filters in one filter panel, with clear-search and reset-all controls.
- Inline one-tap actions on each card: log call, add note, update stage, snooze (+1 / +3 / +7 days), complete.
- Session activity feed ("Just updated") confirming each action.
- Email screen: contact list, pre-filled composer driven by the selected queue item, template chips, thread view, sent log, `mailto:` handoff.
- Navigation between queue and email, including deep link `/email?item=<id>` that pre-selects and pre-fills.
- Mock CRM chrome: left sidebar and breadcrumb placing the queue inside a larger product.
- Loading state on every page and contact load; "No data" error state with retry.

**Out of scope (this phase)**
- Any persistence, database, authentication, or multi-user data. State is in-memory and resets on reload.
- Real email sending, inbox sync, reply capture, or open/click tracking.
- Real CRM object model: pipelines, ticket systems, custom fields, record detail pages.
- Configurable or ML-based prioritization; the ranking rule is fixed.
- Manager/rollup views, reporting, quotas, forecasting.
- Bulk actions, keyboard-driven queue navigation, notifications, mobile-native app.
- Edit of past activity entries.
- Production-grade undo semantics. The prototype may include a short-lived UI undo for Complete/Snooze only to test accidental-action recovery; persistence and conflict handling remain out of scope.

## Requirements

| # | Requirement | Priority | Acceptance criteria |
|---|---|---|---|
| R1 | Unified queue across deal / onboarding / support / follow-up work | Must | One list renders items of all four work types; each card shows contact, account, work-type label, stage, account value, committed-date label. |
| R2 | Deterministic prioritization | Must | Order is overdue → due today → upcoming; within a bucket, earlier committed date wins; exact-date ties break by higher account value. Re-running the same dataset produces the same order. Rank number is shown on each card. |
| R3 | Visible priority rationale | Must | Each card shows a "Why here" explanation that exposes the actual signals used by the current ranking rule: urgency/committed-date status and account value. The explanation must never imply that unmodeled signals such as SLA, renewal date or deal stage affected the rank. |
| R4 | Urgency signalling | Must | Overdue = red badge, due today = yellow, upcoming = neutral, completed = green and dimmed card. |
| R5 | Progress counters | Should | Header shows counts for items left, overdue, and done; counts update immediately after any action. |
| R6 | One-tap log call / add note | Must | Action opens an inline single optional text field; saving with an empty field still succeeds; card's last-touch text updates and an entry appears in the activity feed. No required fields anywhere. |
| R7 | Stage update from the card | Must | Stage options offered are only those valid for the item's work type; selecting one updates the card immediately. |
| R8 | Snooze | Should | Tomorrow / +3 days / +7 days shift the committed date; the item re-sorts into its new position and its urgency badge updates. |
| R9 | Complete | Must | Completing dims the card, marks it Done, hides its action row, decrements "left", and provides a short-lived UI undo so the interaction can be tested safely. Undo restores the prior in-memory state only. |
| R10 | Session activity feed | Could | Each action appends a timestamped line to a "Just updated" list visible on the queue screen for the session. |
| R11 | Search | Should | Free-text match over contact, account, next action and context; list filters as the user types. |
| R12 | Filters | Should | When / Work type / Stage filters combine with search; a count of active filters is shown. |
| R13 | Clear controls | Should | A clear button appears in the search field only when text is present; "Reset all" appears only when at least one filter or search is active and restores the unfiltered list. |
| R14 | Email from a queue item | Must | An Email action on each card navigates to `/email?item=<id>`; the target contact is selected and To / Subject / Body are pre-filled from that item's account, stage, next action and context. |
| R15 | Email templates | Could | Three chips (Next step, Gentle nudge, Book time) replace subject and body with item-aware copy. |
| R16 | Send behaviour | Should | Sending appends to the session Sent log and sets the queue item's last touch to "just now · email". No message leaves the app. |
| R17 | Mail-app handoff | Could | "Open in mail app" opens a `mailto:` link carrying the composed subject and body. |
| R18 | Thread context | Could | Selecting a contact shows prior messages for that contact so the screen reads as CRM email rather than a blank form. |
| R19 | Loading state | Should | Page load and contact switch show a spinner with a "Loading" label for roughly 0.6–0.9s before content renders. |
| R20 | Error state | Should | When a fetch fails, the region shows the "No data" state with a retry control; retry re-runs the fetch. |
| R21 | CRM placement | Could | Left sidebar names the product and lists Daily Action Queue and Email as active destinations plus non-interactive mock records; breadcrumb reads Northstar CRM / Workspace / Customer Success / Daily Action Queue. |
| R22 | Consistent design language | Should | Both screens use the same bordered blocks, uppercase display headings, mono labels and signal colors; no ad-hoc colors outside the token set. |

Priority key: Must = the hypothesis fails without it; Should = needed for a credible daily tool; Could = polish / realism.

## Data & events

**Entities (mocked, in-memory)**

`QueueItem` — id, contact, account, workType (deal | onboarding | support | followup), stage, value (number), committed (ISO date), nextAction, context, lastTouch, done. Seeded with representative records generated relative to today's date so urgency buckets are always populated. The validation dataset should include deliberate ranking conflicts (for example, low-value overdue work versus high-value upcoming work) so users can challenge the ordering rather than only see obvious cases.

`SentEmail` — id, itemId, to, contact, subject, body, timestamp. Session only.

Derived, not stored: rank, urgency bucket, committed-date label, priority rationale, email address (`first.last@account.com`), subject and body drafts. The current prototype does not need a separate weighted `priorityScore`; rank should be derived directly from the documented deterministic sort so the explanation and ordering cannot diverge.

**Events the prototype records (client-side only, rendered into the activity/sent feeds):**
- `call_logged` (itemId, optional note)
- `note_added` (itemId, optional note)
- `stage_changed` (itemId, fromStage, toStage)
- `item_snoozed` (itemId, days, newCommittedDate)
- `item_completed` (itemId)
- `email_composed_from_item` (itemId)
- `email_sent` (itemId, to, subject) — simulated
- `mailto_opened` (itemId)
- `filter_applied` / `filters_reset` / `search_changed`
- `fetch_started` / `fetch_failed` / `fetch_retried`

**Real vs. mocked**
- Real: all UI, routing, deep-linking with search params, sorting and filtering logic, state updates and their propagation between the queue and email screens via a shared in-memory store, `mailto:` handoff.
- Mocked: the dataset, email addresses and threads, the "fetch" itself (a timer with a manual failure switch in the sidebar), sending, the sidebar's non-queue records, and the awaiting-reply counter.
- Absent: persistence, auth, API, deliverability, real timestamps beyond the session.

**For a real build** these events would be emitted to an analytics sink and to the CRM activity timeline, and the documented deterministic priority ordering would be computed server-side so ordering is identical for every client.

## Prototype validation plan

The next iteration should increase confidence by testing the riskiest assumptions rather than expanding feature breadth. No result below should be marked validated until observed with representative users.

1. **Ranking trust test:** give users a realistic mixed queue with deliberately conflicting signals and ask what they would work first. Compare their ordering with the prototype and capture which signals they expected to matter.
2. **Shadow-spreadsheet replacement test:** ask users to work through a simulated morning using only the queue, then ask whether they would still maintain a separate list and why.
3. **Scale test:** test the queue with roughly 30–60 open items, including multiple overdue items, to check whether the ranking and filters still reduce rather than add cognitive load.
4. **Action-semantics test:** test wording for Snooze and Complete with explicit descriptions of what would change in the CRM. Distinguish "remove from today" from "change committed date" and "mark queue item done" from "close underlying object".
5. **Email workflow test:** compare in-product composition with a lightweight Gmail/Outlook handoff. Validate whether users need a full CRM email surface before committing to mailbox integration.
6. **Measurement baseline:** before claiming success, establish the current time spent building the daily priority list and the current rate of stale/no-touch records so the prototype has a meaningful before/after comparison.

### Proposed success criteria for validation

These are test targets, not established baselines:

- Users can identify their first action from the queue without reconstructing a separate list.
- Users can explain why the top-ranked items are above lower-ranked items using the visible rationale.
- Most routine updates can be recorded without leaving the queue.
- The queue remains understandable with a high-load dataset (30–60 open items).
- Users correctly predict what Snooze and Complete will change before confirming the action.
- Primary product metric candidate: reduction in time spent creating the daily priority list.
- Supporting metric candidates: percentage of queue items touched the same day; reduction in records with no logged touch in 7 days; percentage of test users who say they would no longer need a shadow spreadsheet for daily prioritization.

## Open questions

1. Priority rule: is committed date → account value correct, or should SLA breach, deal stage or renewal date outrank value? Should the weighting be configurable per team?
2. Queue length: should the queue be capped (a true "today" list) or show everything open? What happens when 60 items are due?
3. Where does `committed` come from in a real CRM — next-step due date, task due date, SLA target, or a new field the rep sets?
4. Snooze semantics: does snoozing move the underlying commitment date on the record, or only hide it from today's queue? Managers will care about the difference.
5. Completion semantics: does Complete close the underlying object (ticket, task, onboarding step) or only clear it from the queue?
6. Email: does this become a real send-and-sync mailbox integration (Gmail/Outlook), a send-only service, or stay a compose-and-handoff to the local mail client? Reply capture drives the "awaiting reply" counter.
7. Contact data: real email addresses, consent state and suppression rules must replace the derived addresses before any send is enabled.
8. Multi-user: is the queue strictly personal, or does a manager need visibility and reassignment?
9. Offline / stale data: what does the queue show if the sync fails mid-day — cached list or the "No data" state?
10. Undo: the prototype uses a short-lived in-memory undo for accidental Complete actions; a real build still needs a decision on persistence, audit history, and whether Snooze/Stage changes also require undo.
11. Success metrics: proposed primary is reduction in time spent creating the daily priority list; supporting candidates are percentage of queue items touched the same day, reduction in items with no logged touch in 7 days, and whether users still need a shadow spreadsheet. Baselines and target thresholds still need agreement before instrumentation.
