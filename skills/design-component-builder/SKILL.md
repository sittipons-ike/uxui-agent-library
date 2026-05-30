---
name: design-component-builder
description: Build the components.md file in a 3-file split-architecture design system. Reads design.md (tokens + mood), generates atomic component library (atom / molecule / organism) with full prop + state aliases (surface/content/edge/elevation/focus-halo × rest/hover/active/focus/disabled/selected). All refs use cross-file prefix syntax {design.semantic.*}. Mood-biased state mapping. Supports incremental tier addition (A→E) AND atomic-level reasoning. Triggers on "build components", "add components", "atomic components", "เพิ่ม component", "สร้าง components", "atomic design", "component layer".
version: 3.0.0
user-invokable: true
args:
  - name: source
    description: Path to DESIGN.md (default ./DESIGN.md)
    required: false
  - name: tiers
    description: "Comma-separated tiers to add: A,B,C,D,E or 'all'. Default: A"
    required: false
---

# 🧩 Design Component Builder

Tier-3 layer builder. Reads an existing `DESIGN.md` and appends `component:` tokens that follow its mood and refs.

## When to use
- Have a `DESIGN.md` with primitive + semantic already built (via `design-builder`)
- Want to add components incrementally, tier by tier
- Mood already set — components should match it without re-asking

## When NOT to use
- No `DESIGN.md` yet → run `design-builder` first
- Want to add icons → use `design-icon-builder`
- Want to check what's there → use `design-md-audit`

## 3-tier atomic architecture

| Layer | Examples | Purpose |
|---|---|---|
| **atom** | button, input, icon, badge, label, avatar | Smallest indivisible UI piece |
| **molecule** | form-field, nav-item, search-bar, stat-tile | Atoms composed into a small unit |
| **organism** | sidebar, topbar, hero, table | Molecules composed into a feature block |

Template + page tiers live in app code, NOT in DESIGN.md.

## Tier menu — atoms grouped by function

| Tier | Atoms | Use when |
|---|---|---|
| **A** (MVP) | button, input, card | Always — required baseline |
| **B** Form | checkbox, radio, toggle, select | Forms exist in the product |
| **C** Feedback | badge, alert, toast, tooltip | User feedback is shown anywhere |
| **D** Navigation | tab, nav-link, breadcrumb, pagination | Multi-page or in-page nav |
| **E** Overlay | modal, drawer, popover, dropdown | Dialogs / menus used |

Per-tier completeness: include all 4 OR skip entirely + declare in `## Known Gaps`. **Partial tier = audit fail.**

## Alias names (per NAMING.md § 6)

### Prop aliases
| Use case | Alias |
|---|---|
| container fill | `surface` (was: bg) |
| foreground text/icon | `content` (was: fg) |
| outline / stroke | `edge` (was: border) |
| depth shadow | `elevation` (was: shadow) |
| focus halo | `focus-halo` (was: ring) |
| inner spacing | `inset-x` / `inset-y` (was: padding-x/y) |
| corner softness | `corner` (was: radius) |
| composed type | `text-style` (was: typography) |

### State aliases (canonical 7)
```
rest, hover, active, focus, disabled, selected, error
```
- `rest` (was: default) — idle
- `active` (was: pressed) — CSS `:active`
- `focus` (was: focused) — CSS `:focus`
- `selected` (was: active for nav-current) — chosen / current

## Execution Steps

### 1. Load source
- Default source: `./design.md` (tokens-only file in split architecture)
- Legacy fallback: `./DESIGN.md` (monolithic) — if found, INFO: "Run split-migrate via design-md-audit --migrate"
- Parse YAML frontmatter — verify `scope: 'tokens-only'`
- Verify `primitive:` AND `semantic:` blocks exist — ABORT if missing
- Read `mood.primary` for state-mapping bias
- Read `iconography:` to know if icon layer exists

### 1b. Determine output file
- Default output: `./components.md` (separate file in split architecture)
- If `components.md` already exists → MERGE new tiers; don't overwrite existing keys
- frontmatter MUST include:
  ```yaml
  scope: 'components-only'
  depends-on: ['./design.md']
  ```
- All refs MUST use prefix syntax: `{design.semantic.*}`, `{design.primitive.*}` (avoid primitive), `{components.atom.*}`, `{components.molecule.*}`

### 2. Pick tiers
- If `tiers` arg provided, use it
- Else ask user which tier(s) to add (multi-select)
- Tier A always included unless DESIGN.md already has button+input+card

### 3. Apply mood-biased state mapping

Default canonical mapping (already in NAMING.md § 5.2):
```
default  → semantic.<role>.default
hover    → semantic.<role>.dark
pressed  → semantic.<role>.darker
focused  → semantic.<role>.default + ring
disabled → semantic.secondary.light
```

**Mood overrides:**
| Mood | Override |
|---|---|
| `bold-tech` | hover bumps 2 stops (`darker` not `dark`), pressed adds inner shadow, focused ring `@60%` (thicker) |
| `friendly-warm` | hover uses `soft-light` for bg shifts (gentler), pressed uses `light`, rings `@30%` |
| `premium-editorial` | hover stays `default` + border color change only, pressed `dark`, rings `@20%` (very subtle) |
| `playful-vivid` | hover scales/shifts saturation, pressed darker + slight scale, rings colored `@50%` |
| `technical-dev` | hover `dark`, pressed `darker`, focused thick outline ring, no shadow change |
| `calm-focused` | default canonical mapping (no override) |

Apply this when emitting state values.

### 4. Generate component YAML — atomic + alias shape

Append `component:` block to DESIGN.md. If `component:` already exists, MERGE — don't overwrite.

Structure:
```yaml
component:
  atom:
    <name>:
      sizes:
        md:
          inset-x:    '{semantic.spacing.md}'
          inset-y:    '{semantic.spacing.sm}'
          corner:     '{semantic.radius.md}'
          text-style: '{semantic.typography.label.md}'
      <variant>:
        surface:
          rest:    '{semantic.colors.<role>.default}'
          hover:   '{semantic.colors.<role>.<mood-stop>}'
          active:  '{semantic.colors.<role>.dark}'
          focus:   '{semantic.colors.<role>.default}'
          disabled:'{semantic.colors.secondary.light}'
        content:
          rest:    '{semantic.colors.text.on-bgcolor}'
          # ... full 5 states for interactive variants
        edge:        # optional
          rest: '...'
        elevation:   # optional
          rest: '...'
        focus-halo:
          focus: '{semantic.colors.<role>.light}@<mood-opacity>%'

  molecule:
    <name>:
      composed-of:
        <slot>: '{component.atom.<atom-name>.<variant>}'
        # ... list atoms used
      layout:
        stack-gap: '{semantic.spacing.xs}'
        direction: 'column'
      <state-override>:                    # optional — molecule-level state override
        <slot>.<prop>.<state>: '...'

  organism:
    <name>:
      composed-of:
        <slot>: '{component.molecule.<mol-name>.<variant>}'
        # OR atom slots if no molecule needed
      layout:
        extent-x: 240                       # numeric or token ref
        surface.rest: '{semantic.colors.surface.raised}'
        edge.rest: '{semantic.colors.border.tertiary}'
        inset: '{semantic.spacing.lg}'
        stack-gap: '{semantic.spacing.md}'
```

### Prop aliases — use these (not bg/fg/border)
- **surface** = container fill (was `bg`)
- **content** = text + icon foreground (was `fg`)
- **edge** = outline (was `border`)
- **elevation** = depth shadow (was `shadow`)
- **focus-halo** = focus indicator (was `ring`)

### State aliases — use these
```
rest, hover, active, focus, disabled, selected, error
```

## WCAG AA — required a11y attrs per atom

**Every atom MUST declare these accessibility attrs in its YAML spec.** Audit fails if missing.

### atom.button
```yaml
a11y:
  hit-area-min:  '{design.primitive.a11y.touch-target-min-px}'   # 44
  role:          'button'
  required-attrs: ['aria-label']   # if no visible text
  forbidden:     ['onclick without keyboard handler']
```

Sizes that compute visual height < 44px MUST add invisible padding so total hit area ≥ 44.

### atom.icon
```yaml
a11y:
  required-one-of:
    - 'aria-hidden="true"'    # decorative
    - 'aria-label="<verb>"'   # functional (then role="img")
  forbidden: ['ambiguous role']  # don't leave SVG unannotated
```

Decorative icons (inside button with text label) → `aria-hidden`.
Functional icons (standalone, like icon-only button) → `aria-label` + `role="img"`.

### atom.input
```yaml
a11y:
  required-attrs:
    - 'id'                       # for label association
    - 'aria-describedby'         # link to helper-text / error
  required-slots:
    - label                      # via <label for="..."> OR aria-label
    - helper-text-id             # for aria-describedby target
  error-state:
    add-attrs: ['aria-invalid="true"', 'aria-describedby points to error message']
```

### atom.label
```yaml
a11y:
  required-attr: 'for'           # MUST associate with input id
```

### atom.helper-text
```yaml
a11y:
  required-attr: 'id'            # so input.aria-describedby can target it
  error-state:
    role: 'alert'                # screen reader announces immediately
```

### atom.avatar / atom.badge
```yaml
a11y:
  required-one-of:
    - 'aria-label="<person/status name>"'
    - 'aria-hidden="true"'       # if decorative + labeled elsewhere
```

### atom.brand-mark
```yaml
a11y:
  aria-hidden: 'true'            # decorative, brand wordmark adjacent reads it
```

## WCAG AA — required a11y attrs per molecule

### molecule.nav-item
```yaml
a11y:
  selected-state:
    add-attr: 'aria-current="page"'   # OR "step", "location", "true" per context
```

### molecule.form-field
```yaml
a11y:
  composed-a11y:
    - 'label.for = input.id'
    - 'input.aria-describedby = helper.id'
  error-state:
    - 'input.aria-invalid = "true"'
    - 'helper.role = "alert"'
```

### molecule.search-bar
```yaml
a11y:
  required-attrs:
    - 'role="search"'
    - 'input.aria-label = "Search ..."'
  icon.aria-hidden: 'true'       # decorative — input has the label
```

### molecule.user-pill
```yaml
a11y:
  avatar.aria-hidden: 'true'     # name label adjacent
  required-attr: 'role="button" when interactive'
```

## WCAG AA — required a11y attrs per organism

### organism.sidebar
```yaml
a11y:
  landmark: 'nav'
  required-attr: 'aria-label="Primary navigation"'
```

### organism.topbar
```yaml
a11y:
  landmark: 'banner' OR 'header'
```

### organism.modal
```yaml
a11y:
  role: 'dialog'
  required-attrs:
    - 'aria-modal="true"'
    - 'aria-labelledby points to dialog title'
  required-behavior:
    - 'focus trap when open'
    - 'restore focus on close'
    - 'close on ESC'
```
- `rest` (was `default`)
- `active` (was `pressed` — CSS `:active`)
- `focus` (was `focused`)
- `selected` (was `active` for nav-current)

### 5. Required matrix per tier

**Tier A — required variants/states (per-prop completeness):**

Button:
- 5 variants: `primary`, `secondary`, `tertiary`, `ghost`, `destructive`
- Mandatory props: `bg`, `fg`
- Optional: `border`, `shadow`, `ring` — include if visual design uses them (comment `# no shadow by design` when omitted)
- All present props must have all 5 states: default, hover, pressed, focused, disabled

Input:
- 1 variant: `text`
- Mandatory props: `bg`, `fg`, `border`
- States: default, hover, focused, disabled (no pressed — input doesn't press)
- Optional: `ring`, `helper-text`, `error` validation state

Card:
- 2 variants: `default` (display), `interactive`
- `default` states: default, hover (2 only)
- `interactive` states: full 5 states on all props

**Tier B (Form):**
- checkbox + radio: states include `selected` (in addition to interactive 4)
- toggle: state `active` (boolean on/off, not selected)
- select.trigger: same shape as input.text

**Tier C (Feedback):**
- badge: 3 variants (solid, soft, outline) — display only, 1 state each
- alert: 4 status channels (info, success, warning, error), all display
- toast: 4 status channels, all display
- tooltip: 2 variants (default, inverse), 1 state each

**Tier D (Navigation):**
- tab: 2 variants (underline, pills), state `active` (= selected tab)
- nav-link: state `active` (= current route)
- breadcrumb: state `active` (= current crumb)
- pagination: state `active` (= current page)

**Tier E (Overlay):**
- modal, drawer, popover: display variants — 1 default state
- dropdown.item: state `selected` + full interactive

### 6. Update body section

Append `## Component Tokens` section with:
- Tier table for each included tier (Component / Variants / Sizes)
- Required matrix table
- Canonical state→scale mapping (mood-adjusted)
- Variant notes for design decisions worth recording

### 7. Update Known Gaps

If user skipped a tier:
- Add line to `## Known Gaps`: `- **Tier <X> (<name>)** — skipped because <reason>`
- Without this declaration, audit will flag the skip as Major

### 8. Validate output
- [ ] `component:` block has nested `atom:` / `molecule:` / `organism:` keys (3-tier)
- [ ] All refs use `{semantic.*}` or `{component.atom.*}` / `{component.molecule.*}` — NEVER `{primitive.*}`, NEVER raw hex
- [ ] Tier flow valid: organism → molecule/atom, molecule → atom. NO upward refs
- [ ] Prop names use aliases: surface / content / edge / elevation / focus-halo (NOT bg/fg/border/shadow/ring)
- [ ] State names use aliases: rest / hover / active / focus / disabled / selected / error (NOT default/pressed/focused)
- [ ] State is LAST path segment
- [ ] Per-prop completeness per tier kind
- [ ] No partial tiers (all 4 in a tier or declared in Known Gaps)
- [ ] Variants skipping a prop have inline `# comment` explaining why
- [ ] Molecules have `composed-of:` block listing atoms used
- [ ] Organisms have `composed-of:` block listing molecules (or atoms) used
- [ ] No invented state names

**WCAG AA (Critical):**
- [ ] Every atom has `a11y:` block declaring required attrs / hit-area / role
- [ ] atom.button has `hit-area-min: 44` — sizes that compute smaller add invisible padding
- [ ] atom.icon has required-one-of `aria-hidden` or `aria-label`
- [ ] atom.input has required `id`, `aria-describedby`, label association
- [ ] molecule.nav-item declares `aria-current` on selected state
- [ ] molecule.form-field composes label.for + input.aria-describedby
- [ ] organism with role MUST declare landmark + aria-label

### 9. Save
- Edit DESIGN.md in place (additive — don't rewrite primitive/semantic)
- Tell user which tiers added + total component count

## Constraints
- READ DESIGN.md once at start; don't re-read after each component
- Do NOT touch primitive or semantic blocks — additive only
- Do NOT change mood — read it, apply it
- If existing `component:` block already has a key, ask before overwriting
- Keep YAML lean — short comments only, no prose in YAML

## Quality Bar
A second agent reading the resulting `component:` block should:
- Know every state value without inferring
- Be able to map any state to its semantic scale stop in one lookup
- See the mood reflected in the mapping (e.g., friendly-warm uses soft-light hover instead of dark)
