# Reservation Contracts and Error Taxonomy

**Status:** Workshop 2 proposal for review  
**Contract family:** `restaurant.reservations.v1`  
**Depends on:** `mock-restaurant-spec.md`, `reservation-domain-design.md`

## Contract principles

1. Application contracts are channel-neutral. Website forms, text, voice, and staff clients do not receive separate business APIs.
2. The server injects tenant, location, actor, authorization, and current-time context. The model cannot choose its tenant or permissions.
3. JSON schemas are strict: bounded strings/arrays, explicit enums, required properties, and `additionalProperties: false`.
4. Queries are side-effect free.
5. Every command declares its idempotency and concurrency behavior.
6. Stable machine error codes drive workflow; human messages are display text, not branching keys.
7. Public results exclude physical table IDs, internal notes, manage-token hashes, and other customers' data.
8. Critical summaries are rendered from structured fields by trusted application code.

## Common context

The transport authenticates a caller, and the application constructs this context. It is not accepted from an LLM tool payload.

```json
{
  "schema_version": "restaurant.reservations.v1",
  "restaurant_id": "rst_juniper_stone",
  "location_id": "loc_oakland_001",
  "request_id": "req_...",
  "correlation_id": "corr_...",
  "actor": {
    "type": "CUSTOMER_SESSION",
    "id": "ses_...",
    "scopes": ["reservation:search", "reservation:create"]
  },
  "source": "WEB_VOICE",
  "locale": "en-US",
  "time_zone": "America/Los_Angeles",
  "server_now": "2026-10-14T19:00:00Z"
}
```

Allowed `source` values:

- `WEB_FORM`
- `WEB_TEXT`
- `WEB_VOICE`
- `STAFF`
- `WALK_IN`
- `SYSTEM`
- Future adapters only after explicit registration

## Common result envelope

Success:

```json
{
  "ok": true,
  "data": {},
  "meta": {
    "request_id": "req_...",
    "correlation_id": "corr_...",
    "occurred_at": "2026-10-14T19:00:00Z",
    "schema_version": "restaurant.reservations.v1"
  }
}
```

Failure:

```json
{
  "ok": false,
  "error": {
    "code": "HOLD_EXPIRED",
    "message": "That table option expired. Please choose a new time.",
    "retryable": false,
    "field_errors": [],
    "next_action": "SEARCH_AGAIN",
    "safe_context": {
      "service_date": "2026-10-16"
    }
  },
  "meta": {
    "request_id": "req_...",
    "correlation_id": "corr_...",
    "occurred_at": "2026-10-14T19:08:01Z",
    "schema_version": "restaurant.reservations.v1"
  }
}
```

`safe_context` is allowlisted per error. It never includes another guest's data, physical resource IDs, internal stack traces, or secrets.

## Shared value objects

### Local seating time

```json
{
  "service_date": "2026-10-16",
  "local_start_time": "19:00",
  "time_zone": "America/Los_Angeles",
  "utc_start": "2026-10-17T02:00:00Z"
}
```

Clients send local date/time plus time-zone ID. The server validates and computes `utc_start`; clients do not supply authoritative UTC.

### Area preference

```json
{
  "area": "PATIO",
  "required": true
}
```

Allowed areas for Juniper & Stone: `ANY`, `MAIN_DINING`, `PATIO`, `BAR`.

- `required: true`: do not return another area as equivalent; alternatives may be shown separately.
- `required: false`: area is a preference and the response labels the actual area.

### Accessibility needs

```json
{
  "step_free_table": true,
  "wheelchair_space": true
}
```

Accessibility constraints are never downgraded to soft preferences.

### Guest contact

```json
{
  "first_name": "Avery",
  "last_name": "Nguyen",
  "phone_e164": "+15105550163",
  "email": "avery.nguyen@example.com"
}
```

Validation:

- Names: trimmed, 1-80 Unicode characters each.
- Phone: normalized E.164, required.
- Email: optional, normalized, maximum 254 characters.
- Model-facing errors identify the invalid field but do not echo the complete contact value.

### Special requests

```json
{
  "accessibility": {
    "step_free_table": true,
    "wheelchair_space": true
  },
  "high_chair_count": 0,
  "occasion": "BIRTHDAY",
  "dietary_note": "Guest reports a sesame allergy; staff follow-up required.",
  "guest_note": "Quiet table if possible."
}
```

Limits:

- High chairs: 0-4 and cannot exceed covers.
- Free-text note fields: 500 characters each.
- Allergy/dietary notes automatically set `needs_staff_attention`; they never result in a safety guarantee.

## Public reservation representation

```json
{
  "reservation_id": "res_...",
  "reference": "JS-482731",
  "status": "CONFIRMED",
  "version": 1,
  "seating": {
    "service_date": "2026-10-16",
    "local_start_time": "19:00",
    "time_zone": "America/Los_Angeles",
    "display": "Friday, October 16, 2026 at 7:00 PM Pacific time",
    "party_size": 4,
    "area": "MAIN_DINING",
    "expected_dining_minutes": 105
  },
  "guest": {
    "display_name": "Avery Nguyen",
    "masked_phone": "+1 *** *** 0163",
    "masked_email": "a****@example.com"
  },
  "special_requests": {
    "high_chair_count": 0,
    "occasion": "BIRTHDAY",
    "has_dietary_note": true,
    "guest_note": "Quiet table if possible."
  },
  "policy": {
    "arrival_grace_minutes": 15,
    "customer_change_cutoff_minutes": 120,
    "exact_table_guaranteed": false
  },
  "capabilities": {
    "can_modify": true,
    "can_cancel": true,
    "modification_requires_staff": false
  },
  "created_at": "2026-10-14T19:05:10Z"
}
```

Initial creation additionally returns a one-time `manage_token` or manage URL to the customer transport. That secret is excluded from model messages, tool logs, analytics, staff UI copy actions, and subsequent representations.

## Operation catalog

| Operation | Kind | Model-callable | Idempotency |
|---|---|---|---|
| `searchAvailability` | Query | Yes | Not required |
| `holdSlot` | Command | Yes | Required |
| `releaseHold` | Command | Yes | Required |
| `prepareReservation` | Command | Yes | Required |
| `confirmReservation` | Command | Yes | Required |
| `getReservation` | Query | Yes, own reservation only | Not required |
| `searchChangeAvailability` | Query | Yes, authorized reservation | Not required |
| `prepareReservationChange` | Command | Yes | Required |
| `confirmReservationChange` | Command | Yes | Required |
| `prepareReservationCancellation` | Command | Yes | Required |
| `confirmReservationCancellation` | Command | Yes | Required |
| `recordArrival` | Command | Staff only | Required |
| `seatReservation` | Command | Staff only | Required |
| `completeReservation` | Command | Staff only | Required |
| `markNoShow` | Command | Staff only | Required |

The model sees only operations allowed for the authenticated current workflow. Staff operations are never included in customer model tool definitions.

## `searchAvailability`

### Input

```json
{
  "service_date": "2026-10-16",
  "party_size": 4,
  "preferred_local_time": "19:00",
  "flex_minutes": 45,
  "area_preference": {
    "area": "PATIO",
    "required": true
  },
  "accessibility": {
    "step_free_table": false,
    "wheelchair_space": false
  }
}
```

Rules:

- `service_date` is required and must be a valid local calendar date.
- `party_size` is integer 1-8 for public search.
- `preferred_local_time` is required and aligned/normalized to a service's slot interval.
- `flex_minutes` is one of 0, 15, 30, or 45.
- `area_preference` and `accessibility` are required objects with explicit defaults.

### Output

```json
{
  "query_id": "avq_...",
  "requested": {
    "service_date": "2026-10-16",
    "party_size": 4,
    "preferred_local_time": "19:00",
    "area": "PATIO"
  },
  "options": [
    {
      "option_id": "avo_...",
      "slot_token": "opaque_signed_value",
      "service_date": "2026-10-16",
      "local_start_time": "19:15",
      "time_zone": "America/Los_Angeles",
      "display": "Friday, October 16 at 7:15 PM",
      "area": "PATIO",
      "party_size": 4,
      "expected_dining_minutes": 105,
      "preference_match": true,
      "token_expires_at": "2026-10-14T19:05:00Z"
    }
  ],
  "alternate_area_options": [],
  "notices": [
    "Patio seating may change because of weather or air quality."
  ]
}
```

No physical resource ID is returned. Search tokens are valid for five minutes. Token expiration only limits use of the search snapshot; it is not a hold expiration.

## `holdSlot`

### Input

```json
{
  "slot_token": "opaque_signed_value",
  "replace_hold_id": null,
  "idempotency_key": "idem_..."
}
```

### Output

```json
{
  "hold_id": "hld_...",
  "status": "ACTIVE",
  "expires_at": "2026-10-14T19:09:00Z",
  "selection": {
    "service_date": "2026-10-16",
    "local_start_time": "19:15",
    "time_zone": "America/Los_Angeles",
    "display": "Friday, October 16, 2026 at 7:15 PM Pacific time",
    "party_size": 4,
    "area": "PATIO",
    "expected_dining_minutes": 105
  }
}
```

Semantics:

- Revalidates and locks inventory atomically.
- A customer session may own one active hold per location across form, text, and voice. Without a matching `replace_hold_id`, another hold returns `ACTIVE_HOLD_EXISTS`.
- If replacing a hold, creates the new hold before releasing the old one.
- Same key/same payload returns the same hold.
- Same key/different payload returns `IDEMPOTENCY_KEY_REUSED`.

## `releaseHold`

Input:

```json
{
  "hold_id": "hld_...",
  "idempotency_key": "idem_..."
}
```

Output:

```json
{
  "hold_id": "hld_...",
  "status": "RELEASED"
}
```

Releasing an already released/expired hold is a successful idempotent no-op when ownership matches. It never releases a converted hold's reservation.

## `prepareReservation`

### Input

```json
{
  "hold_id": "hld_...",
  "guest": {
    "first_name": "Avery",
    "last_name": "Nguyen",
    "phone_e164": "+15105550163",
    "email": "avery.nguyen@example.com"
  },
  "special_requests": {
    "accessibility": {
      "step_free_table": false,
      "wheelchair_space": false
    },
    "high_chair_count": 0,
    "occasion": "BIRTHDAY",
    "dietary_note": "",
    "guest_note": "Quiet table if possible."
  },
  "idempotency_key": "idem_..."
}
```

### Output

```json
{
  "pending_action_id": "pnd_...",
  "summary_version": 1,
  "summary_presented_event_id": "pres_...",
  "summary_presented_sequence": 17,
  "expires_at": "2026-10-14T19:09:00Z",
  "summary": {
    "action": "CREATE_RESERVATION",
    "display_lines": [
      "Friday, October 16, 2026 at 7:15 PM Pacific time",
      "4 guests, patio",
      "Avery Nguyen, +1 *** *** 0163",
      "Your table is held until 12:09 PM."
    ],
    "policy_statements": [
      "We hold reservations for 15 minutes after the booked time.",
      "Patio seating may change because of weather or air quality.",
      "Your exact table is not guaranteed."
    ],
    "confirmation_prompt": "Would you like me to confirm this reservation?"
  },
  "requires_separate_confirmation": true
}
```

The application, not the model, builds `display_lines` and policy statements. The channel gateway records the trusted `summary_presented` event as it emits the structured summary; confirmation is not valid before that event.

## Finalized conversation turn

Before text/voice confirmation, the channel stores:

```json
{
  "turn_id": "turn_...",
  "session_id": "ses_...",
  "channel": "WEB_VOICE",
  "sequence": 18,
  "speaker": "CUSTOMER",
  "final": true,
  "normalized_text": "confirm the reservation",
  "occurred_at": "2026-10-14T19:06:02Z",
  "content_hash": "sha256:..."
}
```

The confirmation operation accepts only the `turn_id`. It reloads the turn and validates ownership, finality, timing, and explicit intent. The turn must belong to the pending action's session and its trusted sequence must be strictly greater than the current summary's `summary_presented_sequence`. A client/model-supplied transcript or hash is not authoritative.

## `confirmReservation`

### Input

```json
{
  "pending_action_id": "pnd_...",
  "summary_version": 1,
  "confirmation_evidence": {
    "kind": "FINALIZED_TURN",
    "turn_id": "turn_..."
  },
  "policy_acknowledgements": [
    "ARRIVAL_GRACE",
    "AREA_NOT_EXACT_TABLE"
  ],
  "idempotency_key": "idem_..."
}
```

Website form confirmation instead uses:

```json
{
  "kind": "UI_CONFIRM_EVENT",
  "event_id": "uievt_..."
}
```

### Output

```json
{
  "reservation": {
    "reservation_id": "res_...",
    "reference": "JS-482731",
    "status": "CONFIRMED",
    "version": 1,
    "seating": {
      "service_date": "2026-10-16",
      "local_start_time": "19:15",
      "time_zone": "America/Los_Angeles",
      "display": "Friday, October 16, 2026 at 7:15 PM Pacific time",
      "party_size": 4,
      "area": "PATIO",
      "expected_dining_minutes": 105
    }
  },
  "manage_token": "one_time_secret_return",
  "manage_url": "https://juniperandstone.example/reservations/manage#one_time_secret_return"
}
```

The transport removes `manage_token` before the tool result is appended to an LLM conversation. The customer UI stores it securely for the mock session.

## `getReservation`

Authentication is one of:

- Owning customer session.
- Valid manage token.
- Authorized staff session.

Input:

```json
{
  "reservation_id": "res_..."
}
```

Output is the public representation for customers and an explicitly separate staff projection for staff. A reference alone returns `NOT_FOUND` to avoid enumeration.

## Change availability

`searchChangeAvailability` uses the search input plus:

```json
{
  "reservation_id": "res_...",
  "expected_version": 1
}
```

The engine excludes the reservation's current assignment from conflicts while searching but does not release it. Replacement options are held through `holdSlot`.

## `prepareReservationChange`

Input:

```json
{
  "reservation_id": "res_...",
  "expected_version": 1,
  "replacement_hold_id": "hld_...",
  "guest_patch": {},
  "special_requests_patch": {
    "guest_note": "Anniversary dinner."
  },
  "idempotency_key": "idem_..."
}
```

For contact/note-only changes, `replacement_hold_id` is null. The result is a versioned before/after summary and pending-action ID. Inside the customer cutoff it returns `CHANGE_REQUIRES_STAFF`.

## `confirmReservationChange`

Input follows `confirmReservation` and includes pending-action ID/version, later evidence, current expected reservation version, and idempotency key.

Success:

```json
{
  "reservation": {},
  "change_event_id": "evt_...",
  "previous_version": 1,
  "new_version": 2
}
```

The replacement assignment and reservation version update atomically. Failure leaves version 1 and its assignment intact.

## Cancellation

### Prepare

```json
{
  "reservation_id": "res_...",
  "expected_version": 2,
  "idempotency_key": "idem_..."
}
```

Result includes reservation date/time, whether cancellation is late, customer-visible policy, and pending action.

### Confirm

```json
{
  "pending_action_id": "pnd_...",
  "summary_version": 1,
  "confirmation_evidence": {
    "kind": "UI_CONFIRM_EVENT",
    "event_id": "uievt_..."
  },
  "idempotency_key": "idem_..."
}
```

Success returns status `CANCELLED`, cancellation timestamp, late-cancellation flag, and released-inventory confirmation.

## Staff lifecycle commands

Common input:

```json
{
  "reservation_id": "res_...",
  "expected_version": 2,
  "reason": "Guest arrived",
  "idempotency_key": "idem_..."
}
```

Additional rules:

- `recordArrival`: Confirmed to checked in; walk-in create may combine creation and arrival.
- `seatReservation`: Checked in to seated; assignment must still be operational.
- `completeReservation`: Seated to completed.
- `markNoShow`: Confirmed to no-show only after start plus grace.
- Staff cancellation from checked in requires reason.
- Manager override carries `override_code` and non-empty `override_reason`.

## Idempotency contract

- Required for every command.
- Scope: restaurant, location, operation name, authenticated principal, and key.
- Store request fingerprint and committed result for at least 24 hours; reservation create/change/cancel keys may be retained with the aggregate longer.
- Same scope/key/fingerprint returns original status and body.
- Same scope/key with another fingerprint returns `IDEMPOTENCY_KEY_REUSED`.
- An in-progress duplicate waits briefly for the first transaction or returns retryable `COMMAND_IN_PROGRESS`; it never runs a second write.

## Optimistic concurrency

Commands changing a durable reservation require `expected_version`.

- Match: operation may proceed.
- Mismatch: return `RESERVATION_VERSION_CONFLICT` with the current safe public representation and `REFRESH_RESERVATION`.
- The application never silently overwrites newer staff/customer changes.

Holds use assignment/resource locking rather than reservation versioning.

## Error taxonomy

| Code | Category | Retryable | Next action |
|---|---|---:|---|
| `VALIDATION_FAILED` | Input | No | `CORRECT_FIELDS` |
| `TIME_AMBIGUOUS` | Input/time | No | `CLARIFY_TIME` |
| `TIME_NONEXISTENT` | Input/time | No | `CHOOSE_ANOTHER_TIME` |
| `SERVICE_CLOSED` | Policy | No | `CHOOSE_ANOTHER_DATE_OR_SERVICE` |
| `OUTSIDE_BOOKING_HORIZON` | Policy | No | `CHOOSE_ALLOWED_DATE` |
| `LEAD_TIME_NOT_MET` | Policy | No | `CHOOSE_LATER_TIME` |
| `PARTY_SIZE_REQUIRES_STAFF` | Policy | No | `HANDOFF` |
| `NO_AVAILABILITY` | Inventory | No | `SHOW_ALTERNATIVES` |
| `SLOT_TOKEN_INVALID` | Inventory/security | No | `SEARCH_AGAIN` |
| `SLOT_TOKEN_EXPIRED` | Inventory | No | `SEARCH_AGAIN` |
| `SLOT_STALE` | Inventory/conflict | No | `SEARCH_AGAIN` |
| `ACTIVE_HOLD_EXISTS` | State | No | `REVIEW_OR_REPLACE_HOLD` |
| `RESOURCE_HELD` | Inventory/conflict | Yes | `RETRY_OR_CHOOSE_ANOTHER` |
| `HOLD_NOT_FOUND` | State/security | No | `SEARCH_AGAIN` |
| `HOLD_EXPIRED` | State | No | `SEARCH_AGAIN` |
| `HOLD_INVALIDATED` | State/operations | No | `SEARCH_AGAIN_OR_HANDOFF` |
| `HOLD_OWNERSHIP_MISMATCH` | Security | No | `REAUTHENTICATE` |
| `PENDING_ACTION_EXPIRED` | State | No | `PREPARE_AGAIN` |
| `PENDING_ACTION_SUPERSEDED` | State | No | `REVIEW_CURRENT_SUMMARY` |
| `CONFIRMATION_REQUIRED` | State | No | `ASK_EXPLICIT_CONFIRMATION` |
| `CONFIRMATION_AMBIGUOUS` | State | No | `ASK_EXPLICIT_CONFIRMATION` |
| `CONFIRMATION_TOO_EARLY` | State | No | `ASK_AFTER_SUMMARY` |
| `POLICY_ACKNOWLEDGEMENT_REQUIRED` | Policy | No | `PRESENT_POLICY` |
| `RESERVATION_NOT_FOUND` | State/security | No | `CHECK_MANAGE_ACCESS` |
| `RESERVATION_VERSION_CONFLICT` | Concurrency | No | `REFRESH_RESERVATION` |
| `CHANGE_REQUIRES_STAFF` | Policy | No | `HANDOFF` |
| `INVALID_STATE` | State | No | `REFRESH_RESERVATION` |
| `IDEMPOTENCY_KEY_REUSED` | Idempotency | No | `GENERATE_NEW_KEY` |
| `COMMAND_IN_PROGRESS` | Idempotency | Yes | `RETRY_SAME_KEY` |
| `AUTHENTICATION_REQUIRED` | Security | No | `REAUTHENTICATE` |
| `FORBIDDEN` | Security | No | `HANDOFF_OR_STOP` |
| `RATE_LIMITED` | Platform | Yes | `RETRY_AFTER` |
| `PROVIDER_UNAVAILABLE` | Dependency | Yes | `RETRY_OR_FALLBACK` |
| `DATABASE_UNAVAILABLE` | Dependency | Yes | `RETRY_OR_HANDOFF` |
| `INTERNAL_ERROR` | Platform | Maybe | `SAFE_FAILURE` |

### Error-message rules

- Messages state what happened and a valid next step.
- Do not reveal whether another guest owns a conflicting table.
- Do not expose internal identifiers except the caller's safe aggregate IDs.
- Do not tell a customer that a reservation succeeded on a dependency/database error.
- Tool consumers branch on `code` and `next_action`, never English text.

## Model tool restrictions

- Tool schemas contain only the operation-specific input; context is server-injected.
- Tool names are allowlisted per workflow state.
- Free-text fields are length-limited and treated as untrusted.
- Search/hold tools cannot access arbitrary restaurant IDs.
- `confirm*` tools require an existing pending action and server-stored evidence.
- Staff lifecycle and configuration tools are absent from customer models.
- Tool results are reduced to customer-safe projections before returning to the model.
- Manage tokens, API keys, raw audit records, internal notes, and physical assignments never enter model context.

## Event contract

```json
{
  "event_id": "evt_...",
  "event_type": "reservation.created",
  "schema_version": "reservation.event.v1",
  "restaurant_id": "rst_juniper_stone",
  "location_id": "loc_oakland_001",
  "aggregate_id": "res_...",
  "aggregate_version": 1,
  "occurred_at": "2026-10-14T19:06:05Z",
  "local_service_date": "2026-10-16",
  "actor": {
    "type": "CUSTOMER_SESSION",
    "id": "ses_..."
  },
  "source": "WEB_VOICE",
  "correlation_id": "corr_...",
  "causation_id": "pnd_...",
  "payload": {
    "reference": "JS-482731",
    "party_size": 4,
    "area": "PATIO"
  }
}
```

Events are immutable. Consumers are idempotent by `event_id`. Event payloads are versioned and minimized.

## Reservation-provider adapter boundary

The application contracts above remain stable if an external reservation system becomes authoritative. An adapter reports capabilities:

```json
{
  "supports_holds": true,
  "supports_modify": true,
  "supports_cancel": true,
  "supports_webhooks": true,
  "supports_idempotency": true,
  "supports_area_preferences": true
}
```

Normalized provider operations:

- `getCapabilities`
- `searchAvailability`
- `createReservation`
- `getReservation`
- `modifyReservation`
- `cancelReservation`
- `consumeProviderEvent`

Application-level prepare/confirm remains ours. If a provider lacks holds, the adapter declares it, and confirmation performs immediate revalidation; the UI must not claim a guaranteed hold. Unsupported capabilities cause explicit staff handoff rather than fabricated success.

## Versioning

- Additive optional response fields may appear within v1.
- New enum values require tolerant display but cannot trigger writes without recognized handling.
- Removing/renaming fields, changing semantics, or weakening confirmation creates v2.
- Persist schema and rule versions with holds, pending actions, reservations, and events.
- Contract fixtures are validated in CI once implementation begins.

## Review decisions

- Confirm server-injected tenant/actor context.
- Confirm strict model-facing schemas and customer-safe projections.
- Confirm manage token never enters model context.
- Confirm later server-stored finalized turn as text/voice evidence.
- Confirm prepare/confirm pairs for create, change, and cancel.
- Confirm 24-hour minimum idempotency retention.
- Confirm error codes and next actions before implementation.
