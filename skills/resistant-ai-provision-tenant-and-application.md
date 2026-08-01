---
name: Provision a tenant and application
description: Create a tenant and an application under it using the Resistant Documents Tenant Management API.
api: openapi/resistant-ai-tenant-management-openapi.json
operations: [createTenant, createTenantApplication, getTenantApplications]
---

# Provision a tenant and application

Set up the organizational boundary (tenant) and the credential-holding application that will call the Documents API.

## Prerequisites
- OAuth2 client credentials authorized for the Tenant Management API (base `https://{environment}.resistant.ai/v0`).

## Steps

1. **Get an access token** (client-credentials, reuse it).
2. **Create the tenant** — `createTenant` (`POST /tenants`). Returns `201` with the new `tenant_id`.
3. **Create an application under the tenant** — `createTenantApplication` (`POST /tenants/{tenant_id}/applications`). Returns `201` with the new `application_id`.
4. **Verify** — `getTenantApplications` (`GET /tenants/{tenant_id}/applications`) to confirm the application is registered.

## Rules
- Rate limits, event destinations and retention are scoped **per tenant** — see `data-model/resistant-ai-data-model.yml`.
- Errors use `{ "message": "..." }` (application/json). `404` = tenant/application not found, `403` = insufficient scope. See `errors/resistant-ai-problem-types.yml`.
- Clean up with `deleteTenantApplication` / `deleteTenant` (both return `204`).
