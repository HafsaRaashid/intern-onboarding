# BookSwap - Reliability Runbook v0.1

## Failure 1
Azure SQL primary unavailable for 5 minutes
### What the user sees
Any page that reads from or writes to the database returns a 503 Service Unavailable after a 30-second connection timeout. Static or cached pages remain accessible. The user sees: "We're having trouble loading your data. Please try again shortly." 
### Detection
Azure has a built-in alarm called connection_failed
Alert rule: connection_failed > 10 over a 60-second window That alert wakes someone up via a tool called PagerDuty
### Mitigation in design (timeouts, retries, circuit breaker, fallback)
Timeout: connection timeout = 5 s, command timeout = 30 s
Retry: 3 attempts, exponential back-off (1,2,4)
Circuit breaker: open after 5 consecutive faults, stay open for 30 seconds, then half-open to test recovery
Fallback: reads route to the Azure SQL read replica (backup database that's always kept in sync); writes enqueue to Azure Service Bus save it properly once the database comes back.Before writing, it checks for an existing idempotency_key on the listings table if it already exists, the write is skipped.
### Manual response (who is paged, what they do)
PagerDuty pages the on-call engineer.
 Steps: (1) Confirm outage in Azure Portal. (2) If database hasn't automatically switched to the backup within 3 minutes, manually trigger the switchover (3) Post status to the team Slack channel (4) If switchover fails, enable maintenance mode to show a static page so users see a clear message.
### Post-incident actions
Write up what happened and why (RCA - Root Cause Analysis) within 48 hours
Add a heartbeat monitor: a heartbeat row written and read every 30 s, alerting within 90 s of failure so the alarm fires faster next time
Run a quarterly chaos drill: manually trigger failover in staging and make sure the team can actually follow the steps above

## Failure 2
Azure Cache for Redis is down
### What the user sees
Pages that rely on cached session data or cached query results become slow response times spike from ~50 ms to 2-5 s.
### Detection
Azure tracks a metric called connectedclients - the number of things currently connected to Redis.Normally this is a healthy non-zero number.
Alert rule: connectedclients < 1 for 2 consecutive minutes / app logs say "pool timeout"
### Mitigation in design
the application continues to operate with degraded performance by falling back to the primary data store when Redis is temporarily unavailable.
Cache-aside: on a Redis TIMEOUT or CONNECTION exception, fall through to Azure SQL - never hard-fail to the user
Circuit breaker: open after 3 Redis failures in 10 s; while open, bypass Redis entirely and query SQL directly.
TTLs: all cached keys must have explicit TTLs (e.g. 5 min for query results) - prevents stale data surviving a Redis restart
### Manual response
An on-call engineer gets paged. They:
Steps: (1) Check Azure Portal Redis if Redis is broken on Azure's side, or its something in your own app (2) If Azure fault: raise a support ticket with Microsoft and wait (3) If Azure Resource Health shows Redis is healthy but your app is erroring: check Application Insights for "pool timeout" log entries. If found, restart the App Service instances to clear the exhausted connection pool.
### Post-incident actions
Check if anything in the app will completely break (not just slow down) without Redis - those parts need the cache-aside fallback added


## Failure 3
Sunday tabloid spike - 10x sustained traffic
### What the user sees
If planned well: nothing.a little bit slower for a few minutes while new servers spin up. 
If not planned: pages time out, users get 503 errors, and the site goes down under the weight of its traffic.
### Detection
HttpQueueLength is an App Service metric that measures how many requests are waiting to be processed 
Alert rule: HttpQueueLength > 100 over 5 minutes  
### Mitigation in design (autoscale, queue depth, throttling)
Autoscale: When the queue length exceeds 100, Azure automatically starts 2 extra server instances within a few minutes. When the queue drops below 10 (traffic calms down), it slowly scales back down. 
Pre-scaling: Because tabloid spikes are predictable (Sunday mornings), set a rule to already have 3x the normal number of servers running every Sunday before the queue builds up.
Throttling: If a single user (or a bot) is sending hundreds of requests, the system sends them an HTTP 429 response "slow down, try again in 5 seconds." This protects the database from being hammered by one bad actor during a busy moment.

### Manual response
An on-call engineer gets a warning alert. They:
(1) Check that autoscale has actually fired look at the Scale Out history in Azure Portal
(2)If autoscale hit its maximum limit and still isn't enough, manually raise the instance count
(3)If the database is also struggling (CPU > 80%), temporarily turn off any non-critical background jobs that are also using it
### Post-incident actions
Run a load test (a simulated traffic spike) in a test environment 
Make the Sunday pre-scale automatic and permanent in the autoscale settings
