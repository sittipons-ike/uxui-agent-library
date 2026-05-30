---
name: design-ui-builder
description: Build the ui.md file in a 3-file split-architecture design system. Reads design.md (tokens, layout) + components.md (atoms/molecules/organisms), generates UI compositions in 4 categories — page (full screens), pattern (reusable cross-page structures like auth-split / app-shell / empty-state), section (marketing blocks like hero / pricing-table), flow (multi-step sequences like signup / checkout). All composition refs use cross-file prefix syntax {components.organism.*}, {components.molecule.*}, {components.atom.*}. Prefers organism-level refs over atoms (promote to molecule/organism if composing atoms directly). Triggers on "build ui", "build pages", "ui compositions", "create ui.md", "build patterns", "สร้าง pages", "ui layer", "page composition".
version: 1.0.0
user-invokable: true
args:
  - name: source-design
    description: Path to design.md (default ./design.md)
    required: false
  - name: source-components
    description: Path to components.md (default ./components.md)
    required: false
  - name: categories
    description: "Comma-separated categories to build: page,pattern,section,flow or 'all'. Default: page,pattern"
    required: false
---

# 🖼️ Design UI Builder

Final layer in the 3-file split. Reads design + components → outputs ui compositions.

## When to use
- Have `./design.md` + `./components.md` built (via design-builder + design-component-builder)
- Want to define pages / patterns / sections / flows in a structured spec
- Need a source-of-truth that maps every page to component refs

## When NOT to use
- No `./design.md` yet → run `design-builder` first
- No `./components.md` yet → run `design-component-builder` first
- Want to render UI as HTML/Figma → use `design-styleguide`
- Want to check completeness → use `design-md-audit`

## UI taxonomy — 4 categories

| Category | Definition | Examples |
|---|---|---|
| **page** | Full screen with route | login, dashboard, settings, pricing |
| **pattern** | Reusable cross-page structure | auth-split, app-shell, empty-state, error-state, 404, loading |
| **section** | Marketing/landing block | hero, pricing-table, testimonial, cta-banner, faq |
| **flow** | Multi-step sequence | signup, onboarding, checkout, new-receipt |

### When to use which
- `page` — every URL route gets a page entry
- `pattern` — when same structure reused across 3+ pages
- `section` — landing/marketing pages composed from sections
- `flow` — multi-step user journeys

## Execution Steps

### 1. Load sources
- design.md: must exist + scope `'tokens-only'`
- components.md: must exist + scope `'components-only'`
- If either missing → ABORT with clear next-step ("Run design-builder" / "Run design-component-builder")
- Read mood from design.md (informs pattern decisions like auth-split density, hero whitespace)
- Read all atoms / molecules / organisms from components.md (build a manifest)

### 2. Determine output file
- Default: `./ui.md`
- If exists → MERGE new categories; don't overwrite existing
- frontmatter MUST include:
  ```yaml
  scope: 'ui-compositions'
  depends-on:
    - './design.md'
    - './components.md'
  ```

### 3. Pick categories to build
Default: `page, pattern` (most useful set).
Use `categories` arg to extend: `categories=page,pattern,section,flow`.

Ask user which specific items to add per category (multi-select):
- Pages: which routes? (login, signup, dashboard, settings, profile, billing, pricing, 404, ...)
- Patterns: auth-split / app-shell / empty-state / error-state / loading-state / not-found-404 / success-state / split-view
- Sections: hero / pricing-table / testimonial / cta-banner / faq / feature-grid / logo-cloud
- Flows: signup / onboarding / checkout / password-reset / invite-team

### 4. Generate ui.md YAML shape

```yaml
ui:
  page:
    <route-name>:
      uses-pattern: '<pattern-name>'              # optional
      slots:                                       # if uses-pattern
        <slot-name>: { ... content fills ... }
      composed-of:                                 # alternative: explicit composition
        <slot>: '{components.organism.<name>}'
      seo:
        title: '...'
        description: '...'

  pattern:
    <pattern-name>:
      composed-of:
        <slot>: 'slot-placeholder' | '{components.organism.<name>}'
      layout:
        grid: '...'
        responsive: { mobile: '...', tablet: '...' }
        # design.semantic.* refs allowed for layout values

  section:
    <section-name>:
      composed-of:
        <slot>: '{components.atom.<name>}' | '{components.molecule.<name>}'
      layout: { ... }
      content-defaults:
        <field>: 'default text'

  flow:
    <flow-name>:
      steps:
        - step: 1
          page: '<page-name>'
          uses-pattern: '...'
        - step: 2
          modal: '<modal-name>'
      navigation:
        progress-indicator: 'step n of N'
        back-button: 'always-visible'
```

### 4b. WCAG AA — required structure per category

**Every page MUST declare these a11y slots.** Audit fails if missing.

#### page (required structure)
```yaml
ui:
  page:
    <name>:
      a11y:
        h1:               # MUST exist exactly once per page
          slot: 'page-title'
          content: '<page name>'
        landmarks:
          - 'main'        # MUST wrap primary content
          - 'header'      # if topbar/brand present
          - 'nav'         # if sidebar/menu present (with aria-label)
          - 'contentinfo' # if footer present
        skip-link:
          target: '#main'
          label: 'Skip to content'
          visually-hidden-until-focus: true
        page-title:       # <title> tag
          format: '<page name> — <brand>'
        focus-management: 'reset to h1 on route change'
```

If page composition uses a pattern, the pattern declares the landmarks; page fills the h1 slot.

#### pattern (required structure)
```yaml
ui:
  pattern:
    <name>:
      a11y:
        landmarks-provided:
          - 'main'
          - 'nav'   # if applicable
        skip-link-slot: true        # pattern must include skip link slot if has nav
        keyboard-navigation:
          tab-order: 'logical reading order'
          focus-trap: 'only inside modal/drawer'
```

#### flow (required structure)
```yaml
ui:
  flow:
    <name>:
      a11y:
        progress-indicator:
          aria-label: 'Step N of M'
          role: 'progressbar'
        live-region:
          role: 'status'
          aria-live: 'polite'
          purpose: 'announce step transitions'
        back-button:
          aria-label: 'Back to step <N-1>'
          visible: 'always'
        forward-button:
          required-attr: 'aria-disabled when invalid'
```

#### section (required structure)
```yaml
ui:
  section:
    <name>:
      a11y:
        heading-level:    # which h-level this section starts at
          required: 'h2'  # OR specified per use
        landmark: 'region'
        required-attr: 'aria-labelledby points to section heading'
```

### 5. Composition rules

**Prefer organism refs in pages:**
```
✅  page.dashboard.slots.sidebar: '{components.organism.sidebar}'
⚠️  page.dashboard.slots.sidebar: '{components.molecule.nav-item}' × N   # promote to organism
❌  page.dashboard.slots.sidebar: '{components.atom.button.ghost}'        # too granular
```

If you find yourself composing atoms inside a page → promote to molecule/organism in components.md first.

**Patterns are structural skeletons:**
- Layout + slot positions only
- Content comes from pages that use the pattern
- Slots are named placeholders (`'editorial-aside-slot'`)

**Sections are content + layout bundles:**
- Include `content-defaults` so they're usable out-of-the-box
- Pages can override defaults

**Flows orchestrate pages:**
- Don't define new layouts — flows are step lists
- Each step refs a page or modal
- Specify back/skip/progress navigation rules

### 6. Required body sections in ui.md

After the YAML, the markdown body MUST include:
1. `## Overview` — explain split architecture role
2. `## UI taxonomy` — definition of page/pattern/section/flow
3. `## Built so far — v<version>` — list of each category's items
4. `## Composition rules` — organism-first, slot semantics, etc.
5. `## Do's and Don'ts` — at least 5 pairs
6. `## Known Gaps` — refs to components.md items that don't exist yet (dangling refs)
7. `## Iteration Guide` — how to add new pages/patterns/sections/flows

### 7. Validate output
- [ ] YAML has `ui:` root with `page:` / `pattern:` / `section:` / `flow:` (at least one)
- [ ] All component refs use prefix syntax: `{components.atom.*}`, `{components.molecule.*}`, `{components.organism.*}`
- [ ] All token refs (for layout only) use prefix: `{design.semantic.*}`
- [ ] No bare-path refs (`{semantic.*}` without prefix → fail)
- [ ] No upward refs (ui can't be ref'd from components/design)
- [ ] Pages prefer organism over atom refs (warn if 3+ atom refs in a page)
- [ ] Patterns have `layout:` block
- [ ] Sections have `content-defaults:` block
- [ ] Flows have `steps:` block with step number + page/modal
- [ ] Dangling refs surfaced in Known Gaps section

**WCAG AA (Critical):**
- [ ] Every page has `a11y.h1` slot (exactly one h1 per page)
- [ ] Every page has `a11y.landmarks` listing main + nav (if applicable)
- [ ] Every page has `a11y.skip-link` target = `#main`
- [ ] Every page has `a11y.page-title` format declared
- [ ] Patterns declare which landmarks they provide
- [ ] Flows declare `live-region` for step transitions
- [ ] Flows declare `back-button` with aria-label
- [ ] Sections declare heading-level + landmark

### 8. Save
- Write `./ui.md`
- Report to user: categories built, count per category, dangling refs found
- Tell user next: run `design-md-audit` to validate cross-file refs

## Constraints
- READ design.md + components.md once at start; don't re-read after each addition
- Do NOT touch design.md or components.md — additive only on ui.md
- Do NOT invent components — every ref must resolve to something in components.md (otherwise flag as Known Gap)
- Do NOT mix bare-path and prefixed refs — use prefixed everywhere

## Quality Bar
A coding agent reading `ui.md` + `components.md` + `design.md` should be able to build the actual UI without further questions. Each page tells the agent:
- Which pattern/layout to apply
- Which organisms fill which slots
- Which atoms are used directly (rare — should be promoted)
- What content goes where (slot fills)

A designer reading just `ui.md` should understand the page inventory + flow structure without opening `components.md`.
