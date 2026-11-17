# Enterprise Onboarding & Asset Lifecycle Engine (ServiceNow)

## 📌 Executive Overview
An end-to-end Joiner-Mover-Leaver (JML) automation solution built on the Now Platform (Scoped Application: `Enterprise Onboarding and Asset Engine` / `x_2317706_enterp_0`). This solution eliminates manual provisioning bottlenecks by unifying **HR Service Delivery (CIS-HR)**, **Hardware Asset Management (CIS-HAM)**, **Software Asset Management (CIS-SAM)**, and **CMDB Data Foundations (CIS-DF)** into a single zero-touch workflow.

---

## 🏗️ Solution Architecture & Workflow

```text
[Service Portal (/sp)] 
   │  Submits "New Employee Onboarding Portal Request"
   ▼
[HR Onboarding Request (x_2317706_enterp_0_hr_req)]
   │  Triggers After-Insert Business Rule
   ▼
[OnboardingAssetEngine (Script Include)]
   ├──► 1. Hardware Stockroom (x_2317706_enterp_0_hw_stock) ──► Matches Location & Reserves "In Stock" Asset
   ├──► 2. Software Entitlement (x_2317706_enterp_0_sw_entl) ──► Validates License Pool & Increments Allocated Rights
   └──► 3. Global CMDB (cmdb_ci_computer) ─────────────────────► Creates Configuration Item & Binds to Employee
```


### Offboarding Reclamation Flow
* **Trigger:** Updating `Onboarding Status` to `Offboarded` on the `HR Onboarding Request` record fires an After-Update Business Rule (`Reclaim Offboarded Assets`).
* **Automated Actions:** Automatically returns the hardware asset state to `In Stock`, clears `Assigned To`, decrements `Allocated Rights` in the software license pool, and marks the corresponding CMDB Configuration Item as `Non-Operational`.

---

## 🗄️ Data Model & Schema Design

| Table Label | Backend Table Name | Extends | Purpose & CIS Alignment |
| :--- | :--- | :--- | :--- |
| **HR Onboarding Request** | `x_2317706_enterp_0_hr_req` | `Task [task]` | Captures hire details (`employee_name`, `employee_user`, `department`, `location`, `onboarding_status`) with `HRQ` auto-numbering (**CIS-HR**). |
| **Hardware Stockroom** | `x_2317706_enterp_0_hw_stock` | Base Table | Tracks regional inventory (`Bengaluru`, `Hyderabad`), `model_name`, `serial_number`, and `asset_state` (**CIS-HAM**). |
| **Software Entitlement** | `x_2317706_enterp_0_sw_entl` | Base Table | Enforces license compliance (`total_rights` vs. `allocated_rights`) mapped by `target_department` (**CIS-SAM**). |
| **Computer CI** | `cmdb_ci_computer` | `cmdb_ci_hardware` | Maintains live operational configuration items synchronized via Cross-Scope Privileges (**CIS-DF**). |

---

## ⚙️ Key Technical Components
* **Service Portal Entry Point:** `New Employee Onboarding Portal Request` Record Producer configured with variable mapping directly to `x_2317706_enterp_0_hr_req`.
* **Server-Side Orchestration (`OnboardingAssetEngine`):** Modular Script Include handling location-based GlideRecord queries, license capacity checks, and cross-scope CMDB record creation.
* **Cross-Scope Security:** Configured explicit Cross-Scope Privilege records (`Create`, `Write`, `Read`) allowing the scoped application to safely instantiate records in global `cmdb_ci_computer`.

---

## 🧪 Verified Test Execution Results
1. **Portal Submission:** Submitted request for `Abel Tuter` (`Department: Engineering`, `Location: Bengaluru`) via `/sp`.
2. **Hardware Allocation:** Verified `MacBook Pro 16 M3` (`SN-BLR-001`) transitioned automatically from `in_stock` to `allocated` and linked to `Abel Tuter`.
3. **Software Compliance:** Verified `ServiceNow Studio` entitlement pool incremented `Allocated Rights` from `0` to `1`.
4. **CMDB Synchronization:** Verified new CI `MacBook Pro 16 M3` populated in `cmdb_ci_computer` with accurate serial number and user assignment.
