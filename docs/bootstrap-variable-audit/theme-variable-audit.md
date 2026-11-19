# Bootstrap 5.3.9 theme variable and derivation audit

Research document. Nothing in `src/bootstrap/` was modified and no Sass is part of this repository. All counts, values and line numbers below were produced mechanically from the v5.3.9 CSS and Sass (see methodology) and cross-checked by source map.

## Contents

- [A. Introduction and methodology](#a-introduction-and-methodology)
- [B. Summary of findings](#b-summary-of-findings)
- [C. Reference value groups](#c-reference-value-groups)
- [D. Independent values and other patterns](#d-independent-values-and-other-patterns)
- [E. Component/source index](#e-componentsource-index)
- [F. Opportunities for semantic theme tokens](#f-opportunities-for-semantic-theme-tokens)

## A. Introduction and methodology

**What was audited.** Every `--bs-*` custom property declared in the readable Bootstrap 5.3.9 CSS in `src/bootstrap/reference/css/`, with each declaration traced back to the Sass in the official `twbs/bootstrap` tag `v5.3.9` (commit `349bdb5`).

**Files used.**

- `src/bootstrap/reference/css/bootstrap.css` (12,064 lines) is the audit baseline. Bootstrap does not ship per-component CSS files, so it is the only file that contains every component.
- `bootstrap-reboot.css` (`:root` variables), `bootstrap-grid.css` and `bootstrap-utilities.css` are the individual files. Where a declaration also exists in one of them, both locations are listed (individual file first). Every declaration in these three files is also in `bootstrap.css`. The `*.rtl.css` files contain the same declarations and were not listed separately.
- Sass: `scss/_root.scss`, `_variables.scss`, `_variables-dark.scss`, `_maps.scss`, `_functions.scss`, `_utilities.scss`, the component partials, `forms/`, `helpers/` and `mixins/`, plus `vendor/_rfs.scss`. The source was fetched to a temporary directory and has been deleted.

**How properties were traced.**

1. The reference CSS was parsed to get each `--bs-*` declaration with its exact selector, at-rule context, value and line.
2. Bootstrap's Sass (the exact pinned compiler version 1.78.0) was run in a temporary directory to regenerate the CSS with source maps. The output differs from the shipped dist only in vendor-prefixed lines (autoprefixer); every custom property declaration is identical. The source map gives, for each declaration, the Sass line that generated it and the line of the rule/mixin call that produced the selector. These appear in the "Sass source" column (`via` = call site).
3. The generating Sass line was expanded by following each `$variable` and map lookup to its default value (`!default`), stopping at design-level references (theme colours, palette steps, semantic body/border/shadow variables, scales). Mixins with parameters (buttons, tables, pagination, form validation, alert/list-group state loops, grid, utilities API) were bound to the actual call-site arguments, so the expression shown is what was evaluated for that rule.
4. For `var(--bs-x)` aliases the chain was followed to the `:root` declaration, both in light and dark colour mode, to know what the alias resolves to.

**Reading the tables.** "Sass expression" is the bound, expanded expression with `$variable` names left intact; the compiled value is the exact CSS text. Nothing in the compiled value column has been paraphrased. In a heading, `$ANCHOR` stands for the group's reference variable.

**Relationship classification** (not mutually exclusive; a row can carry several):

| Label | Meaning |
|---|---|
| Direct | the value is a bare Sass variable or map entry, with no function or maths |
| Derived | calculated from reference values by a Sass function (`shade-color`, `tint-color`, `shift-color`, `mix`, `color-contrast`, `opaque`, `to-rgb`, `percentage`, `rgba`, `if`, `escape-svg`) or arithmetic |
| Independent | hardcoded literal, or a variable with no `$` and no `var()` in the chain to a shared reference |
| Runtime | the compiled value is `var(--bs-…)` (or contains it), so it follows the other property at run time |
| Compile-time | Sass evaluated a function or arithmetic, so the output is a fixed value that will **not** follow a runtime change of its inputs |
| Mixed | more than one distinct reference, or derivation combined with a runtime reference |

**Grouping rules.** Each declaration context is anchored to the single lowest-level design reference it depends on (theme colour > semantic body/border/shadow variable > palette step > spacing > radius > border width > typography > other). Within an anchor, contexts with the identical expression pattern are grouped into a subgroup. "Shares a reference" and "shares a calculation" are different things: two subgroups under the same anchor share the reference, only rows inside one subgroup share the calculation. Two different anchors that happen to compile to the same value are never merged.

**Limitations.**

- The anchor is a heuristic summary; a property with several references (e.g. a table variant uses `$body-bg` and a theme colour) is listed once, under its lowest-level reference, and its full reference list is in the expression.
- Component-local runtime aliases (`--bs-nav-link-*` → `--bs-navbar-*`, `--bs-table-*-type/state` aliases) point to properties set by another rule at run time; their value cannot be established from the Sass source alone and they are reported as untraceable to a shared reference (section D).
- Colour maths was not re-derived by hand; the compiled values are Bootstrap's own output.
- Dist values were taken from the readable files, not the `.min` ones.

## B. Summary of findings

- **Unique `--bs-*` custom properties:** 449
- **Distinct declaration contexts** (selector + at-rule context + property, so redeclarations count separately): 1186
- Contexts declared inside a dark-mode selector: 69; light/default or non-colour-mode: 1117
- Anchored to a shared reference (section C): 935 contexts; independent or component-local (section D): 251 contexts

**Contexts by anchor category**

| Category | Contexts |
|---|---|
| theme | 438 |
| unanchored | 158 |
| palette | 128 |
| body / emphasis colour | 112 |
| component | 107 |
| spacing | 98 |
| border-radius | 32 |
| typography | 29 |
| border colour | 25 |
| border-width | 15 |
| shadow | 11 |
| breakpoints/grid | 10 |
| z-index | 8 |
| opacity | 7 |
| focus | 4 |
| aspect ratio | 4 |

**Contexts by relationship label**

| Label | Contexts |
|---|---|
| Direct | 684 |
| Mixed | 383 |
| Compile-time | 309 |
| Runtime | 275 |
| Derived | 205 |
| Independent | 147 |

**Most frequent Sass functions (contexts using each)**

| Function | Contexts |
|---|---|
| `color-contrast` | 120 |
| `map-get` | 82 |
| `opaque` | 64 |
| `mix` | 64 |
| `percentage` | 56 |
| `shift-color` | 54 |
| `to-rgb` | 44 |
| `shade-color` | 30 |
| `escape-svg` | 19 |
| `tint-color` | 18 |
| `rgba-css-var` | 8 |
| `inspect` | 3 |
| `rgba` | 1 |
| `if` | 1 |

**Most reused Sass variables / map entries** (all references found in the expanded expressions)

| Reference | Contexts | Distinct properties |
|---|---|---|
| `$primary` | 90 | 62 |
| `$spacer` | 86 | 28 |
| `$body-bg` | 83 | 27 |
| `$success` | 57 | 43 |
| `$danger` | 57 | 43 |
| `$secondary` | 54 | 40 |
| `$info` | 54 | 40 |
| `$warning` | 54 | 40 |
| `$table-bg-scale` | 54 | 9 |
| `$light` | 36 | 25 |
| `$dark` | 36 | 25 |
| `$body-emphasis-color` | 36 | 21 |
| `$white` | 33 | 27 |
| `$black` | 28 | 11 |
| `$border-width` | 20 | 20 |
| `$border-radius` | 18 | 18 |
| `$body-color` | 17 | 17 |
| `$table-striped-bg-factor` | 17 | 2 |
| `$table-active-bg-factor` | 17 | 2 |
| `$table-hover-bg-factor` | 17 | 2 |
| `$font-size-base` | 15 | 11 |
| `$gray-700` | 14 | 9 |
| `$focus-ring-opacity` | 13 | 5 |
| `$border-color` | 12 | 12 |
| `$border-color-translucent` | 12 | 10 |

**Reference dependency map (anchors)**

| Anchor | Category | Contexts | Properties |
|---|---|---|---|
| [`$primary`](#c-primary) | theme | 90 | 62 |
| [`$secondary`](#c-secondary) | theme | 54 | 40 |
| [`$success`](#c-success) | theme | 57 | 43 |
| [`$info`](#c-info) | theme | 54 | 40 |
| [`$warning`](#c-warning) | theme | 54 | 40 |
| [`$danger`](#c-danger) | theme | 57 | 43 |
| [`$light`](#c-light) | theme | 36 | 25 |
| [`$dark`](#c-dark) | theme | 36 | 25 |
| [`$body-bg`](#c-body-bg) | body / emphasis colour | 18 | 18 |
| [`$body-bg-dark`](#c-body-bg-dark) | body / emphasis colour | 2 | 2 |
| [`$body-color`](#c-body-color) | body / emphasis colour | 17 | 17 |
| [`$body-color-dark`](#c-body-color-dark) | body / emphasis colour | 4 | 4 |
| [`$body-emphasis-color`](#c-body-emphasis-color) | body / emphasis colour | 36 | 21 |
| [`$body-emphasis-color-dark`](#c-body-emphasis-color-dark) | body / emphasis colour | 2 | 2 |
| [`$body-secondary-bg`](#c-body-secondary-bg) | body / emphasis colour | 8 | 8 |
| [`$body-secondary-bg-dark`](#c-body-secondary-bg-dark) | body / emphasis colour | 2 | 2 |
| [`$body-secondary-color`](#c-body-secondary-color) | body / emphasis colour | 9 | 9 |
| [`$body-secondary-color-dark`](#c-body-secondary-color-dark) | body / emphasis colour | 2 | 2 |
| [`$body-tertiary-bg`](#c-body-tertiary-bg) | body / emphasis colour | 5 | 5 |
| [`$body-tertiary-bg-dark`](#c-body-tertiary-bg-dark) | body / emphasis colour | 2 | 2 |
| [`$body-tertiary-color`](#c-body-tertiary-color) | body / emphasis colour | 3 | 3 |
| [`$body-tertiary-color-dark`](#c-body-tertiary-color-dark) | body / emphasis colour | 2 | 2 |
| [`$border-color`](#c-border-color) | border colour | 11 | 11 |
| [`$border-color-dark`](#c-border-color-dark) | border colour | 1 | 1 |
| [`$border-color-translucent`](#c-border-color-translucent) | border colour | 12 | 10 |
| [`$border-color-translucent-dark`](#c-border-color-translucent-dark) | border colour | 1 | 1 |
| [`$box-shadow`](#c-box-shadow) | shadow | 5 | 5 |
| [`$box-shadow-inset`](#c-box-shadow-inset) | shadow | 2 | 2 |
| [`$box-shadow-lg`](#c-box-shadow-lg) | shadow | 1 | 1 |
| [`$box-shadow-sm`](#c-box-shadow-sm) | shadow | 3 | 3 |
| [`$black`](#c-black) | palette | 26 | 9 |
| [`$blue`](#c-blue) | palette | 1 | 1 |
| [`$cyan`](#c-cyan) | palette | 1 | 1 |
| [`$gray-100`](#c-gray-100) | palette | 6 | 6 |
| [`$gray-200`](#c-gray-200) | palette | 6 | 6 |
| [`$gray-300`](#c-gray-300) | palette | 4 | 4 |
| [`$gray-400`](#c-gray-400) | palette | 5 | 5 |
| [`$gray-500`](#c-gray-500) | palette | 8 | 8 |
| [`$gray-600`](#c-gray-600) | palette | 4 | 4 |
| [`$gray-700`](#c-gray-700) | palette | 14 | 9 |
| [`$gray-800`](#c-gray-800) | palette | 7 | 7 |
| [`$gray-900`](#c-gray-900) | palette | 1 | 1 |
| [`$green`](#c-green) | palette | 3 | 3 |
| [`$indigo`](#c-indigo) | palette | 1 | 1 |
| [`$orange`](#c-orange) | palette | 1 | 1 |
| [`$pink`](#c-pink) | palette | 3 | 2 |
| [`$purple`](#c-purple) | palette | 1 | 1 |
| [`$red`](#c-red) | palette | 3 | 3 |
| [`$teal`](#c-teal) | palette | 1 | 1 |
| [`$white`](#c-white) | palette | 29 | 24 |
| [`$yellow`](#c-yellow) | palette | 3 | 2 |
| [`$spacer`](#c-spacer) | spacing | 86 | 28 |
| [`$spacers[0]`](#c-spacers-0) | spacing | 12 | 2 |
| [`$border-radius`](#c-border-radius) | border-radius | 18 | 18 |
| [`$border-radius-lg`](#c-border-radius-lg) | border-radius | 7 | 7 |
| [`$border-radius-pill`](#c-border-radius-pill) | border-radius | 1 | 1 |
| [`$border-radius-sm`](#c-border-radius-sm) | border-radius | 3 | 3 |
| [`$border-radius-xl`](#c-border-radius-xl) | border-radius | 1 | 1 |
| [`$border-radius-xxl`](#c-border-radius-xxl) | border-radius | 2 | 2 |
| [`$border-width`](#c-border-width) | border-width | 15 | 15 |
| [`$font-family-monospace`](#c-font-family-monospace) | typography | 1 | 1 |
| [`$font-family-sans-serif`](#c-font-family-sans-serif) | typography | 2 | 2 |
| [`$font-size-base`](#c-font-size-base) | typography | 15 | 11 |
| [`$font-weight-bold`](#c-font-weight-bold) | typography | 1 | 1 |
| [`$font-weight-normal`](#c-font-weight-normal) | typography | 3 | 2 |
| [`$headings-color`](#c-headings-color) | typography | 2 | 2 |
| [`$headings-color-dark`](#c-headings-color-dark) | typography | 1 | 1 |
| [`$line-height-base`](#c-line-height-base) | typography | 4 | 4 |
| [`$focus-ring-opacity`](#c-focus-ring-opacity) | focus | 1 | 1 |
| [`$focus-ring-width`](#c-focus-ring-width) | focus | 3 | 3 |
| [`$zindex-dropdown`](#c-zindex-dropdown) | z-index | 1 | 1 |
| [`$zindex-modal`](#c-zindex-modal) | z-index | 1 | 1 |
| [`$zindex-modal-backdrop`](#c-zindex-modal-backdrop) | z-index | 1 | 1 |
| [`$zindex-offcanvas`](#c-zindex-offcanvas) | z-index | 1 | 1 |
| [`$zindex-popover`](#c-zindex-popover) | z-index | 1 | 1 |
| [`$zindex-toast`](#c-zindex-toast) | z-index | 2 | 1 |
| [`$zindex-tooltip`](#c-zindex-tooltip) | z-index | 1 | 1 |
| [`$grid-breakpoints[lg]`](#c-grid-breakpoints-lg) | breakpoints/grid | 1 | 1 |
| [`$grid-breakpoints[md]`](#c-grid-breakpoints-md) | breakpoints/grid | 1 | 1 |
| [`$grid-breakpoints[sm]`](#c-grid-breakpoints-sm) | breakpoints/grid | 1 | 1 |
| [`$grid-breakpoints[xl]`](#c-grid-breakpoints-xl) | breakpoints/grid | 1 | 1 |
| [`$grid-breakpoints[xs]`](#c-grid-breakpoints-xs) | breakpoints/grid | 1 | 1 |
| [`$grid-breakpoints[xxl]`](#c-grid-breakpoints-xxl) | breakpoints/grid | 1 | 1 |
| [`$grid-gutter-width`](#c-grid-gutter-width) | breakpoints/grid | 4 | 3 |
| [`$aspect-ratios[16x9]`](#c-aspect-ratios-16x9) | aspect ratio | 1 | 1 |
| [`$aspect-ratios[1x1]`](#c-aspect-ratios-1x1) | aspect ratio | 1 | 1 |
| [`$aspect-ratios[21x9]`](#c-aspect-ratios-21x9) | aspect ratio | 1 | 1 |
| [`$aspect-ratios[4x3]`](#c-aspect-ratios-4x3) | aspect ratio | 1 | 1 |
| [`$btn-close-disabled-opacity`](#c-btn-close-disabled-opacity) | opacity | 1 | 1 |
| [`$btn-close-focus-opacity`](#c-btn-close-focus-opacity) | opacity | 1 | 1 |
| [`$btn-close-hover-opacity`](#c-btn-close-hover-opacity) | opacity | 1 | 1 |
| [`$btn-close-opacity`](#c-btn-close-opacity) | opacity | 1 | 1 |
| [`$btn-disabled-opacity`](#c-btn-disabled-opacity) | opacity | 1 | 1 |
| [`$modal-backdrop-opacity`](#c-modal-backdrop-opacity) | opacity | 1 | 1 |
| [`$tooltip-opacity`](#c-tooltip-opacity) | opacity | 1 | 1 |
| [`$accordion-padding-x`](#c-accordion-padding-x) | component | 2 | 2 |
| [`$accordion-padding-y`](#c-accordion-padding-y) | component | 2 | 2 |
| [`$dropdown-padding-y`](#c-dropdown-padding-y) | component | 2 | 2 |
| [`$spinner-width`](#c-spinner-width) | component | 4 | 2 |
| [`$spinner-width-sm`](#c-spinner-width-sm) | component | 4 | 2 |

**Key findings**

- Colour theming is driven almost entirely by the eight `$theme-colors` entries; each one is the anchor of the same set of derived properties (root colour, `-rgb`, `-text-emphasis`, `-bg-subtle`, `-border-subtle`, button, alert, list-group, table, link and focus values).
- Colours derived in the **component** sections for buttons, outline buttons and table variants are compiled to fixed values (`shade-color`/`tint-color`/`color-contrast`/`mix` evaluated by Sass). Overriding `--bs-primary` at run time does **not** change `.btn-primary`, `.table-primary` and so on. Alerts and list-group variants, by contrast, are pure runtime aliases of `--bs-<state>-text-emphasis`, `-bg-subtle` and `-border-subtle` and so do follow run-time changes.
- Dark mode redeclares the semantic colour roots (`--bs-body-*`, `--bs-emphasis-color`, `--bs-border-color`, `--bs-link-*`, `*-text-emphasis`, `*-bg-subtle`, `*-border-subtle`) using different calculations (palette steps and a different tint/shade amount), not the same calculation with different inputs.
- Spacing, radius, border-width and font-size values are single maps or variables that fan out to many components; spacing (`$spacer`) is the main reuse among non-colour values.
- `--bs-link-color-rgb`/`--bs-link-hover-color-rgb` are the only link values that derive from `$primary` in light mode; in dark mode they derive from palette steps (`$blue-300`, `$blue-200`), independent of `$primary`.

## C. Reference value groups

Each group is one anchor. Under it, subgroups share an identical expression pattern. Row columns: property, exact compiled value, selector and context, CSS source (individual file first where present), Sass source (`via` = rule/mixin call site, `+` = other call sites), Sass expression, relationship.

<a id="c-primary"></a>
### `$primary` (theme)

Definition: `$primary: $blue` (`scss/_variables.scss:301`). 90 contexts, 62 properties.

<a id="c-primary-1"></a>
#### $primary – pattern 1: `$ANCHOR`

22 contexts, 19 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-bg` | `#0d6efd` | `.btn-outline-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3178 | scss/mixins/_buttons.scss:53<br>via scss/_buttons.scss:158 | `$primary` | Direct |
| `--bs-btn-active-border-color` | `#0d6efd` | `.btn-outline-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3179 | scss/mixins/_buttons.scss:54<br>via scss/_buttons.scss:158 | `$primary` | Direct |
| `--bs-btn-bg` | `#0d6efd` | `.btn-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3036 | scss/mixins/_buttons.scss:22<br>via scss/_buttons.scss:132 | `$primary` | Direct |
| `--bs-btn-border-color` | `#0d6efd` | `.btn-outline-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3172 | scss/mixins/_buttons.scss:47<br>via scss/_buttons.scss:158 | `$primary` | Direct |
| `--bs-btn-border-color` | `#0d6efd` | `.btn-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3037 | scss/mixins/_buttons.scss:23<br>via scss/_buttons.scss:132 | `$primary` | Direct |
| `--bs-btn-color` | `#0d6efd` | `.btn-outline-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3171 | scss/mixins/_buttons.scss:46<br>via scss/_buttons.scss:158 | `$primary` | Direct |
| `--bs-btn-disabled-bg` | `#0d6efd` | `.btn-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3047 | scss/mixins/_buttons.scss:33<br>via scss/_buttons.scss:132 | `$primary` | Direct |
| `--bs-btn-disabled-border-color` | `#0d6efd` | `.btn-outline-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3183 | scss/mixins/_buttons.scss:58<br>via scss/_buttons.scss:158 | `$primary` | Direct |
| `--bs-btn-disabled-border-color` | `#0d6efd` | `.btn-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3048 | scss/mixins/_buttons.scss:34<br>via scss/_buttons.scss:132 | `$primary` | Direct |
| `--bs-btn-disabled-color` | `#0d6efd` | `.btn-outline-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3181 | scss/mixins/_buttons.scss:56<br>via scss/_buttons.scss:158 | `$primary` | Direct |
| `--bs-btn-hover-bg` | `#0d6efd` | `.btn-outline-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3174 | scss/mixins/_buttons.scss:49<br>via scss/_buttons.scss:158 | `$primary` | Direct |
| `--bs-btn-hover-border-color` | `#0d6efd` | `.btn-outline-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3175 | scss/mixins/_buttons.scss:50<br>via scss/_buttons.scss:158 | `$primary` | Direct |
| `--bs-dropdown-link-active-bg` | `#0d6efd` | `.dropdown-menu`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3425 | scss/_dropdown.scss:40<br>via scss/_dropdown.scss:19 | `$primary` | Direct |
| `--bs-dropdown-link-active-bg` | `#0d6efd` | `.dropdown-menu-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3691 | scss/_dropdown.scss:246<br>via scss/_dropdown.scss:235 | `$primary` | Direct |
| `--bs-link-color` | `#0d6efd` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:95<br>src/bootstrap/reference/css/bootstrap.css:96 | scss/_root.scss:82<br>via scss/_root.scss:1 | `$primary` | Direct |
| `--bs-list-group-active-bg` | `#0d6efd` | `.list-group`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5009 | scss/_list-group.scss:22<br>via scss/_list-group.scss:5 | `$primary` | Direct |
| `--bs-list-group-active-border-color` | `#0d6efd` | `.list-group`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5010 | scss/_list-group.scss:23<br>via scss/_list-group.scss:5 | `$primary` | Direct |
| `--bs-nav-pills-link-active-bg` | `#0d6efd` | `.nav-pills`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3875 | scss/_nav.scss:109<br>via scss/_nav.scss:105 | `$primary` | Direct |
| `--bs-pagination-active-bg` | `#0d6efd` | `.pagination`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4731 | scss/_pagination.scss:18<br>via scss/_pagination.scss:1 | `$primary` | Direct |
| `--bs-pagination-active-border-color` | `#0d6efd` | `.pagination`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4732 | scss/_pagination.scss:19<br>via scss/_pagination.scss:1 | `$primary` | Direct |
| `--bs-primary` | `#0d6efd` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:31<br>src/bootstrap/reference/css/bootstrap.css:32 | scss/_root.scss:18 | `$primary` | Direct |
| `--bs-progress-bar-bg` | `#0d6efd` | `.progress, .progress-stacked`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4944 | scss/_progress.scss:20<br>via scss/_progress.scss:11 | `$primary` | Direct |

<a id="c-primary-2"></a>
#### $primary – pattern 2: `var(--bs-primary-text-emphasis)`

6 contexts, 6 properties. Relationship: Direct, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-accordion-active-color` | `var(--bs-primary-text-emphasis)` | `.accordion`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4566 | scss/_accordion.scss:26<br>via scss/_accordion.scss:5 | `var(--bs-primary-text-emphasis)` | Direct, Runtime, Mixed |
| `--bs-alert-color` | `var(--bs-primary-text-emphasis)` | `.alert-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4876 | scss/_alert.scss:62<br>via scss/_alert.scss:61 | `var(--bs-primary-text-emphasis)` | Runtime, Mixed |
| `--bs-alert-link-color` | `var(--bs-primary-text-emphasis)` | `.alert-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4879 | scss/_alert.scss:65<br>via scss/_alert.scss:61 | `var(--bs-primary-text-emphasis)` | Runtime, Mixed |
| `--bs-list-group-active-bg` | `var(--bs-primary-text-emphasis)` | `.list-group-item-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5241 | scss/_list-group.scss:195<br>via scss/_list-group.scss:186 | `var(--bs-primary-text-emphasis)` | Runtime, Mixed |
| `--bs-list-group-active-border-color` | `var(--bs-primary-text-emphasis)` | `.list-group-item-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5242 | scss/_list-group.scss:196<br>via scss/_list-group.scss:186 | `var(--bs-primary-text-emphasis)` | Runtime, Mixed |
| `--bs-list-group-color` | `var(--bs-primary-text-emphasis)` | `.list-group-item-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5233 | scss/_list-group.scss:187<br>via scss/_list-group.scss:186 | `var(--bs-primary-text-emphasis)` | Runtime, Mixed |

<a id="c-primary-3"></a>
#### $primary – pattern 3: `var(--bs-link-hover-color)`

5 contexts, 5 properties. Relationship: Direct, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-color` | `var(--bs-link-hover-color)` | `.btn-link`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3313 | scss/_buttons.scss:177<br>via scss/_buttons.scss:170 | `var(--bs-link-hover-color)` | Direct, Runtime, Mixed |
| `--bs-btn-hover-color` | `var(--bs-link-hover-color)` | `.btn-link`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3311 | scss/_buttons.scss:175<br>via scss/_buttons.scss:170 | `var(--bs-link-hover-color)` | Direct, Runtime, Mixed |
| `--bs-nav-link-hover-color` | `var(--bs-link-hover-color)` | `.nav`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3802 | scss/_nav.scss:13<br>via scss/_nav.scss:6 | `var(--bs-link-hover-color)` | Direct, Runtime, Mixed |
| `--bs-pagination-focus-color` | `var(--bs-link-hover-color)` | `.pagination`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4727 | scss/_pagination.scss:14<br>via scss/_pagination.scss:1 | `var(--bs-link-hover-color)` | Direct, Runtime, Mixed |
| `--bs-pagination-hover-color` | `var(--bs-link-hover-color)` | `.pagination`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4724 | scss/_pagination.scss:11<br>via scss/_pagination.scss:1 | `var(--bs-link-hover-color)` | Direct, Runtime, Mixed |

<a id="c-primary-4"></a>
#### $primary – pattern 4: `color-contrast($ANCHOR)`

4 contexts, 4 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-color` | `#fff` | `.btn-outline-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3177 | scss/mixins/_buttons.scss:52<br>via scss/_buttons.scss:158 | `color-contrast($primary)` | Derived, Compile-time |
| `--bs-btn-color` | `#fff` | `.btn-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3035 | scss/mixins/_buttons.scss:21<br>via scss/_buttons.scss:132 | `color-contrast($primary)` | Derived, Compile-time |
| `--bs-btn-disabled-color` | `#fff` | `.btn-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3046 | scss/mixins/_buttons.scss:32<br>via scss/_buttons.scss:132 | `color-contrast($primary)` | Derived, Compile-time |
| `--bs-btn-hover-color` | `#fff` | `.btn-outline-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3173 | scss/mixins/_buttons.scss:48<br>via scss/_buttons.scss:158 | `color-contrast($primary)` | Derived, Compile-time |

<a id="c-primary-5"></a>
#### $primary – pattern 5: `var(--bs-primary-bg-subtle)`

4 contexts, 4 properties. Relationship: Direct, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-accordion-active-bg` | `var(--bs-primary-bg-subtle)` | `.accordion`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4567 | scss/_accordion.scss:27<br>via scss/_accordion.scss:5 | `var(--bs-primary-bg-subtle)` | Direct, Runtime, Mixed |
| `--bs-alert-bg` | `var(--bs-primary-bg-subtle)` | `.alert-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4877 | scss/_alert.scss:63<br>via scss/_alert.scss:61 | `var(--bs-primary-bg-subtle)` | Runtime, Mixed |
| `--bs-list-group-active-color` | `var(--bs-primary-bg-subtle)` | `.list-group-item-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5240 | scss/_list-group.scss:194<br>via scss/_list-group.scss:186 | `var(--bs-primary-bg-subtle)` | Runtime, Mixed |
| `--bs-list-group-bg` | `var(--bs-primary-bg-subtle)` | `.list-group-item-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5234 | scss/_list-group.scss:188<br>via scss/_list-group.scss:186 | `var(--bs-primary-bg-subtle)` | Runtime, Mixed |

<a id="c-primary-6"></a>
#### $primary – pattern 6: `var(--bs-primary-border-subtle)`

4 contexts, 4 properties. Relationship: Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-alert-border-color` | `var(--bs-primary-border-subtle)` | `.alert-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4878 | scss/_alert.scss:64<br>via scss/_alert.scss:61 | `var(--bs-primary-border-subtle)` | Runtime, Mixed |
| `--bs-list-group-action-active-bg` | `var(--bs-primary-border-subtle)` | `.list-group-item-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5239 | scss/_list-group.scss:193<br>via scss/_list-group.scss:186 | `var(--bs-primary-border-subtle)` | Runtime, Mixed |
| `--bs-list-group-action-hover-bg` | `var(--bs-primary-border-subtle)` | `.list-group-item-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5237 | scss/_list-group.scss:191<br>via scss/_list-group.scss:186 | `var(--bs-primary-border-subtle)` | Runtime, Mixed |
| `--bs-list-group-border-color` | `var(--bs-primary-border-subtle)` | `.list-group-item-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5235 | scss/_list-group.scss:189<br>via scss/_list-group.scss:186 | `var(--bs-primary-border-subtle)` | Runtime, Mixed |

<a id="c-primary-7"></a>
#### $primary – pattern 7: `(0 0 $focus-ring-blur $focus-ring-width rgba($ANCHOR, $focus-ring-opacity))`

3 contexts, 3 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-accordion-btn-focus-box-shadow` | `0 0 0 0.25rem rgba(13, 110, 253, 0.25)` | `.accordion`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4563 | scss/_accordion.scss:23<br>via scss/_accordion.scss:5 | `(0 0 $focus-ring-blur $focus-ring-width rgba($primary, $focus-ring-opacity))` | Direct |
| `--bs-btn-close-focus-shadow` | `0 0 0 0.25rem rgba(13, 110, 253, 0.25)` | `.btn-close`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5341 | scss/_close.scss:12<br>via scss/_close.scss:6 | `(0 0 $focus-ring-blur $focus-ring-width rgba($primary, $focus-ring-opacity))` | Direct |
| `--bs-pagination-focus-box-shadow` | `0 0 0 0.25rem rgba(13, 110, 253, 0.25)` | `.pagination`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4729 | scss/_pagination.scss:16<br>via scss/_pagination.scss:1 | `(0 0 $focus-ring-blur $focus-ring-width rgba($primary, $focus-ring-opacity))` | Direct |

<a id="c-primary-8"></a>
#### $primary – pattern 8: `to-rgb($ANCHOR)`

3 contexts, 3 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-focus-shadow-rgb` | `13, 110, 253` | `.btn-outline-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3176 | scss/mixins/_buttons.scss:51<br>via scss/_buttons.scss:158 | `to-rgb($primary)` | Derived, Compile-time |
| `--bs-link-color-rgb` | `13, 110, 253` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:96<br>src/bootstrap/reference/css/bootstrap.css:97 | scss/_root.scss:83<br>via scss/_root.scss:1 | `to-rgb($primary)` | Derived, Compile-time |
| `--bs-primary-rgb` | `13, 110, 253` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:39<br>src/bootstrap/reference/css/bootstrap.css:40 | scss/_root.scss:22 | `to-rgb($primary)` | Derived, Compile-time |

<a id="c-primary-9"></a>
#### $primary – pattern 9: `var(--bs-link-color)`

3 contexts, 3 properties. Relationship: Direct, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-color` | `var(--bs-link-color)` | `.btn-link`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3308 | scss/_buttons.scss:172<br>via scss/_buttons.scss:170 | `var(--bs-link-color)` | Direct, Runtime, Mixed |
| `--bs-nav-link-color` | `var(--bs-link-color)` | `.nav`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3801 | scss/_nav.scss:12<br>via scss/_nav.scss:6 | `var(--bs-link-color)` | Direct, Runtime, Mixed |
| `--bs-pagination-color` | `var(--bs-link-color)` | `.pagination`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4719 | scss/_pagination.scss:6<br>via scss/_pagination.scss:1 | `var(--bs-link-color)` | Direct, Runtime, Mixed |

<a id="c-primary-10"></a>
#### $primary – pattern 10: `escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 16 16' fill='tint-color($ANCHOR, 40%)'><path fill-rule='evenodd' d='M1.6…`

2 contexts, 2 properties. Relationship: Compile-time.

Full expression:

```scss
escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 16 16' fill='tint-color($ANCHOR, 40%)'><path fill-rule='evenodd' d='M1.646 4.646a.5.5 0 0 1 .708 0L8 10.293l5.646-5.647a.5.5 0 0 1 .708.708l-6 6a.5.5 0 0 1-.708 0l-6-6a.5.5 0 0 1 0-.708'/></svg>"))
```

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-accordion-btn-active-icon` | `url("data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 16 16' fill='%236ea8fe'%3e%3cpath fill-rule='evenodd' d='M1.646 4.646a.5.5 0 0 1 .708 0L8 10.293l5.646-5.647a.5.5 0 0 1 .708.708l-6 6a.5.5 0 0 1-.708 0l-6-6a.5.5 0 0 1 0-.708'/%3e%3c/svg%3e")` | `[data-bs-theme=dark] .accordion-button::after`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4680 | scss/_accordion.scss:150<br>via scss/_accordion.scss:148 | `escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 16 16' fill='tint-color($primary, 40%)'><path fill-rule='evenodd' d='M1.646 4.646a.5.5 0 0 1 .708 0L8 10.293l5.646-5.647a.5.5 0 0 1 .708.708l-6 6a.5.5 0 0 1-.708 0l-6-6a.5.5 0 0 1 0-.708'/></svg>"))` | Compile-time |
| `--bs-accordion-btn-icon` | `url("data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 16 16' fill='%236ea8fe'%3e%3cpath fill-rule='evenodd' d='M1.646 4.646a.5.5 0 0 1 .708 0L8 10.293l5.646-5.647a.5.5 0 0 1 .708.708l-6 6a.5.5 0 0 1-.708 0l-6-6a.5.5 0 0 1 0-.708'/%3e%3c/svg%3e")` | `[data-bs-theme=dark] .accordion-button::after`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4679 | scss/_accordion.scss:149<br>via scss/_accordion.scss:148 | `escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 16 16' fill='tint-color($primary, 40%)'><path fill-rule='evenodd' d='M1.646 4.646a.5.5 0 0 1 .708 0L8 10.293l5.646-5.647a.5.5 0 0 1 .708.708l-6 6a.5.5 0 0 1-.708 0l-6-6a.5.5 0 0 1 0-.708'/></svg>"))` | Compile-time |

<a id="c-primary-11"></a>
#### $primary – pattern 11: `tint-color($ANCHOR, 40%)`

2 contexts, 2 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-link-color` | `#6ea8fe` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:168<br>src/bootstrap/reference/css/bootstrap.css:169 | scss/_root.scss:169<br>via scss/mixins/_color-mode.scss:16 | `tint-color($primary, 40%)` | Direct |
| `--bs-primary-text-emphasis` | `#6ea8fe` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:143<br>src/bootstrap/reference/css/bootstrap.css:144 | scss/_root.scss:156<br>via scss/mixins/_color-mode.scss:16 | `tint-color($primary, 40%)` | Direct |

<a id="c-primary-12"></a>
#### $primary – pattern 12: `to-rgb(mix(color-contrast($ANCHOR), $ANCHOR, 15%))`

2 contexts, 1 properties. Relationship: Compile-time, Derived, Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-focus-shadow-rgb` | `49, 132, 253` | `.btn-link`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3318 | scss/_buttons.scss:182<br>via scss/_buttons.scss:170 | `to-rgb(mix(color-contrast($primary), $primary, 15%))` | Direct |
| `--bs-btn-focus-shadow-rgb` | `49, 132, 253` | `.btn-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3041 | scss/mixins/_buttons.scss:27<br>via scss/_buttons.scss:132 | `to-rgb(mix(color-contrast($primary), $primary, 15%))` | Derived, Compile-time |

<a id="c-primary-13"></a>
#### $primary – pattern 13: `color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-active-…`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

Full expression:

```scss
color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-active-bg-factor)))
```

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-active-color` | `#000` | `.table-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1949 | scss/mixins/_table-variants.scss:16<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($primary, $table-bg-scale)))), (shift-color($primary, $table-bg-scale)), percentage($table-active-bg-factor)))` | Derived, Compile-time, Mixed |

<a id="c-primary-14"></a>
#### $primary – pattern 14: `color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-hover-b…`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

Full expression:

```scss
color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-hover-bg-factor)))
```

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-hover-color` | `#000` | `.table-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1951 | scss/mixins/_table-variants.scss:18<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($primary, $table-bg-scale)))), (shift-color($primary, $table-bg-scale)), percentage($table-hover-bg-factor)))` | Derived, Compile-time, Mixed |

<a id="c-primary-15"></a>
#### $primary – pattern 15: `color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-striped…`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

Full expression:

```scss
color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-striped-bg-factor)))
```

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-striped-color` | `#000` | `.table-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1947 | scss/mixins/_table-variants.scss:14<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($primary, $table-bg-scale)))), (shift-color($primary, $table-bg-scale)), percentage($table-striped-bg-factor)))` | Derived, Compile-time, Mixed |

<a id="c-primary-16"></a>
#### $primary – pattern 16: `color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale))))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-color` | `#000` | `.table-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1943 | scss/mixins/_table-variants.scss:10<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `color-contrast(opaque($body-bg, (shift-color($primary, $table-bg-scale))))` | Derived, Compile-time, Mixed |

<a id="c-primary-17"></a>
#### $primary – pattern 17: `color-contrast(shade-color($ANCHOR, $btn-active-bg-shade-amount))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-color` | `#fff` | `.btn-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3042 | scss/mixins/_buttons.scss:28<br>via scss/_buttons.scss:132 | `color-contrast(shade-color($primary, $btn-active-bg-shade-amount))` | Derived, Compile-time, Mixed |

<a id="c-primary-18"></a>
#### $primary – pattern 18: `color-contrast(shade-color($ANCHOR, $btn-hover-bg-shade-amount))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-hover-color` | `#fff` | `.btn-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3038 | scss/mixins/_buttons.scss:24<br>via scss/_buttons.scss:132 | `color-contrast(shade-color($primary, $btn-hover-bg-shade-amount))` | Derived, Compile-time, Mixed |

<a id="c-primary-19"></a>
#### $primary – pattern 19: `escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='-4 -4 8 8'><circle r='3' fill='tint-color($ANCHOR, 50%)'/></svg>"))`

1 contexts, 1 properties. Relationship: Compile-time.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-form-switch-bg` | `url("data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='-4 -4 8 8'%3e%3ccircle r='3' fill='%2386b7fe'/%3e%3c/svg%3e")` | `.form-switch .form-check-input:focus`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2470 | scss/forms/_form-check.scss:138<br>via scss/forms/_form-check.scss:137 | `escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='-4 -4 8 8'><circle r='3' fill='tint-color($primary, 50%)'/></svg>"))` | Compile-time |

<a id="c-primary-20"></a>
#### $primary – pattern 20: `escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 16 16' fill='none' stroke='shade-color($ANCHOR, 60%)' stroke-linecap='ro…`

1 contexts, 1 properties. Relationship: Compile-time.

Full expression:

```scss
escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 16 16' fill='none' stroke='shade-color($ANCHOR, 60%)' stroke-linecap='round' stroke-linejoin='round'><path d='m2 5 6 6 6-6'/></svg>"))
```

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-accordion-btn-active-icon` | `url("data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 16 16' fill='none' stroke='%23052c65' stroke-linecap='round' stroke-linejoin='round'%3e%3cpath d='m2 5 6 6 6-6'/%3e%3c/svg%3e")` | `.accordion`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4562 | scss/_accordion.scss:22<br>via scss/_accordion.scss:5 | `escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 16 16' fill='none' stroke='shade-color($primary, 60%)' stroke-linecap='round' stroke-linejoin='round'><path d='m2 5 6 6 6-6'/></svg>"))` | Compile-time |

<a id="c-primary-21"></a>
#### $primary – pattern 21: `mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-active-bg-factor))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-active-bg` | `#bacbe6` | `.table-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1948 | scss/mixins/_table-variants.scss:15<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `mix(color-contrast(opaque($body-bg, (shift-color($primary, $table-bg-scale)))), (shift-color($primary, $table-bg-scale)), percentage($table-active-bg-factor))` | Derived, Compile-time, Mixed |

<a id="c-primary-22"></a>
#### $primary – pattern 22: `mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-border-factor))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-border-color` | `#a6b5cc` | `.table-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1945 | scss/mixins/_table-variants.scss:12<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `mix(color-contrast(opaque($body-bg, (shift-color($primary, $table-bg-scale)))), (shift-color($primary, $table-bg-scale)), percentage($table-border-factor))` | Derived, Compile-time, Mixed |

<a id="c-primary-23"></a>
#### $primary – pattern 23: `mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-hover-bg-factor))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-hover-bg` | `#bfd1ec` | `.table-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1950 | scss/mixins/_table-variants.scss:17<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `mix(color-contrast(opaque($body-bg, (shift-color($primary, $table-bg-scale)))), (shift-color($primary, $table-bg-scale)), percentage($table-hover-bg-factor))` | Derived, Compile-time, Mixed |

<a id="c-primary-24"></a>
#### $primary – pattern 24: `mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-striped-bg-factor))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-striped-bg` | `#c5d7f2` | `.table-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1946 | scss/mixins/_table-variants.scss:13<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `mix(color-contrast(opaque($body-bg, (shift-color($primary, $table-bg-scale)))), (shift-color($primary, $table-bg-scale)), percentage($table-striped-bg-factor))` | Derived, Compile-time, Mixed |

<a id="c-primary-25"></a>
#### $primary – pattern 25: `rgba($ANCHOR, $focus-ring-opacity)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-focus-ring-color` | `rgba(13, 110, 253, 0.25)` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:120<br>src/bootstrap/reference/css/bootstrap.css:121 | scss/_root.scss:121<br>via scss/_root.scss:1 | `rgba($primary, $focus-ring-opacity)` | Direct |

<a id="c-primary-26"></a>
#### $primary – pattern 26: `rgba-css-var("primary", "focus-ring") → rgba(var(--bs-primary-rgb), var(--bs-focus-ring-opacity))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-focus-ring-color` | `rgba(var(--bs-primary-rgb), var(--bs-focus-ring-opacity))` | `.focus-ring-primary`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:742<br>src/bootstrap/reference/css/bootstrap.css:7401 | scss/mixins/_utilities.scss:58<br>via scss/mixins/_utilities.scss:57<br>+ scss/_utilities.scss:92 | `rgba-css-var("primary", "focus-ring") → rgba(var(--bs-primary-rgb), var(--bs-focus-ring-opacity))` | Derived, Runtime, Compile-time, Mixed |

<a id="c-primary-27"></a>
#### $primary – pattern 27: `shade-color($ANCHOR, $btn-active-bg-shade-amount)`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-bg` | `#0a58ca` | `.btn-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3043 | scss/mixins/_buttons.scss:29<br>via scss/_buttons.scss:132 | `shade-color($primary, $btn-active-bg-shade-amount)` | Derived, Compile-time, Mixed |

<a id="c-primary-28"></a>
#### $primary – pattern 28: `shade-color($ANCHOR, $btn-active-border-shade-amount)`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-border-color` | `#0a53be` | `.btn-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3044 | scss/mixins/_buttons.scss:30<br>via scss/_buttons.scss:132 | `shade-color($primary, $btn-active-border-shade-amount)` | Derived, Compile-time, Mixed |

<a id="c-primary-29"></a>
#### $primary – pattern 29: `shade-color($ANCHOR, $btn-hover-bg-shade-amount)`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-hover-bg` | `#0b5ed7` | `.btn-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3039 | scss/mixins/_buttons.scss:25<br>via scss/_buttons.scss:132 | `shade-color($primary, $btn-hover-bg-shade-amount)` | Derived, Compile-time, Mixed |

<a id="c-primary-30"></a>
#### $primary – pattern 30: `shade-color($ANCHOR, $btn-hover-border-shade-amount)`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-hover-border-color` | `#0a58ca` | `.btn-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3040 | scss/mixins/_buttons.scss:26<br>via scss/_buttons.scss:132 | `shade-color($primary, $btn-hover-border-shade-amount)` | Derived, Compile-time, Mixed |

<a id="c-primary-31"></a>
#### $primary – pattern 31: `shade-color($ANCHOR, 40%)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-primary-border-subtle` | `#084298` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:159<br>src/bootstrap/reference/css/bootstrap.css:160 | scss/_root.scss:164<br>via scss/mixins/_color-mode.scss:16 | `shade-color($primary, 40%)` | Direct |

<a id="c-primary-32"></a>
#### $primary – pattern 32: `shade-color($ANCHOR, 60%)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-primary-text-emphasis` | `#052c65` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:47<br>src/bootstrap/reference/css/bootstrap.css:48 | scss/_root.scss:26 | `shade-color($primary, 60%)` | Direct |

<a id="c-primary-33"></a>
#### $primary – pattern 33: `shade-color($ANCHOR, 80%)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-primary-bg-subtle` | `#031633` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:151<br>src/bootstrap/reference/css/bootstrap.css:152 | scss/_root.scss:160<br>via scss/mixins/_color-mode.scss:16 | `shade-color($primary, 80%)` | Direct |

<a id="c-primary-34"></a>
#### $primary – pattern 34: `shift-color($ANCHOR, $link-shade-percentage)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-link-hover-color` | `#0a58ca` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:98<br>src/bootstrap/reference/css/bootstrap.css:99 | scss/_root.scss:86<br>via scss/_root.scss:1 | `shift-color($primary, $link-shade-percentage)` | Direct |

<a id="c-primary-35"></a>
#### $primary – pattern 35: `shift-color($ANCHOR, $table-bg-scale)`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-bg` | `#cfe2ff` | `.table-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1944 | scss/mixins/_table-variants.scss:11<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `shift-color($primary, $table-bg-scale)` | Derived, Compile-time, Mixed |

<a id="c-primary-36"></a>
#### $primary – pattern 36: `shift-color(tint-color($ANCHOR, 40%), -$link-shade-percentage)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-link-hover-color` | `#8bb9fe` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:169<br>src/bootstrap/reference/css/bootstrap.css:170 | scss/_root.scss:170<br>via scss/mixins/_color-mode.scss:16 | `shift-color(tint-color($primary, 40%), -$link-shade-percentage)` | Direct |

<a id="c-primary-37"></a>
#### $primary – pattern 37: `tint-color($ANCHOR, 60%)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-primary-border-subtle` | `#9ec5fe` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:63<br>src/bootstrap/reference/css/bootstrap.css:64 | scss/_root.scss:34 | `tint-color($primary, 60%)` | Direct |

<a id="c-primary-38"></a>
#### $primary – pattern 38: `tint-color($ANCHOR, 80%)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-primary-bg-subtle` | `#cfe2ff` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:55<br>src/bootstrap/reference/css/bootstrap.css:56 | scss/_root.scss:30 | `tint-color($primary, 80%)` | Direct |

<a id="c-primary-39"></a>
#### $primary – pattern 39: `to-rgb(shift-color($ANCHOR, $link-shade-percentage))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-link-hover-color-rgb` | `10, 88, 202` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:99<br>src/bootstrap/reference/css/bootstrap.css:100 | scss/_root.scss:87<br>via scss/_root.scss:1 | `to-rgb(shift-color($primary, $link-shade-percentage))` | Derived, Compile-time, Mixed |

<a id="c-primary-40"></a>
#### $primary – pattern 40: `to-rgb(shift-color(tint-color($ANCHOR, 40%), -$link-shade-percentage))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-link-hover-color-rgb` | `139, 185, 254` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:171<br>src/bootstrap/reference/css/bootstrap.css:172 | scss/_root.scss:172<br>via scss/mixins/_color-mode.scss:16 | `to-rgb(shift-color(tint-color($primary, 40%), -$link-shade-percentage))` | Derived, Compile-time, Mixed |

<a id="c-primary-41"></a>
#### $primary – pattern 41: `to-rgb(tint-color($ANCHOR, 40%))`

1 contexts, 1 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-link-color-rgb` | `110, 168, 254` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:170<br>src/bootstrap/reference/css/bootstrap.css:171 | scss/_root.scss:171<br>via scss/mixins/_color-mode.scss:16 | `to-rgb(tint-color($primary, 40%))` | Derived, Compile-time |

<a id="c-primary-42"></a>
#### $primary – pattern 42: `var(--bs-link-hover-color-rgb)`

1 contexts, 1 properties. Relationship: Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-link-color-rgb` | `var(--bs-link-hover-color-rgb)` | `a:hover`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:355<br>src/bootstrap/reference/css/bootstrap.css:356 | scss/_reboot.scss:249<br>via scss/_reboot.scss:248 | `var(--bs-link-hover-color-rgb)` | Runtime, Mixed |

<a id="c-secondary"></a>
### `$secondary` (theme)

Definition: `$secondary: $gray-600` (`scss/_variables.scss:302`). 54 contexts, 40 properties.

<a id="c-secondary-1"></a>
#### $secondary – pattern 1: `$ANCHOR`

13 contexts, 11 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-bg` | `#6c757d` | `.btn-outline-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3195 | scss/mixins/_buttons.scss:53<br>via scss/_buttons.scss:158 | `$secondary` | Direct |
| `--bs-btn-active-border-color` | `#6c757d` | `.btn-outline-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3196 | scss/mixins/_buttons.scss:54<br>via scss/_buttons.scss:158 | `$secondary` | Direct |
| `--bs-btn-bg` | `#6c757d` | `.btn-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3053 | scss/mixins/_buttons.scss:22<br>via scss/_buttons.scss:132 | `$secondary` | Direct |
| `--bs-btn-border-color` | `#6c757d` | `.btn-outline-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3189 | scss/mixins/_buttons.scss:47<br>via scss/_buttons.scss:158 | `$secondary` | Direct |
| `--bs-btn-border-color` | `#6c757d` | `.btn-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3054 | scss/mixins/_buttons.scss:23<br>via scss/_buttons.scss:132 | `$secondary` | Direct |
| `--bs-btn-color` | `#6c757d` | `.btn-outline-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3188 | scss/mixins/_buttons.scss:46<br>via scss/_buttons.scss:158 | `$secondary` | Direct |
| `--bs-btn-disabled-bg` | `#6c757d` | `.btn-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3064 | scss/mixins/_buttons.scss:33<br>via scss/_buttons.scss:132 | `$secondary` | Direct |
| `--bs-btn-disabled-border-color` | `#6c757d` | `.btn-outline-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3200 | scss/mixins/_buttons.scss:58<br>via scss/_buttons.scss:158 | `$secondary` | Direct |
| `--bs-btn-disabled-border-color` | `#6c757d` | `.btn-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3065 | scss/mixins/_buttons.scss:34<br>via scss/_buttons.scss:132 | `$secondary` | Direct |
| `--bs-btn-disabled-color` | `#6c757d` | `.btn-outline-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3198 | scss/mixins/_buttons.scss:56<br>via scss/_buttons.scss:158 | `$secondary` | Direct |
| `--bs-btn-hover-bg` | `#6c757d` | `.btn-outline-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3191 | scss/mixins/_buttons.scss:49<br>via scss/_buttons.scss:158 | `$secondary` | Direct |
| `--bs-btn-hover-border-color` | `#6c757d` | `.btn-outline-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3192 | scss/mixins/_buttons.scss:50<br>via scss/_buttons.scss:158 | `$secondary` | Direct |
| `--bs-secondary` | `#6c757d` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:32<br>src/bootstrap/reference/css/bootstrap.css:33 | scss/_root.scss:18 | `$secondary` | Direct |

<a id="c-secondary-2"></a>
#### $secondary – pattern 2: `var(--bs-secondary-text-emphasis)`

5 contexts, 5 properties. Relationship: Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-alert-color` | `var(--bs-secondary-text-emphasis)` | `.alert-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4883 | scss/_alert.scss:62<br>via scss/_alert.scss:61 | `var(--bs-secondary-text-emphasis)` | Runtime, Mixed |
| `--bs-alert-link-color` | `var(--bs-secondary-text-emphasis)` | `.alert-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4886 | scss/_alert.scss:65<br>via scss/_alert.scss:61 | `var(--bs-secondary-text-emphasis)` | Runtime, Mixed |
| `--bs-list-group-active-bg` | `var(--bs-secondary-text-emphasis)` | `.list-group-item-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5254 | scss/_list-group.scss:195<br>via scss/_list-group.scss:186 | `var(--bs-secondary-text-emphasis)` | Runtime, Mixed |
| `--bs-list-group-active-border-color` | `var(--bs-secondary-text-emphasis)` | `.list-group-item-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5255 | scss/_list-group.scss:196<br>via scss/_list-group.scss:186 | `var(--bs-secondary-text-emphasis)` | Runtime, Mixed |
| `--bs-list-group-color` | `var(--bs-secondary-text-emphasis)` | `.list-group-item-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5246 | scss/_list-group.scss:187<br>via scss/_list-group.scss:186 | `var(--bs-secondary-text-emphasis)` | Runtime, Mixed |

<a id="c-secondary-3"></a>
#### $secondary – pattern 3: `color-contrast($ANCHOR)`

4 contexts, 4 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-color` | `#fff` | `.btn-outline-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3194 | scss/mixins/_buttons.scss:52<br>via scss/_buttons.scss:158 | `color-contrast($secondary)` | Derived, Compile-time |
| `--bs-btn-color` | `#fff` | `.btn-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3052 | scss/mixins/_buttons.scss:21<br>via scss/_buttons.scss:132 | `color-contrast($secondary)` | Derived, Compile-time |
| `--bs-btn-disabled-color` | `#fff` | `.btn-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3063 | scss/mixins/_buttons.scss:32<br>via scss/_buttons.scss:132 | `color-contrast($secondary)` | Derived, Compile-time |
| `--bs-btn-hover-color` | `#fff` | `.btn-outline-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3190 | scss/mixins/_buttons.scss:48<br>via scss/_buttons.scss:158 | `color-contrast($secondary)` | Derived, Compile-time |

<a id="c-secondary-4"></a>
#### $secondary – pattern 4: `var(--bs-secondary-border-subtle)`

4 contexts, 4 properties. Relationship: Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-alert-border-color` | `var(--bs-secondary-border-subtle)` | `.alert-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4885 | scss/_alert.scss:64<br>via scss/_alert.scss:61 | `var(--bs-secondary-border-subtle)` | Runtime, Mixed |
| `--bs-list-group-action-active-bg` | `var(--bs-secondary-border-subtle)` | `.list-group-item-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5252 | scss/_list-group.scss:193<br>via scss/_list-group.scss:186 | `var(--bs-secondary-border-subtle)` | Runtime, Mixed |
| `--bs-list-group-action-hover-bg` | `var(--bs-secondary-border-subtle)` | `.list-group-item-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5250 | scss/_list-group.scss:191<br>via scss/_list-group.scss:186 | `var(--bs-secondary-border-subtle)` | Runtime, Mixed |
| `--bs-list-group-border-color` | `var(--bs-secondary-border-subtle)` | `.list-group-item-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5248 | scss/_list-group.scss:189<br>via scss/_list-group.scss:186 | `var(--bs-secondary-border-subtle)` | Runtime, Mixed |

<a id="c-secondary-5"></a>
#### $secondary – pattern 5: `var(--bs-secondary-bg-subtle)`

3 contexts, 3 properties. Relationship: Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-alert-bg` | `var(--bs-secondary-bg-subtle)` | `.alert-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4884 | scss/_alert.scss:63<br>via scss/_alert.scss:61 | `var(--bs-secondary-bg-subtle)` | Runtime, Mixed |
| `--bs-list-group-active-color` | `var(--bs-secondary-bg-subtle)` | `.list-group-item-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5253 | scss/_list-group.scss:194<br>via scss/_list-group.scss:186 | `var(--bs-secondary-bg-subtle)` | Runtime, Mixed |
| `--bs-list-group-bg` | `var(--bs-secondary-bg-subtle)` | `.list-group-item-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5247 | scss/_list-group.scss:188<br>via scss/_list-group.scss:186 | `var(--bs-secondary-bg-subtle)` | Runtime, Mixed |

<a id="c-secondary-6"></a>
#### $secondary – pattern 6: `to-rgb($ANCHOR)`

2 contexts, 2 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-focus-shadow-rgb` | `108, 117, 125` | `.btn-outline-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3193 | scss/mixins/_buttons.scss:51<br>via scss/_buttons.scss:158 | `to-rgb($secondary)` | Derived, Compile-time |
| `--bs-secondary-rgb` | `108, 117, 125` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:40<br>src/bootstrap/reference/css/bootstrap.css:41 | scss/_root.scss:22 | `to-rgb($secondary)` | Derived, Compile-time |

<a id="c-secondary-7"></a>
#### $secondary – pattern 7: `color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-active-…`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

Full expression:

```scss
color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-active-bg-factor)))
```

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-active-color` | `#000` | `.table-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1963 | scss/mixins/_table-variants.scss:16<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($secondary, $table-bg-scale)))), (shift-color($secondary, $table-bg-scale)), percentage($table-active-bg-factor)))` | Derived, Compile-time, Mixed |

<a id="c-secondary-8"></a>
#### $secondary – pattern 8: `color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-hover-b…`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

Full expression:

```scss
color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-hover-bg-factor)))
```

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-hover-color` | `#000` | `.table-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1965 | scss/mixins/_table-variants.scss:18<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($secondary, $table-bg-scale)))), (shift-color($secondary, $table-bg-scale)), percentage($table-hover-bg-factor)))` | Derived, Compile-time, Mixed |

<a id="c-secondary-9"></a>
#### $secondary – pattern 9: `color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-striped…`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

Full expression:

```scss
color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-striped-bg-factor)))
```

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-striped-color` | `#000` | `.table-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1961 | scss/mixins/_table-variants.scss:14<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($secondary, $table-bg-scale)))), (shift-color($secondary, $table-bg-scale)), percentage($table-striped-bg-factor)))` | Derived, Compile-time, Mixed |

<a id="c-secondary-10"></a>
#### $secondary – pattern 10: `color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale))))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-color` | `#000` | `.table-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1957 | scss/mixins/_table-variants.scss:10<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `color-contrast(opaque($body-bg, (shift-color($secondary, $table-bg-scale))))` | Derived, Compile-time, Mixed |

<a id="c-secondary-11"></a>
#### $secondary – pattern 11: `color-contrast(shade-color($ANCHOR, $btn-active-bg-shade-amount))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-color` | `#fff` | `.btn-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3059 | scss/mixins/_buttons.scss:28<br>via scss/_buttons.scss:132 | `color-contrast(shade-color($secondary, $btn-active-bg-shade-amount))` | Derived, Compile-time, Mixed |

<a id="c-secondary-12"></a>
#### $secondary – pattern 12: `color-contrast(shade-color($ANCHOR, $btn-hover-bg-shade-amount))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-hover-color` | `#fff` | `.btn-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3055 | scss/mixins/_buttons.scss:24<br>via scss/_buttons.scss:132 | `color-contrast(shade-color($secondary, $btn-hover-bg-shade-amount))` | Derived, Compile-time, Mixed |

<a id="c-secondary-13"></a>
#### $secondary – pattern 13: `mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-active-bg-factor))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-active-bg` | `#cbccce` | `.table-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1962 | scss/mixins/_table-variants.scss:15<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `mix(color-contrast(opaque($body-bg, (shift-color($secondary, $table-bg-scale)))), (shift-color($secondary, $table-bg-scale)), percentage($table-active-bg-factor))` | Derived, Compile-time, Mixed |

<a id="c-secondary-14"></a>
#### $secondary – pattern 14: `mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-border-factor))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-border-color` | `#b5b6b7` | `.table-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1959 | scss/mixins/_table-variants.scss:12<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `mix(color-contrast(opaque($body-bg, (shift-color($secondary, $table-bg-scale)))), (shift-color($secondary, $table-bg-scale)), percentage($table-border-factor))` | Derived, Compile-time, Mixed |

<a id="c-secondary-15"></a>
#### $secondary – pattern 15: `mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-hover-bg-factor))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-hover-bg` | `#d1d2d4` | `.table-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1964 | scss/mixins/_table-variants.scss:17<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `mix(color-contrast(opaque($body-bg, (shift-color($secondary, $table-bg-scale)))), (shift-color($secondary, $table-bg-scale)), percentage($table-hover-bg-factor))` | Derived, Compile-time, Mixed |

<a id="c-secondary-16"></a>
#### $secondary – pattern 16: `mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-striped-bg-factor))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-striped-bg` | `#d7d8da` | `.table-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1960 | scss/mixins/_table-variants.scss:13<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `mix(color-contrast(opaque($body-bg, (shift-color($secondary, $table-bg-scale)))), (shift-color($secondary, $table-bg-scale)), percentage($table-striped-bg-factor))` | Derived, Compile-time, Mixed |

<a id="c-secondary-17"></a>
#### $secondary – pattern 17: `rgba-css-var("secondary", "focus-ring") → rgba(var(--bs-secondary-rgb), var(--bs-focus-ring-opacity))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-focus-ring-color` | `rgba(var(--bs-secondary-rgb), var(--bs-focus-ring-opacity))` | `.focus-ring-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:746<br>src/bootstrap/reference/css/bootstrap.css:7405 | scss/mixins/_utilities.scss:58<br>via scss/mixins/_utilities.scss:57<br>+ scss/_utilities.scss:92 | `rgba-css-var("secondary", "focus-ring") → rgba(var(--bs-secondary-rgb), var(--bs-focus-ring-opacity))` | Derived, Runtime, Compile-time, Mixed |

<a id="c-secondary-18"></a>
#### $secondary – pattern 18: `shade-color($ANCHOR, $btn-active-bg-shade-amount)`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-bg` | `#565e64` | `.btn-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3060 | scss/mixins/_buttons.scss:29<br>via scss/_buttons.scss:132 | `shade-color($secondary, $btn-active-bg-shade-amount)` | Derived, Compile-time, Mixed |

<a id="c-secondary-19"></a>
#### $secondary – pattern 19: `shade-color($ANCHOR, $btn-active-border-shade-amount)`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-border-color` | `#51585e` | `.btn-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3061 | scss/mixins/_buttons.scss:30<br>via scss/_buttons.scss:132 | `shade-color($secondary, $btn-active-border-shade-amount)` | Derived, Compile-time, Mixed |

<a id="c-secondary-20"></a>
#### $secondary – pattern 20: `shade-color($ANCHOR, $btn-hover-bg-shade-amount)`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-hover-bg` | `#5c636a` | `.btn-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3056 | scss/mixins/_buttons.scss:25<br>via scss/_buttons.scss:132 | `shade-color($secondary, $btn-hover-bg-shade-amount)` | Derived, Compile-time, Mixed |

<a id="c-secondary-21"></a>
#### $secondary – pattern 21: `shade-color($ANCHOR, $btn-hover-border-shade-amount)`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-hover-border-color` | `#565e64` | `.btn-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3057 | scss/mixins/_buttons.scss:26<br>via scss/_buttons.scss:132 | `shade-color($secondary, $btn-hover-border-shade-amount)` | Derived, Compile-time, Mixed |

<a id="c-secondary-22"></a>
#### $secondary – pattern 22: `shade-color($ANCHOR, 40%)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-secondary-border-subtle` | `#41464b` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:160<br>src/bootstrap/reference/css/bootstrap.css:161 | scss/_root.scss:164<br>via scss/mixins/_color-mode.scss:16 | `shade-color($secondary, 40%)` | Direct |

<a id="c-secondary-23"></a>
#### $secondary – pattern 23: `shade-color($ANCHOR, 60%)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-secondary-text-emphasis` | `#2b2f32` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:48<br>src/bootstrap/reference/css/bootstrap.css:49 | scss/_root.scss:26 | `shade-color($secondary, 60%)` | Direct |

<a id="c-secondary-24"></a>
#### $secondary – pattern 24: `shade-color($ANCHOR, 80%)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-secondary-bg-subtle` | `#161719` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:152<br>src/bootstrap/reference/css/bootstrap.css:153 | scss/_root.scss:160<br>via scss/mixins/_color-mode.scss:16 | `shade-color($secondary, 80%)` | Direct |

<a id="c-secondary-25"></a>
#### $secondary – pattern 25: `shift-color($ANCHOR, $table-bg-scale)`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-bg` | `#e2e3e5` | `.table-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1958 | scss/mixins/_table-variants.scss:11<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `shift-color($secondary, $table-bg-scale)` | Derived, Compile-time, Mixed |

<a id="c-secondary-26"></a>
#### $secondary – pattern 26: `tint-color($ANCHOR, 40%)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-secondary-text-emphasis` | `#a7acb1` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:144<br>src/bootstrap/reference/css/bootstrap.css:145 | scss/_root.scss:156<br>via scss/mixins/_color-mode.scss:16 | `tint-color($secondary, 40%)` | Direct |

<a id="c-secondary-27"></a>
#### $secondary – pattern 27: `tint-color($ANCHOR, 60%)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-secondary-border-subtle` | `#c4c8cb` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:64<br>src/bootstrap/reference/css/bootstrap.css:65 | scss/_root.scss:34 | `tint-color($secondary, 60%)` | Direct |

<a id="c-secondary-28"></a>
#### $secondary – pattern 28: `tint-color($ANCHOR, 80%)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-secondary-bg-subtle` | `#e2e3e5` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:56<br>src/bootstrap/reference/css/bootstrap.css:57 | scss/_root.scss:30 | `tint-color($secondary, 80%)` | Direct |

<a id="c-secondary-29"></a>
#### $secondary – pattern 29: `to-rgb(mix(color-contrast($ANCHOR), $ANCHOR, 15%))`

1 contexts, 1 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-focus-shadow-rgb` | `130, 138, 145` | `.btn-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3058 | scss/mixins/_buttons.scss:27<br>via scss/_buttons.scss:132 | `to-rgb(mix(color-contrast($secondary), $secondary, 15%))` | Derived, Compile-time |

<a id="c-success"></a>
### `$success` (theme)

Definition: `$success: $green` (`scss/_variables.scss:303`). 57 contexts, 43 properties.

<a id="c-success-1"></a>
#### $success – pattern 1: `$ANCHOR`

15 contexts, 13 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-bg` | `#198754` | `.btn-outline-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3212 | scss/mixins/_buttons.scss:53<br>via scss/_buttons.scss:158 | `$success` | Direct |
| `--bs-btn-active-border-color` | `#198754` | `.btn-outline-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3213 | scss/mixins/_buttons.scss:54<br>via scss/_buttons.scss:158 | `$success` | Direct |
| `--bs-btn-bg` | `#198754` | `.btn-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3070 | scss/mixins/_buttons.scss:22<br>via scss/_buttons.scss:132 | `$success` | Direct |
| `--bs-btn-border-color` | `#198754` | `.btn-outline-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3206 | scss/mixins/_buttons.scss:47<br>via scss/_buttons.scss:158 | `$success` | Direct |
| `--bs-btn-border-color` | `#198754` | `.btn-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3071 | scss/mixins/_buttons.scss:23<br>via scss/_buttons.scss:132 | `$success` | Direct |
| `--bs-btn-color` | `#198754` | `.btn-outline-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3205 | scss/mixins/_buttons.scss:46<br>via scss/_buttons.scss:158 | `$success` | Direct |
| `--bs-btn-disabled-bg` | `#198754` | `.btn-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3081 | scss/mixins/_buttons.scss:33<br>via scss/_buttons.scss:132 | `$success` | Direct |
| `--bs-btn-disabled-border-color` | `#198754` | `.btn-outline-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3217 | scss/mixins/_buttons.scss:58<br>via scss/_buttons.scss:158 | `$success` | Direct |
| `--bs-btn-disabled-border-color` | `#198754` | `.btn-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3082 | scss/mixins/_buttons.scss:34<br>via scss/_buttons.scss:132 | `$success` | Direct |
| `--bs-btn-disabled-color` | `#198754` | `.btn-outline-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3215 | scss/mixins/_buttons.scss:56<br>via scss/_buttons.scss:158 | `$success` | Direct |
| `--bs-btn-hover-bg` | `#198754` | `.btn-outline-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3208 | scss/mixins/_buttons.scss:49<br>via scss/_buttons.scss:158 | `$success` | Direct |
| `--bs-btn-hover-border-color` | `#198754` | `.btn-outline-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3209 | scss/mixins/_buttons.scss:50<br>via scss/_buttons.scss:158 | `$success` | Direct |
| `--bs-form-valid-border-color` | `#198754` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:122<br>src/bootstrap/reference/css/bootstrap.css:123 | scss/_root.scss:126<br>via scss/_root.scss:1 | `$success` | Direct |
| `--bs-form-valid-color` | `#198754` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:121<br>src/bootstrap/reference/css/bootstrap.css:122 | scss/_root.scss:125<br>via scss/_root.scss:1 | `$success` | Direct |
| `--bs-success` | `#198754` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:33<br>src/bootstrap/reference/css/bootstrap.css:34 | scss/_root.scss:18 | `$success` | Direct |

<a id="c-success-2"></a>
#### $success – pattern 2: `var(--bs-success-text-emphasis)`

5 contexts, 5 properties. Relationship: Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-alert-color` | `var(--bs-success-text-emphasis)` | `.alert-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4890 | scss/_alert.scss:62<br>via scss/_alert.scss:61 | `var(--bs-success-text-emphasis)` | Runtime, Mixed |
| `--bs-alert-link-color` | `var(--bs-success-text-emphasis)` | `.alert-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4893 | scss/_alert.scss:65<br>via scss/_alert.scss:61 | `var(--bs-success-text-emphasis)` | Runtime, Mixed |
| `--bs-list-group-active-bg` | `var(--bs-success-text-emphasis)` | `.list-group-item-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5267 | scss/_list-group.scss:195<br>via scss/_list-group.scss:186 | `var(--bs-success-text-emphasis)` | Runtime, Mixed |
| `--bs-list-group-active-border-color` | `var(--bs-success-text-emphasis)` | `.list-group-item-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5268 | scss/_list-group.scss:196<br>via scss/_list-group.scss:186 | `var(--bs-success-text-emphasis)` | Runtime, Mixed |
| `--bs-list-group-color` | `var(--bs-success-text-emphasis)` | `.list-group-item-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5259 | scss/_list-group.scss:187<br>via scss/_list-group.scss:186 | `var(--bs-success-text-emphasis)` | Runtime, Mixed |

<a id="c-success-3"></a>
#### $success – pattern 3: `color-contrast($ANCHOR)`

4 contexts, 4 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-color` | `#fff` | `.btn-outline-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3211 | scss/mixins/_buttons.scss:52<br>via scss/_buttons.scss:158 | `color-contrast($success)` | Derived, Compile-time |
| `--bs-btn-color` | `#fff` | `.btn-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3069 | scss/mixins/_buttons.scss:21<br>via scss/_buttons.scss:132 | `color-contrast($success)` | Derived, Compile-time |
| `--bs-btn-disabled-color` | `#fff` | `.btn-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3080 | scss/mixins/_buttons.scss:32<br>via scss/_buttons.scss:132 | `color-contrast($success)` | Derived, Compile-time |
| `--bs-btn-hover-color` | `#fff` | `.btn-outline-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3207 | scss/mixins/_buttons.scss:48<br>via scss/_buttons.scss:158 | `color-contrast($success)` | Derived, Compile-time |

<a id="c-success-4"></a>
#### $success – pattern 4: `var(--bs-success-border-subtle)`

4 contexts, 4 properties. Relationship: Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-alert-border-color` | `var(--bs-success-border-subtle)` | `.alert-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4892 | scss/_alert.scss:64<br>via scss/_alert.scss:61 | `var(--bs-success-border-subtle)` | Runtime, Mixed |
| `--bs-list-group-action-active-bg` | `var(--bs-success-border-subtle)` | `.list-group-item-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5265 | scss/_list-group.scss:193<br>via scss/_list-group.scss:186 | `var(--bs-success-border-subtle)` | Runtime, Mixed |
| `--bs-list-group-action-hover-bg` | `var(--bs-success-border-subtle)` | `.list-group-item-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5263 | scss/_list-group.scss:191<br>via scss/_list-group.scss:186 | `var(--bs-success-border-subtle)` | Runtime, Mixed |
| `--bs-list-group-border-color` | `var(--bs-success-border-subtle)` | `.list-group-item-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5261 | scss/_list-group.scss:189<br>via scss/_list-group.scss:186 | `var(--bs-success-border-subtle)` | Runtime, Mixed |

<a id="c-success-5"></a>
#### $success – pattern 5: `var(--bs-success-bg-subtle)`

3 contexts, 3 properties. Relationship: Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-alert-bg` | `var(--bs-success-bg-subtle)` | `.alert-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4891 | scss/_alert.scss:63<br>via scss/_alert.scss:61 | `var(--bs-success-bg-subtle)` | Runtime, Mixed |
| `--bs-list-group-active-color` | `var(--bs-success-bg-subtle)` | `.list-group-item-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5266 | scss/_list-group.scss:194<br>via scss/_list-group.scss:186 | `var(--bs-success-bg-subtle)` | Runtime, Mixed |
| `--bs-list-group-bg` | `var(--bs-success-bg-subtle)` | `.list-group-item-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5260 | scss/_list-group.scss:188<br>via scss/_list-group.scss:186 | `var(--bs-success-bg-subtle)` | Runtime, Mixed |

<a id="c-success-6"></a>
#### $success – pattern 6: `to-rgb($ANCHOR)`

2 contexts, 2 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-focus-shadow-rgb` | `25, 135, 84` | `.btn-outline-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3210 | scss/mixins/_buttons.scss:51<br>via scss/_buttons.scss:158 | `to-rgb($success)` | Derived, Compile-time |
| `--bs-success-rgb` | `25, 135, 84` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:41<br>src/bootstrap/reference/css/bootstrap.css:42 | scss/_root.scss:22 | `to-rgb($success)` | Derived, Compile-time |

<a id="c-success-7"></a>
#### $success – pattern 7: `color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-active-…`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

Full expression:

```scss
color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-active-bg-factor)))
```

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-active-color` | `#000` | `.table-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1977 | scss/mixins/_table-variants.scss:16<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($success, $table-bg-scale)))), (shift-color($success, $table-bg-scale)), percentage($table-active-bg-factor)))` | Derived, Compile-time, Mixed |

<a id="c-success-8"></a>
#### $success – pattern 8: `color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-hover-b…`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

Full expression:

```scss
color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-hover-bg-factor)))
```

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-hover-color` | `#000` | `.table-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1979 | scss/mixins/_table-variants.scss:18<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($success, $table-bg-scale)))), (shift-color($success, $table-bg-scale)), percentage($table-hover-bg-factor)))` | Derived, Compile-time, Mixed |

<a id="c-success-9"></a>
#### $success – pattern 9: `color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-striped…`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

Full expression:

```scss
color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-striped-bg-factor)))
```

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-striped-color` | `#000` | `.table-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1975 | scss/mixins/_table-variants.scss:14<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($success, $table-bg-scale)))), (shift-color($success, $table-bg-scale)), percentage($table-striped-bg-factor)))` | Derived, Compile-time, Mixed |

<a id="c-success-10"></a>
#### $success – pattern 10: `color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale))))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-color` | `#000` | `.table-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1971 | scss/mixins/_table-variants.scss:10<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `color-contrast(opaque($body-bg, (shift-color($success, $table-bg-scale))))` | Derived, Compile-time, Mixed |

<a id="c-success-11"></a>
#### $success – pattern 11: `color-contrast(shade-color($ANCHOR, $btn-active-bg-shade-amount))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-color` | `#fff` | `.btn-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3076 | scss/mixins/_buttons.scss:28<br>via scss/_buttons.scss:132 | `color-contrast(shade-color($success, $btn-active-bg-shade-amount))` | Derived, Compile-time, Mixed |

<a id="c-success-12"></a>
#### $success – pattern 12: `color-contrast(shade-color($ANCHOR, $btn-hover-bg-shade-amount))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-hover-color` | `#fff` | `.btn-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3072 | scss/mixins/_buttons.scss:24<br>via scss/_buttons.scss:132 | `color-contrast(shade-color($success, $btn-hover-bg-shade-amount))` | Derived, Compile-time, Mixed |

<a id="c-success-13"></a>
#### $success – pattern 13: `escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 8 8'><path fill='$ANCHOR' d='M2.3 6.73.6 4.53c-.4-1.04.46-1.4 1.1-.8l1.1…`

1 contexts, 1 properties. Relationship: Compile-time.

Full expression:

```scss
escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 8 8'><path fill='$ANCHOR' d='M2.3 6.73.6 4.53c-.4-1.04.46-1.4 1.1-.8l1.1 1.4 3.4-3.8c.6-.63 1.6-.27 1.2.7l-4 4.6c-.43.5-.8.4-1.1.1'/></svg>"))
```

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-form-select-bg-icon` | `url("data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 8 8'%3e%3cpath fill='%23198754' d='M2.3 6.73.6 4.53c-.4-1.04.46-1.4 1.1-.8l1.1 1.4 3.4-3.8c.6-.63 1.6-.27 1.2.7l-4 4.6c-.43.5-.8.4-1.1.1'/%3e%3c/svg%3e")` | `.was-validated .form-select:valid:not([multiple]):not([size]), .was-validated .form-select:valid:not([multiple])[size="1…`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2824 | scss/mixins/_forms.scss:99<br>via scss/mixins/_forms.scss:97<br>+ scss/forms/_validation.scss:10 | `escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 8 8'><path fill='$success' d='M2.3 6.73.6 4.53c-.4-1.04.46-1.4 1.1-.8l1.1 1.4 3.4-3.8c.6-.63 1.6-.27 1.2.7l-4 4.6c-.43.5-.8.4-1.1.1'/></svg>"))` | Compile-time |

<a id="c-success-14"></a>
#### $success – pattern 14: `mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-active-bg-factor))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-active-bg` | `#bcd0c7` | `.table-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1976 | scss/mixins/_table-variants.scss:15<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `mix(color-contrast(opaque($body-bg, (shift-color($success, $table-bg-scale)))), (shift-color($success, $table-bg-scale)), percentage($table-active-bg-factor))` | Derived, Compile-time, Mixed |

<a id="c-success-15"></a>
#### $success – pattern 15: `mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-border-factor))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-border-color` | `#a7b9b1` | `.table-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1973 | scss/mixins/_table-variants.scss:12<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `mix(color-contrast(opaque($body-bg, (shift-color($success, $table-bg-scale)))), (shift-color($success, $table-bg-scale)), percentage($table-border-factor))` | Derived, Compile-time, Mixed |

<a id="c-success-16"></a>
#### $success – pattern 16: `mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-hover-bg-factor))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-hover-bg` | `#c1d6cc` | `.table-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1978 | scss/mixins/_table-variants.scss:17<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `mix(color-contrast(opaque($body-bg, (shift-color($success, $table-bg-scale)))), (shift-color($success, $table-bg-scale)), percentage($table-hover-bg-factor))` | Derived, Compile-time, Mixed |

<a id="c-success-17"></a>
#### $success – pattern 17: `mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-striped-bg-factor))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-striped-bg` | `#c7dbd2` | `.table-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1974 | scss/mixins/_table-variants.scss:13<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `mix(color-contrast(opaque($body-bg, (shift-color($success, $table-bg-scale)))), (shift-color($success, $table-bg-scale)), percentage($table-striped-bg-factor))` | Derived, Compile-time, Mixed |

<a id="c-success-18"></a>
#### $success – pattern 18: `rgba-css-var("success", "focus-ring") → rgba(var(--bs-success-rgb), var(--bs-focus-ring-opacity))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-focus-ring-color` | `rgba(var(--bs-success-rgb), var(--bs-focus-ring-opacity))` | `.focus-ring-success`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:750<br>src/bootstrap/reference/css/bootstrap.css:7409 | scss/mixins/_utilities.scss:58<br>via scss/mixins/_utilities.scss:57<br>+ scss/_utilities.scss:92 | `rgba-css-var("success", "focus-ring") → rgba(var(--bs-success-rgb), var(--bs-focus-ring-opacity))` | Derived, Runtime, Compile-time, Mixed |

<a id="c-success-19"></a>
#### $success – pattern 19: `shade-color($ANCHOR, $btn-active-bg-shade-amount)`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-bg` | `#146c43` | `.btn-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3077 | scss/mixins/_buttons.scss:29<br>via scss/_buttons.scss:132 | `shade-color($success, $btn-active-bg-shade-amount)` | Derived, Compile-time, Mixed |

<a id="c-success-20"></a>
#### $success – pattern 20: `shade-color($ANCHOR, $btn-active-border-shade-amount)`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-border-color` | `#13653f` | `.btn-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3078 | scss/mixins/_buttons.scss:30<br>via scss/_buttons.scss:132 | `shade-color($success, $btn-active-border-shade-amount)` | Derived, Compile-time, Mixed |

<a id="c-success-21"></a>
#### $success – pattern 21: `shade-color($ANCHOR, $btn-hover-bg-shade-amount)`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-hover-bg` | `#157347` | `.btn-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3073 | scss/mixins/_buttons.scss:25<br>via scss/_buttons.scss:132 | `shade-color($success, $btn-hover-bg-shade-amount)` | Derived, Compile-time, Mixed |

<a id="c-success-22"></a>
#### $success – pattern 22: `shade-color($ANCHOR, $btn-hover-border-shade-amount)`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-hover-border-color` | `#146c43` | `.btn-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3074 | scss/mixins/_buttons.scss:26<br>via scss/_buttons.scss:132 | `shade-color($success, $btn-hover-border-shade-amount)` | Derived, Compile-time, Mixed |

<a id="c-success-23"></a>
#### $success – pattern 23: `shade-color($ANCHOR, 40%)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-success-border-subtle` | `#0f5132` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:161<br>src/bootstrap/reference/css/bootstrap.css:162 | scss/_root.scss:164<br>via scss/mixins/_color-mode.scss:16 | `shade-color($success, 40%)` | Direct |

<a id="c-success-24"></a>
#### $success – pattern 24: `shade-color($ANCHOR, 60%)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-success-text-emphasis` | `#0a3622` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:49<br>src/bootstrap/reference/css/bootstrap.css:50 | scss/_root.scss:26 | `shade-color($success, 60%)` | Direct |

<a id="c-success-25"></a>
#### $success – pattern 25: `shade-color($ANCHOR, 80%)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-success-bg-subtle` | `#051b11` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:153<br>src/bootstrap/reference/css/bootstrap.css:154 | scss/_root.scss:160<br>via scss/mixins/_color-mode.scss:16 | `shade-color($success, 80%)` | Direct |

<a id="c-success-26"></a>
#### $success – pattern 26: `shift-color($ANCHOR, $table-bg-scale)`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-bg` | `#d1e7dd` | `.table-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1972 | scss/mixins/_table-variants.scss:11<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `shift-color($success, $table-bg-scale)` | Derived, Compile-time, Mixed |

<a id="c-success-27"></a>
#### $success – pattern 27: `tint-color($ANCHOR, 40%)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-success-text-emphasis` | `#75b798` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:145<br>src/bootstrap/reference/css/bootstrap.css:146 | scss/_root.scss:156<br>via scss/mixins/_color-mode.scss:16 | `tint-color($success, 40%)` | Direct |

<a id="c-success-28"></a>
#### $success – pattern 28: `tint-color($ANCHOR, 60%)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-success-border-subtle` | `#a3cfbb` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:65<br>src/bootstrap/reference/css/bootstrap.css:66 | scss/_root.scss:34 | `tint-color($success, 60%)` | Direct |

<a id="c-success-29"></a>
#### $success – pattern 29: `tint-color($ANCHOR, 80%)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-success-bg-subtle` | `#d1e7dd` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:57<br>src/bootstrap/reference/css/bootstrap.css:58 | scss/_root.scss:30 | `tint-color($success, 80%)` | Direct |

<a id="c-success-30"></a>
#### $success – pattern 30: `to-rgb(mix(color-contrast($ANCHOR), $ANCHOR, 15%))`

1 contexts, 1 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-focus-shadow-rgb` | `60, 153, 110` | `.btn-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3075 | scss/mixins/_buttons.scss:27<br>via scss/_buttons.scss:132 | `to-rgb(mix(color-contrast($success), $success, 15%))` | Derived, Compile-time |

<a id="c-info"></a>
### `$info` (theme)

Definition: `$info: $cyan` (`scss/_variables.scss:304`). 54 contexts, 40 properties.

<a id="c-info-1"></a>
#### $info – pattern 1: `$ANCHOR`

13 contexts, 11 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-bg` | `#0dcaf0` | `.btn-outline-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3229 | scss/mixins/_buttons.scss:53<br>via scss/_buttons.scss:158 | `$info` | Direct |
| `--bs-btn-active-border-color` | `#0dcaf0` | `.btn-outline-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3230 | scss/mixins/_buttons.scss:54<br>via scss/_buttons.scss:158 | `$info` | Direct |
| `--bs-btn-bg` | `#0dcaf0` | `.btn-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3087 | scss/mixins/_buttons.scss:22<br>via scss/_buttons.scss:132 | `$info` | Direct |
| `--bs-btn-border-color` | `#0dcaf0` | `.btn-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3088 | scss/mixins/_buttons.scss:23<br>via scss/_buttons.scss:132 | `$info` | Direct |
| `--bs-btn-border-color` | `#0dcaf0` | `.btn-outline-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3223 | scss/mixins/_buttons.scss:47<br>via scss/_buttons.scss:158 | `$info` | Direct |
| `--bs-btn-color` | `#0dcaf0` | `.btn-outline-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3222 | scss/mixins/_buttons.scss:46<br>via scss/_buttons.scss:158 | `$info` | Direct |
| `--bs-btn-disabled-bg` | `#0dcaf0` | `.btn-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3098 | scss/mixins/_buttons.scss:33<br>via scss/_buttons.scss:132 | `$info` | Direct |
| `--bs-btn-disabled-border-color` | `#0dcaf0` | `.btn-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3099 | scss/mixins/_buttons.scss:34<br>via scss/_buttons.scss:132 | `$info` | Direct |
| `--bs-btn-disabled-border-color` | `#0dcaf0` | `.btn-outline-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3234 | scss/mixins/_buttons.scss:58<br>via scss/_buttons.scss:158 | `$info` | Direct |
| `--bs-btn-disabled-color` | `#0dcaf0` | `.btn-outline-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3232 | scss/mixins/_buttons.scss:56<br>via scss/_buttons.scss:158 | `$info` | Direct |
| `--bs-btn-hover-bg` | `#0dcaf0` | `.btn-outline-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3225 | scss/mixins/_buttons.scss:49<br>via scss/_buttons.scss:158 | `$info` | Direct |
| `--bs-btn-hover-border-color` | `#0dcaf0` | `.btn-outline-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3226 | scss/mixins/_buttons.scss:50<br>via scss/_buttons.scss:158 | `$info` | Direct |
| `--bs-info` | `#0dcaf0` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:34<br>src/bootstrap/reference/css/bootstrap.css:35 | scss/_root.scss:18 | `$info` | Direct |

<a id="c-info-2"></a>
#### $info – pattern 2: `var(--bs-info-text-emphasis)`

5 contexts, 5 properties. Relationship: Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-alert-color` | `var(--bs-info-text-emphasis)` | `.alert-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4897 | scss/_alert.scss:62<br>via scss/_alert.scss:61 | `var(--bs-info-text-emphasis)` | Runtime, Mixed |
| `--bs-alert-link-color` | `var(--bs-info-text-emphasis)` | `.alert-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4900 | scss/_alert.scss:65<br>via scss/_alert.scss:61 | `var(--bs-info-text-emphasis)` | Runtime, Mixed |
| `--bs-list-group-active-bg` | `var(--bs-info-text-emphasis)` | `.list-group-item-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5280 | scss/_list-group.scss:195<br>via scss/_list-group.scss:186 | `var(--bs-info-text-emphasis)` | Runtime, Mixed |
| `--bs-list-group-active-border-color` | `var(--bs-info-text-emphasis)` | `.list-group-item-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5281 | scss/_list-group.scss:196<br>via scss/_list-group.scss:186 | `var(--bs-info-text-emphasis)` | Runtime, Mixed |
| `--bs-list-group-color` | `var(--bs-info-text-emphasis)` | `.list-group-item-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5272 | scss/_list-group.scss:187<br>via scss/_list-group.scss:186 | `var(--bs-info-text-emphasis)` | Runtime, Mixed |

<a id="c-info-3"></a>
#### $info – pattern 3: `color-contrast($ANCHOR)`

4 contexts, 4 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-color` | `#000` | `.btn-outline-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3228 | scss/mixins/_buttons.scss:52<br>via scss/_buttons.scss:158 | `color-contrast($info)` | Derived, Compile-time |
| `--bs-btn-color` | `#000` | `.btn-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3086 | scss/mixins/_buttons.scss:21<br>via scss/_buttons.scss:132 | `color-contrast($info)` | Derived, Compile-time |
| `--bs-btn-disabled-color` | `#000` | `.btn-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3097 | scss/mixins/_buttons.scss:32<br>via scss/_buttons.scss:132 | `color-contrast($info)` | Derived, Compile-time |
| `--bs-btn-hover-color` | `#000` | `.btn-outline-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3224 | scss/mixins/_buttons.scss:48<br>via scss/_buttons.scss:158 | `color-contrast($info)` | Derived, Compile-time |

<a id="c-info-4"></a>
#### $info – pattern 4: `var(--bs-info-border-subtle)`

4 contexts, 4 properties. Relationship: Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-alert-border-color` | `var(--bs-info-border-subtle)` | `.alert-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4899 | scss/_alert.scss:64<br>via scss/_alert.scss:61 | `var(--bs-info-border-subtle)` | Runtime, Mixed |
| `--bs-list-group-action-active-bg` | `var(--bs-info-border-subtle)` | `.list-group-item-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5278 | scss/_list-group.scss:193<br>via scss/_list-group.scss:186 | `var(--bs-info-border-subtle)` | Runtime, Mixed |
| `--bs-list-group-action-hover-bg` | `var(--bs-info-border-subtle)` | `.list-group-item-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5276 | scss/_list-group.scss:191<br>via scss/_list-group.scss:186 | `var(--bs-info-border-subtle)` | Runtime, Mixed |
| `--bs-list-group-border-color` | `var(--bs-info-border-subtle)` | `.list-group-item-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5274 | scss/_list-group.scss:189<br>via scss/_list-group.scss:186 | `var(--bs-info-border-subtle)` | Runtime, Mixed |

<a id="c-info-5"></a>
#### $info – pattern 5: `var(--bs-info-bg-subtle)`

3 contexts, 3 properties. Relationship: Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-alert-bg` | `var(--bs-info-bg-subtle)` | `.alert-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4898 | scss/_alert.scss:63<br>via scss/_alert.scss:61 | `var(--bs-info-bg-subtle)` | Runtime, Mixed |
| `--bs-list-group-active-color` | `var(--bs-info-bg-subtle)` | `.list-group-item-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5279 | scss/_list-group.scss:194<br>via scss/_list-group.scss:186 | `var(--bs-info-bg-subtle)` | Runtime, Mixed |
| `--bs-list-group-bg` | `var(--bs-info-bg-subtle)` | `.list-group-item-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5273 | scss/_list-group.scss:188<br>via scss/_list-group.scss:186 | `var(--bs-info-bg-subtle)` | Runtime, Mixed |

<a id="c-info-6"></a>
#### $info – pattern 6: `to-rgb($ANCHOR)`

2 contexts, 2 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-focus-shadow-rgb` | `13, 202, 240` | `.btn-outline-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3227 | scss/mixins/_buttons.scss:51<br>via scss/_buttons.scss:158 | `to-rgb($info)` | Derived, Compile-time |
| `--bs-info-rgb` | `13, 202, 240` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:42<br>src/bootstrap/reference/css/bootstrap.css:43 | scss/_root.scss:22 | `to-rgb($info)` | Derived, Compile-time |

<a id="c-info-7"></a>
#### $info – pattern 7: `color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-active-…`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

Full expression:

```scss
color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-active-bg-factor)))
```

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-active-color` | `#000` | `.table-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1991 | scss/mixins/_table-variants.scss:16<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($info, $table-bg-scale)))), (shift-color($info, $table-bg-scale)), percentage($table-active-bg-factor)))` | Derived, Compile-time, Mixed |

<a id="c-info-8"></a>
#### $info – pattern 8: `color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-hover-b…`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

Full expression:

```scss
color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-hover-bg-factor)))
```

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-hover-color` | `#000` | `.table-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1993 | scss/mixins/_table-variants.scss:18<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($info, $table-bg-scale)))), (shift-color($info, $table-bg-scale)), percentage($table-hover-bg-factor)))` | Derived, Compile-time, Mixed |

<a id="c-info-9"></a>
#### $info – pattern 9: `color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-striped…`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

Full expression:

```scss
color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-striped-bg-factor)))
```

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-striped-color` | `#000` | `.table-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1989 | scss/mixins/_table-variants.scss:14<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($info, $table-bg-scale)))), (shift-color($info, $table-bg-scale)), percentage($table-striped-bg-factor)))` | Derived, Compile-time, Mixed |

<a id="c-info-10"></a>
#### $info – pattern 10: `color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale))))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-color` | `#000` | `.table-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1985 | scss/mixins/_table-variants.scss:10<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `color-contrast(opaque($body-bg, (shift-color($info, $table-bg-scale))))` | Derived, Compile-time, Mixed |

<a id="c-info-11"></a>
#### $info – pattern 11: `color-contrast(tint-color($ANCHOR, $btn-active-bg-tint-amount))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-color` | `#000` | `.btn-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3093 | scss/mixins/_buttons.scss:28<br>via scss/_buttons.scss:132 | `color-contrast(tint-color($info, $btn-active-bg-tint-amount))` | Derived, Compile-time, Mixed |

<a id="c-info-12"></a>
#### $info – pattern 12: `color-contrast(tint-color($ANCHOR, $btn-hover-bg-tint-amount))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-hover-color` | `#000` | `.btn-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3089 | scss/mixins/_buttons.scss:24<br>via scss/_buttons.scss:132 | `color-contrast(tint-color($info, $btn-hover-bg-tint-amount))` | Derived, Compile-time, Mixed |

<a id="c-info-13"></a>
#### $info – pattern 13: `mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-active-bg-factor))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-active-bg` | `#badce3` | `.table-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1990 | scss/mixins/_table-variants.scss:15<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `mix(color-contrast(opaque($body-bg, (shift-color($info, $table-bg-scale)))), (shift-color($info, $table-bg-scale)), percentage($table-active-bg-factor))` | Derived, Compile-time, Mixed |

<a id="c-info-14"></a>
#### $info – pattern 14: `mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-border-factor))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-border-color` | `#a6c3ca` | `.table-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1987 | scss/mixins/_table-variants.scss:12<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `mix(color-contrast(opaque($body-bg, (shift-color($info, $table-bg-scale)))), (shift-color($info, $table-bg-scale)), percentage($table-border-factor))` | Derived, Compile-time, Mixed |

<a id="c-info-15"></a>
#### $info – pattern 15: `mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-hover-bg-factor))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-hover-bg` | `#bfe2e9` | `.table-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1992 | scss/mixins/_table-variants.scss:17<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `mix(color-contrast(opaque($body-bg, (shift-color($info, $table-bg-scale)))), (shift-color($info, $table-bg-scale)), percentage($table-hover-bg-factor))` | Derived, Compile-time, Mixed |

<a id="c-info-16"></a>
#### $info – pattern 16: `mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-striped-bg-factor))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-striped-bg` | `#c5e8ef` | `.table-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1988 | scss/mixins/_table-variants.scss:13<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `mix(color-contrast(opaque($body-bg, (shift-color($info, $table-bg-scale)))), (shift-color($info, $table-bg-scale)), percentage($table-striped-bg-factor))` | Derived, Compile-time, Mixed |

<a id="c-info-17"></a>
#### $info – pattern 17: `rgba-css-var("info", "focus-ring") → rgba(var(--bs-info-rgb), var(--bs-focus-ring-opacity))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-focus-ring-color` | `rgba(var(--bs-info-rgb), var(--bs-focus-ring-opacity))` | `.focus-ring-info`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:754<br>src/bootstrap/reference/css/bootstrap.css:7413 | scss/mixins/_utilities.scss:58<br>via scss/mixins/_utilities.scss:57<br>+ scss/_utilities.scss:92 | `rgba-css-var("info", "focus-ring") → rgba(var(--bs-info-rgb), var(--bs-focus-ring-opacity))` | Derived, Runtime, Compile-time, Mixed |

<a id="c-info-18"></a>
#### $info – pattern 18: `shade-color($ANCHOR, 40%)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-info-border-subtle` | `#087990` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:162<br>src/bootstrap/reference/css/bootstrap.css:163 | scss/_root.scss:164<br>via scss/mixins/_color-mode.scss:16 | `shade-color($info, 40%)` | Direct |

<a id="c-info-19"></a>
#### $info – pattern 19: `shade-color($ANCHOR, 60%)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-info-text-emphasis` | `#055160` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:50<br>src/bootstrap/reference/css/bootstrap.css:51 | scss/_root.scss:26 | `shade-color($info, 60%)` | Direct |

<a id="c-info-20"></a>
#### $info – pattern 20: `shade-color($ANCHOR, 80%)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-info-bg-subtle` | `#032830` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:154<br>src/bootstrap/reference/css/bootstrap.css:155 | scss/_root.scss:160<br>via scss/mixins/_color-mode.scss:16 | `shade-color($info, 80%)` | Direct |

<a id="c-info-21"></a>
#### $info – pattern 21: `shift-color($ANCHOR, $table-bg-scale)`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-bg` | `#cff4fc` | `.table-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1986 | scss/mixins/_table-variants.scss:11<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `shift-color($info, $table-bg-scale)` | Derived, Compile-time, Mixed |

<a id="c-info-22"></a>
#### $info – pattern 22: `tint-color($ANCHOR, $btn-active-bg-tint-amount)`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-bg` | `#3dd5f3` | `.btn-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3094 | scss/mixins/_buttons.scss:29<br>via scss/_buttons.scss:132 | `tint-color($info, $btn-active-bg-tint-amount)` | Derived, Compile-time, Mixed |

<a id="c-info-23"></a>
#### $info – pattern 23: `tint-color($ANCHOR, $btn-active-border-tint-amount)`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-border-color` | `#25cff2` | `.btn-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3095 | scss/mixins/_buttons.scss:30<br>via scss/_buttons.scss:132 | `tint-color($info, $btn-active-border-tint-amount)` | Derived, Compile-time, Mixed |

<a id="c-info-24"></a>
#### $info – pattern 24: `tint-color($ANCHOR, $btn-hover-bg-tint-amount)`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-hover-bg` | `#31d2f2` | `.btn-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3090 | scss/mixins/_buttons.scss:25<br>via scss/_buttons.scss:132 | `tint-color($info, $btn-hover-bg-tint-amount)` | Derived, Compile-time, Mixed |

<a id="c-info-25"></a>
#### $info – pattern 25: `tint-color($ANCHOR, $btn-hover-border-tint-amount)`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-hover-border-color` | `#25cff2` | `.btn-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3091 | scss/mixins/_buttons.scss:26<br>via scss/_buttons.scss:132 | `tint-color($info, $btn-hover-border-tint-amount)` | Derived, Compile-time, Mixed |

<a id="c-info-26"></a>
#### $info – pattern 26: `tint-color($ANCHOR, 40%)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-info-text-emphasis` | `#6edff6` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:146<br>src/bootstrap/reference/css/bootstrap.css:147 | scss/_root.scss:156<br>via scss/mixins/_color-mode.scss:16 | `tint-color($info, 40%)` | Direct |

<a id="c-info-27"></a>
#### $info – pattern 27: `tint-color($ANCHOR, 60%)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-info-border-subtle` | `#9eeaf9` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:66<br>src/bootstrap/reference/css/bootstrap.css:67 | scss/_root.scss:34 | `tint-color($info, 60%)` | Direct |

<a id="c-info-28"></a>
#### $info – pattern 28: `tint-color($ANCHOR, 80%)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-info-bg-subtle` | `#cff4fc` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:58<br>src/bootstrap/reference/css/bootstrap.css:59 | scss/_root.scss:30 | `tint-color($info, 80%)` | Direct |

<a id="c-info-29"></a>
#### $info – pattern 29: `to-rgb(mix(color-contrast($ANCHOR), $ANCHOR, 15%))`

1 contexts, 1 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-focus-shadow-rgb` | `11, 172, 204` | `.btn-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3092 | scss/mixins/_buttons.scss:27<br>via scss/_buttons.scss:132 | `to-rgb(mix(color-contrast($info), $info, 15%))` | Derived, Compile-time |

<a id="c-warning"></a>
### `$warning` (theme)

Definition: `$warning: $yellow` (`scss/_variables.scss:305`). 54 contexts, 40 properties.

<a id="c-warning-1"></a>
#### $warning – pattern 1: `$ANCHOR`

13 contexts, 11 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-bg` | `#ffc107` | `.btn-outline-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3246 | scss/mixins/_buttons.scss:53<br>via scss/_buttons.scss:158 | `$warning` | Direct |
| `--bs-btn-active-border-color` | `#ffc107` | `.btn-outline-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3247 | scss/mixins/_buttons.scss:54<br>via scss/_buttons.scss:158 | `$warning` | Direct |
| `--bs-btn-bg` | `#ffc107` | `.btn-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3104 | scss/mixins/_buttons.scss:22<br>via scss/_buttons.scss:132 | `$warning` | Direct |
| `--bs-btn-border-color` | `#ffc107` | `.btn-outline-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3240 | scss/mixins/_buttons.scss:47<br>via scss/_buttons.scss:158 | `$warning` | Direct |
| `--bs-btn-border-color` | `#ffc107` | `.btn-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3105 | scss/mixins/_buttons.scss:23<br>via scss/_buttons.scss:132 | `$warning` | Direct |
| `--bs-btn-color` | `#ffc107` | `.btn-outline-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3239 | scss/mixins/_buttons.scss:46<br>via scss/_buttons.scss:158 | `$warning` | Direct |
| `--bs-btn-disabled-bg` | `#ffc107` | `.btn-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3115 | scss/mixins/_buttons.scss:33<br>via scss/_buttons.scss:132 | `$warning` | Direct |
| `--bs-btn-disabled-border-color` | `#ffc107` | `.btn-outline-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3251 | scss/mixins/_buttons.scss:58<br>via scss/_buttons.scss:158 | `$warning` | Direct |
| `--bs-btn-disabled-border-color` | `#ffc107` | `.btn-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3116 | scss/mixins/_buttons.scss:34<br>via scss/_buttons.scss:132 | `$warning` | Direct |
| `--bs-btn-disabled-color` | `#ffc107` | `.btn-outline-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3249 | scss/mixins/_buttons.scss:56<br>via scss/_buttons.scss:158 | `$warning` | Direct |
| `--bs-btn-hover-bg` | `#ffc107` | `.btn-outline-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3242 | scss/mixins/_buttons.scss:49<br>via scss/_buttons.scss:158 | `$warning` | Direct |
| `--bs-btn-hover-border-color` | `#ffc107` | `.btn-outline-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3243 | scss/mixins/_buttons.scss:50<br>via scss/_buttons.scss:158 | `$warning` | Direct |
| `--bs-warning` | `#ffc107` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:35<br>src/bootstrap/reference/css/bootstrap.css:36 | scss/_root.scss:18 | `$warning` | Direct |

<a id="c-warning-2"></a>
#### $warning – pattern 2: `var(--bs-warning-text-emphasis)`

5 contexts, 5 properties. Relationship: Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-alert-color` | `var(--bs-warning-text-emphasis)` | `.alert-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4904 | scss/_alert.scss:62<br>via scss/_alert.scss:61 | `var(--bs-warning-text-emphasis)` | Runtime, Mixed |
| `--bs-alert-link-color` | `var(--bs-warning-text-emphasis)` | `.alert-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4907 | scss/_alert.scss:65<br>via scss/_alert.scss:61 | `var(--bs-warning-text-emphasis)` | Runtime, Mixed |
| `--bs-list-group-active-bg` | `var(--bs-warning-text-emphasis)` | `.list-group-item-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5293 | scss/_list-group.scss:195<br>via scss/_list-group.scss:186 | `var(--bs-warning-text-emphasis)` | Runtime, Mixed |
| `--bs-list-group-active-border-color` | `var(--bs-warning-text-emphasis)` | `.list-group-item-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5294 | scss/_list-group.scss:196<br>via scss/_list-group.scss:186 | `var(--bs-warning-text-emphasis)` | Runtime, Mixed |
| `--bs-list-group-color` | `var(--bs-warning-text-emphasis)` | `.list-group-item-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5285 | scss/_list-group.scss:187<br>via scss/_list-group.scss:186 | `var(--bs-warning-text-emphasis)` | Runtime, Mixed |

<a id="c-warning-3"></a>
#### $warning – pattern 3: `color-contrast($ANCHOR)`

4 contexts, 4 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-color` | `#000` | `.btn-outline-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3245 | scss/mixins/_buttons.scss:52<br>via scss/_buttons.scss:158 | `color-contrast($warning)` | Derived, Compile-time |
| `--bs-btn-color` | `#000` | `.btn-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3103 | scss/mixins/_buttons.scss:21<br>via scss/_buttons.scss:132 | `color-contrast($warning)` | Derived, Compile-time |
| `--bs-btn-disabled-color` | `#000` | `.btn-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3114 | scss/mixins/_buttons.scss:32<br>via scss/_buttons.scss:132 | `color-contrast($warning)` | Derived, Compile-time |
| `--bs-btn-hover-color` | `#000` | `.btn-outline-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3241 | scss/mixins/_buttons.scss:48<br>via scss/_buttons.scss:158 | `color-contrast($warning)` | Derived, Compile-time |

<a id="c-warning-4"></a>
#### $warning – pattern 4: `var(--bs-warning-border-subtle)`

4 contexts, 4 properties. Relationship: Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-alert-border-color` | `var(--bs-warning-border-subtle)` | `.alert-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4906 | scss/_alert.scss:64<br>via scss/_alert.scss:61 | `var(--bs-warning-border-subtle)` | Runtime, Mixed |
| `--bs-list-group-action-active-bg` | `var(--bs-warning-border-subtle)` | `.list-group-item-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5291 | scss/_list-group.scss:193<br>via scss/_list-group.scss:186 | `var(--bs-warning-border-subtle)` | Runtime, Mixed |
| `--bs-list-group-action-hover-bg` | `var(--bs-warning-border-subtle)` | `.list-group-item-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5289 | scss/_list-group.scss:191<br>via scss/_list-group.scss:186 | `var(--bs-warning-border-subtle)` | Runtime, Mixed |
| `--bs-list-group-border-color` | `var(--bs-warning-border-subtle)` | `.list-group-item-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5287 | scss/_list-group.scss:189<br>via scss/_list-group.scss:186 | `var(--bs-warning-border-subtle)` | Runtime, Mixed |

<a id="c-warning-5"></a>
#### $warning – pattern 5: `var(--bs-warning-bg-subtle)`

3 contexts, 3 properties. Relationship: Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-alert-bg` | `var(--bs-warning-bg-subtle)` | `.alert-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4905 | scss/_alert.scss:63<br>via scss/_alert.scss:61 | `var(--bs-warning-bg-subtle)` | Runtime, Mixed |
| `--bs-list-group-active-color` | `var(--bs-warning-bg-subtle)` | `.list-group-item-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5292 | scss/_list-group.scss:194<br>via scss/_list-group.scss:186 | `var(--bs-warning-bg-subtle)` | Runtime, Mixed |
| `--bs-list-group-bg` | `var(--bs-warning-bg-subtle)` | `.list-group-item-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5286 | scss/_list-group.scss:188<br>via scss/_list-group.scss:186 | `var(--bs-warning-bg-subtle)` | Runtime, Mixed |

<a id="c-warning-6"></a>
#### $warning – pattern 6: `to-rgb($ANCHOR)`

2 contexts, 2 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-focus-shadow-rgb` | `255, 193, 7` | `.btn-outline-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3244 | scss/mixins/_buttons.scss:51<br>via scss/_buttons.scss:158 | `to-rgb($warning)` | Derived, Compile-time |
| `--bs-warning-rgb` | `255, 193, 7` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:43<br>src/bootstrap/reference/css/bootstrap.css:44 | scss/_root.scss:22 | `to-rgb($warning)` | Derived, Compile-time |

<a id="c-warning-7"></a>
#### $warning – pattern 7: `color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-active-…`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

Full expression:

```scss
color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-active-bg-factor)))
```

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-active-color` | `#000` | `.table-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2005 | scss/mixins/_table-variants.scss:16<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($warning, $table-bg-scale)))), (shift-color($warning, $table-bg-scale)), percentage($table-active-bg-factor)))` | Derived, Compile-time, Mixed |

<a id="c-warning-8"></a>
#### $warning – pattern 8: `color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-hover-b…`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

Full expression:

```scss
color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-hover-bg-factor)))
```

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-hover-color` | `#000` | `.table-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2007 | scss/mixins/_table-variants.scss:18<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($warning, $table-bg-scale)))), (shift-color($warning, $table-bg-scale)), percentage($table-hover-bg-factor)))` | Derived, Compile-time, Mixed |

<a id="c-warning-9"></a>
#### $warning – pattern 9: `color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-striped…`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

Full expression:

```scss
color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-striped-bg-factor)))
```

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-striped-color` | `#000` | `.table-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2003 | scss/mixins/_table-variants.scss:14<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($warning, $table-bg-scale)))), (shift-color($warning, $table-bg-scale)), percentage($table-striped-bg-factor)))` | Derived, Compile-time, Mixed |

<a id="c-warning-10"></a>
#### $warning – pattern 10: `color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale))))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-color` | `#000` | `.table-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1999 | scss/mixins/_table-variants.scss:10<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `color-contrast(opaque($body-bg, (shift-color($warning, $table-bg-scale))))` | Derived, Compile-time, Mixed |

<a id="c-warning-11"></a>
#### $warning – pattern 11: `color-contrast(tint-color($ANCHOR, $btn-active-bg-tint-amount))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-color` | `#000` | `.btn-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3110 | scss/mixins/_buttons.scss:28<br>via scss/_buttons.scss:132 | `color-contrast(tint-color($warning, $btn-active-bg-tint-amount))` | Derived, Compile-time, Mixed |

<a id="c-warning-12"></a>
#### $warning – pattern 12: `color-contrast(tint-color($ANCHOR, $btn-hover-bg-tint-amount))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-hover-color` | `#000` | `.btn-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3106 | scss/mixins/_buttons.scss:24<br>via scss/_buttons.scss:132 | `color-contrast(tint-color($warning, $btn-hover-bg-tint-amount))` | Derived, Compile-time, Mixed |

<a id="c-warning-13"></a>
#### $warning – pattern 13: `mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-active-bg-factor))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-active-bg` | `#e6dbb9` | `.table-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2004 | scss/mixins/_table-variants.scss:15<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `mix(color-contrast(opaque($body-bg, (shift-color($warning, $table-bg-scale)))), (shift-color($warning, $table-bg-scale)), percentage($table-active-bg-factor))` | Derived, Compile-time, Mixed |

<a id="c-warning-14"></a>
#### $warning – pattern 14: `mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-border-factor))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-border-color` | `#ccc2a4` | `.table-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2001 | scss/mixins/_table-variants.scss:12<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `mix(color-contrast(opaque($body-bg, (shift-color($warning, $table-bg-scale)))), (shift-color($warning, $table-bg-scale)), percentage($table-border-factor))` | Derived, Compile-time, Mixed |

<a id="c-warning-15"></a>
#### $warning – pattern 15: `mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-hover-bg-factor))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-hover-bg` | `#ece1be` | `.table-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2006 | scss/mixins/_table-variants.scss:17<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `mix(color-contrast(opaque($body-bg, (shift-color($warning, $table-bg-scale)))), (shift-color($warning, $table-bg-scale)), percentage($table-hover-bg-factor))` | Derived, Compile-time, Mixed |

<a id="c-warning-16"></a>
#### $warning – pattern 16: `mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-striped-bg-factor))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-striped-bg` | `#f2e7c3` | `.table-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2002 | scss/mixins/_table-variants.scss:13<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `mix(color-contrast(opaque($body-bg, (shift-color($warning, $table-bg-scale)))), (shift-color($warning, $table-bg-scale)), percentage($table-striped-bg-factor))` | Derived, Compile-time, Mixed |

<a id="c-warning-17"></a>
#### $warning – pattern 17: `rgba-css-var("warning", "focus-ring") → rgba(var(--bs-warning-rgb), var(--bs-focus-ring-opacity))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-focus-ring-color` | `rgba(var(--bs-warning-rgb), var(--bs-focus-ring-opacity))` | `.focus-ring-warning`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:758<br>src/bootstrap/reference/css/bootstrap.css:7417 | scss/mixins/_utilities.scss:58<br>via scss/mixins/_utilities.scss:57<br>+ scss/_utilities.scss:92 | `rgba-css-var("warning", "focus-ring") → rgba(var(--bs-warning-rgb), var(--bs-focus-ring-opacity))` | Derived, Runtime, Compile-time, Mixed |

<a id="c-warning-18"></a>
#### $warning – pattern 18: `shade-color($ANCHOR, 40%)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-warning-border-subtle` | `#997404` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:163<br>src/bootstrap/reference/css/bootstrap.css:164 | scss/_root.scss:164<br>via scss/mixins/_color-mode.scss:16 | `shade-color($warning, 40%)` | Direct |

<a id="c-warning-19"></a>
#### $warning – pattern 19: `shade-color($ANCHOR, 60%)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-warning-text-emphasis` | `#664d03` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:51<br>src/bootstrap/reference/css/bootstrap.css:52 | scss/_root.scss:26 | `shade-color($warning, 60%)` | Direct |

<a id="c-warning-20"></a>
#### $warning – pattern 20: `shade-color($ANCHOR, 80%)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-warning-bg-subtle` | `#332701` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:155<br>src/bootstrap/reference/css/bootstrap.css:156 | scss/_root.scss:160<br>via scss/mixins/_color-mode.scss:16 | `shade-color($warning, 80%)` | Direct |

<a id="c-warning-21"></a>
#### $warning – pattern 21: `shift-color($ANCHOR, $table-bg-scale)`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-bg` | `#fff3cd` | `.table-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2000 | scss/mixins/_table-variants.scss:11<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `shift-color($warning, $table-bg-scale)` | Derived, Compile-time, Mixed |

<a id="c-warning-22"></a>
#### $warning – pattern 22: `tint-color($ANCHOR, $btn-active-bg-tint-amount)`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-bg` | `#ffcd39` | `.btn-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3111 | scss/mixins/_buttons.scss:29<br>via scss/_buttons.scss:132 | `tint-color($warning, $btn-active-bg-tint-amount)` | Derived, Compile-time, Mixed |

<a id="c-warning-23"></a>
#### $warning – pattern 23: `tint-color($ANCHOR, $btn-active-border-tint-amount)`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-border-color` | `#ffc720` | `.btn-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3112 | scss/mixins/_buttons.scss:30<br>via scss/_buttons.scss:132 | `tint-color($warning, $btn-active-border-tint-amount)` | Derived, Compile-time, Mixed |

<a id="c-warning-24"></a>
#### $warning – pattern 24: `tint-color($ANCHOR, $btn-hover-bg-tint-amount)`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-hover-bg` | `#ffca2c` | `.btn-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3107 | scss/mixins/_buttons.scss:25<br>via scss/_buttons.scss:132 | `tint-color($warning, $btn-hover-bg-tint-amount)` | Derived, Compile-time, Mixed |

<a id="c-warning-25"></a>
#### $warning – pattern 25: `tint-color($ANCHOR, $btn-hover-border-tint-amount)`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-hover-border-color` | `#ffc720` | `.btn-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3108 | scss/mixins/_buttons.scss:26<br>via scss/_buttons.scss:132 | `tint-color($warning, $btn-hover-border-tint-amount)` | Derived, Compile-time, Mixed |

<a id="c-warning-26"></a>
#### $warning – pattern 26: `tint-color($ANCHOR, 40%)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-warning-text-emphasis` | `#ffda6a` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:147<br>src/bootstrap/reference/css/bootstrap.css:148 | scss/_root.scss:156<br>via scss/mixins/_color-mode.scss:16 | `tint-color($warning, 40%)` | Direct |

<a id="c-warning-27"></a>
#### $warning – pattern 27: `tint-color($ANCHOR, 60%)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-warning-border-subtle` | `#ffe69c` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:67<br>src/bootstrap/reference/css/bootstrap.css:68 | scss/_root.scss:34 | `tint-color($warning, 60%)` | Direct |

<a id="c-warning-28"></a>
#### $warning – pattern 28: `tint-color($ANCHOR, 80%)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-warning-bg-subtle` | `#fff3cd` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:59<br>src/bootstrap/reference/css/bootstrap.css:60 | scss/_root.scss:30 | `tint-color($warning, 80%)` | Direct |

<a id="c-warning-29"></a>
#### $warning – pattern 29: `to-rgb(mix(color-contrast($ANCHOR), $ANCHOR, 15%))`

1 contexts, 1 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-focus-shadow-rgb` | `217, 164, 6` | `.btn-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3109 | scss/mixins/_buttons.scss:27<br>via scss/_buttons.scss:132 | `to-rgb(mix(color-contrast($warning), $warning, 15%))` | Derived, Compile-time |

<a id="c-danger"></a>
### `$danger` (theme)

Definition: `$danger: $red` (`scss/_variables.scss:306`). 57 contexts, 43 properties.

<a id="c-danger-1"></a>
#### $danger – pattern 1: `$ANCHOR`

15 contexts, 13 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-bg` | `#dc3545` | `.btn-outline-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3263 | scss/mixins/_buttons.scss:53<br>via scss/_buttons.scss:158 | `$danger` | Direct |
| `--bs-btn-active-border-color` | `#dc3545` | `.btn-outline-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3264 | scss/mixins/_buttons.scss:54<br>via scss/_buttons.scss:158 | `$danger` | Direct |
| `--bs-btn-bg` | `#dc3545` | `.btn-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3121 | scss/mixins/_buttons.scss:22<br>via scss/_buttons.scss:132 | `$danger` | Direct |
| `--bs-btn-border-color` | `#dc3545` | `.btn-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3122 | scss/mixins/_buttons.scss:23<br>via scss/_buttons.scss:132 | `$danger` | Direct |
| `--bs-btn-border-color` | `#dc3545` | `.btn-outline-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3257 | scss/mixins/_buttons.scss:47<br>via scss/_buttons.scss:158 | `$danger` | Direct |
| `--bs-btn-color` | `#dc3545` | `.btn-outline-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3256 | scss/mixins/_buttons.scss:46<br>via scss/_buttons.scss:158 | `$danger` | Direct |
| `--bs-btn-disabled-bg` | `#dc3545` | `.btn-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3132 | scss/mixins/_buttons.scss:33<br>via scss/_buttons.scss:132 | `$danger` | Direct |
| `--bs-btn-disabled-border-color` | `#dc3545` | `.btn-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3133 | scss/mixins/_buttons.scss:34<br>via scss/_buttons.scss:132 | `$danger` | Direct |
| `--bs-btn-disabled-border-color` | `#dc3545` | `.btn-outline-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3268 | scss/mixins/_buttons.scss:58<br>via scss/_buttons.scss:158 | `$danger` | Direct |
| `--bs-btn-disabled-color` | `#dc3545` | `.btn-outline-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3266 | scss/mixins/_buttons.scss:56<br>via scss/_buttons.scss:158 | `$danger` | Direct |
| `--bs-btn-hover-bg` | `#dc3545` | `.btn-outline-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3259 | scss/mixins/_buttons.scss:49<br>via scss/_buttons.scss:158 | `$danger` | Direct |
| `--bs-btn-hover-border-color` | `#dc3545` | `.btn-outline-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3260 | scss/mixins/_buttons.scss:50<br>via scss/_buttons.scss:158 | `$danger` | Direct |
| `--bs-danger` | `#dc3545` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:36<br>src/bootstrap/reference/css/bootstrap.css:37 | scss/_root.scss:18 | `$danger` | Direct |
| `--bs-form-invalid-border-color` | `#dc3545` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:124<br>src/bootstrap/reference/css/bootstrap.css:125 | scss/_root.scss:128<br>via scss/_root.scss:1 | `$danger` | Direct |
| `--bs-form-invalid-color` | `#dc3545` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:123<br>src/bootstrap/reference/css/bootstrap.css:124 | scss/_root.scss:127<br>via scss/_root.scss:1 | `$danger` | Direct |

<a id="c-danger-2"></a>
#### $danger – pattern 2: `var(--bs-danger-text-emphasis)`

5 contexts, 5 properties. Relationship: Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-alert-color` | `var(--bs-danger-text-emphasis)` | `.alert-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4911 | scss/_alert.scss:62<br>via scss/_alert.scss:61 | `var(--bs-danger-text-emphasis)` | Runtime, Mixed |
| `--bs-alert-link-color` | `var(--bs-danger-text-emphasis)` | `.alert-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4914 | scss/_alert.scss:65<br>via scss/_alert.scss:61 | `var(--bs-danger-text-emphasis)` | Runtime, Mixed |
| `--bs-list-group-active-bg` | `var(--bs-danger-text-emphasis)` | `.list-group-item-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5306 | scss/_list-group.scss:195<br>via scss/_list-group.scss:186 | `var(--bs-danger-text-emphasis)` | Runtime, Mixed |
| `--bs-list-group-active-border-color` | `var(--bs-danger-text-emphasis)` | `.list-group-item-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5307 | scss/_list-group.scss:196<br>via scss/_list-group.scss:186 | `var(--bs-danger-text-emphasis)` | Runtime, Mixed |
| `--bs-list-group-color` | `var(--bs-danger-text-emphasis)` | `.list-group-item-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5298 | scss/_list-group.scss:187<br>via scss/_list-group.scss:186 | `var(--bs-danger-text-emphasis)` | Runtime, Mixed |

<a id="c-danger-3"></a>
#### $danger – pattern 3: `color-contrast($ANCHOR)`

4 contexts, 4 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-color` | `#fff` | `.btn-outline-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3262 | scss/mixins/_buttons.scss:52<br>via scss/_buttons.scss:158 | `color-contrast($danger)` | Derived, Compile-time |
| `--bs-btn-color` | `#fff` | `.btn-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3120 | scss/mixins/_buttons.scss:21<br>via scss/_buttons.scss:132 | `color-contrast($danger)` | Derived, Compile-time |
| `--bs-btn-disabled-color` | `#fff` | `.btn-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3131 | scss/mixins/_buttons.scss:32<br>via scss/_buttons.scss:132 | `color-contrast($danger)` | Derived, Compile-time |
| `--bs-btn-hover-color` | `#fff` | `.btn-outline-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3258 | scss/mixins/_buttons.scss:48<br>via scss/_buttons.scss:158 | `color-contrast($danger)` | Derived, Compile-time |

<a id="c-danger-4"></a>
#### $danger – pattern 4: `var(--bs-danger-border-subtle)`

4 contexts, 4 properties. Relationship: Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-alert-border-color` | `var(--bs-danger-border-subtle)` | `.alert-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4913 | scss/_alert.scss:64<br>via scss/_alert.scss:61 | `var(--bs-danger-border-subtle)` | Runtime, Mixed |
| `--bs-list-group-action-active-bg` | `var(--bs-danger-border-subtle)` | `.list-group-item-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5304 | scss/_list-group.scss:193<br>via scss/_list-group.scss:186 | `var(--bs-danger-border-subtle)` | Runtime, Mixed |
| `--bs-list-group-action-hover-bg` | `var(--bs-danger-border-subtle)` | `.list-group-item-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5302 | scss/_list-group.scss:191<br>via scss/_list-group.scss:186 | `var(--bs-danger-border-subtle)` | Runtime, Mixed |
| `--bs-list-group-border-color` | `var(--bs-danger-border-subtle)` | `.list-group-item-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5300 | scss/_list-group.scss:189<br>via scss/_list-group.scss:186 | `var(--bs-danger-border-subtle)` | Runtime, Mixed |

<a id="c-danger-5"></a>
#### $danger – pattern 5: `var(--bs-danger-bg-subtle)`

3 contexts, 3 properties. Relationship: Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-alert-bg` | `var(--bs-danger-bg-subtle)` | `.alert-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4912 | scss/_alert.scss:63<br>via scss/_alert.scss:61 | `var(--bs-danger-bg-subtle)` | Runtime, Mixed |
| `--bs-list-group-active-color` | `var(--bs-danger-bg-subtle)` | `.list-group-item-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5305 | scss/_list-group.scss:194<br>via scss/_list-group.scss:186 | `var(--bs-danger-bg-subtle)` | Runtime, Mixed |
| `--bs-list-group-bg` | `var(--bs-danger-bg-subtle)` | `.list-group-item-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5299 | scss/_list-group.scss:188<br>via scss/_list-group.scss:186 | `var(--bs-danger-bg-subtle)` | Runtime, Mixed |

<a id="c-danger-6"></a>
#### $danger – pattern 6: `to-rgb($ANCHOR)`

2 contexts, 2 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-focus-shadow-rgb` | `220, 53, 69` | `.btn-outline-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3261 | scss/mixins/_buttons.scss:51<br>via scss/_buttons.scss:158 | `to-rgb($danger)` | Derived, Compile-time |
| `--bs-danger-rgb` | `220, 53, 69` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:44<br>src/bootstrap/reference/css/bootstrap.css:45 | scss/_root.scss:22 | `to-rgb($danger)` | Derived, Compile-time |

<a id="c-danger-7"></a>
#### $danger – pattern 7: `color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-active-…`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

Full expression:

```scss
color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-active-bg-factor)))
```

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-active-color` | `#000` | `.table-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2019 | scss/mixins/_table-variants.scss:16<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($danger, $table-bg-scale)))), (shift-color($danger, $table-bg-scale)), percentage($table-active-bg-factor)))` | Derived, Compile-time, Mixed |

<a id="c-danger-8"></a>
#### $danger – pattern 8: `color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-hover-b…`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

Full expression:

```scss
color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-hover-bg-factor)))
```

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-hover-color` | `#000` | `.table-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2021 | scss/mixins/_table-variants.scss:18<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($danger, $table-bg-scale)))), (shift-color($danger, $table-bg-scale)), percentage($table-hover-bg-factor)))` | Derived, Compile-time, Mixed |

<a id="c-danger-9"></a>
#### $danger – pattern 9: `color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-striped…`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

Full expression:

```scss
color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-striped-bg-factor)))
```

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-striped-color` | `#000` | `.table-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2017 | scss/mixins/_table-variants.scss:14<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `color-contrast(mix(color-contrast(opaque($body-bg, (shift-color($danger, $table-bg-scale)))), (shift-color($danger, $table-bg-scale)), percentage($table-striped-bg-factor)))` | Derived, Compile-time, Mixed |

<a id="c-danger-10"></a>
#### $danger – pattern 10: `color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale))))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-color` | `#000` | `.table-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2013 | scss/mixins/_table-variants.scss:10<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `color-contrast(opaque($body-bg, (shift-color($danger, $table-bg-scale))))` | Derived, Compile-time, Mixed |

<a id="c-danger-11"></a>
#### $danger – pattern 11: `color-contrast(shade-color($ANCHOR, $btn-active-bg-shade-amount))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-color` | `#fff` | `.btn-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3127 | scss/mixins/_buttons.scss:28<br>via scss/_buttons.scss:132 | `color-contrast(shade-color($danger, $btn-active-bg-shade-amount))` | Derived, Compile-time, Mixed |

<a id="c-danger-12"></a>
#### $danger – pattern 12: `color-contrast(shade-color($ANCHOR, $btn-hover-bg-shade-amount))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-hover-color` | `#fff` | `.btn-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3123 | scss/mixins/_buttons.scss:24<br>via scss/_buttons.scss:132 | `color-contrast(shade-color($danger, $btn-hover-bg-shade-amount))` | Derived, Compile-time, Mixed |

<a id="c-danger-13"></a>
#### $danger – pattern 13: `escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 12 12' width='12' height='12' fill='none' stroke='$ANCHOR'><circle cx='6…`

1 contexts, 1 properties. Relationship: Compile-time.

Full expression:

```scss
escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 12 12' width='12' height='12' fill='none' stroke='$ANCHOR'><circle cx='6' cy='6' r='4.5'/><path stroke-linejoin='round' d='M5.8 3.6h.4L6 6.5z'/><circle cx='6' cy='8.2' r='.6' fill='$ANCHOR' stroke='none'/></svg>"))
```

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-form-select-bg-icon` | `url("data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 12 12' width='12' height='12' fill='none' stroke='%23dc3545'%3e%3ccircle cx='6' cy='6' r='4.5'/%3e%3cpath stroke-linejoin='round' d='M5.8 3.6h.4L6 6.5z'/%3e%3ccircle cx='6' cy='8.2' r='.6' fill='%23dc3545' stroke='none'/%3e%3c/svg%3e")` | `.was-validated .form-select:invalid:not([multiple]):not([size]), .was-validated .form-select:invalid:not([multiple])[siz…`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2914 | scss/mixins/_forms.scss:99<br>via scss/mixins/_forms.scss:97<br>+ scss/forms/_validation.scss:10 | `escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 12 12' width='12' height='12' fill='none' stroke='$danger'><circle cx='6' cy='6' r='4.5'/><path stroke-linejoin='round' d='M5.8 3.6h.4L6 6.5z'/><circle cx='6' cy='8.2' r='.6' fill='$danger' stroke='none'/></svg>"))` | Compile-time |

<a id="c-danger-14"></a>
#### $danger – pattern 14: `mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-active-bg-factor))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-active-bg` | `#dfc2c4` | `.table-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2018 | scss/mixins/_table-variants.scss:15<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `mix(color-contrast(opaque($body-bg, (shift-color($danger, $table-bg-scale)))), (shift-color($danger, $table-bg-scale)), percentage($table-active-bg-factor))` | Derived, Compile-time, Mixed |

<a id="c-danger-15"></a>
#### $danger – pattern 15: `mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-border-factor))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-border-color` | `#c6acae` | `.table-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2015 | scss/mixins/_table-variants.scss:12<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `mix(color-contrast(opaque($body-bg, (shift-color($danger, $table-bg-scale)))), (shift-color($danger, $table-bg-scale)), percentage($table-border-factor))` | Derived, Compile-time, Mixed |

<a id="c-danger-16"></a>
#### $danger – pattern 16: `mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-hover-bg-factor))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-hover-bg` | `#e5c7ca` | `.table-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2020 | scss/mixins/_table-variants.scss:17<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `mix(color-contrast(opaque($body-bg, (shift-color($danger, $table-bg-scale)))), (shift-color($danger, $table-bg-scale)), percentage($table-hover-bg-factor))` | Derived, Compile-time, Mixed |

<a id="c-danger-17"></a>
#### $danger – pattern 17: `mix(color-contrast(opaque($body-bg, (shift-color($ANCHOR, $table-bg-scale)))), (shift-color($ANCHOR, $table-bg-scale)), percentage($table-striped-bg-factor))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-striped-bg` | `#eccccf` | `.table-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2016 | scss/mixins/_table-variants.scss:13<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `mix(color-contrast(opaque($body-bg, (shift-color($danger, $table-bg-scale)))), (shift-color($danger, $table-bg-scale)), percentage($table-striped-bg-factor))` | Derived, Compile-time, Mixed |

<a id="c-danger-18"></a>
#### $danger – pattern 18: `rgba-css-var("danger", "focus-ring") → rgba(var(--bs-danger-rgb), var(--bs-focus-ring-opacity))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-focus-ring-color` | `rgba(var(--bs-danger-rgb), var(--bs-focus-ring-opacity))` | `.focus-ring-danger`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:762<br>src/bootstrap/reference/css/bootstrap.css:7421 | scss/mixins/_utilities.scss:58<br>via scss/mixins/_utilities.scss:57<br>+ scss/_utilities.scss:92 | `rgba-css-var("danger", "focus-ring") → rgba(var(--bs-danger-rgb), var(--bs-focus-ring-opacity))` | Derived, Runtime, Compile-time, Mixed |

<a id="c-danger-19"></a>
#### $danger – pattern 19: `shade-color($ANCHOR, $btn-active-bg-shade-amount)`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-bg` | `#b02a37` | `.btn-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3128 | scss/mixins/_buttons.scss:29<br>via scss/_buttons.scss:132 | `shade-color($danger, $btn-active-bg-shade-amount)` | Derived, Compile-time, Mixed |

<a id="c-danger-20"></a>
#### $danger – pattern 20: `shade-color($ANCHOR, $btn-active-border-shade-amount)`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-border-color` | `#a52834` | `.btn-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3129 | scss/mixins/_buttons.scss:30<br>via scss/_buttons.scss:132 | `shade-color($danger, $btn-active-border-shade-amount)` | Derived, Compile-time, Mixed |

<a id="c-danger-21"></a>
#### $danger – pattern 21: `shade-color($ANCHOR, $btn-hover-bg-shade-amount)`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-hover-bg` | `#bb2d3b` | `.btn-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3124 | scss/mixins/_buttons.scss:25<br>via scss/_buttons.scss:132 | `shade-color($danger, $btn-hover-bg-shade-amount)` | Derived, Compile-time, Mixed |

<a id="c-danger-22"></a>
#### $danger – pattern 22: `shade-color($ANCHOR, $btn-hover-border-shade-amount)`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-hover-border-color` | `#b02a37` | `.btn-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3125 | scss/mixins/_buttons.scss:26<br>via scss/_buttons.scss:132 | `shade-color($danger, $btn-hover-border-shade-amount)` | Derived, Compile-time, Mixed |

<a id="c-danger-23"></a>
#### $danger – pattern 23: `shade-color($ANCHOR, 40%)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-danger-border-subtle` | `#842029` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:164<br>src/bootstrap/reference/css/bootstrap.css:165 | scss/_root.scss:164<br>via scss/mixins/_color-mode.scss:16 | `shade-color($danger, 40%)` | Direct |

<a id="c-danger-24"></a>
#### $danger – pattern 24: `shade-color($ANCHOR, 60%)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-danger-text-emphasis` | `#58151c` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:52<br>src/bootstrap/reference/css/bootstrap.css:53 | scss/_root.scss:26 | `shade-color($danger, 60%)` | Direct |

<a id="c-danger-25"></a>
#### $danger – pattern 25: `shade-color($ANCHOR, 80%)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-danger-bg-subtle` | `#2c0b0e` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:156<br>src/bootstrap/reference/css/bootstrap.css:157 | scss/_root.scss:160<br>via scss/mixins/_color-mode.scss:16 | `shade-color($danger, 80%)` | Direct |

<a id="c-danger-26"></a>
#### $danger – pattern 26: `shift-color($ANCHOR, $table-bg-scale)`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-bg` | `#f8d7da` | `.table-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2014 | scss/mixins/_table-variants.scss:11<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `shift-color($danger, $table-bg-scale)` | Derived, Compile-time, Mixed |

<a id="c-danger-27"></a>
#### $danger – pattern 27: `tint-color($ANCHOR, 40%)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-danger-text-emphasis` | `#ea868f` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:148<br>src/bootstrap/reference/css/bootstrap.css:149 | scss/_root.scss:156<br>via scss/mixins/_color-mode.scss:16 | `tint-color($danger, 40%)` | Direct |

<a id="c-danger-28"></a>
#### $danger – pattern 28: `tint-color($ANCHOR, 60%)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-danger-border-subtle` | `#f1aeb5` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:68<br>src/bootstrap/reference/css/bootstrap.css:69 | scss/_root.scss:34 | `tint-color($danger, 60%)` | Direct |

<a id="c-danger-29"></a>
#### $danger – pattern 29: `tint-color($ANCHOR, 80%)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-danger-bg-subtle` | `#f8d7da` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:60<br>src/bootstrap/reference/css/bootstrap.css:61 | scss/_root.scss:30 | `tint-color($danger, 80%)` | Direct |

<a id="c-danger-30"></a>
#### $danger – pattern 30: `to-rgb(mix(color-contrast($ANCHOR), $ANCHOR, 15%))`

1 contexts, 1 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-focus-shadow-rgb` | `225, 83, 97` | `.btn-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3126 | scss/mixins/_buttons.scss:27<br>via scss/_buttons.scss:132 | `to-rgb(mix(color-contrast($danger), $danger, 15%))` | Derived, Compile-time |

<a id="c-light"></a>
### `$light` (theme)

Definition: `$light: $gray-100` (`scss/_variables.scss:307`). 36 contexts, 25 properties.

<a id="c-light-1"></a>
#### $light – pattern 1: `$ANCHOR`

14 contexts, 12 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-bg` | `#f8f9fa` | `.btn-outline-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3280 | scss/mixins/_buttons.scss:53<br>via scss/_buttons.scss:158 | `$light` | Direct |
| `--bs-btn-active-border-color` | `#f8f9fa` | `.btn-outline-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3281 | scss/mixins/_buttons.scss:54<br>via scss/_buttons.scss:158 | `$light` | Direct |
| `--bs-btn-bg` | `#f8f9fa` | `.btn-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3138 | scss/mixins/_buttons.scss:22<br>via scss/_buttons.scss:132 | `$light` | Direct |
| `--bs-btn-border-color` | `#f8f9fa` | `.btn-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3139 | scss/mixins/_buttons.scss:23<br>via scss/_buttons.scss:132 | `$light` | Direct |
| `--bs-btn-border-color` | `#f8f9fa` | `.btn-outline-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3274 | scss/mixins/_buttons.scss:47<br>via scss/_buttons.scss:158 | `$light` | Direct |
| `--bs-btn-color` | `#f8f9fa` | `.btn-outline-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3273 | scss/mixins/_buttons.scss:46<br>via scss/_buttons.scss:158 | `$light` | Direct |
| `--bs-btn-disabled-bg` | `#f8f9fa` | `.btn-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3149 | scss/mixins/_buttons.scss:33<br>via scss/_buttons.scss:132 | `$light` | Direct |
| `--bs-btn-disabled-border-color` | `#f8f9fa` | `.btn-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3150 | scss/mixins/_buttons.scss:34<br>via scss/_buttons.scss:132 | `$light` | Direct |
| `--bs-btn-disabled-border-color` | `#f8f9fa` | `.btn-outline-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3285 | scss/mixins/_buttons.scss:58<br>via scss/_buttons.scss:158 | `$light` | Direct |
| `--bs-btn-disabled-color` | `#f8f9fa` | `.btn-outline-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3283 | scss/mixins/_buttons.scss:56<br>via scss/_buttons.scss:158 | `$light` | Direct |
| `--bs-btn-hover-bg` | `#f8f9fa` | `.btn-outline-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3276 | scss/mixins/_buttons.scss:49<br>via scss/_buttons.scss:158 | `$light` | Direct |
| `--bs-btn-hover-border-color` | `#f8f9fa` | `.btn-outline-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3277 | scss/mixins/_buttons.scss:50<br>via scss/_buttons.scss:158 | `$light` | Direct |
| `--bs-light` | `#f8f9fa` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:37<br>src/bootstrap/reference/css/bootstrap.css:38 | scss/_root.scss:18 | `$light` | Direct |
| `--bs-table-bg` | `#f8f9fa` | `.table-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2028 | scss/mixins/_table-variants.scss:11<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `$light` | Direct |

<a id="c-light-2"></a>
#### $light – pattern 2: `color-contrast($ANCHOR)`

4 contexts, 4 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-color` | `#000` | `.btn-outline-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3279 | scss/mixins/_buttons.scss:52<br>via scss/_buttons.scss:158 | `color-contrast($light)` | Derived, Compile-time |
| `--bs-btn-color` | `#000` | `.btn-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3137 | scss/mixins/_buttons.scss:21<br>via scss/_buttons.scss:132 | `color-contrast($light)` | Derived, Compile-time |
| `--bs-btn-disabled-color` | `#000` | `.btn-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3148 | scss/mixins/_buttons.scss:32<br>via scss/_buttons.scss:132 | `color-contrast($light)` | Derived, Compile-time |
| `--bs-btn-hover-color` | `#000` | `.btn-outline-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3275 | scss/mixins/_buttons.scss:48<br>via scss/_buttons.scss:158 | `color-contrast($light)` | Derived, Compile-time |

<a id="c-light-3"></a>
#### $light – pattern 3: `to-rgb($ANCHOR)`

2 contexts, 2 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-focus-shadow-rgb` | `248, 249, 250` | `.btn-outline-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3278 | scss/mixins/_buttons.scss:51<br>via scss/_buttons.scss:158 | `to-rgb($light)` | Derived, Compile-time |
| `--bs-light-rgb` | `248, 249, 250` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:45<br>src/bootstrap/reference/css/bootstrap.css:46 | scss/_root.scss:22 | `to-rgb($light)` | Derived, Compile-time |

<a id="c-light-4"></a>
#### $light – pattern 4: `color-contrast(mix(color-contrast(opaque($body-bg, ($ANCHOR))), ($ANCHOR), percentage($table-active-bg-factor)))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-active-color` | `#000` | `.table-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2033 | scss/mixins/_table-variants.scss:16<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `color-contrast(mix(color-contrast(opaque($body-bg, ($light))), ($light), percentage($table-active-bg-factor)))` | Derived, Compile-time, Mixed |

<a id="c-light-5"></a>
#### $light – pattern 5: `color-contrast(mix(color-contrast(opaque($body-bg, ($ANCHOR))), ($ANCHOR), percentage($table-hover-bg-factor)))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-hover-color` | `#000` | `.table-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2035 | scss/mixins/_table-variants.scss:18<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `color-contrast(mix(color-contrast(opaque($body-bg, ($light))), ($light), percentage($table-hover-bg-factor)))` | Derived, Compile-time, Mixed |

<a id="c-light-6"></a>
#### $light – pattern 6: `color-contrast(mix(color-contrast(opaque($body-bg, ($ANCHOR))), ($ANCHOR), percentage($table-striped-bg-factor)))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-striped-color` | `#000` | `.table-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2031 | scss/mixins/_table-variants.scss:14<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `color-contrast(mix(color-contrast(opaque($body-bg, ($light))), ($light), percentage($table-striped-bg-factor)))` | Derived, Compile-time, Mixed |

<a id="c-light-7"></a>
#### $light – pattern 7: `color-contrast(opaque($body-bg, ($ANCHOR)))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-color` | `#000` | `.table-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2027 | scss/mixins/_table-variants.scss:10<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `color-contrast(opaque($body-bg, ($light)))` | Derived, Compile-time, Mixed |

<a id="c-light-8"></a>
#### $light – pattern 8: `color-contrast(shade-color($ANCHOR, $btn-active-bg-shade-amount))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-color` | `#000` | `.btn-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3144 | scss/mixins/_buttons.scss:28<br>via scss/_buttons.scss:132 | `color-contrast(shade-color($light, $btn-active-bg-shade-amount))` | Derived, Compile-time, Mixed |

<a id="c-light-9"></a>
#### $light – pattern 9: `color-contrast(shade-color($ANCHOR, $btn-hover-bg-shade-amount))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-hover-color` | `#000` | `.btn-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3140 | scss/mixins/_buttons.scss:24<br>via scss/_buttons.scss:132 | `color-contrast(shade-color($light, $btn-hover-bg-shade-amount))` | Derived, Compile-time, Mixed |

<a id="c-light-10"></a>
#### $light – pattern 10: `mix(color-contrast(opaque($body-bg, ($ANCHOR))), ($ANCHOR), percentage($table-active-bg-factor))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-active-bg` | `#dfe0e1` | `.table-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2032 | scss/mixins/_table-variants.scss:15<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `mix(color-contrast(opaque($body-bg, ($light))), ($light), percentage($table-active-bg-factor))` | Derived, Compile-time, Mixed |

<a id="c-light-11"></a>
#### $light – pattern 11: `mix(color-contrast(opaque($body-bg, ($ANCHOR))), ($ANCHOR), percentage($table-border-factor))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-border-color` | `#c6c7c8` | `.table-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2029 | scss/mixins/_table-variants.scss:12<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `mix(color-contrast(opaque($body-bg, ($light))), ($light), percentage($table-border-factor))` | Derived, Compile-time, Mixed |

<a id="c-light-12"></a>
#### $light – pattern 12: `mix(color-contrast(opaque($body-bg, ($ANCHOR))), ($ANCHOR), percentage($table-hover-bg-factor))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-hover-bg` | `#e5e6e7` | `.table-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2034 | scss/mixins/_table-variants.scss:17<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `mix(color-contrast(opaque($body-bg, ($light))), ($light), percentage($table-hover-bg-factor))` | Derived, Compile-time, Mixed |

<a id="c-light-13"></a>
#### $light – pattern 13: `mix(color-contrast(opaque($body-bg, ($ANCHOR))), ($ANCHOR), percentage($table-striped-bg-factor))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-striped-bg` | `#ecedee` | `.table-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2030 | scss/mixins/_table-variants.scss:13<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `mix(color-contrast(opaque($body-bg, ($light))), ($light), percentage($table-striped-bg-factor))` | Derived, Compile-time, Mixed |

<a id="c-light-14"></a>
#### $light – pattern 14: `rgba-css-var("light", "focus-ring") → rgba(var(--bs-light-rgb), var(--bs-focus-ring-opacity))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-focus-ring-color` | `rgba(var(--bs-light-rgb), var(--bs-focus-ring-opacity))` | `.focus-ring-light`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:766<br>src/bootstrap/reference/css/bootstrap.css:7425 | scss/mixins/_utilities.scss:58<br>via scss/mixins/_utilities.scss:57<br>+ scss/_utilities.scss:92 | `rgba-css-var("light", "focus-ring") → rgba(var(--bs-light-rgb), var(--bs-focus-ring-opacity))` | Derived, Runtime, Compile-time, Mixed |

<a id="c-light-15"></a>
#### $light – pattern 15: `shade-color($ANCHOR, $btn-active-bg-shade-amount)`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-bg` | `#c6c7c8` | `.btn-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3145 | scss/mixins/_buttons.scss:29<br>via scss/_buttons.scss:132 | `shade-color($light, $btn-active-bg-shade-amount)` | Derived, Compile-time, Mixed |

<a id="c-light-16"></a>
#### $light – pattern 16: `shade-color($ANCHOR, $btn-active-border-shade-amount)`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-border-color` | `#babbbc` | `.btn-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3146 | scss/mixins/_buttons.scss:30<br>via scss/_buttons.scss:132 | `shade-color($light, $btn-active-border-shade-amount)` | Derived, Compile-time, Mixed |

<a id="c-light-17"></a>
#### $light – pattern 17: `shade-color($ANCHOR, $btn-hover-bg-shade-amount)`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-hover-bg` | `#d3d4d5` | `.btn-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3141 | scss/mixins/_buttons.scss:25<br>via scss/_buttons.scss:132 | `shade-color($light, $btn-hover-bg-shade-amount)` | Derived, Compile-time, Mixed |

<a id="c-light-18"></a>
#### $light – pattern 18: `shade-color($ANCHOR, $btn-hover-border-shade-amount)`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-hover-border-color` | `#c6c7c8` | `.btn-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3142 | scss/mixins/_buttons.scss:26<br>via scss/_buttons.scss:132 | `shade-color($light, $btn-hover-border-shade-amount)` | Derived, Compile-time, Mixed |

<a id="c-light-19"></a>
#### $light – pattern 19: `to-rgb(mix(color-contrast($ANCHOR), $ANCHOR, 15%))`

1 contexts, 1 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-focus-shadow-rgb` | `211, 212, 213` | `.btn-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3143 | scss/mixins/_buttons.scss:27<br>via scss/_buttons.scss:132 | `to-rgb(mix(color-contrast($light), $light, 15%))` | Derived, Compile-time |

<a id="c-dark"></a>
### `$dark` (theme)

Definition: `$dark: $gray-900` (`scss/_variables.scss:308`). 36 contexts, 25 properties.

<a id="c-dark-1"></a>
#### $dark – pattern 1: `$ANCHOR`

14 contexts, 12 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-bg` | `#212529` | `.btn-outline-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3297 | scss/mixins/_buttons.scss:53<br>via scss/_buttons.scss:158 | `$dark` | Direct |
| `--bs-btn-active-border-color` | `#212529` | `.btn-outline-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3298 | scss/mixins/_buttons.scss:54<br>via scss/_buttons.scss:158 | `$dark` | Direct |
| `--bs-btn-bg` | `#212529` | `.btn-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3155 | scss/mixins/_buttons.scss:22<br>via scss/_buttons.scss:132 | `$dark` | Direct |
| `--bs-btn-border-color` | `#212529` | `.btn-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3156 | scss/mixins/_buttons.scss:23<br>via scss/_buttons.scss:132 | `$dark` | Direct |
| `--bs-btn-border-color` | `#212529` | `.btn-outline-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3291 | scss/mixins/_buttons.scss:47<br>via scss/_buttons.scss:158 | `$dark` | Direct |
| `--bs-btn-color` | `#212529` | `.btn-outline-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3290 | scss/mixins/_buttons.scss:46<br>via scss/_buttons.scss:158 | `$dark` | Direct |
| `--bs-btn-disabled-bg` | `#212529` | `.btn-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3166 | scss/mixins/_buttons.scss:33<br>via scss/_buttons.scss:132 | `$dark` | Direct |
| `--bs-btn-disabled-border-color` | `#212529` | `.btn-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3167 | scss/mixins/_buttons.scss:34<br>via scss/_buttons.scss:132 | `$dark` | Direct |
| `--bs-btn-disabled-border-color` | `#212529` | `.btn-outline-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3302 | scss/mixins/_buttons.scss:58<br>via scss/_buttons.scss:158 | `$dark` | Direct |
| `--bs-btn-disabled-color` | `#212529` | `.btn-outline-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3300 | scss/mixins/_buttons.scss:56<br>via scss/_buttons.scss:158 | `$dark` | Direct |
| `--bs-btn-hover-bg` | `#212529` | `.btn-outline-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3293 | scss/mixins/_buttons.scss:49<br>via scss/_buttons.scss:158 | `$dark` | Direct |
| `--bs-btn-hover-border-color` | `#212529` | `.btn-outline-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3294 | scss/mixins/_buttons.scss:50<br>via scss/_buttons.scss:158 | `$dark` | Direct |
| `--bs-dark` | `#212529` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:38<br>src/bootstrap/reference/css/bootstrap.css:39 | scss/_root.scss:18 | `$dark` | Direct |
| `--bs-table-bg` | `#212529` | `.table-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2042 | scss/mixins/_table-variants.scss:11<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `$dark` | Direct |

<a id="c-dark-2"></a>
#### $dark – pattern 2: `color-contrast($ANCHOR)`

4 contexts, 4 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-color` | `#fff` | `.btn-outline-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3296 | scss/mixins/_buttons.scss:52<br>via scss/_buttons.scss:158 | `color-contrast($dark)` | Derived, Compile-time |
| `--bs-btn-color` | `#fff` | `.btn-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3154 | scss/mixins/_buttons.scss:21<br>via scss/_buttons.scss:132 | `color-contrast($dark)` | Derived, Compile-time |
| `--bs-btn-disabled-color` | `#fff` | `.btn-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3165 | scss/mixins/_buttons.scss:32<br>via scss/_buttons.scss:132 | `color-contrast($dark)` | Derived, Compile-time |
| `--bs-btn-hover-color` | `#fff` | `.btn-outline-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3292 | scss/mixins/_buttons.scss:48<br>via scss/_buttons.scss:158 | `color-contrast($dark)` | Derived, Compile-time |

<a id="c-dark-3"></a>
#### $dark – pattern 3: `to-rgb($ANCHOR)`

2 contexts, 2 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-focus-shadow-rgb` | `33, 37, 41` | `.btn-outline-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3295 | scss/mixins/_buttons.scss:51<br>via scss/_buttons.scss:158 | `to-rgb($dark)` | Derived, Compile-time |
| `--bs-dark-rgb` | `33, 37, 41` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:46<br>src/bootstrap/reference/css/bootstrap.css:47 | scss/_root.scss:22 | `to-rgb($dark)` | Derived, Compile-time |

<a id="c-dark-4"></a>
#### $dark – pattern 4: `color-contrast(mix(color-contrast(opaque($body-bg, ($ANCHOR))), ($ANCHOR), percentage($table-active-bg-factor)))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-active-color` | `#fff` | `.table-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2047 | scss/mixins/_table-variants.scss:16<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `color-contrast(mix(color-contrast(opaque($body-bg, ($dark))), ($dark), percentage($table-active-bg-factor)))` | Derived, Compile-time, Mixed |

<a id="c-dark-5"></a>
#### $dark – pattern 5: `color-contrast(mix(color-contrast(opaque($body-bg, ($ANCHOR))), ($ANCHOR), percentage($table-hover-bg-factor)))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-hover-color` | `#fff` | `.table-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2049 | scss/mixins/_table-variants.scss:18<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `color-contrast(mix(color-contrast(opaque($body-bg, ($dark))), ($dark), percentage($table-hover-bg-factor)))` | Derived, Compile-time, Mixed |

<a id="c-dark-6"></a>
#### $dark – pattern 6: `color-contrast(mix(color-contrast(opaque($body-bg, ($ANCHOR))), ($ANCHOR), percentage($table-striped-bg-factor)))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-striped-color` | `#fff` | `.table-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2045 | scss/mixins/_table-variants.scss:14<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `color-contrast(mix(color-contrast(opaque($body-bg, ($dark))), ($dark), percentage($table-striped-bg-factor)))` | Derived, Compile-time, Mixed |

<a id="c-dark-7"></a>
#### $dark – pattern 7: `color-contrast(opaque($body-bg, ($ANCHOR)))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-color` | `#fff` | `.table-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2041 | scss/mixins/_table-variants.scss:10<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `color-contrast(opaque($body-bg, ($dark)))` | Derived, Compile-time, Mixed |

<a id="c-dark-8"></a>
#### $dark – pattern 8: `color-contrast(tint-color($ANCHOR, $btn-active-bg-tint-amount))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-color` | `#fff` | `.btn-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3161 | scss/mixins/_buttons.scss:28<br>via scss/_buttons.scss:132 | `color-contrast(tint-color($dark, $btn-active-bg-tint-amount))` | Derived, Compile-time, Mixed |

<a id="c-dark-9"></a>
#### $dark – pattern 9: `color-contrast(tint-color($ANCHOR, $btn-hover-bg-tint-amount))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-hover-color` | `#fff` | `.btn-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3157 | scss/mixins/_buttons.scss:24<br>via scss/_buttons.scss:132 | `color-contrast(tint-color($dark, $btn-hover-bg-tint-amount))` | Derived, Compile-time, Mixed |

<a id="c-dark-10"></a>
#### $dark – pattern 10: `mix(color-contrast(opaque($body-bg, ($ANCHOR))), ($ANCHOR), percentage($table-active-bg-factor))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-active-bg` | `#373b3e` | `.table-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2046 | scss/mixins/_table-variants.scss:15<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `mix(color-contrast(opaque($body-bg, ($dark))), ($dark), percentage($table-active-bg-factor))` | Derived, Compile-time, Mixed |

<a id="c-dark-11"></a>
#### $dark – pattern 11: `mix(color-contrast(opaque($body-bg, ($ANCHOR))), ($ANCHOR), percentage($table-border-factor))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-border-color` | `#4d5154` | `.table-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2043 | scss/mixins/_table-variants.scss:12<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `mix(color-contrast(opaque($body-bg, ($dark))), ($dark), percentage($table-border-factor))` | Derived, Compile-time, Mixed |

<a id="c-dark-12"></a>
#### $dark – pattern 12: `mix(color-contrast(opaque($body-bg, ($ANCHOR))), ($ANCHOR), percentage($table-hover-bg-factor))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-hover-bg` | `#323539` | `.table-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2048 | scss/mixins/_table-variants.scss:17<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `mix(color-contrast(opaque($body-bg, ($dark))), ($dark), percentage($table-hover-bg-factor))` | Derived, Compile-time, Mixed |

<a id="c-dark-13"></a>
#### $dark – pattern 13: `mix(color-contrast(opaque($body-bg, ($ANCHOR))), ($ANCHOR), percentage($table-striped-bg-factor))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-striped-bg` | `#2c3034` | `.table-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2044 | scss/mixins/_table-variants.scss:13<br>via scss/mixins/_table-variants.scss:3<br>+ scss/_tables.scss:153 | `mix(color-contrast(opaque($body-bg, ($dark))), ($dark), percentage($table-striped-bg-factor))` | Derived, Compile-time, Mixed |

<a id="c-dark-14"></a>
#### $dark – pattern 14: `rgba-css-var("dark", "focus-ring") → rgba(var(--bs-dark-rgb), var(--bs-focus-ring-opacity))`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-focus-ring-color` | `rgba(var(--bs-dark-rgb), var(--bs-focus-ring-opacity))` | `.focus-ring-dark`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:770<br>src/bootstrap/reference/css/bootstrap.css:7429 | scss/mixins/_utilities.scss:58<br>via scss/mixins/_utilities.scss:57<br>+ scss/_utilities.scss:92 | `rgba-css-var("dark", "focus-ring") → rgba(var(--bs-dark-rgb), var(--bs-focus-ring-opacity))` | Derived, Runtime, Compile-time, Mixed |

<a id="c-dark-15"></a>
#### $dark – pattern 15: `tint-color($ANCHOR, $btn-active-bg-tint-amount)`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-bg` | `#4d5154` | `.btn-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3162 | scss/mixins/_buttons.scss:29<br>via scss/_buttons.scss:132 | `tint-color($dark, $btn-active-bg-tint-amount)` | Derived, Compile-time, Mixed |

<a id="c-dark-16"></a>
#### $dark – pattern 16: `tint-color($ANCHOR, $btn-active-border-tint-amount)`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-border-color` | `#373b3e` | `.btn-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3163 | scss/mixins/_buttons.scss:30<br>via scss/_buttons.scss:132 | `tint-color($dark, $btn-active-border-tint-amount)` | Derived, Compile-time, Mixed |

<a id="c-dark-17"></a>
#### $dark – pattern 17: `tint-color($ANCHOR, $btn-hover-bg-tint-amount)`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-hover-bg` | `#424649` | `.btn-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3158 | scss/mixins/_buttons.scss:25<br>via scss/_buttons.scss:132 | `tint-color($dark, $btn-hover-bg-tint-amount)` | Derived, Compile-time, Mixed |

<a id="c-dark-18"></a>
#### $dark – pattern 18: `tint-color($ANCHOR, $btn-hover-border-tint-amount)`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Mixed.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-hover-border-color` | `#373b3e` | `.btn-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3159 | scss/mixins/_buttons.scss:26<br>via scss/_buttons.scss:132 | `tint-color($dark, $btn-hover-border-tint-amount)` | Derived, Compile-time, Mixed |

<a id="c-dark-19"></a>
#### $dark – pattern 19: `to-rgb(mix(color-contrast($ANCHOR), $ANCHOR, 15%))`

1 contexts, 1 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-focus-shadow-rgb` | `66, 70, 73` | `.btn-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3160 | scss/mixins/_buttons.scss:27<br>via scss/_buttons.scss:132 | `to-rgb(mix(color-contrast($dark), $dark, 15%))` | Derived, Compile-time |

<a id="c-body-bg"></a>
### `$body-bg` (body / emphasis colour)

Definition: `$body-bg: $white` (`scss/_variables.scss:439`). 18 contexts, 18 properties.

<a id="c-body-bg-1"></a>
#### $body-bg – pattern 1: `var(--bs-body-bg)`

13 contexts, 13 properties. Relationship: Direct, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-accordion-bg` | `var(--bs-body-bg)` | `.accordion`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4548 | scss/_accordion.scss:8<br>via scss/_accordion.scss:5 | `var(--bs-body-bg)` | Direct, Runtime, Mixed |
| `--bs-card-bg` | `var(--bs-body-bg)` | `.card`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4376 | scss/_card.scss:23<br>via scss/_card.scss:5 | `var(--bs-body-bg)` | Direct, Runtime, Mixed |
| `--bs-dropdown-bg` | `var(--bs-body-bg)` | `.dropdown-menu`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3413 | scss/_dropdown.scss:28<br>via scss/_dropdown.scss:19 | `var(--bs-body-bg)` | Direct, Runtime, Mixed |
| `--bs-form-check-bg` | `var(--bs-body-bg)` | `.form-check-input`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2394 | scss/forms/_form-check.scss:30<br>via scss/forms/_form-check.scss:29 | `var(--bs-body-bg)` | Direct, Runtime, Mixed |
| `--bs-list-group-bg` | `var(--bs-body-bg)` | `.list-group`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4995 | scss/_list-group.scss:8<br>via scss/_list-group.scss:5 | `var(--bs-body-bg)` | Direct, Runtime, Mixed |
| `--bs-list-group-disabled-bg` | `var(--bs-body-bg)` | `.list-group`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5007 | scss/_list-group.scss:20<br>via scss/_list-group.scss:5 | `var(--bs-body-bg)` | Direct, Runtime, Mixed |
| `--bs-modal-bg` | `var(--bs-body-bg)` | `.modal`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5461 | scss/_modal.scss:17<br>via scss/_modal.scss:10 | `var(--bs-body-bg)` | Direct, Runtime, Mixed |
| `--bs-nav-tabs-link-active-bg` | `var(--bs-body-bg)` | `.nav-tabs`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3846 | scss/_nav.scss:66<br>via scss/_nav.scss:59 | `var(--bs-body-bg)` | Direct, Runtime, Mixed |
| `--bs-offcanvas-bg` | `var(--bs-body-bg)` | `.offcanvas, .offcanvas-xxl, .offcanvas-xl, .offcanvas-lg, .offcanvas-md, .offcanvas-sm`<br>(none) | src/bootstrap/reference/css/bootstrap.css:6289 | scss/_offcanvas.scss:11<br>via scss/_offcanvas.scss:3 | `var(--bs-body-bg)` | Direct, Runtime, Mixed |
| `--bs-pagination-bg` | `var(--bs-body-bg)` | `.pagination`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4720 | scss/_pagination.scss:7<br>via scss/_pagination.scss:1 | `var(--bs-body-bg)` | Direct, Runtime, Mixed |
| `--bs-popover-bg` | `var(--bs-body-bg)` | `.popover`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5862 | scss/_popover.scss:6<br>via scss/_popover.scss:1 | `var(--bs-body-bg)` | Direct, Runtime, Mixed |
| `--bs-table-bg` | `var(--bs-body-bg)` | `.table`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1868 | scss/_tables.scss:13<br>via scss/_tables.scss:5 | `var(--bs-body-bg)` | Direct, Runtime, Mixed |
| `--bs-tooltip-color` | `var(--bs-body-bg)` | `.tooltip`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5763 | scss/_tooltip.scss:10<br>via scss/_tooltip.scss:2 | `var(--bs-body-bg)` | Direct, Runtime, Mixed |

<a id="c-body-bg-2"></a>
#### $body-bg – pattern 2: `rgba(var(--bs-body-bg-rgb), .85)`

2 contexts, 2 properties. Relationship: Direct, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-toast-bg` | `rgba(var(--bs-body-bg-rgb), 0.85)` | `.toast`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5394 | scss/_toasts.scss:10<br>via scss/_toasts.scss:1 | `rgba(var(--bs-body-bg-rgb), .85)` | Direct, Runtime, Mixed |
| `--bs-toast-header-bg` | `rgba(var(--bs-body-bg-rgb), 0.85)` | `.toast`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5400 | scss/_toasts.scss:16<br>via scss/_toasts.scss:1 | `rgba(var(--bs-body-bg-rgb), .85)` | Direct, Runtime, Mixed |

<a id="c-body-bg-3"></a>
#### $body-bg – pattern 3: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-body-bg` | `#fff` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:82<br>src/bootstrap/reference/css/bootstrap.css:83 | scss/_root.scss:63<br>via scss/_root.scss:1 | `$body-bg` | Direct |

<a id="c-body-bg-4"></a>
#### $body-bg – pattern 4: `to-rgb($ANCHOR)`

1 contexts, 1 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-body-bg-rgb` | `255, 255, 255` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:83<br>src/bootstrap/reference/css/bootstrap.css:84 | scss/_root.scss:64<br>via scss/_root.scss:1 | `to-rgb($body-bg)` | Derived, Compile-time |

<a id="c-body-bg-5"></a>
#### $body-bg – pattern 5: `var(--bs-accordion-bg)`

1 contexts, 1 properties. Relationship: Direct, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-accordion-btn-bg` | `var(--bs-accordion-bg)` | `.accordion`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4557 | scss/_accordion.scss:17<br>via scss/_accordion.scss:5 | `var(--bs-accordion-bg)` | Direct, Runtime, Mixed |

<a id="c-body-bg-dark"></a>
### `$body-bg-dark` (body / emphasis colour)

Definition: `$body-bg-dark: $gray-900` (`scss/_variables-dark.scss:44`). 2 contexts, 2 properties.

<a id="c-body-bg-dark-1"></a>
#### $body-bg-dark – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-body-bg` | `#212529` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:131<br>src/bootstrap/reference/css/bootstrap.css:132 | scss/_root.scss:139<br>via scss/mixins/_color-mode.scss:16 | `$body-bg-dark` | Direct |

<a id="c-body-bg-dark-2"></a>
#### $body-bg-dark – pattern 2: `to-rgb($ANCHOR)`

1 contexts, 1 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-body-bg-rgb` | `33, 37, 41` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:132<br>src/bootstrap/reference/css/bootstrap.css:133 | scss/_root.scss:140<br>via scss/mixins/_color-mode.scss:16 | `to-rgb($body-bg-dark)` | Derived, Compile-time |

<a id="c-body-color"></a>
### `$body-color` (body / emphasis colour)

Definition: `$body-color: $gray-900` (`scss/_variables.scss:438`). 17 contexts, 17 properties.

<a id="c-body-color-1"></a>
#### $body-color – pattern 1: `var(--bs-body-color)`

11 contexts, 11 properties. Relationship: Direct, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-accordion-btn-color` | `var(--bs-body-color)` | `.accordion`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4556 | scss/_accordion.scss:16<br>via scss/_accordion.scss:5 | `var(--bs-body-color)` | Direct, Runtime, Mixed |
| `--bs-accordion-color` | `var(--bs-body-color)` | `.accordion`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4547 | scss/_accordion.scss:7<br>via scss/_accordion.scss:5 | `var(--bs-body-color)` | Direct, Runtime, Mixed |
| `--bs-btn-color` | `var(--bs-body-color)` | `.btn`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2960 | scss/_buttons.scss:13<br>via scss/_buttons.scss:5 | `var(--bs-body-color)` | Direct, Runtime, Mixed |
| `--bs-dropdown-color` | `var(--bs-body-color)` | `.dropdown-menu`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3412 | scss/_dropdown.scss:27<br>via scss/_dropdown.scss:19 | `var(--bs-body-color)` | Direct, Runtime, Mixed |
| `--bs-dropdown-link-color` | `var(--bs-body-color)` | `.dropdown-menu`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3421 | scss/_dropdown.scss:36<br>via scss/_dropdown.scss:19 | `var(--bs-body-color)` | Direct, Runtime, Mixed |
| `--bs-dropdown-link-hover-color` | `var(--bs-body-color)` | `.dropdown-menu`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3422 | scss/_dropdown.scss:37<br>via scss/_dropdown.scss:19 | `var(--bs-body-color)` | Direct, Runtime, Mixed |
| `--bs-list-group-action-active-color` | `var(--bs-body-color)` | `.list-group`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5004 | scss/_list-group.scss:17<br>via scss/_list-group.scss:5 | `var(--bs-body-color)` | Direct, Runtime, Mixed |
| `--bs-list-group-color` | `var(--bs-body-color)` | `.list-group`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4994 | scss/_list-group.scss:7<br>via scss/_list-group.scss:5 | `var(--bs-body-color)` | Direct, Runtime, Mixed |
| `--bs-modal-color` | `var(--bs-body-color)` | `.modal`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5460 | scss/_modal.scss:16<br>via scss/_modal.scss:10 | `var(--bs-body-color)` | Direct, Runtime, Mixed |
| `--bs-offcanvas-color` | `var(--bs-body-color)` | `.offcanvas, .offcanvas-xxl, .offcanvas-xl, .offcanvas-lg, .offcanvas-md, .offcanvas-sm`<br>(none) | src/bootstrap/reference/css/bootstrap.css:6288 | scss/_offcanvas.scss:10<br>via scss/_offcanvas.scss:3 | `var(--bs-body-color)` | Direct, Runtime, Mixed |
| `--bs-popover-body-color` | `var(--bs-body-color)` | `.popover`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5875 | scss/_popover.scss:19<br>via scss/_popover.scss:1 | `var(--bs-body-color)` | Direct, Runtime, Mixed |

<a id="c-body-color-2"></a>
#### $body-color – pattern 2: `$ANCHOR`

2 contexts, 2 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-body-color` | `#212529` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:80<br>src/bootstrap/reference/css/bootstrap.css:81 | scss/_root.scss:61<br>via scss/_root.scss:1 | `$body-color` | Direct |
| `--bs-highlight-color` | `#212529` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:101<br>src/bootstrap/reference/css/bootstrap.css:102 | scss/_root.scss:94<br>via scss/_root.scss:1 | `$body-color` | Direct |

<a id="c-body-color-3"></a>
#### $body-color – pattern 3: `escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 16 16' fill='none' stroke='$ANCHOR' stroke-linecap='round' stroke-linejo…`

1 contexts, 1 properties. Relationship: Compile-time.

Full expression:

```scss
escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 16 16' fill='none' stroke='$ANCHOR' stroke-linecap='round' stroke-linejoin='round'><path d='m2 5 6 6 6-6'/></svg>"))
```

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-accordion-btn-icon` | `url("data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 16 16' fill='none' stroke='%23212529' stroke-linecap='round' stroke-linejoin='round'%3e%3cpath d='m2 5 6 6 6-6'/%3e%3c/svg%3e")` | `.accordion`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4558 | scss/_accordion.scss:18<br>via scss/_accordion.scss:5 | `escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 16 16' fill='none' stroke='$body-color' stroke-linecap='round' stroke-linejoin='round'><path d='m2 5 6 6 6-6'/></svg>"))` | Compile-time |

<a id="c-body-color-4"></a>
#### $body-color – pattern 4: `escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 30 30'><path stroke='rgba($ANCHOR, .75)' stroke-linecap='round' stroke-m…`

1 contexts, 1 properties. Relationship: Compile-time.

Full expression:

```scss
escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 30 30'><path stroke='rgba($ANCHOR, .75)' stroke-linecap='round' stroke-miterlimit='10' stroke-width='2' d='M4 7h22M4 15h22M4 23h22'/></svg>"))
```

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-navbar-toggler-icon-bg` | `url("data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 30 30'%3e%3cpath stroke='rgba%2833, 37, 41, 0.75%29' stroke-linecap='round' stroke-miterlimit='10' stroke-width='2' d='M4 7h22M4 15h22M4 23h22'/%3e%3c/svg%3e")` | `.navbar`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3948 | scss/_navbar.scss:23<br>via scss/_navbar.scss:6 | `escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 30 30'><path stroke='rgba($body-color, .75)' stroke-linecap='round' stroke-miterlimit='10' stroke-width='2' d='M4 7h22M4 15h22M4 23h22'/></svg>"))` | Compile-time |

<a id="c-body-color-5"></a>
#### $body-color – pattern 5: `rgba(var(--bs-body-color-rgb), .03)`

1 contexts, 1 properties. Relationship: Direct, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-card-cap-bg` | `rgba(var(--bs-body-color-rgb), 0.03)` | `.card`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4372 | scss/_card.scss:19<br>via scss/_card.scss:5 | `rgba(var(--bs-body-color-rgb), .03)` | Direct, Runtime, Mixed |

<a id="c-body-color-6"></a>
#### $body-color – pattern 6: `to-rgb($ANCHOR)`

1 contexts, 1 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-body-color-rgb` | `33, 37, 41` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:81<br>src/bootstrap/reference/css/bootstrap.css:82 | scss/_root.scss:62<br>via scss/_root.scss:1 | `to-rgb($body-color)` | Derived, Compile-time |

<a id="c-body-color-dark"></a>
### `$body-color-dark` (body / emphasis colour)

Definition: `$body-color-dark: $gray-300` (`scss/_variables-dark.scss:43`). 4 contexts, 4 properties.

<a id="c-body-color-dark-1"></a>
#### $body-color-dark – pattern 1: `$ANCHOR`

2 contexts, 2 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-body-color` | `#dee2e6` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:129<br>src/bootstrap/reference/css/bootstrap.css:130 | scss/_root.scss:137<br>via scss/mixins/_color-mode.scss:16 | `$body-color-dark` | Direct |
| `--bs-highlight-color` | `#dee2e6` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:173<br>src/bootstrap/reference/css/bootstrap.css:174 | scss/_root.scss:175<br>via scss/mixins/_color-mode.scss:16 | `$body-color-dark` | Direct |

<a id="c-body-color-dark-2"></a>
#### $body-color-dark – pattern 2: `escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 16 16'><path fill='none' stroke='$ANCHOR' stroke-linecap='round' stroke-…`

1 contexts, 1 properties. Relationship: Compile-time.

Full expression:

```scss
escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 16 16'><path fill='none' stroke='$ANCHOR' stroke-linecap='round' stroke-linejoin='round' stroke-width='2' d='m2 5 6 6 6-6'/></svg>"))
```

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-form-select-bg-img` | `url("data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 16 16'%3e%3cpath fill='none' stroke='%23dee2e6' stroke-linecap='round' stroke-linejoin='round' stroke-width='2' d='m2 5 6 6 6-6'/%3e%3c/svg%3e")` | `[data-bs-theme=dark] .form-select`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2368 | scss/forms/_form-select.scss:77<br>via scss/forms/_form-select.scss:76 | `escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 16 16'><path fill='none' stroke='$body-color-dark' stroke-linecap='round' stroke-linejoin='round' stroke-width='2' d='m2 5 6 6 6-6'/></svg>"))` | Compile-time |

<a id="c-body-color-dark-3"></a>
#### $body-color-dark – pattern 3: `to-rgb($ANCHOR)`

1 contexts, 1 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-body-color-rgb` | `222, 226, 230` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:130<br>src/bootstrap/reference/css/bootstrap.css:131 | scss/_root.scss:138<br>via scss/mixins/_color-mode.scss:16 | `to-rgb($body-color-dark)` | Derived, Compile-time |

<a id="c-body-emphasis-color"></a>
### `$body-emphasis-color` (body / emphasis colour)

Definition: `$body-emphasis-color: $black` (`scss/_variables.scss:447`). 36 contexts, 21 properties.

<a id="c-body-emphasis-color-1"></a>
#### $body-emphasis-color – pattern 1: `var(--bs-emphasis-color)`

24 contexts, 9 properties. Relationship: Direct, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-list-group-action-active-color` | `var(--bs-emphasis-color)` | `.list-group-item-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5303 | scss/_list-group.scss:192<br>via scss/_list-group.scss:186 | `var(--bs-emphasis-color)` | Runtime, Mixed |
| `--bs-list-group-action-active-color` | `var(--bs-emphasis-color)` | `.list-group-item-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5329 | scss/_list-group.scss:192<br>via scss/_list-group.scss:186 | `var(--bs-emphasis-color)` | Runtime, Mixed |
| `--bs-list-group-action-active-color` | `var(--bs-emphasis-color)` | `.list-group-item-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5277 | scss/_list-group.scss:192<br>via scss/_list-group.scss:186 | `var(--bs-emphasis-color)` | Runtime, Mixed |
| `--bs-list-group-action-active-color` | `var(--bs-emphasis-color)` | `.list-group-item-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5316 | scss/_list-group.scss:192<br>via scss/_list-group.scss:186 | `var(--bs-emphasis-color)` | Runtime, Mixed |
| `--bs-list-group-action-active-color` | `var(--bs-emphasis-color)` | `.list-group-item-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5238 | scss/_list-group.scss:192<br>via scss/_list-group.scss:186 | `var(--bs-emphasis-color)` | Runtime, Mixed |
| `--bs-list-group-action-active-color` | `var(--bs-emphasis-color)` | `.list-group-item-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5251 | scss/_list-group.scss:192<br>via scss/_list-group.scss:186 | `var(--bs-emphasis-color)` | Runtime, Mixed |
| `--bs-list-group-action-active-color` | `var(--bs-emphasis-color)` | `.list-group-item-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5264 | scss/_list-group.scss:192<br>via scss/_list-group.scss:186 | `var(--bs-emphasis-color)` | Runtime, Mixed |
| `--bs-list-group-action-active-color` | `var(--bs-emphasis-color)` | `.list-group-item-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5290 | scss/_list-group.scss:192<br>via scss/_list-group.scss:186 | `var(--bs-emphasis-color)` | Runtime, Mixed |
| `--bs-list-group-action-hover-color` | `var(--bs-emphasis-color)` | `.list-group`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5002 | scss/_list-group.scss:15<br>via scss/_list-group.scss:5 | `var(--bs-emphasis-color)` | Direct, Runtime, Mixed |
| `--bs-list-group-action-hover-color` | `var(--bs-emphasis-color)` | `.list-group-item-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5301 | scss/_list-group.scss:190<br>via scss/_list-group.scss:186 | `var(--bs-emphasis-color)` | Runtime, Mixed |
| `--bs-list-group-action-hover-color` | `var(--bs-emphasis-color)` | `.list-group-item-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5327 | scss/_list-group.scss:190<br>via scss/_list-group.scss:186 | `var(--bs-emphasis-color)` | Runtime, Mixed |
| `--bs-list-group-action-hover-color` | `var(--bs-emphasis-color)` | `.list-group-item-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5275 | scss/_list-group.scss:190<br>via scss/_list-group.scss:186 | `var(--bs-emphasis-color)` | Runtime, Mixed |
| `--bs-list-group-action-hover-color` | `var(--bs-emphasis-color)` | `.list-group-item-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5314 | scss/_list-group.scss:190<br>via scss/_list-group.scss:186 | `var(--bs-emphasis-color)` | Runtime, Mixed |
| `--bs-list-group-action-hover-color` | `var(--bs-emphasis-color)` | `.list-group-item-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5236 | scss/_list-group.scss:190<br>via scss/_list-group.scss:186 | `var(--bs-emphasis-color)` | Runtime, Mixed |
| `--bs-list-group-action-hover-color` | `var(--bs-emphasis-color)` | `.list-group-item-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5249 | scss/_list-group.scss:190<br>via scss/_list-group.scss:186 | `var(--bs-emphasis-color)` | Runtime, Mixed |
| `--bs-list-group-action-hover-color` | `var(--bs-emphasis-color)` | `.list-group-item-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5262 | scss/_list-group.scss:190<br>via scss/_list-group.scss:186 | `var(--bs-emphasis-color)` | Runtime, Mixed |
| `--bs-list-group-action-hover-color` | `var(--bs-emphasis-color)` | `.list-group-item-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5288 | scss/_list-group.scss:190<br>via scss/_list-group.scss:186 | `var(--bs-emphasis-color)` | Runtime, Mixed |
| `--bs-nav-tabs-link-active-color` | `var(--bs-emphasis-color)` | `.nav-tabs`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3845 | scss/_nav.scss:65<br>via scss/_nav.scss:59 | `var(--bs-emphasis-color)` | Direct, Runtime, Mixed |
| `--bs-nav-underline-link-active-color` | `var(--bs-emphasis-color)` | `.nav-underline`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3889 | scss/_nav.scss:132<br>via scss/_nav.scss:128 | `var(--bs-emphasis-color)` | Direct, Runtime, Mixed |
| `--bs-table-active-color` | `var(--bs-emphasis-color)` | `.table`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1873 | scss/_tables.scss:18<br>via scss/_tables.scss:5 | `var(--bs-emphasis-color)` | Direct, Runtime, Mixed |
| `--bs-table-color` | `var(--bs-emphasis-color)` | `.table`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1867 | scss/_tables.scss:12<br>via scss/_tables.scss:5 | `var(--bs-emphasis-color)` | Direct, Runtime, Mixed |
| `--bs-table-hover-color` | `var(--bs-emphasis-color)` | `.table`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1875 | scss/_tables.scss:20<br>via scss/_tables.scss:5 | `var(--bs-emphasis-color)` | Direct, Runtime, Mixed |
| `--bs-table-striped-color` | `var(--bs-emphasis-color)` | `.table`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1871 | scss/_tables.scss:16<br>via scss/_tables.scss:5 | `var(--bs-emphasis-color)` | Direct, Runtime, Mixed |
| `--bs-tooltip-bg` | `var(--bs-emphasis-color)` | `.tooltip`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5764 | scss/_tooltip.scss:11<br>via scss/_tooltip.scss:2 | `var(--bs-emphasis-color)` | Direct, Runtime, Mixed |

<a id="c-body-emphasis-color-2"></a>
#### $body-emphasis-color – pattern 2: `rgba(var(--bs-emphasis-color-rgb), 1)`

3 contexts, 3 properties. Relationship: Direct, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-navbar-active-color` | `rgba(var(--bs-emphasis-color-rgb), 1)` | `.navbar`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3938 | scss/_navbar.scss:13<br>via scss/_navbar.scss:6 | `rgba(var(--bs-emphasis-color-rgb), 1)` | Direct, Runtime, Mixed |
| `--bs-navbar-brand-color` | `rgba(var(--bs-emphasis-color-rgb), 1)` | `.navbar`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3942 | scss/_navbar.scss:17<br>via scss/_navbar.scss:6 | `rgba(var(--bs-emphasis-color-rgb), 1)` | Direct, Runtime, Mixed |
| `--bs-navbar-brand-hover-color` | `rgba(var(--bs-emphasis-color-rgb), 1)` | `.navbar`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3943 | scss/_navbar.scss:18<br>via scss/_navbar.scss:6 | `rgba(var(--bs-emphasis-color-rgb), 1)` | Direct, Runtime, Mixed |

<a id="c-body-emphasis-color-3"></a>
#### $body-emphasis-color – pattern 3: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-emphasis-color` | `#000` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:84<br>src/bootstrap/reference/css/bootstrap.css:85 | scss/_root.scss:66<br>via scss/_root.scss:1 | `$body-emphasis-color` | Direct |

<a id="c-body-emphasis-color-4"></a>
#### $body-emphasis-color – pattern 4: `rgba(var(--bs-emphasis-color-rgb), $table-active-bg-factor)`

1 contexts, 1 properties. Relationship: Direct, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-active-bg` | `rgba(var(--bs-emphasis-color-rgb), 0.1)` | `.table`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1874 | scss/_tables.scss:19<br>via scss/_tables.scss:5 | `rgba(var(--bs-emphasis-color-rgb), $table-active-bg-factor)` | Direct, Runtime, Mixed |

<a id="c-body-emphasis-color-5"></a>
#### $body-emphasis-color – pattern 5: `rgba(var(--bs-emphasis-color-rgb), $table-hover-bg-factor)`

1 contexts, 1 properties. Relationship: Direct, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-hover-bg` | `rgba(var(--bs-emphasis-color-rgb), 0.075)` | `.table`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1876 | scss/_tables.scss:21<br>via scss/_tables.scss:5 | `rgba(var(--bs-emphasis-color-rgb), $table-hover-bg-factor)` | Direct, Runtime, Mixed |

<a id="c-body-emphasis-color-6"></a>
#### $body-emphasis-color – pattern 6: `rgba(var(--bs-emphasis-color-rgb), $table-striped-bg-factor)`

1 contexts, 1 properties. Relationship: Direct, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-table-striped-bg` | `rgba(var(--bs-emphasis-color-rgb), 0.05)` | `.table`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1872 | scss/_tables.scss:17<br>via scss/_tables.scss:5 | `rgba(var(--bs-emphasis-color-rgb), $table-striped-bg-factor)` | Direct, Runtime, Mixed |

<a id="c-body-emphasis-color-7"></a>
#### $body-emphasis-color – pattern 7: `rgba(var(--bs-emphasis-color-rgb), .15)`

1 contexts, 1 properties. Relationship: Direct, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-navbar-toggler-border-color` | `rgba(var(--bs-emphasis-color-rgb), 0.15)` | `.navbar`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3949 | scss/_navbar.scss:24<br>via scss/_navbar.scss:6 | `rgba(var(--bs-emphasis-color-rgb), .15)` | Direct, Runtime, Mixed |

<a id="c-body-emphasis-color-8"></a>
#### $body-emphasis-color – pattern 8: `rgba(var(--bs-emphasis-color-rgb), .3)`

1 contexts, 1 properties. Relationship: Direct, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-navbar-disabled-color` | `rgba(var(--bs-emphasis-color-rgb), 0.3)` | `.navbar`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3937 | scss/_navbar.scss:12<br>via scss/_navbar.scss:6 | `rgba(var(--bs-emphasis-color-rgb), .3)` | Direct, Runtime, Mixed |

<a id="c-body-emphasis-color-9"></a>
#### $body-emphasis-color – pattern 9: `rgba(var(--bs-emphasis-color-rgb), .65)`

1 contexts, 1 properties. Relationship: Direct, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-navbar-color` | `rgba(var(--bs-emphasis-color-rgb), 0.65)` | `.navbar`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3935 | scss/_navbar.scss:10<br>via scss/_navbar.scss:6 | `rgba(var(--bs-emphasis-color-rgb), .65)` | Direct, Runtime, Mixed |

<a id="c-body-emphasis-color-10"></a>
#### $body-emphasis-color – pattern 10: `rgba(var(--bs-emphasis-color-rgb), .8)`

1 contexts, 1 properties. Relationship: Direct, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-navbar-hover-color` | `rgba(var(--bs-emphasis-color-rgb), 0.8)` | `.navbar`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3936 | scss/_navbar.scss:11<br>via scss/_navbar.scss:6 | `rgba(var(--bs-emphasis-color-rgb), .8)` | Direct, Runtime, Mixed |

<a id="c-body-emphasis-color-11"></a>
#### $body-emphasis-color – pattern 11: `to-rgb($ANCHOR)`

1 contexts, 1 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-emphasis-color-rgb` | `0, 0, 0` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:85<br>src/bootstrap/reference/css/bootstrap.css:86 | scss/_root.scss:67<br>via scss/_root.scss:1 | `to-rgb($body-emphasis-color)` | Derived, Compile-time |

<a id="c-body-emphasis-color-dark"></a>
### `$body-emphasis-color-dark` (body / emphasis colour)

Definition: `$body-emphasis-color-dark: $white` (`scss/_variables-dark.scss:49`). 2 contexts, 2 properties.

<a id="c-body-emphasis-color-dark-1"></a>
#### $body-emphasis-color-dark – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-emphasis-color` | `#fff` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:133<br>src/bootstrap/reference/css/bootstrap.css:134 | scss/_root.scss:142<br>via scss/mixins/_color-mode.scss:16 | `$body-emphasis-color-dark` | Direct |

<a id="c-body-emphasis-color-dark-2"></a>
#### $body-emphasis-color-dark – pattern 2: `to-rgb($ANCHOR)`

1 contexts, 1 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-emphasis-color-rgb` | `255, 255, 255` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:134<br>src/bootstrap/reference/css/bootstrap.css:135 | scss/_root.scss:143<br>via scss/mixins/_color-mode.scss:16 | `to-rgb($body-emphasis-color-dark)` | Derived, Compile-time |

<a id="c-body-secondary-bg"></a>
### `$body-secondary-bg` (body / emphasis colour)

Definition: `$body-secondary-bg: $gray-200` (`scss/_variables.scss:442`). 8 contexts, 8 properties.

<a id="c-body-secondary-bg-1"></a>
#### $body-secondary-bg – pattern 1: `var(--bs-secondary-bg)`

5 contexts, 5 properties. Relationship: Direct, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-list-group-action-active-bg` | `var(--bs-secondary-bg)` | `.list-group`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5005 | scss/_list-group.scss:18<br>via scss/_list-group.scss:5 | `var(--bs-secondary-bg)` | Direct, Runtime, Mixed |
| `--bs-pagination-disabled-bg` | `var(--bs-secondary-bg)` | `.pagination`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4734 | scss/_pagination.scss:21<br>via scss/_pagination.scss:1 | `var(--bs-secondary-bg)` | Direct, Runtime, Mixed |
| `--bs-pagination-focus-bg` | `var(--bs-secondary-bg)` | `.pagination`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4728 | scss/_pagination.scss:15<br>via scss/_pagination.scss:1 | `var(--bs-secondary-bg)` | Direct, Runtime, Mixed |
| `--bs-popover-header-bg` | `var(--bs-secondary-bg)` | `.popover`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5872 | scss/_popover.scss:16<br>via scss/_popover.scss:1 | `var(--bs-secondary-bg)` | Direct, Runtime, Mixed |
| `--bs-progress-bg` | `var(--bs-secondary-bg)` | `.progress, .progress-stacked`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4940 | scss/_progress.scss:16<br>via scss/_progress.scss:11 | `var(--bs-secondary-bg)` | Direct, Runtime, Mixed |

<a id="c-body-secondary-bg-2"></a>
#### $body-secondary-bg – pattern 2: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-secondary-bg` | `#e9ecef` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:88<br>src/bootstrap/reference/css/bootstrap.css:89 | scss/_root.scss:71<br>via scss/_root.scss:1 | `$body-secondary-bg` | Direct |

<a id="c-body-secondary-bg-3"></a>
#### $body-secondary-bg – pattern 3: `(var(--bs-secondary-bg) var(--bs-secondary-bg) var(--bs-border-color))`

1 contexts, 1 properties. Relationship: Direct, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-nav-tabs-link-hover-border-color` | `var(--bs-secondary-bg) var(--bs-secondary-bg) var(--bs-border-color)` | `.nav-tabs`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3844 | scss/_nav.scss:64<br>via scss/_nav.scss:59 | `(var(--bs-secondary-bg) var(--bs-secondary-bg) var(--bs-border-color))` | Direct, Runtime, Mixed |

<a id="c-body-secondary-bg-4"></a>
#### $body-secondary-bg – pattern 4: `to-rgb($ANCHOR)`

1 contexts, 1 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-secondary-bg-rgb` | `233, 236, 239` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:89<br>src/bootstrap/reference/css/bootstrap.css:90 | scss/_root.scss:72<br>via scss/_root.scss:1 | `to-rgb($body-secondary-bg)` | Derived, Compile-time |

<a id="c-body-secondary-bg-dark"></a>
### `$body-secondary-bg-dark` (body / emphasis colour)

Definition: `$body-secondary-bg-dark: $gray-800` (`scss/_variables-dark.scss:46`). 2 contexts, 2 properties.

<a id="c-body-secondary-bg-dark-1"></a>
#### $body-secondary-bg-dark – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-secondary-bg` | `#343a40` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:137<br>src/bootstrap/reference/css/bootstrap.css:138 | scss/_root.scss:147<br>via scss/mixins/_color-mode.scss:16 | `$body-secondary-bg-dark` | Direct |

<a id="c-body-secondary-bg-dark-2"></a>
#### $body-secondary-bg-dark – pattern 2: `to-rgb($ANCHOR)`

1 contexts, 1 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-secondary-bg-rgb` | `52, 58, 64` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:138<br>src/bootstrap/reference/css/bootstrap.css:139 | scss/_root.scss:148<br>via scss/mixins/_color-mode.scss:16 | `to-rgb($body-secondary-bg-dark)` | Derived, Compile-time |

<a id="c-body-secondary-color"></a>
### `$body-secondary-color` (body / emphasis colour)

Definition: `$body-secondary-color: rgba($body-color, .75)` (`scss/_variables.scss:441`). 9 contexts, 9 properties.

<a id="c-body-secondary-color-1"></a>
#### $body-secondary-color – pattern 1: `var(--bs-secondary-color)`

7 contexts, 7 properties. Relationship: Direct, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-breadcrumb-divider-color` | `var(--bs-secondary-color)` | `.breadcrumb`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4689 | scss/_breadcrumb.scss:9<br>via scss/_breadcrumb.scss:1 | `var(--bs-secondary-color)` | Direct, Runtime, Mixed |
| `--bs-breadcrumb-item-active-color` | `var(--bs-secondary-color)` | `.breadcrumb`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4691 | scss/_breadcrumb.scss:11<br>via scss/_breadcrumb.scss:1 | `var(--bs-secondary-color)` | Direct, Runtime, Mixed |
| `--bs-list-group-action-color` | `var(--bs-secondary-color)` | `.list-group`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5001 | scss/_list-group.scss:14<br>via scss/_list-group.scss:5 | `var(--bs-secondary-color)` | Direct, Runtime, Mixed |
| `--bs-list-group-disabled-color` | `var(--bs-secondary-color)` | `.list-group`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5006 | scss/_list-group.scss:19<br>via scss/_list-group.scss:5 | `var(--bs-secondary-color)` | Direct, Runtime, Mixed |
| `--bs-nav-link-disabled-color` | `var(--bs-secondary-color)` | `.nav`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3803 | scss/_nav.scss:14<br>via scss/_nav.scss:6 | `var(--bs-secondary-color)` | Direct, Runtime, Mixed |
| `--bs-pagination-disabled-color` | `var(--bs-secondary-color)` | `.pagination`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4733 | scss/_pagination.scss:20<br>via scss/_pagination.scss:1 | `var(--bs-secondary-color)` | Direct, Runtime, Mixed |
| `--bs-toast-header-color` | `var(--bs-secondary-color)` | `.toast`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5399 | scss/_toasts.scss:15<br>via scss/_toasts.scss:1 | `var(--bs-secondary-color)` | Direct, Runtime, Mixed |

<a id="c-body-secondary-color-2"></a>
#### $body-secondary-color – pattern 2: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-secondary-color` | `rgba(33, 37, 41, 0.75)` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:86<br>src/bootstrap/reference/css/bootstrap.css:87 | scss/_root.scss:69<br>via scss/_root.scss:1 | `$body-secondary-color` | Direct |

<a id="c-body-secondary-color-3"></a>
#### $body-secondary-color – pattern 3: `to-rgb($ANCHOR)`

1 contexts, 1 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-secondary-color-rgb` | `33, 37, 41` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:87<br>src/bootstrap/reference/css/bootstrap.css:88 | scss/_root.scss:70<br>via scss/_root.scss:1 | `to-rgb($body-secondary-color)` | Derived, Compile-time |

<a id="c-body-secondary-color-dark"></a>
### `$body-secondary-color-dark` (body / emphasis colour)

Definition: `$body-secondary-color-dark: rgba($body-color-dark, .75)` (`scss/_variables-dark.scss:45`). 2 contexts, 2 properties.

<a id="c-body-secondary-color-dark-1"></a>
#### $body-secondary-color-dark – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-secondary-color` | `rgba(222, 226, 230, 0.75)` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:135<br>src/bootstrap/reference/css/bootstrap.css:136 | scss/_root.scss:145<br>via scss/mixins/_color-mode.scss:16 | `$body-secondary-color-dark` | Direct |

<a id="c-body-secondary-color-dark-2"></a>
#### $body-secondary-color-dark – pattern 2: `to-rgb($ANCHOR)`

1 contexts, 1 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-secondary-color-rgb` | `222, 226, 230` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:136<br>src/bootstrap/reference/css/bootstrap.css:137 | scss/_root.scss:146<br>via scss/mixins/_color-mode.scss:16 | `to-rgb($body-secondary-color-dark)` | Derived, Compile-time |

<a id="c-body-tertiary-bg"></a>
### `$body-tertiary-bg` (body / emphasis colour)

Definition: `$body-tertiary-bg: $gray-100` (`scss/_variables.scss:445`). 5 contexts, 5 properties.

<a id="c-body-tertiary-bg-1"></a>
#### $body-tertiary-bg – pattern 1: `var(--bs-tertiary-bg)`

3 contexts, 3 properties. Relationship: Direct, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-dropdown-link-hover-bg` | `var(--bs-tertiary-bg)` | `.dropdown-menu`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3423 | scss/_dropdown.scss:38<br>via scss/_dropdown.scss:19 | `var(--bs-tertiary-bg)` | Direct, Runtime, Mixed |
| `--bs-list-group-action-hover-bg` | `var(--bs-tertiary-bg)` | `.list-group`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5003 | scss/_list-group.scss:16<br>via scss/_list-group.scss:5 | `var(--bs-tertiary-bg)` | Direct, Runtime, Mixed |
| `--bs-pagination-hover-bg` | `var(--bs-tertiary-bg)` | `.pagination`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4725 | scss/_pagination.scss:12<br>via scss/_pagination.scss:1 | `var(--bs-tertiary-bg)` | Direct, Runtime, Mixed |

<a id="c-body-tertiary-bg-2"></a>
#### $body-tertiary-bg – pattern 2: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-tertiary-bg` | `#f8f9fa` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:92<br>src/bootstrap/reference/css/bootstrap.css:93 | scss/_root.scss:76<br>via scss/_root.scss:1 | `$body-tertiary-bg` | Direct |

<a id="c-body-tertiary-bg-3"></a>
#### $body-tertiary-bg – pattern 3: `to-rgb($ANCHOR)`

1 contexts, 1 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-tertiary-bg-rgb` | `248, 249, 250` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:93<br>src/bootstrap/reference/css/bootstrap.css:94 | scss/_root.scss:77<br>via scss/_root.scss:1 | `to-rgb($body-tertiary-bg)` | Derived, Compile-time |

<a id="c-body-tertiary-bg-dark"></a>
### `$body-tertiary-bg-dark` (body / emphasis colour)

Definition: `$body-tertiary-bg-dark: mix($gray-800, $gray-900, 50%)` (`scss/_variables-dark.scss:48`). 2 contexts, 2 properties.

<a id="c-body-tertiary-bg-dark-1"></a>
#### $body-tertiary-bg-dark – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-tertiary-bg` | `#2b3035` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:141<br>src/bootstrap/reference/css/bootstrap.css:142 | scss/_root.scss:152<br>via scss/mixins/_color-mode.scss:16 | `$body-tertiary-bg-dark` | Direct |

<a id="c-body-tertiary-bg-dark-2"></a>
#### $body-tertiary-bg-dark – pattern 2: `to-rgb($ANCHOR)`

1 contexts, 1 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-tertiary-bg-rgb` | `43, 48, 53` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:142<br>src/bootstrap/reference/css/bootstrap.css:143 | scss/_root.scss:153<br>via scss/mixins/_color-mode.scss:16 | `to-rgb($body-tertiary-bg-dark)` | Derived, Compile-time |

<a id="c-body-tertiary-color"></a>
### `$body-tertiary-color` (body / emphasis colour)

Definition: `$body-tertiary-color: rgba($body-color, .5)` (`scss/_variables.scss:444`). 3 contexts, 3 properties.

<a id="c-body-tertiary-color-1"></a>
#### $body-tertiary-color – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-tertiary-color` | `rgba(33, 37, 41, 0.5)` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:90<br>src/bootstrap/reference/css/bootstrap.css:91 | scss/_root.scss:74<br>via scss/_root.scss:1 | `$body-tertiary-color` | Direct |

<a id="c-body-tertiary-color-2"></a>
#### $body-tertiary-color – pattern 2: `to-rgb($ANCHOR)`

1 contexts, 1 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-tertiary-color-rgb` | `33, 37, 41` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:91<br>src/bootstrap/reference/css/bootstrap.css:92 | scss/_root.scss:75<br>via scss/_root.scss:1 | `to-rgb($body-tertiary-color)` | Derived, Compile-time |

<a id="c-body-tertiary-color-3"></a>
#### $body-tertiary-color – pattern 3: `var(--bs-tertiary-color)`

1 contexts, 1 properties. Relationship: Direct, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-dropdown-link-disabled-color` | `var(--bs-tertiary-color)` | `.dropdown-menu`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3426 | scss/_dropdown.scss:41<br>via scss/_dropdown.scss:19 | `var(--bs-tertiary-color)` | Direct, Runtime, Mixed |

<a id="c-body-tertiary-color-dark"></a>
### `$body-tertiary-color-dark` (body / emphasis colour)

Definition: `$body-tertiary-color-dark: rgba($body-color-dark, .5)` (`scss/_variables-dark.scss:47`). 2 contexts, 2 properties.

<a id="c-body-tertiary-color-dark-1"></a>
#### $body-tertiary-color-dark – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-tertiary-color` | `rgba(222, 226, 230, 0.5)` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:139<br>src/bootstrap/reference/css/bootstrap.css:140 | scss/_root.scss:150<br>via scss/mixins/_color-mode.scss:16 | `$body-tertiary-color-dark` | Direct |

<a id="c-body-tertiary-color-dark-2"></a>
#### $body-tertiary-color-dark – pattern 2: `to-rgb($ANCHOR)`

1 contexts, 1 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-tertiary-color-rgb` | `222, 226, 230` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:140<br>src/bootstrap/reference/css/bootstrap.css:141 | scss/_root.scss:151<br>via scss/mixins/_color-mode.scss:16 | `to-rgb($body-tertiary-color-dark)` | Derived, Compile-time |

<a id="c-border-color"></a>
### `$border-color` (border colour)

Definition: `$border-color: $gray-300` (`scss/_variables.scss:542`). 11 contexts, 11 properties.

<a id="c-border-color-1"></a>
#### $border-color – pattern 1: `var(--bs-border-color)`

9 contexts, 9 properties. Relationship: Direct, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-accordion-border-color` | `var(--bs-border-color)` | `.accordion`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4550 | scss/_accordion.scss:10<br>via scss/_accordion.scss:5 | `var(--bs-border-color)` | Direct, Runtime, Mixed |
| `--bs-list-group-border-color` | `var(--bs-border-color)` | `.list-group`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4996 | scss/_list-group.scss:9<br>via scss/_list-group.scss:5 | `var(--bs-border-color)` | Direct, Runtime, Mixed |
| `--bs-modal-footer-border-color` | `var(--bs-border-color)` | `.modal`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5475 | scss/_modal.scss:31<br>via scss/_modal.scss:10 | `var(--bs-border-color)` | Direct, Runtime, Mixed |
| `--bs-modal-header-border-color` | `var(--bs-border-color)` | `.modal`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5470 | scss/_modal.scss:26<br>via scss/_modal.scss:10 | `var(--bs-border-color)` | Direct, Runtime, Mixed |
| `--bs-nav-tabs-border-color` | `var(--bs-border-color)` | `.nav-tabs`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3842 | scss/_nav.scss:62<br>via scss/_nav.scss:59 | `var(--bs-border-color)` | Direct, Runtime, Mixed |
| `--bs-pagination-border-color` | `var(--bs-border-color)` | `.pagination`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4722 | scss/_pagination.scss:9<br>via scss/_pagination.scss:1 | `var(--bs-border-color)` | Direct, Runtime, Mixed |
| `--bs-pagination-disabled-border-color` | `var(--bs-border-color)` | `.pagination`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4735 | scss/_pagination.scss:22<br>via scss/_pagination.scss:1 | `var(--bs-border-color)` | Direct, Runtime, Mixed |
| `--bs-pagination-hover-border-color` | `var(--bs-border-color)` | `.pagination`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4726 | scss/_pagination.scss:13<br>via scss/_pagination.scss:1 | `var(--bs-border-color)` | Direct, Runtime, Mixed |
| `--bs-table-border-color` | `var(--bs-border-color)` | `.table`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1869 | scss/_tables.scss:14<br>via scss/_tables.scss:5 | `var(--bs-border-color)` | Direct, Runtime, Mixed |

<a id="c-border-color-2"></a>
#### $border-color – pattern 2: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-border-color` | `#dee2e6` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:105<br>src/bootstrap/reference/css/bootstrap.css:106 | scss/_root.scss:100<br>via scss/_root.scss:1 | `$border-color` | Direct |

<a id="c-border-color-3"></a>
#### $border-color – pattern 3: `(var(--bs-border-color) var(--bs-border-color) var(--bs-body-bg))`

1 contexts, 1 properties. Relationship: Direct, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-nav-tabs-link-active-border-color` | `var(--bs-border-color) var(--bs-border-color) var(--bs-body-bg)` | `.nav-tabs`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3847 | scss/_nav.scss:67<br>via scss/_nav.scss:59 | `(var(--bs-border-color) var(--bs-border-color) var(--bs-body-bg))` | Direct, Runtime, Mixed |

<a id="c-border-color-dark"></a>
### `$border-color-dark` (border colour)

Definition: `$border-color-dark: $gray-700` (`scss/_variables-dark.scss:50`). 1 contexts, 1 properties.

<a id="c-border-color-dark-1"></a>
#### $border-color-dark – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-border-color` | `#495057` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:175<br>src/bootstrap/reference/css/bootstrap.css:176 | scss/_root.scss:178<br>via scss/mixins/_color-mode.scss:16 | `$border-color-dark` | Direct |

<a id="c-border-color-translucent"></a>
### `$border-color-translucent` (border colour)

Definition: `$border-color-translucent: rgba($black, .175)` (`scss/_variables.scss:543`). 12 contexts, 10 properties.

<a id="c-border-color-translucent-1"></a>
#### $border-color-translucent – pattern 1: `var(--bs-border-color-translucent)`

10 contexts, 8 properties. Relationship: Direct, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-card-border-color` | `var(--bs-border-color-translucent)` | `.card`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4366 | scss/_card.scss:13<br>via scss/_card.scss:5 | `var(--bs-border-color-translucent)` | Direct, Runtime, Mixed |
| `--bs-dropdown-border-color` | `var(--bs-border-color-translucent)` | `.dropdown-menu`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3414 | scss/_dropdown.scss:29<br>via scss/_dropdown.scss:19 | `var(--bs-border-color-translucent)` | Direct, Runtime, Mixed |
| `--bs-dropdown-border-color` | `var(--bs-border-color-translucent)` | `.dropdown-menu-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3684 | scss/_dropdown.scss:239<br>via scss/_dropdown.scss:235 | `var(--bs-border-color-translucent)` | Direct, Runtime, Mixed |
| `--bs-dropdown-divider-bg` | `var(--bs-border-color-translucent)` | `.dropdown-menu`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3418 | scss/_dropdown.scss:33<br>via scss/_dropdown.scss:19 | `var(--bs-border-color-translucent)` | Direct, Runtime, Mixed |
| `--bs-dropdown-divider-bg` | `var(--bs-border-color-translucent)` | `.dropdown-menu-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3688 | scss/_dropdown.scss:243<br>via scss/_dropdown.scss:235 | `var(--bs-border-color-translucent)` | Direct, Runtime, Mixed |
| `--bs-modal-border-color` | `var(--bs-border-color-translucent)` | `.modal`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5462 | scss/_modal.scss:18<br>via scss/_modal.scss:10 | `var(--bs-border-color-translucent)` | Direct, Runtime, Mixed |
| `--bs-offcanvas-border-color` | `var(--bs-border-color-translucent)` | `.offcanvas, .offcanvas-xxl, .offcanvas-xl, .offcanvas-lg, .offcanvas-md, .offcanvas-sm`<br>(none) | src/bootstrap/reference/css/bootstrap.css:6291 | scss/_offcanvas.scss:13<br>via scss/_offcanvas.scss:3 | `var(--bs-border-color-translucent)` | Direct, Runtime, Mixed |
| `--bs-popover-border-color` | `var(--bs-border-color-translucent)` | `.popover`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5864 | scss/_popover.scss:8<br>via scss/_popover.scss:1 | `var(--bs-border-color-translucent)` | Direct, Runtime, Mixed |
| `--bs-toast-border-color` | `var(--bs-border-color-translucent)` | `.toast`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5396 | scss/_toasts.scss:12<br>via scss/_toasts.scss:1 | `var(--bs-border-color-translucent)` | Direct, Runtime, Mixed |
| `--bs-toast-header-border-color` | `var(--bs-border-color-translucent)` | `.toast`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5401 | scss/_toasts.scss:17<br>via scss/_toasts.scss:1 | `var(--bs-border-color-translucent)` | Direct, Runtime, Mixed |

<a id="c-border-color-translucent-2"></a>
#### $border-color-translucent – pattern 2: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-border-color-translucent` | `rgba(0, 0, 0, 0.175)` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:106<br>src/bootstrap/reference/css/bootstrap.css:107 | scss/_root.scss:101<br>via scss/_root.scss:1 | `$border-color-translucent` | Direct |

<a id="c-border-color-translucent-3"></a>
#### $border-color-translucent – pattern 3: `var(--bs-popover-border-color)`

1 contexts, 1 properties. Relationship: Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-popover-arrow-border` | `var(--bs-popover-border-color)` | `.popover`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5878 | scss/_popover.scss:22<br>via scss/_popover.scss:1 | `var(--bs-popover-border-color)` | Runtime, Mixed |

<a id="c-border-color-translucent-dark"></a>
### `$border-color-translucent-dark` (border colour)

Definition: `$border-color-translucent-dark: rgba($white, .15)` (`scss/_variables-dark.scss:51`). 1 contexts, 1 properties.

<a id="c-border-color-translucent-dark-1"></a>
#### $border-color-translucent-dark – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-border-color-translucent` | `rgba(255, 255, 255, 0.15)` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:176<br>src/bootstrap/reference/css/bootstrap.css:177 | scss/_root.scss:179<br>via scss/mixins/_color-mode.scss:16 | `$border-color-translucent-dark` | Direct |

<a id="c-box-shadow"></a>
### `$box-shadow` (shadow)

Definition: `$box-shadow: 0 .5rem 1rem rgba($black, .15)` (`scss/_variables.scss:559`). 5 contexts, 5 properties.

<a id="c-box-shadow-1"></a>
#### $box-shadow – pattern 1: `var(--bs-box-shadow)`

4 contexts, 4 properties. Relationship: Direct, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-dropdown-box-shadow` | `var(--bs-box-shadow)` | `.dropdown-menu`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3420 | scss/_dropdown.scss:35<br>via scss/_dropdown.scss:19 | `var(--bs-box-shadow)` | Direct, Runtime, Mixed |
| `--bs-modal-box-shadow` | `var(--bs-box-shadow)` | `.modal`<br>@media (min-width: 576px) | src/bootstrap/reference/css/bootstrap.css:5608 | scss/_modal.scss:183<br>via scss/_modal.scss:181 | `var(--bs-box-shadow)` | Direct, Runtime, Mixed |
| `--bs-popover-box-shadow` | `var(--bs-box-shadow)` | `.popover`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5867 | scss/_popover.scss:11<br>via scss/_popover.scss:1 | `var(--bs-box-shadow)` | Direct, Runtime, Mixed |
| `--bs-toast-box-shadow` | `var(--bs-box-shadow)` | `.toast`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5398 | scss/_toasts.scss:14<br>via scss/_toasts.scss:1 | `var(--bs-box-shadow)` | Direct, Runtime, Mixed |

<a id="c-box-shadow-2"></a>
#### $box-shadow – pattern 2: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-box-shadow` | `0 0.5rem 1rem rgba(0, 0, 0, 0.15)` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:114<br>src/bootstrap/reference/css/bootstrap.css:115 | scss/_root.scss:112<br>via scss/_root.scss:1 | `$box-shadow` | Direct |

<a id="c-box-shadow-inset"></a>
### `$box-shadow-inset` (shadow)

Definition: `$box-shadow-inset: inset 0 1px 2px rgba($black, .075)` (`scss/_variables.scss:562`). 2 contexts, 2 properties.

<a id="c-box-shadow-inset-1"></a>
#### $box-shadow-inset – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-box-shadow-inset` | `inset 0 1px 2px rgba(0, 0, 0, 0.075)` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:117<br>src/bootstrap/reference/css/bootstrap.css:118 | scss/_root.scss:115<br>via scss/_root.scss:1 | `$box-shadow-inset` | Direct |

<a id="c-box-shadow-inset-2"></a>
#### $box-shadow-inset – pattern 2: `var(--bs-box-shadow-inset)`

1 contexts, 1 properties. Relationship: Direct, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-progress-box-shadow` | `var(--bs-box-shadow-inset)` | `.progress, .progress-stacked`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4942 | scss/_progress.scss:18<br>via scss/_progress.scss:11 | `var(--bs-box-shadow-inset)` | Direct, Runtime, Mixed |

<a id="c-box-shadow-lg"></a>
### `$box-shadow-lg` (shadow)

Definition: `$box-shadow-lg: 0 1rem 3rem rgba($black, .175)` (`scss/_variables.scss:561`). 1 contexts, 1 properties.

<a id="c-box-shadow-lg-1"></a>
#### $box-shadow-lg – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-box-shadow-lg` | `0 1rem 3rem rgba(0, 0, 0, 0.175)` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:116<br>src/bootstrap/reference/css/bootstrap.css:117 | scss/_root.scss:114<br>via scss/_root.scss:1 | `$box-shadow-lg` | Direct |

<a id="c-box-shadow-sm"></a>
### `$box-shadow-sm` (shadow)

Definition: `$box-shadow-sm: 0 .125rem .25rem rgba($black, .075)` (`scss/_variables.scss:560`). 3 contexts, 3 properties.

<a id="c-box-shadow-sm-1"></a>
#### $box-shadow-sm – pattern 1: `var(--bs-box-shadow-sm)`

2 contexts, 2 properties. Relationship: Direct, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-modal-box-shadow` | `var(--bs-box-shadow-sm)` | `.modal`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5465 | scss/_modal.scss:21<br>via scss/_modal.scss:10 | `var(--bs-box-shadow-sm)` | Direct, Runtime, Mixed |
| `--bs-offcanvas-box-shadow` | `var(--bs-box-shadow-sm)` | `.offcanvas, .offcanvas-xxl, .offcanvas-xl, .offcanvas-lg, .offcanvas-md, .offcanvas-sm`<br>(none) | src/bootstrap/reference/css/bootstrap.css:6292 | scss/_offcanvas.scss:14<br>via scss/_offcanvas.scss:3 | `var(--bs-box-shadow-sm)` | Direct, Runtime, Mixed |

<a id="c-box-shadow-sm-2"></a>
#### $box-shadow-sm – pattern 2: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-box-shadow-sm` | `0 0.125rem 0.25rem rgba(0, 0, 0, 0.075)` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:115<br>src/bootstrap/reference/css/bootstrap.css:116 | scss/_root.scss:113<br>via scss/_root.scss:1 | `$box-shadow-sm` | Direct |

<a id="c-black"></a>
### `$black` (palette)

Definition: `$black: #000` (`scss/_variables.scss:19`). 26 contexts, 9 properties.

<a id="c-black-1"></a>
#### $black – pattern 1: `(inset 0 3px 5px rgba($ANCHOR, .125))`

16 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-active-shadow` | `inset 0 3px 5px rgba(0, 0, 0, 0.125)` | `.btn-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3130 | scss/mixins/_buttons.scss:31<br>via scss/_buttons.scss:132 | `(inset 0 3px 5px rgba($black, .125))` | Direct |
| `--bs-btn-active-shadow` | `inset 0 3px 5px rgba(0, 0, 0, 0.125)` | `.btn-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3164 | scss/mixins/_buttons.scss:31<br>via scss/_buttons.scss:132 | `(inset 0 3px 5px rgba($black, .125))` | Direct |
| `--bs-btn-active-shadow` | `inset 0 3px 5px rgba(0, 0, 0, 0.125)` | `.btn-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3096 | scss/mixins/_buttons.scss:31<br>via scss/_buttons.scss:132 | `(inset 0 3px 5px rgba($black, .125))` | Direct |
| `--bs-btn-active-shadow` | `inset 0 3px 5px rgba(0, 0, 0, 0.125)` | `.btn-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3147 | scss/mixins/_buttons.scss:31<br>via scss/_buttons.scss:132 | `(inset 0 3px 5px rgba($black, .125))` | Direct |
| `--bs-btn-active-shadow` | `inset 0 3px 5px rgba(0, 0, 0, 0.125)` | `.btn-outline-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3265 | scss/mixins/_buttons.scss:55<br>via scss/_buttons.scss:158 | `(inset 0 3px 5px rgba($black, .125))` | Direct |
| `--bs-btn-active-shadow` | `inset 0 3px 5px rgba(0, 0, 0, 0.125)` | `.btn-outline-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3299 | scss/mixins/_buttons.scss:55<br>via scss/_buttons.scss:158 | `(inset 0 3px 5px rgba($black, .125))` | Direct |
| `--bs-btn-active-shadow` | `inset 0 3px 5px rgba(0, 0, 0, 0.125)` | `.btn-outline-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3231 | scss/mixins/_buttons.scss:55<br>via scss/_buttons.scss:158 | `(inset 0 3px 5px rgba($black, .125))` | Direct |
| `--bs-btn-active-shadow` | `inset 0 3px 5px rgba(0, 0, 0, 0.125)` | `.btn-outline-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3282 | scss/mixins/_buttons.scss:55<br>via scss/_buttons.scss:158 | `(inset 0 3px 5px rgba($black, .125))` | Direct |
| `--bs-btn-active-shadow` | `inset 0 3px 5px rgba(0, 0, 0, 0.125)` | `.btn-outline-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3180 | scss/mixins/_buttons.scss:55<br>via scss/_buttons.scss:158 | `(inset 0 3px 5px rgba($black, .125))` | Direct |
| `--bs-btn-active-shadow` | `inset 0 3px 5px rgba(0, 0, 0, 0.125)` | `.btn-outline-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3197 | scss/mixins/_buttons.scss:55<br>via scss/_buttons.scss:158 | `(inset 0 3px 5px rgba($black, .125))` | Direct |
| `--bs-btn-active-shadow` | `inset 0 3px 5px rgba(0, 0, 0, 0.125)` | `.btn-outline-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3214 | scss/mixins/_buttons.scss:55<br>via scss/_buttons.scss:158 | `(inset 0 3px 5px rgba($black, .125))` | Direct |
| `--bs-btn-active-shadow` | `inset 0 3px 5px rgba(0, 0, 0, 0.125)` | `.btn-outline-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3248 | scss/mixins/_buttons.scss:55<br>via scss/_buttons.scss:158 | `(inset 0 3px 5px rgba($black, .125))` | Direct |
| `--bs-btn-active-shadow` | `inset 0 3px 5px rgba(0, 0, 0, 0.125)` | `.btn-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3045 | scss/mixins/_buttons.scss:31<br>via scss/_buttons.scss:132 | `(inset 0 3px 5px rgba($black, .125))` | Direct |
| `--bs-btn-active-shadow` | `inset 0 3px 5px rgba(0, 0, 0, 0.125)` | `.btn-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3062 | scss/mixins/_buttons.scss:31<br>via scss/_buttons.scss:132 | `(inset 0 3px 5px rgba($black, .125))` | Direct |
| `--bs-btn-active-shadow` | `inset 0 3px 5px rgba(0, 0, 0, 0.125)` | `.btn-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3079 | scss/mixins/_buttons.scss:31<br>via scss/_buttons.scss:132 | `(inset 0 3px 5px rgba($black, .125))` | Direct |
| `--bs-btn-active-shadow` | `inset 0 3px 5px rgba(0, 0, 0, 0.125)` | `.btn-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3113 | scss/mixins/_buttons.scss:31<br>via scss/_buttons.scss:132 | `(inset 0 3px 5px rgba($black, .125))` | Direct |

<a id="c-black-2"></a>
#### $black – pattern 2: `$ANCHOR`

7 contexts, 5 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-backdrop-bg` | `#000` | `.modal-backdrop`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5544 | scss/_modal.scss:116<br>via scss/_modal.scss:113 | `$black` | Direct |
| `--bs-black` | `#000` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:18<br>src/bootstrap/reference/css/bootstrap.css:19 | scss/_root.scss:10 | `$black` | Direct |
| `--bs-btn-close-color` | `#000` | `.btn-close`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5337 | scss/_close.scss:8<br>via scss/_close.scss:6 | `$black` | Direct |
| `--bs-carousel-caption-color` | `#000` | `.carousel-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:6202 | scss/_carousel.scss:207<br>via scss/_carousel.scss:211 | `$black` | Direct |
| `--bs-carousel-caption-color` | `#000` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap.css:6215 | scss/_carousel.scss:207<br>via scss/mixins/_color-mode.scss:16 | `$black` | Direct |
| `--bs-carousel-indicator-active-bg` | `#000` | `.carousel-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:6201 | scss/_carousel.scss:206<br>via scss/_carousel.scss:211 | `$black` | Direct |
| `--bs-carousel-indicator-active-bg` | `#000` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap.css:6214 | scss/_carousel.scss:206<br>via scss/mixins/_color-mode.scss:16 | `$black` | Direct |

<a id="c-black-3"></a>
#### $black – pattern 3: `escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='-4 -4 8 8'><circle r='3' fill='rgba($ANCHOR, .25)'/></svg>"))`

1 contexts, 1 properties. Relationship: Compile-time.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-form-switch-bg` | `url("data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='-4 -4 8 8'%3e%3ccircle r='3' fill='rgba%280, 0, 0, 0.25%29'/%3e%3c/svg%3e")` | `.form-switch .form-check-input`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2456 | scss/forms/_form-check.scss:128<br>via scss/forms/_form-check.scss:127 | `escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='-4 -4 8 8'><circle r='3' fill='rgba($black, .25)'/></svg>"))` | Compile-time |

<a id="c-black-4"></a>
#### $black – pattern 4: `escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 16 16' fill='$ANCHOR'><path d='M.293.293a1 1 0 0 1 1.414 0L8 6.586 14.29…`

1 contexts, 1 properties. Relationship: Compile-time.

Full expression:

```scss
escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 16 16' fill='$ANCHOR'><path d='M.293.293a1 1 0 0 1 1.414 0L8 6.586 14.293.293a1 1 0 1 1 1.414 1.414L9.414 8l6.293 6.293a1 1 0 0 1-1.414 1.414L8 9.414l-6.293 6.293a1 1 0 0 1-1.414-1.414L6.586 8 .293 1.707a1 1 0 0 1 0-1.414'/></svg>"))
```

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-close-bg` | `url("data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 16 16' fill='%23000'%3e%3cpath d='M.293.293a1 1 0 0 1 1.414 0L8 6.586 14.293.293a1 1 0 1 1 1.414 1.414L9.414 8l6.293 6.293a1 1 0 0 1-1.414 1.414L8 9.414l-6.293 6.293a1 1 0 0 1-1.414-1.414L6.586 8 .293 1.707a1 1 0 0 1 0-1.414'/%3e%3c/svg%3e")` | `.btn-close`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5338 | scss/_close.scss:9<br>via scss/_close.scss:6 | `escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 16 16' fill='$black'><path d='M.293.293a1 1 0 0 1 1.414 0L8 6.586 14.293.293a1 1 0 1 1 1.414 1.414L9.414 8l6.293 6.293a1 1 0 0 1-1.414 1.414L8 9.414l-6.293 6.293a1 1 0 0 1-1.414-1.414L6.586 8 .293 1.707a1 1 0 0 1 0-1.414'/></svg>"))` | Compile-time |

<a id="c-black-5"></a>
#### $black – pattern 5: `to-rgb($ANCHOR)`

1 contexts, 1 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-black-rgb` | `0, 0, 0` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:72<br>src/bootstrap/reference/css/bootstrap.css:73 | scss/_root.scss:38<br>via scss/_root.scss:1 | `to-rgb($black)` | Derived, Compile-time |

<a id="c-blue"></a>
### `$blue` (palette)

Definition: `$blue: #0d6efd` (`scss/_variables.scss:39`). 1 contexts, 1 properties.

<a id="c-blue-1"></a>
#### $blue – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-blue` | `#0d6efd` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:8<br>src/bootstrap/reference/css/bootstrap.css:9 | scss/_root.scss:10 | `$blue` | Direct |

<a id="c-cyan"></a>
### `$cyan` (palette)

Definition: `$cyan: #0dcaf0` (`scss/_variables.scss:48`). 1 contexts, 1 properties.

<a id="c-cyan-1"></a>
#### $cyan – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-cyan` | `#0dcaf0` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:17<br>src/bootstrap/reference/css/bootstrap.css:18 | scss/_root.scss:10 | `$cyan` | Direct |

<a id="c-gray-100"></a>
### `$gray-100` (palette)

Definition: `$gray-100: #f8f9fa` (`scss/_variables.scss:10`). 6 contexts, 6 properties.

<a id="c-gray-100-1"></a>
#### $gray-100 – pattern 1: `var(--bs-light-bg-subtle)`

3 contexts, 3 properties. Relationship: Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-alert-bg` | `var(--bs-light-bg-subtle)` | `.alert-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4919 | scss/_alert.scss:63<br>via scss/_alert.scss:61 | `var(--bs-light-bg-subtle)` | Runtime, Mixed |
| `--bs-list-group-active-color` | `var(--bs-light-bg-subtle)` | `.list-group-item-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5318 | scss/_list-group.scss:194<br>via scss/_list-group.scss:186 | `var(--bs-light-bg-subtle)` | Runtime, Mixed |
| `--bs-list-group-bg` | `var(--bs-light-bg-subtle)` | `.list-group-item-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5312 | scss/_list-group.scss:188<br>via scss/_list-group.scss:186 | `var(--bs-light-bg-subtle)` | Runtime, Mixed |

<a id="c-gray-100-2"></a>
#### $gray-100 – pattern 2: `$ANCHOR`

2 contexts, 2 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-gray-100` | `#f8f9fa` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:22<br>src/bootstrap/reference/css/bootstrap.css:23 | scss/_root.scss:14 | `$gray-100` | Direct |
| `--bs-light-text-emphasis` | `#f8f9fa` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:149<br>src/bootstrap/reference/css/bootstrap.css:150 | scss/_root.scss:156<br>via scss/mixins/_color-mode.scss:16 | `$gray-100` | Direct |

<a id="c-gray-100-3"></a>
#### $gray-100 – pattern 3: `mix($ANCHOR, $white)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-light-bg-subtle` | `#fcfcfd` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:61<br>src/bootstrap/reference/css/bootstrap.css:62 | scss/_root.scss:30 | `mix($gray-100, $white)` | Direct |

<a id="c-gray-200"></a>
### `$gray-200` (palette)

Definition: `$gray-200: #e9ecef` (`scss/_variables.scss:11`). 6 contexts, 6 properties.

<a id="c-gray-200-1"></a>
#### $gray-200 – pattern 1: `var(--bs-light-border-subtle)`

4 contexts, 4 properties. Relationship: Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-alert-border-color` | `var(--bs-light-border-subtle)` | `.alert-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4920 | scss/_alert.scss:64<br>via scss/_alert.scss:61 | `var(--bs-light-border-subtle)` | Runtime, Mixed |
| `--bs-list-group-action-active-bg` | `var(--bs-light-border-subtle)` | `.list-group-item-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5317 | scss/_list-group.scss:193<br>via scss/_list-group.scss:186 | `var(--bs-light-border-subtle)` | Runtime, Mixed |
| `--bs-list-group-action-hover-bg` | `var(--bs-light-border-subtle)` | `.list-group-item-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5315 | scss/_list-group.scss:191<br>via scss/_list-group.scss:186 | `var(--bs-light-border-subtle)` | Runtime, Mixed |
| `--bs-list-group-border-color` | `var(--bs-light-border-subtle)` | `.list-group-item-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5313 | scss/_list-group.scss:189<br>via scss/_list-group.scss:186 | `var(--bs-light-border-subtle)` | Runtime, Mixed |

<a id="c-gray-200-2"></a>
#### $gray-200 – pattern 2: `$ANCHOR`

2 contexts, 2 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-gray-200` | `#e9ecef` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:23<br>src/bootstrap/reference/css/bootstrap.css:24 | scss/_root.scss:14 | `$gray-200` | Direct |
| `--bs-light-border-subtle` | `#e9ecef` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:69<br>src/bootstrap/reference/css/bootstrap.css:70 | scss/_root.scss:34 | `$gray-200` | Direct |

<a id="c-gray-300"></a>
### `$gray-300` (palette)

Definition: `$gray-300: #dee2e6` (`scss/_variables.scss:12`). 4 contexts, 4 properties.

<a id="c-gray-300-1"></a>
#### $gray-300 – pattern 1: `$ANCHOR`

4 contexts, 4 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-dark-text-emphasis` | `#dee2e6` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:150<br>src/bootstrap/reference/css/bootstrap.css:151 | scss/_root.scss:156<br>via scss/mixins/_color-mode.scss:16 | `$gray-300` | Direct |
| `--bs-dropdown-color` | `#dee2e6` | `.dropdown-menu-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3682 | scss/_dropdown.scss:237<br>via scss/_dropdown.scss:235 | `$gray-300` | Direct |
| `--bs-dropdown-link-color` | `#dee2e6` | `.dropdown-menu-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3686 | scss/_dropdown.scss:241<br>via scss/_dropdown.scss:235 | `$gray-300` | Direct |
| `--bs-gray-300` | `#dee2e6` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:24<br>src/bootstrap/reference/css/bootstrap.css:25 | scss/_root.scss:14 | `$gray-300` | Direct |

<a id="c-gray-400"></a>
### `$gray-400` (palette)

Definition: `$gray-400: #ced4da` (`scss/_variables.scss:13`). 5 contexts, 5 properties.

<a id="c-gray-400-1"></a>
#### $gray-400 – pattern 1: `var(--bs-dark-bg-subtle)`

3 contexts, 3 properties. Relationship: Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-alert-bg` | `var(--bs-dark-bg-subtle)` | `.alert-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4926 | scss/_alert.scss:63<br>via scss/_alert.scss:61 | `var(--bs-dark-bg-subtle)` | Runtime, Mixed |
| `--bs-list-group-active-color` | `var(--bs-dark-bg-subtle)` | `.list-group-item-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5331 | scss/_list-group.scss:194<br>via scss/_list-group.scss:186 | `var(--bs-dark-bg-subtle)` | Runtime, Mixed |
| `--bs-list-group-bg` | `var(--bs-dark-bg-subtle)` | `.list-group-item-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5325 | scss/_list-group.scss:188<br>via scss/_list-group.scss:186 | `var(--bs-dark-bg-subtle)` | Runtime, Mixed |

<a id="c-gray-400-2"></a>
#### $gray-400 – pattern 2: `$ANCHOR`

2 contexts, 2 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-dark-bg-subtle` | `#ced4da` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:62<br>src/bootstrap/reference/css/bootstrap.css:63 | scss/_root.scss:30 | `$gray-400` | Direct |
| `--bs-gray-400` | `#ced4da` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:25<br>src/bootstrap/reference/css/bootstrap.css:26 | scss/_root.scss:14 | `$gray-400` | Direct |

<a id="c-gray-500"></a>
### `$gray-500` (palette)

Definition: `$gray-500: #adb5bd` (`scss/_variables.scss:14`). 8 contexts, 8 properties.

<a id="c-gray-500-1"></a>
#### $gray-500 – pattern 1: `$ANCHOR`

4 contexts, 4 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-dark-border-subtle` | `#adb5bd` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:70<br>src/bootstrap/reference/css/bootstrap.css:71 | scss/_root.scss:34 | `$gray-500` | Direct |
| `--bs-dropdown-header-color` | `#adb5bd` | `.dropdown-menu-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3693 | scss/_dropdown.scss:248<br>via scss/_dropdown.scss:235 | `$gray-500` | Direct |
| `--bs-dropdown-link-disabled-color` | `#adb5bd` | `.dropdown-menu-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3692 | scss/_dropdown.scss:247<br>via scss/_dropdown.scss:235 | `$gray-500` | Direct |
| `--bs-gray-500` | `#adb5bd` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:26<br>src/bootstrap/reference/css/bootstrap.css:27 | scss/_root.scss:14 | `$gray-500` | Direct |

<a id="c-gray-500-2"></a>
#### $gray-500 – pattern 2: `var(--bs-dark-border-subtle)`

4 contexts, 4 properties. Relationship: Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-alert-border-color` | `var(--bs-dark-border-subtle)` | `.alert-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4927 | scss/_alert.scss:64<br>via scss/_alert.scss:61 | `var(--bs-dark-border-subtle)` | Runtime, Mixed |
| `--bs-list-group-action-active-bg` | `var(--bs-dark-border-subtle)` | `.list-group-item-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5330 | scss/_list-group.scss:193<br>via scss/_list-group.scss:186 | `var(--bs-dark-border-subtle)` | Runtime, Mixed |
| `--bs-list-group-action-hover-bg` | `var(--bs-dark-border-subtle)` | `.list-group-item-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5328 | scss/_list-group.scss:191<br>via scss/_list-group.scss:186 | `var(--bs-dark-border-subtle)` | Runtime, Mixed |
| `--bs-list-group-border-color` | `var(--bs-dark-border-subtle)` | `.list-group-item-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5326 | scss/_list-group.scss:189<br>via scss/_list-group.scss:186 | `var(--bs-dark-border-subtle)` | Runtime, Mixed |

<a id="c-gray-600"></a>
### `$gray-600` (palette)

Definition: `$gray-600: #6c757d` (`scss/_variables.scss:15`). 4 contexts, 4 properties.

<a id="c-gray-600-1"></a>
#### $gray-600 – pattern 1: `$ANCHOR`

4 contexts, 4 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-disabled-color` | `#6c757d` | `.btn-link`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3315 | scss/_buttons.scss:179<br>via scss/_buttons.scss:170 | `$gray-600` | Direct |
| `--bs-dropdown-header-color` | `#6c757d` | `.dropdown-menu`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3429 | scss/_dropdown.scss:44<br>via scss/_dropdown.scss:19 | `$gray-600` | Direct |
| `--bs-gray` | `#6c757d` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:20<br>src/bootstrap/reference/css/bootstrap.css:21 | scss/_root.scss:10 | `$gray-600` | Direct |
| `--bs-gray-600` | `#6c757d` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:27<br>src/bootstrap/reference/css/bootstrap.css:28 | scss/_root.scss:14 | `$gray-600` | Direct |

<a id="c-gray-700"></a>
### `$gray-700` (palette)

Definition: `$gray-700: #495057` (`scss/_variables.scss:16`). 14 contexts, 9 properties.

<a id="c-gray-700-1"></a>
#### $gray-700 – pattern 1: `var(--bs-dark-text-emphasis)`

5 contexts, 5 properties. Relationship: Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-alert-color` | `var(--bs-dark-text-emphasis)` | `.alert-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4925 | scss/_alert.scss:62<br>via scss/_alert.scss:61 | `var(--bs-dark-text-emphasis)` | Runtime, Mixed |
| `--bs-alert-link-color` | `var(--bs-dark-text-emphasis)` | `.alert-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4928 | scss/_alert.scss:65<br>via scss/_alert.scss:61 | `var(--bs-dark-text-emphasis)` | Runtime, Mixed |
| `--bs-list-group-active-bg` | `var(--bs-dark-text-emphasis)` | `.list-group-item-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5332 | scss/_list-group.scss:195<br>via scss/_list-group.scss:186 | `var(--bs-dark-text-emphasis)` | Runtime, Mixed |
| `--bs-list-group-active-border-color` | `var(--bs-dark-text-emphasis)` | `.list-group-item-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5333 | scss/_list-group.scss:196<br>via scss/_list-group.scss:186 | `var(--bs-dark-text-emphasis)` | Runtime, Mixed |
| `--bs-list-group-color` | `var(--bs-dark-text-emphasis)` | `.list-group-item-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5324 | scss/_list-group.scss:187<br>via scss/_list-group.scss:186 | `var(--bs-dark-text-emphasis)` | Runtime, Mixed |

<a id="c-gray-700-2"></a>
#### $gray-700 – pattern 2: `var(--bs-light-text-emphasis)`

5 contexts, 5 properties. Relationship: Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-alert-color` | `var(--bs-light-text-emphasis)` | `.alert-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4918 | scss/_alert.scss:62<br>via scss/_alert.scss:61 | `var(--bs-light-text-emphasis)` | Runtime, Mixed |
| `--bs-alert-link-color` | `var(--bs-light-text-emphasis)` | `.alert-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4921 | scss/_alert.scss:65<br>via scss/_alert.scss:61 | `var(--bs-light-text-emphasis)` | Runtime, Mixed |
| `--bs-list-group-active-bg` | `var(--bs-light-text-emphasis)` | `.list-group-item-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5319 | scss/_list-group.scss:195<br>via scss/_list-group.scss:186 | `var(--bs-light-text-emphasis)` | Runtime, Mixed |
| `--bs-list-group-active-border-color` | `var(--bs-light-text-emphasis)` | `.list-group-item-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5320 | scss/_list-group.scss:196<br>via scss/_list-group.scss:186 | `var(--bs-light-text-emphasis)` | Runtime, Mixed |
| `--bs-list-group-color` | `var(--bs-light-text-emphasis)` | `.list-group-item-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5311 | scss/_list-group.scss:187<br>via scss/_list-group.scss:186 | `var(--bs-light-text-emphasis)` | Runtime, Mixed |

<a id="c-gray-700-3"></a>
#### $gray-700 – pattern 3: `$ANCHOR`

4 contexts, 4 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-dark-text-emphasis` | `#495057` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:54<br>src/bootstrap/reference/css/bootstrap.css:55 | scss/_root.scss:26 | `$gray-700` | Direct |
| `--bs-gray-700` | `#495057` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:28<br>src/bootstrap/reference/css/bootstrap.css:29 | scss/_root.scss:14 | `$gray-700` | Direct |
| `--bs-light-border-subtle` | `#495057` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:165<br>src/bootstrap/reference/css/bootstrap.css:166 | scss/_root.scss:164<br>via scss/mixins/_color-mode.scss:16 | `$gray-700` | Direct |
| `--bs-light-text-emphasis` | `#495057` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:53<br>src/bootstrap/reference/css/bootstrap.css:54 | scss/_root.scss:26 | `$gray-700` | Direct |

<a id="c-gray-800"></a>
### `$gray-800` (palette)

Definition: `$gray-800: #343a40` (`scss/_variables.scss:17`). 7 contexts, 7 properties.

<a id="c-gray-800-1"></a>
#### $gray-800 – pattern 1: `$ANCHOR`

5 contexts, 5 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-dark-border-subtle` | `#343a40` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:166<br>src/bootstrap/reference/css/bootstrap.css:167 | scss/_root.scss:164<br>via scss/mixins/_color-mode.scss:16 | `$gray-800` | Direct |
| `--bs-dropdown-bg` | `#343a40` | `.dropdown-menu-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3683 | scss/_dropdown.scss:238<br>via scss/_dropdown.scss:235 | `$gray-800` | Direct |
| `--bs-gray-800` | `#343a40` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:29<br>src/bootstrap/reference/css/bootstrap.css:30 | scss/_root.scss:14 | `$gray-800` | Direct |
| `--bs-gray-dark` | `#343a40` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:21<br>src/bootstrap/reference/css/bootstrap.css:22 | scss/_root.scss:10 | `$gray-800` | Direct |
| `--bs-light-bg-subtle` | `#343a40` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:157<br>src/bootstrap/reference/css/bootstrap.css:158 | scss/_root.scss:160<br>via scss/mixins/_color-mode.scss:16 | `$gray-800` | Direct |

<a id="c-gray-800-2"></a>
#### $gray-800 – pattern 2: `escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 16 16'><path fill='none' stroke='$ANCHOR' stroke-linecap='round' stroke-…`

1 contexts, 1 properties. Relationship: Compile-time.

Full expression:

```scss
escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 16 16'><path fill='none' stroke='$ANCHOR' stroke-linecap='round' stroke-linejoin='round' stroke-width='2' d='m2 5 6 6 6-6'/></svg>"))
```

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-form-select-bg-img` | `url("data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 16 16'%3e%3cpath fill='none' stroke='%23343a40' stroke-linecap='round' stroke-linejoin='round' stroke-width='2' d='m2 5 6 6 6-6'/%3e%3c/svg%3e")` | `.form-select`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2309 | scss/forms/_form-select.scss:7<br>via scss/forms/_form-select.scss:6 | `escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 16 16'><path fill='none' stroke='$gray-800' stroke-linecap='round' stroke-linejoin='round' stroke-width='2' d='m2 5 6 6 6-6'/></svg>"))` | Compile-time |

<a id="c-gray-800-3"></a>
#### $gray-800 – pattern 3: `mix($ANCHOR, $black)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-dark-bg-subtle` | `#1a1d20` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:158<br>src/bootstrap/reference/css/bootstrap.css:159 | scss/_root.scss:160<br>via scss/mixins/_color-mode.scss:16 | `mix($gray-800, $black)` | Direct |

<a id="c-gray-900"></a>
### `$gray-900` (palette)

Definition: `$gray-900: #212529` (`scss/_variables.scss:18`). 1 contexts, 1 properties.

<a id="c-gray-900-1"></a>
#### $gray-900 – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-gray-900` | `#212529` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:30<br>src/bootstrap/reference/css/bootstrap.css:31 | scss/_root.scss:14 | `$gray-900` | Direct |

<a id="c-green"></a>
### `$green` (palette)

Definition: `$green: #198754` (`scss/_variables.scss:46`). 3 contexts, 3 properties.

<a id="c-green-1"></a>
#### $green – pattern 1: `tint-color($ANCHOR, 40%)`

2 contexts, 2 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-form-valid-border-color` | `#75b798` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:178<br>src/bootstrap/reference/css/bootstrap.css:179 | scss/_root.scss:182<br>via scss/mixins/_color-mode.scss:16 | `tint-color($green, 40%)` | Direct |
| `--bs-form-valid-color` | `#75b798` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:177<br>src/bootstrap/reference/css/bootstrap.css:178 | scss/_root.scss:181<br>via scss/mixins/_color-mode.scss:16 | `tint-color($green, 40%)` | Direct |

<a id="c-green-2"></a>
#### $green – pattern 2: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-green` | `#198754` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:15<br>src/bootstrap/reference/css/bootstrap.css:16 | scss/_root.scss:10 | `$green` | Direct |

<a id="c-indigo"></a>
### `$indigo` (palette)

Definition: `$indigo: #6610f2` (`scss/_variables.scss:40`). 1 contexts, 1 properties.

<a id="c-indigo-1"></a>
#### $indigo – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-indigo` | `#6610f2` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:9<br>src/bootstrap/reference/css/bootstrap.css:10 | scss/_root.scss:10 | `$indigo` | Direct |

<a id="c-orange"></a>
### `$orange` (palette)

Definition: `$orange: #fd7e14` (`scss/_variables.scss:44`). 1 contexts, 1 properties.

<a id="c-orange-1"></a>
#### $orange – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-orange` | `#fd7e14` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:13<br>src/bootstrap/reference/css/bootstrap.css:14 | scss/_root.scss:10 | `$orange` | Direct |

<a id="c-pink"></a>
### `$pink` (palette)

Definition: `$pink: #d63384` (`scss/_variables.scss:42`). 3 contexts, 2 properties.

<a id="c-pink-1"></a>
#### $pink – pattern 1: `$ANCHOR`

2 contexts, 2 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-code-color` | `#d63384` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:100<br>src/bootstrap/reference/css/bootstrap.css:101 | scss/_root.scss:93<br>via scss/_root.scss:1 | `$pink` | Direct |
| `--bs-pink` | `#d63384` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:11<br>src/bootstrap/reference/css/bootstrap.css:12 | scss/_root.scss:10 | `$pink` | Direct |

<a id="c-pink-2"></a>
#### $pink – pattern 2: `tint-color($ANCHOR, 40%)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-code-color` | `#e685b5` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:172<br>src/bootstrap/reference/css/bootstrap.css:173 | scss/_root.scss:174<br>via scss/mixins/_color-mode.scss:16 | `tint-color($pink, 40%)` | Direct |

<a id="c-purple"></a>
### `$purple` (palette)

Definition: `$purple: #6f42c1` (`scss/_variables.scss:41`). 1 contexts, 1 properties.

<a id="c-purple-1"></a>
#### $purple – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-purple` | `#6f42c1` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:10<br>src/bootstrap/reference/css/bootstrap.css:11 | scss/_root.scss:10 | `$purple` | Direct |

<a id="c-red"></a>
### `$red` (palette)

Definition: `$red: #dc3545` (`scss/_variables.scss:43`). 3 contexts, 3 properties.

<a id="c-red-1"></a>
#### $red – pattern 1: `tint-color($ANCHOR, 40%)`

2 contexts, 2 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-form-invalid-border-color` | `#ea868f` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:180<br>src/bootstrap/reference/css/bootstrap.css:181 | scss/_root.scss:184<br>via scss/mixins/_color-mode.scss:16 | `tint-color($red, 40%)` | Direct |
| `--bs-form-invalid-color` | `#ea868f` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:179<br>src/bootstrap/reference/css/bootstrap.css:180 | scss/_root.scss:183<br>via scss/mixins/_color-mode.scss:16 | `tint-color($red, 40%)` | Direct |

<a id="c-red-2"></a>
#### $red – pattern 2: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-red` | `#dc3545` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:12<br>src/bootstrap/reference/css/bootstrap.css:13 | scss/_root.scss:10 | `$red` | Direct |

<a id="c-teal"></a>
### `$teal` (palette)

Definition: `$teal: #20c997` (`scss/_variables.scss:47`). 1 contexts, 1 properties.

<a id="c-teal-1"></a>
#### $teal – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-teal` | `#20c997` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:16<br>src/bootstrap/reference/css/bootstrap.css:17 | scss/_root.scss:10 | `$teal` | Direct |

<a id="c-white"></a>
### `$white` (palette)

Definition: `$white: #fff` (`scss/_variables.scss:9`). 29 contexts, 24 properties.

<a id="c-white-1"></a>
#### $white – pattern 1: `$ANCHOR`

14 contexts, 13 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-badge-color` | `#fff` | `.badge`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4814 | scss/_badge.scss:12<br>via scss/_badge.scss:6 | `$white` | Direct |
| `--bs-carousel-caption-color` | `#fff` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap.css:6209 | scss/_carousel.scss:218<br>via scss/_carousel.scss:215 | `$white` | Direct |
| `--bs-carousel-indicator-active-bg` | `#fff` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap.css:6208 | scss/_carousel.scss:217<br>via scss/_carousel.scss:215 | `$white` | Direct |
| `--bs-dropdown-link-active-color` | `#fff` | `.dropdown-menu`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3424 | scss/_dropdown.scss:39<br>via scss/_dropdown.scss:19 | `$white` | Direct |
| `--bs-dropdown-link-active-color` | `#fff` | `.dropdown-menu-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3690 | scss/_dropdown.scss:245<br>via scss/_dropdown.scss:235 | `$white` | Direct |
| `--bs-dropdown-link-hover-color` | `#fff` | `.dropdown-menu-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3687 | scss/_dropdown.scss:242<br>via scss/_dropdown.scss:235 | `$white` | Direct |
| `--bs-list-group-active-color` | `#fff` | `.list-group`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5008 | scss/_list-group.scss:21<br>via scss/_list-group.scss:5 | `$white` | Direct |
| `--bs-nav-pills-link-active-color` | `#fff` | `.nav-pills`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3874 | scss/_nav.scss:108<br>via scss/_nav.scss:105 | `$white` | Direct |
| `--bs-navbar-active-color` | `#fff` | `.navbar-dark, .navbar[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4348 | scss/_navbar.scss:275<br>via scss/_navbar.scss:269 | `$white` | Direct |
| `--bs-navbar-brand-color` | `#fff` | `.navbar-dark, .navbar[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4349 | scss/_navbar.scss:276<br>via scss/_navbar.scss:269 | `$white` | Direct |
| `--bs-navbar-brand-hover-color` | `#fff` | `.navbar-dark, .navbar[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4350 | scss/_navbar.scss:277<br>via scss/_navbar.scss:269 | `$white` | Direct |
| `--bs-pagination-active-color` | `#fff` | `.pagination`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4730 | scss/_pagination.scss:17<br>via scss/_pagination.scss:1 | `$white` | Direct |
| `--bs-progress-bar-color` | `#fff` | `.progress, .progress-stacked`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4943 | scss/_progress.scss:19<br>via scss/_progress.scss:11 | `$white` | Direct |
| `--bs-white` | `#fff` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:19<br>src/bootstrap/reference/css/bootstrap.css:20 | scss/_root.scss:10 | `$white` | Direct |

<a id="c-white-2"></a>
#### $white – pattern 2: `escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 30 30'><path stroke='rgba($ANCHOR, .55)' stroke-linecap='round' stroke-m…`

2 contexts, 1 properties. Relationship: Compile-time.

Full expression:

```scss
escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 30 30'><path stroke='rgba($ANCHOR, .55)' stroke-linecap='round' stroke-miterlimit='10' stroke-width='2' d='M4 7h22M4 15h22M4 23h22'/></svg>"))
```

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-navbar-toggler-icon-bg` | `url("data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 30 30'%3e%3cpath stroke='rgba%28255, 255, 255, 0.55%29' stroke-linecap='round' stroke-miterlimit='10' stroke-width='2' d='M4 7h22M4 15h22M4 23h22'/%3e%3c/svg%3e")` | `.navbar-dark, .navbar[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4352 | scss/_navbar.scss:279<br>via scss/_navbar.scss:269 | `escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 30 30'><path stroke='rgba($white, .55)' stroke-linecap='round' stroke-miterlimit='10' stroke-width='2' d='M4 7h22M4 15h22M4 23h22'/></svg>"))` | Compile-time |
| `--bs-navbar-toggler-icon-bg` | `url("data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 30 30'%3e%3cpath stroke='rgba%28255, 255, 255, 0.55%29' stroke-linecap='round' stroke-miterlimit='10' stroke-width='2' d='M4 7h22M4 15h22M4 23h22'/%3e%3c/svg%3e")` | `[data-bs-theme=dark] .navbar-toggler-icon`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4356 | scss/_navbar.scss:286<br>via scss/_navbar.scss:285 | `escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 30 30'><path stroke='rgba($white, .55)' stroke-linecap='round' stroke-miterlimit='10' stroke-width='2' d='M4 7h22M4 15h22M4 23h22'/></svg>"))` | Compile-time |

<a id="c-white-3"></a>
#### $white – pattern 3: `(inset 0 1px 0 rgba($ANCHOR, .15), 0 1px 1px rgba($black, .075))`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-box-shadow` | `inset 0 1px 0 rgba(255, 255, 255, 0.15), 0 1px 1px rgba(0, 0, 0, 0.075)` | `.btn`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2966 | scss/_buttons.scss:19<br>via scss/_buttons.scss:5 | `(inset 0 1px 0 rgba($white, .15), 0 1px 1px rgba($black, .075))` | Direct |

<a id="c-white-4"></a>
#### $white – pattern 4: `escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='-4 -4 8 8'><circle r='2' fill='$ANCHOR'/></svg>"))`

1 contexts, 1 properties. Relationship: Compile-time.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-form-check-bg-image` | `url("data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='-4 -4 8 8'%3e%3ccircle r='2' fill='%23fff'/%3e%3c/svg%3e")` | `.form-check-input:checked[type=radio]`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2435 | scss/forms/_form-check.scss:82<br>via scss/forms/_form-check.scss:78 | `escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='-4 -4 8 8'><circle r='2' fill='$white'/></svg>"))` | Compile-time |

<a id="c-white-5"></a>
#### $white – pattern 5: `escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='-4 -4 8 8'><circle r='3' fill='$ANCHOR'/></svg>"))`

1 contexts, 1 properties. Relationship: Compile-time.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-form-switch-bg` | `url("data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='-4 -4 8 8'%3e%3ccircle r='3' fill='%23fff'/%3e%3c/svg%3e")` | `.form-switch .form-check-input:checked`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2474 | scss/forms/_form-check.scss:147<br>via scss/forms/_form-check.scss:141 | `escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='-4 -4 8 8'><circle r='3' fill='$white'/></svg>"))` | Compile-time |

<a id="c-white-6"></a>
#### $white – pattern 6: `escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='-4 -4 8 8'><circle r='3' fill='rgba($ANCHOR, .25)'/></svg>"))`

1 contexts, 1 properties. Relationship: Compile-time.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-form-switch-bg` | `url("data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='-4 -4 8 8'%3e%3ccircle r='3' fill='rgba%28255, 255, 255, 0.25%29'/%3e%3c/svg%3e")` | `[data-bs-theme=dark] .form-switch .form-check-input:not(:checked):not(:focus)`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2502 | scss/forms/_form-check.scss:186<br>via scss/forms/_form-check.scss:185 | `escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='-4 -4 8 8'><circle r='3' fill='rgba($white, .25)'/></svg>"))` | Compile-time |

<a id="c-white-7"></a>
#### $white – pattern 7: `escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 20 20'><path fill='none' stroke='$ANCHOR' stroke-linecap='round' stroke-…`

1 contexts, 1 properties. Relationship: Compile-time.

Full expression:

```scss
escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 20 20'><path fill='none' stroke='$ANCHOR' stroke-linecap='round' stroke-linejoin='round' stroke-width='3' d='M6 10h8'/></svg>"))
```

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-form-check-bg-image` | `url("data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 20 20'%3e%3cpath fill='none' stroke='%23fff' stroke-linecap='round' stroke-linejoin='round' stroke-width='3' d='M6 10h8'/%3e%3c/svg%3e")` | `.form-check-input[type=checkbox]:indeterminate`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2440 | scss/forms/_form-check.scss:94<br>via scss/forms/_form-check.scss:87 | `escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 20 20'><path fill='none' stroke='$white' stroke-linecap='round' stroke-linejoin='round' stroke-width='3' d='M6 10h8'/></svg>"))` | Compile-time |

<a id="c-white-8"></a>
#### $white – pattern 8: `escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 20 20'><path fill='none' stroke='$ANCHOR' stroke-linecap='round' stroke-…`

1 contexts, 1 properties. Relationship: Compile-time.

Full expression:

```scss
escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 20 20'><path fill='none' stroke='$ANCHOR' stroke-linecap='round' stroke-linejoin='round' stroke-width='3' d='m6 10 3 3 6-6'/></svg>"))
```

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-form-check-bg-image` | `url("data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 20 20'%3e%3cpath fill='none' stroke='%23fff' stroke-linecap='round' stroke-linejoin='round' stroke-width='3' d='m6 10 3 3 6-6'/%3e%3c/svg%3e")` | `.form-check-input:checked[type=checkbox]`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2432 | scss/forms/_form-check.scss:74<br>via scss/forms/_form-check.scss:70 | `escape-svg(url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 20 20'><path fill='none' stroke='$white' stroke-linecap='round' stroke-linejoin='round' stroke-width='3' d='m6 10 3 3 6-6'/></svg>"))` | Compile-time |

<a id="c-white-9"></a>
#### $white – pattern 9: `linear-gradient(180deg, rgba($ANCHOR, .15), rgba($ANCHOR, 0))`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-gradient` | `linear-gradient(180deg, rgba(255, 255, 255, 0.15), rgba(255, 255, 255, 0))` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:75<br>src/bootstrap/reference/css/bootstrap.css:76 | scss/_root.scss:46<br>via scss/_root.scss:1 | `linear-gradient(180deg, rgba($white, .15), rgba($white, 0))` | Direct |

<a id="c-white-10"></a>
#### $white – pattern 10: `rgba($ANCHOR, .1)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-navbar-toggler-border-color` | `rgba(255, 255, 255, 0.1)` | `.navbar-dark, .navbar[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4351 | scss/_navbar.scss:278<br>via scss/_navbar.scss:269 | `rgba($white, .1)` | Direct |

<a id="c-white-11"></a>
#### $white – pattern 11: `rgba($ANCHOR, .15)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-dropdown-link-hover-bg` | `rgba(255, 255, 255, 0.15)` | `.dropdown-menu-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3689 | scss/_dropdown.scss:244<br>via scss/_dropdown.scss:235 | `rgba($white, .15)` | Direct |

<a id="c-white-12"></a>
#### $white – pattern 12: `rgba($ANCHOR, .25)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-navbar-disabled-color` | `rgba(255, 255, 255, 0.25)` | `.navbar-dark, .navbar[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4347 | scss/_navbar.scss:274<br>via scss/_navbar.scss:269 | `rgba($white, .25)` | Direct |

<a id="c-white-13"></a>
#### $white – pattern 13: `rgba($ANCHOR, .55)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-navbar-color` | `rgba(255, 255, 255, 0.55)` | `.navbar-dark, .navbar[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4345 | scss/_navbar.scss:272<br>via scss/_navbar.scss:269 | `rgba($white, .55)` | Direct |

<a id="c-white-14"></a>
#### $white – pattern 14: `rgba($ANCHOR, .75)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-navbar-hover-color` | `rgba(255, 255, 255, 0.75)` | `.navbar-dark, .navbar[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4346 | scss/_navbar.scss:273<br>via scss/_navbar.scss:269 | `rgba($white, .75)` | Direct |

<a id="c-white-15"></a>
#### $white – pattern 15: `to-rgb($ANCHOR)`

1 contexts, 1 properties. Relationship: Compile-time, Derived.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-white-rgb` | `255, 255, 255` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:71<br>src/bootstrap/reference/css/bootstrap.css:72 | scss/_root.scss:37<br>via scss/_root.scss:1 | `to-rgb($white)` | Derived, Compile-time |

<a id="c-yellow"></a>
### `$yellow` (palette)

Definition: `$yellow: #ffc107` (`scss/_variables.scss:45`). 3 contexts, 2 properties.

<a id="c-yellow-1"></a>
#### $yellow – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-yellow` | `#ffc107` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:14<br>src/bootstrap/reference/css/bootstrap.css:15 | scss/_root.scss:10 | `$yellow` | Direct |

<a id="c-yellow-2"></a>
#### $yellow – pattern 2: `shade-color($ANCHOR, 60%)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-highlight-bg` | `#664d03` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:174<br>src/bootstrap/reference/css/bootstrap.css:175 | scss/_root.scss:176<br>via scss/mixins/_color-mode.scss:16 | `shade-color($yellow, 60%)` | Direct |

<a id="c-yellow-3"></a>
#### $yellow – pattern 3: `tint-color($ANCHOR, 80%)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-highlight-bg` | `#fff3cd` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:102<br>src/bootstrap/reference/css/bootstrap.css:103 | scss/_root.scss:95<br>via scss/_root.scss:1 | `tint-color($yellow, 80%)` | Direct |

<a id="c-spacer"></a>
### `$spacer` (spacing)

Definition: `$spacer: 1rem` (`scss/_variables.scss:410`). 86 contexts, 28 properties.

<a id="c-spacer-1"></a>
#### $spacer – pattern 1: `$ANCHOR`

29 contexts, 19 properties. Relationship: Compile-time, Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-alert-padding-x` | `1rem` | `.alert`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4838 | scss/_alert.scss:8<br>via scss/_alert.scss:5 | `$spacer` | Direct |
| `--bs-alert-padding-y` | `1rem` | `.alert`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4839 | scss/_alert.scss:9<br>via scss/_alert.scss:5 | `$spacer` | Direct |
| `--bs-card-cap-padding-x` | `1rem` | `.card`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4371 | scss/_card.scss:18<br>via scss/_card.scss:5 | `$spacer` | Direct |
| `--bs-card-img-overlay-padding` | `1rem` | `.card`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4377 | scss/_card.scss:24<br>via scss/_card.scss:5 | `$spacer` | Direct |
| `--bs-card-spacer-x` | `1rem` | `.card`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4361 | scss/_card.scss:8<br>via scss/_card.scss:5 | `$spacer` | Direct |
| `--bs-card-spacer-y` | `1rem` | `.card`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4360 | scss/_card.scss:7<br>via scss/_card.scss:5 | `$spacer` | Direct |
| `--bs-dropdown-header-padding-x` | `1rem` | `.dropdown-menu`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3430 | scss/_dropdown.scss:45<br>via scss/_dropdown.scss:19 | `$spacer` | Direct |
| `--bs-dropdown-item-padding-x` | `1rem` | `.dropdown-menu`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3427 | scss/_dropdown.scss:42<br>via scss/_dropdown.scss:19 | `$spacer` | Direct |
| `--bs-gutter-x` | `1rem` | `.g-3, .gx-3`<br>(none) | src/bootstrap/reference/css/bootstrap-grid.css:255<br>src/bootstrap/reference/css/bootstrap.css:989 | scss/mixins/_grid.scss:117<br>via scss/mixins/_grid.scss:115 | `$spacer` | Direct, Compile-time |
| `--bs-gutter-x` | `1rem` | `.g-lg-3, .gx-lg-3`<br>@media (min-width: 992px) | src/bootstrap/reference/css/bootstrap-grid.css:767<br>src/bootstrap/reference/css/bootstrap.css:1501 | scss/mixins/_grid.scss:117<br>via scss/mixins/_grid.scss:115 | `$spacer` | Direct, Compile-time |
| `--bs-gutter-x` | `1rem` | `.g-md-3, .gx-md-3`<br>@media (min-width: 768px) | src/bootstrap/reference/css/bootstrap-grid.css:598<br>src/bootstrap/reference/css/bootstrap.css:1332 | scss/mixins/_grid.scss:117<br>via scss/mixins/_grid.scss:115 | `$spacer` | Direct, Compile-time |
| `--bs-gutter-x` | `1rem` | `.g-sm-3, .gx-sm-3`<br>@media (min-width: 576px) | src/bootstrap/reference/css/bootstrap-grid.css:429<br>src/bootstrap/reference/css/bootstrap.css:1163 | scss/mixins/_grid.scss:117<br>via scss/mixins/_grid.scss:115 | `$spacer` | Direct, Compile-time |
| `--bs-gutter-x` | `1rem` | `.g-xl-3, .gx-xl-3`<br>@media (min-width: 1200px) | src/bootstrap/reference/css/bootstrap-grid.css:936<br>src/bootstrap/reference/css/bootstrap.css:1670 | scss/mixins/_grid.scss:117<br>via scss/mixins/_grid.scss:115 | `$spacer` | Direct, Compile-time |
| `--bs-gutter-x` | `1rem` | `.g-xxl-3, .gx-xxl-3`<br>@media (min-width: 1400px) | src/bootstrap/reference/css/bootstrap-grid.css:1105<br>src/bootstrap/reference/css/bootstrap.css:1839 | scss/mixins/_grid.scss:117<br>via scss/mixins/_grid.scss:115 | `$spacer` | Direct, Compile-time |
| `--bs-gutter-y` | `1rem` | `.g-3, .gy-3`<br>(none) | src/bootstrap/reference/css/bootstrap-grid.css:260<br>src/bootstrap/reference/css/bootstrap.css:994 | scss/mixins/_grid.scss:122<br>via scss/mixins/_grid.scss:120 | `$spacer` | Direct, Compile-time |
| `--bs-gutter-y` | `1rem` | `.g-lg-3, .gy-lg-3`<br>@media (min-width: 992px) | src/bootstrap/reference/css/bootstrap-grid.css:771<br>src/bootstrap/reference/css/bootstrap.css:1505 | scss/mixins/_grid.scss:122<br>via scss/mixins/_grid.scss:120 | `$spacer` | Direct, Compile-time |
| `--bs-gutter-y` | `1rem` | `.g-md-3, .gy-md-3`<br>@media (min-width: 768px) | src/bootstrap/reference/css/bootstrap-grid.css:602<br>src/bootstrap/reference/css/bootstrap.css:1336 | scss/mixins/_grid.scss:122<br>via scss/mixins/_grid.scss:120 | `$spacer` | Direct, Compile-time |
| `--bs-gutter-y` | `1rem` | `.g-sm-3, .gy-sm-3`<br>@media (min-width: 576px) | src/bootstrap/reference/css/bootstrap-grid.css:433<br>src/bootstrap/reference/css/bootstrap.css:1167 | scss/mixins/_grid.scss:122<br>via scss/mixins/_grid.scss:120 | `$spacer` | Direct, Compile-time |
| `--bs-gutter-y` | `1rem` | `.g-xl-3, .gy-xl-3`<br>@media (min-width: 1200px) | src/bootstrap/reference/css/bootstrap-grid.css:940<br>src/bootstrap/reference/css/bootstrap.css:1674 | scss/mixins/_grid.scss:122<br>via scss/mixins/_grid.scss:120 | `$spacer` | Direct, Compile-time |
| `--bs-gutter-y` | `1rem` | `.g-xxl-3, .gy-xxl-3`<br>@media (min-width: 1400px) | src/bootstrap/reference/css/bootstrap-grid.css:1109<br>src/bootstrap/reference/css/bootstrap.css:1843 | scss/mixins/_grid.scss:122<br>via scss/mixins/_grid.scss:120 | `$spacer` | Direct, Compile-time |
| `--bs-list-group-item-padding-x` | `1rem` | `.list-group`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4999 | scss/_list-group.scss:12<br>via scss/_list-group.scss:5 | `$spacer` | Direct |
| `--bs-modal-header-padding-x` | `1rem` | `.modal`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5467 | scss/_modal.scss:23<br>via scss/_modal.scss:10 | `$spacer` | Direct |
| `--bs-modal-header-padding-y` | `1rem` | `.modal`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5468 | scss/_modal.scss:24<br>via scss/_modal.scss:10 | `$spacer` | Direct |
| `--bs-modal-padding` | `1rem` | `.modal`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5458 | scss/_modal.scss:14<br>via scss/_modal.scss:10 | `$spacer` | Direct |
| `--bs-offcanvas-padding-x` | `1rem` | `.offcanvas, .offcanvas-xxl, .offcanvas-xl, .offcanvas-lg, .offcanvas-md, .offcanvas-sm`<br>(none) | src/bootstrap/reference/css/bootstrap.css:6286 | scss/_offcanvas.scss:8<br>via scss/_offcanvas.scss:3 | `$spacer` | Direct |
| `--bs-offcanvas-padding-y` | `1rem` | `.offcanvas, .offcanvas-xxl, .offcanvas-xl, .offcanvas-lg, .offcanvas-md, .offcanvas-sm`<br>(none) | src/bootstrap/reference/css/bootstrap.css:6287 | scss/_offcanvas.scss:9<br>via scss/_offcanvas.scss:3 | `$spacer` | Direct |
| `--bs-popover-body-padding-x` | `1rem` | `.popover`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5873 | scss/_popover.scss:17<br>via scss/_popover.scss:1 | `$spacer` | Direct |
| `--bs-popover-body-padding-y` | `1rem` | `.popover`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5874 | scss/_popover.scss:18<br>via scss/_popover.scss:1 | `$spacer` | Direct |
| `--bs-popover-header-padding-x` | `1rem` | `.popover`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5868 | scss/_popover.scss:12<br>via scss/_popover.scss:1 | `$spacer` | Direct |

<a id="c-spacer-2"></a>
#### $spacer – pattern 2: `($ANCHOR * .5)`

18 contexts, 8 properties. Relationship: Compile-time, Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-card-cap-padding-y` | `0.5rem` | `.card`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4370 | scss/_card.scss:17<br>via scss/_card.scss:5 | `($spacer * .5)` | Direct |
| `--bs-card-title-spacer-y` | `0.5rem` | `.card`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4362 | scss/_card.scss:9<br>via scss/_card.scss:5 | `($spacer * .5)` | Direct |
| `--bs-dropdown-divider-margin-y` | `0.5rem` | `.dropdown-menu`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3419 | scss/_dropdown.scss:34<br>via scss/_dropdown.scss:19 | `($spacer * .5)` | Direct |
| `--bs-gutter-x` | `0.5rem` | `.g-2, .gx-2`<br>(none) | src/bootstrap/reference/css/bootstrap-grid.css:245<br>src/bootstrap/reference/css/bootstrap.css:979 | scss/mixins/_grid.scss:117<br>via scss/mixins/_grid.scss:115 | `($spacer * .5)` | Direct, Compile-time |
| `--bs-gutter-x` | `0.5rem` | `.g-lg-2, .gx-lg-2`<br>@media (min-width: 992px) | src/bootstrap/reference/css/bootstrap-grid.css:759<br>src/bootstrap/reference/css/bootstrap.css:1493 | scss/mixins/_grid.scss:117<br>via scss/mixins/_grid.scss:115 | `($spacer * .5)` | Direct, Compile-time |
| `--bs-gutter-x` | `0.5rem` | `.g-md-2, .gx-md-2`<br>@media (min-width: 768px) | src/bootstrap/reference/css/bootstrap-grid.css:590<br>src/bootstrap/reference/css/bootstrap.css:1324 | scss/mixins/_grid.scss:117<br>via scss/mixins/_grid.scss:115 | `($spacer * .5)` | Direct, Compile-time |
| `--bs-gutter-x` | `0.5rem` | `.g-sm-2, .gx-sm-2`<br>@media (min-width: 576px) | src/bootstrap/reference/css/bootstrap-grid.css:421<br>src/bootstrap/reference/css/bootstrap.css:1155 | scss/mixins/_grid.scss:117<br>via scss/mixins/_grid.scss:115 | `($spacer * .5)` | Direct, Compile-time |
| `--bs-gutter-x` | `0.5rem` | `.g-xl-2, .gx-xl-2`<br>@media (min-width: 1200px) | src/bootstrap/reference/css/bootstrap-grid.css:928<br>src/bootstrap/reference/css/bootstrap.css:1662 | scss/mixins/_grid.scss:117<br>via scss/mixins/_grid.scss:115 | `($spacer * .5)` | Direct, Compile-time |
| `--bs-gutter-x` | `0.5rem` | `.g-xxl-2, .gx-xxl-2`<br>@media (min-width: 1400px) | src/bootstrap/reference/css/bootstrap-grid.css:1097<br>src/bootstrap/reference/css/bootstrap.css:1831 | scss/mixins/_grid.scss:117<br>via scss/mixins/_grid.scss:115 | `($spacer * .5)` | Direct, Compile-time |
| `--bs-gutter-y` | `0.5rem` | `.g-2, .gy-2`<br>(none) | src/bootstrap/reference/css/bootstrap-grid.css:250<br>src/bootstrap/reference/css/bootstrap.css:984 | scss/mixins/_grid.scss:122<br>via scss/mixins/_grid.scss:120 | `($spacer * .5)` | Direct, Compile-time |
| `--bs-gutter-y` | `0.5rem` | `.g-lg-2, .gy-lg-2`<br>@media (min-width: 992px) | src/bootstrap/reference/css/bootstrap-grid.css:763<br>src/bootstrap/reference/css/bootstrap.css:1497 | scss/mixins/_grid.scss:122<br>via scss/mixins/_grid.scss:120 | `($spacer * .5)` | Direct, Compile-time |
| `--bs-gutter-y` | `0.5rem` | `.g-md-2, .gy-md-2`<br>@media (min-width: 768px) | src/bootstrap/reference/css/bootstrap-grid.css:594<br>src/bootstrap/reference/css/bootstrap.css:1328 | scss/mixins/_grid.scss:122<br>via scss/mixins/_grid.scss:120 | `($spacer * .5)` | Direct, Compile-time |
| `--bs-gutter-y` | `0.5rem` | `.g-sm-2, .gy-sm-2`<br>@media (min-width: 576px) | src/bootstrap/reference/css/bootstrap-grid.css:425<br>src/bootstrap/reference/css/bootstrap.css:1159 | scss/mixins/_grid.scss:122<br>via scss/mixins/_grid.scss:120 | `($spacer * .5)` | Direct, Compile-time |
| `--bs-gutter-y` | `0.5rem` | `.g-xl-2, .gy-xl-2`<br>@media (min-width: 1200px) | src/bootstrap/reference/css/bootstrap-grid.css:932<br>src/bootstrap/reference/css/bootstrap.css:1666 | scss/mixins/_grid.scss:122<br>via scss/mixins/_grid.scss:120 | `($spacer * .5)` | Direct, Compile-time |
| `--bs-gutter-y` | `0.5rem` | `.g-xxl-2, .gy-xxl-2`<br>@media (min-width: 1400px) | src/bootstrap/reference/css/bootstrap-grid.css:1101<br>src/bootstrap/reference/css/bootstrap.css:1835 | scss/mixins/_grid.scss:122<br>via scss/mixins/_grid.scss:120 | `($spacer * .5)` | Direct, Compile-time |
| `--bs-list-group-item-padding-y` | `0.5rem` | `.list-group`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5000 | scss/_list-group.scss:13<br>via scss/_list-group.scss:5 | `($spacer * .5)` | Direct |
| `--bs-navbar-padding-y` | `0.5rem` | `.navbar`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3934 | scss/_navbar.scss:9<br>via scss/_navbar.scss:6 | `($spacer * .5)` | Direct |
| `--bs-tooltip-padding-x` | `0.5rem` | `.tooltip`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5759 | scss/_tooltip.scss:6<br>via scss/_tooltip.scss:2 | `($spacer * .5)` | Direct |

<a id="c-spacer-3"></a>
#### $spacer – pattern 3: `($ANCHOR * .25)`

14 contexts, 4 properties. Relationship: Compile-time, Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-dropdown-item-padding-y` | `0.25rem` | `.dropdown-menu`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3428 | scss/_dropdown.scss:43<br>via scss/_dropdown.scss:19 | `($spacer * .25)` | Direct |
| `--bs-gutter-x` | `0.25rem` | `.g-1, .gx-1`<br>(none) | src/bootstrap/reference/css/bootstrap-grid.css:235<br>src/bootstrap/reference/css/bootstrap.css:969 | scss/mixins/_grid.scss:117<br>via scss/mixins/_grid.scss:115 | `($spacer * .25)` | Direct, Compile-time |
| `--bs-gutter-x` | `0.25rem` | `.g-lg-1, .gx-lg-1`<br>@media (min-width: 992px) | src/bootstrap/reference/css/bootstrap-grid.css:751<br>src/bootstrap/reference/css/bootstrap.css:1485 | scss/mixins/_grid.scss:117<br>via scss/mixins/_grid.scss:115 | `($spacer * .25)` | Direct, Compile-time |
| `--bs-gutter-x` | `0.25rem` | `.g-md-1, .gx-md-1`<br>@media (min-width: 768px) | src/bootstrap/reference/css/bootstrap-grid.css:582<br>src/bootstrap/reference/css/bootstrap.css:1316 | scss/mixins/_grid.scss:117<br>via scss/mixins/_grid.scss:115 | `($spacer * .25)` | Direct, Compile-time |
| `--bs-gutter-x` | `0.25rem` | `.g-sm-1, .gx-sm-1`<br>@media (min-width: 576px) | src/bootstrap/reference/css/bootstrap-grid.css:413<br>src/bootstrap/reference/css/bootstrap.css:1147 | scss/mixins/_grid.scss:117<br>via scss/mixins/_grid.scss:115 | `($spacer * .25)` | Direct, Compile-time |
| `--bs-gutter-x` | `0.25rem` | `.g-xl-1, .gx-xl-1`<br>@media (min-width: 1200px) | src/bootstrap/reference/css/bootstrap-grid.css:920<br>src/bootstrap/reference/css/bootstrap.css:1654 | scss/mixins/_grid.scss:117<br>via scss/mixins/_grid.scss:115 | `($spacer * .25)` | Direct, Compile-time |
| `--bs-gutter-x` | `0.25rem` | `.g-xxl-1, .gx-xxl-1`<br>@media (min-width: 1400px) | src/bootstrap/reference/css/bootstrap-grid.css:1089<br>src/bootstrap/reference/css/bootstrap.css:1823 | scss/mixins/_grid.scss:117<br>via scss/mixins/_grid.scss:115 | `($spacer * .25)` | Direct, Compile-time |
| `--bs-gutter-y` | `0.25rem` | `.g-1, .gy-1`<br>(none) | src/bootstrap/reference/css/bootstrap-grid.css:240<br>src/bootstrap/reference/css/bootstrap.css:974 | scss/mixins/_grid.scss:122<br>via scss/mixins/_grid.scss:120 | `($spacer * .25)` | Direct, Compile-time |
| `--bs-gutter-y` | `0.25rem` | `.g-lg-1, .gy-lg-1`<br>@media (min-width: 992px) | src/bootstrap/reference/css/bootstrap-grid.css:755<br>src/bootstrap/reference/css/bootstrap.css:1489 | scss/mixins/_grid.scss:122<br>via scss/mixins/_grid.scss:120 | `($spacer * .25)` | Direct, Compile-time |
| `--bs-gutter-y` | `0.25rem` | `.g-md-1, .gy-md-1`<br>@media (min-width: 768px) | src/bootstrap/reference/css/bootstrap-grid.css:586<br>src/bootstrap/reference/css/bootstrap.css:1320 | scss/mixins/_grid.scss:122<br>via scss/mixins/_grid.scss:120 | `($spacer * .25)` | Direct, Compile-time |
| `--bs-gutter-y` | `0.25rem` | `.g-sm-1, .gy-sm-1`<br>@media (min-width: 576px) | src/bootstrap/reference/css/bootstrap-grid.css:417<br>src/bootstrap/reference/css/bootstrap.css:1151 | scss/mixins/_grid.scss:122<br>via scss/mixins/_grid.scss:120 | `($spacer * .25)` | Direct, Compile-time |
| `--bs-gutter-y` | `0.25rem` | `.g-xl-1, .gy-xl-1`<br>@media (min-width: 1200px) | src/bootstrap/reference/css/bootstrap-grid.css:924<br>src/bootstrap/reference/css/bootstrap.css:1658 | scss/mixins/_grid.scss:122<br>via scss/mixins/_grid.scss:120 | `($spacer * .25)` | Direct, Compile-time |
| `--bs-gutter-y` | `0.25rem` | `.g-xxl-1, .gy-xxl-1`<br>@media (min-width: 1400px) | src/bootstrap/reference/css/bootstrap-grid.css:1093<br>src/bootstrap/reference/css/bootstrap.css:1827 | scss/mixins/_grid.scss:122<br>via scss/mixins/_grid.scss:120 | `($spacer * .25)` | Direct, Compile-time |
| `--bs-tooltip-padding-y` | `0.25rem` | `.tooltip`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5760 | scss/_tooltip.scss:7<br>via scss/_tooltip.scss:2 | `($spacer * .25)` | Direct |

<a id="c-spacer-4"></a>
#### $spacer – pattern 4: `($ANCHOR * 1.5)`

12 contexts, 2 properties. Relationship: Compile-time, Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-gutter-x` | `1.5rem` | `.g-4, .gx-4`<br>(none) | src/bootstrap/reference/css/bootstrap-grid.css:265<br>src/bootstrap/reference/css/bootstrap.css:999 | scss/mixins/_grid.scss:117<br>via scss/mixins/_grid.scss:115 | `($spacer * 1.5)` | Direct, Compile-time |
| `--bs-gutter-x` | `1.5rem` | `.g-lg-4, .gx-lg-4`<br>@media (min-width: 992px) | src/bootstrap/reference/css/bootstrap-grid.css:775<br>src/bootstrap/reference/css/bootstrap.css:1509 | scss/mixins/_grid.scss:117<br>via scss/mixins/_grid.scss:115 | `($spacer * 1.5)` | Direct, Compile-time |
| `--bs-gutter-x` | `1.5rem` | `.g-md-4, .gx-md-4`<br>@media (min-width: 768px) | src/bootstrap/reference/css/bootstrap-grid.css:606<br>src/bootstrap/reference/css/bootstrap.css:1340 | scss/mixins/_grid.scss:117<br>via scss/mixins/_grid.scss:115 | `($spacer * 1.5)` | Direct, Compile-time |
| `--bs-gutter-x` | `1.5rem` | `.g-sm-4, .gx-sm-4`<br>@media (min-width: 576px) | src/bootstrap/reference/css/bootstrap-grid.css:437<br>src/bootstrap/reference/css/bootstrap.css:1171 | scss/mixins/_grid.scss:117<br>via scss/mixins/_grid.scss:115 | `($spacer * 1.5)` | Direct, Compile-time |
| `--bs-gutter-x` | `1.5rem` | `.g-xl-4, .gx-xl-4`<br>@media (min-width: 1200px) | src/bootstrap/reference/css/bootstrap-grid.css:944<br>src/bootstrap/reference/css/bootstrap.css:1678 | scss/mixins/_grid.scss:117<br>via scss/mixins/_grid.scss:115 | `($spacer * 1.5)` | Direct, Compile-time |
| `--bs-gutter-x` | `1.5rem` | `.g-xxl-4, .gx-xxl-4`<br>@media (min-width: 1400px) | src/bootstrap/reference/css/bootstrap-grid.css:1113<br>src/bootstrap/reference/css/bootstrap.css:1847 | scss/mixins/_grid.scss:117<br>via scss/mixins/_grid.scss:115 | `($spacer * 1.5)` | Direct, Compile-time |
| `--bs-gutter-y` | `1.5rem` | `.g-4, .gy-4`<br>(none) | src/bootstrap/reference/css/bootstrap-grid.css:270<br>src/bootstrap/reference/css/bootstrap.css:1004 | scss/mixins/_grid.scss:122<br>via scss/mixins/_grid.scss:120 | `($spacer * 1.5)` | Direct, Compile-time |
| `--bs-gutter-y` | `1.5rem` | `.g-lg-4, .gy-lg-4`<br>@media (min-width: 992px) | src/bootstrap/reference/css/bootstrap-grid.css:779<br>src/bootstrap/reference/css/bootstrap.css:1513 | scss/mixins/_grid.scss:122<br>via scss/mixins/_grid.scss:120 | `($spacer * 1.5)` | Direct, Compile-time |
| `--bs-gutter-y` | `1.5rem` | `.g-md-4, .gy-md-4`<br>@media (min-width: 768px) | src/bootstrap/reference/css/bootstrap-grid.css:610<br>src/bootstrap/reference/css/bootstrap.css:1344 | scss/mixins/_grid.scss:122<br>via scss/mixins/_grid.scss:120 | `($spacer * 1.5)` | Direct, Compile-time |
| `--bs-gutter-y` | `1.5rem` | `.g-sm-4, .gy-sm-4`<br>@media (min-width: 576px) | src/bootstrap/reference/css/bootstrap-grid.css:441<br>src/bootstrap/reference/css/bootstrap.css:1175 | scss/mixins/_grid.scss:122<br>via scss/mixins/_grid.scss:120 | `($spacer * 1.5)` | Direct, Compile-time |
| `--bs-gutter-y` | `1.5rem` | `.g-xl-4, .gy-xl-4`<br>@media (min-width: 1200px) | src/bootstrap/reference/css/bootstrap-grid.css:948<br>src/bootstrap/reference/css/bootstrap.css:1682 | scss/mixins/_grid.scss:122<br>via scss/mixins/_grid.scss:120 | `($spacer * 1.5)` | Direct, Compile-time |
| `--bs-gutter-y` | `1.5rem` | `.g-xxl-4, .gy-xxl-4`<br>@media (min-width: 1400px) | src/bootstrap/reference/css/bootstrap-grid.css:1117<br>src/bootstrap/reference/css/bootstrap.css:1851 | scss/mixins/_grid.scss:122<br>via scss/mixins/_grid.scss:120 | `($spacer * 1.5)` | Direct, Compile-time |

<a id="c-spacer-5"></a>
#### $spacer – pattern 5: `($ANCHOR * 3)`

12 contexts, 2 properties. Relationship: Compile-time, Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-gutter-x` | `3rem` | `.g-5, .gx-5`<br>(none) | src/bootstrap/reference/css/bootstrap-grid.css:275<br>src/bootstrap/reference/css/bootstrap.css:1009 | scss/mixins/_grid.scss:117<br>via scss/mixins/_grid.scss:115 | `($spacer * 3)` | Direct, Compile-time |
| `--bs-gutter-x` | `3rem` | `.g-lg-5, .gx-lg-5`<br>@media (min-width: 992px) | src/bootstrap/reference/css/bootstrap-grid.css:783<br>src/bootstrap/reference/css/bootstrap.css:1517 | scss/mixins/_grid.scss:117<br>via scss/mixins/_grid.scss:115 | `($spacer * 3)` | Direct, Compile-time |
| `--bs-gutter-x` | `3rem` | `.g-md-5, .gx-md-5`<br>@media (min-width: 768px) | src/bootstrap/reference/css/bootstrap-grid.css:614<br>src/bootstrap/reference/css/bootstrap.css:1348 | scss/mixins/_grid.scss:117<br>via scss/mixins/_grid.scss:115 | `($spacer * 3)` | Direct, Compile-time |
| `--bs-gutter-x` | `3rem` | `.g-sm-5, .gx-sm-5`<br>@media (min-width: 576px) | src/bootstrap/reference/css/bootstrap-grid.css:445<br>src/bootstrap/reference/css/bootstrap.css:1179 | scss/mixins/_grid.scss:117<br>via scss/mixins/_grid.scss:115 | `($spacer * 3)` | Direct, Compile-time |
| `--bs-gutter-x` | `3rem` | `.g-xl-5, .gx-xl-5`<br>@media (min-width: 1200px) | src/bootstrap/reference/css/bootstrap-grid.css:952<br>src/bootstrap/reference/css/bootstrap.css:1686 | scss/mixins/_grid.scss:117<br>via scss/mixins/_grid.scss:115 | `($spacer * 3)` | Direct, Compile-time |
| `--bs-gutter-x` | `3rem` | `.g-xxl-5, .gx-xxl-5`<br>@media (min-width: 1400px) | src/bootstrap/reference/css/bootstrap-grid.css:1121<br>src/bootstrap/reference/css/bootstrap.css:1855 | scss/mixins/_grid.scss:117<br>via scss/mixins/_grid.scss:115 | `($spacer * 3)` | Direct, Compile-time |
| `--bs-gutter-y` | `3rem` | `.g-5, .gy-5`<br>(none) | src/bootstrap/reference/css/bootstrap-grid.css:280<br>src/bootstrap/reference/css/bootstrap.css:1014 | scss/mixins/_grid.scss:122<br>via scss/mixins/_grid.scss:120 | `($spacer * 3)` | Direct, Compile-time |
| `--bs-gutter-y` | `3rem` | `.g-lg-5, .gy-lg-5`<br>@media (min-width: 992px) | src/bootstrap/reference/css/bootstrap-grid.css:787<br>src/bootstrap/reference/css/bootstrap.css:1521 | scss/mixins/_grid.scss:122<br>via scss/mixins/_grid.scss:120 | `($spacer * 3)` | Direct, Compile-time |
| `--bs-gutter-y` | `3rem` | `.g-md-5, .gy-md-5`<br>@media (min-width: 768px) | src/bootstrap/reference/css/bootstrap-grid.css:618<br>src/bootstrap/reference/css/bootstrap.css:1352 | scss/mixins/_grid.scss:122<br>via scss/mixins/_grid.scss:120 | `($spacer * 3)` | Direct, Compile-time |
| `--bs-gutter-y` | `3rem` | `.g-sm-5, .gy-sm-5`<br>@media (min-width: 576px) | src/bootstrap/reference/css/bootstrap-grid.css:449<br>src/bootstrap/reference/css/bootstrap.css:1183 | scss/mixins/_grid.scss:122<br>via scss/mixins/_grid.scss:120 | `($spacer * 3)` | Direct, Compile-time |
| `--bs-gutter-y` | `3rem` | `.g-xl-5, .gy-xl-5`<br>@media (min-width: 1200px) | src/bootstrap/reference/css/bootstrap-grid.css:956<br>src/bootstrap/reference/css/bootstrap.css:1690 | scss/mixins/_grid.scss:122<br>via scss/mixins/_grid.scss:120 | `($spacer * 3)` | Direct, Compile-time |
| `--bs-gutter-y` | `3rem` | `.g-xxl-5, .gy-xxl-5`<br>@media (min-width: 1400px) | src/bootstrap/reference/css/bootstrap-grid.css:1125<br>src/bootstrap/reference/css/bootstrap.css:1859 | scss/mixins/_grid.scss:122<br>via scss/mixins/_grid.scss:120 | `($spacer * 3)` | Direct, Compile-time |

<a id="c-spacer-6"></a>
#### $spacer – pattern 6: `($ANCHOR $ANCHOR)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-modal-header-padding` | `1rem 1rem` | `.modal`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5469 | scss/_modal.scss:25<br>via scss/_modal.scss:10 | `($spacer $spacer)` | Direct |

<a id="c-spacers-0"></a>
### `$spacers[0]` (spacing)

Definition: (definition not located by name). 12 contexts, 2 properties.

<a id="c-spacers-0-1"></a>
#### $spacers[0] – pattern 1: `$ANCHOR`

12 contexts, 2 properties. Relationship: Compile-time, Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-gutter-x` | `0` | `.g-0, .gx-0`<br>(none) | src/bootstrap/reference/css/bootstrap-grid.css:225<br>src/bootstrap/reference/css/bootstrap.css:959 | scss/mixins/_grid.scss:117<br>via scss/mixins/_grid.scss:115 | `$spacers[0]` | Direct, Compile-time |
| `--bs-gutter-x` | `0` | `.g-lg-0, .gx-lg-0`<br>@media (min-width: 992px) | src/bootstrap/reference/css/bootstrap-grid.css:743<br>src/bootstrap/reference/css/bootstrap.css:1477 | scss/mixins/_grid.scss:117<br>via scss/mixins/_grid.scss:115 | `$spacers[0]` | Direct, Compile-time |
| `--bs-gutter-x` | `0` | `.g-md-0, .gx-md-0`<br>@media (min-width: 768px) | src/bootstrap/reference/css/bootstrap-grid.css:574<br>src/bootstrap/reference/css/bootstrap.css:1308 | scss/mixins/_grid.scss:117<br>via scss/mixins/_grid.scss:115 | `$spacers[0]` | Direct, Compile-time |
| `--bs-gutter-x` | `0` | `.g-sm-0, .gx-sm-0`<br>@media (min-width: 576px) | src/bootstrap/reference/css/bootstrap-grid.css:405<br>src/bootstrap/reference/css/bootstrap.css:1139 | scss/mixins/_grid.scss:117<br>via scss/mixins/_grid.scss:115 | `$spacers[0]` | Direct, Compile-time |
| `--bs-gutter-x` | `0` | `.g-xl-0, .gx-xl-0`<br>@media (min-width: 1200px) | src/bootstrap/reference/css/bootstrap-grid.css:912<br>src/bootstrap/reference/css/bootstrap.css:1646 | scss/mixins/_grid.scss:117<br>via scss/mixins/_grid.scss:115 | `$spacers[0]` | Direct, Compile-time |
| `--bs-gutter-x` | `0` | `.g-xxl-0, .gx-xxl-0`<br>@media (min-width: 1400px) | src/bootstrap/reference/css/bootstrap-grid.css:1081<br>src/bootstrap/reference/css/bootstrap.css:1815 | scss/mixins/_grid.scss:117<br>via scss/mixins/_grid.scss:115 | `$spacers[0]` | Direct, Compile-time |
| `--bs-gutter-y` | `0` | `.g-0, .gy-0`<br>(none) | src/bootstrap/reference/css/bootstrap-grid.css:230<br>src/bootstrap/reference/css/bootstrap.css:964 | scss/mixins/_grid.scss:122<br>via scss/mixins/_grid.scss:120 | `$spacers[0]` | Direct, Compile-time |
| `--bs-gutter-y` | `0` | `.g-lg-0, .gy-lg-0`<br>@media (min-width: 992px) | src/bootstrap/reference/css/bootstrap-grid.css:747<br>src/bootstrap/reference/css/bootstrap.css:1481 | scss/mixins/_grid.scss:122<br>via scss/mixins/_grid.scss:120 | `$spacers[0]` | Direct, Compile-time |
| `--bs-gutter-y` | `0` | `.g-md-0, .gy-md-0`<br>@media (min-width: 768px) | src/bootstrap/reference/css/bootstrap-grid.css:578<br>src/bootstrap/reference/css/bootstrap.css:1312 | scss/mixins/_grid.scss:122<br>via scss/mixins/_grid.scss:120 | `$spacers[0]` | Direct, Compile-time |
| `--bs-gutter-y` | `0` | `.g-sm-0, .gy-sm-0`<br>@media (min-width: 576px) | src/bootstrap/reference/css/bootstrap-grid.css:409<br>src/bootstrap/reference/css/bootstrap.css:1143 | scss/mixins/_grid.scss:122<br>via scss/mixins/_grid.scss:120 | `$spacers[0]` | Direct, Compile-time |
| `--bs-gutter-y` | `0` | `.g-xl-0, .gy-xl-0`<br>@media (min-width: 1200px) | src/bootstrap/reference/css/bootstrap-grid.css:916<br>src/bootstrap/reference/css/bootstrap.css:1650 | scss/mixins/_grid.scss:122<br>via scss/mixins/_grid.scss:120 | `$spacers[0]` | Direct, Compile-time |
| `--bs-gutter-y` | `0` | `.g-xxl-0, .gy-xxl-0`<br>@media (min-width: 1400px) | src/bootstrap/reference/css/bootstrap-grid.css:1085<br>src/bootstrap/reference/css/bootstrap.css:1819 | scss/mixins/_grid.scss:122<br>via scss/mixins/_grid.scss:120 | `$spacers[0]` | Direct, Compile-time |

<a id="c-border-radius"></a>
### `$border-radius` (border-radius)

Definition: `$border-radius: .375rem` (`scss/_variables.scss:547`). 18 contexts, 18 properties.

<a id="c-border-radius-1"></a>
#### $border-radius – pattern 1: `var(--bs-border-radius)`

14 contexts, 14 properties. Relationship: Direct, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-accordion-border-radius` | `var(--bs-border-radius)` | `.accordion`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4552 | scss/_accordion.scss:12<br>via scss/_accordion.scss:5 | `var(--bs-border-radius)` | Direct, Runtime, Mixed |
| `--bs-alert-border-radius` | `var(--bs-border-radius)` | `.alert`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4844 | scss/_alert.scss:14<br>via scss/_alert.scss:5 | `var(--bs-border-radius)` | Direct, Runtime, Mixed |
| `--bs-badge-border-radius` | `var(--bs-border-radius)` | `.badge`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4815 | scss/_badge.scss:13<br>via scss/_badge.scss:6 | `var(--bs-border-radius)` | Direct, Runtime, Mixed |
| `--bs-btn-border-radius` | `var(--bs-border-radius)` | `.btn`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2964 | scss/_buttons.scss:17<br>via scss/_buttons.scss:5 | `var(--bs-border-radius)` | Direct, Runtime, Mixed |
| `--bs-card-border-radius` | `var(--bs-border-radius)` | `.card`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4367 | scss/_card.scss:14<br>via scss/_card.scss:5 | `var(--bs-border-radius)` | Direct, Runtime, Mixed |
| `--bs-dropdown-border-radius` | `var(--bs-border-radius)` | `.dropdown-menu`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3415 | scss/_dropdown.scss:30<br>via scss/_dropdown.scss:19 | `var(--bs-border-radius)` | Direct, Runtime, Mixed |
| `--bs-list-group-border-radius` | `var(--bs-border-radius)` | `.list-group`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4998 | scss/_list-group.scss:11<br>via scss/_list-group.scss:5 | `var(--bs-border-radius)` | Direct, Runtime, Mixed |
| `--bs-nav-pills-border-radius` | `var(--bs-border-radius)` | `.nav-pills`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3873 | scss/_nav.scss:107<br>via scss/_nav.scss:105 | `var(--bs-border-radius)` | Direct, Runtime, Mixed |
| `--bs-nav-tabs-border-radius` | `var(--bs-border-radius)` | `.nav-tabs`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3843 | scss/_nav.scss:63<br>via scss/_nav.scss:59 | `var(--bs-border-radius)` | Direct, Runtime, Mixed |
| `--bs-navbar-toggler-border-radius` | `var(--bs-border-radius)` | `.navbar`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3950 | scss/_navbar.scss:25<br>via scss/_navbar.scss:6 | `var(--bs-border-radius)` | Direct, Runtime, Mixed |
| `--bs-pagination-border-radius` | `var(--bs-border-radius)` | `.pagination`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4723 | scss/_pagination.scss:10<br>via scss/_pagination.scss:1 | `var(--bs-border-radius)` | Direct, Runtime, Mixed |
| `--bs-progress-border-radius` | `var(--bs-border-radius)` | `.progress, .progress-stacked`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4941 | scss/_progress.scss:17<br>via scss/_progress.scss:11 | `var(--bs-border-radius)` | Direct, Runtime, Mixed |
| `--bs-toast-border-radius` | `var(--bs-border-radius)` | `.toast`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5397 | scss/_toasts.scss:13<br>via scss/_toasts.scss:1 | `var(--bs-border-radius)` | Direct, Runtime, Mixed |
| `--bs-tooltip-border-radius` | `var(--bs-border-radius)` | `.tooltip`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5765 | scss/_tooltip.scss:12<br>via scss/_tooltip.scss:2 | `var(--bs-border-radius)` | Direct, Runtime, Mixed |

<a id="c-border-radius-2"></a>
#### $border-radius – pattern 2: `subtract(var(--bs-border-radius), var(--bs-border-width))`

2 contexts, 2 properties. Relationship: Direct, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-accordion-inner-border-radius` | `calc(var(--bs-border-radius) - (var(--bs-border-width)))` | `.accordion`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4553 | scss/_accordion.scss:13<br>via scss/_accordion.scss:5 | `subtract(var(--bs-border-radius), var(--bs-border-width))` | Direct, Runtime, Mixed |
| `--bs-card-inner-border-radius` | `calc(var(--bs-border-radius) - (var(--bs-border-width)))` | `.card`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4369 | scss/_card.scss:16<br>via scss/_card.scss:5 | `subtract(var(--bs-border-radius), var(--bs-border-width))` | Direct, Runtime, Mixed |

<a id="c-border-radius-3"></a>
#### $border-radius – pattern 3: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-border-radius` | `0.375rem` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:107<br>src/bootstrap/reference/css/bootstrap.css:108 | scss/_root.scss:103<br>via scss/_root.scss:1 | `$border-radius` | Direct |

<a id="c-border-radius-4"></a>
#### $border-radius – pattern 4: `calc(var(--bs-border-radius) - var(--bs-border-width))`

1 contexts, 1 properties. Relationship: Direct, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-dropdown-inner-border-radius` | `calc(var(--bs-border-radius) - var(--bs-border-width))` | `.dropdown-menu`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3417 | scss/_dropdown.scss:32<br>via scss/_dropdown.scss:19 | `calc(var(--bs-border-radius) - var(--bs-border-width))` | Direct, Runtime, Mixed |

<a id="c-border-radius-lg"></a>
### `$border-radius-lg` (border-radius)

Definition: `$border-radius-lg: .5rem` (`scss/_variables.scss:549`). 7 contexts, 7 properties.

<a id="c-border-radius-lg-1"></a>
#### $border-radius-lg – pattern 1: `var(--bs-border-radius-lg)`

4 contexts, 4 properties. Relationship: Direct, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-border-radius` | `var(--bs-border-radius-lg)` | `.btn-lg, .btn-group-lg > .btn`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3332 | scss/mixins/_buttons.scss:68<br>via scss/_buttons.scss:210<br>+ scss/_buttons.scss:211 | `var(--bs-border-radius-lg)` | Direct, Runtime, Mixed |
| `--bs-modal-border-radius` | `var(--bs-border-radius-lg)` | `.modal`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5464 | scss/_modal.scss:20<br>via scss/_modal.scss:10 | `var(--bs-border-radius-lg)` | Direct, Runtime, Mixed |
| `--bs-pagination-border-radius` | `var(--bs-border-radius-lg)` | `.pagination-lg`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4799 | scss/mixins/_pagination.scss:8<br>via scss/_pagination.scss:103<br>+ scss/_pagination.scss:104 | `var(--bs-border-radius-lg)` | Direct, Runtime, Mixed |
| `--bs-popover-border-radius` | `var(--bs-border-radius-lg)` | `.popover`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5865 | scss/_popover.scss:9<br>via scss/_popover.scss:1 | `var(--bs-border-radius-lg)` | Direct, Runtime, Mixed |

<a id="c-border-radius-lg-2"></a>
#### $border-radius-lg – pattern 2: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-border-radius-lg` | `0.5rem` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:109<br>src/bootstrap/reference/css/bootstrap.css:110 | scss/_root.scss:105<br>via scss/_root.scss:1 | `$border-radius-lg` | Direct |

<a id="c-border-radius-lg-3"></a>
#### $border-radius-lg – pattern 3: `calc(var(--bs-border-radius-lg) - var(--bs-border-width))`

1 contexts, 1 properties. Relationship: Direct, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-popover-inner-border-radius` | `calc(var(--bs-border-radius-lg) - var(--bs-border-width))` | `.popover`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5866 | scss/_popover.scss:10<br>via scss/_popover.scss:1 | `calc(var(--bs-border-radius-lg) - var(--bs-border-width))` | Direct, Runtime, Mixed |

<a id="c-border-radius-lg-4"></a>
#### $border-radius-lg – pattern 4: `subtract(var(--bs-border-radius-lg), var(--bs-border-width))`

1 contexts, 1 properties. Relationship: Direct, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-modal-inner-border-radius` | `calc(var(--bs-border-radius-lg) - (var(--bs-border-width)))` | `.modal`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5466 | scss/_modal.scss:22<br>via scss/_modal.scss:10 | `subtract(var(--bs-border-radius-lg), var(--bs-border-width))` | Direct, Runtime, Mixed |

<a id="c-border-radius-pill"></a>
### `$border-radius-pill` (border-radius)

Definition: `$border-radius-pill: 50rem` (`scss/_variables.scss:552`). 1 contexts, 1 properties.

<a id="c-border-radius-pill-1"></a>
#### $border-radius-pill – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-border-radius-pill` | `50rem` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:113<br>src/bootstrap/reference/css/bootstrap.css:114 | scss/_root.scss:109<br>via scss/_root.scss:1 | `$border-radius-pill` | Direct |

<a id="c-border-radius-sm"></a>
### `$border-radius-sm` (border-radius)

Definition: `$border-radius-sm: .25rem` (`scss/_variables.scss:548`). 3 contexts, 3 properties.

<a id="c-border-radius-sm-1"></a>
#### $border-radius-sm – pattern 1: `var(--bs-border-radius-sm)`

2 contexts, 2 properties. Relationship: Direct, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-border-radius` | `var(--bs-border-radius-sm)` | `.btn-sm, .btn-group-sm > .btn`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3339 | scss/mixins/_buttons.scss:68<br>via scss/_buttons.scss:214<br>+ scss/_buttons.scss:215 | `var(--bs-border-radius-sm)` | Direct, Runtime, Mixed |
| `--bs-pagination-border-radius` | `var(--bs-border-radius-sm)` | `.pagination-sm`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4806 | scss/mixins/_pagination.scss:8<br>via scss/_pagination.scss:107<br>+ scss/_pagination.scss:108 | `var(--bs-border-radius-sm)` | Direct, Runtime, Mixed |

<a id="c-border-radius-sm-2"></a>
#### $border-radius-sm – pattern 2: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-border-radius-sm` | `0.25rem` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:108<br>src/bootstrap/reference/css/bootstrap.css:109 | scss/_root.scss:104<br>via scss/_root.scss:1 | `$border-radius-sm` | Direct |

<a id="c-border-radius-xl"></a>
### `$border-radius-xl` (border-radius)

Definition: `$border-radius-xl: 1rem` (`scss/_variables.scss:550`). 1 contexts, 1 properties.

<a id="c-border-radius-xl-1"></a>
#### $border-radius-xl – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-border-radius-xl` | `1rem` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:110<br>src/bootstrap/reference/css/bootstrap.css:111 | scss/_root.scss:106<br>via scss/_root.scss:1 | `$border-radius-xl` | Direct |

<a id="c-border-radius-xxl"></a>
### `$border-radius-xxl` (border-radius)

Definition: `$border-radius-xxl: 2rem` (`scss/_variables.scss:551`). 2 contexts, 2 properties.

<a id="c-border-radius-xxl-1"></a>
#### $border-radius-xxl – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-border-radius-xxl` | `2rem` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:111<br>src/bootstrap/reference/css/bootstrap.css:112 | scss/_root.scss:107<br>via scss/_root.scss:1 | `$border-radius-xxl` | Direct |

<a id="c-border-radius-xxl-2"></a>
#### $border-radius-xxl – pattern 2: `var(--bs-border-radius-xxl)`

1 contexts, 1 properties. Relationship: Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-border-radius-2xl` | `var(--bs-border-radius-xxl)` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:112<br>src/bootstrap/reference/css/bootstrap.css:113 | scss/_root.scss:108<br>via scss/_root.scss:1 | `var(--bs-border-radius-xxl)` | Runtime, Mixed |

<a id="c-border-width"></a>
### `$border-width` (border-width)

Definition: `$border-width: 1px` (`scss/_variables.scss:533`). 15 contexts, 15 properties.

<a id="c-border-width-1"></a>
#### $border-width – pattern 1: `var(--bs-border-width)`

13 contexts, 13 properties. Relationship: Direct, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-accordion-border-width` | `var(--bs-border-width)` | `.accordion`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4551 | scss/_accordion.scss:11<br>via scss/_accordion.scss:5 | `var(--bs-border-width)` | Direct, Runtime, Mixed |
| `--bs-btn-border-width` | `var(--bs-border-width)` | `.btn`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2962 | scss/_buttons.scss:15<br>via scss/_buttons.scss:5 | `var(--bs-border-width)` | Direct, Runtime, Mixed |
| `--bs-card-border-width` | `var(--bs-border-width)` | `.card`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4365 | scss/_card.scss:12<br>via scss/_card.scss:5 | `var(--bs-border-width)` | Direct, Runtime, Mixed |
| `--bs-dropdown-border-width` | `var(--bs-border-width)` | `.dropdown-menu`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3416 | scss/_dropdown.scss:31<br>via scss/_dropdown.scss:19 | `var(--bs-border-width)` | Direct, Runtime, Mixed |
| `--bs-list-group-border-width` | `var(--bs-border-width)` | `.list-group`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4997 | scss/_list-group.scss:10<br>via scss/_list-group.scss:5 | `var(--bs-border-width)` | Direct, Runtime, Mixed |
| `--bs-modal-border-width` | `var(--bs-border-width)` | `.modal`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5463 | scss/_modal.scss:19<br>via scss/_modal.scss:10 | `var(--bs-border-width)` | Direct, Runtime, Mixed |
| `--bs-modal-footer-border-width` | `var(--bs-border-width)` | `.modal`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5476 | scss/_modal.scss:32<br>via scss/_modal.scss:10 | `var(--bs-border-width)` | Direct, Runtime, Mixed |
| `--bs-modal-header-border-width` | `var(--bs-border-width)` | `.modal`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5471 | scss/_modal.scss:27<br>via scss/_modal.scss:10 | `var(--bs-border-width)` | Direct, Runtime, Mixed |
| `--bs-nav-tabs-border-width` | `var(--bs-border-width)` | `.nav-tabs`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3841 | scss/_nav.scss:61<br>via scss/_nav.scss:59 | `var(--bs-border-width)` | Direct, Runtime, Mixed |
| `--bs-offcanvas-border-width` | `var(--bs-border-width)` | `.offcanvas, .offcanvas-xxl, .offcanvas-xl, .offcanvas-lg, .offcanvas-md, .offcanvas-sm`<br>(none) | src/bootstrap/reference/css/bootstrap.css:6290 | scss/_offcanvas.scss:12<br>via scss/_offcanvas.scss:3 | `var(--bs-border-width)` | Direct, Runtime, Mixed |
| `--bs-pagination-border-width` | `var(--bs-border-width)` | `.pagination`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4721 | scss/_pagination.scss:8<br>via scss/_pagination.scss:1 | `var(--bs-border-width)` | Direct, Runtime, Mixed |
| `--bs-popover-border-width` | `var(--bs-border-width)` | `.popover`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5863 | scss/_popover.scss:7<br>via scss/_popover.scss:1 | `var(--bs-border-width)` | Direct, Runtime, Mixed |
| `--bs-toast-border-width` | `var(--bs-border-width)` | `.toast`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5395 | scss/_toasts.scss:11<br>via scss/_toasts.scss:1 | `var(--bs-border-width)` | Direct, Runtime, Mixed |

<a id="c-border-width-2"></a>
#### $border-width – pattern 2: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-border-width` | `1px` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:103<br>src/bootstrap/reference/css/bootstrap.css:104 | scss/_root.scss:98<br>via scss/_root.scss:1 | `$border-width` | Direct |

<a id="c-border-width-3"></a>
#### $border-width – pattern 3: `var(--bs-border-width) solid var(--bs-alert-border-color)`

1 contexts, 1 properties. Relationship: Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-alert-border` | `var(--bs-border-width) solid var(--bs-alert-border-color)` | `.alert`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4843 | scss/_alert.scss:13<br>via scss/_alert.scss:5 | `var(--bs-border-width) solid var(--bs-alert-border-color)` | Runtime, Mixed |

<a id="c-font-family-monospace"></a>
### `$font-family-monospace` (typography)

Definition: `$font-family-monospace: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace` (`scss/_variables.scss:607`). 1 contexts, 1 properties.

<a id="c-font-family-monospace-1"></a>
#### $font-family-monospace – pattern 1: `inspect($ANCHOR)`

1 contexts, 1 properties. Relationship: Compile-time.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-font-monospace` | `SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:74<br>src/bootstrap/reference/css/bootstrap.css:75 | scss/_root.scss:45<br>via scss/_root.scss:1 | `inspect($font-family-monospace)` | Compile-time |

<a id="c-font-family-sans-serif"></a>
### `$font-family-sans-serif` (typography)

Definition: `$font-family-sans-serif: system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", "Noto Sans", "Liberation Sans", Arial, sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol", "Noto Color Emoji"` (`scss/_variables.scss:606`). 2 contexts, 2 properties.

<a id="c-font-family-sans-serif-1"></a>
#### $font-family-sans-serif – pattern 1: `inspect($ANCHOR)`

1 contexts, 1 properties. Relationship: Compile-time.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-font-sans-serif` | `system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", "Noto Sans", "Liberation Sans", Arial, sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol", "Noto Color Emoji"` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:73<br>src/bootstrap/reference/css/bootstrap.css:74 | scss/_root.scss:44<br>via scss/_root.scss:1 | `inspect($font-family-sans-serif)` | Compile-time |

<a id="c-font-family-sans-serif-2"></a>
#### $font-family-sans-serif – pattern 2: `inspect(var(--bs-font-sans-serif))`

1 contexts, 1 properties. Relationship: Compile-time, Mixed, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-body-font-family` | `var(--bs-font-sans-serif)` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:76<br>src/bootstrap/reference/css/bootstrap.css:77 | scss/_root.scss:53<br>via scss/_root.scss:1 | `inspect(var(--bs-font-sans-serif))` | Runtime, Compile-time, Mixed |

<a id="c-font-size-base"></a>
### `$font-size-base` (typography)

Definition: `$font-size-base: 1rem` (`scss/_variables.scss:615`). 15 contexts, 11 properties.

<a id="c-font-size-base-1"></a>
#### $font-size-base – pattern 1: `$ANCHOR`

5 contexts, 5 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-body-font-size` | `1rem` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:77<br>src/bootstrap/reference/css/bootstrap.css:78 | scss/vendor/_rfs.scss:288<br>via scss/_root.scss:1<br>+ scss/_root.scss:None | `$font-size-base` | Direct |
| `--bs-btn-font-size` | `1rem` | `.btn`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2957 | scss/vendor/_rfs.scss:288<br>via scss/_buttons.scss:5<br>+ scss/_buttons.scss:None | `$font-size-base` | Direct |
| `--bs-dropdown-font-size` | `1rem` | `.dropdown-menu`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3411 | scss/vendor/_rfs.scss:288<br>via scss/_dropdown.scss:19<br>+ scss/_dropdown.scss:None | `$font-size-base` | Direct |
| `--bs-pagination-font-size` | `1rem` | `.pagination`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4718 | scss/vendor/_rfs.scss:288<br>via scss/_pagination.scss:1<br>+ scss/_pagination.scss:None | `$font-size-base` | Direct |
| `--bs-popover-header-font-size` | `1rem` | `.popover`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5870 | scss/vendor/_rfs.scss:288<br>via scss/_popover.scss:1<br>+ scss/_popover.scss:None | `$font-size-base` | Direct |

<a id="c-font-size-base-2"></a>
#### $font-size-base – pattern 2: `($ANCHOR * .875)`

4 contexts, 4 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-font-size` | `0.875rem` | `.btn-sm, .btn-group-sm > .btn`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3338 | scss/vendor/_rfs.scss:288<br>via scss/_buttons.scss:214<br>+ scss/_buttons.scss:215 | `($font-size-base * .875)` | Direct |
| `--bs-pagination-font-size` | `0.875rem` | `.pagination-sm`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4805 | scss/vendor/_rfs.scss:288<br>via scss/_pagination.scss:107<br>+ scss/_pagination.scss:108 | `($font-size-base * .875)` | Direct |
| `--bs-popover-font-size` | `0.875rem` | `.popover`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5861 | scss/vendor/_rfs.scss:288<br>via scss/_popover.scss:1<br>+ scss/_popover.scss:None | `($font-size-base * .875)` | Direct |
| `--bs-tooltip-font-size` | `0.875rem` | `.tooltip`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5762 | scss/vendor/_rfs.scss:288<br>via scss/_tooltip.scss:2<br>+ scss/_tooltip.scss:None | `($font-size-base * .875)` | Direct |

<a id="c-font-size-base-3"></a>
#### $font-size-base – pattern 3: `($ANCHOR * 1.25)`

4 contexts, 4 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-font-size` | `1.25rem` | `.btn-lg, .btn-group-lg > .btn`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3331 | scss/vendor/_rfs.scss:288<br>via scss/_buttons.scss:210<br>+ scss/_buttons.scss:211 | `($font-size-base * 1.25)` | Direct |
| `--bs-navbar-brand-font-size` | `1.25rem` | `.navbar`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3941 | scss/_navbar.scss:16<br>via scss/_navbar.scss:6 | `($font-size-base * 1.25)` | Direct |
| `--bs-navbar-toggler-font-size` | `1.25rem` | `.navbar`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3947 | scss/_navbar.scss:22<br>via scss/_navbar.scss:6 | `($font-size-base * 1.25)` | Direct |
| `--bs-pagination-font-size` | `1.25rem` | `.pagination-lg`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4798 | scss/vendor/_rfs.scss:288<br>via scss/_pagination.scss:103<br>+ scss/_pagination.scss:104 | `($font-size-base * 1.25)` | Direct |

<a id="c-font-size-base-4"></a>
#### $font-size-base – pattern 4: `($ANCHOR * .75)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-progress-font-size` | `0.75rem` | `.progress, .progress-stacked`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4939 | scss/vendor/_rfs.scss:288<br>via scss/_progress.scss:11<br>+ scss/_progress.scss:None | `($font-size-base * .75)` | Direct |

<a id="c-font-size-base-5"></a>
#### $font-size-base – pattern 5: `((($ANCHOR * $line-height-base + $nav-link-padding-y * 2) - (($ANCHOR * 1.25) * $line-height-base)) * .5)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-navbar-brand-padding-y` | `0.3125rem` | `.navbar`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3939 | scss/_navbar.scss:14<br>via scss/_navbar.scss:6 | `((($font-size-base * $line-height-base + $nav-link-padding-y * 2) - (($font-size-base * 1.25) * $line-height-base)) * .5)` | Direct |

<a id="c-font-weight-bold"></a>
### `$font-weight-bold` (typography)

Definition: `$font-weight-bold: 700` (`scss/_variables.scss:624`). 1 contexts, 1 properties.

<a id="c-font-weight-bold-1"></a>
#### $font-weight-bold – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-badge-font-weight` | `700` | `.badge`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4813 | scss/_badge.scss:11<br>via scss/_badge.scss:6 | `$font-weight-bold` | Direct |

<a id="c-font-weight-normal"></a>
### `$font-weight-normal` (typography)

Definition: `$font-weight-normal: 400` (`scss/_variables.scss:621`). 3 contexts, 2 properties.

<a id="c-font-weight-normal-1"></a>
#### $font-weight-normal – pattern 1: `$ANCHOR`

3 contexts, 2 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-body-font-weight` | `400` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:78<br>src/bootstrap/reference/css/bootstrap.css:79 | scss/_root.scss:55<br>via scss/_root.scss:1 | `$font-weight-normal` | Direct |
| `--bs-btn-font-weight` | `400` | `.btn`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2958 | scss/_buttons.scss:11<br>via scss/_buttons.scss:5 | `$font-weight-normal` | Direct |
| `--bs-btn-font-weight` | `400` | `.btn-link`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3307 | scss/_buttons.scss:171<br>via scss/_buttons.scss:170 | `$font-weight-normal` | Direct |

<a id="c-headings-color"></a>
### `$headings-color` (typography)

Definition: `$headings-color: inherit` (`scss/_variables.scss:658`). 2 contexts, 2 properties.

<a id="c-headings-color-1"></a>
#### $headings-color – pattern 1: `$ANCHOR`

2 contexts, 2 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-heading-color` | `inherit` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:94<br>src/bootstrap/reference/css/bootstrap.css:95 | scss/_root.scss:80<br>via scss/_root.scss:1 | `$headings-color` | Direct |
| `--bs-popover-header-color` | `inherit` | `.popover`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5871 | scss/_popover.scss:15<br>via scss/_popover.scss:1 | `$headings-color` | Direct |

<a id="c-headings-color-dark"></a>
### `$headings-color-dark` (typography)

Definition: `$headings-color-dark: inherit` (`scss/_variables-dark.scss:52`). 1 contexts, 1 properties.

<a id="c-headings-color-dark-1"></a>
#### $headings-color-dark – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-heading-color` | `inherit` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:167<br>src/bootstrap/reference/css/bootstrap.css:168 | scss/_root.scss:167<br>via scss/mixins/_color-mode.scss:16 | `$headings-color-dark` | Direct |

<a id="c-line-height-base"></a>
### `$line-height-base` (typography)

Definition: `$line-height-base: 1.5` (`scss/_variables.scss:629`). 4 contexts, 4 properties.

<a id="c-line-height-base-1"></a>
#### $line-height-base – pattern 1: `$ANCHOR`

4 contexts, 4 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-body-line-height` | `1.5` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:79<br>src/bootstrap/reference/css/bootstrap.css:80 | scss/_root.scss:56<br>via scss/_root.scss:1 | `$line-height-base` | Direct |
| `--bs-btn-line-height` | `1.5` | `.btn`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2959 | scss/_buttons.scss:12<br>via scss/_buttons.scss:5 | `$line-height-base` | Direct |
| `--bs-modal-title-line-height` | `1.5` | `.modal`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5472 | scss/_modal.scss:28<br>via scss/_modal.scss:10 | `$line-height-base` | Direct |
| `--bs-offcanvas-title-line-height` | `1.5` | `.offcanvas, .offcanvas-xxl, .offcanvas-xl, .offcanvas-lg, .offcanvas-md, .offcanvas-sm`<br>(none) | src/bootstrap/reference/css/bootstrap.css:6294 | scss/_offcanvas.scss:16<br>via scss/_offcanvas.scss:3 | `$line-height-base` | Direct |

<a id="c-focus-ring-opacity"></a>
### `$focus-ring-opacity` (focus)

Definition: `$focus-ring-opacity: .25` (`scss/_variables.scss:570`). 1 contexts, 1 properties.

<a id="c-focus-ring-opacity-1"></a>
#### $focus-ring-opacity – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-focus-ring-opacity` | `0.25` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:119<br>src/bootstrap/reference/css/bootstrap.css:120 | scss/_root.scss:120<br>via scss/_root.scss:1 | `$focus-ring-opacity` | Direct |

<a id="c-focus-ring-width"></a>
### `$focus-ring-width` (focus)

Definition: `$focus-ring-width: .25rem` (`scss/_variables.scss:569`). 3 contexts, 3 properties.

<a id="c-focus-ring-width-1"></a>
#### $focus-ring-width – pattern 1: `$ANCHOR`

2 contexts, 2 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-focus-ring-width` | `0.25rem` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:118<br>src/bootstrap/reference/css/bootstrap.css:119 | scss/_root.scss:119<br>via scss/_root.scss:1 | `$focus-ring-width` | Direct |
| `--bs-navbar-toggler-focus-width` | `0.25rem` | `.navbar`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3951 | scss/_navbar.scss:26<br>via scss/_navbar.scss:6 | `$focus-ring-width` | Direct |

<a id="c-focus-ring-width-2"></a>
#### $focus-ring-width – pattern 2: `0 0 0 $ANCHOR rgba(var(--bs-btn-focus-shadow-rgb), .5)`

1 contexts, 1 properties. Relationship: Compile-time, Derived, Runtime.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-focus-box-shadow` | `0 0 0 0.25rem rgba(var(--bs-btn-focus-shadow-rgb), .5)` | `.btn`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2968 | scss/_buttons.scss:21<br>via scss/_buttons.scss:5 | `0 0 0 $focus-ring-width rgba(var(--bs-btn-focus-shadow-rgb), .5)` | Derived, Runtime, Compile-time |

<a id="c-zindex-dropdown"></a>
### `$zindex-dropdown` (z-index)

Definition: `$zindex-dropdown: 1000` (`scss/_variables.scss:1133`). 1 contexts, 1 properties.

<a id="c-zindex-dropdown-1"></a>
#### $zindex-dropdown – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-dropdown-zindex` | `1000` | `.dropdown-menu`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3406 | scss/_dropdown.scss:21<br>via scss/_dropdown.scss:19 | `$zindex-dropdown` | Direct |

<a id="c-zindex-modal"></a>
### `$zindex-modal` (z-index)

Definition: `$zindex-modal: 1055` (`scss/_variables.scss:1139`). 1 contexts, 1 properties.

<a id="c-zindex-modal-1"></a>
#### $zindex-modal – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-modal-zindex` | `1055` | `.modal`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5456 | scss/_modal.scss:12<br>via scss/_modal.scss:10 | `$zindex-modal` | Direct |

<a id="c-zindex-modal-backdrop"></a>
### `$zindex-modal-backdrop` (z-index)

Definition: `$zindex-modal-backdrop: 1050` (`scss/_variables.scss:1138`). 1 contexts, 1 properties.

<a id="c-zindex-modal-backdrop-1"></a>
#### $zindex-modal-backdrop – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-backdrop-zindex` | `1050` | `.modal-backdrop`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5543 | scss/_modal.scss:115<br>via scss/_modal.scss:113 | `$zindex-modal-backdrop` | Direct |

<a id="c-zindex-offcanvas"></a>
### `$zindex-offcanvas` (z-index)

Definition: `$zindex-offcanvas: 1045` (`scss/_variables.scss:1137`). 1 contexts, 1 properties.

<a id="c-zindex-offcanvas-1"></a>
#### $zindex-offcanvas – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-offcanvas-zindex` | `1045` | `.offcanvas, .offcanvas-xxl, .offcanvas-xl, .offcanvas-lg, .offcanvas-md, .offcanvas-sm`<br>(none) | src/bootstrap/reference/css/bootstrap.css:6283 | scss/_offcanvas.scss:5<br>via scss/_offcanvas.scss:3 | `$zindex-offcanvas` | Direct |

<a id="c-zindex-popover"></a>
### `$zindex-popover` (z-index)

Definition: `$zindex-popover: 1070` (`scss/_variables.scss:1140`). 1 contexts, 1 properties.

<a id="c-zindex-popover-1"></a>
#### $zindex-popover – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-popover-zindex` | `1070` | `.popover`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5859 | scss/_popover.scss:3<br>via scss/_popover.scss:1 | `$zindex-popover` | Direct |

<a id="c-zindex-toast"></a>
### `$zindex-toast` (z-index)

Definition: `$zindex-toast: 1090` (`scss/_variables.scss:1142`). 2 contexts, 1 properties.

<a id="c-zindex-toast-1"></a>
#### $zindex-toast – pattern 1: `$ANCHOR`

2 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-toast-zindex` | `1090` | `.toast`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5387 | scss/_toasts.scss:3<br>via scss/_toasts.scss:1 | `$zindex-toast` | Direct |
| `--bs-toast-zindex` | `1090` | `.toast-container`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5421 | scss/_toasts.scss:41<br>via scss/_toasts.scss:40 | `$zindex-toast` | Direct |

<a id="c-zindex-tooltip"></a>
### `$zindex-tooltip` (z-index)

Definition: `$zindex-tooltip: 1080` (`scss/_variables.scss:1141`). 1 contexts, 1 properties.

<a id="c-zindex-tooltip-1"></a>
#### $zindex-tooltip – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-tooltip-zindex` | `1080` | `.tooltip`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5757 | scss/_tooltip.scss:4<br>via scss/_tooltip.scss:2 | `$zindex-tooltip` | Direct |

<a id="c-grid-breakpoints-lg"></a>
### `$grid-breakpoints[lg]` (breakpoints/grid)

Definition: (definition not located by name). 1 contexts, 1 properties.

<a id="c-grid-breakpoints-lg-1"></a>
#### $grid-breakpoints[lg] – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Compile-time, Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-breakpoint-lg` | `992px` | `:root`<br>(none) | src/bootstrap/reference/css/bootstrap-grid.css:51<br>src/bootstrap/reference/css/bootstrap.css:786 | scss/_grid.scss:7<br>via scss/_grid.scss:5 | `$grid-breakpoints[lg]` | Direct, Compile-time |

<a id="c-grid-breakpoints-md"></a>
### `$grid-breakpoints[md]` (breakpoints/grid)

Definition: (definition not located by name). 1 contexts, 1 properties.

<a id="c-grid-breakpoints-md-1"></a>
#### $grid-breakpoints[md] – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Compile-time, Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-breakpoint-md` | `768px` | `:root`<br>(none) | src/bootstrap/reference/css/bootstrap-grid.css:50<br>src/bootstrap/reference/css/bootstrap.css:785 | scss/_grid.scss:7<br>via scss/_grid.scss:5 | `$grid-breakpoints[md]` | Direct, Compile-time |

<a id="c-grid-breakpoints-sm"></a>
### `$grid-breakpoints[sm]` (breakpoints/grid)

Definition: (definition not located by name). 1 contexts, 1 properties.

<a id="c-grid-breakpoints-sm-1"></a>
#### $grid-breakpoints[sm] – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Compile-time, Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-breakpoint-sm` | `576px` | `:root`<br>(none) | src/bootstrap/reference/css/bootstrap-grid.css:49<br>src/bootstrap/reference/css/bootstrap.css:784 | scss/_grid.scss:7<br>via scss/_grid.scss:5 | `$grid-breakpoints[sm]` | Direct, Compile-time |

<a id="c-grid-breakpoints-xl"></a>
### `$grid-breakpoints[xl]` (breakpoints/grid)

Definition: (definition not located by name). 1 contexts, 1 properties.

<a id="c-grid-breakpoints-xl-1"></a>
#### $grid-breakpoints[xl] – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Compile-time, Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-breakpoint-xl` | `1200px` | `:root`<br>(none) | src/bootstrap/reference/css/bootstrap-grid.css:52<br>src/bootstrap/reference/css/bootstrap.css:787 | scss/_grid.scss:7<br>via scss/_grid.scss:5 | `$grid-breakpoints[xl]` | Direct, Compile-time |

<a id="c-grid-breakpoints-xs"></a>
### `$grid-breakpoints[xs]` (breakpoints/grid)

Definition: (definition not located by name). 1 contexts, 1 properties.

<a id="c-grid-breakpoints-xs-1"></a>
#### $grid-breakpoints[xs] – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Compile-time, Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-breakpoint-xs` | `0` | `:root`<br>(none) | src/bootstrap/reference/css/bootstrap-grid.css:48<br>src/bootstrap/reference/css/bootstrap.css:783 | scss/_grid.scss:7<br>via scss/_grid.scss:5 | `$grid-breakpoints[xs]` | Direct, Compile-time |

<a id="c-grid-breakpoints-xxl"></a>
### `$grid-breakpoints[xxl]` (breakpoints/grid)

Definition: (definition not located by name). 1 contexts, 1 properties.

<a id="c-grid-breakpoints-xxl-1"></a>
#### $grid-breakpoints[xxl] – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Compile-time, Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-breakpoint-xxl` | `1400px` | `:root`<br>(none) | src/bootstrap/reference/css/bootstrap-grid.css:53<br>src/bootstrap/reference/css/bootstrap.css:788 | scss/_grid.scss:7<br>via scss/_grid.scss:5 | `$grid-breakpoints[xxl]` | Direct, Compile-time |

<a id="c-grid-gutter-width"></a>
### `$grid-gutter-width` (breakpoints/grid)

Definition: `$grid-gutter-width: 1.5rem` (`scss/_variables.scss:520`). 4 contexts, 3 properties.

<a id="c-grid-gutter-width-1"></a>
#### $grid-gutter-width – pattern 1: `$ANCHOR`

3 contexts, 2 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-gutter-x` | `1.5rem` | `.container, .container-fluid, .container-xxl, .container-xl, .container-lg, .container-md, .container-sm`<br>(none) | src/bootstrap/reference/css/bootstrap-grid.css:13<br>src/bootstrap/reference/css/bootstrap.css:748 | scss/mixins/_container.scss:4<br>via scss/_containers.scss:7<br>+ scss/_containers.scss:10 | `$grid-gutter-width` | Direct |
| `--bs-gutter-x` | `1.5rem` | `.row`<br>(none) | src/bootstrap/reference/css/bootstrap-grid.css:57<br>src/bootstrap/reference/css/bootstrap.css:792 | scss/mixins/_grid.scss:6<br>via scss/_grid.scss:12<br>+ scss/_grid.scss:13 | `$grid-gutter-width` | Direct |
| `--bs-toast-spacing` | `1.5rem` | `.toast`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5390 | scss/_toasts.scss:6<br>via scss/_toasts.scss:1 | `$grid-gutter-width` | Direct |

<a id="c-grid-gutter-width-2"></a>
#### $grid-gutter-width – pattern 2: `($ANCHOR * .5)`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-card-group-margin` | `0.75rem` | `.card`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4378 | scss/_card.scss:25<br>via scss/_card.scss:5 | `($grid-gutter-width * .5)` | Direct |

<a id="c-aspect-ratios-16x9"></a>
### `$aspect-ratios[16x9]` (aspect ratio)

Definition: (definition not located by name). 1 contexts, 1 properties.

<a id="c-aspect-ratios-16x9-1"></a>
#### $aspect-ratios[16x9] – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Compile-time, Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-aspect-ratio` | `56.25%` | `.ratio-16x9`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:386<br>src/bootstrap/reference/css/bootstrap.css:7045 | scss/helpers/_ratio.scss:24<br>via scss/helpers/_ratio.scss:23 | `$aspect-ratios[16x9]` | Direct, Compile-time |

<a id="c-aspect-ratios-1x1"></a>
### `$aspect-ratios[1x1]` (aspect ratio)

Definition: (definition not located by name). 1 contexts, 1 properties.

<a id="c-aspect-ratios-1x1-1"></a>
#### $aspect-ratios[1x1] – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Compile-time, Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-aspect-ratio` | `100%` | `.ratio-1x1`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:378<br>src/bootstrap/reference/css/bootstrap.css:7037 | scss/helpers/_ratio.scss:24<br>via scss/helpers/_ratio.scss:23 | `$aspect-ratios[1x1]` | Direct, Compile-time |

<a id="c-aspect-ratios-21x9"></a>
### `$aspect-ratios[21x9]` (aspect ratio)

Definition: (definition not located by name). 1 contexts, 1 properties.

<a id="c-aspect-ratios-21x9-1"></a>
#### $aspect-ratios[21x9] – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Compile-time, Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-aspect-ratio` | `42.8571428571%` | `.ratio-21x9`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:390<br>src/bootstrap/reference/css/bootstrap.css:7049 | scss/helpers/_ratio.scss:24<br>via scss/helpers/_ratio.scss:23 | `$aspect-ratios[21x9]` | Direct, Compile-time |

<a id="c-aspect-ratios-4x3"></a>
### `$aspect-ratios[4x3]` (aspect ratio)

Definition: (definition not located by name). 1 contexts, 1 properties.

<a id="c-aspect-ratios-4x3-1"></a>
#### $aspect-ratios[4x3] – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Compile-time, Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-aspect-ratio` | `75%` | `.ratio-4x3`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:382<br>src/bootstrap/reference/css/bootstrap.css:7041 | scss/helpers/_ratio.scss:24<br>via scss/helpers/_ratio.scss:23 | `$aspect-ratios[4x3]` | Direct, Compile-time |

<a id="c-btn-close-disabled-opacity"></a>
### `$btn-close-disabled-opacity` (opacity)

Definition: `$btn-close-disabled-opacity: .25` (`scss/_variables.scss:1716`). 1 contexts, 1 properties.

<a id="c-btn-close-disabled-opacity-1"></a>
#### $btn-close-disabled-opacity – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-close-disabled-opacity` | `0.25` | `.btn-close`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5343 | scss/_close.scss:14<br>via scss/_close.scss:6 | `$btn-close-disabled-opacity` | Direct |

<a id="c-btn-close-focus-opacity"></a>
### `$btn-close-focus-opacity` (opacity)

Definition: `$btn-close-focus-opacity: 1` (`scss/_variables.scss:1715`). 1 contexts, 1 properties.

<a id="c-btn-close-focus-opacity-1"></a>
#### $btn-close-focus-opacity – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-close-focus-opacity` | `1` | `.btn-close`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5342 | scss/_close.scss:13<br>via scss/_close.scss:6 | `$btn-close-focus-opacity` | Direct |

<a id="c-btn-close-hover-opacity"></a>
### `$btn-close-hover-opacity` (opacity)

Definition: `$btn-close-hover-opacity: .75` (`scss/_variables.scss:1714`). 1 contexts, 1 properties.

<a id="c-btn-close-hover-opacity-1"></a>
#### $btn-close-hover-opacity – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-close-hover-opacity` | `0.75` | `.btn-close`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5340 | scss/_close.scss:11<br>via scss/_close.scss:6 | `$btn-close-hover-opacity` | Direct |

<a id="c-btn-close-opacity"></a>
### `$btn-close-opacity` (opacity)

Definition: `$btn-close-opacity: .5` (`scss/_variables.scss:1713`). 1 contexts, 1 properties.

<a id="c-btn-close-opacity-1"></a>
#### $btn-close-opacity – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-close-opacity` | `0.5` | `.btn-close`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5339 | scss/_close.scss:10<br>via scss/_close.scss:6 | `$btn-close-opacity` | Direct |

<a id="c-btn-disabled-opacity"></a>
### `$btn-disabled-opacity` (opacity)

Definition: `$btn-disabled-opacity: .65` (`scss/_variables.scss:840`). 1 contexts, 1 properties.

<a id="c-btn-disabled-opacity-1"></a>
#### $btn-disabled-opacity – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-btn-disabled-opacity` | `0.65` | `.btn`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2967 | scss/_buttons.scss:20<br>via scss/_buttons.scss:5 | `$btn-disabled-opacity` | Direct |

<a id="c-modal-backdrop-opacity"></a>
### `$modal-backdrop-opacity` (opacity)

Definition: `$modal-backdrop-opacity: .5` (`scss/_variables.scss:1520`). 1 contexts, 1 properties.

<a id="c-modal-backdrop-opacity-1"></a>
#### $modal-backdrop-opacity – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-backdrop-opacity` | `0.5` | `.modal-backdrop`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5545 | scss/_modal.scss:117<br>via scss/_modal.scss:113 | `$modal-backdrop-opacity` | Direct |

<a id="c-tooltip-opacity"></a>
### `$tooltip-opacity` (opacity)

Definition: `$tooltip-opacity: .9` (`scss/_variables.scss:1409`). 1 contexts, 1 properties.

<a id="c-tooltip-opacity-1"></a>
#### $tooltip-opacity – pattern 1: `$ANCHOR`

1 contexts, 1 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-tooltip-opacity` | `0.9` | `.tooltip`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5766 | scss/_tooltip.scss:13<br>via scss/_tooltip.scss:2 | `$tooltip-opacity` | Direct |

<a id="c-accordion-padding-x"></a>
### `$accordion-padding-x` (component)

Definition: `$accordion-padding-x: 1.25rem` (`scss/_variables.scss:1367`). 2 contexts, 2 properties.

<a id="c-accordion-padding-x-1"></a>
#### $accordion-padding-x – pattern 1: `$ANCHOR`

2 contexts, 2 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-accordion-body-padding-x` | `1.25rem` | `.accordion`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4564 | scss/_accordion.scss:24<br>via scss/_accordion.scss:5 | `$accordion-padding-x` | Direct |
| `--bs-accordion-btn-padding-x` | `1.25rem` | `.accordion`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4554 | scss/_accordion.scss:14<br>via scss/_accordion.scss:5 | `$accordion-padding-x` | Direct |

<a id="c-accordion-padding-y"></a>
### `$accordion-padding-y` (component)

Definition: `$accordion-padding-y: 1rem` (`scss/_variables.scss:1366`). 2 contexts, 2 properties.

<a id="c-accordion-padding-y-1"></a>
#### $accordion-padding-y – pattern 1: `$ANCHOR`

2 contexts, 2 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-accordion-body-padding-y` | `1rem` | `.accordion`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4565 | scss/_accordion.scss:25<br>via scss/_accordion.scss:5 | `$accordion-padding-y` | Direct |
| `--bs-accordion-btn-padding-y` | `1rem` | `.accordion`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4555 | scss/_accordion.scss:15<br>via scss/_accordion.scss:5 | `$accordion-padding-y` | Direct |

<a id="c-dropdown-padding-y"></a>
### `$dropdown-padding-y` (component)

Definition: `$dropdown-padding-y: .5rem` (`scss/_variables.scss:1240`). 2 contexts, 2 properties.

<a id="c-dropdown-padding-y-1"></a>
#### $dropdown-padding-y – pattern 1: `$ANCHOR`

2 contexts, 2 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-dropdown-header-padding-y` | `0.5rem` | `.dropdown-menu`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3431 | scss/_dropdown.scss:46<br>via scss/_dropdown.scss:19 | `$dropdown-padding-y` | Direct |
| `--bs-dropdown-padding-y` | `0.5rem` | `.dropdown-menu`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3409 | scss/_dropdown.scss:24<br>via scss/_dropdown.scss:19 | `$dropdown-padding-y` | Direct |

<a id="c-spinner-width"></a>
### `$spinner-width` (component)

Definition: `$spinner-width: 2rem` (`scss/_variables.scss:1691`). 4 contexts, 2 properties.

<a id="c-spinner-width-1"></a>
#### $spinner-width – pattern 1: `$ANCHOR`

4 contexts, 2 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-spinner-height` | `2rem` | `.spinner-border`<br>(none) | src/bootstrap/reference/css/bootstrap.css:6237 | scss/_spinners.scss:26<br>via scss/_spinners.scss:23 | `$spinner-width` | Direct |
| `--bs-spinner-height` | `2rem` | `.spinner-grow`<br>(none) | src/bootstrap/reference/css/bootstrap.css:6263 | scss/_spinners.scss:64<br>via scss/_spinners.scss:61 | `$spinner-width` | Direct |
| `--bs-spinner-width` | `2rem` | `.spinner-border`<br>(none) | src/bootstrap/reference/css/bootstrap.css:6236 | scss/_spinners.scss:25<br>via scss/_spinners.scss:23 | `$spinner-width` | Direct |
| `--bs-spinner-width` | `2rem` | `.spinner-grow`<br>(none) | src/bootstrap/reference/css/bootstrap.css:6262 | scss/_spinners.scss:63<br>via scss/_spinners.scss:61 | `$spinner-width` | Direct |

<a id="c-spinner-width-sm"></a>
### `$spinner-width-sm` (component)

Definition: `$spinner-width-sm: 1rem` (`scss/_variables.scss:1697`). 4 contexts, 2 properties.

<a id="c-spinner-width-sm-1"></a>
#### $spinner-width-sm – pattern 1: `$ANCHOR`

4 contexts, 2 properties. Relationship: Direct.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-spinner-height` | `1rem` | `.spinner-border-sm`<br>(none) | src/bootstrap/reference/css/bootstrap.css:6248 | scss/_spinners.scss:40<br>via scss/_spinners.scss:37 | `$spinner-width-sm` | Direct |
| `--bs-spinner-height` | `1rem` | `.spinner-grow-sm`<br>(none) | src/bootstrap/reference/css/bootstrap.css:6273 | scss/_spinners.scss:76<br>via scss/_spinners.scss:74 | `$spinner-width-sm` | Direct |
| `--bs-spinner-width` | `1rem` | `.spinner-border-sm`<br>(none) | src/bootstrap/reference/css/bootstrap.css:6247 | scss/_spinners.scss:39<br>via scss/_spinners.scss:37 | `$spinner-width-sm` | Direct |
| `--bs-spinner-width` | `1rem` | `.spinner-grow-sm`<br>(none) | src/bootstrap/reference/css/bootstrap.css:6272 | scss/_spinners.scss:75<br>via scss/_spinners.scss:74 | `$spinner-width-sm` | Direct |

## D. Independent values and other patterns

Contexts that have no shared design reference. They are grouped by how the value is defined.

<a id="d-1"></a>
### D1 Component-specific variable used by a single property

93 contexts, 70 properties.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-accordion-btn-icon-transform` | `rotate(-180deg)` | `.accordion`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4560 | scss/_accordion.scss:20<br>via scss/_accordion.scss:5 | `$accordion-icon-transform` | Direct |
| `--bs-accordion-btn-icon-transition` | `transform 0.2s ease-in-out` | `.accordion`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4561 | scss/_accordion.scss:21<br>via scss/_accordion.scss:5 | `$accordion-icon-transition` | Direct |
| `--bs-accordion-btn-icon-width` | `1.25rem` | `.accordion`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4559 | scss/_accordion.scss:19<br>via scss/_accordion.scss:5 | `$accordion-icon-width` | Direct |
| `--bs-accordion-transition` | `color 0.15s ease-in-out, background-color 0.15s ease-in-out, border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out, border-radius 0.15s ease` | `.accordion`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4549 | scss/_accordion.scss:9<br>via scss/_accordion.scss:5 | `($btn-transition, border-radius .15s ease)` | Direct |
| `--bs-alert-margin-bottom` | `1rem` | `.alert`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4840 | scss/_alert.scss:10<br>via scss/_alert.scss:5 | `$alert-margin-bottom` | Direct |
| `--bs-badge-font-size` | `0.75em` | `.badge`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4812 | scss/vendor/_rfs.scss:288<br>via scss/_badge.scss:6<br>+ scss/_badge.scss:None | `$badge-font-size` | Direct |
| `--bs-badge-padding-x` | `0.65em` | `.badge`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4810 | scss/_badge.scss:8<br>via scss/_badge.scss:6 | `$badge-padding-x` | Direct |
| `--bs-badge-padding-y` | `0.35em` | `.badge`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4811 | scss/_badge.scss:9<br>via scss/_badge.scss:6 | `$badge-padding-y` | Direct |
| `--bs-border-style` | `solid` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:104<br>src/bootstrap/reference/css/bootstrap.css:105 | scss/_root.scss:99<br>via scss/_root.scss:1 | `$border-style` | Direct |
| `--bs-breadcrumb-bg` | `` | `.breadcrumb`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4687 | scss/_breadcrumb.scss:7<br>via scss/_breadcrumb.scss:1 | `$breadcrumb-bg` | Direct |
| `--bs-breadcrumb-border-radius` | `` | `.breadcrumb`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4688 | scss/_breadcrumb.scss:8<br>via scss/_breadcrumb.scss:1 | `$breadcrumb-border-radius` | Direct |
| `--bs-breadcrumb-item-padding-x` | `0.5rem` | `.breadcrumb`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4690 | scss/_breadcrumb.scss:10<br>via scss/_breadcrumb.scss:1 | `$breadcrumb-item-padding-x` | Direct |
| `--bs-breadcrumb-margin-bottom` | `1rem` | `.breadcrumb`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4686 | scss/_breadcrumb.scss:5<br>via scss/_breadcrumb.scss:1 | `$breadcrumb-margin-bottom` | Direct |
| `--bs-breadcrumb-padding-x` | `0` | `.breadcrumb`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4684 | scss/_breadcrumb.scss:3<br>via scss/_breadcrumb.scss:1 | `$breadcrumb-padding-x` | Direct |
| `--bs-breadcrumb-padding-y` | `0` | `.breadcrumb`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4685 | scss/_breadcrumb.scss:4<br>via scss/_breadcrumb.scss:1 | `$breadcrumb-padding-y` | Direct |
| `--bs-btn-close-filter` | `invert(1) grayscale(100%) brightness(200%)` | `.btn-close-white`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5374 | scss/_close.scss:50<br>via scss/_close.scss:53 | `$btn-close-white-filter` | Direct |
| `--bs-btn-close-filter` | `` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5379 | scss/_close.scss:59<br>via scss/_close.scss:57 | `$btn-close-filter` | Direct |
| `--bs-btn-close-filter` | `invert(1) grayscale(100%) brightness(200%)` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5383 | scss/_close.scss:50<br>via scss/mixins/_color-mode.scss:16 | `$btn-close-white-filter` | Direct |
| `--bs-btn-font-family` | `` | `.btn`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2956 | scss/_buttons.scss:9<br>via scss/_buttons.scss:5 | `$input-btn-font-family` | Direct |
| `--bs-btn-padding-x` | `0.75rem` | `.btn`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2954 | scss/_buttons.scss:7<br>via scss/_buttons.scss:5 | `$input-btn-padding-x` | Direct |
| `--bs-btn-padding-x` | `1rem` | `.btn-lg, .btn-group-lg > .btn`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3330 | scss/mixins/_buttons.scss:66<br>via scss/_buttons.scss:210<br>+ scss/_buttons.scss:211 | `$input-btn-padding-x-lg` | Direct |
| `--bs-btn-padding-x` | `0.5rem` | `.btn-sm, .btn-group-sm > .btn`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3337 | scss/mixins/_buttons.scss:66<br>via scss/_buttons.scss:214<br>+ scss/_buttons.scss:215 | `$input-btn-padding-x-sm` | Direct |
| `--bs-btn-padding-y` | `0.375rem` | `.btn`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2955 | scss/_buttons.scss:8<br>via scss/_buttons.scss:5 | `$input-btn-padding-y` | Direct |
| `--bs-btn-padding-y` | `0.5rem` | `.btn-lg, .btn-group-lg > .btn`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3329 | scss/mixins/_buttons.scss:65<br>via scss/_buttons.scss:210<br>+ scss/_buttons.scss:211 | `$input-btn-padding-y-lg` | Direct |
| `--bs-btn-padding-y` | `0.25rem` | `.btn-sm, .btn-group-sm > .btn`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3336 | scss/mixins/_buttons.scss:65<br>via scss/_buttons.scss:214<br>+ scss/_buttons.scss:215 | `$input-btn-padding-y-sm` | Direct |
| `--bs-card-box-shadow` | `` | `.card`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4368 | scss/_card.scss:15<br>via scss/_card.scss:5 | `$card-box-shadow` | Direct |
| `--bs-card-cap-color` | `` | `.card`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4373 | scss/_card.scss:20<br>via scss/_card.scss:5 | `$card-cap-color` | Direct |
| `--bs-card-color` | `` | `.card`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4375 | scss/_card.scss:22<br>via scss/_card.scss:5 | `$card-color` | Direct |
| `--bs-card-height` | `` | `.card`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4374 | scss/_card.scss:21<br>via scss/_card.scss:5 | `$card-height` | Direct |
| `--bs-card-subtitle-color` | `` | `.card`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4364 | scss/_card.scss:11<br>via scss/_card.scss:5 | `$card-subtitle-color` | Direct |
| `--bs-card-title-color` | `` | `.card`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4363 | scss/_card.scss:10<br>via scss/_card.scss:5 | `$card-title-color` | Direct |
| `--bs-carousel-control-icon-filter` | `invert(1) grayscale(100)` | `.carousel-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:6203 | scss/_carousel.scss:208<br>via scss/_carousel.scss:211 | `$carousel-dark-control-icon-filter` | Direct |
| `--bs-carousel-control-icon-filter` | `` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap.css:6210 | scss/_carousel.scss:219<br>via scss/_carousel.scss:215 | `$carousel-control-icon-filter` | Direct |
| `--bs-carousel-control-icon-filter` | `invert(1) grayscale(100)` | `[data-bs-theme=dark]`<br>(none) | src/bootstrap/reference/css/bootstrap.css:6216 | scss/_carousel.scss:208<br>via scss/mixins/_color-mode.scss:16 | `$carousel-dark-control-icon-filter` | Direct |
| `--bs-dropdown-box-shadow` | `` | `.dropdown-menu-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3685 | scss/_dropdown.scss:240<br>via scss/_dropdown.scss:235 | `$dropdown-dark-box-shadow` | Direct |
| `--bs-dropdown-min-width` | `10rem` | `.dropdown-menu`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3407 | scss/_dropdown.scss:22<br>via scss/_dropdown.scss:19 | `$dropdown-min-width` | Direct |
| `--bs-dropdown-padding-x` | `0` | `.dropdown-menu`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3408 | scss/_dropdown.scss:23<br>via scss/_dropdown.scss:19 | `$dropdown-padding-x` | Direct |
| `--bs-dropdown-spacer` | `0.125rem` | `.dropdown-menu`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3410 | scss/_dropdown.scss:25<br>via scss/_dropdown.scss:19 | `$dropdown-spacer` | Direct |
| `--bs-link-decoration` | `underline` | `:root, [data-bs-theme=light]`<br>(none) | src/bootstrap/reference/css/bootstrap-reboot.css:97<br>src/bootstrap/reference/css/bootstrap.css:98 | scss/_root.scss:84<br>via scss/_root.scss:1 | `$link-decoration` | Direct |
| `--bs-modal-footer-bg` | `` | `.modal`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5474 | scss/_modal.scss:30<br>via scss/_modal.scss:10 | `$modal-footer-bg` | Direct |
| `--bs-modal-footer-gap` | `0.5rem` | `.modal`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5473 | scss/_modal.scss:29<br>via scss/_modal.scss:10 | `$modal-footer-margin-between` | Direct |
| `--bs-modal-margin` | `0.5rem` | `.modal`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5459 | scss/_modal.scss:15<br>via scss/_modal.scss:10 | `$modal-dialog-margin` | Direct |
| `--bs-modal-margin` | `1.75rem` | `.modal`<br>@media (min-width: 576px) | src/bootstrap/reference/css/bootstrap.css:5607 | scss/_modal.scss:182<br>via scss/_modal.scss:181 | `$modal-dialog-margin-y-sm-up` | Direct |
| `--bs-modal-width` | `500px` | `.modal`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5457 | scss/_modal.scss:13<br>via scss/_modal.scss:10 | `$modal-md` | Direct |
| `--bs-modal-width` | `800px` | `.modal-lg, .modal-xl, .modal-xxl`<br>@media (min-width: 992px) | src/bootstrap/reference/css/bootstrap.css:5623 | scss/_modal.scss:202<br>via scss/_modal.scss:199 | `$modal-lg` | Direct |
| `--bs-modal-width` | `300px` | `.modal-sm`<br>@media (min-width: 576px) | src/bootstrap/reference/css/bootstrap.css:5616 | scss/_modal.scss:194<br>via scss/_modal.scss:193 | `$modal-sm` | Direct |
| `--bs-modal-width` | `1140px` | `.modal-xl, .modal-xxl`<br>@media (min-width: 1200px) | src/bootstrap/reference/css/bootstrap.css:5629 | scss/_modal.scss:209<br>via scss/_modal.scss:207 | `$modal-xl` | Direct |
| `--bs-modal-width` | `1320px` | `.modal-xxl`<br>@media (min-width: 1400px) | src/bootstrap/reference/css/bootstrap.css:5634 | scss/_modal.scss:215<br>via scss/_modal.scss:214 | `$modal-xxl` | Direct |
| `--bs-nav-link-font-weight` | `` | `.nav`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3800 | scss/_nav.scss:11<br>via scss/_nav.scss:6 | `$nav-link-font-weight` | Direct |
| `--bs-nav-link-font-weight` | `` | `.navbar-nav`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3988 | scss/_navbar.scss:91<br>via scss/_navbar.scss:86 | `$nav-link-font-weight` | Direct |
| `--bs-nav-link-padding-x` | `1rem` | `.nav`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3798 | scss/_nav.scss:8<br>via scss/_nav.scss:6 | `$nav-link-padding-x` | Direct |
| `--bs-nav-link-padding-y` | `0.5rem` | `.nav`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3799 | scss/_nav.scss:9<br>via scss/_nav.scss:6 | `$nav-link-padding-y` | Direct |
| `--bs-nav-link-padding-y` | `0.5rem` | `.navbar-nav`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3987 | scss/_navbar.scss:89<br>via scss/_navbar.scss:86 | `$nav-link-padding-y` | Direct |
| `--bs-nav-underline-border-width` | `0.125rem` | `.nav-underline`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3888 | scss/_nav.scss:131<br>via scss/_nav.scss:128 | `$nav-underline-border-width` | Direct |
| `--bs-nav-underline-gap` | `1rem` | `.nav-underline`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3887 | scss/_nav.scss:130<br>via scss/_nav.scss:128 | `$nav-underline-gap` | Direct |
| `--bs-navbar-brand-margin-end` | `1rem` | `.navbar`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3940 | scss/_navbar.scss:15<br>via scss/_navbar.scss:6 | `$navbar-brand-margin-end` | Direct |
| `--bs-navbar-nav-link-padding-x` | `0.5rem` | `.navbar`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3944 | scss/_navbar.scss:19<br>via scss/_navbar.scss:6 | `$navbar-nav-link-padding-x` | Direct |
| `--bs-navbar-padding-x` | `0` | `.navbar`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3933 | scss/_navbar.scss:8<br>via scss/_navbar.scss:6 | `if($navbar-padding-x == null, 0, $navbar-padding-x)` | Derived, Compile-time |
| `--bs-navbar-toggler-padding-x` | `0.75rem` | `.navbar`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3946 | scss/_navbar.scss:21<br>via scss/_navbar.scss:6 | `$navbar-toggler-padding-x` | Direct |
| `--bs-navbar-toggler-padding-y` | `0.25rem` | `.navbar`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3945 | scss/_navbar.scss:20<br>via scss/_navbar.scss:6 | `$navbar-toggler-padding-y` | Direct |
| `--bs-navbar-toggler-transition` | `box-shadow 0.15s ease-in-out` | `.navbar`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3952 | scss/_navbar.scss:27<br>via scss/_navbar.scss:6 | `$navbar-toggler-transition` | Direct |
| `--bs-offcanvas-height` | `30vh` | `.offcanvas, .offcanvas-xxl, .offcanvas-xl, .offcanvas-lg, .offcanvas-md, .offcanvas-sm`<br>(none) | src/bootstrap/reference/css/bootstrap.css:6285 | scss/_offcanvas.scss:7<br>via scss/_offcanvas.scss:3 | `$offcanvas-vertical-height` | Direct |
| `--bs-offcanvas-transition` | `transform 0.3s ease-in-out` | `.offcanvas, .offcanvas-xxl, .offcanvas-xl, .offcanvas-lg, .offcanvas-md, .offcanvas-sm`<br>(none) | src/bootstrap/reference/css/bootstrap.css:6293 | scss/_offcanvas.scss:15<br>via scss/_offcanvas.scss:3 | `transform $offcanvas-transition-duration ease-in-out` | Independent |
| `--bs-offcanvas-width` | `400px` | `.offcanvas, .offcanvas-xxl, .offcanvas-xl, .offcanvas-lg, .offcanvas-md, .offcanvas-sm`<br>(none) | src/bootstrap/reference/css/bootstrap.css:6284 | scss/_offcanvas.scss:6<br>via scss/_offcanvas.scss:3 | `$offcanvas-horizontal-width` | Direct |
| `--bs-pagination-padding-x` | `0.75rem` | `.pagination`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4716 | scss/_pagination.scss:3<br>via scss/_pagination.scss:1 | `$pagination-padding-x` | Direct |
| `--bs-pagination-padding-x` | `1.5rem` | `.pagination-lg`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4796 | scss/mixins/_pagination.scss:5<br>via scss/_pagination.scss:103<br>+ scss/_pagination.scss:104 | `$pagination-padding-x-lg` | Direct |
| `--bs-pagination-padding-x` | `0.5rem` | `.pagination-sm`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4803 | scss/mixins/_pagination.scss:5<br>via scss/_pagination.scss:107<br>+ scss/_pagination.scss:108 | `$pagination-padding-x-sm` | Direct |
| `--bs-pagination-padding-y` | `0.375rem` | `.pagination`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4717 | scss/_pagination.scss:4<br>via scss/_pagination.scss:1 | `$pagination-padding-y` | Direct |
| `--bs-pagination-padding-y` | `0.75rem` | `.pagination-lg`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4797 | scss/mixins/_pagination.scss:6<br>via scss/_pagination.scss:103<br>+ scss/_pagination.scss:104 | `$pagination-padding-y-lg` | Direct |
| `--bs-pagination-padding-y` | `0.25rem` | `.pagination-sm`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4804 | scss/mixins/_pagination.scss:6<br>via scss/_pagination.scss:107<br>+ scss/_pagination.scss:108 | `$pagination-padding-y-sm` | Direct |
| `--bs-popover-arrow-height` | `0.5rem` | `.popover`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5877 | scss/_popover.scss:21<br>via scss/_popover.scss:1 | `$popover-arrow-height` | Direct |
| `--bs-popover-arrow-width` | `1rem` | `.popover`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5876 | scss/_popover.scss:20<br>via scss/_popover.scss:1 | `$popover-arrow-width` | Direct |
| `--bs-popover-header-padding-y` | `0.5rem` | `.popover`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5869 | scss/_popover.scss:13<br>via scss/_popover.scss:1 | `$popover-header-padding-y` | Direct |
| `--bs-popover-max-width` | `276px` | `.popover`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5860 | scss/_popover.scss:4<br>via scss/_popover.scss:1 | `$popover-max-width` | Direct |
| `--bs-progress-bar-transition` | `width 0.6s ease` | `.progress, .progress-stacked`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4945 | scss/_progress.scss:21<br>via scss/_progress.scss:11 | `$progress-bar-transition` | Direct |
| `--bs-progress-height` | `1rem` | `.progress, .progress-stacked`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4938 | scss/_progress.scss:14<br>via scss/_progress.scss:11 | `$progress-height` | Direct |
| `--bs-spinner-animation-speed` | `0.75s` | `.spinner-border`<br>(none) | src/bootstrap/reference/css/bootstrap.css:6240 | scss/_spinners.scss:29<br>via scss/_spinners.scss:23 | `$spinner-animation-speed` | Direct |
| `--bs-spinner-animation-speed` | `1.5s` | `.spinner-border, .spinner-grow`<br>@media (prefers-reduced-motion: reduce) | src/bootstrap/reference/css/bootstrap.css:6279 | scss/_spinners.scss:83<br>via scss/_spinners.scss:81 | `$spinner-animation-speed * 2` | Derived, Compile-time |
| `--bs-spinner-animation-speed` | `0.75s` | `.spinner-grow`<br>(none) | src/bootstrap/reference/css/bootstrap.css:6265 | scss/_spinners.scss:66<br>via scss/_spinners.scss:61 | `$spinner-animation-speed` | Direct |
| `--bs-spinner-border-width` | `0.25em` | `.spinner-border`<br>(none) | src/bootstrap/reference/css/bootstrap.css:6239 | scss/_spinners.scss:28<br>via scss/_spinners.scss:23 | `$spinner-border-width` | Direct |
| `--bs-spinner-border-width` | `0.2em` | `.spinner-border-sm`<br>(none) | src/bootstrap/reference/css/bootstrap.css:6249 | scss/_spinners.scss:41<br>via scss/_spinners.scss:37 | `$spinner-border-width-sm` | Direct |
| `--bs-spinner-vertical-align` | `-0.125em` | `.spinner-border`<br>(none) | src/bootstrap/reference/css/bootstrap.css:6238 | scss/_spinners.scss:27<br>via scss/_spinners.scss:23 | `$spinner-vertical-align` | Direct |
| `--bs-spinner-vertical-align` | `-0.125em` | `.spinner-grow`<br>(none) | src/bootstrap/reference/css/bootstrap.css:6264 | scss/_spinners.scss:65<br>via scss/_spinners.scss:61 | `$spinner-vertical-align` | Direct |
| `--bs-table-accent-bg` | `transparent` | `.table`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1870 | scss/_tables.scss:15<br>via scss/_tables.scss:5 | `$table-accent-bg` | Direct |
| `--bs-toast-color` | `` | `.toast`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5393 | scss/_toasts.scss:9<br>via scss/_toasts.scss:1 | `$toast-color` | Direct |
| `--bs-toast-font-size` | `0.875rem` | `.toast`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5392 | scss/vendor/_rfs.scss:288<br>via scss/_toasts.scss:1<br>+ scss/_toasts.scss:None | `$toast-font-size` | Direct |
| `--bs-toast-max-width` | `350px` | `.toast`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5391 | scss/_toasts.scss:7<br>via scss/_toasts.scss:1 | `$toast-max-width` | Direct |
| `--bs-toast-padding-x` | `0.75rem` | `.toast`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5388 | scss/_toasts.scss:4<br>via scss/_toasts.scss:1 | `$toast-padding-x` | Direct |
| `--bs-toast-padding-y` | `0.5rem` | `.toast`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5389 | scss/_toasts.scss:5<br>via scss/_toasts.scss:1 | `$toast-padding-y` | Direct |
| `--bs-tooltip-arrow-height` | `0.4rem` | `.tooltip`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5768 | scss/_tooltip.scss:15<br>via scss/_tooltip.scss:2 | `$tooltip-arrow-height` | Direct |
| `--bs-tooltip-arrow-width` | `0.8rem` | `.tooltip`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5767 | scss/_tooltip.scss:14<br>via scss/_tooltip.scss:2 | `$tooltip-arrow-width` | Direct |
| `--bs-tooltip-margin` | `` | `.tooltip`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5761 | scss/_tooltip.scss:8<br>via scss/_tooltip.scss:2 | `$tooltip-margin` | Direct |
| `--bs-tooltip-max-width` | `200px` | `.tooltip`<br>(none) | src/bootstrap/reference/css/bootstrap.css:5758 | scss/_tooltip.scss:5<br>via scss/_tooltip.scss:2 | `$tooltip-max-width` | Direct |

<a id="d-2"></a>
### D2 Component-local runtime alias (target set by another rule; not traceable to a Sass reference)

11 contexts, 7 properties.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-nav-link-color` | `var(--bs-navbar-color)` | `.navbar-nav`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3989 | scss/_navbar.scss:92<br>via scss/_navbar.scss:86 | `var(--bs-navbar-color)` | Runtime |
| `--bs-nav-link-disabled-color` | `var(--bs-navbar-disabled-color)` | `.navbar-nav`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3991 | scss/_navbar.scss:94<br>via scss/_navbar.scss:86 | `var(--bs-navbar-disabled-color)` | Runtime |
| `--bs-nav-link-hover-color` | `var(--bs-navbar-hover-color)` | `.navbar-nav`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3990 | scss/_navbar.scss:93<br>via scss/_navbar.scss:86 | `var(--bs-navbar-hover-color)` | Runtime |
| `--bs-table-bg-state` | `var(--bs-table-active-bg)` | `.table-active`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1934 | scss/_tables.scss:133<br>via scss/_tables.scss:131 | `var(--bs-table-active-bg)` | Runtime |
| `--bs-table-bg-state` | `var(--bs-table-hover-bg)` | `.table-hover > tbody > tr:hover > *`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1939 | scss/_tables.scss:143<br>via scss/_tables.scss:141 | `var(--bs-table-hover-bg)` | Runtime |
| `--bs-table-bg-type` | `var(--bs-table-striped-bg)` | `.table-striped > tbody > tr:nth-of-type(odd) > *`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1924 | scss/_tables.scss:115<br>via scss/_tables.scss:113 | `var(--bs-table-striped-bg)` | Runtime |
| `--bs-table-bg-type` | `var(--bs-table-striped-bg)` | `.table-striped-columns > :not(caption) > tr > :nth-child(even)`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1929 | scss/_tables.scss:123<br>via scss/_tables.scss:121 | `var(--bs-table-striped-bg)` | Runtime |
| `--bs-table-color-state` | `var(--bs-table-active-color)` | `.table-active`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1933 | scss/_tables.scss:132<br>via scss/_tables.scss:131 | `var(--bs-table-active-color)` | Runtime |
| `--bs-table-color-state` | `var(--bs-table-hover-color)` | `.table-hover > tbody > tr:hover > *`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1938 | scss/_tables.scss:142<br>via scss/_tables.scss:141 | `var(--bs-table-hover-color)` | Runtime |
| `--bs-table-color-type` | `var(--bs-table-striped-color)` | `.table-striped > tbody > tr:nth-of-type(odd) > *`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1923 | scss/_tables.scss:114<br>via scss/_tables.scss:113 | `var(--bs-table-striped-color)` | Runtime |
| `--bs-table-color-type` | `var(--bs-table-striped-color)` | `.table-striped-columns > :not(caption) > tr > :nth-child(even)`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1928 | scss/_tables.scss:122<br>via scss/_tables.scss:121 | `var(--bs-table-striped-color)` | Runtime |

<a id="d-3"></a>
### D3 Hardcoded literal in a Sass map or utility configuration

95 contexts, 8 properties.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-bg-opacity` | `1` | `.bg-black`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2196<br>src/bootstrap/reference/css/bootstrap.css:8855 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:666 | `1` | Independent |
| `--bs-bg-opacity` | `1` | `.bg-body`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2206<br>src/bootstrap/reference/css/bootstrap.css:8865 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:666 | `1` | Independent |
| `--bs-bg-opacity` | `1` | `.bg-body-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2216<br>src/bootstrap/reference/css/bootstrap.css:8875 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:666 | `1` | Independent |
| `--bs-bg-opacity` | `1` | `.bg-body-tertiary`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2221<br>src/bootstrap/reference/css/bootstrap.css:8880 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:666 | `1` | Independent |
| `--bs-bg-opacity` | `1` | `.bg-danger`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2181<br>src/bootstrap/reference/css/bootstrap.css:8840 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:666 | `1` | Independent |
| `--bs-bg-opacity` | `1` | `.bg-dark`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2191<br>src/bootstrap/reference/css/bootstrap.css:8850 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:666 | `1` | Independent |
| `--bs-bg-opacity` | `1` | `.bg-info`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2171<br>src/bootstrap/reference/css/bootstrap.css:8830 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:666 | `1` | Independent |
| `--bs-bg-opacity` | `1` | `.bg-light`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2186<br>src/bootstrap/reference/css/bootstrap.css:8845 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:666 | `1` | Independent |
| `--bs-bg-opacity` | `0.1` | `.bg-opacity-10`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2226<br>src/bootstrap/reference/css/bootstrap.css:8885 | scss/mixins/_utilities.scss:58<br>via scss/mixins/_utilities.scss:57<br>+ scss/_utilities.scss:677 | `0.1` | Independent |
| `--bs-bg-opacity` | `1` | `.bg-opacity-100`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2242<br>src/bootstrap/reference/css/bootstrap.css:8901 | scss/mixins/_utilities.scss:58<br>via scss/mixins/_utilities.scss:57<br>+ scss/_utilities.scss:677 | `1` | Independent |
| `--bs-bg-opacity` | `0.25` | `.bg-opacity-25`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2230<br>src/bootstrap/reference/css/bootstrap.css:8889 | scss/mixins/_utilities.scss:58<br>via scss/mixins/_utilities.scss:57<br>+ scss/_utilities.scss:677 | `0.25` | Independent |
| `--bs-bg-opacity` | `0.5` | `.bg-opacity-50`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2234<br>src/bootstrap/reference/css/bootstrap.css:8893 | scss/mixins/_utilities.scss:58<br>via scss/mixins/_utilities.scss:57<br>+ scss/_utilities.scss:677 | `0.5` | Independent |
| `--bs-bg-opacity` | `0.75` | `.bg-opacity-75`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2238<br>src/bootstrap/reference/css/bootstrap.css:8897 | scss/mixins/_utilities.scss:58<br>via scss/mixins/_utilities.scss:57<br>+ scss/_utilities.scss:677 | `0.75` | Independent |
| `--bs-bg-opacity` | `1` | `.bg-primary`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2156<br>src/bootstrap/reference/css/bootstrap.css:8815 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:666 | `1` | Independent |
| `--bs-bg-opacity` | `1` | `.bg-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2161<br>src/bootstrap/reference/css/bootstrap.css:8820 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:666 | `1` | Independent |
| `--bs-bg-opacity` | `1` | `.bg-success`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2166<br>src/bootstrap/reference/css/bootstrap.css:8825 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:666 | `1` | Independent |
| `--bs-bg-opacity` | `1` | `.bg-transparent`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2211<br>src/bootstrap/reference/css/bootstrap.css:8870 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:666 | `1` | Independent |
| `--bs-bg-opacity` | `1` | `.bg-warning`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2176<br>src/bootstrap/reference/css/bootstrap.css:8835 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:666 | `1` | Independent |
| `--bs-bg-opacity` | `1` | `.bg-white`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2201<br>src/bootstrap/reference/css/bootstrap.css:8860 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:666 | `1` | Independent |
| `--bs-border-opacity` | `1` | `.border-black`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:935<br>src/bootstrap/reference/css/bootstrap.css:7594 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:170 | `1` | Independent |
| `--bs-border-opacity` | `1` | `.border-danger`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:920<br>src/bootstrap/reference/css/bootstrap.css:7579 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:170 | `1` | Independent |
| `--bs-border-opacity` | `1` | `.border-dark`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:930<br>src/bootstrap/reference/css/bootstrap.css:7589 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:170 | `1` | Independent |
| `--bs-border-opacity` | `1` | `.border-info`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:910<br>src/bootstrap/reference/css/bootstrap.css:7569 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:170 | `1` | Independent |
| `--bs-border-opacity` | `1` | `.border-light`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:925<br>src/bootstrap/reference/css/bootstrap.css:7584 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:170 | `1` | Independent |
| `--bs-border-opacity` | `0.1` | `.border-opacity-10`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:997<br>src/bootstrap/reference/css/bootstrap.css:7656 | scss/mixins/_utilities.scss:58<br>via scss/mixins/_utilities.scss:57<br>+ scss/_utilities.scss:184 | `0.1` | Independent |
| `--bs-border-opacity` | `1` | `.border-opacity-100`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:1013<br>src/bootstrap/reference/css/bootstrap.css:7672 | scss/mixins/_utilities.scss:58<br>via scss/mixins/_utilities.scss:57<br>+ scss/_utilities.scss:184 | `1` | Independent |
| `--bs-border-opacity` | `0.25` | `.border-opacity-25`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:1001<br>src/bootstrap/reference/css/bootstrap.css:7660 | scss/mixins/_utilities.scss:58<br>via scss/mixins/_utilities.scss:57<br>+ scss/_utilities.scss:184 | `0.25` | Independent |
| `--bs-border-opacity` | `0.5` | `.border-opacity-50`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:1005<br>src/bootstrap/reference/css/bootstrap.css:7664 | scss/mixins/_utilities.scss:58<br>via scss/mixins/_utilities.scss:57<br>+ scss/_utilities.scss:184 | `0.5` | Independent |
| `--bs-border-opacity` | `0.75` | `.border-opacity-75`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:1009<br>src/bootstrap/reference/css/bootstrap.css:7668 | scss/mixins/_utilities.scss:58<br>via scss/mixins/_utilities.scss:57<br>+ scss/_utilities.scss:184 | `0.75` | Independent |
| `--bs-border-opacity` | `1` | `.border-primary`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:895<br>src/bootstrap/reference/css/bootstrap.css:7554 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:170 | `1` | Independent |
| `--bs-border-opacity` | `1` | `.border-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:900<br>src/bootstrap/reference/css/bootstrap.css:7559 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:170 | `1` | Independent |
| `--bs-border-opacity` | `1` | `.border-success`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:905<br>src/bootstrap/reference/css/bootstrap.css:7564 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:170 | `1` | Independent |
| `--bs-border-opacity` | `1` | `.border-warning`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:915<br>src/bootstrap/reference/css/bootstrap.css:7574 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:170 | `1` | Independent |
| `--bs-border-opacity` | `1` | `.border-white`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:940<br>src/bootstrap/reference/css/bootstrap.css:7599 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:170 | `1` | Independent |
| `--bs-gutter-y` | `0` | `.container, .container-fluid, .container-xxl, .container-xl, .container-lg, .container-md, .container-sm`<br>(none) | src/bootstrap/reference/css/bootstrap-grid.css:14<br>src/bootstrap/reference/css/bootstrap.css:749 | scss/mixins/_container.scss:5<br>via scss/_containers.scss:7<br>+ scss/_containers.scss:10 | `0` | Independent |
| `--bs-gutter-y` | `0` | `.row`<br>(none) | src/bootstrap/reference/css/bootstrap-grid.css:58<br>src/bootstrap/reference/css/bootstrap.css:793 | scss/mixins/_grid.scss:7<br>via scss/_grid.scss:12 | `0` | Independent |
| `--bs-link-opacity` | `0.1` | `.link-opacity-10`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:1990<br>src/bootstrap/reference/css/bootstrap.css:8649 | scss/mixins/_utilities.scss:58<br>via scss/mixins/_utilities.scss:57<br>+ scss/_utilities.scss:612 | `0.1` | Independent |
| `--bs-link-opacity` | `0.1` | `.link-opacity-10-hover:hover`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:1994<br>src/bootstrap/reference/css/bootstrap.css:8653 | scss/mixins/_utilities.scss:63<br>via scss/mixins/_utilities.scss:62<br>+ scss/_utilities.scss:612 | `0.1` | Independent |
| `--bs-link-opacity` | `1` | `.link-opacity-100`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2022<br>src/bootstrap/reference/css/bootstrap.css:8681 | scss/mixins/_utilities.scss:58<br>via scss/mixins/_utilities.scss:57<br>+ scss/_utilities.scss:612 | `1` | Independent |
| `--bs-link-opacity` | `1` | `.link-opacity-100-hover:hover`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2026<br>src/bootstrap/reference/css/bootstrap.css:8685 | scss/mixins/_utilities.scss:63<br>via scss/mixins/_utilities.scss:62<br>+ scss/_utilities.scss:612 | `1` | Independent |
| `--bs-link-opacity` | `0.25` | `.link-opacity-25`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:1998<br>src/bootstrap/reference/css/bootstrap.css:8657 | scss/mixins/_utilities.scss:58<br>via scss/mixins/_utilities.scss:57<br>+ scss/_utilities.scss:612 | `0.25` | Independent |
| `--bs-link-opacity` | `0.25` | `.link-opacity-25-hover:hover`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2002<br>src/bootstrap/reference/css/bootstrap.css:8661 | scss/mixins/_utilities.scss:63<br>via scss/mixins/_utilities.scss:62<br>+ scss/_utilities.scss:612 | `0.25` | Independent |
| `--bs-link-opacity` | `0.5` | `.link-opacity-50`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2006<br>src/bootstrap/reference/css/bootstrap.css:8665 | scss/mixins/_utilities.scss:58<br>via scss/mixins/_utilities.scss:57<br>+ scss/_utilities.scss:612 | `0.5` | Independent |
| `--bs-link-opacity` | `0.5` | `.link-opacity-50-hover:hover`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2010<br>src/bootstrap/reference/css/bootstrap.css:8669 | scss/mixins/_utilities.scss:63<br>via scss/mixins/_utilities.scss:62<br>+ scss/_utilities.scss:612 | `0.5` | Independent |
| `--bs-link-opacity` | `0.75` | `.link-opacity-75`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2014<br>src/bootstrap/reference/css/bootstrap.css:8673 | scss/mixins/_utilities.scss:58<br>via scss/mixins/_utilities.scss:57<br>+ scss/_utilities.scss:612 | `0.75` | Independent |
| `--bs-link-opacity` | `0.75` | `.link-opacity-75-hover:hover`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2018<br>src/bootstrap/reference/css/bootstrap.css:8677 | scss/mixins/_utilities.scss:63<br>via scss/mixins/_utilities.scss:62<br>+ scss/_utilities.scss:612 | `0.75` | Independent |
| `--bs-link-underline-opacity` | `1` | `.link-underline`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2102<br>src/bootstrap/reference/css/bootstrap.css:8761 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:638 | `1` | Independent |
| `--bs-link-underline-opacity` | `1` | `.link-underline-danger`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2084<br>src/bootstrap/reference/css/bootstrap.css:8743 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:638 | `1` | Independent |
| `--bs-link-underline-opacity` | `1` | `.link-underline-dark`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2096<br>src/bootstrap/reference/css/bootstrap.css:8755 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:638 | `1` | Independent |
| `--bs-link-underline-opacity` | `1` | `.link-underline-info`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2072<br>src/bootstrap/reference/css/bootstrap.css:8731 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:638 | `1` | Independent |
| `--bs-link-underline-opacity` | `1` | `.link-underline-light`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2090<br>src/bootstrap/reference/css/bootstrap.css:8749 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:638 | `1` | Independent |
| `--bs-link-underline-opacity` | `0` | `.link-underline-opacity-0`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2108<br>src/bootstrap/reference/css/bootstrap.css:8767 | scss/mixins/_utilities.scss:58<br>via scss/mixins/_utilities.scss:57<br>+ scss/_utilities.scss:647 | `0` | Independent |
| `--bs-link-underline-opacity` | `0` | `.link-underline-opacity-0-hover:hover`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2112<br>src/bootstrap/reference/css/bootstrap.css:8771 | scss/mixins/_utilities.scss:63<br>via scss/mixins/_utilities.scss:62<br>+ scss/_utilities.scss:647 | `0` | Independent |
| `--bs-link-underline-opacity` | `0.1` | `.link-underline-opacity-10`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2116<br>src/bootstrap/reference/css/bootstrap.css:8775 | scss/mixins/_utilities.scss:58<br>via scss/mixins/_utilities.scss:57<br>+ scss/_utilities.scss:647 | `0.1` | Independent |
| `--bs-link-underline-opacity` | `0.1` | `.link-underline-opacity-10-hover:hover`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2120<br>src/bootstrap/reference/css/bootstrap.css:8779 | scss/mixins/_utilities.scss:63<br>via scss/mixins/_utilities.scss:62<br>+ scss/_utilities.scss:647 | `0.1` | Independent |
| `--bs-link-underline-opacity` | `1` | `.link-underline-opacity-100`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2148<br>src/bootstrap/reference/css/bootstrap.css:8807 | scss/mixins/_utilities.scss:58<br>via scss/mixins/_utilities.scss:57<br>+ scss/_utilities.scss:647 | `1` | Independent |
| `--bs-link-underline-opacity` | `1` | `.link-underline-opacity-100-hover:hover`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2152<br>src/bootstrap/reference/css/bootstrap.css:8811 | scss/mixins/_utilities.scss:63<br>via scss/mixins/_utilities.scss:62<br>+ scss/_utilities.scss:647 | `1` | Independent |
| `--bs-link-underline-opacity` | `0.25` | `.link-underline-opacity-25`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2124<br>src/bootstrap/reference/css/bootstrap.css:8783 | scss/mixins/_utilities.scss:58<br>via scss/mixins/_utilities.scss:57<br>+ scss/_utilities.scss:647 | `0.25` | Independent |
| `--bs-link-underline-opacity` | `0.25` | `.link-underline-opacity-25-hover:hover`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2128<br>src/bootstrap/reference/css/bootstrap.css:8787 | scss/mixins/_utilities.scss:63<br>via scss/mixins/_utilities.scss:62<br>+ scss/_utilities.scss:647 | `0.25` | Independent |
| `--bs-link-underline-opacity` | `0.5` | `.link-underline-opacity-50`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2132<br>src/bootstrap/reference/css/bootstrap.css:8791 | scss/mixins/_utilities.scss:58<br>via scss/mixins/_utilities.scss:57<br>+ scss/_utilities.scss:647 | `0.5` | Independent |
| `--bs-link-underline-opacity` | `0.5` | `.link-underline-opacity-50-hover:hover`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2136<br>src/bootstrap/reference/css/bootstrap.css:8795 | scss/mixins/_utilities.scss:63<br>via scss/mixins/_utilities.scss:62<br>+ scss/_utilities.scss:647 | `0.5` | Independent |
| `--bs-link-underline-opacity` | `0.75` | `.link-underline-opacity-75`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2140<br>src/bootstrap/reference/css/bootstrap.css:8799 | scss/mixins/_utilities.scss:58<br>via scss/mixins/_utilities.scss:57<br>+ scss/_utilities.scss:647 | `0.75` | Independent |
| `--bs-link-underline-opacity` | `0.75` | `.link-underline-opacity-75-hover:hover`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2144<br>src/bootstrap/reference/css/bootstrap.css:8803 | scss/mixins/_utilities.scss:63<br>via scss/mixins/_utilities.scss:62<br>+ scss/_utilities.scss:647 | `0.75` | Independent |
| `--bs-link-underline-opacity` | `1` | `.link-underline-primary`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2054<br>src/bootstrap/reference/css/bootstrap.css:8713 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:638 | `1` | Independent |
| `--bs-link-underline-opacity` | `1` | `.link-underline-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2060<br>src/bootstrap/reference/css/bootstrap.css:8719 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:638 | `1` | Independent |
| `--bs-link-underline-opacity` | `1` | `.link-underline-success`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2066<br>src/bootstrap/reference/css/bootstrap.css:8725 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:638 | `1` | Independent |
| `--bs-link-underline-opacity` | `1` | `.link-underline-warning`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:2078<br>src/bootstrap/reference/css/bootstrap.css:8737 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:638 | `1` | Independent |
| `--bs-nav-link-padding-x` | `0` | `.navbar-nav`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3986 | scss/_navbar.scss:88<br>via scss/_navbar.scss:86 | `0` | Independent |
| `--bs-offcanvas-border-width` | `0` | `.offcanvas-lg`<br>@media (min-width: 992px) | src/bootstrap/reference/css/bootstrap.css:6516 | scss/_offcanvas.scss:99<br>via scss/_offcanvas.scss:33 | `0` | Independent |
| `--bs-offcanvas-border-width` | `0` | `.offcanvas-md`<br>@media (min-width: 768px) | src/bootstrap/reference/css/bootstrap.css:6438 | scss/_offcanvas.scss:99<br>via scss/_offcanvas.scss:33 | `0` | Independent |
| `--bs-offcanvas-border-width` | `0` | `.offcanvas-sm`<br>@media (min-width: 576px) | src/bootstrap/reference/css/bootstrap.css:6360 | scss/_offcanvas.scss:99<br>via scss/_offcanvas.scss:33 | `0` | Independent |
| `--bs-offcanvas-border-width` | `0` | `.offcanvas-xl`<br>@media (min-width: 1200px) | src/bootstrap/reference/css/bootstrap.css:6594 | scss/_offcanvas.scss:99<br>via scss/_offcanvas.scss:33 | `0` | Independent |
| `--bs-offcanvas-border-width` | `0` | `.offcanvas-xxl`<br>@media (min-width: 1400px) | src/bootstrap/reference/css/bootstrap.css:6672 | scss/_offcanvas.scss:99<br>via scss/_offcanvas.scss:33 | `0` | Independent |
| `--bs-text-opacity` | `1` | `.text-black`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:1892<br>src/bootstrap/reference/css/bootstrap.css:8551 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:580 | `1` | Independent |
| `--bs-text-opacity` | `1` | `.text-black-50`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:1912<br>src/bootstrap/reference/css/bootstrap.css:8571 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:580 | `1` | Independent |
| `--bs-text-opacity` | `1` | `.text-body`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:1902<br>src/bootstrap/reference/css/bootstrap.css:8561 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:580 | `1` | Independent |
| `--bs-text-opacity` | `1` | `.text-body-emphasis`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:1932<br>src/bootstrap/reference/css/bootstrap.css:8591 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:580 | `1` | Independent |
| `--bs-text-opacity` | `1` | `.text-body-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:1922<br>src/bootstrap/reference/css/bootstrap.css:8581 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:580 | `1` | Independent |
| `--bs-text-opacity` | `1` | `.text-body-tertiary`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:1927<br>src/bootstrap/reference/css/bootstrap.css:8586 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:580 | `1` | Independent |
| `--bs-text-opacity` | `1` | `.text-danger`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:1877<br>src/bootstrap/reference/css/bootstrap.css:8536 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:580 | `1` | Independent |
| `--bs-text-opacity` | `1` | `.text-dark`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:1887<br>src/bootstrap/reference/css/bootstrap.css:8546 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:580 | `1` | Independent |
| `--bs-text-opacity` | `1` | `.text-info`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:1867<br>src/bootstrap/reference/css/bootstrap.css:8526 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:580 | `1` | Independent |
| `--bs-text-opacity` | `1` | `.text-light`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:1882<br>src/bootstrap/reference/css/bootstrap.css:8541 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:580 | `1` | Independent |
| `--bs-text-opacity` | `1` | `.text-muted`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:1907<br>src/bootstrap/reference/css/bootstrap.css:8566 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:580 | `1` | Independent |
| `--bs-text-opacity` | `1` | `.text-opacity-100`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:1954<br>src/bootstrap/reference/css/bootstrap.css:8613 | scss/mixins/_utilities.scss:58<br>via scss/mixins/_utilities.scss:57<br>+ scss/_utilities.scss:595 | `1` | Independent |
| `--bs-text-opacity` | `0.25` | `.text-opacity-25`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:1942<br>src/bootstrap/reference/css/bootstrap.css:8601 | scss/mixins/_utilities.scss:58<br>via scss/mixins/_utilities.scss:57<br>+ scss/_utilities.scss:595 | `0.25` | Independent |
| `--bs-text-opacity` | `0.5` | `.text-opacity-50`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:1946<br>src/bootstrap/reference/css/bootstrap.css:8605 | scss/mixins/_utilities.scss:58<br>via scss/mixins/_utilities.scss:57<br>+ scss/_utilities.scss:595 | `0.5` | Independent |
| `--bs-text-opacity` | `0.75` | `.text-opacity-75`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:1950<br>src/bootstrap/reference/css/bootstrap.css:8609 | scss/mixins/_utilities.scss:58<br>via scss/mixins/_utilities.scss:57<br>+ scss/_utilities.scss:595 | `0.75` | Independent |
| `--bs-text-opacity` | `1` | `.text-primary`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:1852<br>src/bootstrap/reference/css/bootstrap.css:8511 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:580 | `1` | Independent |
| `--bs-text-opacity` | `1` | `.text-reset`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:1937<br>src/bootstrap/reference/css/bootstrap.css:8596 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:580 | `1` | Independent |
| `--bs-text-opacity` | `1` | `.text-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:1857<br>src/bootstrap/reference/css/bootstrap.css:8516 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:580 | `1` | Independent |
| `--bs-text-opacity` | `1` | `.text-success`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:1862<br>src/bootstrap/reference/css/bootstrap.css:8521 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:580 | `1` | Independent |
| `--bs-text-opacity` | `1` | `.text-warning`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:1872<br>src/bootstrap/reference/css/bootstrap.css:8531 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:580 | `1` | Independent |
| `--bs-text-opacity` | `1` | `.text-white`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:1897<br>src/bootstrap/reference/css/bootstrap.css:8556 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:580 | `1` | Independent |
| `--bs-text-opacity` | `1` | `.text-white-50`<br>(none) | src/bootstrap/reference/css/bootstrap-utilities.css:1917<br>src/bootstrap/reference/css/bootstrap.css:8576 | scss/mixins/_utilities.scss:71<br>via scss/mixins/_utilities.scss:67<br>+ scss/_utilities.scss:580 | `1` | Independent |

<a id="d-4"></a>
### D4 Hardcoded literal value

52 contexts, 19 properties.

| Property | Compiled value | Selector / context | CSS source | Sass source | Sass expression | Relationship |
|---|---|---|---|---|---|---|
| `--bs-alert-bg` | `transparent` | `.alert`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4837 | scss/_alert.scss:7<br>via scss/_alert.scss:5 | `transparent` | Independent |
| `--bs-alert-border-color` | `transparent` | `.alert`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4842 | scss/_alert.scss:12<br>via scss/_alert.scss:5 | `transparent` | Independent |
| `--bs-alert-color` | `inherit` | `.alert`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4841 | scss/_alert.scss:11<br>via scss/_alert.scss:5 | `inherit` | Independent |
| `--bs-alert-link-color` | `inherit` | `.alert`<br>(none) | src/bootstrap/reference/css/bootstrap.css:4845 | scss/_alert.scss:15<br>via scss/_alert.scss:5 | `inherit` | Independent |
| `--bs-btn-active-border-color` | `transparent` | `.btn-link`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3314 | scss/_buttons.scss:178<br>via scss/_buttons.scss:170 | `transparent` | Independent |
| `--bs-btn-bg` | `transparent` | `.btn`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2961 | scss/_buttons.scss:14<br>via scss/_buttons.scss:5 | `transparent` | Independent |
| `--bs-btn-bg` | `transparent` | `.btn-link`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3309 | scss/_buttons.scss:173<br>via scss/_buttons.scss:170 | `transparent` | Independent |
| `--bs-btn-border-color` | `transparent` | `.btn`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2963 | scss/_buttons.scss:16<br>via scss/_buttons.scss:5 | `transparent` | Independent |
| `--bs-btn-border-color` | `transparent` | `.btn-link`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3310 | scss/_buttons.scss:174<br>via scss/_buttons.scss:170 | `transparent` | Independent |
| `--bs-btn-box-shadow` | `0 0 0 #000` | `.btn-link`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3317 | scss/_buttons.scss:181<br>via scss/_buttons.scss:170 | `0 0 0 #000` | Independent |
| `--bs-btn-disabled-bg` | `transparent` | `.btn-outline-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3267 | scss/mixins/_buttons.scss:57<br>via scss/_buttons.scss:158 | `transparent` | Independent |
| `--bs-btn-disabled-bg` | `transparent` | `.btn-outline-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3301 | scss/mixins/_buttons.scss:57<br>via scss/_buttons.scss:158 | `transparent` | Independent |
| `--bs-btn-disabled-bg` | `transparent` | `.btn-outline-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3233 | scss/mixins/_buttons.scss:57<br>via scss/_buttons.scss:158 | `transparent` | Independent |
| `--bs-btn-disabled-bg` | `transparent` | `.btn-outline-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3284 | scss/mixins/_buttons.scss:57<br>via scss/_buttons.scss:158 | `transparent` | Independent |
| `--bs-btn-disabled-bg` | `transparent` | `.btn-outline-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3182 | scss/mixins/_buttons.scss:57<br>via scss/_buttons.scss:158 | `transparent` | Independent |
| `--bs-btn-disabled-bg` | `transparent` | `.btn-outline-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3199 | scss/mixins/_buttons.scss:57<br>via scss/_buttons.scss:158 | `transparent` | Independent |
| `--bs-btn-disabled-bg` | `transparent` | `.btn-outline-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3216 | scss/mixins/_buttons.scss:57<br>via scss/_buttons.scss:158 | `transparent` | Independent |
| `--bs-btn-disabled-bg` | `transparent` | `.btn-outline-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3250 | scss/mixins/_buttons.scss:57<br>via scss/_buttons.scss:158 | `transparent` | Independent |
| `--bs-btn-disabled-border-color` | `transparent` | `.btn-link`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3316 | scss/_buttons.scss:180<br>via scss/_buttons.scss:170 | `transparent` | Independent |
| `--bs-btn-hover-border-color` | `transparent` | `.btn`<br>(none) | src/bootstrap/reference/css/bootstrap.css:2965 | scss/_buttons.scss:18<br>via scss/_buttons.scss:5 | `transparent` | Independent |
| `--bs-btn-hover-border-color` | `transparent` | `.btn-link`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3312 | scss/_buttons.scss:176<br>via scss/_buttons.scss:170 | `transparent` | Independent |
| `--bs-gradient` | `none` | `.btn-outline-danger`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3269 | scss/mixins/_buttons.scss:59<br>via scss/_buttons.scss:158 | `none` | Independent |
| `--bs-gradient` | `none` | `.btn-outline-dark`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3303 | scss/mixins/_buttons.scss:59<br>via scss/_buttons.scss:158 | `none` | Independent |
| `--bs-gradient` | `none` | `.btn-outline-info`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3235 | scss/mixins/_buttons.scss:59<br>via scss/_buttons.scss:158 | `none` | Independent |
| `--bs-gradient` | `none` | `.btn-outline-light`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3286 | scss/mixins/_buttons.scss:59<br>via scss/_buttons.scss:158 | `none` | Independent |
| `--bs-gradient` | `none` | `.btn-outline-primary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3184 | scss/mixins/_buttons.scss:59<br>via scss/_buttons.scss:158 | `none` | Independent |
| `--bs-gradient` | `none` | `.btn-outline-secondary`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3201 | scss/mixins/_buttons.scss:59<br>via scss/_buttons.scss:158 | `none` | Independent |
| `--bs-gradient` | `none` | `.btn-outline-success`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3218 | scss/mixins/_buttons.scss:59<br>via scss/_buttons.scss:158 | `none` | Independent |
| `--bs-gradient` | `none` | `.btn-outline-warning`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3252 | scss/mixins/_buttons.scss:59<br>via scss/_buttons.scss:158 | `none` | Independent |
| `--bs-offcanvas-height` | `auto` | `.offcanvas-lg`<br>@media (min-width: 992px) | src/bootstrap/reference/css/bootstrap.css:6515 | scss/_offcanvas.scss:98<br>via scss/_offcanvas.scss:33 | `auto` | Independent |
| `--bs-offcanvas-height` | `auto` | `.offcanvas-md`<br>@media (min-width: 768px) | src/bootstrap/reference/css/bootstrap.css:6437 | scss/_offcanvas.scss:98<br>via scss/_offcanvas.scss:33 | `auto` | Independent |
| `--bs-offcanvas-height` | `auto` | `.offcanvas-sm`<br>@media (min-width: 576px) | src/bootstrap/reference/css/bootstrap.css:6359 | scss/_offcanvas.scss:98<br>via scss/_offcanvas.scss:33 | `auto` | Independent |
| `--bs-offcanvas-height` | `auto` | `.offcanvas-xl`<br>@media (min-width: 1200px) | src/bootstrap/reference/css/bootstrap.css:6593 | scss/_offcanvas.scss:98<br>via scss/_offcanvas.scss:33 | `auto` | Independent |
| `--bs-offcanvas-height` | `auto` | `.offcanvas-xxl`<br>@media (min-width: 1400px) | src/bootstrap/reference/css/bootstrap.css:6671 | scss/_offcanvas.scss:98<br>via scss/_offcanvas.scss:33 | `auto` | Independent |
| `--bs-position` | `end` | `.dropdown-menu-end`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3462 | scss/_dropdown.scss:102<br>via scss/_dropdown.scss:101 | `end` | Independent |
| `--bs-position` | `end` | `.dropdown-menu-lg-end`<br>@media (min-width: 992px) | src/bootstrap/reference/css/bootstrap.css:3510 | scss/_dropdown.scss:102<br>via scss/_dropdown.scss:101 | `end` | Independent |
| `--bs-position` | `start` | `.dropdown-menu-lg-start`<br>@media (min-width: 992px) | src/bootstrap/reference/css/bootstrap.css:3503 | scss/_dropdown.scss:93<br>via scss/_dropdown.scss:92 | `start` | Independent |
| `--bs-position` | `end` | `.dropdown-menu-md-end`<br>@media (min-width: 768px) | src/bootstrap/reference/css/bootstrap.css:3494 | scss/_dropdown.scss:102<br>via scss/_dropdown.scss:101 | `end` | Independent |
| `--bs-position` | `start` | `.dropdown-menu-md-start`<br>@media (min-width: 768px) | src/bootstrap/reference/css/bootstrap.css:3487 | scss/_dropdown.scss:93<br>via scss/_dropdown.scss:92 | `start` | Independent |
| `--bs-position` | `end` | `.dropdown-menu-sm-end`<br>@media (min-width: 576px) | src/bootstrap/reference/css/bootstrap.css:3478 | scss/_dropdown.scss:102<br>via scss/_dropdown.scss:101 | `end` | Independent |
| `--bs-position` | `start` | `.dropdown-menu-sm-start`<br>@media (min-width: 576px) | src/bootstrap/reference/css/bootstrap.css:3471 | scss/_dropdown.scss:93<br>via scss/_dropdown.scss:92 | `start` | Independent |
| `--bs-position` | `start` | `.dropdown-menu-start`<br>(none) | src/bootstrap/reference/css/bootstrap.css:3454 | scss/_dropdown.scss:93<br>via scss/_dropdown.scss:92 | `start` | Independent |
| `--bs-position` | `end` | `.dropdown-menu-xl-end`<br>@media (min-width: 1200px) | src/bootstrap/reference/css/bootstrap.css:3526 | scss/_dropdown.scss:102<br>via scss/_dropdown.scss:101 | `end` | Independent |
| `--bs-position` | `start` | `.dropdown-menu-xl-start`<br>@media (min-width: 1200px) | src/bootstrap/reference/css/bootstrap.css:3519 | scss/_dropdown.scss:93<br>via scss/_dropdown.scss:92 | `start` | Independent |
| `--bs-position` | `end` | `.dropdown-menu-xxl-end`<br>@media (min-width: 1400px) | src/bootstrap/reference/css/bootstrap.css:3542 | scss/_dropdown.scss:102<br>via scss/_dropdown.scss:101 | `end` | Independent |
| `--bs-position` | `start` | `.dropdown-menu-xxl-start`<br>@media (min-width: 1400px) | src/bootstrap/reference/css/bootstrap.css:3535 | scss/_dropdown.scss:93<br>via scss/_dropdown.scss:92 | `start` | Independent |
| `--bs-spinner-animation-name` | `spinner-border` | `.spinner-border`<br>(none) | src/bootstrap/reference/css/bootstrap.css:6241 | scss/_spinners.scss:30<br>via scss/_spinners.scss:23 | `spinner-border` | Independent |
| `--bs-spinner-animation-name` | `spinner-grow` | `.spinner-grow`<br>(none) | src/bootstrap/reference/css/bootstrap.css:6266 | scss/_spinners.scss:67<br>via scss/_spinners.scss:61 | `spinner-grow` | Independent |
| `--bs-table-bg-state` | `initial` | `.table`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1866 | scss/_tables.scss:10<br>via scss/_tables.scss:5 | `initial` | Independent |
| `--bs-table-bg-type` | `initial` | `.table`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1864 | scss/_tables.scss:8<br>via scss/_tables.scss:5 | `initial` | Independent |
| `--bs-table-color-state` | `initial` | `.table`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1865 | scss/_tables.scss:9<br>via scss/_tables.scss:5 | `initial` | Independent |
| `--bs-table-color-type` | `initial` | `.table`<br>(none) | src/bootstrap/reference/css/bootstrap.css:1863 | scss/_tables.scss:7<br>via scss/_tables.scss:5 | `initial` | Independent |

## E. Component/source index

Every property, by Bootstrap component (derived from the Sass file / mixin that generates the rule). Links go to the subgroup where each context is audited. "n" is the number of declaration contexts of that property in the component.

### Root

117 properties, 169 contexts.

| Property | n | Audited in |
|---|---|---|
| `--bs-black` | 1 | [c-black-2](#c-black-2) |
| `--bs-black-rgb` | 1 | [c-black-5](#c-black-5) |
| `--bs-blue` | 1 | [c-blue-1](#c-blue-1) |
| `--bs-body-bg` | 2 | [c-body-bg-3](#c-body-bg-3), [c-body-bg-dark-1](#c-body-bg-dark-1) |
| `--bs-body-bg-rgb` | 2 | [c-body-bg-4](#c-body-bg-4), [c-body-bg-dark-2](#c-body-bg-dark-2) |
| `--bs-body-color` | 2 | [c-body-color-2](#c-body-color-2), [c-body-color-dark-1](#c-body-color-dark-1) |
| `--bs-body-color-rgb` | 2 | [c-body-color-6](#c-body-color-6), [c-body-color-dark-3](#c-body-color-dark-3) |
| `--bs-body-font-family` | 1 | [c-font-family-sans-serif-2](#c-font-family-sans-serif-2) |
| `--bs-body-font-size` | 1 | [c-font-size-base-1](#c-font-size-base-1) |
| `--bs-body-font-weight` | 1 | [c-font-weight-normal-1](#c-font-weight-normal-1) |
| `--bs-body-line-height` | 1 | [c-line-height-base-1](#c-line-height-base-1) |
| `--bs-border-color` | 2 | [c-border-color-2](#c-border-color-2), [c-border-color-dark-1](#c-border-color-dark-1) |
| `--bs-border-color-translucent` | 2 | [c-border-color-translucent-2](#c-border-color-translucent-2), [c-border-color-translucent-dark-1](#c-border-color-translucent-dark-1) |
| `--bs-border-radius` | 1 | [c-border-radius-3](#c-border-radius-3) |
| `--bs-border-radius-2xl` | 1 | [c-border-radius-xxl-2](#c-border-radius-xxl-2) |
| `--bs-border-radius-lg` | 1 | [c-border-radius-lg-2](#c-border-radius-lg-2) |
| `--bs-border-radius-pill` | 1 | [c-border-radius-pill-1](#c-border-radius-pill-1) |
| `--bs-border-radius-sm` | 1 | [c-border-radius-sm-2](#c-border-radius-sm-2) |
| `--bs-border-radius-xl` | 1 | [c-border-radius-xl-1](#c-border-radius-xl-1) |
| `--bs-border-radius-xxl` | 1 | [c-border-radius-xxl-1](#c-border-radius-xxl-1) |
| `--bs-border-style` | 1 | [d-1](#d-1) |
| `--bs-border-width` | 1 | [c-border-width-2](#c-border-width-2) |
| `--bs-box-shadow` | 1 | [c-box-shadow-2](#c-box-shadow-2) |
| `--bs-box-shadow-inset` | 1 | [c-box-shadow-inset-1](#c-box-shadow-inset-1) |
| `--bs-box-shadow-lg` | 1 | [c-box-shadow-lg-1](#c-box-shadow-lg-1) |
| `--bs-box-shadow-sm` | 1 | [c-box-shadow-sm-2](#c-box-shadow-sm-2) |
| `--bs-code-color` | 2 | [c-pink-1](#c-pink-1), [c-pink-2](#c-pink-2) |
| `--bs-cyan` | 1 | [c-cyan-1](#c-cyan-1) |
| `--bs-danger` | 1 | [c-danger-1](#c-danger-1) |
| `--bs-danger-bg-subtle` | 2 | [c-danger-29](#c-danger-29), [c-danger-25](#c-danger-25) |
| `--bs-danger-border-subtle` | 2 | [c-danger-28](#c-danger-28), [c-danger-23](#c-danger-23) |
| `--bs-danger-rgb` | 1 | [c-danger-6](#c-danger-6) |
| `--bs-danger-text-emphasis` | 2 | [c-danger-24](#c-danger-24), [c-danger-27](#c-danger-27) |
| `--bs-dark` | 1 | [c-dark-1](#c-dark-1) |
| `--bs-dark-bg-subtle` | 2 | [c-gray-400-2](#c-gray-400-2), [c-gray-800-3](#c-gray-800-3) |
| `--bs-dark-border-subtle` | 2 | [c-gray-500-1](#c-gray-500-1), [c-gray-800-1](#c-gray-800-1) |
| `--bs-dark-rgb` | 1 | [c-dark-3](#c-dark-3) |
| `--bs-dark-text-emphasis` | 2 | [c-gray-700-3](#c-gray-700-3), [c-gray-300-1](#c-gray-300-1) |
| `--bs-emphasis-color` | 2 | [c-body-emphasis-color-3](#c-body-emphasis-color-3), [c-body-emphasis-color-dark-1](#c-body-emphasis-color-dark-1) |
| `--bs-emphasis-color-rgb` | 2 | [c-body-emphasis-color-11](#c-body-emphasis-color-11), [c-body-emphasis-color-dark-2](#c-body-emphasis-color-dark-2) |
| `--bs-focus-ring-color` | 1 | [c-primary-25](#c-primary-25) |
| `--bs-focus-ring-opacity` | 1 | [c-focus-ring-opacity-1](#c-focus-ring-opacity-1) |
| `--bs-focus-ring-width` | 1 | [c-focus-ring-width-1](#c-focus-ring-width-1) |
| `--bs-font-monospace` | 1 | [c-font-family-monospace-1](#c-font-family-monospace-1) |
| `--bs-font-sans-serif` | 1 | [c-font-family-sans-serif-1](#c-font-family-sans-serif-1) |
| `--bs-form-invalid-border-color` | 2 | [c-danger-1](#c-danger-1), [c-red-1](#c-red-1) |
| `--bs-form-invalid-color` | 2 | [c-danger-1](#c-danger-1), [c-red-1](#c-red-1) |
| `--bs-form-valid-border-color` | 2 | [c-success-1](#c-success-1), [c-green-1](#c-green-1) |
| `--bs-form-valid-color` | 2 | [c-success-1](#c-success-1), [c-green-1](#c-green-1) |
| `--bs-gradient` | 1 | [c-white-9](#c-white-9) |
| `--bs-gray` | 1 | [c-gray-600-1](#c-gray-600-1) |
| `--bs-gray-100` | 1 | [c-gray-100-2](#c-gray-100-2) |
| `--bs-gray-200` | 1 | [c-gray-200-2](#c-gray-200-2) |
| `--bs-gray-300` | 1 | [c-gray-300-1](#c-gray-300-1) |
| `--bs-gray-400` | 1 | [c-gray-400-2](#c-gray-400-2) |
| `--bs-gray-500` | 1 | [c-gray-500-1](#c-gray-500-1) |
| `--bs-gray-600` | 1 | [c-gray-600-1](#c-gray-600-1) |
| `--bs-gray-700` | 1 | [c-gray-700-3](#c-gray-700-3) |
| `--bs-gray-800` | 1 | [c-gray-800-1](#c-gray-800-1) |
| `--bs-gray-900` | 1 | [c-gray-900-1](#c-gray-900-1) |
| `--bs-gray-dark` | 1 | [c-gray-800-1](#c-gray-800-1) |
| `--bs-green` | 1 | [c-green-2](#c-green-2) |
| `--bs-heading-color` | 2 | [c-headings-color-1](#c-headings-color-1), [c-headings-color-dark-1](#c-headings-color-dark-1) |
| `--bs-highlight-bg` | 2 | [c-yellow-3](#c-yellow-3), [c-yellow-2](#c-yellow-2) |
| `--bs-highlight-color` | 2 | [c-body-color-2](#c-body-color-2), [c-body-color-dark-1](#c-body-color-dark-1) |
| `--bs-indigo` | 1 | [c-indigo-1](#c-indigo-1) |
| `--bs-info` | 1 | [c-info-1](#c-info-1) |
| `--bs-info-bg-subtle` | 2 | [c-info-28](#c-info-28), [c-info-20](#c-info-20) |
| `--bs-info-border-subtle` | 2 | [c-info-27](#c-info-27), [c-info-18](#c-info-18) |
| `--bs-info-rgb` | 1 | [c-info-6](#c-info-6) |
| `--bs-info-text-emphasis` | 2 | [c-info-19](#c-info-19), [c-info-26](#c-info-26) |
| `--bs-light` | 1 | [c-light-1](#c-light-1) |
| `--bs-light-bg-subtle` | 2 | [c-gray-100-3](#c-gray-100-3), [c-gray-800-1](#c-gray-800-1) |
| `--bs-light-border-subtle` | 2 | [c-gray-200-2](#c-gray-200-2), [c-gray-700-3](#c-gray-700-3) |
| `--bs-light-rgb` | 1 | [c-light-3](#c-light-3) |
| `--bs-light-text-emphasis` | 2 | [c-gray-700-3](#c-gray-700-3), [c-gray-100-2](#c-gray-100-2) |
| `--bs-link-color` | 2 | [c-primary-1](#c-primary-1), [c-primary-11](#c-primary-11) |
| `--bs-link-color-rgb` | 2 | [c-primary-8](#c-primary-8), [c-primary-41](#c-primary-41) |
| `--bs-link-decoration` | 1 | [d-1](#d-1) |
| `--bs-link-hover-color` | 2 | [c-primary-34](#c-primary-34), [c-primary-36](#c-primary-36) |
| `--bs-link-hover-color-rgb` | 2 | [c-primary-39](#c-primary-39), [c-primary-40](#c-primary-40) |
| `--bs-orange` | 1 | [c-orange-1](#c-orange-1) |
| `--bs-pink` | 1 | [c-pink-1](#c-pink-1) |
| `--bs-primary` | 1 | [c-primary-1](#c-primary-1) |
| `--bs-primary-bg-subtle` | 2 | [c-primary-38](#c-primary-38), [c-primary-33](#c-primary-33) |
| `--bs-primary-border-subtle` | 2 | [c-primary-37](#c-primary-37), [c-primary-31](#c-primary-31) |
| `--bs-primary-rgb` | 1 | [c-primary-8](#c-primary-8) |
| `--bs-primary-text-emphasis` | 2 | [c-primary-32](#c-primary-32), [c-primary-11](#c-primary-11) |
| `--bs-purple` | 1 | [c-purple-1](#c-purple-1) |
| `--bs-red` | 1 | [c-red-2](#c-red-2) |
| `--bs-secondary` | 1 | [c-secondary-1](#c-secondary-1) |
| `--bs-secondary-bg` | 2 | [c-body-secondary-bg-2](#c-body-secondary-bg-2), [c-body-secondary-bg-dark-1](#c-body-secondary-bg-dark-1) |
| `--bs-secondary-bg-rgb` | 2 | [c-body-secondary-bg-4](#c-body-secondary-bg-4), [c-body-secondary-bg-dark-2](#c-body-secondary-bg-dark-2) |
| `--bs-secondary-bg-subtle` | 2 | [c-secondary-28](#c-secondary-28), [c-secondary-24](#c-secondary-24) |
| `--bs-secondary-border-subtle` | 2 | [c-secondary-27](#c-secondary-27), [c-secondary-22](#c-secondary-22) |
| `--bs-secondary-color` | 2 | [c-body-secondary-color-2](#c-body-secondary-color-2), [c-body-secondary-color-dark-1](#c-body-secondary-color-dark-1) |
| `--bs-secondary-color-rgb` | 2 | [c-body-secondary-color-3](#c-body-secondary-color-3), [c-body-secondary-color-dark-2](#c-body-secondary-color-dark-2) |
| `--bs-secondary-rgb` | 1 | [c-secondary-6](#c-secondary-6) |
| `--bs-secondary-text-emphasis` | 2 | [c-secondary-23](#c-secondary-23), [c-secondary-26](#c-secondary-26) |
| `--bs-success` | 1 | [c-success-1](#c-success-1) |
| `--bs-success-bg-subtle` | 2 | [c-success-29](#c-success-29), [c-success-25](#c-success-25) |
| `--bs-success-border-subtle` | 2 | [c-success-28](#c-success-28), [c-success-23](#c-success-23) |
| `--bs-success-rgb` | 1 | [c-success-6](#c-success-6) |
| `--bs-success-text-emphasis` | 2 | [c-success-24](#c-success-24), [c-success-27](#c-success-27) |
| `--bs-teal` | 1 | [c-teal-1](#c-teal-1) |
| `--bs-tertiary-bg` | 2 | [c-body-tertiary-bg-2](#c-body-tertiary-bg-2), [c-body-tertiary-bg-dark-1](#c-body-tertiary-bg-dark-1) |
| `--bs-tertiary-bg-rgb` | 2 | [c-body-tertiary-bg-3](#c-body-tertiary-bg-3), [c-body-tertiary-bg-dark-2](#c-body-tertiary-bg-dark-2) |
| `--bs-tertiary-color` | 2 | [c-body-tertiary-color-1](#c-body-tertiary-color-1), [c-body-tertiary-color-dark-1](#c-body-tertiary-color-dark-1) |
| `--bs-tertiary-color-rgb` | 2 | [c-body-tertiary-color-2](#c-body-tertiary-color-2), [c-body-tertiary-color-dark-2](#c-body-tertiary-color-dark-2) |
| `--bs-warning` | 1 | [c-warning-1](#c-warning-1) |
| `--bs-warning-bg-subtle` | 2 | [c-warning-28](#c-warning-28), [c-warning-20](#c-warning-20) |
| `--bs-warning-border-subtle` | 2 | [c-warning-27](#c-warning-27), [c-warning-18](#c-warning-18) |
| `--bs-warning-rgb` | 1 | [c-warning-6](#c-warning-6) |
| `--bs-warning-text-emphasis` | 2 | [c-warning-19](#c-warning-19), [c-warning-26](#c-warning-26) |
| `--bs-white` | 1 | [c-white-1](#c-white-1) |
| `--bs-white-rgb` | 1 | [c-white-15](#c-white-15) |
| `--bs-yellow` | 1 | [c-yellow-1](#c-yellow-1) |

### Reboot

1 properties, 1 contexts.

| Property | n | Audited in |
|---|---|---|
| `--bs-link-color-rgb` | 1 | [c-primary-42](#c-primary-42) |

### Containers

2 properties, 2 contexts.

| Property | n | Audited in |
|---|---|---|
| `--bs-gutter-x` | 1 | [c-grid-gutter-width-1](#c-grid-gutter-width-1) |
| `--bs-gutter-y` | 1 | [d-3](#d-3) |

### Grid

8 properties, 80 contexts.

| Property | n | Audited in |
|---|---|---|
| `--bs-breakpoint-lg` | 1 | [c-grid-breakpoints-lg-1](#c-grid-breakpoints-lg-1) |
| `--bs-breakpoint-md` | 1 | [c-grid-breakpoints-md-1](#c-grid-breakpoints-md-1) |
| `--bs-breakpoint-sm` | 1 | [c-grid-breakpoints-sm-1](#c-grid-breakpoints-sm-1) |
| `--bs-breakpoint-xl` | 1 | [c-grid-breakpoints-xl-1](#c-grid-breakpoints-xl-1) |
| `--bs-breakpoint-xs` | 1 | [c-grid-breakpoints-xs-1](#c-grid-breakpoints-xs-1) |
| `--bs-breakpoint-xxl` | 1 | [c-grid-breakpoints-xxl-1](#c-grid-breakpoints-xxl-1) |
| `--bs-gutter-x` | 37 | [c-grid-gutter-width-1](#c-grid-gutter-width-1), [c-spacers-0-1](#c-spacers-0-1), [c-spacer-3](#c-spacer-3), [c-spacer-2](#c-spacer-2), [c-spacer-1](#c-spacer-1), [c-spacer-4](#c-spacer-4) (+1 more) |
| `--bs-gutter-y` | 37 | [d-3](#d-3), [c-spacers-0-1](#c-spacers-0-1), [c-spacer-3](#c-spacer-3), [c-spacer-2](#c-spacer-2), [c-spacer-1](#c-spacer-1), [c-spacer-4](#c-spacer-4) (+1 more) |

### Tables

14 properties, 94 contexts.

| Property | n | Audited in |
|---|---|---|
| `--bs-table-accent-bg` | 1 | [d-1](#d-1) |
| `--bs-table-active-bg` | 9 | [c-body-emphasis-color-4](#c-body-emphasis-color-4), [c-primary-21](#c-primary-21), [c-secondary-13](#c-secondary-13), [c-success-14](#c-success-14), [c-info-13](#c-info-13), [c-warning-13](#c-warning-13) (+3 more) |
| `--bs-table-active-color` | 9 | [c-body-emphasis-color-1](#c-body-emphasis-color-1), [c-primary-13](#c-primary-13), [c-secondary-7](#c-secondary-7), [c-success-7](#c-success-7), [c-info-7](#c-info-7), [c-warning-7](#c-warning-7) (+3 more) |
| `--bs-table-bg` | 9 | [c-body-bg-1](#c-body-bg-1), [c-primary-35](#c-primary-35), [c-secondary-25](#c-secondary-25), [c-success-26](#c-success-26), [c-info-21](#c-info-21), [c-warning-21](#c-warning-21) (+3 more) |
| `--bs-table-bg-state` | 3 | [d-4](#d-4), [d-2](#d-2) |
| `--bs-table-bg-type` | 3 | [d-4](#d-4), [d-2](#d-2) |
| `--bs-table-border-color` | 9 | [c-border-color-1](#c-border-color-1), [c-primary-22](#c-primary-22), [c-secondary-14](#c-secondary-14), [c-success-15](#c-success-15), [c-info-14](#c-info-14), [c-warning-14](#c-warning-14) (+3 more) |
| `--bs-table-color` | 9 | [c-body-emphasis-color-1](#c-body-emphasis-color-1), [c-primary-16](#c-primary-16), [c-secondary-10](#c-secondary-10), [c-success-10](#c-success-10), [c-info-10](#c-info-10), [c-warning-10](#c-warning-10) (+3 more) |
| `--bs-table-color-state` | 3 | [d-4](#d-4), [d-2](#d-2) |
| `--bs-table-color-type` | 3 | [d-4](#d-4), [d-2](#d-2) |
| `--bs-table-hover-bg` | 9 | [c-body-emphasis-color-5](#c-body-emphasis-color-5), [c-primary-23](#c-primary-23), [c-secondary-15](#c-secondary-15), [c-success-16](#c-success-16), [c-info-15](#c-info-15), [c-warning-15](#c-warning-15) (+3 more) |
| `--bs-table-hover-color` | 9 | [c-body-emphasis-color-1](#c-body-emphasis-color-1), [c-primary-14](#c-primary-14), [c-secondary-8](#c-secondary-8), [c-success-8](#c-success-8), [c-info-8](#c-info-8), [c-warning-8](#c-warning-8) (+3 more) |
| `--bs-table-striped-bg` | 9 | [c-body-emphasis-color-6](#c-body-emphasis-color-6), [c-primary-24](#c-primary-24), [c-secondary-16](#c-secondary-16), [c-success-17](#c-success-17), [c-info-16](#c-info-16), [c-warning-16](#c-warning-16) (+3 more) |
| `--bs-table-striped-color` | 9 | [c-body-emphasis-color-1](#c-body-emphasis-color-1), [c-primary-15](#c-primary-15), [c-secondary-9](#c-secondary-9), [c-success-9](#c-success-9), [c-info-9](#c-info-9), [c-warning-9](#c-warning-9) (+3 more) |

### Forms

5 properties, 12 contexts.

| Property | n | Audited in |
|---|---|---|
| `--bs-form-check-bg` | 1 | [c-body-bg-1](#c-body-bg-1) |
| `--bs-form-check-bg-image` | 3 | [c-white-8](#c-white-8), [c-white-4](#c-white-4), [c-white-7](#c-white-7) |
| `--bs-form-select-bg-icon` | 2 | [c-success-13](#c-success-13), [c-danger-13](#c-danger-13) |
| `--bs-form-select-bg-img` | 2 | [c-gray-800-2](#c-gray-800-2), [c-body-color-dark-2](#c-body-color-dark-2) |
| `--bs-form-switch-bg` | 4 | [c-black-3](#c-black-3), [c-primary-19](#c-primary-19), [c-white-5](#c-white-5), [c-white-6](#c-white-6) |

### Buttons

26 properties, 259 contexts.

| Property | n | Audited in |
|---|---|---|
| `--bs-btn-active-bg` | 16 | [c-primary-27](#c-primary-27), [c-secondary-18](#c-secondary-18), [c-success-19](#c-success-19), [c-info-22](#c-info-22), [c-warning-22](#c-warning-22), [c-danger-19](#c-danger-19) (+10 more) |
| `--bs-btn-active-border-color` | 17 | [c-primary-28](#c-primary-28), [c-secondary-19](#c-secondary-19), [c-success-20](#c-success-20), [c-info-23](#c-info-23), [c-warning-23](#c-warning-23), [c-danger-20](#c-danger-20) (+11 more) |
| `--bs-btn-active-color` | 17 | [c-primary-17](#c-primary-17), [c-secondary-11](#c-secondary-11), [c-success-11](#c-success-11), [c-info-11](#c-info-11), [c-warning-11](#c-warning-11), [c-danger-11](#c-danger-11) (+11 more) |
| `--bs-btn-active-shadow` | 16 | [c-black-1](#c-black-1) |
| `--bs-btn-bg` | 10 | [d-4](#d-4), [c-primary-1](#c-primary-1), [c-secondary-1](#c-secondary-1), [c-success-1](#c-success-1), [c-info-1](#c-info-1), [c-warning-1](#c-warning-1) (+3 more) |
| `--bs-btn-border-color` | 18 | [d-4](#d-4), [c-primary-1](#c-primary-1), [c-secondary-1](#c-secondary-1), [c-success-1](#c-success-1), [c-info-1](#c-info-1), [c-warning-1](#c-warning-1) (+3 more) |
| `--bs-btn-border-radius` | 3 | [c-border-radius-1](#c-border-radius-1), [c-border-radius-lg-1](#c-border-radius-lg-1), [c-border-radius-sm-1](#c-border-radius-sm-1) |
| `--bs-btn-border-width` | 1 | [c-border-width-1](#c-border-width-1) |
| `--bs-btn-box-shadow` | 2 | [c-white-3](#c-white-3), [d-4](#d-4) |
| `--bs-btn-color` | 18 | [c-body-color-1](#c-body-color-1), [c-primary-4](#c-primary-4), [c-secondary-3](#c-secondary-3), [c-success-3](#c-success-3), [c-info-3](#c-info-3), [c-warning-3](#c-warning-3) (+12 more) |
| `--bs-btn-disabled-bg` | 16 | [c-primary-1](#c-primary-1), [c-secondary-1](#c-secondary-1), [c-success-1](#c-success-1), [c-info-1](#c-info-1), [c-warning-1](#c-warning-1), [c-danger-1](#c-danger-1) (+3 more) |
| `--bs-btn-disabled-border-color` | 17 | [c-primary-1](#c-primary-1), [c-secondary-1](#c-secondary-1), [c-success-1](#c-success-1), [c-info-1](#c-info-1), [c-warning-1](#c-warning-1), [c-danger-1](#c-danger-1) (+3 more) |
| `--bs-btn-disabled-color` | 17 | [c-primary-4](#c-primary-4), [c-secondary-3](#c-secondary-3), [c-success-3](#c-success-3), [c-info-3](#c-info-3), [c-warning-3](#c-warning-3), [c-danger-3](#c-danger-3) (+11 more) |
| `--bs-btn-disabled-opacity` | 1 | [c-btn-disabled-opacity-1](#c-btn-disabled-opacity-1) |
| `--bs-btn-focus-box-shadow` | 1 | [c-focus-ring-width-2](#c-focus-ring-width-2) |
| `--bs-btn-focus-shadow-rgb` | 17 | [c-primary-12](#c-primary-12), [c-secondary-29](#c-secondary-29), [c-success-30](#c-success-30), [c-info-29](#c-info-29), [c-warning-29](#c-warning-29), [c-danger-30](#c-danger-30) (+10 more) |
| `--bs-btn-font-family` | 1 | [d-1](#d-1) |
| `--bs-btn-font-size` | 3 | [c-font-size-base-1](#c-font-size-base-1), [c-font-size-base-3](#c-font-size-base-3), [c-font-size-base-2](#c-font-size-base-2) |
| `--bs-btn-font-weight` | 2 | [c-font-weight-normal-1](#c-font-weight-normal-1) |
| `--bs-btn-hover-bg` | 16 | [c-primary-29](#c-primary-29), [c-secondary-20](#c-secondary-20), [c-success-21](#c-success-21), [c-info-24](#c-info-24), [c-warning-24](#c-warning-24), [c-danger-21](#c-danger-21) (+10 more) |
| `--bs-btn-hover-border-color` | 18 | [d-4](#d-4), [c-primary-30](#c-primary-30), [c-secondary-21](#c-secondary-21), [c-success-22](#c-success-22), [c-info-25](#c-info-25), [c-warning-25](#c-warning-25) (+11 more) |
| `--bs-btn-hover-color` | 17 | [c-primary-18](#c-primary-18), [c-secondary-12](#c-secondary-12), [c-success-12](#c-success-12), [c-info-12](#c-info-12), [c-warning-12](#c-warning-12), [c-danger-12](#c-danger-12) (+11 more) |
| `--bs-btn-line-height` | 1 | [c-line-height-base-1](#c-line-height-base-1) |
| `--bs-btn-padding-x` | 3 | [d-1](#d-1) |
| `--bs-btn-padding-y` | 3 | [d-1](#d-1) |
| `--bs-gradient` | 8 | [d-4](#d-4) |

### Dropdown

27 properties, 50 contexts.

| Property | n | Audited in |
|---|---|---|
| `--bs-dropdown-bg` | 2 | [c-body-bg-1](#c-body-bg-1), [c-gray-800-1](#c-gray-800-1) |
| `--bs-dropdown-border-color` | 2 | [c-border-color-translucent-1](#c-border-color-translucent-1) |
| `--bs-dropdown-border-radius` | 1 | [c-border-radius-1](#c-border-radius-1) |
| `--bs-dropdown-border-width` | 1 | [c-border-width-1](#c-border-width-1) |
| `--bs-dropdown-box-shadow` | 2 | [c-box-shadow-1](#c-box-shadow-1), [d-1](#d-1) |
| `--bs-dropdown-color` | 2 | [c-body-color-1](#c-body-color-1), [c-gray-300-1](#c-gray-300-1) |
| `--bs-dropdown-divider-bg` | 2 | [c-border-color-translucent-1](#c-border-color-translucent-1) |
| `--bs-dropdown-divider-margin-y` | 1 | [c-spacer-2](#c-spacer-2) |
| `--bs-dropdown-font-size` | 1 | [c-font-size-base-1](#c-font-size-base-1) |
| `--bs-dropdown-header-color` | 2 | [c-gray-600-1](#c-gray-600-1), [c-gray-500-1](#c-gray-500-1) |
| `--bs-dropdown-header-padding-x` | 1 | [c-spacer-1](#c-spacer-1) |
| `--bs-dropdown-header-padding-y` | 1 | [c-dropdown-padding-y-1](#c-dropdown-padding-y-1) |
| `--bs-dropdown-inner-border-radius` | 1 | [c-border-radius-4](#c-border-radius-4) |
| `--bs-dropdown-item-padding-x` | 1 | [c-spacer-1](#c-spacer-1) |
| `--bs-dropdown-item-padding-y` | 1 | [c-spacer-3](#c-spacer-3) |
| `--bs-dropdown-link-active-bg` | 2 | [c-primary-1](#c-primary-1) |
| `--bs-dropdown-link-active-color` | 2 | [c-white-1](#c-white-1) |
| `--bs-dropdown-link-color` | 2 | [c-body-color-1](#c-body-color-1), [c-gray-300-1](#c-gray-300-1) |
| `--bs-dropdown-link-disabled-color` | 2 | [c-body-tertiary-color-3](#c-body-tertiary-color-3), [c-gray-500-1](#c-gray-500-1) |
| `--bs-dropdown-link-hover-bg` | 2 | [c-body-tertiary-bg-1](#c-body-tertiary-bg-1), [c-white-11](#c-white-11) |
| `--bs-dropdown-link-hover-color` | 2 | [c-body-color-1](#c-body-color-1), [c-white-1](#c-white-1) |
| `--bs-dropdown-min-width` | 1 | [d-1](#d-1) |
| `--bs-dropdown-padding-x` | 1 | [d-1](#d-1) |
| `--bs-dropdown-padding-y` | 1 | [c-dropdown-padding-y-1](#c-dropdown-padding-y-1) |
| `--bs-dropdown-spacer` | 1 | [d-1](#d-1) |
| `--bs-dropdown-zindex` | 1 | [c-zindex-dropdown-1](#c-zindex-dropdown-1) |
| `--bs-position` | 12 | [d-4](#d-4) |

### Nav

19 properties, 19 contexts.

| Property | n | Audited in |
|---|---|---|
| `--bs-nav-link-color` | 1 | [c-primary-9](#c-primary-9) |
| `--bs-nav-link-disabled-color` | 1 | [c-body-secondary-color-1](#c-body-secondary-color-1) |
| `--bs-nav-link-font-weight` | 1 | [d-1](#d-1) |
| `--bs-nav-link-hover-color` | 1 | [c-primary-3](#c-primary-3) |
| `--bs-nav-link-padding-x` | 1 | [d-1](#d-1) |
| `--bs-nav-link-padding-y` | 1 | [d-1](#d-1) |
| `--bs-nav-pills-border-radius` | 1 | [c-border-radius-1](#c-border-radius-1) |
| `--bs-nav-pills-link-active-bg` | 1 | [c-primary-1](#c-primary-1) |
| `--bs-nav-pills-link-active-color` | 1 | [c-white-1](#c-white-1) |
| `--bs-nav-tabs-border-color` | 1 | [c-border-color-1](#c-border-color-1) |
| `--bs-nav-tabs-border-radius` | 1 | [c-border-radius-1](#c-border-radius-1) |
| `--bs-nav-tabs-border-width` | 1 | [c-border-width-1](#c-border-width-1) |
| `--bs-nav-tabs-link-active-bg` | 1 | [c-body-bg-1](#c-body-bg-1) |
| `--bs-nav-tabs-link-active-border-color` | 1 | [c-border-color-3](#c-border-color-3) |
| `--bs-nav-tabs-link-active-color` | 1 | [c-body-emphasis-color-1](#c-body-emphasis-color-1) |
| `--bs-nav-tabs-link-hover-border-color` | 1 | [c-body-secondary-bg-3](#c-body-secondary-bg-3) |
| `--bs-nav-underline-border-width` | 1 | [d-1](#d-1) |
| `--bs-nav-underline-gap` | 1 | [d-1](#d-1) |
| `--bs-nav-underline-link-active-color` | 1 | [c-body-emphasis-color-1](#c-body-emphasis-color-1) |

### Navbar

26 properties, 35 contexts.

| Property | n | Audited in |
|---|---|---|
| `--bs-nav-link-color` | 1 | [d-2](#d-2) |
| `--bs-nav-link-disabled-color` | 1 | [d-2](#d-2) |
| `--bs-nav-link-font-weight` | 1 | [d-1](#d-1) |
| `--bs-nav-link-hover-color` | 1 | [d-2](#d-2) |
| `--bs-nav-link-padding-x` | 1 | [d-3](#d-3) |
| `--bs-nav-link-padding-y` | 1 | [d-1](#d-1) |
| `--bs-navbar-active-color` | 2 | [c-body-emphasis-color-2](#c-body-emphasis-color-2), [c-white-1](#c-white-1) |
| `--bs-navbar-brand-color` | 2 | [c-body-emphasis-color-2](#c-body-emphasis-color-2), [c-white-1](#c-white-1) |
| `--bs-navbar-brand-font-size` | 1 | [c-font-size-base-3](#c-font-size-base-3) |
| `--bs-navbar-brand-hover-color` | 2 | [c-body-emphasis-color-2](#c-body-emphasis-color-2), [c-white-1](#c-white-1) |
| `--bs-navbar-brand-margin-end` | 1 | [d-1](#d-1) |
| `--bs-navbar-brand-padding-y` | 1 | [c-font-size-base-5](#c-font-size-base-5) |
| `--bs-navbar-color` | 2 | [c-body-emphasis-color-9](#c-body-emphasis-color-9), [c-white-13](#c-white-13) |
| `--bs-navbar-disabled-color` | 2 | [c-body-emphasis-color-8](#c-body-emphasis-color-8), [c-white-12](#c-white-12) |
| `--bs-navbar-hover-color` | 2 | [c-body-emphasis-color-10](#c-body-emphasis-color-10), [c-white-14](#c-white-14) |
| `--bs-navbar-nav-link-padding-x` | 1 | [d-1](#d-1) |
| `--bs-navbar-padding-x` | 1 | [d-1](#d-1) |
| `--bs-navbar-padding-y` | 1 | [c-spacer-2](#c-spacer-2) |
| `--bs-navbar-toggler-border-color` | 2 | [c-body-emphasis-color-7](#c-body-emphasis-color-7), [c-white-10](#c-white-10) |
| `--bs-navbar-toggler-border-radius` | 1 | [c-border-radius-1](#c-border-radius-1) |
| `--bs-navbar-toggler-focus-width` | 1 | [c-focus-ring-width-1](#c-focus-ring-width-1) |
| `--bs-navbar-toggler-font-size` | 1 | [c-font-size-base-3](#c-font-size-base-3) |
| `--bs-navbar-toggler-icon-bg` | 3 | [c-body-color-4](#c-body-color-4), [c-white-2](#c-white-2) |
| `--bs-navbar-toggler-padding-x` | 1 | [d-1](#d-1) |
| `--bs-navbar-toggler-padding-y` | 1 | [d-1](#d-1) |
| `--bs-navbar-toggler-transition` | 1 | [d-1](#d-1) |

### Card

19 properties, 19 contexts.

| Property | n | Audited in |
|---|---|---|
| `--bs-card-bg` | 1 | [c-body-bg-1](#c-body-bg-1) |
| `--bs-card-border-color` | 1 | [c-border-color-translucent-1](#c-border-color-translucent-1) |
| `--bs-card-border-radius` | 1 | [c-border-radius-1](#c-border-radius-1) |
| `--bs-card-border-width` | 1 | [c-border-width-1](#c-border-width-1) |
| `--bs-card-box-shadow` | 1 | [d-1](#d-1) |
| `--bs-card-cap-bg` | 1 | [c-body-color-5](#c-body-color-5) |
| `--bs-card-cap-color` | 1 | [d-1](#d-1) |
| `--bs-card-cap-padding-x` | 1 | [c-spacer-1](#c-spacer-1) |
| `--bs-card-cap-padding-y` | 1 | [c-spacer-2](#c-spacer-2) |
| `--bs-card-color` | 1 | [d-1](#d-1) |
| `--bs-card-group-margin` | 1 | [c-grid-gutter-width-2](#c-grid-gutter-width-2) |
| `--bs-card-height` | 1 | [d-1](#d-1) |
| `--bs-card-img-overlay-padding` | 1 | [c-spacer-1](#c-spacer-1) |
| `--bs-card-inner-border-radius` | 1 | [c-border-radius-2](#c-border-radius-2) |
| `--bs-card-spacer-x` | 1 | [c-spacer-1](#c-spacer-1) |
| `--bs-card-spacer-y` | 1 | [c-spacer-1](#c-spacer-1) |
| `--bs-card-subtitle-color` | 1 | [d-1](#d-1) |
| `--bs-card-title-color` | 1 | [d-1](#d-1) |
| `--bs-card-title-spacer-y` | 1 | [c-spacer-2](#c-spacer-2) |

### Accordion

21 properties, 23 contexts.

| Property | n | Audited in |
|---|---|---|
| `--bs-accordion-active-bg` | 1 | [c-primary-5](#c-primary-5) |
| `--bs-accordion-active-color` | 1 | [c-primary-2](#c-primary-2) |
| `--bs-accordion-bg` | 1 | [c-body-bg-1](#c-body-bg-1) |
| `--bs-accordion-body-padding-x` | 1 | [c-accordion-padding-x-1](#c-accordion-padding-x-1) |
| `--bs-accordion-body-padding-y` | 1 | [c-accordion-padding-y-1](#c-accordion-padding-y-1) |
| `--bs-accordion-border-color` | 1 | [c-border-color-1](#c-border-color-1) |
| `--bs-accordion-border-radius` | 1 | [c-border-radius-1](#c-border-radius-1) |
| `--bs-accordion-border-width` | 1 | [c-border-width-1](#c-border-width-1) |
| `--bs-accordion-btn-active-icon` | 2 | [c-primary-20](#c-primary-20), [c-primary-10](#c-primary-10) |
| `--bs-accordion-btn-bg` | 1 | [c-body-bg-5](#c-body-bg-5) |
| `--bs-accordion-btn-color` | 1 | [c-body-color-1](#c-body-color-1) |
| `--bs-accordion-btn-focus-box-shadow` | 1 | [c-primary-7](#c-primary-7) |
| `--bs-accordion-btn-icon` | 2 | [c-body-color-3](#c-body-color-3), [c-primary-10](#c-primary-10) |
| `--bs-accordion-btn-icon-transform` | 1 | [d-1](#d-1) |
| `--bs-accordion-btn-icon-transition` | 1 | [d-1](#d-1) |
| `--bs-accordion-btn-icon-width` | 1 | [d-1](#d-1) |
| `--bs-accordion-btn-padding-x` | 1 | [c-accordion-padding-x-1](#c-accordion-padding-x-1) |
| `--bs-accordion-btn-padding-y` | 1 | [c-accordion-padding-y-1](#c-accordion-padding-y-1) |
| `--bs-accordion-color` | 1 | [c-body-color-1](#c-body-color-1) |
| `--bs-accordion-inner-border-radius` | 1 | [c-border-radius-2](#c-border-radius-2) |
| `--bs-accordion-transition` | 1 | [d-1](#d-1) |

### Breadcrumb

8 properties, 8 contexts.

| Property | n | Audited in |
|---|---|---|
| `--bs-breadcrumb-bg` | 1 | [d-1](#d-1) |
| `--bs-breadcrumb-border-radius` | 1 | [d-1](#d-1) |
| `--bs-breadcrumb-divider-color` | 1 | [c-body-secondary-color-1](#c-body-secondary-color-1) |
| `--bs-breadcrumb-item-active-color` | 1 | [c-body-secondary-color-1](#c-body-secondary-color-1) |
| `--bs-breadcrumb-item-padding-x` | 1 | [d-1](#d-1) |
| `--bs-breadcrumb-margin-bottom` | 1 | [d-1](#d-1) |
| `--bs-breadcrumb-padding-x` | 1 | [d-1](#d-1) |
| `--bs-breadcrumb-padding-y` | 1 | [d-1](#d-1) |

### Pagination

20 properties, 28 contexts.

| Property | n | Audited in |
|---|---|---|
| `--bs-pagination-active-bg` | 1 | [c-primary-1](#c-primary-1) |
| `--bs-pagination-active-border-color` | 1 | [c-primary-1](#c-primary-1) |
| `--bs-pagination-active-color` | 1 | [c-white-1](#c-white-1) |
| `--bs-pagination-bg` | 1 | [c-body-bg-1](#c-body-bg-1) |
| `--bs-pagination-border-color` | 1 | [c-border-color-1](#c-border-color-1) |
| `--bs-pagination-border-radius` | 3 | [c-border-radius-1](#c-border-radius-1), [c-border-radius-lg-1](#c-border-radius-lg-1), [c-border-radius-sm-1](#c-border-radius-sm-1) |
| `--bs-pagination-border-width` | 1 | [c-border-width-1](#c-border-width-1) |
| `--bs-pagination-color` | 1 | [c-primary-9](#c-primary-9) |
| `--bs-pagination-disabled-bg` | 1 | [c-body-secondary-bg-1](#c-body-secondary-bg-1) |
| `--bs-pagination-disabled-border-color` | 1 | [c-border-color-1](#c-border-color-1) |
| `--bs-pagination-disabled-color` | 1 | [c-body-secondary-color-1](#c-body-secondary-color-1) |
| `--bs-pagination-focus-bg` | 1 | [c-body-secondary-bg-1](#c-body-secondary-bg-1) |
| `--bs-pagination-focus-box-shadow` | 1 | [c-primary-7](#c-primary-7) |
| `--bs-pagination-focus-color` | 1 | [c-primary-3](#c-primary-3) |
| `--bs-pagination-font-size` | 3 | [c-font-size-base-1](#c-font-size-base-1), [c-font-size-base-3](#c-font-size-base-3), [c-font-size-base-2](#c-font-size-base-2) |
| `--bs-pagination-hover-bg` | 1 | [c-body-tertiary-bg-1](#c-body-tertiary-bg-1) |
| `--bs-pagination-hover-border-color` | 1 | [c-border-color-1](#c-border-color-1) |
| `--bs-pagination-hover-color` | 1 | [c-primary-3](#c-primary-3) |
| `--bs-pagination-padding-x` | 3 | [d-1](#d-1) |
| `--bs-pagination-padding-y` | 3 | [d-1](#d-1) |

### Badge

6 properties, 6 contexts.

| Property | n | Audited in |
|---|---|---|
| `--bs-badge-border-radius` | 1 | [c-border-radius-1](#c-border-radius-1) |
| `--bs-badge-color` | 1 | [c-white-1](#c-white-1) |
| `--bs-badge-font-size` | 1 | [d-1](#d-1) |
| `--bs-badge-font-weight` | 1 | [c-font-weight-bold-1](#c-font-weight-bold-1) |
| `--bs-badge-padding-x` | 1 | [d-1](#d-1) |
| `--bs-badge-padding-y` | 1 | [d-1](#d-1) |

### Alert

9 properties, 41 contexts.

| Property | n | Audited in |
|---|---|---|
| `--bs-alert-bg` | 9 | [d-4](#d-4), [c-primary-5](#c-primary-5), [c-secondary-5](#c-secondary-5), [c-success-5](#c-success-5), [c-info-5](#c-info-5), [c-warning-5](#c-warning-5) (+3 more) |
| `--bs-alert-border` | 1 | [c-border-width-3](#c-border-width-3) |
| `--bs-alert-border-color` | 9 | [d-4](#d-4), [c-primary-6](#c-primary-6), [c-secondary-4](#c-secondary-4), [c-success-4](#c-success-4), [c-info-4](#c-info-4), [c-warning-4](#c-warning-4) (+3 more) |
| `--bs-alert-border-radius` | 1 | [c-border-radius-1](#c-border-radius-1) |
| `--bs-alert-color` | 9 | [d-4](#d-4), [c-primary-2](#c-primary-2), [c-secondary-2](#c-secondary-2), [c-success-2](#c-success-2), [c-info-2](#c-info-2), [c-warning-2](#c-warning-2) (+3 more) |
| `--bs-alert-link-color` | 9 | [d-4](#d-4), [c-primary-2](#c-primary-2), [c-secondary-2](#c-secondary-2), [c-success-2](#c-success-2), [c-info-2](#c-info-2), [c-warning-2](#c-warning-2) (+3 more) |
| `--bs-alert-margin-bottom` | 1 | [d-1](#d-1) |
| `--bs-alert-padding-x` | 1 | [c-spacer-1](#c-spacer-1) |
| `--bs-alert-padding-y` | 1 | [c-spacer-1](#c-spacer-1) |

### Progress

8 properties, 8 contexts.

| Property | n | Audited in |
|---|---|---|
| `--bs-progress-bar-bg` | 1 | [c-primary-1](#c-primary-1) |
| `--bs-progress-bar-color` | 1 | [c-white-1](#c-white-1) |
| `--bs-progress-bar-transition` | 1 | [d-1](#d-1) |
| `--bs-progress-bg` | 1 | [c-body-secondary-bg-1](#c-body-secondary-bg-1) |
| `--bs-progress-border-radius` | 1 | [c-border-radius-1](#c-border-radius-1) |
| `--bs-progress-box-shadow` | 1 | [c-box-shadow-inset-2](#c-box-shadow-inset-2) |
| `--bs-progress-font-size` | 1 | [c-font-size-base-4](#c-font-size-base-4) |
| `--bs-progress-height` | 1 | [d-1](#d-1) |

### List group

17 properties, 97 contexts.

| Property | n | Audited in |
|---|---|---|
| `--bs-list-group-action-active-bg` | 9 | [c-body-secondary-bg-1](#c-body-secondary-bg-1), [c-primary-6](#c-primary-6), [c-secondary-4](#c-secondary-4), [c-success-4](#c-success-4), [c-info-4](#c-info-4), [c-warning-4](#c-warning-4) (+3 more) |
| `--bs-list-group-action-active-color` | 9 | [c-body-color-1](#c-body-color-1), [c-body-emphasis-color-1](#c-body-emphasis-color-1) |
| `--bs-list-group-action-color` | 1 | [c-body-secondary-color-1](#c-body-secondary-color-1) |
| `--bs-list-group-action-hover-bg` | 9 | [c-body-tertiary-bg-1](#c-body-tertiary-bg-1), [c-primary-6](#c-primary-6), [c-secondary-4](#c-secondary-4), [c-success-4](#c-success-4), [c-info-4](#c-info-4), [c-warning-4](#c-warning-4) (+3 more) |
| `--bs-list-group-action-hover-color` | 9 | [c-body-emphasis-color-1](#c-body-emphasis-color-1) |
| `--bs-list-group-active-bg` | 9 | [c-primary-1](#c-primary-1), [c-primary-2](#c-primary-2), [c-secondary-2](#c-secondary-2), [c-success-2](#c-success-2), [c-info-2](#c-info-2), [c-warning-2](#c-warning-2) (+3 more) |
| `--bs-list-group-active-border-color` | 9 | [c-primary-1](#c-primary-1), [c-primary-2](#c-primary-2), [c-secondary-2](#c-secondary-2), [c-success-2](#c-success-2), [c-info-2](#c-info-2), [c-warning-2](#c-warning-2) (+3 more) |
| `--bs-list-group-active-color` | 9 | [c-white-1](#c-white-1), [c-primary-5](#c-primary-5), [c-secondary-5](#c-secondary-5), [c-success-5](#c-success-5), [c-info-5](#c-info-5), [c-warning-5](#c-warning-5) (+3 more) |
| `--bs-list-group-bg` | 9 | [c-body-bg-1](#c-body-bg-1), [c-primary-5](#c-primary-5), [c-secondary-5](#c-secondary-5), [c-success-5](#c-success-5), [c-info-5](#c-info-5), [c-warning-5](#c-warning-5) (+3 more) |
| `--bs-list-group-border-color` | 9 | [c-border-color-1](#c-border-color-1), [c-primary-6](#c-primary-6), [c-secondary-4](#c-secondary-4), [c-success-4](#c-success-4), [c-info-4](#c-info-4), [c-warning-4](#c-warning-4) (+3 more) |
| `--bs-list-group-border-radius` | 1 | [c-border-radius-1](#c-border-radius-1) |
| `--bs-list-group-border-width` | 1 | [c-border-width-1](#c-border-width-1) |
| `--bs-list-group-color` | 9 | [c-body-color-1](#c-body-color-1), [c-primary-2](#c-primary-2), [c-secondary-2](#c-secondary-2), [c-success-2](#c-success-2), [c-info-2](#c-info-2), [c-warning-2](#c-warning-2) (+3 more) |
| `--bs-list-group-disabled-bg` | 1 | [c-body-bg-1](#c-body-bg-1) |
| `--bs-list-group-disabled-color` | 1 | [c-body-secondary-color-1](#c-body-secondary-color-1) |
| `--bs-list-group-item-padding-x` | 1 | [c-spacer-1](#c-spacer-1) |
| `--bs-list-group-item-padding-y` | 1 | [c-spacer-2](#c-spacer-2) |

### Close button

8 properties, 10 contexts.

| Property | n | Audited in |
|---|---|---|
| `--bs-btn-close-bg` | 1 | [c-black-4](#c-black-4) |
| `--bs-btn-close-color` | 1 | [c-black-2](#c-black-2) |
| `--bs-btn-close-disabled-opacity` | 1 | [c-btn-close-disabled-opacity-1](#c-btn-close-disabled-opacity-1) |
| `--bs-btn-close-filter` | 3 | [d-1](#d-1) |
| `--bs-btn-close-focus-opacity` | 1 | [c-btn-close-focus-opacity-1](#c-btn-close-focus-opacity-1) |
| `--bs-btn-close-focus-shadow` | 1 | [c-primary-7](#c-primary-7) |
| `--bs-btn-close-hover-opacity` | 1 | [c-btn-close-hover-opacity-1](#c-btn-close-hover-opacity-1) |
| `--bs-btn-close-opacity` | 1 | [c-btn-close-opacity-1](#c-btn-close-opacity-1) |

### Toasts

15 properties, 16 contexts.

| Property | n | Audited in |
|---|---|---|
| `--bs-toast-bg` | 1 | [c-body-bg-2](#c-body-bg-2) |
| `--bs-toast-border-color` | 1 | [c-border-color-translucent-1](#c-border-color-translucent-1) |
| `--bs-toast-border-radius` | 1 | [c-border-radius-1](#c-border-radius-1) |
| `--bs-toast-border-width` | 1 | [c-border-width-1](#c-border-width-1) |
| `--bs-toast-box-shadow` | 1 | [c-box-shadow-1](#c-box-shadow-1) |
| `--bs-toast-color` | 1 | [d-1](#d-1) |
| `--bs-toast-font-size` | 1 | [d-1](#d-1) |
| `--bs-toast-header-bg` | 1 | [c-body-bg-2](#c-body-bg-2) |
| `--bs-toast-header-border-color` | 1 | [c-border-color-translucent-1](#c-border-color-translucent-1) |
| `--bs-toast-header-color` | 1 | [c-body-secondary-color-1](#c-body-secondary-color-1) |
| `--bs-toast-max-width` | 1 | [d-1](#d-1) |
| `--bs-toast-padding-x` | 1 | [d-1](#d-1) |
| `--bs-toast-padding-y` | 1 | [d-1](#d-1) |
| `--bs-toast-spacing` | 1 | [c-grid-gutter-width-1](#c-grid-gutter-width-1) |
| `--bs-toast-zindex` | 2 | [c-zindex-toast-1](#c-zindex-toast-1) |

### Modal

24 properties, 30 contexts.

| Property | n | Audited in |
|---|---|---|
| `--bs-backdrop-bg` | 1 | [c-black-2](#c-black-2) |
| `--bs-backdrop-opacity` | 1 | [c-modal-backdrop-opacity-1](#c-modal-backdrop-opacity-1) |
| `--bs-backdrop-zindex` | 1 | [c-zindex-modal-backdrop-1](#c-zindex-modal-backdrop-1) |
| `--bs-modal-bg` | 1 | [c-body-bg-1](#c-body-bg-1) |
| `--bs-modal-border-color` | 1 | [c-border-color-translucent-1](#c-border-color-translucent-1) |
| `--bs-modal-border-radius` | 1 | [c-border-radius-lg-1](#c-border-radius-lg-1) |
| `--bs-modal-border-width` | 1 | [c-border-width-1](#c-border-width-1) |
| `--bs-modal-box-shadow` | 2 | [c-box-shadow-sm-1](#c-box-shadow-sm-1), [c-box-shadow-1](#c-box-shadow-1) |
| `--bs-modal-color` | 1 | [c-body-color-1](#c-body-color-1) |
| `--bs-modal-footer-bg` | 1 | [d-1](#d-1) |
| `--bs-modal-footer-border-color` | 1 | [c-border-color-1](#c-border-color-1) |
| `--bs-modal-footer-border-width` | 1 | [c-border-width-1](#c-border-width-1) |
| `--bs-modal-footer-gap` | 1 | [d-1](#d-1) |
| `--bs-modal-header-border-color` | 1 | [c-border-color-1](#c-border-color-1) |
| `--bs-modal-header-border-width` | 1 | [c-border-width-1](#c-border-width-1) |
| `--bs-modal-header-padding` | 1 | [c-spacer-6](#c-spacer-6) |
| `--bs-modal-header-padding-x` | 1 | [c-spacer-1](#c-spacer-1) |
| `--bs-modal-header-padding-y` | 1 | [c-spacer-1](#c-spacer-1) |
| `--bs-modal-inner-border-radius` | 1 | [c-border-radius-lg-4](#c-border-radius-lg-4) |
| `--bs-modal-margin` | 2 | [d-1](#d-1) |
| `--bs-modal-padding` | 1 | [c-spacer-1](#c-spacer-1) |
| `--bs-modal-title-line-height` | 1 | [c-line-height-base-1](#c-line-height-base-1) |
| `--bs-modal-width` | 5 | [d-1](#d-1) |
| `--bs-modal-zindex` | 1 | [c-zindex-modal-1](#c-zindex-modal-1) |

### Tooltip

12 properties, 12 contexts.

| Property | n | Audited in |
|---|---|---|
| `--bs-tooltip-arrow-height` | 1 | [d-1](#d-1) |
| `--bs-tooltip-arrow-width` | 1 | [d-1](#d-1) |
| `--bs-tooltip-bg` | 1 | [c-body-emphasis-color-1](#c-body-emphasis-color-1) |
| `--bs-tooltip-border-radius` | 1 | [c-border-radius-1](#c-border-radius-1) |
| `--bs-tooltip-color` | 1 | [c-body-bg-1](#c-body-bg-1) |
| `--bs-tooltip-font-size` | 1 | [c-font-size-base-2](#c-font-size-base-2) |
| `--bs-tooltip-margin` | 1 | [d-1](#d-1) |
| `--bs-tooltip-max-width` | 1 | [d-1](#d-1) |
| `--bs-tooltip-opacity` | 1 | [c-tooltip-opacity-1](#c-tooltip-opacity-1) |
| `--bs-tooltip-padding-x` | 1 | [c-spacer-2](#c-spacer-2) |
| `--bs-tooltip-padding-y` | 1 | [c-spacer-3](#c-spacer-3) |
| `--bs-tooltip-zindex` | 1 | [c-zindex-tooltip-1](#c-zindex-tooltip-1) |

### Popover

20 properties, 20 contexts.

| Property | n | Audited in |
|---|---|---|
| `--bs-popover-arrow-border` | 1 | [c-border-color-translucent-3](#c-border-color-translucent-3) |
| `--bs-popover-arrow-height` | 1 | [d-1](#d-1) |
| `--bs-popover-arrow-width` | 1 | [d-1](#d-1) |
| `--bs-popover-bg` | 1 | [c-body-bg-1](#c-body-bg-1) |
| `--bs-popover-body-color` | 1 | [c-body-color-1](#c-body-color-1) |
| `--bs-popover-body-padding-x` | 1 | [c-spacer-1](#c-spacer-1) |
| `--bs-popover-body-padding-y` | 1 | [c-spacer-1](#c-spacer-1) |
| `--bs-popover-border-color` | 1 | [c-border-color-translucent-1](#c-border-color-translucent-1) |
| `--bs-popover-border-radius` | 1 | [c-border-radius-lg-1](#c-border-radius-lg-1) |
| `--bs-popover-border-width` | 1 | [c-border-width-1](#c-border-width-1) |
| `--bs-popover-box-shadow` | 1 | [c-box-shadow-1](#c-box-shadow-1) |
| `--bs-popover-font-size` | 1 | [c-font-size-base-2](#c-font-size-base-2) |
| `--bs-popover-header-bg` | 1 | [c-body-secondary-bg-1](#c-body-secondary-bg-1) |
| `--bs-popover-header-color` | 1 | [c-headings-color-1](#c-headings-color-1) |
| `--bs-popover-header-font-size` | 1 | [c-font-size-base-1](#c-font-size-base-1) |
| `--bs-popover-header-padding-x` | 1 | [c-spacer-1](#c-spacer-1) |
| `--bs-popover-header-padding-y` | 1 | [d-1](#d-1) |
| `--bs-popover-inner-border-radius` | 1 | [c-border-radius-lg-3](#c-border-radius-lg-3) |
| `--bs-popover-max-width` | 1 | [d-1](#d-1) |
| `--bs-popover-zindex` | 1 | [c-zindex-popover-1](#c-zindex-popover-1) |

### Carousel

3 properties, 9 contexts.

| Property | n | Audited in |
|---|---|---|
| `--bs-carousel-caption-color` | 3 | [c-black-2](#c-black-2), [c-white-1](#c-white-1) |
| `--bs-carousel-control-icon-filter` | 3 | [d-1](#d-1) |
| `--bs-carousel-indicator-active-bg` | 3 | [c-black-2](#c-black-2), [c-white-1](#c-white-1) |

### Spinners

6 properties, 17 contexts.

| Property | n | Audited in |
|---|---|---|
| `--bs-spinner-animation-name` | 2 | [d-4](#d-4) |
| `--bs-spinner-animation-speed` | 3 | [d-1](#d-1) |
| `--bs-spinner-border-width` | 2 | [d-1](#d-1) |
| `--bs-spinner-height` | 4 | [c-spinner-width-1](#c-spinner-width-1), [c-spinner-width-sm-1](#c-spinner-width-sm-1) |
| `--bs-spinner-vertical-align` | 2 | [d-1](#d-1) |
| `--bs-spinner-width` | 4 | [c-spinner-width-1](#c-spinner-width-1), [c-spinner-width-sm-1](#c-spinner-width-sm-1) |

### Offcanvas

12 properties, 22 contexts.

| Property | n | Audited in |
|---|---|---|
| `--bs-offcanvas-bg` | 1 | [c-body-bg-1](#c-body-bg-1) |
| `--bs-offcanvas-border-color` | 1 | [c-border-color-translucent-1](#c-border-color-translucent-1) |
| `--bs-offcanvas-border-width` | 6 | [c-border-width-1](#c-border-width-1), [d-3](#d-3) |
| `--bs-offcanvas-box-shadow` | 1 | [c-box-shadow-sm-1](#c-box-shadow-sm-1) |
| `--bs-offcanvas-color` | 1 | [c-body-color-1](#c-body-color-1) |
| `--bs-offcanvas-height` | 6 | [d-1](#d-1), [d-4](#d-4) |
| `--bs-offcanvas-padding-x` | 1 | [c-spacer-1](#c-spacer-1) |
| `--bs-offcanvas-padding-y` | 1 | [c-spacer-1](#c-spacer-1) |
| `--bs-offcanvas-title-line-height` | 1 | [c-line-height-base-1](#c-line-height-base-1) |
| `--bs-offcanvas-transition` | 1 | [d-1](#d-1) |
| `--bs-offcanvas-width` | 1 | [d-1](#d-1) |
| `--bs-offcanvas-zindex` | 1 | [c-zindex-offcanvas-1](#c-zindex-offcanvas-1) |

### Helpers

1 properties, 4 contexts.

| Property | n | Audited in |
|---|---|---|
| `--bs-aspect-ratio` | 4 | [c-aspect-ratios-1x1-1](#c-aspect-ratios-1x1-1), [c-aspect-ratios-4x3-1](#c-aspect-ratios-4x3-1), [c-aspect-ratios-16x9-1](#c-aspect-ratios-16x9-1), [c-aspect-ratios-21x9-1](#c-aspect-ratios-21x9-1) |

### Utilities API

6 properties, 95 contexts.

| Property | n | Audited in |
|---|---|---|
| `--bs-bg-opacity` | 19 | [d-3](#d-3) |
| `--bs-border-opacity` | 15 | [d-3](#d-3) |
| `--bs-focus-ring-color` | 8 | [c-primary-26](#c-primary-26), [c-secondary-17](#c-secondary-17), [c-success-18](#c-success-18), [c-info-17](#c-info-17), [c-warning-17](#c-warning-17), [c-danger-18](#c-danger-18) (+2 more) |
| `--bs-link-opacity` | 10 | [d-3](#d-3) |
| `--bs-link-underline-opacity` | 21 | [d-3](#d-3) |
| `--bs-text-opacity` | 22 | [d-3](#d-3) |

## F. Opportunities for semantic theme tokens

Each point is based on the groups above. Counts are declaration contexts.

1. **One token per theme colour.** `$primary` alone anchors 90 contexts (62 properties), and each of the other theme colours anchors the same family. A theme system that sets the eight theme colours (and their light/dark counterparts) controls most colour output.
2. **Separate "same reference" from "same calculation".** Within a theme colour the subgroups in section C show which properties move together: the root derivatives (`-rgb`, `-text-emphasis`, `-bg-subtle`, `-border-subtle`), the button values, the table variant values and the link/focus values each use different calculations. A token layer should expose the base colour plus the individual amounts (shade/tint weights), not a single "primary" token.
3. **Compile-time vs runtime.** Button, outline-button, table-variant, and form-validation values are evaluated by Sass and are fixed in the CSS. To make them follow a runtime token they would have to be overridden explicitly; alert and list-group variants already follow runtime tokens.
4. **Semantic roles that already exist** and are good token candidates: `--bs-body-color/bg`, `--bs-emphasis-color`, `--bs-secondary-color/bg`, `--bs-tertiary-color/bg`, `--bs-border-color`, `--bs-link-color/hover`, `--bs-focus-ring-*`, each with a dark-mode redeclaration.
5. **Scales.** `$spacer`/`$spacers`, the radius variables (`--bs-border-radius*`), `--bs-border-width`, font-size variables and `--bs-font-*` are shared scale roots used across many components.
6. **Not controllable through shared tokens:** the hardcoded and single-use values in section D.

