# BookSwap — Storage and Cache Decisions

## 1. Data inventory
| Data type | Example record | Volume estimate (1y) | Read/write ratio |
|-----------|----------------|----------------------|------------------|
| Book listing | { bookId: "b-001", title: "Harry Potter", author: "J.K. Rowling", isbn: "978-3-16", condition: "new", photoUrl: "https://blob.azure.com/photos/HarryPotter.jpg", available: true }| ~50,000 across all buildings | read-heavy |
| Book photo | image file stored and referenced by photoUrl in Book Listing | ~50,000 across all buildings - 1 image per book | read-heavy | 
| Book Requests | { requestId: "br-001", bookId: "b-001", requestedBy: "u-042", status: "pending", requestedAt: "2025-01-10T09:00:00Z" } | ~20,000 requests | balanced |
| Loans | { loanId: "l-001", bookId: "b-001", borrowedBy: "u-042", borrowedAt: "2025-01-11T10:00:00Z", dueDate: "2025-01-25T10:00:00Z", returnedAt: null, overdue: false } | ~30,000 Loans | balanced |
| User Profiles | { userId: "u-042", name: "Hafsa", email: "hafsa@email.com", building: "Block A" } | ~10,000 Users | read-heavy |

## 2. Storage selection
| Data type | Chosen store | Why this store | Why not the alternatives |
|-----------|--------------|----------------|--------------------------|
| Book listing | Azure SQL | Relational with FK to users | Document DB unnecessary, relational joins useful |
| Book photo | Azure Blob Storage | Binary, big | Database BLOBs would bloat backups |
| Book Requests | Azure SQL |  Relational with FK to users,book | Cache not appropriate as data changes frequently and needs persistence, Document DB unnecessary, relational joins useful |
| Loans | Azure SQL | Relational with FK to users,book | Cache not appropriate as volatile and Loans must be presistent for viewing loan history, Document DB unnecessary as all loans will have the same fields, relational joins useful |
| User Profiles | Azure SQL | Related to loans,requests and listings | Cache not appropriate as volatile and user data must eb presistent, Document DB unnecessary as all profiles will have the same fields, relational joins useful |
|Book listings searches | Azure Cache for Redis | Frequently searched results repeated across many users; reducing database load under concurrent usage | Not a permanent store — short TTL (~4 min); falls back to Azure SQL on expiry |


## 3. Cache plan
Azure SQL is the source of truth for all book data. 
- What is hot enough to cache?

Book listings searches

- Cache-aside pattern in pseudocode

1. Receive search request
2. Check if result exists in Redis cache If true return cached result
3. If false query Azure SQL, store result in Redis with TTL of 4 minutes, return result
5. After TTL (4 mins) expires, cache entry is deleted automatically
6. Repeat

- TTL choice and invalidation strategy

A TTL of 4 minutes was chosen to meet the 300ms p95 search response time NFR while accepting slight staleness in availability status. The risk of stale availability within the TTL window is low and acceptable considering the cost of repeated querying.

## 4. Queue plan
- Which work goes on a queue and why
Email notifications — decouples listing creation from the email service so listing creation succeeds even if the email service is down (per NFR).

- What happens if the consumer is down for 30 minutes
Azure Queue Storage retains messages in the queue until the consumer is back up. When the consumer (background worker) comes back up, it reads and processes the backlog of messages. Messages that fail repeatedly after maximum retries are moved to a dead-letter queue for manual inspection, ensuring no notifications are silently dropped. Members will receive their email notifications late but no listings are lost.