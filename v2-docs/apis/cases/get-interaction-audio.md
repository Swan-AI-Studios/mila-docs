---
title: Get Interaction Audio
description: Fetch a signed URL to the recorded audio for a specific interaction.
sidebar_position: 4
---

# Get Interaction Audio

```
GET /v2/cases/{caseId}/interactions/{interactionId}/audio
```

Returns a signed URL to the recorded audio for a specific interaction. The URL is short-lived and intended for direct retrieval by the Provider client.

## Authorities

`SCOPE_milaadmindevscope`, `SCOPE_miladevprovidersscope`, `APPROLE_Mila.ALL`, `APPROLE_Mila.PROVIDER`

## Request

**Path parameters**

| Name            | Type            | Required | Description                          |
| --------------- | --------------- | -------- | ------------------------------------ |
| `caseId`        | `CaseId`        | yes      | Identifier of the case               |
| `interactionId` | `InteractionId` | yes      | Identifier of the interaction        |

## Response

`200 OK` — `InteractionAudioResponse`

```json
{
  "caseId": "<CaseId>",
  "interactionId": "<InteractionId>",
  "interactionAudioUrl": "https://..."
}
```

## Errors

| Status | Cause                                                       |
| ------ | ----------------------------------------------------------- |
| 400    | Invalid request                                             |
| 404    | Case or interaction not found, or audio not available       |