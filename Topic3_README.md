# Topic 3 - Data Validation, Cleaning & Duplicate Prevention

## Project overview
An n8n workflow that generates five mock marketing leads, standardizes contact data, checks lead quality, separates invalid records, detects duplicates among valid records, and marks unique leads for acceptance.

## Workflow architecture
Manual Trigger -> CodeGenerateRawLeads -> CodeSanitizeAndFormat -> CodeValidateLeadQuality -> IF IsQualityLead

- False: Log QuarantineInvalid
- True: Code DeduplicateLeads -> IF IsUniqueLead
  - True: Mark Unique Leads
  - False: Log Duplicate Leads

## Node-by-node explanation
1. **Manual Trigger:** Starts a test run on demand.
2. **CodeGenerateRawLeads:** Produces five mock records with valid, invalid, and repeated data.
3. **CodeSanitizeAndFormat:** Trims names/company values, lowercases emails, and normalizes phone numbers.
4. **CodeValidateLeadQuality:** Checks email syntax, disposable email domains, phone format, and lead quality; adds validation status/errors.
5. **IF IsQualityLead:** Sends valid leads down the true branch and invalid leads to quarantine.
6. **Log QuarantineInvalid:** Tags rejected records as `REJECTED INVALID LEAD`, preserving source fields.
7. **Code DeduplicateLeads:** Evaluates three valid leads for duplicates and adds uniqueness fields.
8. **IF IsUniqueLead:** Routes unique leads to acceptance and duplicates to rejection.
9. **Mark Unique Leads:** Tags two unique leads as `ACCEPTED UNIQUE LEAD` and retains the source fields.
10. **Log Duplicate Leads:** Tags one duplicate lead as `REJECTED DUPLICATE LEAD` and retains the source fields.

## Observed test results (screenshots, October 15, 2026)
| Check | Observed result |
| --- | --- |
| Mock leads generated | 5 |
| Leads passing quality gate | 3 |
| Invalid leads quarantined | 2 |
| Unique leads accepted | 2 |
| Duplicate leads rejected | 1 |
| Workflow execution | Green checks on all displayed nodes |

The screenshots show `LD-001` (Sarah Connor) in the accepted unique branch and `LD-004` (duplicate Sarah Connor submission) in the rejected duplicate branch. The invalid example `LD-002` has a malformed email address. The implementation plan also expects `LD-003` to be rejected for a disposable email domain and `LD-005` to be accepted as a unique lead; their individual final output details are not visible in the supplied screenshots.

## Evidence and validation notes
- The `IF IsQualityLead` condition uses `{{ $json.isQualityLead }}` with `is true`; its output shows 3 true and 2 false items.
- The `IF IsUniqueLead` condition uses `{{ $json.isUniqueLead }}` with `is true`; its output shows 2 true and 1 false items.
- The acceptance and duplicate-log nodes have `Include Other Input Fields` enabled, preserving the lead data and validation metadata.
- The screenshots show no CRM HTTP request node or final metrics node. Consequently, CRM synchronization and final aggregate metrics are **not verified**.

## Alignment with supplied source documents
The implementation plan describes a mock CRM HTTP POST step and a final metrics step. The requirements file focuses on field validation, normalization, deduplication, and safe rejection. The workflow shown demonstrates the cleaning and routing stages; CRM posting and metrics remain optional next steps for completing the implementation-plan architecture.

## Suggested remaining steps
1. Configure a mock CRM endpoint and add `HTTP_PostCleanLeadToCRM` after `Mark Unique Leads`.
2. Add `Code_PipelineMetrics` to summarize the successful CRM responses.
3. Run the full workflow and capture the HTTP response and metrics output.
4. Retest missing required fields and missing optional phone values, as specified by the requirements file.

## Reference documents
- `imptop3(1).pdf` - Topic 3 implementation plan.
- `REQTOP3.docx` - Topic 3 requirements.

## Scope disclaimer
This README records what is visible in the supplied n8n screenshots; it does not assert a production CRM integration or independently audited data persistence.
