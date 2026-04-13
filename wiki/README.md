# HospitalRun v2.0.0-alpha.7 - Business Analysis Wiki

**Analyzed By:** TrangN via Kiro/bmad  
**Analysis Date:** April 2026  
**Purpose:** Evaluation for Vietnam Healthcare Market

---

## 📚 Documentation Overview

This wiki contains a comprehensive reverse engineering and business analysis of HospitalRun v2.0.0-alpha.7, an open-source Electronic Health Record (EHR) system. The analysis spans 7 detailed documents totaling approximately 100 pages of technical and business insights.

---

## 🗂️ Wiki Structure

```
wiki/
├── Home.md                              # Main landing page with executive summary
├── _Sidebar.md                          # Navigation sidebar for GitHub Wiki
├── Quick-Reference.md                   # At-a-glance summary and quick facts
├── README.md                            # This file
│
├── 00-overview/
│   ├── architecture-overview.md         # System architecture and technology stack
│   └── system-interaction-map.md        # Data flow, bottlenecks, capability assessment
│
├── 01-domain/
│   └── healthcare-glossary.md           # Ubiquitous language and entity definitions
│
├── 03-modules/
│   └── workflow-analysis.md             # Workflow glue, master data, cross-entity logic
│
└── 07-data/
    ├── patient-entity-detail.md         # Patient aggregate root analysis
    ├── visit-entity-detail.md           # Visit entity lifecycle and integration
    └── clinical-entities-overview.md    # Lab, Imaging, Medication comparison
```

---

## 🎯 Key Findings Summary

### Overall Verdict
**High Technical Potential / Low Business Maturity (30%)**

- ✅ **Strengths:** Offline-first architecture, clean entity model, modern tech stack
- ⚠️ **Limitations:** Only 30% feature parity with professional EHR systems
- 🔴 **Blockers:** No billing, no clinical decision support, no interoperability
- 📊 **Scalability:** 3 critical bottlenecks identified that will break at scale

### Quick Stats

| Metric | Value |
|--------|-------|
| Feature Parity | 30% vs. Professional EHR |
| Production Ready | ❌ No (12-18 months minimum) |
| Suitable For | Small clinics (<50 patients/day) |
| Not Suitable For | Hospitals (>100 patients/day) |
| Investment Needed | $1.6-2M USD over 2-3 years |
| Critical Bottlenecks | 3 (document size, full loads, no indexes) |
| Missing Features | 40+ (billing, CDS, interoperability, etc.) |

---

## 📖 How to Navigate

### Start Here
1. **[Home.md](Home.md)** - Executive summary and overview
2. **[Quick-Reference.md](Quick-Reference.md)** - At-a-glance facts and ratings

### For Business Analysis
1. [Home.md](Home.md) - Executive summary and recommendations
2. [system-interaction-map.md](00-overview/system-interaction-map.md) - Capability assessment and product summary
3. [Quick-Reference.md](Quick-Reference.md) - Quick facts and investment estimates

### For Technical Analysis
1. [architecture-overview.md](00-overview/architecture-overview.md) - System architecture
2. [system-interaction-map.md](00-overview/system-interaction-map.md) - Data flow and bottlenecks
3. [patient-entity-detail.md](07-data/patient-entity-detail.md) - Entity deep-dive
4. [workflow-analysis.md](03-modules/workflow-analysis.md) - Workflow patterns

### For Domain Understanding
1. [healthcare-glossary.md](01-domain/healthcare-glossary.md) - Ubiquitous language
2. [clinical-entities-overview.md](07-data/clinical-entities-overview.md) - Entity comparison
3. [visit-entity-detail.md](07-data/visit-entity-detail.md) - Visit lifecycle

---

## 🎓 Document Descriptions

### Home.md
**Purpose:** Main landing page with executive summary  
**Audience:** All stakeholders  
**Key Content:**
- Executive summary of research journey
- Quick facts table
- Strengths and limitations
- Capability assessment matrix
- Recommendations for Vietnam market
- How to use this wiki

### Quick-Reference.md
**Purpose:** At-a-glance summary for quick lookups  
**Audience:** Busy executives and decision-makers  
**Key Content:**
- System profile and ratings
- What's implemented vs. missing
- Critical bottlenecks
- Use case fit analysis
- Investment estimates
- Quick links

### 00-overview/architecture-overview.md
**Purpose:** Technical architecture and system design  
**Audience:** Technical architects, developers  
**Key Content:**
- System purpose and technology stack
- High-level architecture diagrams
- Folder structure and module organization
- Data layer and state management
- Routing and navigation patterns
- Key technical insights

### 00-overview/system-interaction-map.md
**Purpose:** End-to-end data flow and capability assessment  
**Audience:** Business analysts, product managers, architects  
**Key Content:**
- Complete sequence diagram (patient → visit → orders → results)
- Data ownership mapping table
- 3 critical scalability bottlenecks with code locations
- BA product summary comparing to professional EHR
- Capability matrix (30% feature parity)
- Investment roadmap

### 01-domain/healthcare-glossary.md
**Purpose:** Ubiquitous language and domain model  
**Audience:** All team members, new developers  
**Key Content:**
- 16 core healthcare terms defined
- Entity relationships and cardinality
- Status enumerations for all entities
- Business definitions vs. technical types
- Key domain insights

### 03-modules/workflow-analysis.md
**Purpose:** How entities connect and workflows operate  
**Audience:** Business analysts, developers  
**Key Content:**
- Lab/Imaging request workflow (5 steps)
- Visit selection mechanism
- Master data analysis (finding: none exists!)
- Cross-entity logic (finding: zero automation!)
- 7 workflow gaps identified
- Code examples and sequence flows

### 07-data/patient-entity-detail.md
**Purpose:** Deep-dive into Patient aggregate root  
**Audience:** Developers, data architects  
**Key Content:**
- Mermaid class diagram
- Complete field map (40+ fields)
- Validation rules with code examples
- 8 business rules (code generation, full name computation, etc.)
- Data transformation pipelines
- 12 key insights

### 07-data/visit-entity-detail.md
**Purpose:** Visit entity lifecycle and integration  
**Audience:** Developers, business analysts  
**Key Content:**
- Visit entity structure and relationships
- 7-state status lifecycle with transition diagram
- Validation rules
- Integration points (embedded in Patient, referenced by Lab/Imaging/Diagnosis)
- Finding: No status transition enforcement
- Finding: No visit update/delete capability

### 07-data/clinical-entities-overview.md
**Purpose:** Comparison of Lab, Imaging, Medication entities  
**Audience:** Business analysts, developers  
**Key Content:**
- Side-by-side entity comparison table
- Status values comparison (8 for Medication, 3 for Lab/Imaging)
- Result storage analysis (only Lab has result field)
- Billing integration analysis (finding: none!)
- 13 key insights

---

## 🔍 Key Insights Across Documents

### Technical Insights
- 💡 Offline-first architecture is the system's strongest feature
- 💡 Patient-centric aggregate root design is clean and well-structured
- 💡 Hybrid embedded/referenced pattern optimizes for common access patterns
- ⚠️ Document size bottleneck will break at 2,000-4,000 visits per patient
- ⚠️ No query optimization will cause browser crashes at 10,000+ records
- ⚠️ Full document loads waste bandwidth and memory

### Business Insights
- 💡 30% feature parity with professional EHR systems
- 💡 Suitable for small clinics, not for hospitals
- ⚠️ No billing/revenue cycle at all (0% implementation)
- ⚠️ No clinical decision support (0% implementation)
- ⚠️ No interoperability (0% implementation)
- ⚠️ Free-text master data creates data quality risks

### Workflow Insights
- 💡 Visit-centric ordering enforces temporal context
- 💡 Manual workflow philosophy keeps system simple
- ⚠️ Zero cross-entity automation (no automatic updates)
- ⚠️ No master data catalogs (medications, labs, imaging types)
- ⚠️ No visit filtering (can select closed visits)

---

## 📊 Analysis Metrics

| Metric | Count |
|--------|-------|
| Total Documents | 9 (including Home, Sidebar, Quick Reference) |
| Analysis Documents | 7 core documents |
| Total Pages | ~100 pages |
| Mermaid Diagrams | 5 (class, sequence, state) |
| Comparison Tables | 20+ |
| Code Examples | 30+ TypeScript snippets |
| Key Insights | 60+ with emoji indicators |
| Strategic Questions | 50+ |
| Action Items | 80+ todos |
| Entities Analyzed | 7 (Patient, Visit, Lab, Imaging, Medication, Appointment, Incident) |
| Bottlenecks Identified | 3 critical scalability issues |
| Missing Features | 40+ compared to professional EHR |

---

## 🎯 Recommendations Summary

### For Vietnam Market

**✅ Recommended Use Cases:**
- Small rural clinics (<50 patients/day)
- Mobile health units with offline requirements
- Pilot projects and proof-of-concepts
- Training environments

**❌ Not Recommended For:**
- Provincial/city hospitals (>100 patients/day)
- Facilities requiring billing integration
- Organizations needing e-prescribing
- Hospitals requiring interoperability

**💰 Investment Required:**
- Phase 1 (Production Ready): 12-18 months, $300-400K USD
- Phase 2 (Essential Features): 18-24 months, $500-600K USD
- Phase 3 (Market Ready): 24-36 months, $800K-1M USD
- **Total: 2-3 years, $1.6-2M USD**

---

## 📞 Contact Information

**Business Analyst:** TrangN  
**Analysis Tool:** Kiro/bmad  
**Analysis Period:** April 2026  
**Version Analyzed:** HospitalRun v2.0.0-alpha.7

For questions, clarifications, or additional analysis requests, please contact the business analysis team.

---

## 🔗 External Resources

- **HospitalRun GitHub:** https://github.com/HospitalRun/hospitalrun-frontend
- **HospitalRun Website:** http://hospitalrun.io/
- **Documentation:** https://hospitalrun-frontend.readthedocs.io
- **License:** MIT (Open Source)

---

## 📝 Document Conventions

### Emoji Legend
- 💡 **Key Insight** - Important finding or observation
- ⚠️ **Warning/Risk** - Potential issue or limitation
- 🔗 **Connection** - Relationship or integration point
- ✅ **Implemented** - Feature exists in current system
- ❌ **Missing** - Feature not implemented
- 🔴 **Critical** - High-priority issue or blocker
- 📊 **Data/Metric** - Quantitative information
- 🎯 **Recommendation** - Suggested action or decision

### Document Structure
All documents follow a consistent template:
1. Title and metadata (Analyzed By, Last Updated)
2. Related Files section
3. Table of Contents
4. Overview section
5. Detailed analysis sections
6. Key Insights section (with emojis)
7. Questions & Todos section

---

## 🏁 Conclusion

This wiki represents a comprehensive business and technical analysis of HospitalRun v2.0.0-alpha.7. The documentation provides:

✅ Complete understanding of system architecture and capabilities  
✅ Identification of strengths, gaps, and limitations  
✅ Assessment of scalability bottlenecks and risks  
✅ Comparison against professional EHR standards  
✅ Clear recommendations for Vietnam market deployment  
✅ Foundation for informed decision-making and planning

**Use this wiki to:**
- Evaluate HospitalRun for your organization
- Plan customization and development efforts
- Understand technical architecture and constraints
- Make informed build vs. buy decisions
- Communicate findings to stakeholders

---

**End of README**

*Start your journey at [Home.md](Home.md) →*
