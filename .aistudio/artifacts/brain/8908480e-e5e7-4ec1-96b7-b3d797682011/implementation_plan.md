# Implementation Plan: Unified Expandable/Collapsible Left Sidebar Navigation

## Overview
Currently, several portal pages (`collection.html`, `result.html`, `worksheet.html`) rely on a header-based hamburger dropdown menu even on desktop viewports. The requirement is to eliminate the header hamburger pattern on desktop across all relevant pages and replace it with a unified, responsive left sidebar navigation bar that:
1. Sits on the left side of the screen across desktop viewports.
2. Can be expanded (showing icons + labels) or collapsed (showing icons only with tooltips).
3. Places the expand/collapse toggle button at the top of the sidebar next to the institution/portal branding.
4. Uses dedicated expand/collapse icons (e.g. `fa-chevron-left` / `fa-chevron-right` or `fa-angles-left` / `fa-angles-right`) instead of hamburger bars on desktop.
5. Remembers the user's desktop expanded/collapsed preference in `localStorage`.
6. Gracefully handles mobile viewports (<1024px) via an off-canvas slide-out drawer with a backdrop overlay.
7. Leaves `index.html` and `student.html` untouched as requested.
8. Ensures 100% English UI copy throughout all labels, buttons, and notifications.

---

## Target Pages & Scope

| Page | Current Navigation Pattern | Target Navigation Architecture |
|---|---|---|
| `admin.html` | Side drawer exists, but header has desktop hamburger (`#hamburger-btn`), footer has legacy Malayalam text | Hide header hamburger on desktop (`lg:hidden`), keep collapse toggle at sidebar top, replace Malayalam labels with English, align layout |
| `collection.html` | Header dropdown menu via hamburger button | Convert to unified left sidebar (`#side-drawer`), desktop expand/collapse with `localStorage`, mobile drawer with backdrop overlay |
| `result.html` | Header dropdown menu via hamburger button | Convert to unified left sidebar (`#side-drawer`), desktop expand/collapse with `localStorage`, mobile drawer with backdrop overlay, maintain role-based tab guards |
| `worksheet.html` | Header dropdown menu via hamburger button | Convert to unified left sidebar (`#side-drawer`), desktop expand/collapse with `localStorage`, mobile drawer with backdrop overlay, seamless full-width Excel canvas integration |
| `index.html` | Intact (No changes) | Untouched as instructed |
| `student.html` | Intact (No changes) | Untouched as instructed |

---

## Architectural Specification

### 1. Unified CSS Component (`styles/main.css`)
- **Shared Desktop Classes**:
  - Container shell: `.has-desktop-sidebar` on root app container.
  - Sidebar: `#side-drawer` (fixed to left, `top: 0`, `bottom: 0`, `z-index: 40`, default expanded `width: 16rem` / 256px, collapsed `width: 4.5rem` / 72px).
  - Main wrapper: `.main-wrapper` with smooth margin transition (`margin-left: 16rem` expanded, `margin-left: 4.5rem` collapsed).
  - Collapsed styling: Hides text labels (`.nav-text`, `.drawer-heading`, `.drawer-header-title`), centers icons, and flips the toggle button icon.
- **Mobile Off-Canvas Drawer**:
  - At `<1024px`, sidebar transforms off-canvas (`-translate-x-full`).
  - `#drawer-overlay` backdrop with smooth fade (`opacity-0` to `opacity-50`).
  - Header has mobile-only toggle button (`#mobile-menu-btn`, visible only on `<1024px`).
  - Closing drawer triggered by: clicking overlay, clicking close button (`#close-drawer-btn`), or selecting any navigation link.

### 2. State Persistence & JavaScript Pattern
- Shared storage key: `app_desktop_sidebar_collapsed` (or per-portal keys like `admin_sidebar_collapsed`, `collection_sidebar_collapsed`, etc.).
- Default on first visit: Expanded.
- Toggle action updates `localStorage` and toggles `.sidebar-collapsed` class on the parent app element.
- Auto-handles window resize: closes mobile drawer when transitioning to desktop.

### 3. Page Layout Integration
- **`admin.html` & `scripts/pages/admin-page.js`**:
  - Add `lg:hidden` to `#hamburger-btn` in header.
  - Replace Malayalam footer button labels (`ബാക്ക് to ഒഫീഷ്യൽ` -> `Back to Official`, `അഡ്മിൻ ലോഗൗട്ട്` -> `Admin Logout`).
- **`collection.html` & `scripts/pages/collection-page.js`**:
  - Replace `<header>` dropdown menu with `#side-drawer` markup + `#drawer-overlay`.
  - Adjust `.collection-shell` and `#app` layout with `.collection-main-wrapper`.
  - Update `collection-page.js` menu handler to support sidebar collapse/expand toggle on desktop and drawer on mobile.
- **`result.html` & `scripts/pages/result-page.js`**:
  - Refactor `<header>` dropdown menu into left `#side-drawer` + `#drawer-overlay`.
  - Support admin-only tabs (`[data-role-scope="admin-only"]`) and Settings subtabs inside the sidebar cleanly.
  - Update `result-page.js` to manage sidebar state and hide drawer on link click.
- **`worksheet.html` & `scripts/pages/official-shared-bootstrap.js`**:
  - Refactor `<header>` dropdown into left `#side-drawer` + `#drawer-overlay`.
  - Ensure Excel workspace canvas (`#official-tab-export`) adjusts width correctly with expanded/collapsed sidebar.
  - Update `official-shared-bootstrap.js` to handle sidebar toggle and mobile drawer.

---

## Affected Files
1. `styles/main.css`: Add and standardize responsive expandable/collapsible sidebar styles across all portals.
2. `admin.html`: Hide desktop hamburger in header; standardize collapse button and English UI copy.
3. `scripts/pages/admin-page.js`: Minor refinement for unified storage keys and English labels.
4. `collection.html`: Replace header hamburger dropdown with left sidebar structure, mobile overlay, and collapse toggle.
5. `scripts/pages/collection-page.js`: Integrate sidebar collapse/expand state handler, persistence, and mobile drawer toggle.
6. `result.html`: Replace header hamburger dropdown with left sidebar structure, mobile overlay, and collapse toggle.
7. `scripts/pages/result-page.js`: Integrate sidebar collapse/expand state handler, persistence, and mobile drawer toggle.
8. `worksheet.html`: Replace header hamburger dropdown with left sidebar structure, mobile overlay, and collapse toggle.
9. `scripts/pages/official-shared-bootstrap.js`: Integrate sidebar collapse/expand state handler and mobile drawer toggle for worksheet.

---

## Potential Risks & Mitigation

| Risk | Mitigation |
|---|---|
| Large tables (Master Ledger, Worksheet Excel grid, Mark lists) getting squeezed by sidebar | Collapsed mode (`4.5rem`) gives 95%+ viewport width. In addition, when tables load in full-width mode, sidebar smooth transitions maintain responsive fluidity. |
| Inconsistent storage behavior across different browsers | Safe `try...catch` wrapper around `localStorage` with fallback to expanded state. |
| Role-based tabs (e.g. Result Super Admin vs Staff) breaking | Preserve all `data-role-scope="admin-only"` and tab data attributes unchanged. |
| Accidental Malayalam text in UI | Strict verification that all buttons, titles, tooltips, and badges are in English only. |

---

## Verification & Testing Protocol
1. **Compilation Check**: Run `compile_applet` and `lint_applet` to verify no syntax errors or broken references.
2. **Desktop Testing (>= 1024px)**:
   - Verify sidebar is pinned on the left across `admin.html`, `collection.html`, `result.html`, `worksheet.html`.
   - Verify no hamburger button is visible in headers on desktop.
   - Verify clicking the toggle icon at the top of the sidebar expands and collapses the sidebar smoothly.
   - Verify refresh persists the collapsed/expanded state.
   - Verify all tab switches function properly in both expanded and collapsed modes.
3. **Mobile Testing (< 1024px)**:
   - Verify header shows mobile menu trigger.
   - Verify clicking menu slides in off-canvas drawer from the left with dark backdrop.
   - Verify clicking overlay, close button, or any link closes the drawer cleanly.
4. **Untouched Pages Check**:
   - Verify `index.html` and `student.html` remain completely untouched and functional.
