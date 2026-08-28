# Full-Stack: Data, Access Rules, Edge Cases, Deploy

> Module 5 · Full-Stack. Add data schemas, access rules, and edge cases; stress-test and deploy.


## Deployed link

https://crm-northstar.lovable.app

_____


## Data schema

| Entity | Key fields | Notes |
|---|---|---|
| `profiles` | `id` (PK, → auth.users), `display_name`, `email`, `created_at`, `updated_at` | Auto-created by `handle_new_user()` trigger on signup. One row per user. |
| `accounts` | `id`, `owner_id`, `name`, `value` (numeric), timestamps | A company/deal account. Seeded with 12 demo accounts per user. |
| `contacts` | `id`, `owner_id`, `account_id` (nullable), `name`, `email`, `role`, timestamps | Partial unique index on `(owner_id, lower(email))` where email is not null — no duplicate contact emails per user, but email-less contacts are allowed. |
| `queue_items` | `id`, `owner_id`, `account_id` (nullable), `contact_id` (nullable), `contact_name`, `account_name`, `work_type` (enum: deal/onboarding/support/followup), `stage`, `value`, `committed_on` (date), `next_action`, `context`, `last_touch`, `done`, `snoozed_until` (date, nullable), `completed_at` (timestamptz, nullable), `sort_rank` (numeric, nullable), `updated_at` | The core entity. `snoozed_until` never overwrites `committed_on` (original commitment is preserved). `updated_at` doubles as an optimistic-concurrency token. Indexed on `(owner_id, done, committed_on)`. |
| `sent_emails` | `id`, `owner_id`, `queue_item_id` (nullable), `to_email`, `contact_name`, `subject`, `body`, `sent_at` | Log of every email "sent" from the composer (simulated send, real persistence). Indexed on `(owner_id, sent_at desc)`. |
| `activity_events` | `id`, `owner_id`, `queue_item_id` (nullable), `kind` (enum: call/note/stage/snooze/complete/email), `detail`, `created_at` | Audit trail written for every inline action. Indexed on `(owner_id, created_at desc)`. |

Supporting objects: enum types `work_type` and `activity_kind`; `seed_demo_queue(p_owner uuid)` (security-definer, service-role-only, idempotent) inserts the 12 demo accounts, contacts and queue items for a new user; `update_updated_at_column()` trigger keeps `updated_at` fresh on every write.

## Access rules

_Who can see / do what? Where are the auth boundaries?_

- **Auth:** Public `/auth` route with email/password and Google sign-in. The `/` (Daily Action Queue) and `/email` screens live under a protected `_authenticated` layout — no session, redirect to `/auth`.
- **Data isolation:** Every row carries `owner_id`. RLS is enabled on all tables with a single policy per table: `USING (auth.uid() = owner_id) / WITH CHECK (auth.uid() = owner_id)`. Users can only ever read or write their own rows — verified live with a second account that saw only its own data.
- **Server boundary:** All reads/writes go through `createServerFn` functions (`listQueueItems`, `updateQueueItem`, `logActivity`, `listSentEmails`, `sendEmail`) guarded by `requireSupabaseAuth` middleware. `owner_id` is always set from the verified session on the server — never accepted from client input.
- **Profiles:** any authenticated user can read profiles; only the owner can update/delete their own.
- **Seeding:** `seed_demo_queue` is executable only by the service role, invoked once per user on first sign-in; it is idempotent (skips owners that already have queue items).
- **Demo vs real:** data, auth, persistence and activity logging are real; the actual email delivery is simulated (emails are recorded in `sent_emails`, not sent over SMTP).

## Edge cases hardened

| Case | Before | After |
|---|---|---|
| Empty / first-run state | Placeholder mock data always visible; no real "empty" concept | First sign-in auto-seeds 12 demo items; when all items are done the queue shows a distinct "Queue is clear" success state; fetch errors show a "No data" screen with a working retry; pending loads show a skeleton. |
| Bad / malicious input | Client-trusted payloads | All server functions validate input and derive `owner_id` from the session; partial unique index blocks duplicate contact emails per owner; RLS rejects cross-owner reads/writes; contacts with no email get a disabled "No email" composer state instead of a broken send. |
| Failure / offline | Controllable mock failure toggle, no recovery | Real query state: expired session waits for token hydration before calling the server; offline detected with a toast; optimistic actions roll back on failure with an error toast; concurrent edits in two tabs are caught by an `updated_at` optimistic-concurrency check — the losing tab re-syncs and retries once, and only a genuine second conflict surfaces a "changed in another tab" toast. |

## Stress test results

- Rapid back-to-back edits (per-item mutation queue + conflict retry)
- Send-email → follow-up edit (stale-token conflict eliminated)
- Unlinked accounts / missing emails
- Offline and expired-session recovery
- All authenticated users were able to read the data of other users
- It hanged while signing in with Google

_____

