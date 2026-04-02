---
name: mockup-guidelines
description: >-
  HTML mockup creation and modification for Platform healthcare SaaS modules (ValueIQ, RebateIQ, AccessIQ).
  USE FOR: create mockup, design mockup, build mockup, HTML prototype, new screen, modify mockup, update mockup, screen mockup, UI mockup, feature mockup, dashboard mockup, manufacturer view, payer view, persona mockup, AccessIQ screen, ValueIQ screen, RebateIQ screen, interactive prototype, mockup file, self-contained HTML, inline CSS, inline JS, multi-persona design, persona-based UI, healthcare mockup, pharma mockup.
  DO NOT USE FOR: backend development, API work, database migrations, production UI implementation (use FastAPI modules instead).
---

# Mockup Guidelines — Platform

> **Audience:** AI coding agents creating or modifying HTML mockups for Platform modules.
> Read this file in full before starting any mockup work.

---

## 0. Before You Start — Clarifying Questions

**Always ask clarifying questions before proceeding with mockup work.** Ambiguity costs more than a quick question. At minimum, confirm:

1. **Which module / product area?** (e.g., AccessIQ, ValueIQ, RebateIQ)
2. **Which persona(s)?** Manufacturer, Payer, Admin, or shared dual-view?
3. **New screen or modification to an existing one?** If modifying, read the current file first.
4. **What is the primary user goal on this screen?** (e.g., review a list, complete a workflow, compare data)
5. **Any specific data or terminology to use?** Prefer realistic healthcare/pharma sample data.
6. **Does this screen need interaction?** (tabs, toggles, modals, inline editing, etc.)
7. **Any screens it should link to or be reachable from?** Sidebar and cross-navigation matter.

If anything is unclear — ask. Do not guess at business logic, personas, or navigation structure.

---

## 1. File Format & Structure

### Self-Contained HTML

Every mockup is a **single `.html` file** with all CSS and JS inline. No external stylesheets, no CDN imports, no build step. The file must open correctly by double-clicking in any browser.

**Desktop only.** Mockups target a standard desktop viewport (~1280px+). Do not add responsive breakpoints, media queries, or mobile layouts.

```
mockups/
├── mockup-guidelines.md              ← This file
├── <feature-name>/                   ← Feature-specific subfolder
│   ├── index.html                    ← Entry point / screen directory for this feature
│   ├── 01-screen-name.html           ← Numbered screens
│   ├── 02-another-screen.html
│   └── ...
└── <another-feature>/
    ├── index.html
    └── ...
```

Each feature or project area gets its own subfolder under `mockups/`. For example:

````
mockups/
├── mockup-guidelines.md
├── accessiq-launch-planning/
│   ├── index.html
│   ├── 01-command-center-dashboard.html
│   ├── 02-account-plan-detail.html
│   └── ...
└── gtn-modelling/
    └── index.html

### Naming Convention

- **Folder:** Lowercase, hyphen-separated feature name (e.g., `accessiq-launch-planning`, `gtn-modelling`)
- **Format:** `NN-short-descriptor.html` (zero-padded two-digit sequence)
- **Numbering:** Sequential within each feature subfolder. Check the subfolder for the highest number and increment.
- **Descriptor:** Lowercase, hyphen-separated, 2–5 words describing the screen.

### index.html

Each feature subfolder has its own `index.html` — a landing page with cards linking to the **main entry-point screens** for that feature (e.g., Command Center Dashboard, Payer Dashboard). Keep it concise — do not add a card for every screen in the subfolder. Secondary screens that are reachable via the sidebar or inline links within other screens can be omitted.

- Only include cards for primary/top-level screens
- Card title, description, and persona role tag
- Footer count reflects the number of cards shown, not total screens
- Remove cards for deleted screens

---

## 2. Design Tokens (CSS Variables)

Every mockup must declare the full `:root` block. Copy this exactly — consistency depends on it.

```css
:root {
  /* Brand / persona */
  --primary: #1a73e8; /* Manufacturer blue */
  --primary-dark: #1557b0;
  --primary-light: #e8f0fe;

  --teal: #0d9488; /* Payer teal */
  --teal-dark: #0f766e;
  --teal-light: #ccfbf1;

  /* Semantic */
  --success: #0d9488;
  --success-light: #ccfbf1;
  --warning: #f59e0b;
  --warning-light: #fef3c7;
  --danger: #ef4444;
  --danger-light: #fee2e2;

  /* Neutrals */
  --gray-50: #f9fafb;
  --gray-100: #f3f4f6;
  --gray-200: #e5e7eb;
  --gray-300: #d1d5db;
  --gray-400: #9ca3af;
  --gray-500: #6b7280;
  --gray-600: #4b5563;
  --gray-700: #374151;
  --gray-800: #1f2937;
  --gray-900: #111827;

  /* Layout */
  --sidebar-bg: #0f172a;
  --sidebar-active: #1e293b;
  --radius: 8px;
  --shadow: 0 1px 3px rgba(0, 0, 0, 0.1), 0 1px 2px rgba(0, 0, 0, 0.06);
}
````

### Persona Color Mapping

| Persona          | Primary accent        | Light bg          | Dark hover       | Use for                                             |
| ---------------- | --------------------- | ----------------- | ---------------- | --------------------------------------------------- |
| **Manufacturer** | `--primary` (#1a73e8) | `--primary-light` | `--primary-dark` | Logo icon, buttons, active tabs, links, role badges |
| **Payer**        | `--teal` (#0d9488)    | `--teal-light`    | `--teal-dark`    | Logo icon, buttons, active tabs, links, role badges |

All screens within a persona family must use that persona's accent color consistently. Do not mix blue buttons on a payer screen or teal buttons on a manufacturer screen.

### Tier / Priority Badges

```css
--badge-a: #7c3aed; /* Purple — Tier A / Critical */
--badge-b: #2563eb; /* Blue — Tier B / Important */
--badge-c: #0891b2; /* Cyan — Tier C / Monitor */
```

---

## 3. Layout Anatomy

Every screen (except `index.html`) follows this structure:

```
┌──────────┬────────────────────────────────────┐
│          │  Topbar (sticky)                    │
│ Sidebar  ├────────────────────────────────────┤
│ (fixed,  │                                    │
│  250px)  │  Content area                      │
│          │  (padding: 28px 32px)              │
│          │                                    │
└──────────┴────────────────────────────────────┘
```

### Sidebar

- **Width:** 250px, fixed left, full height
- **Background:** `var(--sidebar-bg)` (#0f172a)
- **Logo:** Links to `index.html`. Icon uses persona accent color.
- **Nav links:** Grouped under `nav-group-label` uppercase headers (e.g., "AccessIQ", "Documents")
- **Active state:** `var(--sidebar-active)` background, white text, `font-weight: 500`
- **Consistency rule:** All screens in the same persona group must have the **identical sidebar** (same links, same order). When adding/removing a nav item, update every file in that group.

### Topbar

- **Sticky**, white background, bottom border
- **Left:** `<h1>` page title + persona indicator badge
- **Right:** Contextual info or action buttons

### Persona Indicator

```html
<!-- Manufacturer screens -->
<span class="mfr-indicator">Manufacturer</span>
<!-- blue bg/text -->

<!-- Payer screens -->
<span class="payer-indicator">Payer View</span>
<!-- teal bg/text -->
```

---

## 4. Component Patterns

Use these established patterns. Do not invent new component styles when an existing one fits.

### Panels / Cards

```html
<div class="panel">
  <div class="panel-header"><h3>Title</h3></div>
  <div class="panel-body">Content</div>
</div>
```

White background, 1px `--gray-200` border, `var(--radius)` border-radius.

### Metric Cards

Row of summary stats at the top of dashboards. 4 cards in a CSS grid.

### Tables

Standard `<table>` inside panel-body with `padding:0`. Sticky header pattern with `thead` background `var(--gray-50)`.

### Tabs

```html
<div class="tabs">
  <div class="tab active" onclick="showTab('name')">Label</div>
  <div class="tab" onclick="showTab('other')">Label</div>
</div>
<div id="tab-name" class="tab-content active">...</div>
<div id="tab-other" class="tab-content">...</div>
```

Tab switching JS uses the tab name to find `#tab-{name}`. Active tab has persona accent color bottom border.

### Forms (Inline Editing Pattern)

Fields start `disabled` with gray background. An "Edit" button toggles fields to enabled. "Save" commits and returns to read-only. "Cancel" reverts.

```js
function toggleEdit() {
  editing = !editing;
  document.querySelectorAll(".editable-field").forEach((f) => {
    f.disabled = !editing;
    f.style.background = editing ? "#fff" : "";
  });
  // Show/hide save/cancel buttons
}
```

### Workflow Stepper

Horizontal stepper with connected lines. Steps can be `completed`, `current`, or `upcoming`. Steps should be **clickable** to view that stage's details without advancing the workflow. Show a "Viewing — not current stage" indicator when browsing non-current stages.

### Checklists

Items with circular checkboxes (clickable to toggle). Support two item types:

- **AI-suggested items** — show a `✨ AI-suggested` badge (purple gradient background)
- **Manual/standard items** — no badge

Always include an "Add custom item" input below the checklist.

### Status Badges

```css
.severity-critical {
  background: var(--danger-light);
  color: var(--danger);
}
.severity-major {
  background: var(--warning-light);
  color: #b45309;
}
.severity-minor {
  background: var(--gray-100);
  color: var(--gray-500);
}
```

### Buttons

Base `.btn` plus semantic variants:

```css
.btn {
  /* Default: white bg, gray border, gray-700 text */
}
.btn-primary {
  background: var(--primary);
  color: #fff;
  border-color: var(--primary);
}
.btn-success {
  background: var(--success);
  color: #fff;
  border-color: var(--success);
}
.btn-advance {
  background: var(--teal);
  color: #fff;
  border: none;
} /* Payer workflow */
```

Use the persona's accent for primary actions (e.g., `--primary` on manufacturer screens, `--teal` on payer screens). Disabled state uses `var(--gray-300)` background with `cursor: not-allowed`.

### Save Success Banner

Green banner (`--success-light` bg, `--success` border) that appears after save, auto-hides after 4 seconds.

---

## 5. Typography

- **Font stack:** `'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif`
- **Page title (topbar h1):** 20px, weight 600
- **Panel/section h3:** 15–16px, weight 600–700
- **Body text:** 14px, line-height 1.6–1.7, `var(--gray-600)`
- **Labels/captions:** 11–12px, uppercase, `letter-spacing: .5–.8px`, `var(--gray-400)`
- **Small metadata:** 12–13px, `var(--gray-500)`

---

## 6. Interactive Behavior

### JavaScript Rules

- All JS is inline at the bottom of `<body>`, inside a single `<script>` tag.
- Use vanilla JS only. No frameworks, no jQuery.
- Use `onclick` attributes on HTML elements for simple interactions.
- Use `addEventListener` for keyboard events (e.g., Enter key to submit).
- State that would be persisted in production (toggles, checked items) can live in JS variables. Use `alert('Action performed (mockup)')` for actions that would call an API.

### Common Interaction Patterns

| Pattern          | Implementation                                                             |
| ---------------- | -------------------------------------------------------------------------- |
| Tab switching    | Toggle `.active` class on tabs and tab-content divs                        |
| Inline editing   | Toggle `disabled` on form fields + show/hide action buttons                |
| Checklist toggle | Click handler on checkbox div, toggle `.done` class                        |
| Add custom item  | Input + button, push to JS array, re-render list                           |
| Stage browsing   | Click stepper step → update detail panel. Does NOT advance workflow        |
| Advance/submit   | Dedicated button with confirmation. Separate from browsing.                |
| Modal/alert      | Use `alert()` for mockup. Note in comment what the real behavior would be. |

---

## 7. AI-Generated Content Indicators

When a screen displays content that would be AI-generated in production, mark it visually:

```html
<span class="ai-badge">✨ AI-suggested</span>
```

```css
.ai-badge {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  padding: 2px 8px;
  border-radius: 10px;
  font-size: 10px;
  font-weight: 600;
  background: linear-gradient(135deg, #ede9fe, #ddd6fe);
  color: #7c3aed;
  margin-left: 8px;
}
```

Use this on: checklist items, suggested next actions, gap detection results, generated summaries, or any content that comes from AI analysis rather than direct user input.

---

## 8. Sample Data Guidelines

- Use **realistic healthcare/pharma data** — real-sounding drug names, ICD-10 codes, clinical terminology, plausible dates and numbers.
- **Named personas** should be consistent across screens (e.g., "Dr. Sarah Chen" is always the Chief Pharmacy Officer at Aetna).
- **Dates** should be plausible relative to the current date. Use the current year.
- **Numbers** should be realistic for the domain (covered lives in millions, market share as percentages, etc.).
- Do not use "Lorem ipsum" or obviously fake placeholder text.
- **No emoji.** Do not use emoji characters as icons or decorations in mockups. Use plain text labels, Unicode symbols (✓, ←, →), or short text instead.

---

## 9. Cross-Screen Navigation

- **Sidebar links** must work — use relative paths within the feature subfolder (e.g., `href="05-payer-dashboard.html"`).
- **Logo** always links to `index.html` (the feature subfolder's index).
- **Inline links** (e.g., "View Account Plan →") should link to the correct screen file within the same subfolder.
- When creating a new screen, add it to the sidebar of every file in its persona group. Only add a card to `index.html` if it is a primary entry-point screen (e.g., a dashboard or command center) — secondary/detail screens reachable via sidebar or inline links do not need cards.
- When deleting a screen, remove all references from sidebars, `index.html` (if it had a card), and any inline links in other files.

---

## 10. Modification Workflow

When asked to modify existing mockups:

1. **Read the current file(s) first.** Files may have been edited since last session.
2. **Check for ripple effects.** A sidebar change affects all files in the persona group. A deleted screen affects `index.html` + any linking screens.
3. **Preserve existing interaction JS** when editing HTML structure. A checklist that was toggleable must remain toggleable after edits.
4. **Maintain CSS variable usage** — never hard-code a color that has a variable. If you see `#0d9488` in a style, use `var(--teal)` unless it's inside the `:root` definition.
5. **Test mental model:** After your edit, would clicking every link, tab, and button still work?

---

## 11. Don'ts

- **Don't add external dependencies** (CDNs, Google Fonts links, icon libraries). Everything inline.
- **Don't add disclaimer ribbons** ("For client approval", "Not production"). Mockups should look clean.
- **Don't create separate CSS or JS files.** Everything stays in the single HTML file.
- **Don't duplicate screens** when content should be merged into an existing screen (e.g., metadata as a tab rather than a standalone page).
- **Don't use module-level numbering in displayed titles** (show "Command Center Dashboard", not "01 — Command Center Dashboard").
- **Don't forget `index.html`** when adding or removing primary entry-point screens.
- **Don't skip the sidebar** — every non-index screen needs the full sidebar for its persona group.
- **Don't use emoji** as icons or button labels. Use text or Unicode symbols (✓, ←, →) instead.

---

## 12. Checklist Before Delivering

Before marking mockup work as complete, verify:

- [ ] All files are self-contained HTML (open in browser without a server)
- [ ] `:root` variables are complete and consistent across all files
- [ ] Persona accent colors are correct (blue for manufacturer, teal for payer)
- [ ] Sidebar nav is identical across all screens in the same persona group
- [ ] All sidebar links point to correct files (no broken links)
- [ ] `index.html` has cards for primary entry-point screens only (not every screen)
- [ ] `index.html` footer count matches the number of cards shown
- [ ] All interactive elements (tabs, toggles, buttons) function correctly
- [ ] AI-suggested content is marked with the `ai-badge`
- [ ] Sample data is realistic and internally consistent across screens
- [ ] No references to deleted/removed screens remain in any file
