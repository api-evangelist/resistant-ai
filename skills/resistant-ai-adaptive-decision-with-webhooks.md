---
name: Run Adaptive Decision with webhook notification
description: Submit a document with Adaptive Decision enabled and consume the completion event via a Svix webhook instead of polling.
api: openapi/resistant-ai-documents-openapi.json
operations: [createSubmission, getDecision]
---

# Run Adaptive Decision with webhook notification

Get an automated accept/review/reject-style Adaptive Decision and be notified over a webhook when it completes.

## Prerequisites
- OAuth2 client credentials (`submissions.read`, `submissions.write`).
- Webhooks are an optional add-on (prior agreement). Resistant AI (via **Svix**) delivers events to a public HTTPS callback URL you provide, with a signing secret configured at enablement.

## Steps

1. **Get an access token** (client-credentials, reuse it).
2. **Create the submission with decision enabled** — `createSubmission` (`POST /v2/submission`) with `{ "query_id": "<opaque-id>", "enable_decision": true }`. Adaptive Decision result endpoints return `400` if `enable_decision` was not set true.
3. **Upload the file bytes** to the returned `upload_url` (`application/octet-stream`).
4. **Receive the webhook.** Svix sends a `POST` with headers `webhook-id`, `webhook-timestamp`, `webhook-signature` and body `{ type: "documents.adaptive_decision.finished", version, tenant_id, payload: { id, status, result_url } }`. **Verify the Svix signature** and validate timestamp tolerance before trusting the event.
5. **Fetch the authoritative result** — `getDecision` (`GET /v2/submission/{submission_id}/decision`). The webhook is a notification only; always fetch the full decision from the API.

## Rules
- **Idempotency:** webhooks are at-least-once. Deduplicate on the `webhook-id` header or the `(submission_id, event_type, version)` tuple. See `conventions/resistant-ai-conventions.yml`.
- **Fallback:** if webhooks are not enabled, poll `getDecision` with the standard backoff (cap 45s, stop at 15 min). Fraud/quality/classification notifications use Amazon SQS or polling — see `asyncapi/resistant-ai-documents-webhooks.yml`.
- **Errors:** `409` on `getReport` means the analysis/decision did not complete successfully. See `errors/resistant-ai-problem-types.yml`.
