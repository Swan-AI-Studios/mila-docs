---
title: Get Case Distribution
description: Aggregated counts of cases bucketed by one or more dimensions; same scope and filter rules as list cases.
sidebar_position: 9
---

# Get Case Distribution

```
GET /v2/cases/distribution
```

Returns aggregated counts of cases bucketed by one or more dimensions (`distributionBy`) — for example, by state, type, tag, or escalation type — applying the same scope and filter rules as `GET /v2/cases`. Useful for dashboard summaries.

## Authorities

`SCOPE_milaadmindevscope`, `SCOPE_miladevprovidersscope`, `APPROLE_Mila.ALL`, `APPROLE_Mila.PROVIDER`

## Request

**Query parameters**

| Name             | Type                       | Required | Notes                                                                  |
| ---------------- | -------------------------- | -------- | ---------------------------------------------------------------------- |
| `distributionBy` | `Set<DistributionType>`    | **yes**  | Dimensions to bucket by, e.g. `STATE`, `TYPE`, `TAG_NAME`, `ESCALATION_TYPE` |

Plus the same scope, patient, outcome plan, search, date, state, type, tag, escalation, and `isOpen` / `isClosed` filters as [List Cases](./list-cases). Pagination parameters are not used here.

## Response

`200 OK` — `CaseDistributionResponse`

```json
{
  "distributions": {
    "<dimension>": {
      "<bucket>": 0
    }
  }
}
```

Outer key is the distribution dimension; inner key is the bucket value; value is the count.

## Errors

| Status | Cause                                                                       |
| ------ | --------------------------------------------------------------------------- |
| 400    | `distributionBy=TAG_NAME` supplied without non-empty `tagNames`             |
| 400    | `distributionBy=ESCALATION_TYPE` supplied without non-empty `escalationTypes` |
| 400    | `patientId` and `patientMobilePhoneNumber` both supplied                    |
| 400    | `isOpen` or `isClosed` set to `false`, or both set to `true`                |
| 400    | `endDate` is not after `startDate`                                          |
| 401    | Caller identity cannot be determined                                        |
| 403    | Caller not authorized for the resolved scope                                |
