# Topic 2 - Multi-Modal Data Transformation & PDF Extraction

## Project overview
This n8n workflow demonstrates how to process two data sources - simulated PDF invoices and legacy JSON orders - into standardized ERP-ready records. It uses JavaScript Code nodes, Extract from File (PDF), a Merge node, Edit Fields, an HTTP POST request, and a final audit summary.

**Environment:** n8n (self-hosted training workspace)  
**Execution:** Manual Trigger  
**Integration target:** `https://httpbin.org/post` (public echo/test endpoint; **not** a real ERP)  
**Evidence date:** 15 October 2026

## Workflow architecture

```text
Manual Trigger
  |-- Code GenerateLegacyOrders (2 orders)
  |     -> Code NormalizeDates
  |     -> TransformProcessLineItems ---------+
  |                                            |
  +-- Code_SimulatePDFInvoiceBinary (1 PDF)   |
        -> Extract PDF Text                    |
        -> Transform ParsePDFInvoice ---------+ 
                                               |
                                      Merge PDF and JSON (Append: 3 items)
                                               |
                                      EditFieldsStandardizeERP (3 items)
                                               |
                                      HTTP PostCleanOrderToERP (3 responses)
                                               |
                                      Code AuditSummary (1 summary)
```

## Node responsibilities
| Node | Purpose | Observed result |
|---|---|---|
| Manual Trigger | Start an on-demand test run | Both branches execute |
| Code GenerateLegacyOrders | Create two sample legacy orders | 2 items |
| Code NormalizeDates | Convert mixed input dates into consistent UTC dates | 2 items |
| TransformProcessLineItems | Clean customer/product fields and calculate totals | 2 items |
| Code_SimulatePDFInvoiceBinary | Generate a test PDF binary | 1 item |
| Extract PDF Text | Extract text from the PDF | 1 item |
| Transform ParsePDFInvoice | Parse invoice metadata, items and totals | 1 item |
| Merge PDF and JSON | Append two legacy orders and one PDF invoice | 3 items |
| EditFieldsStandardizeERP | Map records to common ERP-ready fields | 3 items |
| HTTP PostCleanOrderToERP | POST each record to the httpbin echo API | 3 items |
| Code AuditSummary | Aggregate the mock POST results | 1 item |

## Execution evidence
The supplied n8n screenshots show green successful node indicators through the audit node. The audit output displays:

| Field | Observed value |
|---|---|
| auditStatus | COMPLETED |
| totalOrders | 3 |
| totalAmount | 1565.46 |
| LEG-8841 | 277.79; POST_SUCCESS |
| LEG-8842 | 616.99; POST_SUCCESS |
| INV-PDF-2026-904 | 670.68; POST_SUCCESS |

**Reconciliation:** 277.79 + 616.99 + 670.68 = **1565.46**.

## How to test
1. Open the workflow in n8n and select **Execute workflow**.
2. Confirm both branches complete and **Merge PDF and JSON** outputs three items.
3. Open **EditFieldsStandardizeERP** to inspect the common field structure.
4. Open **HTTP PostCleanOrderToERP** to inspect the echoed POST data. The `httpbin.org/post` endpoint confirms receipt of a test request only.
5. Open **Code AuditSummary** and confirm `auditStatus = COMPLETED`, `totalOrders = 3`, and `totalAmount = 1565.46`.

## Important limitations
- The PDF and legacy orders are **simulated training inputs**, not live customer data.
- `httpbin.org/post` is an **echo API**, not an ERP endpoint. `POST_SUCCESS` reflects the training workflow's status logic; it is not proof of an ERP import.
- The screenshots show a successful test execution, not production readiness. Authentication, input validation, retry policies, idempotency, monitoring, and reconciliation should be added before connecting a real ERP.
- The supplied implementation plan suggests DummyJSON/Postman mock ERP services; this execution instead uses httpbin as shown in the screenshots.

## Source materials
- Tayana Academy, *Topic 2: Advanced Expressions, PDF Data Extraction & Dynamic Data Transformation* (implementation plan supplied by user).
- User-provided n8n workflow canvas and Code AuditSummary screenshots (15 October 2026).
