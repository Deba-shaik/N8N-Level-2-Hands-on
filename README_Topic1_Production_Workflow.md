# Topic 1: Production Workflow Planning & Design Patterns

## Overview
This n8n assessment demonstrates webhook ingestion, JavaScript-based transaction ID extraction and payload validation, a mock HTTP billing request, configured fixed-delay retries, and centralized failure notification through a separate error workflow.

**Assessment date:** 15 October 2026  
**Main workflow:** `Production Workflow Planning &Design Patterns`  
**Error workflow:** `Topic1_Centralized_Error_Handler`  
**Environment:** `https://n8n.tayanasolutions.com`

## Architecture

```text
POST /webhook/v1/transactions
  -> Webhook
  -> Idempotency Verification (Code)
  -> Validate Inputs (Code)
  -> POST CreateBillingCharge (HTTP Request)

On a failed automatic execution (after retries):
  Error Trigger [separate workflow]
  -> Construct Error Alert (Code)
  -> Send an Email (SMTP)
```

## Main workflow nodes

| Node | Purpose | Evidence / configuration |
|---|---|---|
| Webhook | Accept transaction JSON via HTTP POST | Path `v1/transactions`; immediate response |
| Idempotency Verification | Extract and check presence of `transactionId` | JavaScript Code node; **persistent deduplication not demonstrated** |
| Validate Inputs | Validate `customerEmail` and positive `amount` | JavaScript Code node; valid payload passed |
| POST CreateBillingCharge | Submit validated JSON payload to a test endpoint | `POST https://httpbin.org/post` during successful tests; intentionally invalid hostname during failure test |

### Example test payload

```json
{
  "transactionId": "TXN010",
  "customerEmail": "customer@example.com",
  "amount": 1500,
  "status": "success"
}
```

### Webhook testing

- Test endpoint: `https://n8n.tayanasolutions.com/webhook-test/v1/transactions` (requires **Listen for test event**).
- Production endpoint: `https://n8n.tayanasolutions.com/webhook/v1/transactions` (requires published workflow).
- Example Windows CMD command:

```cmd
curl -X POST "https://n8n.tayanasolutions.com/webhook/v1/transactions" -H "Content-Type: application/json" -d "{"transactionId":"TXN010","customerEmail":"customer@example.com","amount":1500,"status":"success"}"
```

A `{"message":"Workflow was started"}` response confirms acceptance, not completion. Review **Executions** to see the final status.

## HTTP Request and retry configuration

- Method: POST; JSON request body from `{{ $json.payload }}`.
- Retry On Fail: enabled.
- Max Tries: 4 total attempts (initial attempt plus three retries).
- Wait Between Tries: 5000 ms.
- On Error: Stop Workflow, so a failed automatic execution can invoke the configured Error Workflow.
- **Limitation:** these node settings implement a fixed wait, not the exponential 5/10/20-second backoff requested by the implementation plan.
- **Limitation:** the real billing endpoint and production credential-manager authentication were not demonstrated; httpbin.org was used as a mock receiver.

## Centralized error handler

1. Create a separate workflow named `Topic1_Centralized_Error_Handler`.
2. Add **Error Trigger** -> **Construct Error Alert** (Code) -> **Send an Email**.
3. Extract workflow name, execution ID, failed node, error message and timestamp.
4. Configure a working SMTP credential and recipient in the email node; never include secrets in documentation.
5. In the main workflow's **Settings**, select the handler under **Error Workflow** and save/publish.
6. Trigger a **production webhook** execution that intentionally fails. Manual node tests do not prove automatic Error Trigger integration.

## Demonstrated evidence

| Evidence | Observation | Result |
|---|---|---|
| Main workflow failure | 15 Oct 2026, 14:41:36; execution `500454`; failed after 15.074 seconds at `POST CreateBillingCharge` | PASS - intentional failure |
| Error handler execution | 15 Oct 2026, 14:41:51; execution `500455`; succeeded in 3.019 seconds; all three nodes green | PASS |
| Email notification | Subject `n8n Production Workflow Failed`; includes workflow, execution ID `500454`, failed node, hostname error, and timestamp | PASS - received |

The hostname failure was intentionally produced using `https://invalid.example.invalid/test`. Restore `https://httpbin.org/post` and republish after testing.

## Requirement coverage / open work

| Requirement | Status |
|---|---|
| Webhook POST ingestion | Demonstrated |
| Valid payload through validation and mock API | Demonstrated in earlier execution screenshots |
| Centralized error workflow invoked after failed production run | Demonstrated |
| Email notification with useful error context | Demonstrated |
| Configured fixed-delay retries | Configured; execution duration is consistent, but individual retry timestamps were not independently verified |
| Persistent 24-hour idempotency with Redis/database | Not demonstrated; transaction ID extraction alone is insufficient |
| Exponential backoff and jitter | Not demonstrated |
| Real billing API credential-manager integration | Not demonstrated |
| Explicit 400 response for invalid payload | Not demonstrated; immediate webhook response may already have been sent |
| Slack incident alert / execution link | Not demonstrated; email alternative used |
| Database persistence, pruning, queue setup, sticky notes | Not verified from screenshots |

## Suggested additional tests

- Submit a missing/invalid email, nonpositive amount, and missing transaction ID; verify expected handling.
- Submit the same transaction ID twice and verify a persistent deduplication store prevents repeated billing.
- Mock HTTP 429/502/503 and record exact retry intervals, then implement exponential backoff if required.
- Test authentication failure and verify secrets remain only in n8n Credential Manager.
- Verify production webhook response behavior separately from asynchronous execution outcome.

## Evidence file
See `Topic1_Workflow_Evidence.pdf` for the three provided screenshots and observations.

## Source documents
- `TOPIC 1 imp.docx` — implementation plan.
- `top re1.docx` — requirement file, including BRD/PRD/SRS and test cases.
