# Product Support Engineer — Assignment Submission

- **Name:** Sanjay M
- **Date submitted:** 14 November 2026
- **Repository link:** https://github.com/sanjay-complyance/pse-support-lab
- **Total time spent (real time):** 6 hours

> All customer replies and acknowledgements below are drafts only. No real customer emails were sent. The exercise clock is Monday, 16 November 2026, Asia/Dubai. Real working time is recorded separately.

## Items fully investigated and why I chose them

I investigated all five incidents. I prioritised Marina Bay first because it was a P1 production alert affecting multiple customers. I then investigated Oasis, Al Noor and Sahara because of month-end processing, rejected invoices and missing ASP confirmations. I also investigated Ghaf to distinguish a product defect from an input-format issue.

---

## Incident 1 — Oasis Builders upload stuck

### Acknowledgement

- **Sent at:** Not sent — draft only.
- **Reply:**

> Hi Rashid,
>
> We understand the urgency of your November month-end upload and the concern about duplicate document numbers. We are checking the import and invoice-level statuses to identify which invoices have reached the ASP.
>
> Please do not upload the same file again until reconciliation is complete. We will share the verified invoice status and safe next steps.
>
> Regards,
> Product Support

### Priority and justification

- **Priority:** P2.
- **Why:** A large monthly import took approximately 121 seconds. The customer reported repeated upload attempts, duplicate-number messages and month-end business impact. The available evidence confirms the November import completed, but does not establish what happened during all three customer retry attempts.
- **SLA deadlines:**
  - First response: 10:05 GST.
  - Next update: within 2–4 hours of the acknowledgement.
  - Resolution target: 17:05 GST on 16 November.

### Investigation log

| Time (exercise clock) | What I checked / did | What I found |
|---|---|---|
| 09:05 | Read customer report | Customer reported Processing delay, three attempts and duplicate-number messages. |
| Investigation | Queried imports for seller ID 2 | Two imports found: October and November. No separate import records corresponding to the reported three retries were visible. |
| Investigation | Checked November import | Import `4075ceea-e74e-4e85-82c9-c1c12f5a2ff3`; 600 rows, 8 invoices, Completed, 121128 ms. |
| Investigation | Queried invoice statuses | All 8 November invoices were Accepted. |
| Investigation | Queried invoice event timeline | All 8 parsed, validated, submitted and accepted. Validation events spanned approximately 106 seconds. |
| Investigation | Inspected importer source | `importer.go` validates invoices in a sequential loop before enqueueing them for submission. |
| Investigation | Inspected validator source | `validate.go` performs row validation and an existing-invoice status check. No explicit 15-second sleep was found in the inspected validation code. |

Dead ends and limitations:
- The customer reported three retries, but separate retry imports were not found in the imports table. This does not prove that no retry request reached the application.
- The exact technical reason for the approximately 15-second validation event spacing is not established.
- The local replay data confirms the eight invoices in this import, not necessarily every invoice the customer expected to upload.

### Root cause

- **What happened:** The November import completed in approximately 121 seconds. All eight invoices were accepted by the ASP.
- **Where:** Import processing in `services/api/internal/importer/importer.go` (validation loop around lines 96–117); validation and duplicate checking in `services/api/internal/validate/validate.go` (around lines 67–85).
- **Why it happens:** Source code confirms sequential invoice validation. Database timestamps show validation events approximately 15 seconds apart. The exact underlying source of that interval is not proven.
- **Evidence:**
  - Import ID: `4075ceea-e74e-4e85-82c9-c1c12f5a2ff3`.
  - Import: 600 rows, 8 invoices, 121128 ms.
  - Invoice query: all 8 statuses Accepted.
  - Event query: Parsed 8, Validated 8, Submitted 8, Accepted 8.
  - Source inspection: importer validation loop and validator duplicate check.
  - Detailed evidence: [Oasis Builders investigation](evidence/oasis-investigation.txt).
- **Confidence and open questions:** High confidence that all eight invoices in this import were accepted. High confidence that validation is sequential in the inspected code. The exact cause of the observed delay and the outcome of every customer retry remain unconfirmed.

### Customer resolution reply

- **Which one and why:** Customer reply draft, because the import's invoice statuses are verified but reconciliation of all expected invoices remains necessary.

> Hi Rashid,
>
> We checked the November import and confirmed that all eight invoices recorded in that import were accepted by the ASP.
>
> Please do not upload the same eight invoices again, as this could create duplicate-document issues. We found the November import completed in approximately 121 seconds.
>
> To safely identify anything outstanding, please compare your expected November invoice numbers against the eight accepted invoices in the portal. We should only retry invoices confirmed as missing after checking their document numbers and current status.
>
> We are also reviewing the unusually long validation time.
>
> Regards,
> Product Support

### Workaround offered

Do not repeat the complete upload. Reconcile invoice numbers against the eight accepted invoices. Retry only invoices confirmed missing and not already submitted or accepted.

### Prevention ideas

- Review sequential validation performance and identify the slow operation.
- Measure validation duration per invoice and per rule.
- Consider safe bounded concurrency after engineering review.
- Improve upload progress reporting for large files.
- Provide a clear status and reconciliation report after interrupted uploads.
- Add safeguards and clearer guidance against repeated whole-file uploads.

---

## Incident 2 — Marina Bay API restarted

### Acknowledgement

- **Sent at:** Not sent — draft only.
- **Sent to:** Priya Nair, Customer Success Manager.
- **Reply:**

> Hi Priya,
>
> We acknowledge the critical API restart alert and your report of intermittent 502 errors and invoices remaining in Validated status.
>
> We are treating this as a P1 incident and escalating to Engineering on-call. We are investigating API restarts, upload failures and submission processing. We will verify invoice states before recommending retries to avoid duplicates.
>
> We will also assess the impact on Marina Bay Hospitality's planned 18 November go-live. Our next update will be within 30–60 minutes.
>
> Regards,
> Product Support

### Priority and justification

- **Priority:** P1.
- **Why:** Production API restarted three times within 15 minutes. Multiple customers reported intermittent 502 errors, and invoices were reportedly stuck in Validated. Marina Bay has a go-live planned for 18 November.
- **SLA deadlines:**
  - Alert received: 09:42 GST.
  - First response target: 09:57–10:12 GST.
  - Next update: within 30–60 minutes.
  - Resolution target: 13:42 GST.
  - Immediate L3 escalation after confirming impact.

### Investigation log

| Time (exercise clock) | What I checked / did | What I found |
|---|---|---|
| 09:42 | Read Grafana alert | Production API restart count was 3 in 15 minutes; severity critical. |
| 09:48 | Reviewed Customer Success escalation | Multiple customers reported intermittent 502 errors and invoices stuck in Validated since approximately 09:30. |
| 09:50 | Reviewed forwarded alert | Marina Bay go-live on 18 November adds business urgency. |
| Investigation | Checked local API container and logs | Local lab container had RestartCount 0 at inspection; local logs showed successful worker startup after temporary database readiness warnings. |
| Investigation | Reviewed local Grafana information | Local telemetry is not sufficient to establish the production incident's root cause. |
| Investigation | Checked seller invoice records | Local database had no invoices for Marina Bay seller ID 3. |

Dead ends and limitations:
- Local Docker container status and local Grafana data cannot prove the cause of the simulated production alert.
- No production restart exit codes, OOM events, deployment history or production ingress logs were available in the inspected local evidence.
- Marina Bay had no invoice records in the local dataset, so its individual invoice impact could not be reconciled.

### Root cause

- **What happened:** Production API reportedly restarted three times in 15 minutes. Customers reported intermittent 502 responses and invoices stuck in Validated.
- **Where:** Production API/container, ingress or reverse proxy, submission workers, database and ASP connectivity are investigation areas. Exact failing component is not confirmed.
- **Why it happens:** Unknown. The available evidence confirms symptoms and impact but not the production root cause.
- **Evidence:**
  - Grafana alert: `InvoicelyApiRestarted`, `restarts=3`, `env=production`, severity critical.
  - Customer Success report at 09:48 GST: intermittent 502s and invoices stuck in Validated since approximately 09:30.
  - Local API and database checks are not production evidence.
  - Detailed evidence: [Marina Bay investigation](evidence/marina-bay-investigation.txt).
- **Confidence and open questions:** High confidence in the reported P1 symptoms. Production technical root cause remains unknown and requires L3 investigation.

### Customer resolution reply OR escalation note

- **Which one and why:** L3 escalation plus customer update. P1 requires immediate Engineering on-call escalation. L1 retains customer ownership.

### Escalation: Production API restarts and intermittent 502 errors

**Priority:** P1 — repeated production API restarts, multiple affected customers, upload failures and go-live risk.

**Customer / seller:** Marina Bay Hospitality LLC, seller ID 3.

**Reported by / at:** Grafana alert at 09:42 GST; Priya Nair forwarded Customer Success report at 09:50 GST.

**Affected:** Multiple customers reported 502 errors and invoices stuck in Validated. Exact affected invoice IDs and count require production reconciliation. Marina Bay go-live is scheduled for 18 November.

**Symptoms (customer view):**
- Intermittent 502 Bad Gateway, mainly during uploads.
- Some invoices reportedly remain in Validated.
- API process restarted three times in 15 minutes.

**Evidence:**
- Alert: `InvoicelyApiRestarted`, `restarts=3`, production environment.
- Customer Success report: invoices stuck since approximately 09:30.
- Production logs, restart exit codes, ingress traces and affected invoice IDs are still required.

**Steps to reproduce:**
1. Obtain affected customer, import and invoice IDs from production.
2. Correlate upload requests with ingress and API logs.
3. Inspect API restart events and process/container exit codes.
4. Correlate Validated invoices with worker activity and ASP responses.

**What I have already tried / ruled out:**
- Checked local API container restart count; it was zero at inspection.
- Reviewed local API logs; these do not establish production cause.
- Checked local Marina Bay seller records; no invoices were present.
- Did not recommend blind retries.

**Suspected cause (with confidence):** Unknown. Possible areas for investigation include process crashes, memory/OOM, health checks, deployment/configuration changes, ingress failures, worker processing and database/ASP connectivity. These are hypotheses, not confirmed causes.

**Workaround in place:** No confirmed production workaround. Avoid blind invoice retries; reconcile status before retrying.

**Timeline:**
- 09:42 — Critical Grafana alert received.
- 09:48 — Customer Success impact report.
- 09:50 — Forwarded escalation received.
- Next — Page Engineering on-call and investigate production telemetry.

**Ask:** Engineering to investigate restart events, exit codes, memory/OOM, deployment changes, health checks, ingress 502 traces, worker queue depth and age, database/ASP connectivity, and affected invoice states. Provide immediate mitigation and an initial findings update.

**Next customer update due:** Within 30–60 minutes of the acknowledgement.

**Customer update draft:**

> Hi Priya,
>
> This is being handled as a P1 incident and has been escalated to Engineering on-call. We have confirmed the reported alert and customer impact, but the production root cause is not yet established.
>
> Engineering is being asked to correlate API restart events, 502 traces and submission-worker activity. We will reconcile affected invoice statuses before advising any retry. We will provide another update within 30–60 minutes and will continue to assess Marina Bay's 18 November go-live readiness.
>
> Regards,
> Product Support

### Workaround offered

No verified global workaround is available. Ask affected customers not to retry blindly. Reconcile each invoice's portal and ASP status before any controlled retry. Keep the incident open until production stability and invoice state are confirmed.

### Prevention ideas

- Alert on restart rate, API 5xx rate and aged Validated invoices.
- Add dashboards for worker queue depth, processing age and ASP response codes.
- Capture container exit codes, OOM events and deployment changes.
- Add health-check and graceful-shutdown safeguards.
- Test upload and submission recovery under load.
- Run a Marina Bay go-live readiness review only after production stability and invoice reconciliation are verified.

---

## Incident 3 — Al Noor Fuel BR-CO-15

### Acknowledgement

- **Sent at:** Not sent — draft only.
- **Reply:**

> Dear Fatima,
>
> We have reviewed the November import and identified three invoices rejected by the ASP with BR-CO-15. The remaining three invoices in the import were accepted.
>
> We are checking the reported amount differences and will provide correction guidance. Please do not resend the entire file; only confirmed rejected invoices should be corrected and resubmitted after duplicate-status checks.
>
> Regards,
> Product Support

### Priority and justification

- **Priority:** P2.
- **Why:** Three of six invoices were rejected by the ASP. The customer has an external auditor reviewing its VAT position, creating financial and reporting impact.
- **SLA deadlines:**
  - Received: 10:15 GST.
  - First response target: 11:15 GST.
  - Next update: within 2–4 hours.
  - Resolution target: Tuesday 17 November, 08:15 GST.

### Investigation log

| Time (exercise clock) | What I checked / did | What I found |
|---|---|---|
| 10:15 | Read customer report | Customer reported BR-CO-15 on several November invoices. |
| Investigation | Queried import and invoice statuses | Import `3fc4ff01-7f42-459b-bf1a-2c5ab1a08fa5`; six invoices, three Accepted and three Rejected. |
| Investigation | Reviewed rejected invoice errors | Three rejected invoices showed small differences between IBT-112 and IBT-109 + IBT-110. |
| Investigation | Checked invoice events | Rejected invoices passed portal validation, were submitted to ASP and then rejected by ASP. |
| Investigation | Compared accepted invoices | Other three invoices in the same import were Accepted. |

### Root cause

- **What happened:** Three invoices were rejected by the ASP with BR-CO-15 because the invoice total including VAT did not equal the sum of the exclusive amount and VAT amount.
- **Where:** Invoice totals and tax calculation in the submitted invoice payload; rejection occurred at ASP validation.
- **Why it happens:** Observed differences:
  - `ANF-2611-0042-GBO1`: 15022.42 vs expected 15022.43.
  - `ANF-2611-0044-GBO1`: 11817.57 vs expected 11817.59.
  - `ANF-2611-0046-GBO1`: 5610.99 vs expected 5611.00.
  These are small amount differences. The exact calculation or rounding step responsible is not confirmed.
- **Evidence:**
  - Import ID `3fc4ff01-7f42-459b-bf1a-2c5ab1a08fa5`.
  - Six invoices: three Accepted, three Rejected.
  - Invoice events show portal validation passed before ASP rejection.
  - ASP error BR-CO-15 identifies the arithmetic mismatch.
  - Detailed evidence: [Al Noor Fuel investigation](evidence/al-noor-investigation.txt).
- **Confidence and open questions:** High confidence in the reported arithmetic mismatch and ASP rejection. Exact source of rounding difference needs payload and calculation review.

### Customer resolution reply

- **Which one and why:** Customer reply draft explaining the confirmed rule failure and safe correction process.

> Dear Fatima,
>
> We confirmed that three invoices were rejected by the ASP with BR-CO-15. The rule requires the invoice total including VAT (IBT-112) to equal the total excluding VAT (IBT-109) plus total VAT (IBT-110).
>
> The differences are small:
>
> - ANF-2611-0042-GBO1: 0.01
> - ANF-2611-0044-GBO1: 0.02
> - ANF-2611-0046-GBO1: 0.01
>
> The other three invoices in the same import were accepted. Please review the calculation and rounding applied to the three rejected invoices so the totals satisfy the rule. Once corrected, check the portal status and resubmit only those rejected invoices. Do not resend the complete batch.
>
> We have not established whether the discrepancy originated from line-level or invoice-level rounding, so we do not want to assume a specific calculation step without reviewing the source calculation.
>
> Regards,
> Product Support

### Workaround offered

Recalculate the three rejected invoices using a consistent decimal precision and rounding policy. Confirm IBT-112 equals IBT-109 plus IBT-110. Check invoice status and duplicate receipt before resubmitting only the rejected invoices.

### Prevention ideas

- Apply a consistent rounding policy across line and invoice totals.
- Add pre-submission BR-CO-15 validation.
- Add regression tests for small decimal differences.
- Show the three relevant amounts clearly in validation errors.

---

## Incident 4 — BluePeak / Sahara Medical invoices not received

### Acknowledgement

- **Sent at:** Not sent — draft only.
- **Reply:**

> Hi Daniel,
>
> We have reviewed the Sahara batch. The import endpoint accepted the upload, but an HTTP 201 response confirms the import request, not final ASP acceptance for every invoice.
>
> We found eight invoices Accepted and 32 still showing Submitted without an ASP message ID in the available database. Please do not resend the full batch until the remaining invoice references are reconciled with the ASP.
>
> Regards,
> Product Support

### Priority and justification

- **Priority:** P2.
- **Why:** 32 of 40 invoices have no confirmed ASP acceptance in the available evidence. Sahara's hospital customer has received only some invoices, and a payment run is due this week.
- **SLA deadlines:**
  - Received: 11:30 GST.
  - First response target: 12:30 GST.
  - Next update: within 2–4 hours.
  - Resolution target: Tuesday 17 November, 09:30 GST.

### Investigation log

| Time (exercise clock) | What I checked / did | What I found |
|---|---|---|
| 11:30 | Read BluePeak report | Connector received HTTP 201; 40 invoices imported; buyer received only some. |
| Investigation | Queried import | Import `9d10b954-c31c-4f7b-a980-259372d7f45e`; 60 rows and 40 invoices. |
| Investigation | Queried invoice statuses | Eight Accepted and 32 Submitted. |
| Investigation | Checked ASP references | The 32 Submitted invoices had no ASP message ID or ASP response in the available records. |
| Investigation | Reviewed API logs | ASP requests showed HTTP 429 for invoices 009–040. |
| Investigation | Reviewed invoice events | All 40 were Parsed, Validated and Submitted; only eight had Accepted events. |

### Root cause

- **What happened:** The import endpoint accepted the batch, but only eight invoices had confirmed ASP acceptance in the available data. Thirty-two remained Submitted without ASP message IDs.
- **Where:** ASP submission integration and rate-limiting response handling.
- **Why it happens:** API logs show HTTP 429 responses for invoices 009–040, consistent with ASP rate limiting. Final receipt or processing outcome for those invoices is not confirmed.
- **Evidence:**
  - Import ID `9d10b954-c31c-4f7b-a980-259372d7f45e`.
  - 40 invoices: eight Accepted, 32 Submitted.
  - 32 Submitted invoices had no ASP message ID or ASP response.
  - API logs showed HTTP 429 for invoices 009–040.
  - Detailed evidence: [Sahara Medical investigation](evidence/sahara-investigation.txt).
- **Confidence and open questions:** High confidence that 32 invoices lack confirmed acceptance in the available records and that HTTP 429 responses occurred. The final ASP-side state of each invoice must be reconciled before retrying.

### Customer resolution reply

- **Which one and why:** Customer reply draft explaining the difference between import acceptance and ASP acceptance.

> Hi Daniel,
>
> We reviewed the Sahara batch. The import contains 40 invoices. Eight have confirmed ASP acceptance. The remaining 32 are still shown as Submitted in our available records, without ASP message IDs.
>
> Our API logs show HTTP 429 responses for invoices 009–040, which indicates rate limiting. We cannot yet confirm the final ASP-side state of those invoices.
>
> Please hold off on resending the entire batch. We should reconcile the 32 invoice references with the ASP first and retry only those confirmed as not received, using controlled backoff.
>
> Regards,
> Product Support

### Workaround offered

Reconcile the 32 invoice references with ASP records. Retry only invoices confirmed as not received. Use controlled backoff and avoid resending the full batch.

### Prevention ideas

- Implement exponential backoff with jitter for retryable 429 responses.
- Respect `Retry-After` when supplied.
- Limit concurrent ASP submissions.
- Monitor 429 counts and aged Submitted invoices.
- Persist and reconcile ASP references and final responses.
- Distinguish import endpoint acceptance from final ASP acceptance in portal messaging.

---

## Incident 5 — Ghaf Tree Café date error

### Acknowledgement

- **Sent at:** Not sent — draft only.
- **Reply:**

> Hello Hessa,
>
> We checked the November upload. The dates in the file are written as DD/MM/YYYY, while the portal requires YYYY-MM-DD. The date itself may be correct; the format is what caused the validation rejection.
>
> We can help you correct the dates and upload the file again. Please check invoice status before retrying to avoid duplicates.
>
> Regards,
> Product Support

### Priority and justification

- **Priority:** P3.
- **Why:** One customer has a file-format issue with a documented workaround. The available evidence does not indicate a wider service outage.
- **SLA deadlines:**
  - Received: 12:10 GST.
  - First response target: 16:10 GST.
  - Next update: daily.
  - Resolution target: Thursday 19 November, 12:10 GST.

### Investigation log

| Time (exercise clock) | What I checked / did | What I found |
|---|---|---|
| 12:10 | Read customer report | Customer entered dates in DD/MM/YYYY format. |
| Investigation | Queried Ghaf import | Import `d4ddb3b6-9031-4635-9ad7-b0f33033e741`; eight rows and four invoices. |
| Investigation | Checked invoice statuses | All four invoices were Rejected by portal validation. |
| Investigation | Reviewed validation errors | Rule AE-BR-04 requires YYYY-MM-DD. Uploaded dates included `13/11/2026` and `13/12/2026`. |

### Root cause

- **What happened:** All four invoices were rejected during portal validation because the supplied date strings did not match the required format.
- **Where:** Portal date validation, rule AE-BR-04.
- **Why it happens:** The uploaded file used DD/MM/YYYY, while the portal expects YYYY-MM-DD. This is a format mismatch, not evidence that the calendar date itself is invalid.
- **Evidence:**
  - Import ID `d4ddb3b6-9031-4635-9ad7-b0f33033e741`.
  - Four invoices rejected.
  - Validation error AE-BR-04.
  - Example input: `13/11/2026`; expected: `2026-11-13`.
  - Detailed evidence: [Ghaf Cafe investigation](evidence/ghaf-investigation.txt).
- **Confidence and open questions:** High confidence in the date-format cause. No evidence indicates a platform defect.

### Customer resolution reply

- **Which one and why:** Customer reply draft, because the portal is enforcing its documented input format and a customer-side correction is available.

> Hello Hessa,
>
> We checked your upload. The issue is the date format rather than the date value.
>
> The portal requires YYYY-MM-DD. For example:
>
> - `13/11/2026` should be `2026-11-13`.
> - `13/12/2026` should be `2026-12-13`.
>
> Please update the date cells in the file and upload the corrected file. Before retrying, check the invoice status in the portal to ensure the same documents have not already been processed.
>
> We will also raise a suggestion to make the date-format instructions clearer in the template.
>
> Regards,
> Product Support

### Workaround offered

Convert the dates to YYYY-MM-DD, for example `2026-11-13` and `2026-12-13`. Check invoice status and then upload the corrected file.

### Prevention ideas

- Add date-format examples directly to the spreadsheet template.
- Provide clearer localized date guidance.
- Validate date formats before upload and highlight the affected cells.
- Improve the error message with an example of the expected format.

---

## Overall

### Prioritisation

I prioritised Marina Bay first because the alert indicated repeated production API restarts and multiple customers were reporting 502 errors and stuck invoices. This met the P1 criteria and required immediate L3 escalation.

Oasis, Al Noor and Sahara were P2 because they involved significant invoice processing or acceptance problems with month-end, audit or payment-run impact. Ghaf was P3 because the issue affected one customer and had a clear input-format workaround.

I kept the priority of Marina Bay at P1 even though the exact production root cause was unknown. Incident priority is based on impact and urgency, not on whether the root cause is already known.

### What I would do differently

- Capture the exact customer invoice numbers and import IDs at the start of each investigation.
- Collect production logs, metrics and database evidence together before drawing conclusions.
- Record timestamps and time zones consistently.
- Reconcile invoice state with the ASP before advising any retry.
- For large uploads, measure validation time per invoice and identify slow operations.
- Provide customers with clearer upload progress and invoice-level status information.

### What I learned / what I would need to learn

I learned how to investigate invoice processing through import records, invoice statuses and event history in PostgreSQL. I used SQL queries to compare Parsed, Validated, Submitted, Accepted and Rejected states.

I also learned to distinguish an HTTP 201 import response from final ASP acceptance, and to recognise HTTP 429 as a rate-limiting response that requires careful reconciliation and retry handling.

I inspected the Go importer and validation code to understand the processing sequence. The importer validates invoices sequentially before sending them to the submission queue.

I used Docker Compose to inspect the local services and logs, and reviewed Grafana alert information. I learned that local lab telemetry must not be presented as proof of a separate production incident.

Next, I would study:
- PostgreSQL query analysis and indexing.
- Go concurrency, worker pools and profiling.
- HTTP retry strategies, exponential backoff and idempotency.
- Prometheus metrics and Grafana alert investigation.
- UAE e-invoicing validation rules and ASP response handling.
- Production incident management and escalation practices.

### Time log (real time)

| Date | From–to | What I worked on |
|---|---|---|
| [13 Nov 2026] | [9:00P.M - 9:30P.M] | Environment setup, Docker and service startup |
| [13 Nov 2026] | [9:30P.M - 10:00P.M] | Loaded lab data and inspected incidents |
| [13 Nov 2026] | [10:00P.M - 11:30P.M] | Database investigation: Oasis, Al Noor, Sahara and Ghaf |
| [14 Nov 2026] | [8:30A.M - 9:30A.M] | Marina Bay alert and local observability checks |
| [14 Nov 2026] | [9:30A.M - 10:30A.M] | Source-code investigation and report preparation |
| 14 Nov 2026] | [10:30A.M - 11:30A.M] | Evidence collection, final review and GitHub submission |


### AI usage disclosure

I used ChatGPT (OpenAI) to help structure the investigation, explain SQL and shell commands, interpret code and logs, and draft customer communications and this report.

I verified the technical findings against the local lab database, Docker logs, source code and the provided incident files. I treated unverified production causes as unknown and did not present AI-generated hypotheses as confirmed facts.

No real customer emails were sent. Customer communications in this report are drafts only.
