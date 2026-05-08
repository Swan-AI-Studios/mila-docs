---
title: Update Escalation Status
description: Update the status of a specific escalation on a case.
sidebar_position: 10
---

# Update Escalation Status

```
PUT /v2/cases/{caseId}/escalations/{escalationId}/status
```

Updates the status of a specific escalation on a case (e.g., acknowledging, resolving, or dismissing it) and optionally records reviewer notes. Caller must have `WRITE` access to the case's organization group.

## Authorities

`SCOPE_miladevprovidersscope`, `APPROLE_Mila.ALL`, `APPROLE_Mila.PROVIDER`

## Request

**Content-Type:** `application/json`

**Path parameters**

| Name           | Type           | Required |
| -------------- | -------------- | -------- |
| `caseId`       | `CaseId`       | yes      |
| `escalationId` | `EscalationId` | yes      |

**Query parameters**

| Name                  | Type                  | Required | Description                              |
| --------------------- | --------------------- | -------- | ---------------------------------------- |
| `organizationGroupId` | `OrganizationGroupId` | no       | Used for authorization for provider callers |

**Body:** `UpdateEscalationStatusRequest`

| Field    | Type                | Required |
| -------- | ------------------- | -------- |
| `status` | `EscalationStatus`  | yes      |
| `notes`  | string              | no       |

## Response

`200 OK` — `CaseResponse` (the case with the escalation updated).

## Errors

| Status | Cause                                                       |
| ------ | ----------------------------------------------------------- |
| 400    | Invalid status or body                                      |
| 403    | Caller lacks write access for the case's organization group |
| 404    | Case or escalation not found                                |