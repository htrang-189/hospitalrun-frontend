# Architecture Overview

**Last Updated:** 2026-04-11  
**Analyzed By:** TrangN via Kiro/bmad

---

## Related Files
- `src/App.tsx` - Application entry point and routing setup
- `src/HospitalRun.tsx` - Main application shell with navigation
- `src/index.tsx` - React DOM rendering and Redux provider
- `src/shared/store/index.ts` - Redux store configuration
- `src/shared/config/pouchdb.ts` - Database configuration and schema
- `package.json` - Dependencies and project metadata

---

## Table of Contents
1. [Overview](#overview)
2. [System Purpose](#system-purpose)
3. [Technology Stack](#technology-stack)
4. [High-Level Architecture](#high-level-architecture)
5. [Folder Structure](#folder-structure)
6. [Core Modules](#core-modules)
7. [Data Layer](#data-layer)
8. [State Management](#state-management)
9. [Routing & Navigation](#routing--navigation)
10. [Key Insights](#key-insights)
11. [Questions & Todos](#questions--todos)

---

## Overview

HospitalRun Frontend is a React-based web application designed to provide hospital management functionality for developing world hospitals. It is a free, open-source Electronic Health Record (EHR) and Hospital Information System (HIS) that operates offline-first using PouchDB for local data storage with remote synchronization capabilities.

**Version:** 2.0.0-alpha.7  
**License:** MIT  
**Repository:** https://github.com/HospitalRun/hospitalrun-frontend

---

## System Purpose

HospitalRun Frontend serves as the user interface for managing:
- **Patient Records** - Demographics, medical history, allergies, diagnoses
- **Appointments** - Scheduling and appointment management
- **Laboratory** - Lab requests and results tracking
- **Imaging** - Radiology and imaging request management
- **Medications** - Prescription and medication tracking
- **Incidents** - Hospital incident reporting and management
- **Settings** - System configuration and user preferences

The system is designed for:
- Healthcare facilities in resource-constrained environments
- Offline-first operation with eventual synchronization
- Multi-language support (i18next)
- Role-based access control
- Mobile-responsive interface

---

## Technology Stack

### Core Framework
- **React 17.0.1** - UI component library
- **TypeScript 3.8.3** - Type-safe JavaScript
- **React Router 5.3.0** - Client-side routing
- **React Scripts 3.4.0** - Create React App tooling

### State Management
- **Redux 4.1.0** - Global state container
- **Redux Toolkit 1.7.0** - Modern Redux patterns
- **React Redux 7.2.0** - React bindings for Redux
- **Redux Thunk 2.4.0** - Async action handling

### Data Layer
- **PouchDB 7.2.1** - Local NoSQL database
- **PouchDB Find** - Query plugin
- **PouchDB Authentication 1.1.3** - User authentication
- **Relational Pouch 4.0.0** - Relational data modeling
- **React Query 2.25.2** - Server state management

### UI Components
- **@hospitalrun/components 3.4.0** - Custom component library
- **React Bootstrap 1.6.0** - Bootstrap components
- **Bootstrap 5.1.0** - CSS framework

### Internationalization
- **i18next 21.6.0** - Translation framework
- **react-i18next 11.15.0** - React integration
- **i18next-browser-languagedetector 6.1.0** - Language detection

### Utilities
- **date-fns 2.28.0** - Date manipulation
- **lodash 4.17.15** - Utility functions
- **uuid 8.0.0** - Unique ID generation
- **validator 13.0.0** - Data validation

---

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Browser Layer                         │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐            │
│  │   React    │  │  Router    │  │  i18next   │            │
│  │ Components │  │  (Routes)  │  │   (i18n)   │            │
│  └────────────┘  └────────────┘  └────────────┘            │
└─────────────────────────────────────────────────────────────┘
                            │
┌─────────────────────────────────────────────────────────────┐
│                    State Management Layer                    │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐            │
│  │   Redux    │  │   React    │  │   Local    │            │
│  │   Store    │  │   Query    │  │   State    │            │
│  └────────────┘  └────────────┘  └────────────┘            │
└─────────────────────────────────────────────────────────────┘
                            │
┌─────────────────────────────────────────────────────────────┐
│                      Business Logic Layer                    │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐            │
│  │  Custom    │  │ Validation │  │   Domain   │            │
│  │   Hooks    │  │   Utils    │  │   Models   │            │
│  └────────────┘  └────────────┘  └────────────┘            │
└─────────────────────────────────────────────────────────────┘
                            │
┌─────────────────────────────────────────────────────────────┐
│                        Data Layer                            │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐            │
│  │  PouchDB   │  │ Relational │  │   Remote   │            │
│  │   Local    │  │   Pouch    │  │    Sync    │            │
│  └────────────┘  └────────────┘  └────────────┘            │
└─────────────────────────────────────────────────────────────┘
                            │
┌─────────────────────────────────────────────────────────────┐
│                    Backend API (CouchDB)                     │
└─────────────────────────────────────────────────────────────┘
```

---

## Folder Structure

```
src/
├── __mocks__/              # Test mocks for i18next and media queries
├── __tests__/              # Test files mirroring src structure
├── dashboard/              # Dashboard module (landing page)
├── imagings/               # Imaging/Radiology management module
├── incidents/              # Incident reporting module
├── labs/                   # Laboratory management module
├── medications/            # Medication management module
├── page-header/            # Page header components (breadcrumbs, toolbar, title)
├── patients/               # Patient management module (largest module)
├── scheduling/             # Appointment scheduling module
├── settings/               # Application settings module
├── shared/                 # Shared utilities, components, and configuration
│   ├── components/         # Reusable UI components
│   ├── config/             # Configuration (PouchDB, i18n)
│   ├── db/                 # Database utilities and repositories
│   ├── hooks/              # Shared custom hooks
│   ├── locales/            # Translation files
│   ├── model/              # Shared data models
│   ├── static/             # Static assets
│   ├── store/              # Redux store configuration
│   └── util/               # Utility functions
├── user/                   # User authentication and session management
├── App.tsx                 # Root application component
├── HospitalRun.tsx         # Main application shell
└── index.tsx               # Application entry point
```

---

## Core Modules

### 1. **Patients Module** (`src/patients/`)
The largest and most complex module, managing all patient-related functionality:
- Patient demographics and general information
- Allergies management
- Appointments (patient view)
- Care goals and care plans
- Diagnoses
- Patient history
- Lab results (patient view)
- Medications (patient view)
- Clinical notes
- Related persons (family, emergency contacts)
- Patient search
- Visits tracking

### 2. **Scheduling Module** (`src/scheduling/`)
Manages appointment scheduling:
- Appointment creation and editing
- Appointment search and filtering
- Calendar views

### 3. **Labs Module** (`src/labs/`)
Laboratory request and result management:
- Lab request creation
- Lab result entry
- Lab status tracking (requested, completed, canceled)
- Lab search and filtering

### 4. **Imaging Module** (`src/imagings/`)
Radiology and imaging request management:
- Imaging request creation
- Imaging result entry
- Status tracking
- Search and filtering

### 5. **Medications Module** (`src/medications/`)
Medication and prescription management:
- Medication request creation
- Medication search
- Patient medication history

### 6. **Incidents Module** (`src/incidents/`)
Hospital incident reporting and tracking:
- Incident reporting
- Incident categorization
- Status tracking (reported, resolved)
- Incident visualization and analytics

### 7. **Dashboard Module** (`src/dashboard/`)
Landing page with key metrics and quick access

### 8. **Settings Module** (`src/settings/`)
Application configuration and preferences

---

## Data Layer

### Database Architecture

**PouchDB Configuration:**
- **Local Database:** `local_hospitalrun` - Stores data locally in browser
- **Remote Database:** Configured via `REACT_APP_HOSPITALRUN_API` environment variable
- **Synchronization:** Live, bidirectional sync with retry logic
- **Adapter:** Memory adapter for testing, IndexedDB for production

### Relational Schema

The application uses `relational-pouch` to implement relationships between entities:

```javascript
Entities:
- patient (1) ──< appointments (many)
- patient (1) ──< labs (many)
- patient (1) ──< medications (many)
- patient (1) ──< imagings (many)
- appointment (many) >── patient (1)
- lab (many) >── patient (1)
- imaging (many) >── patient (1)
- medication (many) >── patient (1)
- incident (standalone)
```

**Key Relationships:**
- Patients are the central entity with one-to-many relationships
- All clinical data (appointments, labs, medications, imagings) belongs to a patient
- Incidents are standalone entities (not patient-specific)

---

## State Management

### Redux Store Structure

```typescript
RootState {
  patient: PatientState          // Single patient details
  patients: PatientsState        // Patient list and search
  user: UserState                // Current user session
  breadcrumbs: BreadcrumbsState  // Navigation breadcrumbs
  components: ComponentsState    // UI component state (sidebar)
  medication: MedicationState    // Single medication details
}
```

### State Management Strategy

1. **Redux** - Used for:
   - User session management
   - Current patient/medication being viewed
   - UI component state (sidebar collapsed state)
   - Breadcrumb navigation state

2. **React Query** - Used for:
   - Server state caching
   - Data fetching and mutations
   - Optimistic updates
   - Background refetching

3. **Local Component State** - Used for:
   - Form inputs
   - UI interactions
   - Temporary state

---

## Routing & Navigation

### Main Routes

```typescript
/ (root)                    → Dashboard
/appointments/*             → Appointments Module
/patients/*                 → Patients Module
/labs/*                     → Labs Module
/medications/*              → Medications Module
/incidents/*                → Incidents Module
/settings/*                 → Settings Module
/imaging/*                  → Imagings Module
```

### Navigation Components

1. **Navbar** - Top navigation bar with branding and user menu
2. **Sidebar** - Left sidebar with module navigation (collapsible)
3. **Breadcrumbs** - Hierarchical navigation trail
4. **ButtonToolBar** - Context-sensitive action buttons

### Session Management

- Session check on application load (`App.tsx`)
- PouchDB authentication integration
- User context stored in Redux
- Protected routes (authentication required)

---

## Key Insights

💡 **Offline-First Architecture**  
The application is designed to work offline using PouchDB with local storage. Data syncs automatically when connectivity is restored, making it ideal for resource-constrained environments.

💡 **Module-Based Organization**  
Each clinical domain (patients, labs, medications, etc.) is organized as a self-contained module with its own hooks, models, components, and utilities. This promotes maintainability and scalability.

💡 **Dual State Management**  
The application uses both Redux (for UI state and current entities) and React Query (for server state and caching). This hybrid approach optimizes for both local UI state and remote data management.

💡 **Patient-Centric Data Model**  
The relational schema centers around the Patient entity, with all clinical data (appointments, labs, medications, imagings) linked to patients. This reflects the real-world healthcare workflow.

⚠️ **Alpha Version Status**  
The application is in alpha (v2.0.0-alpha.7), indicating ongoing development and potential breaking changes. Some features may be incomplete or subject to refactoring.

⚠️ **Legacy Dependencies**  
Some dependencies are outdated (React 17, React Scripts 3.4, TypeScript 3.8), which may pose security risks and limit access to newer features.

🔗 **Internationalization Ready**  
Full i18next integration with browser language detection enables multi-language support, critical for global healthcare deployment.

🔗 **Component Library Integration**  
Uses custom `@hospitalrun/components` library for consistent UI patterns across the HospitalRun ecosystem.

---

## Questions & Todos

### Questions
1. What is the authentication flow and role-based access control implementation?
2. How are offline conflicts resolved when syncing data?
3. What is the deployment strategy for the remote CouchDB backend?
4. Are there specific compliance requirements (HIPAA, GDPR) being addressed?
5. What is the migration path from v1.0 to v2.0?

### Todos
- [ ] Document the authentication and authorization flow
- [ ] Map out the complete routing structure for each module
- [ ] Analyze the shared component library usage patterns
- [ ] Document the validation rules and business logic for each entity
- [ ] Create data flow diagrams for key user workflows
- [ ] Investigate the incident visualization and analytics features
- [ ] Document the offline sync conflict resolution strategy
- [ ] Analyze the testing strategy and coverage

---

**End of Document**
