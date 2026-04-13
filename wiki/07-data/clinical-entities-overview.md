# Clinical Entities Overview - Medication, Lab, Imaging

**Last Updated:** 2026-04-11  
**Analyzed By:** TrangN via Kiro/bmad

---

## Related Files
- `src/shared/model/Medication.ts` - Medication entity
- `src/shared/model/Lab.ts` - Lab entity
- `src/shared/model/Imaging.ts` - Imaging entity
- `src/labs/utils/validate-lab.ts` - Lab validation logic
- `src/shared/model/Permissions.ts` - Permission definitions

---

## Table of Contents
1. [Overview](#overview)
2. [Entity Comparison Table](#entity-comparison-table)
3. [Status Values Comparison](#status-values-comparison)
4. [Result Storage Analysis](#result-storage-analysis)
5. [Billing Integration](#billing-integration)
6. [Key Insights](#key-insights)
7. [Questions & Todos](#questions--todos)

---

## Overview

This document provides a comparative analysis of the three primary clinical order entities in HospitalRun: **Medication**, **Lab**, and **Imaging**. These entities represent clinical orders that are requested, fulfilled, and tracked throughout the patient care workflow.

**Common Pattern:** All three entities follow a similar lifecycle:
1. **Request** - Order is created by a healthcare provider
2. **Fulfillment** - Order is executed (medication dispensed, test performed, imaging completed)
3. **Completion/Cancellation** - Order reaches terminal state

---

## Entity Comparison Table

| Aspect | Medication | Lab | Imaging |
|--------|-----------|-----|---------|
| **Primary Key** | `id` (string, UUID) | `id` (string, UUID) | `id` (string, UUID) |
| **Business Key** | None (no code field) | `code` (string, auto-generated) | `code` (string, auto-generated) |
| **Foreign Key: Patient** | `patient` (string, required) | `patient` (string, required) | `patient` (string, required) |
| **Foreign Key: Visit** | ❌ None | `visitId` (string, optional) | `visitId` (string, required) |
| **Storage Pattern** | Separate PouchDB document | Separate PouchDB document | Separate PouchDB document |
| **Relationship to Patient** | Referenced (1-to-many) | Referenced (1-to-many) | Referenced (1-to-many) |
| **Extends** | AbstractDBModel | AbstractDBModel | AbstractDBModel |
| **Audit Fields** | `id`, `rev`, `createdAt`, `updatedAt` | `id`, `rev`, `createdAt`, `updatedAt` | `id`, `rev`, `createdAt`, `updatedAt` |
| **Requested By** | `requestedBy` (string, user ID) | `requestedBy` (string, user ID) | `requestedBy` (string, user ID)<br>`requestedByFullName` (string, optional) |
| **Requested On** | `requestedOn` (string, ISO 8601) | `requestedOn` (string, ISO 8601) | `requestedOn` (string, ISO 8601) |
| **Completed On** | `completedOn` (string, ISO 8601) | `completedOn` (string, optional) | `completedOn` (string, optional) |
| **Canceled On** | `canceledOn` (string, ISO 8601) | `canceledOn` (string, optional) | `canceledOn` (string, optional) |
| **Order Details** | `medication` (string, medication name)<br>`quantity` (object: value + unit)<br>`priority` (enum: routine, urgent, asap, stat)<br>`intent` (enum: 8 values) | `type` (string, test type) | `type` (string, imaging type)<br>`fullName` (string, patient name) |
| **Notes/Comments** | `notes` (string) | `notes` (Note[], array of note objects) | `notes` (string, optional) |
| **Result Field** | ❌ None | ✅ `result` (string, optional) | ❌ None |
| **Status Field** | ✅ `status` (8 values) | ✅ `status` (3 values) | ✅ `status` (3 values) |
| **Permissions** | `RequestMedication`<br>`CompleteMedication`<br>`CancelMedication`<br>`ViewMedication`<br>`ViewMedications` | `RequestLab`<br>`CompleteLab`<br>`CancelLab`<br>`ViewLab`<br>`ViewLabs` | `RequestImaging`<br>`ViewImagings` |

---

## Status Values Comparison

### Medication Status (8 Values)

| Status | Description | Terminal? |
|--------|-------------|-----------|
| `draft` | Medication order is being prepared | No |
| `active` | Medication is currently prescribed/being administered | No |
| `on hold` | Medication temporarily suspended | No |
| `canceled` | Medication order cancelled | Yes |
| `completed` | Medication course finished | Yes |
| `entered in error` | Order was created by mistake | Yes |
| `stopped` | Medication discontinued | Yes |
| `unknown` | Status cannot be determined | No |

**Additional Medication Fields:**
- **Intent** (8 values): `proposal`, `plan`, `order`, `original order`, `reflex order`, `filler order`, `instance order`, `option`
- **Priority** (4 values): `routine`, `urgent`, `asap`, `stat`

### Lab Status (3 Values)

| Status | Description | Terminal? |
|--------|-------------|-----------|
| `requested` | Lab test has been ordered | No |
| `completed` | Lab test completed, results available | Yes |
| `canceled` | Lab test cancelled | Yes |

**Simpler Workflow:** Labs have a straightforward request → complete/cancel workflow.

### Imaging Status (3 Values)

| Status | Description | Terminal? |
|--------|-------------|-----------|
| `requested` | Imaging study has been ordered | No |
| `completed` | Imaging study completed | Yes |
| `canceled` | Imaging study cancelled | Yes |

**Identical to Lab:** Imaging uses the same 3-state workflow as labs.

---

## Result Storage Analysis

### Lab Entity - Has Result Field ✅

**Field:** `result?: string` (optional)

**Storage:**
- Free-text string field
- Optional (can be null/undefined)
- Populated when lab status is changed to "completed"

**Validation:**
```typescript
// From validate-lab.ts
if (!lab.result) {
  labError.result = 'labs.requests.error.resultRequiredToComplete'
  labError.message = 'labs.requests.error.unableToComplete'
}
```

**Business Rule:** Result is REQUIRED to complete a lab. Cannot change status to "completed" without entering a result.

**UI Implementation:**
```typescript
// From ViewLab.tsx
<TextFieldWithLabelFormGroup
  name="result"
  label={t('labs.lab.result')}
  value={labToView.result}
  isEditable={isEditable}
  onChange={(event) => onLabChange('result', event.currentTarget.value)}
/>
```

**Result Format:**
- Free-text field (no structured format)
- No standardization (could be numeric, text, or mixed)
- No units or reference ranges stored
- No support for multiple result values (e.g., CBC with multiple components)

**Example Results:**
- "Negative"
- "Glucose: 95 mg/dL"
- "WBC: 7.5, RBC: 4.8, Hemoglobin: 14.2"
- "See attached report"

### Medication Entity - No Result Field ❌

**Result Storage:** None

**Rationale:**
- Medications are orders for treatment, not diagnostic tests
- No "result" concept for medications
- Completion indicates medication was dispensed/administered, not that a result was obtained

**Tracking:**
- Status changes from `active` → `completed` indicate fulfillment
- `completedOn` timestamp records when medication course finished
- No field for "outcome" or "effectiveness" of medication

### Imaging Entity - No Result Field ❌

**Result Storage:** None in the entity model

**Rationale:**
- Imaging results are typically reports or images (DICOM files)
- Too large/complex to store as simple text field
- Likely stored externally (PACS system, file storage)

**Tracking:**
- Status changes from `requested` → `completed` indicate imaging was performed
- `completedOn` timestamp records when imaging was done
- `notes` field (optional string) could contain brief findings or reference to external report

**Implications:**
- No structured storage for radiologist reports
- No image storage in HospitalRun database
- Integration with external PACS/imaging systems would be needed for full functionality

---

## Billing Integration

### Finding: No Billing Fields ❌

**Search Results:** No matches found for:
- `billing`
- `payment`
- `paid`
- `cost`
- `price`
- `charge`
- `invoice`

### Analysis

**No Billing Integration in Current Model:**

None of the three clinical entities (Medication, Lab, Imaging) contain any billing-related fields:
- ❌ No cost/price fields
- ❌ No payment status fields
- ❌ No insurance information
- ❌ No billing codes (CPT, ICD-10)
- ❌ No charge capture mechanism

**Implications:**

1. **Billing is External:** Billing/payment tracking is either:
   - Handled by a separate system
   - Not implemented in v2.0.0-alpha.7
   - Planned for future development

2. **No Revenue Cycle Management:** The system cannot track:
   - Which orders have been billed
   - Which orders have been paid
   - Outstanding charges
   - Insurance claims

3. **No Cost Tracking:** The system cannot:
   - Calculate total cost of care
   - Generate invoices
   - Track medication/supply costs
   - Support financial reporting

4. **Patient Type Field:** The Patient entity has a `type` field with values `charity` and `private`, suggesting billing intent, but no integration with clinical orders.

### Potential Billing Approach (Not Implemented)

**If billing were to be added, likely fields would include:**

```typescript
// Hypothetical billing fields (NOT in current model)
interface BillingInfo {
  billable: boolean           // Is this order billable?
  cost: number                // Cost/charge amount
  currency: string            // Currency code
  billingCode: string         // CPT/procedure code
  billingStatus: 'pending' | 'billed' | 'paid' | 'written-off'
  insuranceClaim?: string     // Reference to insurance claim
  paymentDate?: string        // When payment received
}
```

**Current Workaround:**
- Organizations may track billing in external systems
- Clinical order IDs could be used to link to external billing records
- Patient type (`charity` vs `private`) may determine billing workflow outside the system

---

## Key Insights

💡 **Consistent Order Pattern**  
All three entities follow a similar structure: requested by user, tracked with timestamps, terminal states (completed/canceled). This consistency makes the system easier to understand and maintain.

💡 **Lab is Most Complete**  
Lab is the only entity with a result field and validation requiring results before completion. This reflects the diagnostic nature of lab tests where results are the primary output.

💡 **Medication Has Richest Workflow**  
Medication has 8 status values plus intent and priority fields, reflecting the complex lifecycle of medication orders (draft → active → on hold → completed/stopped/canceled).

💡 **Visit Linking Varies**  
Medication has no visit link, Lab has optional visit link, Imaging has required visit link. This inconsistency suggests different design decisions or use cases for each entity type.

💡 **Code Generation Inconsistency**  
Lab and Imaging have auto-generated `code` fields (business keys), but Medication does not. This makes it harder to reference medications in conversation or documentation.

⚠️ **No Billing Integration**  
Complete absence of billing fields means the system cannot track costs, payments, or revenue. This is a significant gap for real-world hospital operations.

⚠️ **No Imaging Result Storage**  
Imaging has no result field, making it unclear how radiologist reports or findings are stored. Integration with external PACS systems is implied but not documented.

⚠️ **Free-Text Lab Results**  
Lab results are stored as unstructured text, limiting the ability to:
- Perform trend analysis
- Flag abnormal values automatically
- Generate structured reports
- Support clinical decision support

⚠️ **No Structured Medication Dosing**  
Medication quantity is stored as `{value, unit}` but there's no structured dosing schedule (e.g., "twice daily", "every 6 hours"). The `notes` field likely contains this information as free text.

⚠️ **No Order Sets or Protocols**  
No evidence of order sets, protocols, or templates to streamline ordering of common medication/lab/imaging combinations.

🔗 **Permission-Based Workflow**  
Each entity has granular permissions (request, complete, cancel, view) enabling role-based access control and workflow enforcement.

🔗 **Separate Document Storage**  
Unlike visits (embedded in Patient), these entities are stored as separate PouchDB documents, enabling independent querying and scalability.

🔗 **Notes Field Inconsistency**  
Medication and Imaging use simple string notes, while Lab uses structured Note[] array with id, date, text, and deleted flag. This inconsistency suggests different implementation timelines or requirements.

---

## Questions & Todos

### Questions

1. **Why does Medication not have a code field?** Lab and Imaging have auto-generated codes for easy reference. Why not Medication?

2. **Why is visitId optional for Lab but required for Imaging?** What's the business rationale for this difference?

3. **Why does Medication not link to visits?** Medications are often prescribed during visits. Why no visit association?

4. **How are imaging results/reports stored?** With no result field, where do radiologist reports go?

5. **Is there a separate billing system?** How do organizations track costs and payments?

6. **Why are lab results free-text?** Would structured results (numeric values, units, reference ranges) be more useful?

7. **How are medication dosing schedules stored?** Is it all in the notes field?

8. **Are there plans to add billing integration?** Is this a future feature or intentionally out of scope?

9. **How are order sets or protocols handled?** Can users create templates for common orders?

10. **Why does Lab use Note[] while others use string notes?** What drove this design difference?

11. **How are medication refills handled?** Is there a refill mechanism or must new orders be created?

12. **How are stat/urgent orders prioritized?** Does the system alert users or prioritize display?

### Todos

- [ ] Document the complete lifecycle for each entity (request → complete/cancel)
- [ ] Analyze the medication intent field usage and business rules
- [ ] Analyze the medication priority field and how it affects workflow
- [ ] Document how imaging results are accessed (external system integration?)
- [ ] Investigate if there's a billing module or external billing integration
- [ ] Document the lab result entry workflow and UI
- [ ] Analyze the medication completion workflow (dispensing process)
- [ ] Document the imaging completion workflow
- [ ] Investigate if there are order sets or protocol features
- [ ] Document the permission model for each entity in detail
- [ ] Analyze the search and filtering capabilities for each entity
- [ ] Document the reporting capabilities (if any) for clinical orders
- [ ] Investigate if there's a medication formulary or drug database
- [ ] Document the lab test catalog or compendium
- [ ] Investigate the imaging modality types and standardization
- [ ] Analyze the notification system for completed orders (if any)
- [ ] Document the audit trail for order modifications
- [ ] Investigate if there's a medication reconciliation feature
- [ ] Document the order cancellation workflow and permissions
- [ ] Analyze the performance implications of separate document storage vs. embedding

---

**End of Document**
