---
title: Get In-Progress Conversation Interaction
description: Fetch the currently active conversation interaction for a case, when one exists.
sidebar_position: 3
---

# Get In-Progress Conversation Interaction

```
GET /v2/cases/{caseId}/conversation-interaction
```

Returns the currently active (in-progress) conversation interaction for a case, when one exists. Used by Provider clients to drive a live conversational UI.

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

## Responses

| Status        | Body          | When                                                  |
| ------------- | ------------- | ----------------------------------------------------- |
| `200 OK`      | `Interaction` | An in-progress interaction exists                     |
| `202 Accepted`| empty         | No interaction is currently in progress for this case |

`Interaction` includes: `interactionId`, `type`, `communicationChannel`, `startTime`, `endTime`, `status`, `messages`, `metadata`, `evaluation`, `summary`.

## Errors

| Status | Cause                                                       |
| ------ | ----------------------------------------------------------- |
| 403    | Caller not authorized for the case's organization group     |
| 404    | Case not found                                              |