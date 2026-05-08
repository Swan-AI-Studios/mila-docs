---
title: Cases
description: Create, read, update, and search cases; manage escalations.
sidebar_position: 1
slug: /cases
---

# Cases API

Base path: `/v2/cases` (configurable via `mila-case-service.api.version-path`).

| Method | Path                                                              | Operation                                                                  |
| ------ | ----------------------------------------------------------------- | -------------------------------------------------------------------------- |
| POST   | `/v2/cases`                                                       | [Create case](./create-case)                                               |
| GET    | `/v2/cases/{caseId}`                                              | [Get case by ID](./get-case-by-id)                                         |
| GET    | `/v2/cases/{caseId}/conversation-interaction`                     | [Get in-progress conversation interaction](./get-conversation-interaction) |
| GET    | `/v2/cases/{caseId}/interactions/{interactionId}/audio`           | [Get interaction audio](./get-interaction-audio)                           |
| PUT    | `/v2/cases/{caseId}/cancel`                                       | [Cancel case](./cancel-case)                                               |
| PUT    | `/v2/cases/{caseId}/complete`                                     | [Complete case](./complete-case)                                           |
| PUT    | `/v2/cases/{caseId}/close`                                        | [Close case](./close-case)                                                 |
| GET    | `/v2/cases`                                                       | [List / search cases](./list-cases)                                        |
| GET    | `/v2/cases/distribution`                                          | [Get case distribution](./get-case-distribution)                           |
| PUT    | `/v2/cases/{caseId}/escalations/{escalationId}/status`            | [Update escalation status](./update-escalation-status)                     |