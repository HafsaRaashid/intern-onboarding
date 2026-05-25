# BookSwap — Observability Plan

## Setup
- Logs: Azure Monitor Logs — every request logs `requestId`, `memberId`,HTTP method, endpoint, status code, and duration. Retention: 30 days.
- Metrics: Azure Application Insights — request duration, success/failure rate, dependency calls (SQL, Redis, Service Bus), and custom metrics (notification enqueue latency).
- Traces: Application Insights distributed tracing — 100% sample rate for errors, 10% for successful requests.

## Results Summary
| Metric | Target | Achieved |
|--------|--------|----------|
| SLOs covered by an alert | 100% | 60% (3/5 by runtime alerts; SLO 3 and SLO 5 covered by tests) |
| Alerts with a clear runbook link | 100% | 100% |
| Dashboards for ops | 1 health, 1 business | 2 |

## Alert proposal
| Alert | Condition | Severity | Notification | Runbook |
|-------|-----------|----------|--------------|---------|
| Search latency SLO burn | p99 latency > 800ms for 5 minutes | Sev2 | PagerDuty + Teams | reliability/runbook.md#failure-2 |
| Listing creation failure | POST /books 5xx rate > 0.1% over 5 minutes | Sev1 | PagerDuty + Teams | reliability/runbook.md#failure-1 |
| Notification enqueue latency | Enqueue latency > 2s for more than 5% of requests over 10 minutes | Sev2 | PagerDuty + Teams | - |
| Listings endpoint down |Synthetic GET /books returns non-200 for 3 consecutive checks (3 minutes) | Sev1 | PagerDuty + Teams | reliability/runbook.md#failure-1 |
| Redis down | connectedclients < 1 for 2 consecutive minutes | Sev2 | PagerDuty + Teams | reliability/runbook.md#failure-2 |
| SQL connection failures | connection_failed > 10 over 60 seconds | Sev1 | PagerDuty + Teams | reliability/runbook.md#failure-1 |
| Service Bus queue depth | Email digest queue depth > 500 messages | Sev3 | Teams only | - |

SLO 3 and SLO 5 are protected by deployment-time contract and integration tests rather than runtime alerts.

## What we are deliberately NOT alerting on
- 1. 4xx errors on search - a member searching for a book that doesn't exist returns 404. This is expected behaviour, not a system problem. We only alert on 5xx errors.
- 2. Weekly email digest individual delivery failures - the NFR explicitly states the digest is best-effort. A single failed digest email does not page anyone — it is logged and retried on the next scheduled run. However if the queue depth exceeds 500 messages, a Sev3 alert fires.

## PII redaction 

The following are not written to any log, trace, or metric:
- Member email addresses
- Member names
- Addresses or phone numbers

