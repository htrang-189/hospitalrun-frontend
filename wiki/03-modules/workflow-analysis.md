# Workflow Analysis - The Glue Between Entities

**Last Updated:** 2026-04-11  
**Analyzed By:** TrangN via Kiro/bmad

---

## Related Files
- `src/labs/requests/NewLabRequest.tsx` - Lab order creation workflow
- `src/imagings/requests/NewImagingRequest.tsx` - Imaging order creation workflow
- `src/labs/hooks/useRequestLab.ts` - Lab request hook
- `src/imagings/hooks/useRequestImaging.tsx` - Imaging request hook
- `src/labs/hooks/useCompleteLab.ts` - Lab completion hook
- `src/labs/hooks/useUpdateLab.ts` - Lab update hook
- `src/labs/ViewLab.tsx` - Lab view and update UI
- `src/patients/hooks/useAddVisit.tsx` - Visit creation hook

---

## Table of Contents
1. [Overview](#overview)
2. [Lab/Imaging Request Workflow](#labimagingimaging-request-workflow)
3. [Visit Selection Mechanism](#visit-selection-mechanism)
4. [Master Data Analysis](#master-data-analysis)
5. [Cross-Entity Logic](#cross-entity-logic)
6. [Workflow Gaps](#workflow-gaps)
7. [Key Insights](#key-insights)
8. [Questions & Todos](#questions--todos)

---

## Overview

This document analyzes the "glue" that connects entities in HospitalRun - the workflows, triggers, and business logic that orchestrate interactions between Patients, Visits, Labs, Imaging, and Medications. We examine:

1. **How clinical orders are triggered** from patient visits
2. **Where master data comes from** (medication lists, lab types, etc.)
3. **Cross-entity logic** that updates related entities based on events

---

## Lab/Imaging Request Workflow

### Workflow Pattern: Patient → Visit → Clinical Order

Both Lab and Imaging follow the same workflow pattern:

```
┌─────────────────────────────────────────────────────────────┐
│                    1. Select Patient                         │
│  User searches for patient using Typeahead component        │
│  Search: PatientRepository.search(query)                    │
│  Display: Patient.fullName (Patient.code)                   │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                    2. Load Patient Visits                    │
│  When patient selected:                                      │
│  - Retrieve patient.visits[] array                          │
│  - Map to SelectOptions:                                     │
│    label: "{visit.type} at {visit.startDateTime}"          │
│    value: visit.id                                          │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                    3. Select Visit                           │
│  User selects visit from dropdown                           │
│  visitId is captured for the order                          │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                    4. Enter Order Details                    │
│  Lab: type (free-text), notes (optional)                   │
│  Imaging: type (free-text), status, notes (optional)       │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                    5. Save Order                             │
│  Lab: LabRepository.save()                                  │
│  Imaging: ImagingRepository.save()                          │
│  Auto-populate:                                              │
│  - requestedBy: current user ID                             │
│  - requestedOn: current timestamp                           │
│  - status: 'requested' (default)                            │
└─────────────────────────────────────────────────────────────┘
```

### Lab Request Code Flow

**File:** `src/labs/requests/NewLabRequest.tsx`

```typescript
// 1. Patient Selection
const onPatientChange = (patient: Patient) => {
  if (patient) {
    // 2. Extract visits from patient
    const visits = patient.visits?.map((v) => ({
      label: `${v.type} at ${format(new Date(v.startDateTime), 'yyyy-MM-dd hh:mm a')}`,
      value: v.id,
    })) as SelectOption[]

    setVisitOptions(visits)
    
    // 3. Store patient ID
    setNewLabRequest((previousNewLabRequest) => ({
      ...previousNewLabRequest,
      patient: patient.id,
      fullName: patient.fullName,
    }))
  }
}

// 4. Visit Selection
const onVisitChange = (visitId: string) => {
  setNewLabRequest((previousNewLabRequest) => ({
    ...previousNewLabRequest,
    visitId,  // Link to visit
  }))
}

// 5. Save Lab Order
const onSave = async () => {
  const newLab = await mutate(newLabRequest as Lab)
  // Lab is saved as separate document with patient and visitId references
}
```

**Hook:** `src/labs/hooks/useRequestLab.ts`

```typescript
function requestLab(newLab: Lab): Promise<Lab> {
  const requestLabErrors = validateLabRequest(newLab)

  if (isEmpty(requestLabErrors)) {
    // Auto-populate requestedOn timestamp
    newLab.requestedOn = new Date(Date.now().valueOf()).toISOString()
    return LabRepository.save(newLab)
  }

  throw requestLabErrors
}
```

### Imaging Request Code Flow

**File:** `src/imagings/requests/NewImagingRequest.tsx`

**Identical Pattern to Lab:**
1. Patient selection via Typeahead
2. Visits loaded from `patient.visits[]`
3. Visit dropdown populated with formatted visit info
4. User enters imaging type and notes
5. Order saved with auto-populated fields

**Hook:** `src/imagings/hooks/useRequestImaging.tsx`

```typescript
function requestImagingWrapper(user: any) {
  return async function requestImaging(request: ImagingRequest): Promise<void> {
    const error = validateImagingRequest(request)

    if (!isEmpty(error)) {
      throw error
    }

    // Auto-populate user and timestamp
    await ImagingRepository.save({
      ...request,
      requestedBy: user?.id || '',
      requestedByFullName: user?.fullName || '',
      requestedOn: new Date(Date.now()).toISOString(),
    } as Imaging)
  }
}
```

### Key Workflow Characteristics

**1. Visit is NOT Created Automatically**
- Visits must be created BEFORE ordering labs/imaging
- No automatic visit creation during order entry
- If patient has no visits, the visit dropdown is empty
- User must navigate to patient record to create visit first

**2. Visit Selection is Required**
- Lab: `visitId` is optional in model but UI requires selection
- Imaging: `visitId` is required in model and UI
- No ability to create "standalone" orders without visit context

**3. Patient Search is Real-Time**
- Uses `PatientRepository.search(query)` for typeahead
- Searches on `fullName` and `code` fields
- Results displayed as: "Full Name (Code)"

**4. Visit Display Format**
- Format: `"{visit.type} at {formatted startDateTime}"`
- Example: "Emergency at 2026-04-11 09:30 AM"
- Sorted by visit order in array (no explicit sorting)

---

## Visit Selection Mechanism

### How Visits are Loaded

**Source:** Patient entity's embedded `visits[]` array

```typescript
// When patient is selected
const patient = await PatientRepository.find(patientId)

// Visits are already loaded (embedded in patient document)
const visits = patient.visits  // Visit[] array

// Transform to dropdown options
const visitOptions = visits?.map((v) => ({
  label: `${v.type} at ${format(new Date(v.startDateTime), 'yyyy-MM-dd hh:mm a')}`,
  value: v.id,
}))
```

### Visit Filtering

**Current Implementation:** No filtering

- All visits are shown regardless of status
- Finished/cancelled visits are selectable
- No date range filtering
- No status-based filtering

**Implications:**
- Users can associate orders with old/closed visits
- No guidance on which visit is "current"
- Potential for data quality issues

### Visit Creation Workflow

**File:** `src/patients/hooks/useAddVisit.tsx`

```typescript
async function addVisit(request: AddVisitRequest): Promise<Visit[]> {
  const error = validateVisit(request.visit)
  if (isEmpty(error)) {
    const patient = await PatientRepository.find(request.patientId)
    const visits = patient.visits || ([] as Visit[])
    
    // Add new visit to array
    visits.push({
      id: uuid(),
      createdAt: new Date(Date.now().valueOf()).toISOString(),
      ...request.visit,
    })
    
    // Save entire patient document
    await PatientRepository.saveOrUpdate({
      ...patient,
      visits,
    })
    return visits
  }
  throw error
}
```

**Key Points:**
- Visits are added to patient's embedded array
- Entire patient document is saved (not just visit)
- No separate visit creation from order entry screens

---

## Master Data Analysis

### Finding: No Master Data Tables ❌

**Search Results:** No matches found for:
- `catalog`
- `formulary`
- `masterData`
- `medicationList`
- `labTypes`
- `testCatalog`

### Medication Master Data

**Current Implementation:** Free-text entry

```typescript
// In Medication entity
export default interface Medication extends AbstractDBModel {
  medication: string  // Free-text field, no lookup
  // ...
}
```

**No Medication Formulary:**
- ❌ No medication database
- ❌ No drug catalog
- ❌ No standardized medication names
- ❌ No dosing guidelines
- ❌ No drug interactions checking
- ❌ No contraindications

**Implications:**
- Users type medication names manually
- Prone to typos and variations ("Aspirin" vs "aspirin" vs "ASA")
- No standardization for reporting
- No clinical decision support
- No inventory management

### Lab Type Master Data

**Current Implementation:** Free-text entry

```typescript
// In NewLabRequest.tsx
<TextInputWithLabelFormGroup
  name="labType"
  label={t('labs.lab.type')}
  isRequired
  isEditable
  value={newLabRequest.type}
  onChange={onLabTypeChange}
/>
```

**No Lab Test Catalog:**
- ❌ No lab test database
- ❌ No standardized test names
- ❌ No test descriptions or instructions
- ❌ No reference ranges
- ❌ No test panels or profiles
- ❌ No specimen requirements

**Observed Values (from tests):**
- "Blood Test"
- "CBC"
- "Glucose"
- "emergency incident" (appears to be test data error)

**Implications:**
- Users type lab names manually
- Inconsistent naming ("CBC" vs "Complete Blood Count" vs "cbc")
- No standardization for reporting
- No automatic result interpretation
- No test ordering guidance

### Imaging Type Master Data

**Current Implementation:** Free-text entry

```typescript
// In NewImagingRequest.tsx
<TextInputWithLabelFormGroup
  name="imagingType"
  label={t('imagings.imaging.type')}
  isRequired
  isEditable
  value={newImagingRequest.type}
  onChange={onImagingTypeChange}
/>
```

**No Imaging Modality Catalog:**
- ❌ No imaging type database
- ❌ No standardized modality names
- ❌ No imaging protocols
- ❌ No contrast requirements
- ❌ No patient preparation instructions

**Observed Values (from tests):**
- "X-Ray"
- "CT"
- "MRI"
- "imaging type"

**Implications:**
- Users type imaging types manually
- Inconsistent naming ("X-Ray" vs "Xray" vs "X-ray" vs "Radiograph")
- No standardization for reporting
- No protocol guidance
- No safety checks (e.g., contrast allergies)

### Visit Type Master Data

**Current Implementation:** Free-text entry

```typescript
// In VisitForm.tsx
<TextInputWithLabelFormGroup
  label={t('patient.visits.type')}
  name="type"
  value={visit.type}
  onChange={(event) => onFieldChange('type', event.currentTarget.value)}
/>
```

**No Visit Type Catalog:**
- ❌ No visit type database
- ❌ No standardized visit categories

**Observed Values (from tests):**
- "emergency"
- "routine"
- "standard type"
- "follow-up" (implied)

### Master Data Summary

| Entity | Field | Current Implementation | Master Data? |
|--------|-------|------------------------|--------------|
| Medication | `medication` | Free-text input | ❌ None |
| Medication | `quantity.unit` | Free-text input | ❌ None |
| Lab | `type` | Free-text input | ❌ None |
| Imaging | `type` | Free-text input | ❌ None |
| Visit | `type` | Free-text input | ❌ None |
| Visit | `location` | Free-text input | ❌ None |
| Patient | `sex` | Dropdown (4 values) | ✅ Hardcoded |
| Patient | `bloodType` | Dropdown (9 values) | ✅ Hardcoded |
| Patient | `type` | Dropdown (2 values) | ✅ Hardcoded |
| Visit | `status` | Dropdown (7 values) | ✅ Enum |
| Lab | `status` | Dropdown (3 values) | ✅ Enum |
| Imaging | `status` | Dropdown (3 values) | ✅ Enum |
| Medication | `status` | Dropdown (8 values) | ✅ Enum |
| Medication | `intent` | Dropdown (8 values) | ✅ Enum |
| Medication | `priority` | Dropdown (4 values) | ✅ Enum |

**Pattern:**
- Status fields use enums (good)
- Clinical content fields use free-text (problematic)
- No external master data tables or catalogs

---

## Cross-Entity Logic

### Finding: No Automatic Cross-Entity Updates ❌

**Search Results:** No matches found for:
- `updatePatientStatus`
- `updateVisitStatus`
- `onLabComplete`
- `onResultReceived`

### Lab Completion Workflow

**File:** `src/labs/hooks/useCompleteLab.ts`

```typescript
function completeLab(lab: Lab): Promise<Lab> {
  const completeLabErrors = validateLabComplete(lab)

  if (isEmpty(completeLabErrors)) {
    const completedLab: Lab = {
      ...lab,
      completedOn: new Date(Date.now().valueOf()).toISOString(),
      status: 'completed',
    }

    return LabRepository.saveOrUpdate(completedLab)
  }

  throw completeLabErrors
}
```

**What Happens:**
- ✅ Lab status changes to 'completed'
- ✅ `completedOn` timestamp is set
- ✅ Result must be entered (validation)
- ❌ Patient is NOT updated
- ❌ Visit is NOT updated
- ❌ No notifications sent
- ❌ No workflow triggers

### Lab Update Workflow

**File:** `src/labs/hooks/useUpdateLab.ts`

```typescript
function updateLab(labToUpdate: Lab): Promise<Lab> {
  return LabRepository.saveOrUpdate(labToUpdate)
}
```

**What Happens:**
- ✅ Lab is saved
- ✅ Query cache is invalidated
- ❌ No cross-entity updates
- ❌ No event triggers

### Visit Status Updates

**Analysis:** No code found that automatically updates visit status based on:
- Lab completion
- Imaging completion
- Medication administration
- Time elapsed
- Any other trigger

**Implication:** Visit status is entirely manual

### Patient Status Updates

**Analysis:** No code found that automatically updates patient based on:
- Lab results (e.g., flagging abnormal results)
- Diagnosis changes
- Visit completion
- Any clinical events

**Implication:** No automated patient status tracking

### Cross-Entity Logic Summary

| Trigger Event | Expected Action | Current Implementation |
|---------------|-----------------|------------------------|
| Lab completed | Update visit status? | ❌ None |
| Lab completed | Notify ordering provider? | ❌ None |
| Lab completed | Flag abnormal results? | ❌ None |
| Imaging completed | Update visit status? | ❌ None |
| Imaging completed | Notify ordering provider? | ❌ None |
| Medication completed | Update patient record? | ❌ None |
| Visit finished | Complete pending orders? | ❌ None |
| Visit finished | Generate discharge summary? | ❌ None |
| Diagnosis added | Update care plans? | ❌ None |
| Diagnosis added | Suggest relevant orders? | ❌ None |
| Allergy added | Check medication interactions? | ❌ None |
| All visit orders complete | Suggest visit completion? | ❌ None |

**Pattern:** Zero cross-entity automation

---

## Workflow Gaps

### 1. No Visit-Driven Order Entry

**Gap:** Cannot create orders directly from visit context

**Current Flow:**
```
Patient View → Visit Tab → View Visit → [No order buttons]
                                        ↓
                                   [Must navigate to]
                                        ↓
Labs Menu → New Lab → Select Patient → Select Visit
```

**Desired Flow:**
```
Patient View → Visit Tab → View Visit → [Order Lab Button]
                                              ↓
                                    [Lab form pre-filled with patient & visit]
```

**Impact:** Extra navigation steps, potential for selecting wrong visit

### 2. No Visit Context in Order List

**Gap:** When viewing labs/imaging lists, visit context is not displayed

**Current:** Lab list shows patient, type, status, dates
**Missing:** Which visit was this order associated with?

**Impact:** Difficult to understand clinical context of orders

### 3. No Order Status Rollup

**Gap:** No summary of pending/completed orders for a visit

**Missing:**
- Visit view doesn't show associated orders
- No "all orders complete" indicator
- No pending order count

**Impact:** Cannot easily assess visit completion status

### 4. No Master Data Management

**Gap:** No UI or API for managing master data

**Missing:**
- Medication formulary management
- Lab test catalog management
- Imaging protocol management
- Visit type configuration

**Impact:** Inconsistent data entry, no standardization

### 5. No Clinical Decision Support

**Gap:** No automated checks or suggestions

**Missing:**
- Drug-allergy checking
- Drug-drug interaction checking
- Duplicate order detection
- Abnormal result flagging
- Protocol-based order sets

**Impact:** Potential safety issues, inefficient ordering

### 6. No Workflow Automation

**Gap:** No automatic status updates or triggers

**Missing:**
- Auto-complete visit when all orders done
- Auto-notify providers of results
- Auto-flag critical results
- Auto-suggest follow-up orders

**Impact:** Manual workflow, potential delays

### 7. No Order Modification Workflow

**Gap:** Cannot modify orders after creation

**Current:** Labs can be updated (notes, results) but not core fields
**Missing:** 
- Change order type
- Change visit association
- Modify order details

**Impact:** Must cancel and recreate orders for changes

---

## Key Insights

💡 **Visit-Centric Ordering**  
The workflow is designed around visits as the primary context for clinical orders. All labs and imaging must be associated with a visit, enforcing temporal and contextual grouping.

💡 **Embedded Visit Pattern Enables Workflow**  
Because visits are embedded in the patient document, they're immediately available when creating orders. No separate query needed to load visits.

💡 **No Master Data = Maximum Flexibility**  
The absence of master data catalogs provides ultimate flexibility but sacrifices standardization, reporting, and decision support capabilities.

💡 **Manual Workflow Philosophy**  
The system appears designed for manual, user-driven workflows with no automation. This is simple but requires discipline and training.

💡 **Repository Pattern Isolation**  
Each entity has its own repository with no cross-repository logic. This maintains clean separation but prevents cross-entity automation.

⚠️ **No Visit Filtering**  
All visits are shown in order dropdowns regardless of status or date. Users can associate orders with closed/old visits, potentially causing confusion.

⚠️ **No Reverse Navigation**  
Cannot navigate from order back to visit. The relationship is one-way (order → visit) with no UI support for visit → orders.

⚠️ **Free-Text Master Data Risk**  
All clinical content (medication names, lab types, imaging types) is free-text, leading to:
- Inconsistent naming
- Typos and variations
- Difficult reporting
- No decision support

⚠️ **No Cross-Entity Validation**  
No validation checks across entities (e.g., "Does this patient have an active visit?" or "Is this medication contraindicated with patient allergies?").

⚠️ **No Workflow State Machine**  
No enforcement of workflow sequences (e.g., "Cannot complete visit until all orders are completed"). All state changes are independent.

🔗 **React Query Cache Coordination**  
The system uses React Query's cache invalidation to coordinate UI updates across components, providing a lightweight form of event coordination.

🔗 **Permission-Based Workflow Control**  
Permissions control who can perform actions (request, complete, cancel), providing role-based workflow enforcement at the UI level.

🔗 **Typeahead Search Pattern**  
Patient selection uses real-time search with typeahead, providing good UX for finding patients quickly.

---

## Questions & Todos

### Questions

1. **Why no visit filtering in order entry?** Should only active/current visits be selectable?

2. **Why no master data tables?** Was this a deliberate design decision for simplicity, or a missing feature?

3. **Why no cross-entity automation?** Is this intentional (keep it simple) or planned for future?

4. **Why can't orders be created from visit context?** This seems like a natural workflow.

5. **How do users know which visit is "current"?** No indication of active vs. closed visits.

6. **Why is visitId optional for labs but required for imaging?** What's the business rationale?

7. **How do users handle order modifications?** Must they cancel and recreate?

8. **Is there a plan for clinical decision support?** Drug interactions, allergy checking, etc.?

9. **How do users track visit completion?** No indicator of pending orders.

10. **Why no order sets or protocols?** Common order combinations could be templated.

11. **How do users handle stat/urgent orders?** No priority-based workflow.

12. **Is there a notification system?** How do providers know when results are ready?

### Todos

- [ ] Document the complete order lifecycle (request → update → complete/cancel)
- [ ] Analyze the medication ordering workflow (currently not examined)
- [ ] Document the appointment scheduling workflow and its relationship to visits
- [ ] Investigate if there's a way to create visits from order entry screens
- [ ] Document the patient history view and how it aggregates orders
- [ ] Analyze the search and filtering capabilities for orders
- [ ] Document the reporting capabilities (if any) for clinical orders
- [ ] Investigate if there's a way to view all orders for a visit
- [ ] Document the permission model for each workflow step
- [ ] Analyze the query cache invalidation strategy
- [ ] Investigate if there's a way to modify orders after creation
- [ ] Document the order cancellation workflow and business rules
- [ ] Analyze the validation rules for each order type
- [ ] Investigate if there's a duplicate order detection mechanism
- [ ] Document the result entry workflow for labs
- [ ] Analyze the imaging result workflow (external PACS integration?)
- [ ] Investigate if there's a medication administration workflow
- [ ] Document the visit completion workflow
- [ ] Analyze the discharge summary generation (if any)
- [ ] Investigate if there's a follow-up order suggestion mechanism

---

**End of Document**
