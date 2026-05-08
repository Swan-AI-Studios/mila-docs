# Mila Case Service — Provider API Reference

This reference documents every HTTP endpoint in `mila-case-service` that is accessible to callers holding the `APPROLE_Mila.PROVIDER` authority. These endpoints are intended for use by Provider applications authenticating via Azure AD client-credential or user JWT tokens that carry the `Mila.PROVIDER` app role.

## Authorization model

All endpoints listed below are guarded by a Spring Security `@PreAuthorize` annotation that includes `APPROLE_Mila.PROVIDER` in its allowed authorities. The full set of authorities accepted by a given endpoint is shown in the **Authorities** field of each entry. Common authorities seen in this service:

| Authority                     | Description                                                                                       |
| ----------------------------- | ------------------------------------------------------------------------------------------------- |
| `APPROLE_Mila.PROVIDER`       | Provider client-credential token. Caller is scoped to its own organization group.                 |
| `APPROLE_Mila.ALL`            | Mila service-to-service client-credential token. Pre-approved, full access.                       |
| `SCOPE_miladevprovidersscope` | Provider user JWT scope. Caller is an authenticated provider user.                                |
| `SCOPE_milaadmindevscope`     | Mila admin user JWT scope. Reserved for future admin use.                                         |

In addition to authority checks, most endpoints perform `AccessValidator` checks at the organization-group / organization / provider level so a `Mila.PROVIDER` token cannot reach data outside its own scope.

## Base paths

| Controller                     | Base path                |
| ------------------------------ | ------------------------ |
| `CaseController`               | `/v2/cases`              |
| `CaseIngestionJobController`   | `/v1/case-ingestion-jobs`|

The `/v2` prefix is configurable via the `mila-case-service.api.version-path` property.

---

## Cases API (`/v2/cases`)

### 1. Create Case

`POST /v2/cases`

Creates a new Case for a patient under a given outcome plan and provider. The service determines and stamps the `organizationId`, `organizationGroupId`, and `adminGroupId` based on the supplied `providerId`, then validates that the caller is authorized to write to that organization group and that the organization group has access to the requested outcome plan.

* **Authorities:** `SCOPE_milaadmindevscope`, `SCOPE_miladevprovidersscope`, `APPROLE_Mila.ALL`, `APPROLE_Mila.PROVIDER`
* **Content-Type:** `application/json`
* **Request body:** `CreateCaseRequest`
  * `type` *(CaseType, default `ESTABLISHED`)*
  * `name` *(string)*
  * `outcomePlanId` *(VersionedId<OutcomePlanId>, required)*
  * `providerId` *(ProviderId, required)*
  * `patient` *(CreatePatientRequest, required)*
  * `externalId` *(string, optional)*
  * `communicationChannel` *(enum)*
  * `state` *(CaseState, optional initial state)*
  * `initiationType` *(enum)*
  * `preferredLanguage` *(Language, default per system)*
  * `icdCodes`, `cptCodes`, `cdtCodes` *(sets)*
  * `appointments`, `medications` *(lists)*
  * `caseSpecificData`, `userSuppliedData` *(JSON nodes)*
* **Response:** `201 Created` — `CreateCaseResponse { id: CaseId }`
* **Error responses:**
  * `400` — invalid request body
  * `403` — caller not authorized for the resolved organization group, or organization group has no access to the outcome plan

---

### 2. Get Case by ID

`GET /v2/cases/{caseId}`

Returns the full `CaseResponse` for a single case. Provider callers must include `organizationGroupId` so the service can authorize the read against the caller's scope.

* **Authorities:** `SCOPE_milaadmindevscope`, `SCOPE_miladevprovidersscope`, `APPROLE_Mila.ALL`, `APPROLE_Mila.PROVIDER`
* **Path params:**
  * `caseId` *(CaseId, required)*
* **Query params:**
  * `organizationGroupId` *(OrganizationGroupId, optional but expected for provider callers)*
* **Response:** `200 OK` — `CaseResponse` (full case projection including patient, provider, outcome plan, codes, escalations, interactions, tags, and timestamps)
* **Error responses:**
  * `403` — caller not authorized for the case's organization group
  * `404` — case not found

---

### 3. Get In-Progress Conversation Interaction

`GET /v2/cases/{caseId}/conversation-interaction`

Returns the currently active (in-progress) conversation interaction for a case, when one exists. Used by Provider clients to drive a live conversational UI.

* **Authorities:** `SCOPE_milaadmindevscope`, `SCOPE_miladevprovidersscope`, `APPROLE_Mila.ALL`, `APPROLE_Mila.PROVIDER`
* **Path params:** `caseId` *(CaseId, required)*
* **Query params:** `organizationGroupId` *(OrganizationGroupId, optional)*
* **Responses:**
  * `200 OK` — `Interaction` (id, type, channel, start/end times, status, messages, metadata, evaluation, summary) when an in-progress interaction exists
  * `202 Accepted` — empty body, when no interaction is currently in progress
* **Error responses:**
  * `403` — caller not authorized for the case's organization group
  * `404` — case not found

---

### 4. Get Interaction Audio

`GET /v2/cases/{caseId}/interactions/{interactionId}/audio`

Returns a signed URL to the recorded audio for a specific interaction. The URL is short-lived and intended for direct retrieval by the Provider client.

* **Authorities:** `SCOPE_milaadmindevscope`, `SCOPE_miladevprovidersscope`, `APPROLE_Mila.ALL`, `APPROLE_Mila.PROVIDER`
* **Path params:**
  * `caseId` *(CaseId, required)*
  * `interactionId` *(InteractionId, required)*
* **Response:** `200 OK` — `InteractionAudioResponse { caseId, interactionId, interactionAudioUrl }`
* **Error responses:**
  * `400` — invalid request
  * `404` — case or interaction not found, or audio not available

---

### 5. Cancel Case

`PUT /v2/cases/{caseId}/cancel`

Transitions the case to the `CANCELED` terminal state. The case must be in a state from which cancellation is allowed. Caller must have `WRITE` access to the case's organization group.

* **Authorities:** `SCOPE_milaadmindevscope`, `SCOPE_miladevprovidersscope`, `APPROLE_Mila.ALL`, `APPROLE_Mila.PROVIDER`
* **Path params:** `caseId` *(CaseId, required)*
* **Query params:** `organizationGroupId` *(OrganizationGroupId, optional)*
* **Response:** `200 OK` — `UpdateCaseResponse` (full updated case projection)
* **Error responses:**
  * `400` — illegal state transition
  * `403` — caller lacks write access for the case's organization group
  * `404` — case not found

---

### 6. Complete Case

`PUT /v2/cases/{caseId}/complete`

Transitions the case to the `COMPLETED` terminal state. Caller must have `WRITE` access to the case's organization group.

* **Authorities:** `SCOPE_milaadmindevscope`, `SCOPE_miladevprovidersscope`, `APPROLE_Mila.ALL`, `APPROLE_Mila.PROVIDER`
* **Path params:** `caseId` *(CaseId, required)*
* **Query params:** `organizationGroupId` *(OrganizationGroupId, optional)*
* **Response:** `200 OK` — `UpdateCaseResponse`
* **Error responses:**
  * `400` — illegal state transition
  * `403` — caller lacks write access
  * `404` — case not found

---

### 7. Close Case

`PUT /v2/cases/{caseId}/close`

Transitions the case to the `CLOSED` terminal state. Caller must have `WRITE` access to the case's organization group.

* **Authorities:** `SCOPE_milaadmindevscope`, `SCOPE_miladevprovidersscope`, `APPROLE_Mila.ALL`, `APPROLE_Mila.PROVIDER`
* **Path params:** `caseId` *(CaseId, required)*
* **Query params:** `organizationGroupId` *(OrganizationGroupId, optional)*
* **Response:** `200 OK` — `UpdateCaseResponse`
* **Error responses:**
  * `400` — illegal state transition
  * `403` — caller lacks write access
  * `404` — case not found

---

### 8. List / Search Cases

`GET /v2/cases`

Returns a paged list of cases visible to the caller, with optional filtering by scope, patient, outcome plan, free-text search, time range, state, type, tags, and escalation type. The service derives an effective scope from the most specific scope identifier provided (`providerId` > `organizationId` > `organizationGroupId` > `adminGroupId`) and falls back to the caller's identity (admin user or authenticated provider) when none is supplied. Authorization is enforced for whichever scope is resolved.

* **Authorities:** `SCOPE_milaadmindevscope`, `SCOPE_miladevprovidersscope`, `APPROLE_Mila.ALL`, `APPROLE_Mila.PROVIDER`
* **Query params:**
  * Pagination — `page`, `size` *(default size = 25)*, `sort`
  * Scope — `adminGroupId`, `organizationGroupId`, `organizationId`, `providerId`
  * Patient — `patientId`, `patientMobilePhoneNumber` *(at most one of the two)*
  * Outcome plan — `outcomePlanId`
  * Search — `searchText`
  * Time window — `startDate`, `endDate` *(ISO-8601 datetime with offset/Z, or `YYYY-MM-DD`; `startDate` is inclusive, `endDate` is exclusive next-day-start)*
  * State filters — `states` *(set of `CaseState`)*, `isOpen` *(true or omit)*, `isClosed` *(true or omit)*
  * Type filters — `types` *(set of `CaseType`)*
  * Tag filters — `tagNames` *(set of strings)*, `tagMatchMode` *(`ANY` (default) or `ALL`)*
  * Escalation filters — `escalationTypes` *(set of `EscalationType`)*, `escalationMatchMode` *(`ANY` (default) or `ALL`)*
* **Response:** `200 OK` — `GetCasesResponse { cases: List<CaseResponse>, totalCount, pageNumber, pageSize }`
* **Validation errors (`400`):**
  * `patientId` and `patientMobilePhoneNumber` cannot both be supplied
  * `isOpen` and `isClosed` cannot be `false`; cannot both be `true`
  * `endDate` must be after `startDate`
  * `startDate` / `endDate` must be ISO-8601 with offset/Z or `YYYY-MM-DD`

---

### 9. Get Case Distribution

`GET /v2/cases/distribution`

Returns aggregated counts of cases bucketed by one or more dimensions (`distributionBy`) — for example, by state, type, tag, or escalation type — applying the same scope and filter rules as `GET /v2/cases`. Useful for dashboard summaries.

* **Authorities:** `SCOPE_milaadmindevscope`, `SCOPE_miladevprovidersscope`, `APPROLE_Mila.ALL`, `APPROLE_Mila.PROVIDER`
* **Query params:**
  * `distributionBy` *(set of `DistributionType`, **required**)* — e.g. `STATE`, `TYPE`, `TAG_NAME`, `ESCALATION_TYPE`
  * Scope — `adminGroupId`, `organizationGroupId`, `organizationId`, `providerId`
  * `patientId`, `patientMobilePhoneNumber`, `outcomePlanId`, `searchText`
  * `startDate`, `endDate` *(same flexible date parsing as List Cases)*
  * `states`, `types`, `tagNames`, `escalationTypes`, `isOpen`, `isClosed`
* **Response:** `200 OK` — `CaseDistributionResponse { distributions: Map<String, Map<String, Long>> }`
  Outer key is the distribution dimension; inner key is the bucket value; value is the count.
* **Validation errors (`400`):**
  * `distributionBy=TAG_NAME` requires non-empty `tagNames`
  * `distributionBy=ESCALATION_TYPE` requires non-empty `escalationTypes`
  * `patientId` and `patientMobilePhoneNumber` cannot both be supplied
  * `isOpen` / `isClosed` rules same as List Cases
  * `endDate` must be after `startDate`

---

### 10. Update Escalation Status

`PUT /v2/cases/{caseId}/escalations/{escalationId}/status`

Updates the status of a specific escalation on a case (e.g., acknowledging, resolving, or dismissing it) and optionally records reviewer notes. Caller must have `WRITE` access to the case's organization group.

* **Authorities:** `SCOPE_miladevprovidersscope`, `APPROLE_Mila.ALL`, `APPROLE_Mila.PROVIDER`
* **Content-Type:** `application/json`
* **Path params:**
  * `caseId` *(CaseId, required)*
  * `escalationId` *(EscalationId, required)*
* **Query params:** `organizationGroupId` *(OrganizationGroupId, optional)*
* **Request body:** `UpdateEscalationStatusRequest`
  * `status` *(EscalationStatus, required)*
  * `notes` *(string, optional)*
* **Response:** `200 OK` — `CaseResponse` (the case with the escalation updated)
* **Error responses:**
  * `400` — invalid status or body
  * `403` — caller lacks write access for the case's organization group
  * `404` — case or escalation not found

---

## Case Ingestion Jobs API (`/v1/case-ingestion-jobs`)

These endpoints support bulk ingestion of cases from an uploaded file and inspection of the resulting jobs.

### 11. Create Case Ingestion Job

`POST /v1/case-ingestion-jobs`

Submits a new bulk ingestion job. The caller uploads a multipart form containing the JSON ingestion request and the input data file. The service merges the submitted parameters with system defaults, persists the file, kicks off asynchronous ingestion, and returns the created job descriptor.

* **Authorities:** `SCOPE_miladevprovidersscope`, `APPROLE_Mila.ALL`, `APPROLE_Mila.PROVIDER`
* **Content-Type:** `multipart/form-data`
* **Form parts:**
  * `ingestionRequest` *(string, required)* — JSON-serialized `CreateCaseIngestionJobRequest` with:
    * `organizationGroupId` *(OrganizationGroupId, required)*
    * `parameters` *(CaseIngestionJobParameters)* — overrides for default ingestion behavior
    (`type` is fixed to `CASE_INGESTION` server-side)
  * `inputFile` *(file, required)* — the bulk-input file to be ingested
* **Response:** `201 Created` — `CaseIngestionJob` (see fields below)
* **Error responses:**
  * `400` — `ingestionRequest` JSON is unparseable or required fields missing

---

### 12. Get Case Ingestion Job

`GET /v1/case-ingestion-jobs/{jobId}`

Returns the current status and metadata of a previously created ingestion job, including counts of total / succeeded / failed records and pointers to result files.

* **Authorities:** `SCOPE_miladevprovidersscope`, `APPROLE_Mila.ALL`, `APPROLE_Mila.PROVIDER`
* **Path params:** `jobId` *(JobId, required)*
* **Query params:** `organizationGroupId` *(OrganizationGroupId, **required**)*
* **Response:** `200 OK` — `CaseIngestionJob`:
  * `jobId`, `organizationGroupId`, `jobExecutionId`, `type`, `status` *(JobStatus)*
  * `fileName`, `inputFilePath`, `outputFilePath`, `succeededRecordsFilePath`, `failedRecordsFilePath`
  * `totalRecords`, `totalSucceededRecords`, `totalFailedRecords`
  * `parameters` *(CaseIngestionJobParameters)*
  * `createdAt`, `lastModifiedAt`
* **Error responses:**
  * `403` — caller not authorized for the supplied organization group
  * `404` — job not found

---

### 13. List Case Ingestion Jobs

`GET /v1/case-ingestion-jobs`

Returns a paged list of ingestion jobs for an organization group, with optional filtering by status and time window.

* **Authorities:** `SCOPE_miladevprovidersscope`, `APPROLE_Mila.ALL`, `APPROLE_Mila.PROVIDER`
* **Query params:**
  * `organizationGroupId` *(OrganizationGroupId, **required**)*
  * `status` *(JobStatus, optional)*
  * `startDate`, `endDate` *(optional; ISO-8601 datetime with offset/Z or `YYYY-MM-DD`. Same inclusive-start / exclusive-end semantics as the Cases endpoints. If `startDate` is supplied and `endDate` is omitted, `endDate` defaults to "now".)*
  * `pageNumber`, `pageSize` *(optional)*
* **Response:** `200 OK` — `ListCaseIngestionJobsResponse { jobs: List<CaseIngestionJob>, totalCount, pageNumber, pageSize }`
* **Validation errors (`400`):**
  * `endDate` must be after `startDate`
  * Invalid date format

---

## Common error model

All endpoints surface validation and authorization errors through the service's standard exception types, which map to HTTP status codes as follows:

| Exception                       | HTTP status | Typical cause                                         |
| ------------------------------- | ----------- | ----------------------------------------------------- |
| `InvalidInputException`         | 400         | Missing/invalid query, path, or body parameters       |
| `InvalidRequestException`       | 400         | Semantic validation failure on a request              |
| `UnauthorizedAccessException`   | 401         | Caller identity cannot be determined                  |
| `AccessDeniedException`         | 403         | Caller authenticated but not authorized for resource  |
| `ResourceNotFoundException`     | 404         | Target case / interaction / job does not exist        |

Provider clients should treat `403` as a hard scope boundary — supplying an `organizationGroupId` (or other scope identifier) outside the caller's tenant will result in a denial rather than an empty result.
