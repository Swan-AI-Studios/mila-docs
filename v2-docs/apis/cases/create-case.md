---
title: Create Case
description: Create a new Case for a patient under a given outcome plan and provider.
sidebar_position: 1
---

# Create Case

```
POST /v2/cases
```

Creates a new Case for a patient under a given outcome plan and provider. The service determines and stamps the `organizationId`, `organizationGroupId`, and `adminGroupId` based on the supplied `providerId`, then validates that the caller is authorized to write to that organization group and that the organization group has access to the requested outcome plan.

## Authorities

`SCOPE_milaadmindevscope`, `SCOPE_miladevprovidersscope`, `APPROLE_Mila.ALL`, `APPROLE_Mila.PROVIDER`

## Request

**Content-Type:** `application/json`

**Body:** `CreateCaseRequest`

| Field                  | Type                              | Required | Notes                                  |
| ---------------------- | --------------------------------- | -------- | -------------------------------------- |
| `type`                 | `CaseType`                        | no       | Defaults to `ESTABLISHED`              |
| `name`                 | string                            | no       |                                        |
| `outcomePlanId`        | `VersionedId<OutcomePlanId>`      | yes      |                                        |
| `providerId`           | `ProviderId`                      | yes      | Determines the case's organization     |
| `patient`              | `CreatePatientRequest`            | yes      |                                        |
| `externalId`           | string                            | no       |                                        |
| `communicationChannel` | `CommunicationChannel`            | no       |                                        |
| `state`                | `CaseState`                       | no       | Optional initial state                 |
| `initiationType`       | `InitiationType`                  | no       |                                        |
| `preferredLanguage`    | `Language`                        | no       | Defaults to system default             |
| `icdCodes`             | `Set<IcdCode>`                    | no       |                                        |
| `cptCodes`             | `Set<CptCode>`                    | no       |                                        |
| `cdtCodes`             | `Set<CdtCode>`                    | no       |                                        |
| `appointments`         | `List<BaseAppointment>`           | no       |                                        |
| `medications`          | `List<Medication>`                | no       |                                        |
| `caseSpecificData`     | JSON                              | no       |                                        |
| `userSuppliedData`     | JSON                              | no       |                                        |

## Response

`201 Created` — `CreateCaseResponse`

```json
{
  "id": "<CaseId>"
}
```

## Errors

| Status | Cause                                                                                                          |
| ------ | -------------------------------------------------------------------------------------------------------------- |
| 400    | Invalid request body                                                                                           |
| 403    | Caller not authorized for the resolved organization group, or organization group has no access to the outcome plan |
