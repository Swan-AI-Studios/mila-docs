---
title: Create Case Ingestion Job
description: Submit a new bulk ingestion job by uploading an input file and JSON ingestion request.
sidebar_position: 1
---

# Create Case Ingestion Job

```
POST /v1/case-ingestion-jobs
```

Submits a new bulk ingestion job. The caller uploads a multipart form containing the JSON ingestion request and the input data file. The service merges the submitted parameters with system defaults, persists the file, kicks off asynchronous ingestion, and returns the created job descriptor.

## Authorities

`SCOPE_miladevprovidersscope`, `APPROLE_Mila.ALL`, `APPROLE_Mila.PROVIDER`

## Request

**Content-Type:** `multipart/form-data`

**Form parts**

| Part                | Type   | Required | Description                                                                  |
| ------------------- | ------ | -------- | ---------------------------------------------------------------------------- |
| `ingestionRequest`  | string | yes      | JSON-serialized `CreateCaseIngestionJobRequest` (see below)                  |
| `inputFile`         | file   | yes      | The bulk-input file to be ingested                                           |

**`CreateCaseIngestionJobRequest`**

| Field                 | Type                          | Required | Notes                                       |
| --------------------- | ----------------------------- | -------- | ------------------------------------------- |
| `organizationGroupId` | `OrganizationGroupId`         | yes      |                                             |
| `parameters`          | `CaseIngestionJobParameters`  | no       | Overrides for default ingestion behavior    |

`type` is fixed to `CASE_INGESTION` server-side and does not need to be supplied.

## Response

`201 Created` — `CaseIngestionJob` (see [Get Case Ingestion Job](./get-ingestion-job) for the full field list).

## Errors

| Status | Cause                                                                  |
| ------ | ---------------------------------------------------------------------- |
| 400    | `ingestionRequest` JSON is unparseable or required fields are missing  |
