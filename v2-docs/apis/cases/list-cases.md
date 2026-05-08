---
title: List / Search Cases
description: Paged list of cases visible to the caller, with rich filtering by scope, patient, time range, state, type, tags, and escalations.
sidebar_position: 8
---

# List / Search Cases

```
GET /v2/cases
```

Returns a paged list of cases visible to the caller, with optional filtering by scope, patient, outcome plan, free-text search, time range, state, type, tags, and escalation type. The service derives an effective scope from the most specific scope identifier provided (`providerId` > `organizationId` > `organizationGroupId` > `adminGroupId`) and falls back to the caller's identity (admin user or authenticated provider) when none is supplied. Authorization is enforced for whichever scope is resolved.

## Authorities

`SCOPE_milaadmindevscope`, `SCOPE_miladevprovidersscope`, `APPROLE_Mila.ALL`, `APPROLE_Mila.PROVIDER`

## Request

**Pagination**

| Name   | Type    | Required | Description                                   |
| ------ | ------- | -------- | --------------------------------------------- |
| `page` | integer | no       | Zero-based page index                         |
| `size` | integer | no       | Page size (default `25`)                      |
| `sort` | string  | no       | `field,asc` or `field,desc` (Spring Pageable) |

**Scope** *(most specific wins; falls back to caller identity if none provided)*

| Name                  | Type                  | Required |
| --------------------- | --------------------- | -------- |
| `adminGroupId`        | `AdminGroupId`        | no       |
| `organizationGroupId` | `OrganizationGroupId` | no       |
| `organizationId`      | `OrganizationId`      | no       |
| `providerId`          | `ProviderId`          | no       |

**Patient filters** *(at most one)*

| Name                       | Type          | Required |
| -------------------------- | ------------- | -------- |
| `patientId`                | `PatientId`   | no       |
| `patientMobilePhoneNumber` | `PhoneNumber` | no       |

**Other filters**

| Name                   | Type                       | Required | Notes                                                                        |
| ---------------------- | -------------------------- | -------- | ---------------------------------------------------------------------------- |
| `outcomePlanId`        | `OutcomePlanId`            | no       |                                                                              |
| `searchText`           | string                     | no       | Free-text search                                                             |
| `startDate`            | string                     | no       | ISO-8601 with offset/Z, or `YYYY-MM-DD` (inclusive lower bound)              |
| `endDate`              | string                     | no       | ISO-8601 with offset/Z, or `YYYY-MM-DD` (exclusive upper bound, next-day-start) |
| `states`               | `Set<CaseState>`           | no       | Explicit states to filter by                                                 |
| `isOpen`               | boolean                    | no       | Only `true` is allowed (or omit) — uses `CaseState.OPEN_STATES`              |
| `isClosed`             | boolean                    | no       | Only `true` is allowed (or omit) — uses `CaseState.CLOSED_STATES`            |
| `types`                | `Set<CaseType>`            | no       |                                                                              |
| `tagNames`             | `Set<string>`              | no       |                                                                              |
| `tagMatchMode`         | `ANY` \| `ALL`             | no       | Default `ANY`                                                                |
| `escalationTypes`      | `Set<EscalationType>`      | no       |                                                                              |
| `escalationMatchMode`  | `ANY` \| `ALL`             | no       | Default `ANY`                                                                |

## Response

`200 OK` — `GetCasesResponse`

```json
{
  "cases": [ /* List<CaseResponse> */ ],
  "totalCount": 0,
  "pageNumber": 0,
  "pageSize": 25
}
```

## Errors

| Status | Cause                                                                                |
| ------ | ------------------------------------------------------------------------------------ |
| 400    | `patientId` and `patientMobilePhoneNumber` both supplied                             |
| 400    | `isOpen` or `isClosed` set to `false`, or both set to `true`                         |
| 400    | `endDate` is not after `startDate`                                                   |
| 400    | `startDate` / `endDate` not ISO-8601 with offset/Z or `YYYY-MM-DD`                   |
| 401    | Caller identity cannot be determined                                                 |
| 403    | Caller not authorized for the resolved scope                                         |