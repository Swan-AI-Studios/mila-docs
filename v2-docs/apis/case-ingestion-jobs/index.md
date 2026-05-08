---
title: Case Ingestion Jobs
description: Bulk-upload cases via asynchronous ingestion jobs.
sidebar_position: 2
slug: /case-ingestion-jobs
---

# Case Ingestion Jobs API

Base path: `/v1/case-ingestion-jobs`.

These endpoints support bulk ingestion of cases from an uploaded file and inspection of the resulting jobs.

| Method | Path                                | Operation                                                |
| ------ | ----------------------------------- | -------------------------------------------------------- |
| POST   | `/v1/case-ingestion-jobs`           | [Create case ingestion job](./create-ingestion-job)      |
| GET    | `/v1/case-ingestion-jobs/{jobId}`   | [Get case ingestion job](./get-ingestion-job)            |
| GET    | `/v1/case-ingestion-jobs`           | [List case ingestion jobs](./list-ingestion-jobs)        |
