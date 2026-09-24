# Hands-On HL7® FHIR®: Level-by-Level Implementation

This repository documents a practical, test-driven journey through the 5 official levels of HL7® FHIR® (Fast Healthcare Interoperability Resources). Every concept is built, validated offline against an enterprise HAPI FHIR engine, and documented with reproducible API tests.

---

## 🩺 Clinical Summary (For Clinicians & Healthcare Providers)

### The Problem
When patient records travel between different hospital systems, labs, and clinics, data frequently gets misinterpreted, lost, or dumped as unreadable free-text PDFs. 

### What This Project Does
This project demonstrates how health data is broken into structured, standardized building blocks called **FHIR Resources**. 
* **Safe Language:** Ensures a clinical concept (like a diagnosis or blood test) uses an universally agreed terminology rather than local jargon that another clinic's electronic record might misunderstand.
* **Readable & Auditable:** Every data element includes a verified human-readable clinical summary alongside computer code, ensuring doctors can read it safely without technical decoding tools.
* **Extensible:** Shows how to record unique clinic-specific observations (e.g., custom questionnaires or fluency notes) without corrupting the standard patient chart.

---

## 💻 Informatician & Developer Summary (Technical Details)

### Architecture & Standards
* **Specification Target:** HL7 FHIR R4 standard.
* **Engine:** Offline HAPI FHIR JPA server running via Docker (`hapiproject/hapi:latest`) on port `8080`.
* **Testing Client:** Postman (`application/fhir+json` MIME types).
* **Storage & Operations:** Local instance testing using core schema conformance, `$validate` operations, and standard REST interactions.

### Roadmap Across the 5 FHIR Levels
1. **Level 1 (Foundation):** Base serialization (JSON/XML), data types (`CodeableConcept`, `Coding`, `Narrative`), and standard `extension` syntax.
2. **Level 2 (Implementation & Terminology):** RESTful engine bindings, Conformance, `StructureDefinition`, `CodeSystem`, `ValueSet`, and search parameters.
3. **Level 3 (Administration):** Modeling real-world actors: `Patient`, `Practitioner`, `Organization`, `Location`, and `Encounter`.
4. **Level 4 (Clinical & Workflow):** Clinical records: `Observation`, `Condition`, `MedicationRequest`, `DiagnosticReport`, and `Task`.
5. **Level 5 (Clinical Reasoning):** Rules engines, CDS Hooks, `PlanDefinition`, and clinical decision support.

---

## 📂 Current Progress: Level 1 (Foundation)

* **Artifact:** `level-1-foundation/01_datatypes_and_extensions.json`
* **Features Demonstrated:**
  * Base `DomainResource` inheritance using a `Basic` resource envelope.
  * Level 1 complex types: `Narrative` (`text.div` XHTML) ensuring baseline human legibility.
  * Terminology binding with formal `system`, `code`, and `display` tuples.
  * FHIR extension model: standard attribute augmentation via explicit `url` schema keys and strongly-typed primitive values (`valueString`).
  * Server-side `$validate` operation verification returning a compliant `OperationOutcome`.
