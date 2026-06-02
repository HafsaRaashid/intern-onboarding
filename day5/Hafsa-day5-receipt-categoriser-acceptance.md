# Receipt Categoriser — Acceptance Criteria

## AC-01 Happy path: clear meal receipt
**Given** a receipt image of a restaurant bill totalling LKR 2,400
**When** the claimant uploads it via POST `/claims/{id}/receipts/categorise`
**Then** the response is 200 OK with `{ "category": "Meals", "confidence": >= 0.6, "source": "llm" }`
**And** an Application Insights customEvent `categoriser.suggested` is emitted within 4 seconds

## AC-02 Ambiguous receipt
**Given** a receipt with mixed items (food + stationery)
**When** the claimant uploads it via POST `/claims/{id}/receipts/categorise`
**Then** the suggestion includes `confidence < 0.6` and `needs_review: true`
**And** the client displays a "Needs review" badge so the claimant verifies before submitting

## AC-03 LLM unavailable — fallback
**Given** Azure OpenAI is returning 503
**When** the claimant uploads a receipt
**Then** the response is 200 OK with `source: "rule-based"`
**And** the claimant sees a normal suggestion according to the rule and no error message

## AC-04 OCR failure
**Given** an image that Document Intelligence cannot parse
**When** the claimant uploads it via POST `/claims/{id}/receipts/categorise`
**Then** the response is 200 OK with `{ "category": "Other", "confidence": 0.0, "source": "default", "needs_review": true }`
**And** the client displays "Could not read receipt — please select a category manually"

## AC-05 Oversized payload - input error
**Given** a receipt image larger than 10 MB
**When** the claimant uploads it via POST `/claims/{id}/receipts/categorise`
**Then** the response is 413 with `{ "error": "File too large", "max_bytes": 10485760 }`
**And** no call is made to Document Intelligence or Azure OpenAI

## AC-06 PII boundary
**Given** a receipt with a customer name and credit card last 4 digits
**When** the categorizer processes the request 
**Then** the customEvent payload contains no PII
**And** the Azure OpenAI request stays within the BISTEC Azure tenant no receipt data leaves the tenant boundary

## AC-07 Feature flag off
**Given** the categoriser feature flag is set to off in Azure App Configuration
**When** a receipt is uploaded
**Then** the categoriser does not run 
**And** the claimant picks a category manually

## AC-08 Logging completeness
**Given** the claimant receives a suggestion and taps "Accept" or changes the category
**When** the interaction completes
**Then** Application Insights contains a `categoriser.suggested` event with properties: claim_id, category, confidence, source, needs_review, outcome ("accepted" or "overridden"), and latency_ms

## AC-09 Latency
**Given** 100 receipt images of varying quality and size
**When** each is processed via POST `/claims/{id}/receipts/categorise`
**Then** the p95 response time is under 4 seconds