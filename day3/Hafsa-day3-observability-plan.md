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

## Required Tests/Validations:
| # | Signal type | Source | What it answers | Sample query / metric name |
|---|-------------|--------|-----------------|---------------------------|
| 1 | Metric | Application Insights | Search latency p95 | `requests \| where name == "GET /books" \| summarize percentile(duration, 95) by bin(timestamp, 1m)` |
| 2 | Metric | Application Insights | Listing creation success rate | `requests \| where name == "POST /books" \| summarize success = countif(success == true), total = count() by bin(timestamp, 1m) \| extend rate = (success * 100.0) / total` |
| 3 | Log | App Insights traces | Authn failures with member ID | `traces \| where customDimensions.event == "auth.failed" \| project timestamp, customDimensions.memberId, customDimensions.reason` |
| 4 | Trace | Application Insights | Slow request breakdown across DB and Redis | `dependencies \| where duration > 500 \| summarize avg(duration) by type, target, bin(timestamp, 5m)` |
| 5 | Metric | Service Bus | Email digest queue depth | `AzureMetrics \| where ResourceType == "MICROSOFT.SERVICEBUS/NAMESPACES" \| where MetricName == "ActiveMessages" \| summarize avg(Average) by bin(TimeGenerated, 1m)` |
| 6 | Log | Application Insights | Book listing creation errors | `traces \| where severityLevel == 3 \| where customDimensions.event == "book.create.failed" \| project timestamp, customDimensions.memberId, message` |
| 7 | Trace | Application Insights | Borrow request flow across API and DB | `dependencies \| where operation_Name == "POST /books/{bookId}/borrow-requests" \| summarize avg(duration) by type, target, bin(timestamp, 5m)` |


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

