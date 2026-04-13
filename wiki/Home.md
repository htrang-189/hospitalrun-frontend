# HospitalRun v2.0.0-alpha.7 - Business Analysis & Technical Evaluation

**Analyzed By:** TrangN via Kiro/bmad  
**Analysis Period:** April 2026  
**Purpose:** Evaluation for Vietnam Healthcare Market

---

## 📖 Executive Summary

This wiki documents a comprehensive reverse engineering and business analysis of **HospitalRun v2.0.0-alpha.7**, an open-source Electronic Health Record (EHR) and Hospital Information System (HIS). The analysis was conducted to evaluate the system's suitability for deployment in the Vietnam healthcare market, with particular focus on technical architecture, business capabilities, scalability, and feature completeness.

### Research Journey

Our analysis followed a structured approach:

1. **Architecture Discovery** - Mapped the system's technical foundation, technology stack, and high-level design patterns
2. **Domain Modeling** - Extracted the ubiquitous language and core healthcare entities
3. **Entity Deep-Dive** - Analyzed Patient, Visit, Lab, Imaging, and Medication entities in detail
4. **Workflow Analysis** - Examined the "glue" connecting entities and identified automation gaps
5. **System Interaction Mapping** - Created end-to-end sequence diagrams and data flow maps
6. **Capability Assessment** - Compared against professional EHR standards (Epic, Cerner)

### Value to Product Strategy

This documentation provides:

✅ **Technical Due Diligence** - Complete understanding of architecture, data models, and scalability constraints  
✅ **Gap Analysis** - Identification of 40+ missing features compared to professional EHR systems  
✅ **Risk Assessment** - Documentation of 3 critical scalability bottlenecks that will break at scale  
✅ **Market Fit Evaluation** - Clear verdict on suitability for Vietnam market deployment  
✅ **Implementation Roadmap** - Foundation for feature planning and development prioritization  
✅ **Knowledge Transfer** - Comprehensive documentation for development teams and stakeholders

---

## 🎯 Quick Facts

| Aspect | Details |
|--------|---------|
| **System Name** | HospitalRun |
| **Version Analyzed** | v2.0.0-alpha.7 |
| **Core Architecture** | Offline-first (PouchDB/CouchDB) |
| **Technology Stack** | React 17, TypeScript 3.8, Redux Toolkit, PouchDB 7.2 |
| **License** | MIT (Open Source) |
| **Primary Use Case** | Resource-constrained healthcare facilities |
| **Target Market** | Developing world hospitals, small clinics |
| **Development Stage** | Alpha (not production-ready) |
| **Primary Research Goal** | Evaluation for Vietnam Healthcare Market |
| **Analysis Scope** | 7 comprehensive documents, 100+ pages |
| **Key BA Verdict** | **High Tech Potential / Low Business Maturity (30%)** |

---

## 💡 Strengths

### Technical Strengths

🌟 **Offline-First Architecture**  
PouchDB enables full offline operation with automatic sync when connectivity returns. This is HospitalRun's strongest feature and differentiator.

🌟 **Clean Entity Model**  
Patient-centric aggregate root design with clear separation between embedded (visits, allergies) and referenced (labs, imaging) entities.

🌟 **Modern Tech Stack**  
React, TypeScript, and Redux provide a solid foundation for UI development and state management.

🌟 **Multi-Language Support**  
Built-in i18next integration supports internationalization out of the box.

🌟 **Open Source Flexibility**  
MIT license allows unlimited customization and extension without licensing costs.

### Functional Strengths

✅ Patient registration and demographics management  
✅ Visit tracking with status workflow  
✅ Lab, imaging, and medication order management  
✅ Allergy and diagnosis tracking  
✅ Care plan and care goal management  
✅ Appointment scheduling  
✅ Incident reporting and tracking  
✅ Role-based permissions system

---

## ⚠️ Gaps & Limitations

### Critical Missing Features

❌ **No Billing/Revenue Cycle** - Zero billing integration, no charge capture, no payment tracking  
❌ **No Clinical Decision Support** - No drug interactions, allergy checking, or duplicate order detection  
❌ **No Master Data Management** - No medication formulary, lab catalog, or imaging protocols  
❌ **No Interoperability** - No HL7, FHIR, or external system interfaces  
❌ **No Workflow Automation** - All status changes are manual, no automatic notifications  
❌ **No Structured Results** - Lab results are free-text, no trending or analytics  
❌ **No E-Prescribing** - Cannot send prescriptions electronically to pharmacies  
❌ **No Reporting/Analytics** - Minimal reporting capabilities, no business intelligence

### Scalability Bottlenecks

🔴 **Bottleneck #1: Document Size Limit**  
Visits embedded in Patient document will break at 2,000-4,000 visits per patient.

🔴 **Bottleneck #2: Full Document Loads**  
Must load entire patient document (100+ KB) just to display visit dropdown.

🔴 **Bottleneck #3: No Query Optimization**  
Client-side filtering will cause browser crashes with 10,000+ records.

### Data Quality Risks

⚠️ **Free-Text Master Data** - Medication names, lab types, imaging types are all free-text (no standardization)  
⚠️ **No Referential Integrity** - Orphaned references possible, no cascade delete  
⚠️ **No Cross-Entity Validation** - Can associate orders with closed visits, no consistency checks  
⚠️ **Client-Side Validation Only** - No server-side validation, data quality depends on UI

---

## 📊 Capability Assessment

### Feature Parity: 30% of Professional EHR

| Category | HospitalRun | Professional EHR | Gap |
|----------|-------------|------------------|-----|
| Patient Demographics | 80% | 100% | Missing insurance, guarantor |
| Clinical Orders | 40% | 100% | Missing CPOE, order sets, decision support |
| Results Management | 30% | 100% | Missing structured results, trending |
| Clinical Documentation | 20% | 100% | Missing templates, voice dictation |
| Medication Management | 30% | 100% | Missing eRx, formulary, interactions |
| Billing | 0% | 100% | **Entire revenue cycle missing** |
| Reporting | 10% | 100% | Missing analytics, quality measures |
| Interoperability | 0% | 90% | **All interfaces missing** |
| Clinical Decision Support | 0% | 80% | **All CDS features missing** |
| Workflow Automation | 10% | 90% | Most automation missing |

**Overall Score: 30% feature parity**

---

## 🎯 Recommendations

### For Vietnam Market Deployment

#### ✅ Suitable For:
- **Small clinics** (<50 patients/day)
- **Rural health posts** with unreliable internet
- **Mobile health units** requiring offline operation
- **Pilot projects** and proof-of-concepts
- **Training environments** for healthcare IT education

#### ❌ Not Suitable For:
- **Provincial/city hospitals** (>100 patients/day)
- **Facilities requiring billing integration** with insurance
- **Organizations needing e-prescribing** or pharmacy integration
- **Hospitals requiring interoperability** with other systems
- **Facilities with regulatory compliance** requirements

### Development Roadmap

**Phase 1: Production Readiness (12-18 months)**
1. Fix scalability bottlenecks (separate visits collection)
2. Add server-side validation and referential integrity
3. Implement proper database indexing and pagination
4. Add audit logging and security hardening
5. Conduct security audit and penetration testing

**Phase 2: Essential Features (18-24 months)**
1. Add medication formulary and lab test catalog
2. Implement basic clinical decision support (allergy checking)
3. Add structured lab results with reference ranges
4. Implement workflow automation (notifications, status updates)
5. Add basic reporting and analytics

**Phase 3: Market Readiness (24-36 months)**
1. Add billing and charge capture
2. Implement HL7/FHIR interfaces
3. Add e-prescribing capability
4. Implement advanced clinical documentation
5. Add patient portal

**Estimated Investment:**
- Phase 1: 3-4 full-time developers
- Phase 2: 5-6 full-time developers
- Phase 3: 8-10 full-time developers
- Total: 2-3 years, $1-2M USD

### Alternative Strategies

**Strategy A: Use As-Is for Niche Market**
- Deploy only in small rural clinics
- Accept 30% feature parity
- Focus on offline capability as differentiator
- Risk: Limited market size, no revenue cycle

**Strategy B: Fork and Customize**
- Fork the open-source project
- Add Vietnam-specific features (billing, insurance)
- Maintain separate codebase
- Risk: Ongoing maintenance burden

**Strategy C: Hybrid Approach**
- Use HospitalRun for clinical documentation
- Integrate with separate billing system
- Add custom interfaces for local requirements
- Risk: Integration complexity

**Strategy D: Build on Different Platform**
- Use HospitalRun as reference architecture
- Build new system with modern stack
- Incorporate lessons learned
- Risk: Higher initial investment

---

## 📚 Documentation Structure

This wiki is organized into the following sections:

### 00-Overview
- **[Architecture Overview](00-overview/architecture-overview)** - System architecture, technology stack, and high-level design
- **[System Interaction Map](00-overview/system-interaction-map)** - End-to-end data flow, bottleneck analysis, and capability assessment

### 01-Domain
- **[Healthcare Glossary](01-domain/healthcare-glossary)** - Ubiquitous language, entity definitions, and status enumerations

### 03-Modules
- **[Workflow Analysis](03-modules/workflow-analysis)** - How entities connect, master data analysis, and cross-entity logic

### 07-Data
- **[Patient Entity Detail](07-data/patient-entity-detail)** - Complete analysis of Patient aggregate root
- **[Visit Entity Detail](07-data/visit-entity-detail)** - Visit entity lifecycle and integration points
- **[Clinical Entities Overview](07-data/clinical-entities-overview)** - Medication, Lab, and Imaging comparison

---

## 🔍 How to Use This Wiki

### For Business Analysts
1. Start with [Quick Facts](#quick-facts) and [Capability Assessment](#capability-assessment)
2. Review [Gaps & Limitations](#gaps--limitations) for feature gaps
3. Read [Recommendations](#recommendations) for market fit analysis
4. Use [BA Product Summary](00-overview/system-interaction-map#ba-product-summary) for stakeholder presentations

### For Product Managers
1. Review [Strengths](#strengths) and [Gaps & Limitations](#gaps--limitations)
2. Study [Development Roadmap](#development-roadmap) for planning
3. Analyze [Capability Assessment](#capability-assessment) for competitive positioning
4. Use [Alternative Strategies](#alternative-strategies) for decision-making

### For Technical Architects
1. Start with [Architecture Overview](00-overview/architecture-overview)
2. Review [Scalability Bottlenecks](00-overview/system-interaction-map#logical-bottleneck-analysis)
3. Study [Data Ownership Mapping](00-overview/system-interaction-map#data-ownership-mapping)
4. Read entity detail documents for implementation patterns

### For Developers
1. Review [Healthcare Glossary](01-domain/healthcare-glossary) for domain understanding
2. Study [Workflow Analysis](03-modules/workflow-analysis) for business logic
3. Read entity detail documents for data models
4. Use [System Interaction Map](00-overview/system-interaction-map) for sequence flows

### For Stakeholders
1. Read [Executive Summary](#executive-summary)
2. Review [Quick Facts](#quick-facts)
3. Study [Recommendations](#recommendations)
4. Use [Capability Assessment](#capability-assessment) for decision support

---

## 📞 Contact & Feedback

**Business Analyst:** TrangN  
**Analysis Tool:** Kiro/bmad  
**Analysis Date:** April 2026  
**Version:** HospitalRun v2.0.0-alpha.7

For questions, clarifications, or additional analysis requests, please contact the business analysis team.

---

## 📄 Document Metadata

| Metadata | Value |
|----------|-------|
| **Total Documents** | 7 comprehensive analysis documents |
| **Total Pages** | ~100 pages of detailed analysis |
| **Analysis Depth** | Entity-level, workflow-level, system-level |
| **Diagrams** | 5 Mermaid diagrams (class, sequence, state) |
| **Tables** | 20+ comparison and mapping tables |
| **Code Samples** | 30+ TypeScript code examples |
| **Insights** | 60+ key insights with emojis |
| **Questions** | 50+ strategic questions for stakeholders |
| **Todos** | 80+ action items for implementation |

---

## 🏁 Conclusion

HospitalRun v2.0.0-alpha.7 represents a **promising but immature** healthcare information system. Its offline-first architecture and open-source nature provide significant value for resource-constrained environments, but the system lacks critical features required for mainstream healthcare deployment.

**Key Verdict:**
- ✅ **High Technical Potential** - Solid architecture, modern stack, good foundation
- ⚠️ **Low Business Maturity** - Only 30% feature parity with professional EHR
- 🔴 **Not Production-Ready** - Alpha stage, scalability issues, missing critical features

**For Vietnam Market:**
- **Suitable** for small rural clinics and pilot projects
- **Not suitable** for provincial/city hospitals or facilities requiring billing
- **Requires 2-3 years** of development to reach market readiness
- **Alternative strategies** should be considered for faster deployment

This analysis provides the foundation for informed decision-making regarding HospitalRun adoption, customization, or alternative approaches for the Vietnam healthcare market.

---

**End of Home Page**

*Navigate using the sidebar to explore detailed analysis documents →*
