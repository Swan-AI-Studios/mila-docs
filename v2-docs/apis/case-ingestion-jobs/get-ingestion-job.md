---
title: Get Case Ingestion Job
description: Fetch the status and metadata of a previously created ingestion job.
sidebar_position: 2
---

# Get Case Ingestion Job

```
GET /v1/case-ingestion-jobs/{jobId}
```

Returns the current status and metadata of a previously created ingestion job, including counts of total / succeeded / failed records and pointers to result files.

## Authorities

`SCOPE_miladevprovidersscope`, `APPROLE_Mila.ALL`, `APPROLE_Mila.PROVIDER`

## Request

**Path parameters**

| Name    | Type    | Required | Description           |
| ------- | ------- | -------- | --------------------- |
| `jobId` | `JobId` | yes      | Identifier of the job |

**Query parameters**

| Name                  | Type                  | Required | Description                                  |
| --------------------- | --------------------- | -------- | -------------------------------------------- |
| `organizationGroupId` | `OrganizationGroupId` | **yes**  | Organization group that owns the job         |

## Response

`200 OK` — `CaseIngestionJob`

| Field                       | Type                          | Description                                  |
| --------------------------- | ----------------------------- | -------------------------------------------- |
| `jobId`                     | `JobId`                       | Identifier of the job                        |
| `organizationGroupId`       | `OrganizationGroupId`         | Owning organization group                    |
| `jobExecutionId`            | `JobExecutionId`              | Identifier of the underlying execution       |
| `type`                      | `JobType`                     | Always `CASE_INGESTION`                      |
| `status`                    | `JobStatus`                   | Current status                               |
| `fileName`                  | string                        | Original uploaded file name                  |
| `inputFilePath`             | string                        | Path to the stored input file                |
| `outputFilePath`            | string                        | Path to the combined output file             |
| `succeededRecordsFilePath`  | string                        | Path to the succeeded-records output file    |
| `failedRecordsFilePath`     | string                        | Path to the failed-records output file       |
| `totalRecords`              | long                          | Total record count                           |
| `totalSucceededRecords`     | long                          | Successfully ingested records                |
| `totalFailedRecords`        | long                          | Failed records                               |
| `parameters`                | `CaseIngestionJobParameters`  | Effective parameters used for the run        |
| `createdAt`                 | string                        | ISO timestamp                                |
| `lastModifiedAt`            | string                        | ISO timestamp                                |

## Errors

| Status | Cause                                                       |
| ------ | ----------------------------------------------------------- |
| 403    | Caller not authorized for the supplied organization group   |
| 404    | Job not found                                               |