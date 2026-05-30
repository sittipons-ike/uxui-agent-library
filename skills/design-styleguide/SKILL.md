---
name: design-styleguide
description: Render a human-readable Style Guide from a design system. Auto-detects split-architecture (design.md + components.md + ui.md) OR legacy monolithic DESIGN.md. Outputs HTML/Tailwind preview page AND/OR Figma canvas frames (via figma-console MCP). Shows semantic tokens + atomic components (atom/molecule/organism) + ui compositions (page/pattern/section/flow). Skips primitive layer (too technical for designers). Triggers on "style guide", "styleguide", "design system page", "team review page", "render design system", "สร้าง style guide", "ทำหน้า DS", "preview DS ให้ทีม".
version: 2.0.0
user-invokable: true
args:
  - name: source
    description: Path to DESIGN.md (default ./DESIGN.md)
    required: false
  - name: output
    description: "html | figma | both — default: html"
    required: false
---

# 🎨 Design Styleguide

Generate a human-readable Style Guide from a `DESIGN.md` so designers, PMs, and stakeholders can review and discuss the design system without reading YAML.

## When to use
- Designer wants to review tokens visually before approving the DESIGN.md
- Need a shareable link/Figma frame for stakeholder review
- Onboarding new team members
- Before/after comparison when iterating the DS

## When NOT to use
- File doesn't exist yet → use `design-builder` first
- Need machine-readable output → DESIGN.md already is that
- Want token-level audit → use `design-md-audit`

## Scope
**INCLUDED:**
- Semantic colors (all role groups × scale)
- Semantic typography (every role rendered)
- Semantic spacing, radius, elevation
- Component examples (every variant × state where applicable)
- Do's and Don'ts (read from DESIGN.md body)

**EXCLUDED:**
- Primitive layer (raw hex / Tailwind hues) — too technical for design discussion
- YAML / code blocks — designers don't need them

## Execution Steps

### 1. Load source
- **Split mode (preferred):** look for `./design.md` + `./components.md` + `./ui.md`. Load all that exist.
- **Legacy mode:** look for `./DESIGN.md` (monolithic). Use if no split files.
- Parse YAML frontmatter from each file; verify `scope:` matches filename
- Extract semantic tokens from design.md (primitives silent for hex resolution only)
- Extract atom/molecule/organism from components.md (resolve `{design.semantic.*}` refs)
- Extract page/pattern/section/flow from ui.md (resolve `{components.*}` refs)
- For monolithic legacy, all sections come from one file with bare-path refs

### 2. Pick output mode
Based on `output` arg (default `html`):
- `html` — generate `STYLEGUIDE.html` next to DESIGN.md
- `figma` — paint frames on current Figma canvas via `figma-console` MCP
- `both` — do both

### 3a. HTML output

Generate a single self-contained file `STYLEGUIDE.html`:
- Tailwind CDN (no build step required)
- One `<section>` per category
- Final hex/px values inline (not as `{primitive.*}`-refs)
- Designer-friendly labels (semantic names, not paths)

**Sections (in this order):**

1. **Header** — brand name, last-updated date, version (from DESIGN.md frontmatter)
2. **Color roles** — each group renders as swatch grid:
   - `text` — sample text "Aa" + label + resolved hex
   - `surface` / `background` — solid swatch + label + hex
   - `primary` / `secondary` / `tertiary` × 5-stop scale — horizontal row of 5 swatches
   - `status` × 4 channels × 5-stop scale — grouped grid
   - `border` — line samples
3. **Typography** — every role rendered with actual text. Show: role label + `{family} {size}/{line-height} {weight}` + sample paragraph
4. **Spacing scale** — visual bars (xs to 3xl) with px labels
5. **Radius scale** — square samples with each radius
6. **Elevation** — cards with each shadow level
7. **Components** — for each component:
   - Variant name + size
   - All states rendered as live mockups (default, hover, pressed, focused, disabled)
   - Use CSS pseudo-classes for hover/focus so designers can actually interact
   - Below each: the semantic ref path (e.g., `bg.hover → {semantic.colors.primary.dark}`)
8. **Do's and Don'ts** — copy from DESIGN.md body, render as 2-column with ✓ / ✗

**HTML template requirements:**
- Mobile + desktop responsive
- Light + dark mode toggle (use `prefers-color-scheme` AND a manual toggle)
- Search/jump-to-section nav
- Copy-token button next to each swatch (clipboard semantic path)
- Print stylesheet (designers print these for reviews)

### 3b. Figma output

Use `figma-console` MCP. Connection check first via `figma_get_status`.

**Frame layout (top-down on canvas):**

```
Section: Brand Header  (1440 × 200)
Section: Colors        (1440 × auto, grid of swatches)
Section: Typography    (1440 × auto, sample text per role)
Section: Spacing       (1440 × 400)
Section: Radius        (1440 × 400)
Section: Elevation     (1440 × 400)
Section: Components    (1440 × auto, per component)
Section: Do/Don't      (1440 × auto)
```

**Rules:**
- Create one `Section` per category (Figma section, not frame)
- Use existing design system components if found via `figma_search_components`
- Token swatches as auto-layout frames with text labels
- Label every swatch with semantic name (NOT hex — Figma users care about token name)
- DO NOT modify existing pages — create a new page named `Style Guide v<n>`
- Place all sections in one vertical column with 80px gap

**Component rendering rule:**
- For each variant × state combo → 1 frame
- Frame name: `Button / Primary / Hover` (Figma slash naming)
- This lets designers right-click → "Detach instance" to use as starting point

### 4. Validate output
Self-check before delivering:
- [ ] No `{primitive.*}` refs visible in any output (resolve all to hex/px)
- [ ] No YAML / code blocks visible
- [ ] Every color swatch has: visual + label + resolved hex
- [ ] Every component variant has at least default state rendered
- [ ] Do/Don't section has at least 4 pairs
- [ ] HTML passes basic WCAG AA contrast for its OWN UI (not the swatches it shows)
- [ ] Figma output: no duplicate page named "Style Guide v<n>" (increment if exists)

### 5. Deliver
- HTML → tell user the file path + suggest `open STYLEGUIDE.html`
- Figma → tell user the new page name + link to it via `figma_navigate`

## Output Format Rules
- Designer-friendly: no jargon without definition
- Semantic names visible; primitive refs hidden
- Final hex/px shown (resolved, not symbolic)
- Real text in typography samples — not "Lorem ipsum"
- Use brand name from DESIGN.md frontmatter consistently

## Constraints
- Read-only on DESIGN.md — never modify it
- HTML must be self-contained (single file, CDN dependencies OK)
- Figma: never delete existing pages
- If DESIGN.md is incomplete (Phase 1 only / no semantic), output what exists + note "Component layer not yet defined" in a Known Gaps section
- If `primitive:` block is missing → cannot resolve hexes → ABORT with clear message

## Quality Bar
A designer should:
- Understand the entire DS in 5 minutes scanning the styleguide
- Identify which token to use for a new design without reading DESIGN.md
- Spot inconsistencies (e.g., 2 status colors that look identical) visually
- Be able to print + bring to a review meeting

A developer should:
- Use the styleguide as the visual reference + DESIGN.md as the machine reference
- NEVER read the styleguide to extract values — go to DESIGN.md instead
