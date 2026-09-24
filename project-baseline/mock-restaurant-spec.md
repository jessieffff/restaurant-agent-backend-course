# Mock Restaurant Specification: Juniper & Stone

**Status:** Proposed Workshop 2 baseline for review  
**Configuration version:** `mock-restaurant-v1`  
**Data classification:** Entirely fictional; never use these records as real contact or location data

## Decision record

The user was unavailable for the first workshop choice, so this proposal uses the recommended concept: a modern California neighborhood bistro with a dining room, bar, and patio. It was selected because it exercises common restaurant questions, multiple seating preferences, combinable tables, weather-dependent inventory, brunch and dinner services, dietary questions, takeout, and large-party handoff.

Every choice below is configuration, not application code. A later review can replace the brand or rules without changing the architecture.

## Restaurant identity

| Field | Value |
|---|---|
| Restaurant ID | `rst_juniper_stone` |
| Location ID | `loc_oakland_001` |
| Name | Juniper & Stone |
| Concept | Seasonal California neighborhood bistro |
| Positioning | Approachable dinner and weekend brunch; produce-forward menu with a full bar |
| Public tone | Warm, concise, knowledgeable, never overly formal |
| Locale | `en-US` |
| Currency | `USD` |
| Time zone | `America/Los_Angeles` |
| Address | 1250 Example Avenue, Oakland, CA 94612 |
| Phone | +1 510-555-0147 |
| Email | hello@juniperandstone.example |
| Website | https://juniperandstone.example |

The `.example` domain and `555-01xx` phone range are deliberate synthetic values. The mock website must show a persistent "Demo restaurant - no real bookings" banner and must not link the fictional address to live navigation.

## Brand and website direction

- Visual character: warm stone, juniper green, cream, and muted copper.
- Photography direction: seasonal plates, welcoming room, natural light, neighborhood setting.
- Primary calls to action: Reserve a table, Order takeout, Ask Juniper.
- Agent greeting: "Hi, I'm Juniper, the restaurant assistant. I can help with hours, menu questions, reservations, and takeout."
- The assistant identifies itself as automated when a conversation begins and never pretends to be a staff member.
- English is the only authored language in the first mock, but every content record carries a locale so translation can be added later.

## Location information

- Cross street: Example Avenue and Demo Street.
- Transit copy: "About two blocks from the downtown transit station."
- Parking copy: "Metered street parking and a public garage across Demo Street."
- Accessibility copy: "The main entrance and dining room are step-free. Accessible seating is available in the main room and patio. Please mention mobility needs when reserving."
- Patio copy: "The patio is partially covered and may close because of weather or air quality."
- Map treatment: a clearly labeled fictional static illustration, not a Google Maps pin.

## Public hours

| Day | Public hours | Services |
|---|---|---|
| Monday | Closed | None |
| Tuesday | 5:00 PM-10:00 PM | Dinner |
| Wednesday | 5:00 PM-10:00 PM | Dinner |
| Thursday | 5:00 PM-10:00 PM | Dinner |
| Friday | 5:00 PM-11:00 PM | Dinner |
| Saturday | 10:00 AM-2:30 PM; 5:00 PM-11:00 PM | Brunch, Dinner |
| Sunday | 10:00 AM-2:30 PM; 5:00 PM-9:30 PM | Brunch, Dinner |

Public closing time is not the last reservation time. The reservation engine uses the service periods below.

## Service periods

| Service ID | Days | First seating | Last seating | Slot interval |
|---|---|---:|---:|---:|
| `svc_dinner_weekday` | Tue-Thu | 5:00 PM | 8:30 PM | 15 minutes |
| `svc_dinner_weekend` | Fri-Sat | 5:00 PM | 9:30 PM | 15 minutes |
| `svc_dinner_sunday` | Sun | 5:00 PM | 8:00 PM | 15 minutes |
| `svc_brunch_weekend` | Sat-Sun | 10:00 AM | 1:15 PM | 15 minutes |

### Standard turn times

| Party size | Brunch dining time | Dinner dining time | Reset buffer |
|---:|---:|---:|---:|
| 1-2 | 75 minutes | 90 minutes | 15 minutes |
| 3-4 | 90 minutes | 105 minutes | 15 minutes |
| 5-6 | 105 minutes | 120 minutes | 15 minutes |
| 7-8 | 120 minutes | 135 minutes | 15 minutes |

The occupied interval is dining time plus reset buffer. A reservation may not start if its occupied interval would exceed the configured operational end for its service and area.

## Holiday and exceptional hours

The seed data includes explicit overrides:

| Date | Override |
|---|---|
| 2026-11-26 | Closed all day for Thanksgiving |
| 2026-12-24 | Dinner only, 4:00 PM-8:00 PM public hours; last seating 6:30 PM |
| 2026-12-25 | Closed all day |
| 2026-12-31 | Dinner, 5:00 PM-11:30 PM public hours; last seating 9:30 PM; standard menu for the mock |
| 2027-01-01 | Brunch only, 10:00 AM-3:00 PM; last seating 1:45 PM |

An exceptional-hours record always overrides the weekly schedule. A closed override cannot be bypassed by the customer website or agent.

## Dining areas and inventory

### Main dining room

| Resource | Seats | Accessible | Online use |
|---|---:|---|---|
| M01-M04 | 2 each | M01 | Parties 1-2 |
| M05-M08 | 4 each | M05 | Parties 3-4; smaller parties only when necessary |
| M09-M10 | 6 each | M09 | Parties 5-6 |

Allowed combinations:

- M01 + M02: capacity 4
- M03 + M04: capacity 4
- M05 + M06: capacity 8
- M07 + M08: capacity 8
- M09 + M10: capacity 12, staff-created large parties only

### Patio

| Resource | Seats | Accessible | Online use |
|---|---:|---|---|
| P01-P04 | 2 each | P01 | Parties 1-2 |
| P05-P06 | 4 each | P05 | Parties 3-4; smaller parties only when necessary |

Allowed combinations:

- P01 + P02: capacity 4
- P03 + P04: capacity 4
- P05 + P06: capacity 8

The patio has an operational toggle with `open`, `closed_weather`, `closed_air_quality`, and `closed_maintenance` states. Existing patio reservations remain visible when it closes and enter a staff relocation queue; the system does not silently move or cancel them.

### Bar counter

- Eight contiguous seats: B01-B08.
- Online reservations are available for parties of one or two.
- Two-person reservations require adjacent seats.
- Bar seating is counter-height and is not labeled accessible.

### Allocation preferences

1. Use the smallest compatible single table or contiguous bar block.
2. Prefer a single table over a combination.
3. Minimize unused seats.
4. Preserve accessible resources for guests who request them when an equivalent non-accessible assignment exists.
5. Preserve larger tables and combinations for larger parties when equivalent choices exist.
6. Honor an explicit dining-area selection or return alternatives labeled with their area; never silently substitute an area.
7. Treat accessibility as a hard constraint when requested.
8. Treat celebrations, quiet-table requests, window requests, and high chairs as non-guaranteed notes.

The engine assigns inventory for availability protection, but the public confirmation promises an area, not a specific table number.

## Reservation rules

| Rule | Value |
|---|---|
| Booking horizon | 60 calendar days in restaurant-local time |
| Minimum lead time | 60 minutes before seating |
| Online party size | 1-8 covers |
| Large-party behavior | 9+ routes to live staff or a ticket |
| Children | Count as covers |
| High chairs | Request only; do not reduce cover count |
| Slot hold | 7 minutes, visible countdown |
| Hold extension | None; re-search after expiration |
| Arrival grace period | 15 minutes |
| Customer modification cutoff | 2 hours before seating |
| Customer cancellation | Allowed at any time |
| Late cancellation | Cancellation inside 2 hours is accepted and tagged for staff; no fee in the mock |
| Deposit | None |
| Waitlist | Deferred from the first reservation design |
| Table guarantee | Dining area may be confirmed; exact table is never guaranteed |

### Guest information

Required:

- First and last name.
- Mobile phone number.

Optional:

- Email.
- Accessibility need.
- High-chair request.
- Celebration note.
- Dietary or allergy note.
- Other special request.

No OTP verification is included in the mock. Public manage links use a high-entropy secret token; the short human reference number is not sufficient authorization.

### Confirmation policy

- Search does not create inventory.
- Selecting an option creates a seven-minute hold.
- The final summary must state the full date, local time, time zone, party size, dining area, guest contact, applicable policy, and hold expiration.
- Creation requires a separate explicit confirmation after that summary.
- A website confirmation button is authoritative.
- Text or voice confirmation must refer to the current pending summary. Voice requires a finalized explicit phrase such as "Confirm the reservation" or "Yes, book it."
- Any change to time, party size, area, or contact information invalidates the pending confirmation and produces a new summary/version.

### Modification and cancellation

- A modification is atomic: either a replacement assignment is secured and the reservation updates, or the original reservation remains unchanged.
- Customer modifications inside the two-hour cutoff route to staff.
- Cancellation releases inventory immediately and is idempotent.
- A modification emits an event; it does not become a permanent "modified" lifecycle status.
- Staff can override cutoffs with a required reason and audit record.

## Takeout windows

Takeout detail is outside Workshop 2, but the mock restaurant needs public facts:

| Day/service | Pickup window |
|---|---|
| Tue-Thu dinner | 5:15 PM-8:30 PM |
| Fri-Sat dinner | 5:15 PM-9:00 PM |
| Sat-Sun brunch | 10:30 AM-1:30 PM |
| Sunday dinner | 5:15 PM-7:30 PM |

- Default lead time: 30 minutes.
- Payment: at pickup.
- Pickup location: host stand.
- Delivery: not offered.

## Representative menu

Dietary labels describe recipe intent, not a guarantee against cross-contact.

### Dinner

| ID | Item | Price | Labels | Declared major allergens |
|---|---|---:|---|---|
| `din_bread` | Grilled sourdough, cultured butter, sea salt | $8 | Vegetarian | Wheat, milk |
| `din_beets` | Citrus-roasted beets, pistachio, herbs | $15 | Vegan, gluten-free recipe | Tree nuts |
| `din_crab_toast` | Dungeness crab toast, celery, lemon | $19 |  | Shellfish, wheat, egg |
| `din_polenta` | Crispy polenta, tomato fondue, parmesan | $14 | Vegetarian, gluten-free recipe | Milk |
| `din_chicken` | Roasted half chicken, greens, jus | $29 | Gluten-free recipe | Milk |
| `din_salmon` | Grilled salmon, spring vegetables, salsa verde | $32 | Gluten-free recipe | Fish |
| `din_tagliatelle` | Wild mushroom tagliatelle, pecorino | $26 | Vegetarian | Wheat, egg, milk |
| `din_burger` | Juniper burger, cheddar, onion jam, fries | $23 |  | Wheat, milk, egg |
| `din_short_rib` | Red-wine braised short rib, potatoes | $34 | Gluten-free recipe | Milk |
| `din_vegetables` | Seasonal vegetable plate, quinoa, tahini | $24 | Vegan, gluten-free recipe | Sesame |
| `din_olive_oil_cake` | Olive oil cake, citrus cream | $12 | Vegetarian | Wheat, egg, milk |
| `din_chocolate` | Dark chocolate pot de creme | $13 | Vegetarian, gluten-free recipe | Egg, milk |

### Brunch

| ID | Item | Price | Labels | Declared major allergens |
|---|---|---:|---|---|
| `bru_pancakes` | Lemon ricotta pancakes, berries | $18 | Vegetarian | Wheat, egg, milk |
| `bru_avocado` | Avocado toast, soft egg, herbs | $17 | Vegetarian | Wheat, egg |
| `bru_benedict` | Smoked salmon eggs Benedict | $21 |  | Fish, wheat, egg, milk |
| `bru_grain_bowl` | Breakfast grain bowl, vegetables, poached egg | $18 | Vegetarian | Egg |
| `bru_hash` | Short-rib hash, potatoes, peppers, eggs | $22 | Gluten-free recipe | Egg |
| `bru_burger` | Juniper burger, cheddar, onion jam, fries | $23 |  | Wheat, milk, egg |

### Nonalcoholic drinks

| ID | Item | Price |
|---|---|---:|
| `bev_sparkling` | Sparkling water | $5 |
| `bev_lemonade` | Rosemary lemonade | $6 |
| `bev_tea` | Iced black tea | $5 |
| `bev_zero_spritz` | Juniper zero-proof spritz | $10 |

Alcohol details can appear on the public menu but are not orderable through the first takeout workflow.

## Allergen and dietary language

Approved answer:

> We list recipe ingredients and declared major allergens, but our kitchen handles wheat, milk, eggs, fish, shellfish, tree nuts, peanuts, sesame, and soy. We cannot guarantee against cross-contact. Please tell us about any allergy, and I can ask a staff member to help.

The agent must not declare an item "safe" for an allergy. Allergy questions that require cross-contact judgment route to staff.

## Approved FAQ and policy facts

- Reservations are recommended but walk-ins are welcome when space permits.
- Online reservations support parties of one to eight.
- Parties of nine or more require staff assistance.
- Reservations are held for 15 minutes after the booked time.
- Exact tables are not guaranteed.
- Patio requests depend on weather and operational conditions.
- Children are welcome and count toward party size.
- High chairs are limited and treated as requests.
- The main entrance and selected tables are accessible.
- Service animals are welcome; pets are permitted only on the patio when it is open.
- Outside cakes are allowed with advance notice; a $3 per-person plating fee is disclosed but not charged by the mock payment system.
- Corkage is $25 per 750 ml bottle, maximum two bottles, and cannot duplicate a current list item.
- Takeout is paid at pickup; delivery is not offered.
- Gift cards, loyalty, private events, and catering are outside the first mock workflow and route to staff.
- The restaurant does not promise allergen-safe preparation.

## Staff roles

### Manager

- Edit restaurant facts, schedules, inventory, policies, and menu availability.
- Create and modify any reservation.
- Override capacity or cutoff rules with a reason.
- Close an area and manage relocation work.
- View all audit records and staff presence.

### Staff

- Create reservations and walk-ins within configured capacity.
- Check in, seat, complete, cancel, or mark no-show.
- Add internal notes.
- Respond to live handoffs and tickets.
- Cannot change core capacity rules or erase audit history.

Seed accounts use `.example` email addresses. Passwords and API credentials are never stored in this document or committed seed data.

## Temporary operational states

The staff dashboard can activate:

- Entire location closed with public reason and effective interval.
- Dining area closed with operational reason.
- Specific table blocked with effective interval.
- Reservations paused while existing bookings remain valid.
- Takeout paused.
- Item sold out.
- Staff live-chat presence online/offline.

Every state has `starts_at`, optional `ends_at`, actor, reason, and audit event. Expired temporary states do not remain active.

## Review checklist

- Confirm or replace the concept, name, and Oakland setting.
- Confirm that brunch plus dinner is desirable test scope.
- Confirm table inventory and area mix.
- Confirm seven-minute holds and 60-day horizon.
- Confirm party-size and two-hour change thresholds.
- Confirm the fictional menu is sufficient for reservation-context questions.
- Confirm waitlist and deposits remain deferred.
