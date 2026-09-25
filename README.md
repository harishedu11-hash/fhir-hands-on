# FHIR Hands-On Implementation Guide (R4)

This repository contains an end-to-end, production-grade HL7® FHIR® R4 implementation built from scratch against a local HAPI FHIR server. It models the end-to-end administrative and clinical journey of patient **Rajesh Kumar** under the care of **Dr. Harish Mohankumar** at **Apollo Multispeciality Hospital, Chennai**.

Every resource is modeled with standard medical terminologies (**LOINC**, **SNOMED CT**, **RxNorm**, and **UCUM**) and explained through two complementary lenses:
* **The Clinician Perspective**: The bedside clinical utility, patient safety safeguards, and diagnostic workflows.
* **The Informatician Perspective**: The underlying data contracts, referential integrity rules, semantic graph relationships, and interoperability standards (US Core / ABDM / ONC).

---

## Architecture & Complete Semantic Graph

The graph below illustrates how every layer—from Level 1 metadata and Level 3 administration to Level 4 clinical workflows—anchors to `Patient/1007` and `Encounter/1008`.

================================ LEVEL 3: ADMINISTRATIVE BACKBONE ================================
+-----------------------------+
|      Organization/1004      |
| (Apollo Multispeciality)    |
+-----------------------------+
▲
│ managingOrganization
+-----------------------------+
|        Location/1005        |
|  (Emergency Trauma Bay 1)   |
+-----------------------------+
▲
│ location
+-----------------------------+
|       Encounter/1008        |
| (Emergency Room Admission)  |
+-----------------------------+
│                     │
subject (patient)    │                     │  participant (practitioner)
▼                     ▼
+---------------------+       +----------------------+
|    Patient/1007     |       |  Practitioner/1006   |
|   (Rajesh Kumar)    |       | (Dr. Harish M., MD)  |
+---------------------+       +----------------------+
│
================================ LEVEL 4: CLINICAL RECORD-KEEPING ================================
┌──────────────────────────────┼──────────────────────────────┬─────────────────────────┐
▼                              ▼                              ▼                         ▼
[ Diagnostics ]             [ Problems & Eval ]             [ Medications ]           [ Care Provision ]

Specimen                  - Condition (1010)              - MedicationRequest       - CarePlan
(Venous Blood)              (Hypertension)                  (Amlodipine 5mg)        - CareTeam

BodyStructure                   ▲                              ▲                    - Goal (<130 mmHg)
(Median Cubital Vein)           │ evidence.detail              │ reasonReference    - CommunicationRequest

Observation (1009) ─────────────┘                              │                    - Communication
(BP 158/96 mmHg)                                               │

DiagnosticReport          - AllergyIntolerance            - MedicationDispense      [ Orders & Logistics ]
(Troponin Panel)            (Penicillin Hives)            - MedicationAdmin         - ServiceRequest

ImagingStudy (Chest X-ray)- FamilyMemberHistory           - MedicationStatement       (Cardio Consult)

MolecularSequence           (Father Fatal MI)             - MedicationKnowledge     - NutritionOrder (DASH)
(CYP2C9 Gene Variant)     - Procedure (12-Lead ECG)       - Immunization (Flu Shot) - VisionPrescription
- RiskAssessment (ASCVD)        - ImmunizationEvaluation  - DeviceRequest (Cuff)
- ClinicalImpression            - ImmunizationRecommend   - DeviceUseStatement
- DetectedIssue (Target Alert)                            - SupplyRequest
- SupplyDelivery


---

## Resource Catalog by Phase

| Phase | Level Focus | Key Milestones & Deliverables | Primary Standards Used |
| :--- | :--- | :--- | :--- |
| **Level 1** | Foundations & Extensions | Custom Extensions, Primitive & Complex Data Types, `Basic` | HL7 V3, W3C XML/JSON |
| **Level 2** | The FHIR Pipeline | Conformance, Audit Logging, Terminology, StructureDefinitions | LOINC, SNOMED CT |
| **Level 3** | Administrative Backbone | 2 Transaction Bundles linking Provider, Hospital, Bed, & Patient | HL7 ActCode, MCI Registry |
| **Level 4** | Clinical Record-Keeping | 33 Clinical Resources covering Diagnostics, Problems, Rx, & Orders | LOINC, SNOMED, RxNorm, UCUM |

---

## Detailed Resource Breakdown

### Level 1: Foundations & Extensibility
*Focus: Understanding FHIR serialization, structural types, and handling unmapped data.*

| Resource | Clinician Perspective | Informatician Perspective |
| :--- | :--- | :--- |
| **`Basic`** | Acts as an administrative placeholder for clinical workflows that do not yet have an official FHIR resource definition (e.g., specific hospital committee reviews). | Provides a bare resource shell requiring an extension or `code` element to supply semantic meaning. Used when extending FHIR without breaking schema validation. |

---

### Level 2: Conformance, Terminology, & Auditing
*Focus: Inspecting server capabilities, publishing value sets, and recording regulatory audit trails.*

| Resource | Clinician Perspective | Informatician Perspective |
| :--- | :--- | :--- |
| **`CapabilityStatement`** | Confirms whether the hospital EHR supports the specific operations (e.g., prescriptions, vitals search) needed during an emergency consult. | The machine-readable metadata statement (`/metadata`) defining supported FHIR versions, REST operations, search parameters, and profiles. |
| **`CodeSystem`** | Houses the hospital's local medical dictionaries, such as Apollo-specific triage urgency definitions (Urgent Level 2). | Declares a managed namespace of codes, definitions, and hierarchical relationships with a globally unique canonical URL. |
| **`ValueSet`** | Selects the permitted dropdown list of triage codes that Dr. Harish is allowed to choose from in the emergency admitting screen. | A bounded subset of concepts drawn from one or more `CodeSystem` resources, bound to resource elements for schema validation. |
| **`StructureDefinition`** | Enforces hospital clinical policy (e.g., requiring an Indian Medical Council ID for every doctor in the system). | The computable schema definition defining cardinalities, element constraints, type slicing, and custom extension points. |
| **`AuditEvent`** | Guarantees an unalterable medico-legal log proving exactly who viewed Rajesh's emergency chart and when. | Captures regulatory compliance events (ATNA/HIPAA/DISHA) detailing who (`agent`), what (`entity`), and the action taken (`C`, `R`, `U`, `D`). |

---

### Level 3: The Administrative Backbone
*Focus: Establishing the legal entities, healthcare providers, physical locations, and encounters.*

+---------------------+     managingOrganization     +---------------------+
|    Location/1005    | ───────────────────────────► |  Organization/1004  |
| (ER Trauma Bay 1)   |                              | (Apollo Greams Rd)  |
+---------------------+                              +---------------------+
▲                                                    ▲
│ location                                           │ serviceProvider
+──────────────────────────────────────────────────────────────────────────+
|                              Encounter/1008                              |
|                         (Emergency Admission)                            |
+──────────────────────────────────────────────────────────────────────────+
│ subject                                              │ participant
▼                                                      ▼
+---------------------+                              +---------------------+
|    Patient/1007     | ◄─────────────────────────── |  Practitioner/1006  |
|   (Rajesh Kumar)    |     generalPractitioner      | (Dr. Harish Mohank) |
+---------------------+                              +---------------------+


| Resource | Clinician Perspective | Informatician Perspective |
| :--- | :--- | :--- |
| **`Organization`** | Represents Apollo Multispeciality Hospital, establishing the overarching institutional setting for legal and clinical care. | The top-level administrative boundary for multi-tenant data segmentation, billing claims attribution, and managing entity references. |
| **`Location`** | Identifies Emergency Trauma Bay 1 (Bed Level), ensuring staff know the exact physical bed where Rajesh is receiving resuscitation. | Captures physical, spatial, and organizational areas. Uses `mode: instance` and `physicalType: bd` to support bed management engines. |
| **`Practitioner`** | Identifies Dr. Harish Mohankumar, capturing his licensed role as the attending emergency clinician. | Stores provider demographics and state medical council licenses (`TN-MCI-2024-8849`) used across signatures and provenance logs. |
| **`Patient`** | The central chart for Rajesh Kumar, tracking demographics, emergency contacts, and language preferences. | The primary clinical entity in FHIR. Demarcates the root of the **Patient Compartment** (`/Patient/{id}/*`), enabling partitioned clinical queries. |
| **`Encounter`** | Documents Rajesh's emergency room admission episode, tracking arrival time, triage status, and care transitions. | The chronological and administrative junction connecting `Patient`, `Practitioner`, `Location`, and `Organization`. Clinical resources link here for encounter-based billing bundling. |

---

### Level 4: Clinical Record-Keeping (33 Resources)
*Focus: Modeling the diagnostic, therapeutic, surgical, and logistical clinical trajectory.*

#### Domain 1: Diagnostics, Laboratories, & Imaging

+---------------------+          location          +--------------------+
|  BodyStructure/xxx  | ◄───────────────────────── |    Specimen/xxx    |
| (Cubital Fossa)     |                            |   (Venous Blood)   |
+---------------------+                            +--------------------+
▲
│ specimen
+--------------------+
|  Observation/1009  |
| (Troponin: 0.02)   |
+--------------------+
▲
│ result
+---------------------+                             +--------------------+
|  ImagingStudy/xxx   |                             | DiagnosticReport/xx|
| (Chest X-Ray DX)    |                             | (Cardiac Panel)    |
+---------------------+                             +--------------------+


| Resource | Clinician Perspective | Informatician Perspective |
| :--- | :--- | :--- |
| **`Observation`** | Captures discrete quantitative findings such as Rajesh's blood pressure panel (158/96 mmHg) or cardiac troponin levels. | Captures point-in-time or interval measurements bound to **LOINC** and **UCUM**. Uses `.component` arrays to keep multi-part readings semantically coupled. |
| **`DiagnosticReport`** | The structured lab report signed out by the clinical pathologist summarizing all cardiac injury biomarkers. | The parent reporting envelope grouping one or more `Observation` resources, structured narratives, and binary PDF lab attachments. |
| **`Specimen`** | Documents the physical tube of venous blood drawn from Rajesh's arm at 10:30 AM. | Tracks biological sample metadata: source collection site, container additives, fasting status, and specimen processing chains. |
| **`BodyStructure`** | Pinpoints the left median cubital vein as the exact anatomical landmark used for venipuncture. | Models anatomical locations, structural morphologies, or tumor boundaries when terminology alone does not capture spatial specificity. |
| **`ImagingStudy`** | Documents the 2-view chest radiograph taken to rule out pulmonary edema or cardiomegaly. | Bridges the EHR and hospital PACS. Stores DICOM series instance UIDs, modalities (`DX`), and image frame counts without duplicating image binaries in FHIR. |
| **`MolecularSequence`** | Evaluates Rajesh's DNA sequencing data to identify cardiovascular drug metabolism gene variants. | Represents raw and annotated linear sequences (DNA/RNA/amino acids) along with variant coordinates for precision medicine workflows. |

---

#### Domain 2: Problems, Allergies, History & Clinical Assessments

+--------------------+        evidence.detail        +--------------------+
|   Condition/1010   | ────────────────────────────► |  Observation/1009  |
| (Hypertension)     |                               | (158/96 mmHg BP)   |
+--------------------+                               +--------------------+
│
│ basis
▼
+--------------------+        predicts outcome       +--------------------+
| RiskAssessment/xxx | ────────────────────────────► | Outcome: CHF 14.8% |
| (ASCVD 10-Yr Risk) |                               | (Moderate Risk)    |
+--------------------+                               +--------------------+
▲
│ item
+--------------------+        flags safety risk      +--------------------+
|ClinicalImpression/x| ◄──────────────────────────── | DetectedIssue/xxx  |
| (ER Medical Synth) |                               | (BP Target Alert)  |
+--------------------+                               +--------------------+


| Resource | Clinician Perspective | Informatician Perspective |
| :--- | :--- | :--- |
| **`Condition`** | Records Dr. Harish's formal diagnosis of acute essential hypertension, linked directly to the high BP reading that proved it. | Tracks clinical problems across two required axes: `clinicalStatus` (`active`) and `verificationStatus` (`confirmed`). Uses `evidence.detail` to establish diagnostic provenance. |
| **`Procedure`** | Logs the 12-lead Electrocardiogram (ECG) performed in Trauma Bay 1 to monitor cardiac rhythm. | Documents completed, in-progress, or aborted interventional or diagnostic actions performed on the patient, capturing performers and timing. |
| **`AllergyIntolerance`** | Flags Rajesh's severe, life-threatening allergic hives reaction to Penicillin, preventing inadvertent antibiotic prescribing. | Patient safety resource with `criticality: high`. Directly interrogated by EHR order-entry rules engines to trigger prescribing contraindication alerts. |
| **`FamilyMemberHistory`** | Records that Rajesh's biological father suffered a fatal myocardial infarction at age 52, identifying hereditary risk. | Models family medical pedigree with age-of-onset quantities (`onsetAge: 52 yr`), providing computational inputs for risk algorithms. |
| **`ClinicalImpression`** | The comprehensive diagnostic synthesis completed by Dr. Harish in the ER, evaluating vitals, ECG, and labs. | Bridges discrete observations and future care planning by capturing physician assessment narratives and investigation sets. |
| **`RiskAssessment`** | Documents the ASCVD 10-year cardiovascular risk calculator result showing a 14.8% probability of a cardiovascular event. | Stores predictive algorithms, mathematical probabilities (`probabilityDecimal: 0.148`), qualitative bands, and the underlying basis resources. |
| **`DetectedIssue`** | Warns the medical team that Rajesh's cardiovascular risk profile requires aggressive blood pressure lowering ($<130/80\text{ mmHg}$). | Models clinical decision support (CDS) alerts, contraindications, and therapy duplications, tracking provider acknowledgments and overrides. |

---

#### Domain 3: Medications & Immunizations (The Pharmacy Lifecycle)

[ MedicationKnowledge ] ── defines pharmacology ──► [ MedicationRequest/1011 ]
(Amlodipine 5mg Data)                               (Order: Amlodipine 5mg QD)
│
│ authorizingPrescription
▼
[ MedicationAdministration ] ◄── validates dose ──── [ MedicationDispense ]
(Nurse confirms dose given)                         (Pharmacy dispenses 14 tabs)

[ MedicationStatement ] ─── reports home OTC ──────► (Fish Oil 1000mg Daily)

[ Immunization ] ────────── evaluated by ──────────► [ ImmunizationEvaluation ]
(Flu Shot Administered)                             (Dose confirmed valid)
│
│ triggers forecast
▼
[ ImmunizationRecommendation ]
(Pneumococcal PCV13 Due)


| Resource | Clinician Perspective | Informatician Perspective |
| :--- | :--- | :--- |
| **`MedicationRequest`** | Dr. Harish's signed order for Amlodipine 5 mg orally once daily to treat hypertension. | Authoritative clinical order with `intent: order`, structured UCUM dosing instructions, and a `reasonReference` link pointing back to `Condition/1010`. |
| **`MedicationDispense`** | Apollo Inpatient Pharmacy dispenses 14 tablets of Amlodipine 5 mg in a labeled unit-dose blister pack. | Captures the supply event from a pharmacy. Points to `authorizingPrescription`, tracking quantity, lot numbers, and dispensing timestamps. |
| **`MedicationAdministration`**| The bedside nurse scans the drug blister and verifies Rajesh swallowed his first 5 mg dose at 12:00 PM. | The legal record of drug delivery. Vital for closed-loop Barcode Medication Administration (BCMA) workflows to prevent missed or duplicate doses. |
| **`MedicationStatement`** | Captures Rajesh's home medication history during intake reconciliation: Omega-3 Fish Oil 1000 mg daily. | Patient-reported or secondary-source drug history. Does not imply an active hospital order or pharmacy dispense. |
| **`MedicationKnowledge`** | The drug reference monograph for Amlodipine detailing active ingredients, brand synonyms (Norvasc), and tablet dose forms. | Non-patient-specific catalog resource providing reference pharmacology, regulatory schedules, and drug-interaction knowledge bases. |
| **`Immunization`** | Records the administration of Rajesh's seasonal influenza vaccine in his left deltoid muscle. | Captures vaccine administration events using standard **CVX** codes, manufacturer lot numbers, expiration dates, and anatomical injection sites. |
| **`ImmunizationEvaluation`** | Validates that the flu shot was administered in accordance with adult immunization interval guidelines. | The output of public health immunization registries (IIS) determining whether an administered dose is clinically "valid" or given too early. |
| **`ImmunizationRecommendation`**| Clinical reminder alerting the team that Rajesh is due for a Pneumococcal vaccine based on his clinical profile. | The predictive output of immunization forecasting engines, recommending specific CVX vaccines and targeted due dates. |

---

#### Domain 4: Care Provision, Planning & Clinical Communications

                           +--------------------+
                           |    CarePlan/xxx    |
                           | (HTN Outpatient)   |
                           +--------------------+
                              │       │        │
                 addresses    │       │        │   activity
 ┌────────────────────────────┘       │        └──────────────────────────┐
 ▼                                    ▼                                   ▼
+--------------------+          +--------------------+             +--------------------+
|   Condition/1010   |          |      Goal/xxx      |             | CommunicationReq/x |
| (Hypertension)     |          | (BP < 130 mmHg)    |             | (SMS Daily BP Log) |
+--------------------+          +--------------------+             +--------------------+
▲                                   │
│ measured by                       │ triggers delivery
│                                   ▼
+--------------------+             +--------------------+
|  Observation/xxx   |             |  Communication/xxx |
| (Home BP Logging)  |             | (Sodium Diet Sent) |
+--------------------+             +--------------------+


| Resource | Clinician Perspective | Informatician Perspective |
| :--- | :--- | :--- |
| **`CareTeam`** | Assembles the clinical team caring for Rajesh: Dr. Harish (cardiologist) and the outpatient care coordinator. | Manages the dynamic group of internal and external practitioners, family caregivers, and organizations collaborating on a patient's care. |
| **`Goal`** | Sets an explicit therapeutic target: lower systolic blood pressure to $< 130\text{ mmHg}$ by December 2026. | Encapsulates computable targets with mathematical comparators (`<`), standard metric codes (`LOINC 8480-6`), and due dates for progress tracking. |
| **`CarePlan`** | The comprehensive plan coordinating medications, lifestyle modifications, and daily home blood pressure logging. | The overarching container grouping the diagnosis (`addresses`), outcome targets (`goal`), participants (`careTeam`), and planned actions (`activity`). |
| **`CommunicationRequest`** | Automated reminder order to send Rajesh daily morning SMS prompts to log his blood pressure. | An unfulfilled order for information exchange. Used by CRM and patient-engagement engines to trigger notifications across SMS, email, or app pushes. |
| **`Communication`** | Verifies that educational materials regarding the DASH low-sodium diet were delivered to Rajesh's mobile device. | An immutable record of an information exchange event detailing sender, recipient, timestamp, and payload content. |

---

#### Domain 5: Clinical Orders, Nutrition, Optical, Medical Devices & Logistics

                 +---------------------------------------+
                 |             Patient/1007              |
                 +---------------------------------------+
                    │                 │                 │
  prescribes device │                 │ clinical consult│ dietary management
                    ▼                 ▼                 ▼
         +--------------------+ +--------------------+ +--------------------+
         |  DeviceRequest/xxx | | ServiceRequest/xxx | | NutritionOrder/xxx |
         | (Home BP Monitor)  | |  (Cardio Consult)  | | (DASH Diet 2g Na)  |
         +--------------------+ +--------------------+ +--------------------+
                    │
                    │ fulfillment
                    ▼
         +--------------------+ +--------------------+ +--------------------+
         |DeviceUseStatement/x| |VisionPrescription/x| |  SupplyDelivery/xx |
         | (Wearing Glasses)  | | (Reading Glasses)  | | (IV Start Kit bay) |
         +--------------------+ +--------------------+ +--------------------+

| Resource | Clinician Perspective | Informatician Perspective |
| :--- | :--- | :--- |
| **`ServiceRequest`** | Orders an outpatient cardiology consultation with an Apollo subspecialist for advanced workup. | Requisition resource handling all non-drug clinical orders: lab tests, consults, imaging, physical therapy, and surgical procedures. |
| **`NutritionOrder`** | Dietitian's order restricting Rajesh's hospital and home diet to the DASH protocol with $\le 2\text{ g}$ of sodium daily. | Governs inpatient meal tray assembly and enteral nutrition. Drives kitchen management systems with strict nutrient limits and texture modifiers. |
| **`VisionPrescription`** | Prescribes reading glasses for Rajesh (+1.25 Right, +1.50 Left, with +1.0 Add) to correct mild presbyopia. | Stores optical refraction parameters (sphere, cylinder, axis, prism, add) used by optical labs to grind and dispense corrective lenses. |
| **`DeviceRequest`** | Prescribes an automated, digital upper-arm blood pressure monitor for home monitoring. | An order for durable medical equipment (DME) or implantable hardware sent to medical device suppliers or insurance prior-authorization systems. |
| **`DeviceUseStatement`** | Records that Rajesh has worn reading glasses for the past two years, ensuring his sensory needs are supported. | Catalogs medical devices the patient is currently or historically using. Essential for checking MRI safety contraindications (e.g., pacemakers). |
| **`SupplyRequest`** | Emergency nurse places an internal order to restock Trauma Bay 1 with sterile IV catheter insertion kits. | Logistics requisition resource tracking inventory orders from clinical wards to central supply departments. |
| **`SupplyDelivery`** | Apollo Central Supply confirms delivery of 2 sterile IV insertion kits directly to Trauma Bay 1. | Fulfillment record documenting physical delivery of supplies, lot numbers, package quantities, and destination context. |

---

## Verifying the Implementation

### Querying the Patient Compartment
Because every clinical record is linked back to Rajesh Kumar (`Patient/1007`), the entire medical chart can be pulled in a single request:

```bash
# Retrieve a summary of all resources associated with Rajesh Kumar
curl -s -H "Accept: application/fhir+json" \
  "http://localhost:8080/fhir/Patient/1007/*" | grep -o '"resourceType":"[^"]*"' | sort | uniq -c
Traversing the Medication Chain
Trace the clinical justification for Rajesh's prescription using chained searching:

Bash
# Query the medication order, including the encounter and reason condition
curl -s -H "Accept: application/fhir+json" \
  "http://localhost:8080/fhir/MedicationRequest?subject=Patient/1007&_include=MedicationRequest:encounter&_include=MedicationRequest:reasonReference"
Repository Directory Structure
fhir-hands-on/
├── README.md
├── level-1-foundations/
│   ├── 01_patient_extension.json
│   └── 02_basic_resource.json
├── level-2-pipeline/
│   ├── 01_capability_statement.json
│   ├── 02_terminology_codesystem.json
│   ├── 03_terminology_valueset.json
│   ├── 04_audit_event.json
│   └── 05_structure_definition.json
├── level-3-administration/
│   ├── 01_organization_and_location.json
│   ├── 02_practitioner_and_patient.json
│   └── 03_encounter.json
└── level-4-clinical/
    ├── 01_blood_pressure_observation.json
    ├── 02_hypertension_condition.json
    ├── 03_amlodipine_medication_request.json
    ├── 04_diagnostics_domain.json
    ├── 05_problems_and_assessments_domain.json
    ├── 06_medication_and_immunization_domain.json
    ├── 07_care_provision_domain.json
    └── 08_orders_and_devices_domain.json
