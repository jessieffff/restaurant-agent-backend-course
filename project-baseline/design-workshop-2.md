# Design Workshop 2: Mock Restaurant and Reservations

**Status:** Proposed deliverables ready for review  
**Baseline:** [High-Level Design v1](./high-level-design-v1.md)

## 1v1 Agent Backend track continuation

This workshop now serves as the product and reservation-domain input to the
mentor-led
[1v1 Agent Backend Track](../1v1-agent-backend-track.md).
The student
will complete the track through 19 structured 60-minute 1v1 sessions and
implement these requirements through the
[Technical Learning Architecture](../technical-learning-architecture.md),
use only the required tools approved by the
[Zero-Cost Tooling Policy](../free-tooling-policy.md),
deliver work through the
[GitHub Operating Model](../github-operating-model.md),
follow the
[Mentor and Scrum Playbook](../mentor-scrum-playbook.md),
and
graduate against the
[Assessment, Interview, and Resume Evidence](../assessment-interview-resume.md)
rubric.

The learning overlay does not change the Workshop 2 business invariants and
does not add class-course curriculum.

## Purpose

Turn the high-level architecture into a reviewable specification for the
fictional restaurant and its reservation workflow. These Workshop 2 documents
define business and domain decisions only; the learning overlay selects the
implementation route, but implementation itself has not started.

## Deliverables

| Document | Scope |
|---|---|
| [Mock restaurant specification](./mock-restaurant-spec.md) | Juniper & Stone's brand, hours, service periods, table inventory, policies, representative menu, staff roles, and operational states |
| [Reservation domain design](./reservation-domain-design.md) | Availability, allocation, holds, concurrency, lifecycle, confirmation, authorization, staff operations, events, and failure behavior |
| [Reservation contracts](./reservation-contracts.md) | Channel-neutral operations, strict request/result schemas, idempotency, optimistic concurrency, errors, events, and provider boundaries |
| [Reservation acceptance cases](./reservation-acceptance-cases.md) | 43 deterministic scenarios covering form, text, voice, concurrency, policy, security, failures, staff conflicts, and accessibility |

## Proposed decisions

- Use **Juniper & Stone**, a fictional modern California bistro in Oakland, in the `America/Los_Angeles` time zone.
- Model real tables, predefined table combinations, dining areas, and adjacent bar seats rather than one aggregate cover counter.
- Offer public bookings for parties of 1-8 up to 60 days ahead, with 60 minutes minimum lead time.
- Use seven-minute inventory holds and five-minute search tokens.
- Share one active hold per customer session across form, text, and voice.
- Require a versioned review summary and a separate later confirmation before create, change, or cancellation.
- Treat `MODIFIED` as an event and version change, not a reservation lifecycle status.
- Preserve the original reservation until a replacement assignment is atomically secured.
- Permit customer changes until two hours before seating; later changes route to staff.
- Store only a hash of the manage token and never expose the raw token to the language model.
- Defer waitlists, deposits, external reservation adapters, and production PII handling.

## Review checklist

- [ ] Confirm or replace the restaurant concept, Oakland setting, and brunch/dinner scope.
- [ ] Confirm the table inventory, combinations, area rules, and accessibility behavior.
- [ ] Confirm booking horizon, lead time, hold duration, party limits, grace period, and modification cutoff.
- [ ] Confirm the hold, pending-action, and reservation state models.
- [ ] Confirm explicit text/voice confirmation requirements.
- [ ] Confirm staff permissions and manager override behavior.
- [ ] Confirm contract operations, stable error codes, and safe model projections.
- [ ] Confirm the 43 acceptance scenarios are sufficient to begin implementation planning.

## Quality review

An independent design review found no blocking issue. Its actionable edge cases were incorporated:

- Confirmation evidence must belong to the same session and occur after the current summary was presented.
- Form, text, and voice share one active hold per customer session.
- Search tokens have an explicit five-minute lifetime.
- A public race for a stale option returns one stable `SLOT_STALE` result.
- Bar reservations explicitly test adjacent-seat requirements.

## Next step

Record review changes directly in these documents and update this checklist. Do not begin implementation until the proposed decisions are accepted or revised.
