---
title: Provider API Reference
description: HTTP endpoints in mila-case-service accessible to callers holding the APPROLE_Mila.PROVIDER authority.
sidebar_position: 1
slug: /
---

# Mila Case Service — Provider API Reference

This reference documents every HTTP endpoint in `mila-case-service` that is accessible to callers holding the `APPROLE_Mila.PROVIDER` authority. These endpoints are intended for use by Provider applications authenticating via Azure AD client-credential or user JWT tokens that carry the `Mila.PROVIDER` app role.

## Authorization model

All endpoints in this reference are guarded by a Spring Security `@PreAuthorize` annotation that includes `APPROLE_Mila.PROVIDER` in its allowed authorities. The full set of authorities accepted by a given endpoint is shown in the **Authorities** field of each entry. Common authorities seen in this service:

| Authority                     | Description                                                                                       |
| ----------------------------- | ------------------------------------------------------------------------------------------------- |
| `APPROLE_Mila.PROVIDER`       | Provider client-credential token. Caller is scoped to its own organization group.                 |
| `APPROLE_Mila.ALL`            | Mila service-to-service client-credential token. Pre-approved, full access.                       |
| `SCOPE_miladevprovidersscope` | Provider user JWT scope. Caller is an authenticated provider user.                                |
| `SCOPE_milaadmindevscope`     | Mila admin user JWT scope. Reserved for future admin use.                                         |

In addition to authority checks, most endpoints perform `AccessValidator` checks at the organization-group / organization / provider level so a `Mila.PROVIDER` token cannot reach data outside its own scope.

## API groups

| Group                                                | Base path                  | Description                                                  |
| ---------------------------------------------------- | -------------------------- | ------------------------------------------------------------ |
| [Cases](./cases/)                                    | `/v2/cases`                | Create, read, update, and search cases; manage escalations.  |
| [Case Ingestion Jobs](./case-ingestion-jobs/)        | `/v1/case-ingestion-jobs`  | Bulk-upload cases via asynchronous ingestion jobs.           |

The `/v2` prefix on Cases is configurable via the `mila-case-service.api.version-path` property.

## Common error model

All endpoints surface validation and authorization errors through the service's standard exception types, which map to HTTP status codes as follows:

| Exception                       | HTTP status | Typical cause                                         |
| ------------------------------- | ----------- | ----------------------------------------------------- |
| `InvalidInputException`         | 400         | Missing/invalid query, path, or body parameters       |
| `InvalidRequestException`       | 400         | Semantic validation failure on a request              |
| `UnauthorizedAccessException`   | 401         | Caller identity cannot be determined                  |
| `AccessDeniedException`         | 403         | Caller authenticated but not authorized for resource  |
| `ResourceNotFoundException`     | 404         | Target case / interaction / job does not exist        |

Provider clients should treat `403` as a hard scope boundary — supplying an `organizationGroupId` (or other scope identifier) outside the caller's tenant will result in a denial rather than an empty result.
