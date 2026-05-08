---
title: Get Case by ID
description: Fetch the full case projection for a single case.
sidebar_position: 2
---

# Get Case by ID

```
GET /v2/cases/{caseId}
```

Returns the full `CaseResponse` for a single case. Provider callers should include `organizationGroupId` so the service can authorize the read against the caller's scope.

## Authorities

`SCOPE_milaadmindevscope`, `SCOPE_miladevprovidersscope`, `APPROLE_Mila.ALL`, `APPROLE_Mila.PROVIDER`

## Request

**Path parameters**

| Name     | Type     | Required | Description           |
| -------- | -------- | -------- | --------------------- |
| `caseId` | `CaseId` | yes      | Identifier of the case |

**Query parameters**

| Name                  | Type                  | Required | Description                                              |
| --------------------- | --------------------- | -------- | -------------------------------------------------------- |
| `organizationGroupId` | `OrganizationGroupId` | no       | Expected for provider callers; used for authorization    |

## Response

`200 OK` — `CaseResponse`

The full case projection includes patient, provider, outcome plan, codes, escalations, interactions, tags, and timestamps.

## Errors

| Status | Cause                                                       |
| ------ | ----------------------------------------------------------- |
| 403    | Caller not authorized for the case's organization group     |
| 404    | Case not found                                              |