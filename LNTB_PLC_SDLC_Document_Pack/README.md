# LNTB PLC & SDLC Document Pack

**Project:** Leading Next-Gen Tomato Business (LNTB)  
**Solution:** Smart cherry-tomato agriculture using ESP32/IoT, Laravel APIs, Flutter mobile, and AI-assisted ripeness detection  
**Pack version:** 1.0  
**Baseline date:** 2026-07-11  
**Status:** Draft implementation baseline; named approvals and signatures remain pending.

## Purpose

This pack provides the controlled documents and operational registers required to manage the LNTB project through the full Project Life Cycle (PLC) and Software Development Life Cycle (SDLC): initiation, planning, requirements, design, implementation, verification, deployment, pilot acceptance, operations, maintenance, benefits evaluation, and closure.

## Controlled DOCX documents

| No. | Document ID | File | Primary purpose |
|---:|---|---|---|
| 00 | LNTB-GOV-001 | `docs/00_LNTB_Document_Control_Register_and_Governance.docx` | Document control, governance model, lifecycle gates, configuration control, and approval rules. |
| 01 | LNTB-PLC-001 | `docs/01_LNTB_Project_Charter_Business_Case_and_Feasibility.docx` | Charter, business case, feasibility, objectives, constraints, success measures, and authorization. |
| 02 | LNTB-PMP-001 | `docs/02_LNTB_Project_Management_Plan.docx` | Scope, schedule, cost, quality, resource, procurement, monitoring, and delivery controls. |
| 03 | LNTB-STA-001 | `docs/03_LNTB_Stakeholder_Communication_RACI_and_Change_Control.docx` | Stakeholders, communications, RACI, change requests, issues, decisions, and escalation. |
| 04 | LNTB-SRS-001 | `docs/04_LNTB_Software_Requirements_Specification.docx` | Functional/non-functional requirements, use cases, acceptance criteria, interfaces, and traceability. |
| 05 | LNTB-ARC-001 | `docs/05_LNTB_System_Architecture_Data_and_API_Design.docx` | System/deployment architecture, data model, API contracts, synchronization, queues, and technical decisions. |
| 06 | LNTB-HFA-001 | `docs/06_LNTB_Hardware_Firmware_and_AI_Design.docx` | Hardware, firmware state machine, safety, calibration, telemetry, camera, dataset, AI validation, and BOM corrections. |
| 07 | LNTB-UX-001 | `docs/07_LNTB_UX_Localization_and_Accessibility_Specification.docx` | Khmer-first UX, navigation, states, localization, accessibility, usability, and UI acceptance. |
| 08 | LNTB-SEC-001 | `docs/08_LNTB_Security_Privacy_Backup_and_Continuity_Plan.docx` | Security, authorization, secrets, privacy, image retention, backups, continuity, incidents, RPO, and RTO. |
| 09 | LNTB-QA-001 | `docs/09_LNTB_Quality_Assurance_Test_and_Acceptance_Plan.docx` | QA strategy, test levels, environments, calibration, AI validation, defects, UAT, and release gates. |
| 10 | LNTB-OPS-001 | `docs/10_LNTB_Deployment_Operations_Maintenance_and_Support_Plan.docx` | Release, deployment, rollback, monitoring, runbooks, support, maintenance, and service ownership. |
| 11 | LNTB-CLS-001 | `docs/11_LNTB_Training_Pilot_Evaluation_and_Project_Closure_Plan.docx` | Training, pilot execution, benefits measurement, acceptance, handover, lessons learned, and closure. |

## Management workbook

`LNTB_PLC_SDLC_Management_Workbook.xlsx` is the working control system for the project. Its first worksheet is the live dashboard. It contains 21 worksheets:

1. Dashboard
2. Lists
3. Document Register
4. Requirements
5. RTM
6. WBS Schedule
7. Budget & BOM
8. Stakeholders
9. RACI
10. Risks
11. Issues
12. Changes
13. Decisions
14. Test Cases
15. Defects
16. Quality Metrics
17. Deployment Checklist
18. Acceptance
19. Training
20. Benefits & KPI
21. Sources

Update the underlying registers rather than typing over dashboard calculations. Preserve requirement IDs, test IDs, decision IDs, and document IDs once baselined.

## Architecture diagrams

The `diagrams/` directory contains rendered PNG diagrams and editable Graphviz DOT source files for:

- system context
- deployment architecture
- irrigation command sequence
- PLC/SDLC lifecycle and gates

## Source baseline

The `source/` directory preserves the user-provided proposal and the earlier requirements, project-structure, workbook, and presentation materials for traceability. Controlled documents in `docs/` supersede conflicting planning statements after formal approval.

## Critical baseline corrections

The original proposal requires actual water use in m³ and electrical energy use in kWh. The controlled baseline therefore adds a calibrated water-flow meter and an isolated energy meter; pump runtime alone is not accepted as actual consumption.

The MVP uses backend-side AI inference, configurable irrigation thresholds managed by Laravel, a firmware safety fallback during connectivity loss, a single-farm mobile workflow with a multi-farm-capable data model, and 24-hour store-and-forward synchronization with idempotent replay.

The ripeness model is accepted by measured validation evidence and confidence/limitation reporting. The pack does not preserve an unsupported “100% accuracy” claim. The current target is at least 85% macro F1 on a representative held-out dataset, subject to approved field validation.

## Recommended execution order

1. Assign named owners and approvers in the governance and charter documents.
2. Approve the charter, scope, revised BOM, schedule, and lifecycle gates.
3. Baseline the SRS, architecture, hardware/AI, UX, and security documents.
4. Update requirements, RTM, WBS, risks, issues, changes, and decisions continuously in the workbook.
5. Build a vertical slice: simulated device → Laravel ingestion/control → Flutter dashboard.
6. Integrate calibrated hardware, firmware safety, offline synchronization, and AI inference.
7. Execute QA, security, calibration, deployment-readiness, pilot, and UAT gates.
8. Train users, measure observed benefits, complete handover, and close the project.

## Items requiring project-team confirmation

- Sponsor, product owner, project manager, technical lead, and approver names.
- Exact field-pilot site and farmer representative.
- Exact flow-meter, energy-meter, relay, pump, power-supply, enclosure, and camera models.
- Revised budget and procurement authorization.
- Irrigation thresholds, maximum runtime, sampling intervals, and calibrated sensor ranges.
- Image-capture permission, retention period, and deletion authority.
- Production hosting, domain, notification provider, support owner, and service-expiry dates.
- Final dates where the current planning baseline no longer reflects actual progress.

## Control note

Framework/package versions, cloud capabilities, hardware electrical limits, and security guidance must be revalidated against their official documentation at implementation and release time. Any material deviation from this pack requires a recorded change request and impact assessment.
