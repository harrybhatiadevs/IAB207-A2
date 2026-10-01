# Application workflows

## Booking decision flow

```mermaid
flowchart TD
    A[Authenticated attendee submits quantity] --> B[Load event and validate form]
    B --> C{Event active and not cancelled?}
    C -->|No| D[Reject booking]
    C -->|Yes| E{Quantity fits remaining capacity?}
    E -->|No| D
    E -->|Yes| F[Create booking and unique order ID]
    F --> G[Store quantity and Decimal unit price]
    G --> H[Refresh event status]
    H --> I[Commit and show booking history]
```

The route checks aggregate remaining event capacity. Ticket-tier records provide prices and capacity data; this does not establish transactional per-tier inventory reservation. Payments are simulated. Concurrent overselling prevention would require database transactions/locking beyond these checks.

## Event status

```mermaid
flowchart TD
    A[Refresh status] --> B{Manually cancelled?}
    B -->|Yes| C[Keep Cancelled]
    B -->|No| D{Start time is in the past?}
    D -->|Yes| E[Inactive]
    D -->|No| F{Remaining capacity is zero?}
    F -->|Yes| G[Sold Out]
    F -->|No| H[Open]
```

Status is recalculated by `Event.refresh_status` when relevant views are accessed, not by a background scheduler. Owner checks protect event editing, cancellation and deletion.

## Domain relationships

```mermaid
erDiagram
    USER ||--o{ EVENT : owns
    USER ||--o{ BOOKING : places
    USER ||--o{ COMMENT : writes
    EVENT ||--o{ TICKET_TYPE : defines
    EVENT ||--o{ BOOKING : receives
    EVENT ||--o{ COMMENT : receives
```

Booking totals use `Decimal` and fixed-precision columns. Passwords use Werkzeug PBKDF2 hashing. The Flask application factory registers authentication and main blueprints, initialises extensions and seeds the local demo catalogue.
