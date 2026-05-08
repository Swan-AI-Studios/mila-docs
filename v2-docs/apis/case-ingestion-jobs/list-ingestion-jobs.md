---
title: List Case Ingestion Jobs
description: Paged list of ingestion jobs for an organization group.
sidebar_position: 3
---

# List Case Ingestion Jobs

```
GET /v1/case-ingestion-jobs
```

Returns a paged list of ingestion jobs for an organization group, with optional filtering by status and time window.

## Authorities

`SCOPE_miladevprovidersscope`, `APPROLE_Mila.ALL`, `APPROLE_Mila.PROVIDER`

## Request

**Query parameters**

| Name                  | Type                  | Required | Notes                                                                                                                          |
| --------------------- | --------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------ |
| `organizationGroupId` | `OrganizationGroupId` | **yes**  | Organization group whose jobs are listed                                                                                       |
| `status`              | `JobStatus`           | no       | Filter by job status                                                                                                           |
| `startDate`           | string                | no       | ISO-8601 with offset/Z, or `YYYY-MM-DD` (inclusive lower bound)                                                                |
| `endDate`             | string                | no       | ISO-8601 with offset/Z, or `YYYY-MM-DD` (exclusive upper bound). Defaults to "now" if `startDate` is supplied and `endDate` omitted |
| `pageNumber`          | integer               | no       |                                                                                                                                |
| `pageSize`            | integer               | no       |                                                                                                                                |

## Response

`200 OK` — `ListCaseIngestionJobsResponse`

```json
{
  "jobs": [ /* List<CaseIngestionJob> */ ],
  "totalCount": 0,
  "pageNumber": 0,
  "pageSize": 25
}
```

## Errors

| Status | Cause                                                                  |
| ------ | ---------------------------------------------------------------------- |
| 400    | `endDate` is not after `startDate`                                     |
| 400    | Invalid date format (must be ISO-8601 with offset/Z or `YYYY-MM-DD`)   |