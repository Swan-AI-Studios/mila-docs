---
title: Cancel Case
description: Transition a case to the CANCELED terminal state.
sidebar_position: 5
---

# Cancel Case

```
PUT /v2/cases/{caseId}/cancel
```

Transitions the case to the `CANCELED` terminal state. The case must be in a state from which cancellation is allowed. Caller must have `WRITE` access to the case's organization group.

## Authorities

`SCOPE_milaadmindevscope`, `SCOPE_miladevprovidersscope`, `APPROLE_Mila.ALL`, `APPROLE_Mila.PROVIDER`

## Request

**Path parameters**

| Name     | Type     | Required | Description           |
| -------- | -------- | -------- | --------------------- |
| `caseId` | `CaseId` | yes      | Identifier of the case |

**Query parameters**

| Name                  | Type                  | Required | Description                              |
| --------------------- | --------------------- | -------- | ---------------------------------------- |
| `organizationGroupId` | `OrganizationGroupId` | no       | Used for authorization for provider callers |

## Response

`200 OK` — `UpdateCaseResponse` (full updated case projection).

## Errors

| Status | Cause                                                       |
| ------ | ----------------------------------------------------------- |
| 400    | Illegal state transition                                    |
| 403    | Caller lacks write access for the case's organization group |
| 404    | Case not found                                              |