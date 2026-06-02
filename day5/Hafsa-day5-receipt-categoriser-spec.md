# Receipt Categoriser — Feature Spec v0.1

## 1. Why
- The user / business outcome we are solving

Reduce miscategorised expense claims and speed up the submission process, saving Finance time and giving BISTEC more accurate expense data.

Fewer miscategorised claims - Finance spends less time correcting errors
Faster submission - staff tap "accept" instead of browsing a dropdown
Better data - expense reports are more accurate because categories are consistent

- The metric this feature is expected to move

Suggestion acceptance rate — how often staff accept the AI suggestion vs override it (tracked via FR5 logging in Application Insights categoriser.suggested events). 

## 2. Scope
- In scope (1-3 bullets)

Receipt image OCR via Azure AI Document Intelligence, category suggestion via Azure OpenAI, with confidence score (0.0–1.0) and "Needs review" flag below 0.6
Fallback chain: LLM unavailable-> rule-based keyword matching, OCR fails-> default to "Other" with error message
Logging every suggestion (accepted or overridden) to Application Insights customEvents (categoriser.suggested) for future model evaluation

- Affects which containers / services from Day 4

| Container / Service | What changes |
|---|---|
| Web application | New UI showing suggested category, confidence score, and accept/change controls |
| API service: new Categoriser component | Calls Document Intelligence for OCR, calls Azure OpenAI for suggestion, handles fallback logic, returns result |
| API service — Receipts component | After upload completes, triggers the Categoriser component |
| API service — Audit component | Categoriser returns the suggestion, then Audit logs it to Application Insights.  |

**New external services:**

| Service | Why |
|---|---|
| Azure AI Document Intelligence | OCR - extracts text from receipt images (TC) |
| Azure OpenAI Service (gpt-4.1) | Suggests category from extracted text (TC) |
| Azure App Configuration | Feature flag to turn categoriser on/off without redeploying (TC, NFR) |

## 3. Contract
### Inputs
| Field | Type | Constraints | Source |
|---|---|---|---|
| receipt_image_url | string | Valid Blob Storage URL; jpeg or png; ≤ 10 MB (from Day 4 receipt upload) | Blob Storage |
| claim_id | string (UUID) | Must reference an existing claim in Draft or Submitted status | API request |

### Outputs

```json
{
  "claim_id": "string",
  "category": "Meals | Travel | Lodging | Office Supplies | Other",
  "confidence": 0.0 - 1.0,
  "needs_review": true | false,
  "source": "llm | rule-based | default",
}
```
- `category` — one of the five allowed values (FR1)
- `confidence` — 0.0 to 1.0; `needs_review` is true when confidence < 0.6 (FR2)
- `source` — "llm" if Azure OpenAI responded, "rule-based" if LLM was unavailable and fallback triggered, "default" if OCR failed entirely (FR4, NFR)

### Errors
| Code | Condition | Response body |
|---|---|---|
| 400 | Missing claim_id or invalid image format (not jpeg/png) | `{ "error": "Invalid input" }` |
| 404 | claim_id does not exist | `{ "error": "Claim not found" }` |
| 413 | Image exceeds 10 MB | `{ "error": "File too large", "max_bytes": 10485760 }` |


### Side effects
- Application Insights customEvent emitted called `categoriser.suggested`, Event properties: claim_id, category, confidence, source, needs_review, outcome (accepted/overridden), override_category (if changed), latency_ms, timestamp
(Emitted on every suggestion, regardless of success or failure)

## 4. Acceptance criteria

See deliverable 2: [Hafsa-day5-receipt-categoriser-acceptance.md](./Hafsa-day5-receipt-categoriser-acceptance.md)

## 5. Examples
### Example 1 — Happy path (clear travel receipt)

**Input:** Receipt image from PickMe showing "Colombo Fort to BIA airport, LKR 4,500"

**OCR output:** "PickMe Ride Receipt Colombo Fort - Bandaranaike International Airport Total: LKR 4,500"

**Output:**
```json
{
  "claim_id": "abc-123",
  "category": "Travel",
  "confidence": 0.92,
  "needs_review": false,
  "source": "llm",
}
```
Staff sees: **Travel**  (92% confident) - [Accept] [Change]

### Example 2 — Ambiguous receipt (mixed items)

**Input:** Receipt from Arpico Supercentre showing "A4 paper, biscuits, desk lamp, cleaning liquid"

**OCR output:** "Arpico A4 Paper x2 ... Munchee Biscuits ... Desk Lamp ... Cleaning Liquid Total: LKR 3,200"

**Output:**
```json
{
  "claim_id": "def-456",
  "category": "Office Supplies",
  "confidence": 0.45,
  "needs_review": true,
  "source": "llm",
}
```

Staff sees: **Office Supplies**  Needs review (45% confident) — [Accept] [Change]

### Example 3 — OCR failure (blurry photo)

**Input:** Blurry photo of a crumpled receipt, text unreadable

**OCR output:** Fails — Document Intelligence returns an error or empty text

**Output:**
```json
{
  "claim_id": "ghi-789",
  "category": "Other",
  "confidence": 0.0,
  "needs_review": true,
  "source": "default",
}
```

Staff sees: **Other** Could not read receipt — please select a category manually — [Change]

### Example 4 — LLM fallback (Azure OpenAI down)

**Input:** Receipt from Cinnamon Grand Hotel showing "Room charge 2 nights, LKR 45,000"

**OCR output:** "Cinnamon Grand Hotel Room Charge - 2 Nights Total: LKR 45,000"

Azure OpenAI times out. Rule-based fallback scans for keywords: "hotel", "room", "nights" - matches "Lodging".

**Output:**
```json
{
  "claim_id": "jkl-012",
  "category": "Lodging",
  "confidence": 0.7,
  "needs_review": false,
  "source": "rule-based",
}
```

Staff sees: **Lodging** (70% confident) — [Accept] [Change]

---
## 6. Out of scope
- **Multi-receipt batch categorisation** —v1 categorises one receipt at a time because the claimant must review and accept or change each suggestion individually before submitting. Batch categorisation would require a different UI flow for reviewing multiple suggestions at once, which adds design complexity beyond v1.
- **Auto-submission without claimant confirmation** — the spec requires the claimant to accept or change before submitting (FR3). Removing human review is a future consideration after acceptance rate data proves the model is reliable.
- **Training or fine-tuning a custom model** — v1 uses prompt-based classification with gpt-4.1. Fine-tuning requires labelled training data which the FR5 logging will collect over time, but is not in scope for this release.
- **Multi-language receipt support** — v1 assumes multi-language support (Tamil,Sinhala other languages) is deferred.

## 7. Open questions
- What confidence threshold should trigger "Needs review"?
- Do we want to learn from overrides (active learning) in v1?
- If users submit low confidence categories, should it be reviewed by finance?
- What happens if cost per suggestion exceeds LKR 5?
- What exact keywords does the rule-based fallback use?
- What if the receipt is in Sinhala or Tamil?
- How should the rule-based fallback calculate its confidence score? 
