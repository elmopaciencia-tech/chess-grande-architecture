# Engineering decisions

The table describes the benefits and tradeoffs of the implementation. It does not document the original author’s reasons for choosing it.

| Decision | Benefit | Tradeoff |
| --- | --- | --- |
| Extend WordPress with domain modules | Staff can manage content and operations in one administration environment | Modules share runtime, database and deployment risks |
| Keep meetup requests separate from paid tournaments | Family review and waitlisting can evolve without payment semantics | Access, administration and notification patterns need consistent maintenance across modules |
| Calculate tournament rules on the server | Browser input cannot select an arbitrary price or roster size | Frontend previews must stay aligned with server validation |
| Reserve player capacity during checkout | Checkout attempts account for individual and team sizes | Expiry, payment failure and late events require careful reconciliation |
| Store tournament snapshots | A registration retains the event and pricing context used at checkout | Snapshots duplicate data and need a deliberate retention policy |
| Separate payment from fulfilment | Staff can track review and team assignment after payment | The interface must explain several statuses clearly |
| Use a transactional outbox for meetups | A committed decision has a corresponding queued notification | Delivery depends on scheduling, retries and operational monitoring |
| Embed React for the chess assessment | Interactive chess boards run within WordPress | Frontend build dependencies and the event contract need maintenance |
| Reuse private-lesson commerce across presentation versions | A redesigned page can retain backend pricing and order behavior | The newer presentation remains coupled to shared classes from the earlier module |
| Share assessment-credit lifecycle logic | Participating bookings coordinate reservations and redemption | Each booking flow must release abandoned reservations and redeem successful ones correctly |

## Failure handling

Checkout idempotency, webhook deduplication, payment-state transitions and notification retries address different failures. A repeated checkout request, a repeated provider event and a failed email delivery should not be treated as the same problem.

The tournament implementation has its own event ledger and confirmation queue. Meetups use a separate transactional outbox. Each implements its own lifecycle rules.

## Scope

The implementation includes data modeling, server validation, asynchronous payment handling, concurrent capacity checks and staff workflows. It does not establish measured uptime, traffic capacity, revenue impact or a security certification. Those claims would require separate evidence.
