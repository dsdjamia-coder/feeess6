# Result Management Portal Redesign & Role-Based Access Control

Comprehensive architecture and design plan to refine the Result Management Portal (`result.html` & `scripts/pages/result-page.js`), enforcing full Super Admin control, strict Staff/Teacher permission boundaries (assigned classes only), desktop & mobile responsive optimization, and an intuitive 5-tab structure.

---

## User Review & Critical Decisions

> [!IMPORTANT]
> The following architectural decisions were confirmed based on user feedback:
> - **Staff / Teacher Access Boundary**: Staff members with Result access are strictly scoped to **Marks Entry** and **Draft Review** for their **assigned classes only**. They cannot publish results, change schemas, alter grade rules, or view/edit unassigned classes.
> - **Super Admin Privileges**: Super Admins maintain unrestricted access to all classes, publishing controls, subject & exam configurations, grade rule engines, promotion tools, and snapshot backups.
> - **Navigation Architecture**: A classic header dropdown menu across both desktop and mobile viewports, featuring an animated hamburger toggle for mobile devices and clean, accessible desktop links.
> - **Tab Structure (5 Consolidated Tabs)**:
>   1. **Overview**: Key metrics, academic year status, recent exam performance, and class completion health.
>   2. **Marks Entry**: Class-filtered mark entering and saving with real-time total/pass calculations.
>   3. **Draft Review**: Preview unpublished marks, verification status, and tabular draft inspection.
>   4. **Published Results**: Official published registers, class marksheets, and report card generation.
>   5. **Settings & Setup** *(Super Admin Only)*: Unified hub containing Publish Controls, Subjects & Exams, Grade Rules, Class Promotion, and Snapshot Export.

---

## 1. Overview & Core Concept

### What It Does
The updated Result Portal delivers a secure, responsive, and role-aware examination grading and marks management experience:
- **Super Administrators** can configure examination schemas, set subject marks/pass criteria, establish grading bands, lock/unlock publication windows, manage class promotions, and export institutional snapshots.
- **Assigned Teachers/Staff** can seamlessly record student marks, verify calculations, and review drafts for their designated class(es) without exposure to administrative controls or unauthorized classes.

### Target Audience & Personas
1. **Super Admin / Principal**: Oversees all classes, sets academic timelines, approves final marks, and publishes results publicly.
2. **Class Teacher / Subject Staff**: Focuses solely on their class roster, entering scores accurately on desktop or mobile.

### Key Value
- Prevents unauthorized edits or leaks of unpublished results.
- Eliminates cognitive overload for teachers by displaying only their assigned classes.
- Delivers a unified, professional desktop and mobile experience with tabular numerical alignment and zero visual clutter.

---

## 2. User Experience & Visual Design

### Key User Flows

#### A. Super Admin Flow
1. Super Admin logs in via Admin account.
2. The header reveals all 5 tabs: **Overview**, **Marks Entry**, **Draft Review**, **Published Results**, and **Settings & Setup**.
3. Super Admin can select any class across all academic years.
4. In **Settings & Setup**, tabs or sub-sections switch between:
   - *Publish Controls* (Schedule date/time, publish/hide status, lock editing).
   - *Subjects & Exams* (Configure total/pass marks, add terminal/custom exams).
   - *Grade Rules* (Define A+, A, B, C, F percentage bands).
   - *Class Promotion* (Map class progression, preview eligible students, auto-promote).
   - *Snapshot Export* (Backup complete year results into portable formats).

#### B. Teacher / Staff Flow
1. Teacher logs in with their credentials (verified via username/email).
2. The header displays only allowed tabs: **Overview**, **Marks Entry**, and **Draft Review**.
3. The Class dropdown automatically filters to only the classes assigned to that teacher.
4. Marks entry table displays student names, admission numbers, and editable score inputs.
5. In **Draft Review**, the teacher can verify the entered marks for their assigned class and confirm readiness.

### Visual Identity & Layout Architecture
- **Design Tokens**:
  - Dominant Neutral: `#f8fafc` (slate-50 background) with clean white cards (`#ffffff`).
  - Brand Primary: Deep indigo/slate (`#1e1b4b` / `#312e81`) for institutional authority.
  - Accent Actions: Royal blue / indigo (`#4f46e5` / `#3b82f6`) for primary interactive CTAs.
  - Semantic Status:
    - Passed / Published: Emerald (`#10b981`)
    - Pending / Draft / Warning: Amber (`#f59e0b`)
    - Failed / Blocked / Error: Rose (`#f43f5e`)
- **Typography & Tabular Discipline**:
  - Fonts: Plus Jakarta Sans for titles and interface prose.
  - Tabular Numbers: All mark inputs, totals, percentages, and admission numbers use `font-mono tabular-nums`.
- **Desktop & Mobile Responsiveness**:
  - Desktop ($1200\text{px}-1440\text{px}$): High-density data grid with sticky table headers, side-by-side action controls, and ample horizontal padding.
  - Mobile ($<768\text{px}$): Responsive card containers, minimum $44\text{px}$ touch targets on buttons and select inputs, compact sticky summary bar, and a slide-down mobile menu.
- **Language**: English UI only across all forms, tables, buttons, and notifications.

---

## 3. Key Product Decisions & Trade-Offs

| Decision | Approach | Rationale |
| :--- | :--- | :--- |
| **Tab Consolidation** | Group 8 legacy tabs into 5 primary tabs, embedding administrative sub-tools into a single **Settings & Setup** hub | Dramatically simplifies navigation for daily users while keeping all admin tools intact and accessible. |
| **Staff Class Scoping** | Resolve teacher assigned classes dynamically from staff profile records (`assignedClass`, `assignedClasses`, `class`, `classes`, `customFields`) | Ensures zero configuration friction; matches existing staff records already maintained in Admin / Worksheet portals. |
| **Settings Sub-Navigation** | Segmented tab pills within **Settings & Setup** page for Admin | Keeps all setup tools in one unified location without crowding the main header menu. |
| **Real-Time Total Computation** | Client-side reactive calculation of student totals, pass/fail status, and grade on keyup/blur | Provides instantaneous feedback to teachers during marks entry without waiting for server roundtrips. |

---

## 4. Technical Architecture & Implementation Plan

### System Flow Diagram

```
┌────────────────────────────────────────────────────────┐
│                   Authentication Gate                  │
│       (Master Admin Check & Staff Permission Check)     │
└───────────────────────────┬────────────────────────────┘
                            │
            ┌───────────────┴───────────────┐
            ▼                               ▼
   [Super Admin Role]               [Staff / Teacher Role]
            │                               │
   • All 5 Tabs Visible             • Only 3 Tabs (Overview, Entry, Draft)
   • All Classes Selectable         • Strictly Assigned Classes Only
   • Full Edit & Publish Rights     • Entry & Review Privileges Only
            │                               │
            └───────────────┬───────────────┘
                            ▼
┌────────────────────────────────────────────────────────┐
│               Consolidated 5-Tab Router                │
├─────────────────┬───────────────────┬──────────────────┤
│ 1. Overview     │ 2. Marks Entry    │ 3. Draft Review  │
│ (Metrics & KPI) │ (Grid Entry)      │ (Audit & Verify) │
├─────────────────┴───────────────────┴──────────────────┤
│ 4. Published Results                │ 5. Settings      │
│ (Public Register & Print)           │ (Admin Tools)    │
└────────────────────────────────────────────────────────┘
```

### Affected Files
1. `/result.html`:
   - Update header navigation menu to reflect the 5-tab structure.
   - Restructure sections: Consolidate `settings-page`, `subjects-page`, `grades-page`, `promotion-page`, and `snapshot-page` into a unified `settings-page` container with clean sub-tab switching.
   - Enhance mobile header dropdown and layout container classes for high-density desktop and responsive touch mobile.
2. `/scripts/pages/result-page.js`:
   - Update tab activation and role scope logic to manage the 5 tabs.
   - Implement `getStaffAssignedClasses(staffDoc)` helper to extract assigned classes.
   - Scope class dropdowns in Marks Entry and Draft Review when `roleScope === 'staff'`.
   - Implement Settings sub-tab switcher for Super Admin.
   - Preserve all data persistence, calculation, and export functionalities.

---

## Verification & Testing Steps

1. **Role Access Verification**:
   - Log in as Super Admin: Ensure all 5 tabs and all classes are visible and accessible. Verify all sub-tools in Settings (Publish, Subjects, Grades, Promotion, Snapshot, Visibility) function correctly via the hamburger menu.
   - Log in as Staff/Teacher: Verify that Settings and Published Results tabs are hidden. Verify that Marks Entry and Draft Review class selectors only display their assigned class(es).
2. **Published Results Table & Smart Filters Verification**:
   - Verify Published Results displays in a structured, high-density table matching Draft Review.
   - Verify Quick Smart Filters for Published Results (All, Passed, Failed, Absent, P Awarded) dynamically update counters and filter rows.
   - Verify Sort Orders (Admission No, A-Z, Z-A, Rank/Score, Boys First, Girls First, Mixed) correctly sort published rows.
   - Verify Bulk "Move to Draft" moves results back to draft status in Firestore.
3. **Settings & Setup Navigation Reorganization**:
   - Verify Settings & Setup subtabs are removed from under the header.
   - Verify hamburger menu clearly groups Settings & Setup with all 6 subtabs: Publish Controls, Subjects & Exams, Grade Rules, Class Promotion, Snapshot Export, and Student Visibility.
   - Verify clicking any subtab directly activates `settings-page` and reveals the corresponding subpanel.
4. **Overview Page Settings Leak Bug Fix**:
   - Verify that upon initial login / applet load, Settings & Setup contents are strictly hidden while on Overview (`#dashboard-page`).
   - Switching between tabs maintains clean page-content isolation with zero unintended element leakage.
5. **Data Integrity & Build**:
   - Run `lint_applet` and `compile_applet` to verify clean build with zero syntax or import errors.
