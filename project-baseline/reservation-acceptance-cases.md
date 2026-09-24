# Reservation Acceptance Cases

**Status:** Workshop 2 proposal for review  
**Scenario catalog version:** `reservation-acceptance-v1`  
**Depends on:** `mock-restaurant-spec.md`, `reservation-domain-design.md`, `reservation-contracts.md`

## Test strategy

The reservation feature is accepted on business outcomes, not on whether an LLM produced a plausible sentence.

Each test runs from a deterministic reset and asserts:

1. Initial clock, configuration, inventory, holds, reservations, and operational blocks.
2. Customer/staff action sequence.
3. Operation results and stable error codes.
4. Durable aggregate and allocation state.
5. Customer-visible response.
6. Staff-dashboard projection.
7. Domain and audit events.
8. Absence of duplicate or unauthorized side effects.

## Fixed environment

| Setting | Value |
|---|---|
| Restaurant | Juniper & Stone |
| Restaurant ID | `rst_juniper_stone` |
| Location ID | `loc_oakland_001` |
| Time zone | `America/Los_Angeles` |
| Default frozen time | Wednesday, October 14, 2026 at 12:00:00 PM PDT |
| Default UTC time | `2026-10-14T19:00:00Z` |
| Default configuration | `mock-restaurant-v1` |
| External notifications | In-app preview only |
| LLM/voice | Deterministic fake unless a case explicitly requests a manual provider smoke test |

All names, phone numbers, email addresses, transcripts, and references are synthetic.

## Base seed

- Weekly and exceptional schedules from `mock-restaurant-spec.md`.
- All dining areas open.
- All resources active and unblocked.
- No active holds.
- No reservations unless a scenario adds them.
- Staff presence offline unless a scenario changes it.
- Customer A: Avery Nguyen, `+1 510-555-0163`, `avery.nguyen@example.com`.
- Customer B: Jordan Lee, `+1 510-555-0174`, `jordan.lee@example.com`.
- Manager: Morgan Reyes.
- Staff: Casey Patel.

Fixture IDs are stable inside tests but remain opaque in customer responses.

## Allocation assertions

With empty inventory:

- Party 2, main dining: M02 is selected because M01 is preserved as accessible.
- Party 4, main dining: M06 is selected because M05 is preserved as accessible.
- Party 6, main dining: M10 is selected because M09 is preserved as accessible.
- Party 4 requiring accessible main seating: M05 is selected.
- Party 2, patio: P02 is selected because P01 is preserved as accessible.

These are internal assertions only. Customer output states dining area, never resource ID.

## Core booking cases

### R-001: Website form creates one reservation

**Seed delta:** None.

**Actions:**

1. Search Friday, October 16, 2026 at 7:00 PM for four in main dining.
2. Select the exact option and hold it.
3. Enter Customer A details.
4. Prepare and render the summary.
5. Click the current summary's Confirm button.

**Expected state:**

- One converted hold.
- One confirmed reservation, version 1, source `WEB_FORM`.
- Internal assignment M06 from 7:00 PM to 9:00 PM local, including reset buffer.
- One hashed manage token; raw token returned once to the browser only.
- No second reservation on page refresh.

**Customer response:** Full date/time, party four, main dining, reference, policy, and manage link.

**Staff view:** Four covers at 7:00 PM in main dining; source Website; no physical table promised publicly.

**Events:** `hold.created`, `hold.converted`, `reservation.created`; matching audit records.

### R-002: Text agent uses the same booking path

**Seed delta:** None.

**Actions:**

1. Customer says, "Patio for two this Friday around 7."
2. Fake model resolves Friday, October 16 and calls search.
3. Agent displays exact and nearby patio options.
4. Customer selects 7:00 PM, supplies Customer B details, reviews summary, then sends a later "Yes, book it."

**Expected state:**

- One confirmed reservation, source `WEB_TEXT`.
- Internal assignment P02.
- The finalized confirmation turn is later than the prepared summary turn.

**Customer response:** Structured confirmation generated from tool result, not model memory.

**Staff view/events:** Equivalent to R-001 except source and guest.

### R-003: Voice correction before confirmation

**Seed delta:** None.

**Scripted finalized input turns:**

1. "A table for two Saturday at eleven for Avery."
2. "My number is 510-555-0168."
3. "Correction, the last digit is three."
4. After readback: "Confirm the reservation."

**Expected behavior:**

- Date resolves to Saturday, October 17, 2026; service is brunch.
- Phone normalizes to `+15105550163`.
- The correction supersedes the earlier contact detail and creates a new summary version.
- Confirmation references the final summary version and later finalized turn.
- One confirmed reservation at 11:00 AM, source `WEB_VOICE`.
- Raw audio is not retained; finalized transcripts and tool audit are retained.

**Customer response:** Reads back Saturday, October 17, 2026, 11:00 AM Pacific time, two guests, area, and masked phone before confirmation.

### R-004: Ambiguous voice acknowledgment does not book

**Seed delta:** Active hold and prepared voice summary for Customer A.

**Finalized input:** "Okay."

**Expected state:**

- `confirmReservation` returns `CONFIRMATION_AMBIGUOUS`, or the orchestrator asks for a clearer phrase without calling confirm.
- Hold remains active until expiry.
- No reservation or `reservation.created` event exists.

**Customer response:** "Please say 'Confirm the reservation' or use the Confirm button."

### R-005: Same-turn confirmation is rejected

**Seed delta:** None.

**Action:** A model attempts to prepare and confirm using the same customer turn, "Book Friday at seven for four, yes confirm."

**Expected state:**

- Prepare may succeed after a hold, but confirm returns `CONFIRMATION_TOO_EARLY`.
- No reservation exists.

**Customer response:** The exact summary is shown and a new confirmation is requested.

## Availability and concurrency

### R-006: Exact time unavailable; bounded alternatives returned

**Seed delta:** All compatible patio resources allocated at 7:00 PM Friday, October 16, but P02 is free at 7:30 PM.

**Action:** Search patio-required, party two, preferred 7:00 PM, flex 45 minutes.

**Expected state:** No writes.

**Result:**

- No 7:00 PM option.
- 7:30 PM patio option returned.
- Main-room options may appear only under `alternate_area_options`, clearly labeled.
- Party size/date/accessibility are not changed.

### R-007: Two customers race for the last compatible assignment

**Seed delta:** For Friday at 7:00 PM, block every main-dining resource except M06 for the candidate occupied interval.

**Actions:**

1. Sessions A and B both search and receive 7:00 PM.
2. Both call `holdSlot` concurrently with unique idempotency keys.

**Expected state:**

- Exactly one active hold owns M06.
- The loser receives `SLOT_STALE` and must search again.
- No overlapping allocation exists.
- No reservation exists until the winning session confirms.

**Events:** Exactly one `hold.created`.

### R-008: Hold expires before confirmation

**Actions:**

1. Hold a slot at 12:00:00 PM; expiry is 12:07:00 PM.
2. Prepare reservation.
3. Advance server clock to 12:07:01 PM.
4. Confirm.

**Expected state:**

- Confirm returns `HOLD_EXPIRED` or `PENDING_ACTION_EXPIRED`.
- Hold is effectively expired even if cleanup has not run.
- No reservation exists.
- Slot can be held by another session.

### R-009: Replacing a hold does not lose the old hold on failure

**Seed delta:** Customer A holds 7:00 PM; 7:30 PM appears in a stale search result but another session acquires it before replacement.

**Action:** Call `holdSlot` for 7:30 PM with `replace_hold_id` pointing at the 7:00 PM hold.

**Expected state:**

- New hold fails with `SLOT_STALE`.
- Original 7:00 PM hold remains active until its original expiry.
- No release event for the original hold.

### R-010: Idempotent retry after response loss

**Actions:**

1. Confirm with idempotency key K.
2. Commit succeeds, but simulate connection loss before response reaches client.
3. Retry identical command with K.

**Expected state:**

- Retry returns the original reservation ID/reference.
- Exactly one reservation, assignment, converted hold, and `reservation.created` event exist.
- No duplicate notification preview.

### R-011: Idempotency key reused with changed payload

**Action:** Reuse hold or confirm key K with a different slot, pending-action version, or guest payload.

**Expected result:** `IDEMPOTENCY_KEY_REUSED`.

**Expected state:** Original command result remains unchanged; no second side effect.

## Modification and cancellation

### R-012: Atomic time modification succeeds

**Seed delta:** Customer A has a confirmed Friday 7:00 PM party-four reservation assigned to M06, version 1. M07 is available at 8:00 PM.

**Actions:**

1. Authorized customer searches change availability.
2. Holds 8:00 PM replacement assignment.
3. Reviews before/after summary.
4. Confirms in a later event.

**Expected state:**

- Same reservation ID, version 2, now 8:00 PM.
- New assignment active; old M06 interval released in the same transaction.
- Replacement hold converted.
- Status remains `CONFIRMED`, not `MODIFIED`.

**Events:** One `reservation.modified` with before/after safe fields.

### R-013: Failed modification preserves original

**Seed delta:** Same original as R-012. Replacement slot becomes unavailable before hold.

**Action:** Attempt replacement.

**Expected result:** `SLOT_STALE`.

**Expected state:** Reservation remains version 1 at 7:00 PM with M06; no modified event.

### R-014: Customer change inside cutoff routes to staff

**Seed delta:** Friday 7:00 PM reservation; freeze time Friday at 5:30 PM.

**Action:** Customer requests 8:00 PM.

**Expected result:** `CHANGE_REQUIRES_STAFF`.

**Expected state:** Original unchanged; handoff created if staff is offline or live transfer offered if online.

### R-015: Cancellation outside cutoff

**Seed delta:** Confirmed Friday 7:00 PM reservation; default Wednesday clock.

**Actions:** Prepare cancellation, show policy, later confirm.

**Expected state:**

- Reservation `CANCELLED`, version increments.
- `late_cancellation = false`.
- Inventory released.
- Repeating confirmation is an idempotent success.

### R-016: Late cancellation accepted and tagged

**Seed delta:** Confirmed Friday 7:00 PM reservation; freeze Friday at 6:00 PM.

**Actions:** Prepare and explicitly confirm cancellation.

**Expected state:**

- Reservation cancelled and inventory released.
- `late_cancellation = true`.
- Staff view and event include the late flag.
- No fee is charged.

### R-017: Optimistic version conflict

**Seed delta:** Customer loads reservation version 1; staff changes note/time and commits version 2.

**Action:** Customer submits a change with `expected_version = 1`.

**Expected result:** `RESERVATION_VERSION_CONFLICT` and safe current version 2 projection.

**Expected state:** Staff change remains; customer change is not applied.

## Policy, calendar, and accessibility

### R-018: Large party routes to staff

**Action:** Search for nine guests.

**Expected result:** `PARTY_SIZE_REQUIRES_STAFF`.

**Expected state:** No search option, hold, or reservation. Live transfer is offered when staff is online; otherwise one ticket is created.

### R-019: Closed holiday cannot be booked

**Action:** Search Thursday, November 26, 2026 at 7:00 PM.

**Expected result:** `SERVICE_CLOSED` with Thanksgiving closure message.

**Expected state:** No writes, even if weekly Thursday dinner would normally exist.

### R-020: Exceptional Christmas Eve hours override weekly hours

**Actions:**

- Search Thursday, December 24, 2026 at 6:00 PM: option may be returned.
- Search 7:00 PM: no option because last seating is 6:30 PM.

**Expected state:** No write from search; exception version appears in internal trace.

### R-021: Booking horizon and lead time

**Actions:**

- Search 61 local calendar days beyond frozen now.
- Freeze at 6:30 PM and search same-day 7:00 PM.

**Expected results:** `OUTSIDE_BOOKING_HORIZON`; `LEAD_TIME_NOT_MET`.

**Expected state:** No writes.

### R-022: Accessibility is a hard constraint

**Actions:**

1. Search main dining for four with wheelchair space required.
2. With M05 available, select and hold.
3. Repeat after M05 is blocked for that interval.

**Expected results:**

- First search uses internal assignment M05.
- Second search does not silently use M06-M08; returns no main option and may show an explicitly accessible patio alternative if P05 is open.

### R-023: Daylight-saving date resolution

**Seed delta:** Freeze Saturday, October 31, 2026 at 11:30 PM PDT.

**Input:** "Tomorrow at noon for two."

**Expected behavior:**

- Resolves to Sunday, November 1, 2026 at 12:00 PM PST in `America/Los_Angeles`.
- Summary includes full date and Pacific time.
- Stored UTC instant reflects the post-transition offset.
- No ambiguous early-morning time reaches the domain service.

## Staff and operational conflicts

### R-024: Normal staff block conflicts with active hold

**Seed delta:** Customer holds M06 for Friday at 7:00 PM.

**Action:** Staff attempts a normal M06 block covering that interval.

**Expected result:** `RESOURCE_HELD`.

**Expected state:** Hold remains active; no block created.

### R-025: Manager forced block invalidates hold

**Seed delta:** Same as R-024.

**Action:** Manager forces emergency-maintenance block with a reason.

**Expected state:**

- Block created.
- Hold status `INVALIDATED`.
- Pending action expires/supersedes.
- Customer session receives a slot-lost event and cannot confirm.
- Audit identifies manager and reason.

### R-026: Patio closes with confirmed reservations

**Seed delta:** Two confirmed patio reservations for Friday dinner.

**Action:** Manager sets patio `closed_weather`.

**Expected state:**

- Existing reservations remain confirmed and are not silently moved.
- Both enter staff relocation/attention queue.
- New patio searches return no options.
- Customers see no cancellation until staff performs an explicit action.

### R-027: Staff walk-in uses collision protection

**Action:** Staff creates and checks in a party of two as a walk-in while a public hold competes for the last compatible table.

**Expected state:**

- Exactly one assignment wins transactionally.
- If walk-in wins, it begins `CHECKED_IN`, source `WALK_IN`, with creation and arrival events.
- Public customer receives a stale/conflict result, not a false hold.

### R-028: No-show timing

**Seed delta:** Confirmed 7:00 PM reservation.

**Actions:**

- At 7:14 PM, staff attempts no-show.
- At 7:16 PM, staff retries.

**Expected results:** First returns `INVALID_STATE`/policy timing error; second succeeds.

**Expected state:** Reservation `NO_SHOW`, inventory released, staff event attributed.

## Security and privacy

### R-029: Reference number cannot authorize management

**Action:** Anonymous session calls get/cancel using only `JS-482731`.

**Expected result:** `RESERVATION_NOT_FOUND` or `AUTHENTICATION_REQUIRED`.

**Expected state:** No data leakage or mutation.

### R-030: Manage token never enters model context

**Action:** Create through text/voice agent and inspect tool result passed back to the model, application logs, audit, and analytics fixture.

**Expected state:**

- Raw manage token appears only in the secure customer transport result.
- Model-safe tool result contains reference and public reservation, not token.
- Persistence stores only token hash.

### R-031: Cross-tenant identifier is ignored/rejected

**Action:** Model/client payload attempts to set another restaurant/location ID.

**Expected result:** Server-injected scope wins or request returns `FORBIDDEN`.

**Expected state:** No read/write outside current location.

### R-032: Allergy note creates attention, not a safety claim

**Action:** Customer adds "severe sesame allergy."

**Expected state:**

- Note stored under restricted access.
- Reservation has `needs_staff_attention`.
- Agent returns approved cross-contact disclaimer and offers staff.
- No "safe" assertion appears.

## Dependency and channel failures

### R-033: Text/voice model unavailable

**Action:** Simulate provider timeout/rate limit before a reservation write.

**Expected behavior:**

- Structured form remains usable.
- Conversation state and active hold remain visible until expiry.
- UI offers text/form fallback and does not claim success.

**Expected state:** No extra reservation.

### R-034: Voice WebSocket drops after summary

**Action:** Drop voice connection after pending summary but before confirmation.

**Expected behavior:**

- Transcript and pending summary remain in the web session.
- Customer may use the visible Confirm button or text before expiry.
- Reconnecting voice does not count as confirmation.

### R-035: Database unavailable during confirmation

**Action:** Inject database failure before commit.

**Expected result:** `DATABASE_UNAVAILABLE`.

**Expected state:** No reservation, reference, converted hold, or success event. A retry with the same key can safely determine outcome after recovery.

### R-036: Notification preview fails after commit

**Action:** Commit reservation, then fail notification-preview worker.

**Expected behavior:**

- Customer still sees in-app confirmation and reference.
- Reservation remains confirmed.
- Notification job retries independently and is visibly failed to staff if retries exhaust.

**Events:** One reservation creation; notification failure/retry events, no duplicate reservation.

## Browser and accessibility cases

### R-037: Microphone permission denied

**Expected behavior:** Voice mode explains permission state, never loops permission prompts, and preserves full text/form access.

### R-038: Speech recognition fallback unsupported

**Expected behavior:** Feature detection hides unsupported browser-recognition controls; Gemini Live or manual text remains available according to configuration.

### R-039: Keyboard-only booking

**Expected behavior:** Search, option selection, hold countdown announcement, validation, summary, confirmation, and result are operable and screen-reader labeled without a pointer or voice.

### R-040: Hold countdown accessibility

**Expected behavior:** Countdown is visible but does not announce every second. It announces meaningful thresholds and expiration without trapping focus.

### R-041: One cross-channel hold per customer session

**Actions:**

1. Customer session holds Friday 7:00 PM through the conventional form.
2. The same session asks the text agent to hold Friday 7:30 PM without identifying the first hold.
3. The agent then retries with the first hold as `replace_hold_id`.

**Expected behavior:**

- Step 2 returns `ACTIVE_HOLD_EXISTS` and shows the current held option; it does not hoard a second table.
- Step 3 creates the 7:30 PM hold before releasing the 7:00 PM hold.
- If replacement fails, the original hold remains active.
- Form, text, and voice all display the same current hold.

### R-042: Expired search token requires a new search

**Actions:**

1. Search at 12:00 PM and receive a slot token expiring at 12:05 PM.
2. Advance authoritative time to 12:05:01 PM.
3. Attempt `holdSlot`.

**Expected result:** `SLOT_TOKEN_EXPIRED`.

**Expected state:** No hold or allocation is created; the customer receives refreshed options.

### R-043: Bar party requires adjacent seats

**Seed delta:** At Friday 7:00 PM, only B01 and B05 are free; at 7:30 PM, B03 and B04 are free.

**Action:** Search bar-required seating for two around 7:00 PM with 45-minute flexibility.

**Expected behavior:**

- No 7:00 PM option is returned because B01 and B05 are not adjacent.
- A 7:30 PM option is returned using internal contiguous assignment B03+B04.
- Individual bar resource IDs remain hidden from the customer.

## Manual model-provider smoke tests

These do not replace deterministic tests. The required path uses the local
Ollama adapter and synthetic data; an unpaid hosted adapter may be compared
under the
[Zero-Cost Tooling Policy](../free-tooling-policy.md):

- The configured interactive text model chooses the correct allowlisted tool
  for ten representative utterances.
- An optional voice adapter returns input/output transcripts and handles
  interruption; voice is not a core-track graduation requirement.
- Voice reads numeric dates, times, party sizes, and phone digits accurately.
- Tool calls carry no tenant override or unsupported properties.
- Rate-limit and disconnect behavior falls back safely.
- No test uses a real person's contact information.

Model wording may vary. Acceptance is based on tool choice, validated
arguments, state transition, and customer-safe outcome. Hosted-provider
availability never blocks the deterministic or local acceptance path.

## Coverage matrix

| Layer | Primary coverage |
|---|---|
| Domain unit | Time rules, candidate generation, scoring, state transitions, policy |
| Persistence concurrency | Resource overlap, holds, conversion, idempotency, version conflicts |
| Contract | Strict schemas, auth context, safe projections, error taxonomy |
| API integration | Full create/change/cancel, staff lifecycle, failures |
| Browser end-to-end | Form, text UI, voice UI states, accessibility, dashboard consistency |
| Model evaluation | Intent/tool selection and critical-slot extraction with synthetic transcripts |
| Manual provider smoke | Real hosted text/audio behavior and limits |

## Exit criteria

Workshop design is implementation-ready only when:

- Every scenario has an owner test layer.
- R-007, R-008, R-010, R-012, R-013, R-025, R-030, and R-035 are mandatory blocking tests.
- No success case relies on matching exact LLM prose.
- Concurrency tests run repeatedly and never produce overlapping allocations.
- All three customer modalities produce the same reservation state for equivalent input.
- Failure cases never produce a false confirmation.
- Reviewers approve the proposed restaurant rules, state model, contracts, and error codes.
