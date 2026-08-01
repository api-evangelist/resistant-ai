---
name: Analyze a document for fraud
description: Submit a PDF or image to the Resistant Documents API and retrieve the fraud verdict using the create-upload-poll workflow.
api: openapi/resistant-ai-documents-openapi.json
operations: [createSubmission, getFraud]
---

# Analyze a document for fraud

Detect document fraud, manipulation, and reuse in a PDF or image with the Resistant Documents API.

## Prerequisites
- OAuth2 **Client ID** and **Client Secret** provisioned for your tenant.
- The base URL for your tenant's cell (default EU: `https://api.documents.resistant.ai`); use `https://api.documents.testing.resistant.ai` for testing.

## Steps

1. **Get an access token.** POST to your cell token URL with HTTP Basic auth (`client_id:client_secret`), `grant_type=client_credentials`, `scope=submissions.read submissions.write`. Reuse the token across calls — do not request one per request.
2. **Create the submission** — `createSubmission` (`POST /v2/submission`). Body: `{ "query_id": "<opaque-internal-id>", "pipeline_configuration": "FRAUD_ONLY" }`. `query_id` must contain no PII. The response returns `submission_id` and a presigned `upload_url`.
3. **Upload the file bytes.** `PUT` the raw file to `upload_url` with `Content-Type: application/octet-stream` (`--data-binary`). This is a storage upload, not an API operation.
4. **Poll the fraud result** — `getFraud` (`GET /v2/submission/{submission_id}/fraud`). `200` = finished (success or terminal error in the payload); `404` = not ready, retry.

## Rules
- **Polling:** exponential backoff, cap the interval at 45s, stop after the 15-minute hard timeout. Treat a timeout as a terminal failure to surface for investigation.
- **Rate limits (Default tier):** create-submission 4 req/s (6 burst), other requests 10 req/s (15 burst), per tenant. On `429`, back off with jitter — see `rate-limits/resistant-ai-rate-limits.yml`.
- **Errors:** responses use `{ "message": "..." }` (application/json, not RFC 9457). See `errors/resistant-ai-problem-types.yml` (`400` bad request / feature not enabled, `409` analysis not completed).
- **Retention:** results are retained 90 days by default; `deleteSubmission` (`DELETE /v2/submission/{submission_id}`) to delete earlier.
