# BookSwap — SLI/SLO Map

## 1. NFR inventory
| # | NFR (from Day 2) | User-visible behaviour |
|---|------------------|------------------------|
| 1 | Catalogue search response time is <300 ms p95, with up to 5,000 books | User able to search for a book within 300 ms
| 2 | Listing creation succeeds even if the email service is down | User creates a listing eventhough the email sevice gets delayed
| 3 | Photo uploads up to 5 MB JPEG/PNG; never stored in the app DB | User is able to upload images in png and jpeg format within 5 MB
| 4 | in-app notifications  arrive within 2s; email digest is best-effort | after a user creates a listing they get notified in the app within 2 seconds
| 5 | Addresses/phone numbers never returned in API responses | Users can never view other users phone numbers and adresses

## 2. SLI / SLO table
| # | SLI definition | Measurement source | SLO target | Window | Error budget |
|---|----------------|---------------------|------------|--------|--------------|
| 1 | Of all GET /books requests in the last 28 days, the percentage that returned 2xx in under 800ms | App Insights requests table | 99% | 28 days | 1% |
| 2 | Of all POST /books requests in the last 28 days, the percentage that did NOT return 5xx | App Insights requests table | 99% | 28 days | 0.1% | 
| 3 | Of all POST /books/{bookId}/upload requests where the file exceeded 5MB or was not JPEG/PNG, the percentage that returned 4xx | App Insights requests table | 100% | 28 days | 0% |
| 4 | Of all POST /books requests that returned 2xx in the last 28 days, the percentage where a notification was enqueued within 2 seconds of the request completion timestamp | Custom metric in App Insights (consumer logs delta between request time and EnqueuedTimeUtc) | 95% | 28 days | 5% | 
| 5 | Of all API responses in the last 28 days, the percentage that contain no address or phone fields in the response body | Integration/contract test suite — tests run on every deployment against all endpoints | 100% | Per deployment | 0% | 

## 3. Error budget policy
- What the team stops doing when the budget is exhausted
All non-critical feature work and new deployments are frozen until the budget is restored

- Who owns the decision
The on-call engineer owns the initial response. Escalation to the team lead if the breach cannot be resolved within 1 hour.

## 4. Out of budget right now
- One sentence: which SLO would you bet you cannot meet today and why
SLI 1 (search latency) — the system is currently designed for a single building; scaling to 200 buildings with a potential 10× traffic spike on Sunday has not been validated.