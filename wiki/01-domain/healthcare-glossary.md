# Healthcare Glossary - Ubiquitous Language

**Last Updated:** 2026-04-11  
**Analyzed By:** TrangN via Kiro/bmad

---

## Related Files
- `src/shared/model/` - All domain entity definitions
- `src/patients/models/` - Patient-specific models
- `src/shared/model/Permissions.ts` - Access control definitions
- `src/shared/config/pouchdb.ts` - Relational schema configuration

---

## Table of Contents
1. [Overview](#overview)
2. [Ubiquitous Language (Glossary)](#ubiquitous-language-glossary)
3. [Entity Relationships](#entity-relationships)
4. [Status Enumerations](#status-enumerations)
5. [Key Insights](#key-insights)
6. [Questions & Todos](#questions--todos)

---

## Overview

This document defines the **Ubiquitous Language** for the HospitalRun domain. These terms represent the core healthcare concepts used consistently across the codebase, business logic, and user interface. The language bridges the gap between healthcare professionals (domain experts) and the technical implementation.

**Domain-Driven Design Classification:**
- **Entity** - Has a unique identity that persists over time (has `id` field)
- **Value Object** - Defined by its attributes, no unique identity
- **Aggregate Root** - Entity that controls access to a cluster of related entities

---

## Ubiquitous Language (Glossary)

| Term | Business Definition | Technical Type | Key Attributes | Used In |
|------|---------------------|----------------|----------------|---------|
| **Patient** | A person receiving or registered to receive medical care at the hospital. The central entity around which all clinical activities revolve. Contains demographics, medical history, and relationships to all clinical records. | Aggregate Root (Entity) | `id`, `code`, `fullName`, `sex`, `dateOfBirth`, `bloodType`, `preferredLanguage`, `occupation`, `allergies[]`, `diagnoses[]`, `carePlans[]`, `careGoals[]`, `visits[]`, `notes[]`, `relatedPersons[]` | `src/shared/model/Patient.ts`<br>`src/patients/` (entire module)<br>`src/shared/config/pouchdb.ts` (schema) |
| **Allergy** | A documented hypersensitivity or adverse reaction to a substance (medication, food, environmental factor) that a patient has experienced or is at risk for. Critical for medication safety and treatment planning. | Value Object | `id`, `name` | `src/shared/model/Allergy.ts`<br>`src/patients/allergies/`<br>Embedded in `Patient.allergies[]` |
| **Diagnosis** | A medical condition or disease identified by a healthcare provider for a patient. Tracks the lifecycle of the condition from onset through resolution. Links to care plans and visits. | Entity | `id`, `name`, `diagnosisDate`, `onsetDate`, `abatementDate`, `status` (active, recurrence, relapse, inactive, remission, resolved), `note`, `visit` | `src/shared/model/Diagnosis.ts`<br>`src/patients/diagnoses/`<br>Embedded in `Patient.diagnoses[]` |
| **Appointment** | A scheduled meeting between a patient and healthcare provider at a specific time and location for a specific medical reason. Belongs to a patient. | Entity | `id`, `startDateTime`, `endDateTime`, `patient`, `location`, `reason`, `type`, `createdAt`, `updatedAt` | `src/shared/model/Appointment.ts`<br>`src/scheduling/appointments/`<br>`src/patients/appointments/`<br>Related to `Patient` (1-to-many) |
| **Medication** | A prescription or medication request for a patient. Tracks the medication lifecycle from request through completion or cancellation. Includes dosage, priority, and fulfillment status. | Entity | `id`, `medication`, `patient`, `requestedBy`, `requestedOn`, `completedOn`, `canceledOn`, `status` (draft, active, on hold, canceled, completed, entered in error, stopped, unknown), `intent` (proposal, plan, order, etc.), `priority` (routine, urgent, asap, stat), `quantity`, `notes` | `src/shared/model/Medication.ts`<br>`src/medications/`<br>`src/patients/medications/`<br>Related to `Patient` (1-to-many) |
| **Incident** | A safety event, error, near-miss, or adverse occurrence in the hospital that requires documentation and tracking. May or may not be patient-specific. Supports quality improvement and risk management. | Entity | `id`, `code`, `reportedBy`, `reportedOn`, `date`, `department`, `category`, `categoryItem`, `description`, `status` (reported, resolved), `resolvedOn`, `patient?` (optional) | `src/shared/model/Incident.ts`<br>`src/incidents/`<br>Standalone entity (not always patient-related) |
| **Lab** | A laboratory test or diagnostic procedure requested for a patient. Tracks the request, execution, and results of laboratory work. | Entity | `id`, `code`, `patient`, `type`, `requestedBy`, `requestedOn`, `completedOn`, `canceledOn`, `status` (requested, completed, canceled), `result`, `notes[]`, `visitId` | `src/shared/model/Lab.ts`<br>`src/labs/`<br>`src/patients/labs/`<br>Related to `Patient` (1-to-many) |
| **Imaging** | A radiology or medical imaging request (X-ray, CT, MRI, ultrasound, etc.) for a patient. Tracks the imaging order and completion status. | Entity | `id`, `code`, `patient`, `fullName`, `type`, `requestedBy`, `requestedByFullName`, `requestedOn`, `completedOn`, `canceledOn`, `status` (requested, completed, canceled), `visitId`, `notes` | `src/shared/model/Imaging.ts`<br>`src/imagings/`<br>Related to `Patient` (1-to-many) |
| **Visit** | A patient encounter or episode of care at the hospital. Represents a single visit session from arrival through discharge. Tracks visit status, reason, and timing. Links to labs, diagnoses, and other clinical activities. | Entity | `id`, `startDateTime`, `endDateTime`, `type`, `status` (planned, arrived, triaged, in progress, on leave, finished, cancelled), `reason`, `location`, `createdAt`, `updatedAt` | `src/shared/model/Visit.ts`<br>`src/patients/visits/`<br>Embedded in `Patient.visits[]` |
| **Care Plan** | A structured plan of care for managing a patient's diagnosis. Defines treatment goals, interventions, and timelines. Links to a specific diagnosis. | Entity | `id`, `status` (draft, active, on hold, revoked, completed, unknown), `intent` (proposal, plan, order, option), `title`, `description`, `startDate`, `endDate`, `createdOn`, `diagnosisId`, `note` | `src/shared/model/CarePlan.ts`<br>`src/patients/care-plans/`<br>Embedded in `Patient.carePlans[]` |
| **Care Goal** | A specific, measurable objective for a patient's care. Tracks progress toward achieving health outcomes. Part of care planning and patient management. | Entity | `id`, `status` (proposed, planned, accepted, active, on hold, completed, cancelled, rejected), `achievementStatus` (in progress, improving, worsening, no change, achieved, not achieved, no progress, not attainable), `priority` (high, medium, low), `description`, `startDate`, `dueDate`, `createdOn`, `note` | `src/shared/model/CareGoal.ts`<br>`src/patients/care-goals/`<br>Embedded in `Patient.careGoals[]` |
| **Note** | A clinical note or comment added to a patient record, lab, or other entity. Provides additional context and documentation. Supports soft deletion. | Value Object | `id`, `date`, `text`, `deleted` | `src/shared/model/Note.ts`<br>Embedded in `Patient.notes[]`, `Lab.notes[]` |
| **Related Person** | A person related to the patient (family member, emergency contact, guardian, etc.). Stores the relationship type and links to another patient record if that person is also a patient. | Value Object | `id`, `patientId`, `type` | `src/shared/model/RelatedPerson.ts`<br>`src/patients/related-persons/`<br>Embedded in `Patient.relatedPersons[]` |
| **Contact Information** | A patient's contact details including phone numbers, email addresses, and physical addresses. Each piece of contact info has a type (e.g., home, work, mobile). | Value Object | `phoneNumbers[]`, `emails[]`, `addresses[]` (each with `id`, `value`, `type`) | `src/shared/model/ContactInformation.ts`<br>Mixed into `Patient` interface<br>`src/patients/ContactInfo.tsx` |
| **Name** | A person's name components following healthcare naming standards. Supports prefixes, suffixes, and full name display. | Value Object | `prefix`, `givenName`, `familyName`, `suffix`, `fullName` | `src/shared/model/Name.ts`<br>Mixed into `Patient` and `User` interfaces |
| **User** | A system user (healthcare provider, administrator, etc.) who can authenticate and perform actions in the system. Has associated permissions. | Entity | `id`, `givenName`, `familyName`, `fullName` (extends Name) | `src/shared/model/User.ts`<br>`src/user/` |
| **Abstract DB Model** | Base interface for all persistable entities. Provides common database fields for tracking, versioning, and auditing. | Base Interface | `id`, `rev` (revision), `createdAt`, `updatedAt` | `src/shared/model/AbstractDBModel.ts`<br>Extended by all entity types |
| **Permissions** | Access control rights that determine what actions a user can perform on different entities. Follows a resource-action pattern (e.g., `read:patients`, `write:labs`). | Enumeration | 30+ permission values covering CRUD operations on all entities | `src/shared/model/Permissions.ts`<br>Used throughout for authorization |

---

## Entity Relationships

### Core Relationships (from PouchDB Schema)

```
Patient (Aggregate Root)
  ├── hasMany → Appointments
  ├── hasMany → Labs
  ├── hasMany → Medications
  ├── hasMany → Imagings
  ├── embeds → Allergies[]
  ├── embeds → Diagnoses[]
  ├── embeds → Care Plans[]
  ├── embeds → Care Goals[]
  ├── embeds → Visits[]
  ├── embeds → Notes[]
  └── embeds → Related Persons[]

Appointment
  └── belongsTo → Patient

Lab
  └── belongsTo → Patient

Medication
  └── belongsTo → Patient

Imaging
  └── belongsTo → Patient

Incident
  └── optionally references → Patient
```

### Relationship Types

**1. One-to-Many (Relational)**
- Stored as separate documents in PouchDB
- Linked via foreign key (`patient` field)
- Examples: Patient → Appointments, Patient → Labs, Patient → Medications, Patient → Imagings

**2. Embedded (Compositional)**
- Stored as arrays within the Patient document
- No separate database documents
- Examples: Patient.allergies, Patient.diagnoses, Patient.carePlans, Patient.careGoals, Patient.visits, Patient.notes, Patient.relatedPersons

**3. Standalone**
- Independent entities not always linked to patients
- Example: Incident (can be patient-specific or hospital-wide)

---

## Status Enumerations

### Diagnosis Status
```typescript
- active: Currently affecting the patient
- recurrence: Condition has returned after resolution
- relapse: Condition has worsened after improvement
- inactive: Not currently active but not resolved
- remission: Symptoms have decreased or disappeared
- resolved: Condition is completely resolved
```

### Visit Status
```typescript
- planned: Visit is scheduled but patient hasn't arrived
- arrived: Patient has checked in
- triaged: Initial assessment completed
- in progress: Visit is actively occurring
- on leave: Patient temporarily left during visit
- finished: Visit completed normally
- cancelled: Visit was cancelled
```

### Care Plan Status
```typescript
- draft: Plan is being created
- active: Plan is currently being executed
- on hold: Plan is temporarily suspended
- revoked: Plan has been cancelled/withdrawn
- completed: Plan has been fully executed
- unknown: Status cannot be determined
```

### Care Goal Status
```typescript
- proposed: Goal has been suggested
- planned: Goal is planned but not started
- accepted: Goal has been accepted by patient/team
- active: Goal is being actively worked on
- on hold: Goal is temporarily paused
- completed: Goal has been achieved
- cancelled: Goal is no longer relevant
- rejected: Goal was not accepted
```

### Care Goal Achievement Status
```typescript
- in progress: Working toward goal
- improving: Making positive progress
- worsening: Condition is deteriorating
- no change: No progress or regression
- achieved: Goal successfully met
- not achieved: Goal was not met
- no progress: No movement toward goal
- not attainable: Goal cannot be achieved
```

### Medication Status
```typescript
- draft: Medication order is being prepared
- active: Medication is currently prescribed
- on hold: Medication temporarily suspended
- canceled: Medication order cancelled
- completed: Medication course finished
- entered in error: Order was created by mistake
- stopped: Medication discontinued
- unknown: Status cannot be determined
```

### Medication Intent
```typescript
- proposal: Suggested medication
- plan: Planned medication
- order: Active medication order
- original order: Initial order
- reflex order: Automatic order based on results
- filler order: Order to fulfill another order
- instance order: Specific instance of administration
- option: Optional medication
```

### Medication Priority
```typescript
- routine: Normal priority
- urgent: Needs prompt attention
- asap: As soon as possible
- stat: Immediately (emergency)
```

### Lab/Imaging Status
```typescript
- requested: Order has been placed
- completed: Results are available
- canceled: Order was cancelled
```

### Incident Status
```typescript
- reported: Incident has been documented
- resolved: Incident has been addressed and closed
```

---

## Key Insights

💡 **Patient as Aggregate Root**  
The Patient entity is the central aggregate root in the domain model. All clinical activities (appointments, labs, medications, imaging) are organized around the patient. This reflects the patient-centric nature of healthcare delivery.

💡 **Embedded vs. Referenced Relationships**  
The system uses two relationship patterns:
- **Embedded** (allergies, diagnoses, care plans, visits) - Tightly coupled data that's always loaded with the patient
- **Referenced** (appointments, labs, medications, imaging) - Separate entities that can be queried independently

This design optimizes for common access patterns while maintaining data integrity.

💡 **Rich Status Tracking**  
Every clinical entity has detailed status enumerations that reflect real-world healthcare workflows. For example, medications track 8 different statuses and 4 priority levels, enabling precise workflow management.

💡 **Audit Trail Built-In**  
All entities extending `AbstractDBModel` include `createdAt`, `updatedAt`, and `rev` (revision) fields, providing automatic audit trails for compliance and troubleshooting.

💡 **Temporal Data Modeling**  
Many entities track multiple timestamps (requested/completed/canceled dates) to support workflow tracking, reporting, and compliance requirements.

⚠️ **No Encounter/Episode Entity**  
Unlike many healthcare systems, HospitalRun uses "Visit" instead of the more common "Encounter" or "Episode" terminology. This may cause confusion for healthcare professionals familiar with HL7 FHIR or other standards.

⚠️ **Incident Patient Link is Optional**  
Incidents can be patient-specific or hospital-wide (patient field is optional). This dual nature requires careful handling in queries and reports.

⚠️ **Embedded Data Scalability**  
Storing arrays (allergies, diagnoses, visits) directly in the Patient document could cause performance issues for patients with extensive medical histories. PouchDB document size limits may become a constraint.

🔗 **Permission-Based Access Control**  
The Permissions enum defines 30+ granular permissions following a `action:resource` pattern (e.g., `write:patients`, `cancel:lab`). This enables fine-grained role-based access control (RBAC).

🔗 **Soft Delete Pattern**  
Notes use a `deleted` boolean flag for soft deletion rather than physical deletion, preserving audit trails and enabling recovery.

🔗 **Relational Pouch Schema**  
The PouchDB schema configuration in `src/shared/config/pouchdb.ts` defines the relationships between entities, enabling SQL-like joins in a NoSQL database.

---

## Questions & Todos

### Questions
1. **Encounter vs. Visit**: Why was "Visit" chosen over the more standard "Encounter" terminology? Does this affect interoperability with other systems?
2. **Embedded Data Limits**: What is the maximum number of visits/diagnoses/allergies a patient can have before document size becomes an issue?
3. **Care Plan → Diagnosis Link**: Care plans link to diagnoses via `diagnosisId`, but this relationship isn't defined in the PouchDB schema. How is referential integrity maintained?
4. **Related Person as Patient**: The `RelatedPerson.patientId` suggests a related person can also be a patient. How is this bidirectional relationship managed?
5. **Medication Quantity Units**: The `quantity.unit` field is a free-text string. Are there standardized units (e.g., mg, mL, tablets)?
6. **Incident Categories**: What are the valid values for `category` and `categoryItem`? Are these configurable or hardcoded?
7. **Visit → Lab/Diagnosis Link**: Labs and diagnoses reference `visitId`, but this isn't a formal relationship in the schema. How is this enforced?
8. **User Permissions Assignment**: How are permissions assigned to users? Is there a Role entity or direct user-permission mapping?

### Todos
- [ ] Document the valid values for `Patient.type`, `Patient.sex`, `Appointment.type`, `Lab.type`, `Imaging.type`, `Visit.type`
- [ ] Map out the complete lifecycle state machines for each status enumeration
- [ ] Document the business rules for status transitions (e.g., can a medication go from "completed" back to "active"?)
- [ ] Identify which fields are required vs. optional for each entity
- [ ] Document the validation rules for each entity (see `src/patients/util/`, `src/labs/utils/`, etc.)
- [ ] Create sequence diagrams for key workflows (e.g., "Request Lab → Complete Lab → View Results")
- [ ] Document the relationship between Care Plans and Care Goals (are goals always part of a plan?)
- [ ] Analyze the `PatientHistoryRecord` model (currently a .tsx file, should be .ts)
- [ ] Document the incident categorization taxonomy
- [ ] Map permissions to user roles/personas (doctor, nurse, admin, etc.)

---

**End of Document**
