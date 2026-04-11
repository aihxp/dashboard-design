# UI Design Patterns

This file covers the visual design layer — the components, typography, spacing, tokens, motion, and micro-copy that make a dashboard feel polished and professional. The layout structure is in `information-architecture.md`; this file is about what goes inside it.

Dashboard UI is not marketing-site UI. Dashboards are data-dense, functional, used for hours daily. Every decision here optimizes for **scannability, density, and clarity** — not novelty.

---

## Component patterns

### Button hierarchy

Use exactly these variants, in descending visual weight. **One primary per visible context.**

| Variant | Use case | Visual treatment |
|---|---|---|
| **Primary** | Single main action per view (Save, Create, Submit) | Solid brand color bg, white foreground |
| **Secondary** | Supporting actions (Cancel, Back, Export) | Muted bg, standard foreground |
| **Outline** | Tertiary actions, filters, toggles | Border only, transparent bg, hover fills |
| **Ghost** | Inline actions, table row actions, toolbar items | No visible bg at rest, hover fills |
| **Destructive** | Delete, revoke, disconnect — anything irreversible | Red/danger bg, white foreground |
| **Link** | Navigation-styled action, "Learn more" | Text-only, underline on hover |

**Sizes:** `sm` (h-8, text-xs), `default` (h-9, text-sm), `lg` (h-10, text-base), `icon` (h-9 w-9, square).

**Rules:**
- Never put two primary buttons next to each other.
- If a destructive action is the main CTA (like "Delete Account"), use destructive as the primary, paired with ghost/outline cancel.
- Loading state: swap label to present participle ("Saving..."), show spinner inside the button, disable to prevent double-submit.
- Disabled state: reduced opacity (50%), `cursor: not-allowed`, tooltip explaining why.

### Badge / chip / pill

- **Badge** — read-only label, no interaction. Padding 2px 10px, border-radius full, text-xs, font-medium. Use for counts, categories, metadata.
- **Chip** — dismissible or selectable. Same styling plus an X icon (16px) with 4px gap. Use for active filters, tag inputs.
- Semantic colors: `default` (muted bg), `success` (green), `warning` (amber), `error` (red), `info` (blue). Always pair color with a text label — never color alone.

### Avatar

- **Sizes:** 24px (xs), 32px (sm), 40px (md), 48px (lg), 64px (xl).
- **Fallback order:** image > initials (first+last, bg derived from name hash for consistency) > generic user icon.
- **Stack:** Overlap by 25% of width (e.g., -8px margin-left for 32px avatars). Show max 3-5, then `+N` counter in same-size circle with muted bg.
- **Status dot:** 8-12px circle, bottom-right corner, 2px white ring. Green = online, amber = away, gray = offline, red = DND.

### Status indicators

**Always: dot + label + color. Never color alone.**

| Status | Color | Icon supplement |
|---|---|---|
| Active / Healthy / Success | Green | Checkmark |
| Warning / Degraded | Amber | Triangle |
| Error / Failed / Critical | Red | X circle |
| Processing / Pending | Blue | Spinning |
| Inactive / Draft / Disabled | Gray | Dash or empty circle |

Dot: 8px circle, 6px gap to label. In tables, the dot sits inline left of the text in the status cell.

### Toggle vs checkbox

- **Toggle/switch** — for settings that take effect immediately (enable notifications, dark mode). No form submission needed.
- **Checkbox** — for settings that require explicit save/submit (form fields, bulk selection, consent).
- Never use a toggle inside a form that has a Save button — that conflates immediate and deferred actions.

### Progress components

- **Bar:** 8px height (4px compact), border-radius full. Show percentage label above or right. Use `aria-valuenow`, `aria-valuemin`, `aria-valuemax`.
- **Ring/circle:** 40-64px diameter, 4px stroke. Value centered inside. Good for stat cards.
- **Steps:** Horizontal for <6 steps, vertical for 6+. Current = primary fill, completed = primary + checkmark, future = muted/border-only. Connect with 2px lines.

### Dropdown vs command palette

- **Dropdown menu** — 3-12 known options, contextual to a trigger. Max height ~300px, then scroll.
- **Command palette** (Cmd+K) — 10+ items, search-driven, keyboard-first. Group by category. Show keyboard shortcuts right-aligned in muted text.

---

## Typography

### Heading scale

Dashboards rarely need large headings. The working scale:

| Level | Size | Weight | Use |
|---|---|---|---|
| Page title (h1) | 30px / 1.875rem | 700 (bold) | One per page, top of content area |
| Section title (h2) | 24px / 1.5rem | 600 (semibold) | Card group headers, major sections |
| Card title (h3) | 20px / 1.25rem | 600 (semibold) | Individual card/panel headers |
| Subsection (h4) | 16px / 1rem | 600 (semibold) | Sub-headers within cards |
| Label (h5) | 14px / 0.875rem | 500 (medium) | Form group labels, small headers |
| Overline (h6) | 12px / 0.75rem | 500 (medium) | Category labels, ALL CAPS, letter-spacing 0.05em |

### Body text

| Role | Size | Line height | Weight |
|---|---|---|---|
| Body | 14px / 0.875rem | 1.43 (20px) | 400 |
| Body large | 16px / 1rem | 1.5 (24px) | 400 |
| Small / Caption | 12px / 0.75rem | 1.33 (16px) | 400 |
| Tiny / Fine print | 11px / 0.6875rem | 1.27 (14px) | 400 |

**Dashboard base body size is 14px, not 16px.** This is the consensus across Linear, Vercel, Stripe, Ant Design, and every serious data-dense tool. 16px is for marketing sites and long-form reading.

### Monospace and numbers

- Use `font-variant-numeric: tabular-nums` (tabular figures) for all numbers in tables, stat cards, and charts. This ensures digits align vertically.
- For code, IDs, and hashes, use a dedicated monospace font (JetBrains Mono, Fira Code, Geist Mono) at the same size or 1px smaller than surrounding text.
- Right-align all numeric columns. Right-align their headers too.
- Currency: consistent decimal places (always 2 for money). Percentages: 1-2 decimal places.

### Line height rules

- Headings: 1.2-1.3 (tight)
- Body text: 1.4-1.5 (comfortable)
- Single-line labels/values: 1.0
- Multiline table cells: 1.4

### Letter spacing

- Headings >= 24px: `-0.01em` to `-0.02em` (slightly tighter)
- Body: `0` (default)
- Overlines / ALL CAPS: `0.05em` to `0.1em` (looser for legibility)
- Small text < 12px: `0.01em` (slightly looser)

### Font weight usage

- **400** (regular): body text, table cells, descriptions
- **500** (medium): labels, active nav items, badge text, emphasized inline text
- **600** (semibold): headings, card titles, stat values, column headers
- **700** (bold): page titles, hero numbers. Use sparingly in dashboards.

### Truncation

- **Single-line ellipsis:** `overflow: hidden; text-overflow: ellipsis; white-space: nowrap` — for table cells, nav labels, card titles.
- **Multi-line clamp:** `-webkit-line-clamp: 2` (or 3) — for descriptions, preview text.
- **Middle truncation** for URLs/paths: `/users/.../profile` not `/users/settings/pr...` — keep the meaningful suffix visible.
- Always provide full text via `title` attribute or tooltip on hover.
- Tags/badges in a row: show 3, then `+N more` pill. Expand on click.

---

## Spacing and rhythm

### Spacing scale (4px base)

The 4px grid is the standard. Every spacing value is a multiple of 4.

| Value | Use |
|---|---|
| 2px | Icon-to-text micro gap |
| 4px | Tight element gap, badge padding-y |
| 6px | Dense list item padding-y, label-to-input gap |
| 8px | Button padding-y, input padding-y, inline element gap |
| 10px | Badge padding-x |
| 12px | Card padding (compact), button padding-x (sm) |
| 16px | Card padding (default), button padding-x, standard gap |
| 20px | Between form fields |
| 24px | Card padding (comfortable), section spacing within card |
| 32px | Between cards in a grid, between sections |
| 40px | Between major page sections |
| 48px | Page top/bottom margin |
| 64px | Sidebar width collapsed, large section gaps |

### Key decisions

| Element | Spacing |
|---|---|
| Card internal padding | 16px default, 12px compact, 24px comfortable |
| Card gap (grid) | 16px compact, 24px comfortable |
| Between form fields | 16-20px |
| Between form label and input | 6-8px |
| Table cell padding | 8px vertical, 12-16px horizontal (compact: 6px/8px) |
| Section spacing on page | 32px between major sections |
| Sidebar item padding | 8px vertical, 12px horizontal |
| Sidebar width | 240px expanded, 64px collapsed |

### Density modes

Build two modes if the dashboard is data-dense:

| Property | Comfortable | Compact |
|---|---|---|
| Body text | 14px | 13px |
| Table row height | 48px | 36px |
| Table cell padding | 12px 16px | 6px 8px |
| Card padding | 24px | 16px |
| Card gap | 24px | 16px |
| Button height | 36px | 32px |
| Input height | 36px | 32px |

Store the preference per user. Apply via a CSS class on `<body>` or a `data-density` attribute.

---

## Design tokens

### Three-tier system

```
Primitive (raw values) -> Semantic (intent) -> Component (specific usage)
```

**Tier 1 — Primitives:** Raw palette values. Never used directly in components.
```css
--color-gray-50: oklch(0.985 0 0);
--color-gray-900: oklch(0.205 0 0);
--color-blue-500: oklch(0.623 0.214 259);
```

**Tier 2 — Semantic:** Describe purpose, reference primitives. This is where dark mode swapping happens.
```css
--background: var(--color-gray-50);         /* page bg */
--foreground: var(--color-gray-900);        /* primary text */
--muted: var(--color-gray-100);             /* subtle bg */
--muted-foreground: var(--color-gray-500);  /* secondary text */
--primary: var(--color-blue-500);           /* brand action */
--destructive: var(--color-red-500);        /* danger action */
--border: var(--color-gray-200);            /* default borders */
```

**Tier 3 — Component** (optional, for complex systems):
```css
--button-primary-bg: var(--primary);
--card-bg: var(--card);
--card-border: var(--border);
```

### The practical token set

These are the tokens that cover a complete dashboard (matching the shadcn/ui convention, which has become the de facto standard):

`background`, `foreground`, `card`, `card-foreground`, `popover`, `popover-foreground`, `primary`, `primary-foreground`, `secondary`, `secondary-foreground`, `muted`, `muted-foreground`, `accent`, `accent-foreground`, `destructive`, `destructive-foreground`, `border`, `input`, `ring`, `radius`

Plus sidebar variants: `sidebar-background`, `sidebar-foreground`, `sidebar-primary`, `sidebar-accent`, `sidebar-border`

Plus chart tokens: `chart-1` through `chart-5` for data visualization series.

### Naming convention

Pattern: `--{category}-{element}-{property}-{state}`

- `--color-primary` (semantic color)
- `--button-primary-bg` (component token)
- `--button-primary-bg-hover` (component + state)

### Token categories

| Category | What to define |
|---|---|
| Color | background, foreground, primary, secondary, muted, accent, destructive, border, ring, chart-1 through chart-5 |
| Radius | One base value (e.g., 6px), compute sm/md/lg/xl/full from it |
| Shadow | xs through 2xl (use Tailwind defaults) |
| Z-index | See z-index section below |
| Motion | duration-fast (100ms), duration-normal (200ms), duration-slow (300ms), easing |

### Dark mode swapping

Light tokens on `:root`, dark tokens on `.dark`. Use the class strategy (not media query) for dashboards — users want explicit control.

```css
:root {
  --background: oklch(1 0 0);
  --foreground: oklch(0.145 0 0);
}
.dark {
  --background: oklch(0.145 0 0);
  --foreground: oklch(0.985 0 0);
}
```

---

## Motion and animation

### What to animate (and how fast)

| Element | Animation | Duration | Easing |
|---|---|---|---|
| Button hover/press | Background color, scale(0.98) | 150ms | `ease-out` |
| Dropdown open | Opacity 0>1, translateY(-4px>0) | 150ms | `ease-out` |
| Dropdown close | Reverse | 100ms | `ease-in` |
| Modal enter | Overlay fade + content scale(0.95>1) | 200ms | `cubic-bezier(0.16, 1, 0.3, 1)` |
| Modal exit | Reverse | 150ms | `ease-in` |
| Drawer slide in | translateX(100%>0) | 250ms | `cubic-bezier(0.32, 0.72, 0, 1)` |
| Drawer slide out | Reverse | 200ms | `ease-in` |
| Toast enter | translateY(100%>0) + fade | 200ms | `ease-out` |
| Toast exit | Opacity 0, height collapse | 150ms | `ease-in` |
| Skeleton shimmer | Gradient sweep left-to-right | 1.5-2s | `linear` (infinite) |
| Sidebar collapse | Width 240px>64px | 200ms | `ease-in-out` |
| Accordion expand | Height 0>auto | 200ms | `ease-out` |
| Chart data update | Value interpolation | 300-500ms | `ease-out` |

### Duration guidelines

- **Instant feedback** (hover, press, toggle): 100-150ms
- **Standard transitions** (open/close, show/hide): 150-250ms
- **Complex motion** (drawer slide, page transition): 200-350ms
- **Data animation** (charts, counters): 300-600ms
- **Never exceed 500ms** for any UI transition. This is a tool, not a presentation.

### Easing curves

| Use | CSS value |
|---|---|
| Default / general | `ease` or `cubic-bezier(0.4, 0, 0.2, 1)` |
| Enter / appear | `ease-out` or `cubic-bezier(0, 0, 0.2, 1)` |
| Exit / disappear | `ease-in` or `cubic-bezier(0.4, 0, 1, 1)` |
| Modal open (overshoot) | `cubic-bezier(0.16, 1, 0.3, 1)` |
| Drawer slide | `cubic-bezier(0.32, 0.72, 0, 1)` |
| Linear (shimmer, progress) | `linear` |

### Reduced motion

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

Keep opacity fades (safe for motion-sensitive users). Remove all transforms, slides, and bounces.

### What NOT to animate

- Table row data changes — just swap values
- Filter application results — just re-render
- Pagination — just swap content
- Sort column changes
- Breadcrumb updates
- Anything that would delay a power user

---

## Data-dense UI patterns

### Number formatting

Use `Intl.NumberFormat` for all number display. Never hand-roll.

```javascript
// Compact (1.2K, 3.4M)
new Intl.NumberFormat('en', { notation: 'compact', maximumFractionDigits: 1 })

// Currency
new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' })

// Percentage
new Intl.NumberFormat('en', { style: 'percent', minimumFractionDigits: 1 })
```

Rules:
- Compact display for stat cards and chart axes. Full precision in tables and detail views.
- Always locale-aware — pass the user's locale, never hardcode `'en-US'`.
- Tabular figures for all number columns.
- Right-align all numeric columns AND their headers.

### Timestamps

| Context | Format | Example |
|---|---|---|
| Within 1 minute | Relative | "just now" |
| Within 1 hour | Relative | "12 min ago" |
| Within 24 hours | Relative | "3 hours ago" |
| Within 7 days | Relative | "2 days ago" |
| Older than 7 days | Absolute | "Mar 15, 2026" |
| Older than 1 year | Absolute with year | "Mar 15, 2025" |
| Tables / audit logs | Absolute always | "2026-03-15 14:32:05" |
| Hover tooltip on relative | Full absolute | "March 15, 2026 at 2:32 PM EST" |

Rule: Audit logs and data tables always use absolute timestamps. Relative is for activity feeds and casual contexts. Always show full timestamp on hover.

### Inline editing affordance

- Hover: show subtle pencil icon or background change (`bg-accent/50`).
- Click: replace content with an input, auto-focused, pre-filled.
- Save on Enter or blur. Cancel on Escape.
- Brief inline success indicator (checkmark, fades after 1.5s).
- Never inline-edit critical fields (email, permissions, financial amounts) — use a modal or detail page.

### Skeleton loading

- Skeleton shape must match the content it replaces: rectangle for text, circle for avatars, rounded rect for cards.
- Table skeletons: 5-8 rows with **randomized widths** (not identical) so they look organic.
- Shimmer: `linear-gradient(90deg, transparent, rgba(255,255,255,0.4), transparent)` sweeping at 1.5s, linear, infinite.
- If data loads in <200ms, don't show a skeleton at all — use a `setTimeout` delay before rendering to prevent flash.

---

## Micro-copy

### Button labels

- **Primary actions:** Verb + noun. "Create Project", "Save Changes", "Send Invite", "Export Report".
- **When context is obvious:** Verb only. "Save", "Delete" (in a dialog where the noun is clear from the title).
- **Never:** "OK", "Yes", "No", "Click Here", "Submit" (as a generic).
- **Destructive confirmation:** Repeat the specific action. "Delete 3 Users" not "Confirm" or "Yes".
- **Loading state:** Present participle. "Creating..." "Saving..."

### Form labels

- **Sentence case** for all labels ("First name", "Email address", not "First Name", "EMAIL ADDRESS"). This is the modern standard.
- Label above input (not beside, not placeholder-as-label).
- Required: if most fields are required, mark optional fields with "(optional)" suffix instead of asterisks.
- Help text: muted color, 12px, below the input. Specific instructions, not restating the label.

### Error messages

Three parts: **What happened** + **Why** + **What to do.**
- "That email is already registered. Try signing in instead, or use a different email."
- "Password must be at least 8 characters. Add 3 more characters."
- Never: "Invalid input", "Error", "Something went wrong", "Validation failed."

Show inline under the field in danger color, not in a banner at the top (unless the error is form-wide).

### Empty state copy

Structure: **Title** + **Description** + **CTA button**.
- Title: what the user will see here. "No projects yet."
- Description: one sentence of value. "Create your first project to start tracking work."
- CTA: verb + noun. "Create Project".
- Don't say "Nothing to see here." Frame positively.

### Toast messages

| Type | Icon | Duration | Example |
|---|---|---|---|
| Success | Checkmark (green) | 3-5s auto-dismiss | "Project created successfully." |
| Error | X circle (red) | Persistent | "Failed to save. Please try again." |
| Warning | Triangle (amber) | 5-8s | "API key expires in 3 days." |
| Info | Info circle (blue) | 5s | "2 users were imported." |

Position: bottom-right (or top-right). Stack max 3, queue the rest. Always include a dismiss X.

### Confirmation dialog copy

- Title: verb + noun, specific. "Delete 3 users?" not "Are you sure?"
- Body: state the consequence. "This will permanently remove these users and all their data."
- Cancel: "Cancel" (secondary). Always present.
- Confirm: specific verb matching action. "Delete 3 Users" (destructive variant). Never "OK" or "Yes".

### Placeholder text

- Format hints only, not labels. `placeholder="name@company.com"` not `placeholder="Email"`.
- Muted color, lighter weight than real input text.
- If the user needs persistent help, use help text below the input, not placeholder.

---

## Responsive adaptation

### Component-specific behavior

**Tables:**
- >= 1024px: full table, all columns.
- 768-1023px: hide low-priority columns. Keep: name/title, status, primary metric, actions.
- < 768px: horizontal scroll with sticky first column and shadow fade on the right edge. OR convert to card layout (each row becomes a stacked card).

**Filter bars:**
- Desktop: inline horizontal row.
- Tablet: wrap to 2 rows or collapse less-used into "More Filters" dropdown.
- Mobile: single "Filters" button > full-screen drawer with stacked controls. Badge with active filter count.

**Button groups:**
- Desktop: all visible in a row.
- Mobile: primary action visible, rest overflow into a `...` menu.

**Stat cards:**
- Desktop: 4-column grid.
- Tablet: 2-column.
- Mobile: 2-column compact or single-column stack.
- Never 3-column — breaks awkwardly to 2.

**Charts:**
- Maintain aspect ratio, single chart per row on mobile.
- Remove legends on mobile, use direct labels instead.
- Enable horizontal scroll for wide bar charts.

**Forms:**
- Always single column on mobile.
- Desktop: single column for standard forms (reads better). Two columns ONLY for short paired fields (city + state, first + last name).
- Never three-column forms.

---

## Dark mode

### Surface elevation

In dark mode, **lighter = higher elevation** (closer to the light source). Inverted from light mode.

| Level | Light mode | Dark mode |
|---|---|---|
| Page background | White | Near-black (`#0a0a0a` to `#171717`) |
| Card / panel | White (border-separated) | Slightly lighter (`#1a1a1a`) |
| Popover / dropdown | White + shadow | Lighter still (`#262626`) |
| Modal | White + heavy shadow | Even lighter (`#2a2a2a`) |

**Never pure black** (`#000`) for the page background. It creates excessive contrast with text and looks harsh. Use `#0a0a0a` to `#171717`.

### Color adjustments

- **Brand colors:** Reduce saturation ~20%, increase lightness ~10-15%. Vibrant blue on white looks neon/jarring on dark gray.
- **Text hierarchy:** Primary text at ~87-90% white, secondary at ~60-65%, disabled at ~38%.
- **Borders:** More important in dark mode (shadows are less visible). Adjust opacity up slightly.
- **Shadows:** Increase opacity from 10% to 25-40%. Or skip shadows entirely in dark mode and rely on surface color differences + borders.

### Chart colors

- Desaturate ~15-20% and lighten ~10%.
- Every chart color must meet WCAG 3:1 contrast against the chart area background.
- Define dark-mode chart tokens (`chart-1` through `chart-5`) that swap with the theme.

### Image treatment

- Reduce brightness slightly (`filter: brightness(0.9)`) on user-uploaded images.
- Provide light-on-dark logo variant. `filter: invert(1)` as last resort.

---

## Icons

### Sizing scale

| Size | Use |
|---|---|
| 12px | Inline with small/caption text (rare) |
| 16px | Inline with body text, table row actions, dense UI |
| 20px | Inside buttons (with text), form field icons, nav items (compact) |
| 24px | Standalone, nav items (standard), card actions — the default icon size |
| 32px | Empty state feature icons, large action buttons |
| 40-48px | Empty state centerpieces, hero illustrations |

### Filled vs outlined

- **Outlined (stroke):** Default for dashboards. Lower visual weight, cleaner in data-dense layouts. Use for nav, toolbars, table actions, form icons.
- **Filled (solid):** Use for active/selected states (active nav item), status indicators, and sizes below 20px where stroke detail breaks down.
- Never mix filled and outlined in the same context. Active state can switch to filled as a selection indicator.

### Icon + label pairing

- Gap: 8px (default), 6px (compact).
- Icon left of label (LTR). Exception: external link arrow goes right.
- Icon color matches label color unless it's a status indicator.
- Vertical alignment: center to the text's x-height, not full line-height.

### Icon-only buttons

Acceptable ONLY when:
1. The icon is universally understood (X/close, search/magnifier, gear/settings, hamburger/menu, plus/add, trash/delete), AND
2. The button has `aria-label` ("Close dialog", not "X"), AND
3. A tooltip shows the label on hover.

Minimum touch target: 44x44px even if the icon is 16px.

---

## Shadow and elevation

### Shadow scale

| Token | Use |
|---|---|
| `shadow-xs` | Subtle lift — input fields, secondary buttons |
| `shadow-sm` | Standard cards (when not using border-only), floating toolbar |
| `shadow-md` | Dropdowns, popovers, elevated cards |
| `shadow-lg` | Drawers, panels, command palette |
| `shadow-xl` | Modals, dialogs |

### Border vs shadow

Modern dashboard trend (Linear, Vercel, shadcn/ui): prefer **1px borders** over shadows for cards and sections. Reserve shadows for overlays (dropdowns, modals, drawers). This produces a cleaner, flatter, more data-focused look.

### Focus states

- Use `ring` utilities for keyboard focus, not shadow.
- Default: 2px ring in the `ring` token color, 2px offset from the element.
- **Never remove focus outlines.** Style them if the defaults are ugly — don't delete them.

### Dark mode shadows

Increase opacity from 10% to 25-40%. Or skip shadows entirely in dark mode and rely on surface color + borders (the Material Design 3 approach).

---

## Z-index management

### Layering system

| Layer | Value | Components |
|---|---|---|
| Base content | `0` | Page content, cards, tables |
| Sticky elements | `100` | Sticky table headers, sticky sidebar |
| Floating actions | `200` | FABs, floating toolbars |
| Dropdown / Popover | `1000` | Menus, popovers, date pickers, selects |
| Sticky header | `1100` | App bar, top navigation |
| Drawer | `1200` | Sidebar drawer overlay, sheets |
| Modal backdrop + modal | `1300` | Dialog content and overlay |
| Toast | `1400` | Toast notifications (must appear above modals) |
| Tooltip | `1500` | Tooltips (above everything) |

### Rules

1. **Never use arbitrary values** (no `z-index: 99999`). Reference a token.
2. **100-increment gaps** between layers for sub-layering room.
3. **Portals solve most problems:** render modals, dropdowns, tooltips, and toasts at the end of `<body>`. Removes them from parent stacking contexts.
4. **Z-index only works on positioned elements** (relative, absolute, fixed, sticky).
5. **Toasts above modals:** error toasts must be visible even when a modal is open.

```css
:root {
  --z-base: 0;
  --z-sticky: 100;
  --z-dropdown: 1000;
  --z-header: 1100;
  --z-drawer: 1200;
  --z-modal: 1300;
  --z-toast: 1400;
  --z-tooltip: 1500;
}
```

---

## Don'ts

- **Don't use more than 2 font families.** One sans-serif for UI, one monospace for data/code. A third is clutter.
- **Don't use 16px as body text in a dashboard.** 14px is the standard for data-dense tools. 16px wastes space.
- **Don't animate pagination, sort, or filter results.** Swap the content instantly.
- **Don't use rainbow colors for status.** Five semantic colors (green/amber/red/blue/gray) cover every status. Map them consistently.
- **Don't use shadow AND border on the same card.** Pick one. Border is the modern default.
- **Don't mix icon families.** One family (Lucide, Heroicons, Phosphor). Mixed families look chaotic.
- **Don't mix filled and outlined icons** in the same context.
- **Don't use custom z-index values.** Use the token scale. `z-index: 99999` is a symptom.
- **Don't remove focus outlines.** Style them — don't delete them.
- **Don't use `color` alone to convey meaning.** Always pair with icon, label, or shape.
- **Don't exceed 500ms** on any UI transition in a dashboard.
- **Don't show skeletons for loads under 200ms.** Add a delay before rendering the skeleton.
- **Don't use pure black** (`#000`) as dark mode background. Use `#0a0a0a` to `#171717`.
- **Don't hand-roll number formatting.** Use `Intl.NumberFormat` with the user's locale.
- **Don't use "Submit" as a button label.** Use verb + noun: "Create Project", "Save Changes".
