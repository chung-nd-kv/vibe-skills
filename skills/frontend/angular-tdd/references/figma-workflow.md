# Figma Design Workflow for Angular

How to analyze Figma designs and translate them into Angular components.

---

## 1. Requesting the Design

### Prompt Template

When the task involves UI work, ask:

> "Does this task have a Figma design? If yes, please share the Figma link so I can review the design specifications (layout, spacing, colors, typography, responsive breakpoints)."

### No Figma Available — Fallback

If no Figma design exists, ask the user for:

1. **Wireframe description** — rough layout description in text
2. **Screenshot or mockup** — any visual reference (even hand-drawn)
3. **Acceptance criteria** — behavioral requirements for the UI
4. **Reference page** — an existing page in the app to mimic

Document whatever is provided and proceed with best judgment. Flag any design assumptions in the REVIEW phase.

---

## 2. Design Analysis Checklist

When reviewing a Figma design, extract the following:

### Layout Structure
- [ ] Overall layout pattern (flex row/column, grid, sidebar + content)
- [ ] Container widths and max-widths
- [ ] Spacing between sections
- [ ] Alignment rules (center, start, space-between)
- [ ] Overflow behavior (scroll, hidden, wrap)

### Typography
- [ ] Font family (match to project's font or closest available)
- [ ] Font sizes for each text level (heading, body, caption, label)
- [ ] Font weights (regular, medium, semibold, bold)
- [ ] Line heights
- [ ] Letter spacing (if non-default)
- [ ] Text color variations (primary, secondary, muted, error)

### Colors
- [ ] Primary, secondary, accent colors
- [ ] Background colors (surface, card, page)
- [ ] Border colors
- [ ] Text colors per context
- [ ] Status colors (success, warning, error, info)
- [ ] Map to existing design tokens or CSS variables in the project

### Spacing System
- [ ] Consistent spacing scale used (4px, 8px, 16px, 24px, etc.)
- [ ] Padding inside components
- [ ] Margin between components
- [ ] Gap in flex/grid layouts

### Responsive Breakpoints
- [ ] Mobile layout (< 768px)
- [ ] Tablet layout (768px - 1024px)
- [ ] Desktop layout (> 1024px)
- [ ] What changes between breakpoints (column stacking, hidden elements, font sizes)

### Interactive States
- [ ] Default state
- [ ] Hover state
- [ ] Focus state (keyboard navigation)
- [ ] Active/pressed state
- [ ] Disabled state
- [ ] Loading state
- [ ] Error state
- [ ] Empty state

### Animations / Transitions
- [ ] Entry/exit animations
- [ ] Hover transitions (duration, easing)
- [ ] Loading animations (skeleton, spinner)
- [ ] Page transitions

---

## 3. Component Decomposition

Break the design into Angular components following atomic design principles:

### Step 1: Identify Atomic Components
- Buttons, inputs, labels, icons, badges, avatars
- Check if these already exist in the project (Angular Material, shared UI lib)

### Step 2: Identify Molecular Components
- Form fields (label + input + error), card, list item, search bar
- Check if existing shared components can be reused

### Step 3: Identify Organism Components
- Form sections, data tables, navigation, header, sidebar
- These are typically feature-specific

### Step 4: Identify Page/Template Components
- Full page layouts combining organisms
- Smart components that manage state and data flow

### Decomposition Template

```
Page: [PageName]
├── [OrganismComponent] — [responsibility]
│   ├── [MoleculeComponent] — [responsibility]
│   │   ├── [AtomComponent] — existing ✅ / new 🆕
│   │   └── [AtomComponent] — existing ✅ / new 🆕
│   └── [MoleculeComponent] — [responsibility]
└── [OrganismComponent] — [responsibility]
    └── [MoleculeComponent] — [responsibility]
```

---

## 4. Design-to-Angular Mapping

### Spacing → SCSS Variables / Tailwind

```scss
// If project uses SCSS variables
$spacing-xs: 4px;
$spacing-sm: 8px;
$spacing-md: 16px;
$spacing-lg: 24px;
$spacing-xl: 32px;
```

```html
<!-- If project uses Tailwind -->
<div class="p-4 gap-6 mt-8">...</div>
```

### Colors → Theme Tokens

```scss
// Map Figma colors to existing project tokens
// Check _variables.scss, theme.scss, or tailwind.config.js
$color-primary: #...;    // Figma: "Primary/500"
$color-surface: #...;    // Figma: "Surface/Default"
```

### Typography → Styles

```scss
// Map Figma text styles to existing project classes or mixins
// heading-1 → .text-xl.font-bold or @include heading-1
```

### Breakpoints → Media Queries

```scss
// Match Figma responsive frames to project breakpoints
@media (max-width: 768px) { ... }   // Figma "Mobile" frame
@media (max-width: 1024px) { ... }  // Figma "Tablet" frame
```

### Icons → Icon System

- Identify icons used in design
- Check project's icon system (Angular Material icons, custom SVGs, icon font)
- Map Figma icon names to available icons

---

## 5. Post-Implementation Design Review

After implementing the UI, verify against the design:

### Visual Comparison
- [ ] Layout matches design structure
- [ ] Spacing is consistent with design
- [ ] Typography (size, weight, color) matches
- [ ] Colors match design tokens
- [ ] Border radius, shadows match
- [ ] Icons are correct

### Responsive Behavior
- [ ] Mobile layout matches mobile frame
- [ ] Tablet layout matches tablet frame
- [ ] Desktop layout matches desktop frame
- [ ] Transitions between breakpoints are smooth

### Interactive States
- [ ] Hover effects match design
- [ ] Focus states are visible and accessible
- [ ] Disabled states match design
- [ ] Loading states match design
- [ ] Error states match design

### Edge Cases
- [ ] Long text (truncation, wrapping)
- [ ] Missing/null data
- [ ] Empty lists/tables
- [ ] Single item vs many items
- [ ] Very long lists (scroll behavior)

### Accessibility
- [ ] Semantic HTML used (headings, landmarks, lists)
- [ ] ARIA labels where needed
- [ ] Keyboard navigation works
- [ ] Color contrast meets WCAG AA (4.5:1 for text)
- [ ] Focus indicators visible
