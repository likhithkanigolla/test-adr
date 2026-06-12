# TAD-ADR-004: TAD-ADR-001: Title: Tenant Registration API Migration

**Status:** approved
**Date:** 2026-06-12

## Context

The existing tenant onboarding process (V1) is implemented to handle tenant registration. As the platform evolves to support multi-tenancy, onboarding requests has been migrated to APIs exposed under `user_management`(V2). The existing onboarding API (`POST /onboard/user-request`) creates a tenant onboarding request and returns a pending status. Subsequent onboarding actions are performed using tenant-specific information. As the platform evolves to support multi-tenancy, onboarding operations are being migrated to APIs exposed under `user_management`. The new registration API creates an onboarding request resource and returns a unique `request_id`

## Decision

- Migrate tenant onboarding operations from the existing onboarding APIs to the new APIs exposed under the `user_management`.`.
- Tenant registration creates an onboarding request and returns a unique `request_id`.
- Unlike V1, which returned only a pending status, V2 exposes the onboarding request identifier so that the onboarding request must be approved or rejected.

## Consequences

- API consumers must migrate to the new endpoints and adopt request-based operations using request_id.
- Registration no longer directly creates a tenant; it creates a pending onboarding request that follows an approval or rejection workflow.
