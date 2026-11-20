# Theme Token Architecture

# Category 1: Colour & Surfaces

#### A. Core Theme Variables

Foundational values a user configures to define their brand and light/dark surfaces.

| Variable ID / Name | Design Category | Classification | Purpose | Default / Derivation Rule | Override Behaviour | Bootstrap Mappings |
| --- | --- | --- | --- | --- | --- | --- |
| `--ds-color-primary` | Colour | Core | Main brand/action colour | User-configured (e.g., `#0d6efd`) | Primary user input | `--bs-primary`, `.btn-primary` fill/border, active accents |
| `--ds-color-secondary` | Colour | Core | Secondary supporting UI colour | User-configured (e.g., `#6c757d`) | Primary user input | `--bs-secondary`, `.btn-secondary` fill/border |
| `--ds-color-success` | Colour | Core | Positive / valid / success state colour | User-configured (e.g., `#198754`) | Primary user input | `--bs-success`, `--bs-form-valid-color`, `.valid-tooltip` |
| `--ds-color-info` | Colour | Core | Informational state colour | User-configured (e.g., `#0dcaf0`) | Primary user input | `--bs-info` |
| `--ds-color-warning` | Colour | Core | Warning / caution state colour | User-configured (e.g., `#ffc107`) | Primary user input | `--bs-warning` |
| `--ds-color-danger` | Colour | Core | Error / invalid / danger state colour | User-configured (e.g., `#dc3545`) | Primary user input | `--bs-danger`, `--bs-form-invalid-color`, `.invalid-tooltip` |
| `--ds-bg-base` | Surfaces | Core | Main page / container background colour | User-configured per mode (e.g., `#ffffff` light / `#212529` dark) | Primary user input | `--bs-body-bg`, `body`, `.form-control`, `.form-select`, `.img-thumbnail` |
| `--ds-bg-subtle` | Surfaces | Core | Muted/secondary background surface | User-configured per mode | Primary user input | `--bs-secondary-bg`, disabled/hover file buttons, range tracks |
| `--ds-bg-elevated` | Surfaces | Core | Raised/tertiary surface for addons or chrome | User-configured per mode | Primary user input | `--bs-tertiary-bg`, `.input-group-text`, file upload buttons |
| `--ds-text-base` | Typography / Colour | Core | Default body text colour | User-configured per mode (e.g., `#212529` light / `#dee2e6` dark) | Primary user input | `--bs-body-color`, `body`, form text/labels/addons, `kbd` bg |
| `--ds-text-muted` | Typography / Colour | Core | Secondary/muted text, captions, placeholders, helper text | User-configured per mode | Primary user input | `--bs-secondary-color`, `caption`, `.figure-caption`, `.form-text`, `::placeholder` |
| `--ds-text-heading` | Typography / Colour | Core | Heading text colour | User-configured per mode (defaults to `--ds-text-base`) | Primary user input | `--bs-heading-color`, `h1`–`h6` / `.h1`–`.h6` |

#### B. Computed Theme Variables

Automatically derived at runtime from core variables so users don't have to configure every state or shade by hand.

| Variable ID / Name | Design Category | Classification | Purpose | Default / Derivation Rule | Override Behaviour | Bootstrap Mappings |
| --- | --- | --- | --- | --- | --- | --- |
| `--ds-<role>-rgb` *(e.g., `--ds-color-primary-rgb`, `--ds-bg-base-rgb`, `--ds-text-base-rgb`)* | Colour / Surfaces | Computed | RGB channels for alpha/opacity utilities | Derived from matching `--ds-*` colour token | Computed automatically | `--bs-primary-rgb`, `--bs-body-bg-rgb`, `--bs-body-color-rgb`, `--bs-link-color-rgb`, `.bg-*`, `.text-bg-*`, `.text-*` |
| `--ds-<role>-hover` | Colour | Computed | Hover/focus background or fill for interactive roles | Derived from `--ds-<role>` (e.g., darkened/lightened mix for mode) | Computed automatically | `--bs-btn-hover-bg`, `--bs-dropdown-link-hover-bg`, `.page-link:hover`, `.list-group-item-action:hover` |
| `--ds-<role>-active` | Colour | Computed | Active/pressed background or fill for interactive roles | Deeper shift than `--ds-<role>-hover` | Computed automatically | `--bs-btn-active-bg`, `--bs-dropdown-link-active-bg`, `.page-link.active`, `.list-group-item-action:active` |
| `--ds-<role>-disabled` | Colour | Computed | Disabled state background/fill | Attenuated/mixed role or `--ds-bg-subtle` | Computed automatically | `--bs-btn-disabled-bg`, `.page-link.disabled`, `.list-group-item.disabled` |
| `--ds-<role>-subtle` | Surfaces | Computed | Soft tinted surface for subtle badges/alerts/pills | Mode-aware blend of `--ds-<role>` and `--ds-bg-base` | Computed automatically | `--bs-<role>-bg-subtle`, `.bg-<role>-subtle` |
| `--ds-on-<role>` *(e.g., `--ds-on-primary`, `--ds-on-danger`)* | Colour (Text) | Computed | Readable text colour placed on top of a solid role background | High-contrast light or dark text chosen for `--ds-<role>` | Computed automatically | `--bs-btn-color`, `.btn-<role>` text, `.text-bg-<role>` text |
| `--ds-text-emphasis` / `--ds-text-emphasis-rgb` | Typography / Colour | Computed | High-emphasis body/link text colour | Stronger contrast version of `--ds-text-base` | Computed automatically | `--bs-emphasis-color`, `--bs-emphasis-color-rgb`, `.text-body-emphasis`, `.link-body-emphasis` |
| `--ds-text-tertiary` | Typography / Colour | Computed | Low-emphasis/tertiary text colour | Softer contrast step between `--ds-text-muted` and `--ds-bg-base` | Computed automatically | `--bs-tertiary-color`, `.text-body-tertiary` |
| `--ds-<role>-text-emphasis` | Typography / Colour | Computed | High-contrast semantic text for subtle/light backgrounds | Mode-aware contrast-adjusted semantic colour | Computed automatically | `--bs-<role>-text-emphasis`, `.text-<role>-emphasis` |
| `--ds-<role>-hover-rgb` | Colour | Computed | RGB channels for coloured link hover/focus states | Derived from `--ds-<role>-hover` | Computed automatically | Manual override for `.link-<role>:hover, .link-<role>:focus` |
| `--ds-bg-overlay` | Surfaces | Computed | Background for modals, popovers, toasts, dropdowns, offcanvas | Derived from `--ds-bg-base` / `--ds-bg-elevated` per mode | Computed automatically | `--bs-modal-bg`, `--bs-popover-bg`, `--bs-toast-bg`, `--bs-dropdown-bg`, `--bs-offcanvas-bg` |
| `--ds-bg-sunken` | Surfaces | Computed | Subtle background for card/modal/popover/toast headers & footers | Derived from `--ds-bg-subtle` / `--ds-bg-base` | Computed automatically | `--bs-card-cap-bg`, `--bs-modal-footer-bg`, `--bs-popover-header-bg`, `--bs-toast-header-bg` |
| `--ds-backdrop-bg` | Surfaces | Computed | Backdrop overlay behind modals and offcanvas panels | Translucent dark backdrop (e.g., `rgb(0 0 0 / 0.5)`) | Computed automatically | `--bs-backdrop-bg`, `.offcanvas-backdrop` (via manual override) |

#### C. Overridable Derived Variables

Derived by default, with an explicit hook if a user wants component-specific control.

| Variable ID / Name | Design Category | Classification | Purpose | Default / Derivation Rule | Override Behaviour | Bootstrap Mappings |
| --- | --- | --- | --- | --- | --- | --- |
| `--ds-link-color` | Colour | Overridable Derived | Default link colour | Defaults to `--ds-color-primary` | User-overridable | `--bs-link-color`, `--bs-link-color-rgb`, `a` |
| `--ds-highlight-bg` / `--ds-highlight-text` | Colour / Surfaces | Overridable Derived | Highlighted text (`<mark>` / `.mark`) background and text | Derived from `--ds-color-warning` / `--ds-text-base` | User-overridable | `--bs-highlight-bg`, `--bs-highlight-color`, `mark, .mark` |
| `--ds-code-text` | Colour | Overridable Derived | Inline `<code>` text colour | Derived from accent/semantic role | User-overridable | `--bs-code-color`, `code` |
| `--ds-card-bg` / `--ds-card-cap-bg` | Surfaces | Overridable Derived | Card container and card header/footer backgrounds | Derived from `--ds-bg-base` / `--ds-bg-sunken` | User-overridable | `--bs-card-bg`, `--bs-card-cap-bg`, `.card`, `.card-header`, `.card-footer` |
| `--ds-popover-bg` / `--ds-tooltip-bg` | Surfaces | Overridable Derived | Popover and tooltip backgrounds | Derived from `--ds-bg-overlay` / contrast fill | User-overridable | `--bs-popover-bg`, `--bs-popover-header-bg`, `--bs-tooltip-bg` |
| `--ds-progress-bg` / `--ds-progress-bar-bg` | Surfaces / Colour | Overridable Derived | Progress track and progress bar fill | Derived from `--ds-bg-subtle` / `--ds-color-primary` | User-overridable | `--bs-progress-bg`, `--bs-progress-bar-bg` |

### Manual Overrides Needed for This Slice

- **Hardcoded primary/accent colours:** `.form-check-input:checked`, `.form-check-input[type=checkbox]:indeterminate`, and `.form-range` slider thumbs (including `:active`) hardcode `#0d6efd` / `#b6d4fe`.
- **Hardcoded link hover RGBs:** `.link-<role>:hover, .link-<role>:focus` hardcode static RGB values instead of referencing variables.
- **Hardcoded foreground/backdrop literals:** `.blockquote-footer`, `.form-floating > :disabled ~ label`, `.valid-tooltip`, `.invalid-tooltip`, `.carousel-control-prev/next`, `.offcanvas-backdrop`, `.text-black-50`, `.text-white-50`, and `.progress-bar-striped`.
- **`.text-bg-<role>` contrast:** Hardcodes `#fff !important` or `#000 !important` for text colour; override with `!important` using `-ds-on-<role>` if needed.
- **SVG data URIs:** Embedded `data:image/svg+xml` backgrounds (validation icons, carousel arrows, select/checkbox/switch/accordion/navbar-toggler icons) don't recolour automatically from `-ds-*` variables and need per-theme SVG/background overrides.

# Category 2: Borders, Radii & Outlines Specification

#### A. Core Theme Variables

| **Variable ID / Name** | **Design Category** | **Classification** | **Purpose** | **Default / Derivation Rule** | **Override Behaviour** | **Bootstrap Mappings** |
| --- | --- | --- | --- | --- | --- | --- |
| `--ds-border-width` | Borders | Core | Default structural border thickness | User-configured (e.g., `1px`) | Primary user input | `--bs-border-width`, `.border`, `.form-control`, `.card`, `.list-group-item`, `.table` cell borders |
| `--ds-border-style` | Borders | Core | Default border line style | User-configured (e.g., `solid`) | Primary user input | `--bs-border-style`, `.border` utility |
| `--ds-border-color` | Borders | Core | Default neutral border colour | User-configured per mode (e.g., `#dee2e6` light / `#495057` dark) | Primary user input | `--bs-border-color`, `.border`, `.form-control`, `.card`, `.modal-content`, `.dropdown-menu` |
| `--ds-radius-base` | Radii | Core | Base corner radius for standard controls and surfaces | User-configured (e.g., `0.375rem` or `0px` for sharp themes) | Primary user input | `--bs-border-radius`, `.rounded`, `.rounded-2`, `.form-control`, `.input-group-text`, `.btn-group` |
| `--ds-radius-pill` | Radii | Core | Full pill/capsule corner radius | User-configured (e.g., `50rem` or `9999px`) | Primary user input | `--bs-border-radius-pill`, `.rounded-pill` |
| `--ds-outline-width` | Outlines | Core | Default focus outline thickness when outlines are enabled | User-configured (e.g., `2px` or `0px`) | Primary user input | Manual focus-outline overrides |
| `--ds-outline-offset` | Outlines | Core | Default offset between element edge and focus outline | User-configured (e.g., `2px`) | Primary user input | Manual focus-outline overrides (`[type=search]` has `-2px` offset) |

#### B. Computed Theme Variables

| **Variable ID / Name** | **Design Category** | **Classification** | **Purpose** | **Default / Derivation Rule** | **Override Behaviour** | **Bootstrap Mappings** |
| --- | --- | --- | --- | --- | --- | --- |
| `--ds-border-color-rgb` | Borders | Computed | RGB channels for alpha border utilities | Derived from `--ds-border-color` | Computed automatically | `--bs-border-color-rgb` |
| `--ds-<role>-border-subtle` | Borders | Computed | Soft tinted border colour for semantic states | Mode-aware blend of `--ds-color-<role>` and `--ds-bg-base` / `--ds-border-color` | Computed automatically | `--bs-<role>-border-subtle`, `.border-<role>-subtle` |
| `--ds-radius-sm` | Radii | Computed | Smaller corner radius for compact controls | Derived from `--ds-radius-base` (e.g., `calc(var(--ds-radius-base) * 0.67)` or `0.25rem` when `0`) | Computed automatically | `--bs-border-radius-sm`, `.form-control-sm`, `.form-select-sm`, `.rounded-1` |
| `--ds-radius-lg` | Radii | Computed | Larger corner radius for prominent containers | Derived from `--ds-radius-base` (e.g., `calc(var(--ds-radius-base) * 1.33)`) | Computed automatically | `--bs-border-radius-lg`, `.form-control-lg`, `.form-select-lg`, `.rounded-3` |
| `--ds-radius-xl` | Radii | Computed | Extra-large corner radius | Derived from `--ds-radius-base` (e.g., `calc(var(--ds-radius-base) * 2.67)`) | Computed automatically | `--bs-border-radius-xl`, `.rounded-4` |
| `--ds-radius-xxl` | Radii | Computed | 2x-extra-large corner radius | Derived from `--ds-radius-base` (e.g., `calc(var(--ds-radius-base) * 5.33)`) | Computed automatically | `--bs-border-radius-xxl`, `.rounded-5` |
| `--ds-<component>-inner-radius` | Radii | Computed | Nested inner radius inside bordered containers | Derived as `max(0px, calc(var(--ds-<component>-radius) - var(--ds-<component>-border-width)))` | Computed automatically | `--bs-card-inner-border-radius`, `--bs-modal-inner-border-radius`, `--bs-popover-inner-border-radius`, `--bs-accordion-inner-border-radius`, `.toast-header` |
| `--ds-focus-ring-color` | Outlines / Colour | Computed | Focus border/outline colour derived from primary brand colour | Derived from `--ds-color-primary` (or softened/tinted variant) | Computed automatically | Manual focus border/outline overrides (replaces `#86b7fe`) |

#### C. Overridable Derived Variables

| **Variable ID / Name** | **Design Category** | **Classification** | **Purpose** | **Default / Derivation Rule** | **Override Behaviour** | **Bootstrap Mappings** |
| --- | --- | --- | --- | --- | --- | --- |
| `--ds-btn-radius` | Radii | Overridable Derived | Button corner radius | Defaults to `--ds-radius-base` | User-overridable | `--bs-btn-border-radius`, `.btn` |
| `--ds-card-radius` | Radii | Overridable Derived | Card corner radius | Defaults to `--ds-radius-base` | User-overridable | `--bs-card-border-radius`, `.card` |
| `--ds-modal-radius` | Radii | Overridable Derived | Modal dialog corner radius | Defaults to `--ds-radius-base` | User-overridable | `--bs-modal-border-radius`, `.modal-content` |
| `--ds-dropdown-radius` | Radii | Overridable Derived | Dropdown menu corner radius | Defaults to `--ds-radius-base` | User-overridable | `--bs-dropdown-border-radius`, `.dropdown-menu` |
| `--ds-dropdown-item-radius` | Radii | Overridable Derived | Dropdown item corner radius | Defaults to `0px` (or `--ds-radius-sm`) | User-overridable | `--bs-dropdown-item-border-radius` |
| `--ds-tooltip-radius` / `--ds-popover-radius` | Radii | Overridable Derived | Tooltip and popover corner radii | Defaults to `--ds-radius-sm` / `--ds-radius-base` | User-overridable | `--bs-tooltip-border-radius`, `--bs-popover-border-radius` |
| `--ds-card-border-color` / `--ds-card-border-width` | Borders | Overridable Derived | Card border colour and width | Defaults to `--ds-border-color` / `--ds-border-width` | User-overridable | `--bs-card-border-color`, `--bs-card-border-width` |
| `--ds-table-border-color` | Borders | Overridable Derived | Table grid border colour | Defaults to `--ds-border-color` | User-overridable | `--bs-table-border-color`, `.table` |
| `--ds-valid-border-color` / `--ds-invalid-border-color` | Borders | Overridable Derived | Form validation border colours | Defaults to `--ds-color-success` / `--ds-color-danger` | User-overridable | `--bs-form-valid-border-color`, `--bs-form-invalid-border-color` |

### 3. Manual Overrides Needed for This Slice

- **Hardcoded focus & checked border colours:** `.form-control:focus`, `.form-select:focus`, and `.form-check-input:focus` hardcode `border-color: #86b7fe`; `.form-check-input:checked` and `[type=checkbox]:indeterminate` hardcode `border-color: #0d6efd`.
- **Hardcoded component radii:** `kbd`, `.form-check-input[type=checkbox]`, `.form-switch .form-check-input`, `.form-range` thumb/track, and `.btn-close` hardcode literal radii (`0.25rem`, `0.25em`, `2em`, `1rem`, `0.375rem`) rather than reading `-bs-*` radius variables.
- **Outline resets (`outline: 0`):** Bootstrap explicitly zeroes `outline` across `:focus` / `:focus-visible` states on controls, buttons, navs, accordions, pagination, modals, and offcanvas—so any runtime outline token requires a manual selector override.
- **Layout/structural borders to leave alone (or handle with care):** Caret triangles (`.dropdown-toggle::after`, `.dropup`, `.dropend`, `.dropstart`), `.carousel-indicators [data-bs-target]`, `.spinner-border` (`border-right-color: transparent`), and grouped item corner resets (`.input-group`, `.btn-group`, `.card-group`, `.list-group`) use borders/radii structurally to draw shapes or join grouped elements.

Ready for the next audit section whenever you want to paste it—**Typography**, **Spacing**, or **Shadows & Effects**?

# Category 3: Generated Content & Decoration Specification

#### A. Core Theme Variables

*None required in this category.* Generated content and list resets do not represent foundational brand or design-system decisions.

#### B. Computed Theme Variables

*None required in this category.*

#### C. Overridable Derived Variables

| **Variable ID / Name** | **Design Category** | **Classification** | **Purpose** | **Default / Derivation Rule** | **Override Behaviour** | **Bootstrap Mappings** |
| --- | --- | --- | --- | --- | --- | --- |
| `--ds-breadcrumb-divider` | Generated Content / Typography | Overridable Derived | Separator character/string between breadcrumb items | Defaults to `"/"` (or mode/locale-appropriate string) | User-overridable | `--bs-breadcrumb-divider` |
| `--ds-blockquote-prefix` *(optional)* | Generated Content / Typography | Overridable Derived | Prefix string before blockquote footer attribution | Defaults to `"— "` | User-overridable (via manual override) | Manual override on `.blockquote-footer::before` |

### 3. Manual Overrides & Exclusions for This Slice

- **Manual override needed for `-ds-blockquote-prefix` (if used):** `.blockquote-footer::before` hardcodes `"— "` with no `var()`, so overriding it requires a manual rule setting `content: var(--ds-blockquote-prefix, "— ")`.
- **Leave alone (structural / functional):** All `""` pseudo-elements (`.dropdown-toggle::after`, `.dropup`/`.dropend`/`.dropstart`, `.accordion-button::after`, `.tooltip-arrow`, `.popover-arrow`, `.carousel-inner::after`, `.placeholder.btn::before`, `.clearfix::after`, `.ratio::before`, `.stretched-link::after`), `.list-group-numbered > .list-group-item::before` (`counters(...)`), and all `list-style` / `list-style-type` resets.

Ready for the next category whenever you want to paste it!

# Category 4: Motion & Transforms Specification

#### A. Core Theme Variables

| **Variable ID / Name** | **Design Category** | **Classification** | **Purpose** | **Default / Derivation Rule** | **Override Behaviour** | **Bootstrap Mappings** |
| --- | --- | --- | --- | --- | --- | --- |
| `--ds-duration-fast` | Motion | Core | Short duration for micro-interactions (hover, focus, toggles, floating labels) | User-configured (e.g., `0.15s`) | Primary user input | Manual transition overrides for `.form-control`, `.form-select`, `.form-switch`, `.form-range`, `.btn`, `.nav-link`, `.page-link`, `.fade`, `.carousel-control-*` |
| `--ds-duration-base` | Motion | Core | Standard duration for panels, collapses, and modals | User-configured (e.g., `0.3s`) | Primary user input | Manual transition overrides for `.collapsing`, `.modal.fade .modal-dialog`, `--bs-offcanvas-transition` |
| `--ds-duration-slow` | Motion | Core | Longer duration for carousels and broader transitions | User-configured (e.g., `0.6s`) | Primary user input | Manual transition overrides for `.carousel-item`, `.carousel-fade`, `.carousel-indicators` |
| `--ds-ease-standard` | Motion | Core | Default easing curve for UI state transitions | User-configured (e.g., `ease-in-out`) | Primary user input | Manual transition easing overrides across controls/components |
| `--ds-ease-out` | Motion | Core | Decelerating easing curve for entering/opening elements | User-configured (e.g., `ease-out`) | Primary user input | Manual transition overrides for opening panels/modals/drawers |

#### B. Computed Theme Variables

| **Variable ID / Name** | **Design Category** | **Classification** | **Purpose** | **Default / Derivation Rule** | **Override Behaviour** | **Bootstrap Mappings** |
| --- | --- | --- | --- | --- | --- | --- |
| `--ds-transition-control` | Motion | Computed | Standard transition shorthand for interactive controls & buttons | Derived from `--ds-duration-fast` and `--ds-ease-standard` (e.g., `color var(--ds-duration-fast) var(--ds-ease-standard), background-color var(--ds-duration-fast) var(--ds-ease-standard), border-color var(--ds-duration-fast) var(--ds-ease-standard), box-shadow var(--ds-duration-fast) var(--ds-ease-standard)`) | Computed automatically | Manual override on `.btn`, `.form-control`, `.form-select`, `.page-link`, `.nav-link`, file buttons, range thumbs |
| `--ds-transition-panel` | Motion | Computed | Transition shorthand for collapsible/drawer/modal panels | Derived from `--ds-duration-base` and `--ds-ease-out` / `--ds-ease-standard` | Computed automatically | `--bs-offcanvas-transition`, manual override on `.collapsing`, `.modal.fade .modal-dialog` |

#### C. Overridable Derived Variables

| **Variable ID / Name** | **Design Category** | **Classification** | **Purpose** | **Default / Derivation Rule** | **Override Behaviour** | **Bootstrap Mappings** |
| --- | --- | --- | --- | --- | --- | --- |
| `--ds-accordion-transition` | Motion | Overridable Derived | Accordion header/collapse transition | Derived from `--ds-transition-panel` / `--ds-duration-base` | User-overridable | `--bs-accordion-transition` |
| `--ds-accordion-icon-transform` | Transforms | Overridable Derived | Transform applied to accordion indicator when expanded | Defaults to `rotate(-180deg)` (matching Bootstrap's open chevron rotation) | User-overridable | `--bs-accordion-btn-icon-transform` |
| `--ds-icon-link-transform` | Transforms | Overridable Derived | Hover/focus offset transform for `.icon-link-hover` icons | Defaults to `translate3d(0.25em, 0, 0)` | User-overridable | `--bs-icon-link-transform` |
| `--ds-progress-bar-transition` | Motion | Overridable Derived | Progress bar width/fill transition | Derived from `--ds-duration-slow` / `--ds-ease-standard` | User-overridable | `--bs-progress-bar-transition` |

### 3. Manual Overrides & Exclusions for This Slice

- **Manual transition overrides:** Because Bootstrap hardcodes durations/easings on `.btn`, `.form-control`, `.form-select`, `.form-switch`, `.form-range`, `.form-floating > label`, `.nav-link`, `.page-link`, `.fade`, `.collapsing`, `.modal.fade .modal-dialog`, and `.carousel-*`, mapping `-ds-duration-*` and `-ds-ease-*` to those selectors requires manual rules—and any non-reduced-motion rule should be scoped safely so `@media (prefers-reduced-motion: reduce)` still disables motion (`none`).
- **Leave alone (functional / structural):** Offcanvas slide transforms (`±100%`), carousel slide transforms (`±100%`), `.translate-middle*` (`!important`), floating-label positioning (`scale(0.85) translateY(...)` + `transform-origin: 0 0`), spinner keyframes, and striped/placeholder loop animations (`.progress-bar-animated`, `.placeholder-glow`, `.placeholder-wave`).

Ready for the next audit section whenever you want to paste it!

# Category 5: Shadows, Opacity & Filters Specification

#### A. Core Theme Variables

| Variable ID / Name | Design Category | Classification | Purpose | Default / Derivation Rule | Override Behaviour | Bootstrap Mappings |
| --- | --- | --- | --- | --- | --- | --- |
| `--ds-shadow-base` | Shadows | Core | Default elevation shadow for floating/raised surfaces | User-configured per mode (e.g., `0 0.5rem 1rem rgba(0, 0, 0, 0.15)`) | Primary user input | `--bs-box-shadow`, `.shadow`, `--bs-toast-box-shadow` |
| `--ds-focus-ring-width` | Shadows / Focus | Core | Thickness of focus ring / halo | User-configured (e.g., `0.25rem`) | Primary user input | Manual focus-ring overrides; `--bs-focus-ring-width` (where used) |
| `--ds-focus-ring-alpha` | Shadows / Focus | Core | Opacity of focus ring colour | User-configured (e.g., `0.25`) | Primary user input | Manual focus-ring overrides (replaces hardcoded `0.25` alpha) |
| `--ds-disabled-opacity` | Opacity | Core | Default opacity for disabled controls/states | User-configured (e.g., `0.5` or `0.65`) | Primary user input | `--bs-btn-disabled-opacity`, manual overrides on `.form-check-input:disabled` / label |

#### B. Computed Theme Variables

| Variable ID / Name | Design Category | Classification | Purpose | Default / Derivation Rule | Override Behaviour | Bootstrap Mappings |
| --- | --- | --- | --- | --- | --- | --- |
| `--ds-shadow-sm` | Shadows | Computed | Subtle elevation shadow for small/compact elements | Derived from `--ds-shadow-base` (e.g., scaled down in offset/blur/spread) | Computed automatically | `--bs-box-shadow-sm`, `.shadow-sm` |
| `--ds-shadow-lg` | Shadows | Computed | Prominent elevation shadow for dialogs/overlays | Derived from `--ds-shadow-base` (e.g., scaled up in offset/blur/spread) | Computed automatically | `--bs-box-shadow-lg`, `.shadow-lg` |
| `--ds-focus-ring-shadow` | Shadows / Focus | Computed | Standard focus halo shadow using primary colour RGB | Derived from `--ds-focus-ring-width`, `--ds-color-primary-rgb`, and `--ds-focus-ring-alpha` | Computed automatically | Manual override on `.form-control:focus`, `.form-select:focus`, `.form-check-input:focus`, `.nav-link:focus-visible` |

#### C. Overridable Derived Variables

| Variable ID / Name | Design Category | Classification | Purpose | Default / Derivation Rule | Override Behaviour | Bootstrap Mappings |
| --- | --- | --- | --- | --- | --- | --- |
| `--ds-backdrop-opacity` | Opacity | Overridable Derived | Opacity of modal/offcanvas backdrop scrim when shown | Defaults to `0.5` | User-overridable | `--bs-backdrop-opacity`, manual override on `.offcanvas-backdrop.show` |
| `--ds-tooltip-opacity` | Opacity | Overridable Derived | Opacity of shown tooltips | Defaults to `0.9` (Bootstrap default) | User-overridable | `--bs-tooltip-opacity` |
| `--ds-btn-close-opacity` / `--ds-btn-close-hover-opacity` | Opacity | Overridable Derived | Close button rest and hover/focus opacity | Defaults to `0.5` at rest, `0.75` on hover/focus | User-overridable | `--bs-btn-close-opacity`, `--bs-btn-close-hover-opacity`, `--bs-btn-close-focus-opacity`, `--bs-btn-close-disabled-opacity` |
| `--ds-btn-close-filter` / `--ds-carousel-control-filter` | Filters | Overridable Derived | Filter treatment for close and carousel control icons | Mode-aware filter (e.g., invert/brightness in dark mode if needed) | User-overridable | `--bs-btn-close-filter`, `--bs-carousel-control-icon-filter` |
| `--ds-placeholder-opacity` | Opacity | Overridable Derived | Base opacity of `.placeholder` skeleton elements | Defaults to `0.5` | User-overridable | Manual override on `.placeholder` |
| `--ds-divider-opacity` | Opacity | Overridable Derived | Opacity of `<hr>` and `.vr` dividers | Defaults to `0.25` | User-overridable | Manual override on `hr`, `.vr` |

### 3. Manual Overrides & Exclusions for This Slice

- **Hardcoded primary focus rings:** `.form-control:focus`, `.form-select:focus`, `.form-check-input:focus`, and `.nav-link:focus-visible` hardcode `rgba(13, 110, 253, 0.25)`, and `.form-range:focus` thumbs hardcode `#fff` + `rgba(13, 110, 253, 0.25)`—override with `-ds-focus-ring-shadow` (and `-ds-bg-base` for the range ring separator).
    - **Hardcoded state opacities & filters:** `.form-check-input:disabled` (`0.5`), `.form-check-input:disabled ~ .form-check-label` (`0.5`), `.offcanvas-backdrop.show` (`0.5`), `.placeholder` (`0.5`), `hr` / `.vr` (`0.25`), `.carousel-control-*` (`0.5` / `0.9`), and `.form-check-input:active` (`brightness(90%)`).
    - **Leave alone (functional / structural):** Show/hide `opacity: 0` / `1` states (`.fade:not(.show)`, `.toast.showing`, `.tooltip`, `.modal-backdrop.fade`, `.carousel-fade *`, `.spinner-grow`, `.offcanvas-backdrop.fade`), keyframe opacities, `.opacity-*` utilities (`!important`), `.shadow-none` (`!important`), and inset `box-shadow` rules used for table cell fills or accordion hairlines.

# Category 6: Sizing

### A. Core Theme Variables

*None required.* Sizing in Bootstrap is predominantly layout/component geometry rather than a foundational design decision.

### B. Computed Theme Variables

*None required.*

### C. Overridable Derived Variables

| **Variable ID / Name** | **Design Category** | **Classification** | **Purpose** | **Default / Derivation Rule** | **Override Behaviour** | **Bootstrap Mappings** |
| --- | --- | --- | --- | --- | --- | --- |
| `--ds-modal-max-width` | Sizing | Overridable Derived | Default modal dialog width on `sm+` viewports | Defaults to `500px` (Bootstrap's `--bs-modal-width`) | User-overridable | `--bs-modal-width` |
| `--ds-popover-max-width` | Sizing | Overridable Derived | Maximum width of popover containers | Defaults to `276px` (Bootstrap's `--bs-popover-max-width`) | User-overridable | `--bs-popover-max-width` |
| `--ds-tooltip-max-width` | Sizing | Overridable Derived | Maximum width of tooltip text containers | Defaults to `200px` (Bootstrap's `--bs-tooltip-max-width`) | User-overridable | `--bs-tooltip-max-width` |
| `--ds-progress-height` | Sizing | Overridable Derived | Height of progress bars and stacked progress tracks | Defaults to `1rem` (Bootstrap's `--bs-progress-height`) | User-overridable | `--bs-progress-height` |
| `--ds-offcanvas-height` | Sizing | Overridable Derived | Height of top/bottom offcanvas drawers | Defaults to `30vh` (Bootstrap's `--bs-offcanvas-height`) | User-overridable | `--bs-offcanvas-height` |
| `--ds-offcanvas-width` | Sizing | Overridable Derived | Width of left/right (`start`/`end`) offcanvas drawers | Defaults to `400px` (Bootstrap's `--bs-offcanvas-width`) | User-overridable | `--bs-offcanvas-width` |
| `--ds-dropdown-min-width` | Sizing | Overridable Derived | Minimum width of dropdown menus | Defaults to `10rem` (Bootstrap's `--bs-dropdown-min-width`) | User-overridable | `--bs-dropdown-min-width` |
| `--ds-toast-max-width` | Sizing | Overridable Derived | Maximum width of toast notifications | Defaults to `350px` (Bootstrap's `--bs-toast-max-width`) | User-overridable | `--bs-toast-max-width` |
| `--ds-nav-scroll-max-height` | Sizing | Overridable Derived | Max height for scrollable navbar navs | Defaults to `75vh` | User-overridable | `--bs-scroll-height` |

## 3. Manual Overrides & Exclusions for Sizing

- **Optional manual overrides (if intentionally altering control proportions):** Selectors without `var()` such as `.form-control-color` (`3rem`), `.form-range` / thumb / track (`1.5rem` / `1rem` / `0.5rem`), `.form-switch .form-check-input` (`2em`), `.navbar-toggler-icon` (`1.5em`), `.btn-close` (`1em`), and `.carousel-indicators [data-bs-target]` (`30px` × `3px`).
- **Leave alone (layout & structural):** Containers, grid columns/rows, `.row > *` resets, `.input-group` flex `width: 1%`, calculated form/modal heights, arrow boxes, spinners, accordion icon width, `.vr` width, `.ratio`, `.visually-hidden`, `.collapsing`, and all `.w-*` / `.h-*` / `.mw-*` / `.mh-*` / `.vw-*` / `.vh-*` (`!important`) utilities.

Ready for the next category whenever you drop it!

# Category 7: Typography & Text

#### A. Core Theme Variables

| Variable ID / Name | Design Category | Classification | Purpose | Default / Derivation Rule | Override Behaviour | Bootstrap Mappings |
| --- | --- | --- | --- | --- | --- | --- |
| `--ds-font-sans` | Typography | Core | Primary sans-serif font stack | User-configured (e.g., system sans stack) | Primary user input | `--bs-font-sans-serif`, `--bs-body-font-family`, `.tooltip`, `.popover` |
| `--ds-font-mono` | Typography | Core | Monospace font stack for code, keyboard keys, and technical text | User-configured (e.g., system mono stack) | Primary user input | `--bs-font-monospace`, `pre, code, kbd, samp`, `.font-monospace` |
| `--ds-font-size-base` | Typography | Core | Base root/body font size | User-configured (e.g., `1rem`) | Primary user input | `--bs-body-font-size`, `body`, `.form-control`, `.form-select`, `.input-group-text`, `.accordion-button` |
| `--ds-font-weight-base` | Typography | Core | Default body/control font weight | User-configured (e.g., `400`) | Primary user input | `--bs-body-font-weight`, `.form-control`, `.form-select`, `.input-group-text`, `.dropdown-item`, `.tooltip`, `.popover` |
| `--ds-font-weight-heading` | Typography | Core | Default weight for headings (`h1`–`h6` / `.h1`–`.h6`) | User-configured (e.g., `500`) | Primary user input | Manual override on `h1`–`h6, .h1`–`.h6` |
| `--ds-font-weight-display` | Typography | Core | Font weight for display headings and lead text | User-configured (e.g., `300`) | Primary user input | Manual override on `.display-1`–`.display-6`, `.lead` |
| `--ds-font-weight-bold` | Typography | Core | Strong/emphasis weight for active navs, alerts, and definition terms | User-configured (e.g., `700`) | Primary user input | Manual override on `dt`, `.nav-underline .nav-link.active`, `.alert-link`, `.fw-bold` |
| `--ds-line-height-base` | Typography | Core | Default body and form control line height | User-configured (e.g., `1.5`) | Primary user input | `--bs-body-line-height`, `body`, `.form-control`, `.form-select`, `.input-group-text`, `.tooltip`, `.popover` |
| `--ds-line-height-heading` | Typography | Core | Line height for headings and display text | User-configured (e.g., `1.2`) | Primary user input | Manual override on `h1`–`h6, .h1`–`.h6`, `.display-1`–`.display-6` |
| `--ds-text-align-base` | Typography | Core | Default document text alignment | User-configured (e.g., `start` / `left`) | Primary user input | `--bs-body-text-align`, `body` |

#### B. Computed Theme Variables

| Variable ID / Name | Design Category | Classification | Purpose | Default / Derivation Rule | Override Behaviour | Bootstrap Mappings |
| --- | --- | --- | --- | --- | --- | --- |
| `--ds-font-size-sm` | Typography | Computed | Smaller UI text step (helper text, small inputs, captions, feedback) | Derived from `--ds-font-size-base` (e.g., `calc(var(--ds-font-size-base) * 0.875)`) | Computed automatically | Manual override on `small, .small`, `pre`, `code`, `kbd`, `.form-text`, `.form-control-sm`, `.form-select-sm`, `.valid-*`, `.invalid-*`, `.dropdown-header` |
| `--ds-font-size-lg` | Typography | Computed | Larger UI text step (large inputs, lead, blockquote) | Derived from `--ds-font-size-base` (e.g., `calc(var(--ds-font-size-base) * 1.25)`) | Computed automatically | Manual override on `.lead`, `.blockquote`, `.form-control-lg`, `.form-select-lg`, `.input-group-lg *` |
| `--ds-heading-scale-ratio` | Typography | Computed | Scale multiplier driving heading/display sizes from `--ds-font-size-base` | Derived from `--ds-font-size-base` (default `1` when base is `1rem`) | Computed automatically | Manual RFS/heading overrides on `h1`–`h6` and `.display-1`–`.display-6` |
| `--ds-link-underline-opacity` | Typography | Computed | Default underline opacity for links and underline utilities | Derived from link emphasis/contrast settings (default `1`) | Computed automatically | `--bs-link-underline-opacity`, `.link-*`, `.link-underline*`, `.icon-link` |

#### C. Overridable Derived Variables

| Variable ID / Name | Design Category | Classification | Purpose | Default / Derivation Rule | Override Behaviour | Bootstrap Mappings |
| --- | --- | --- | --- | --- | --- | --- |
| `--ds-btn-font-family` / `--ds-btn-font-size` / `--ds-btn-font-weight` / `--ds-btn-line-height` | Typography | Overridable Derived | Button typography properties | Derived from `--ds-font-sans`, `--ds-font-size-base`, `--ds-font-weight-base` (or `500`), `--ds-line-height-base` | User-overridable | `--bs-btn-font-family`, `--bs-btn-font-size`, `--bs-btn-font-weight`, `--bs-btn-line-height` |
| `--ds-nav-link-font-size` / `--ds-nav-link-font-weight` | Typography | Overridable Derived | Navigation link size and weight | Derived from `--ds-font-size-base` / `--ds-font-weight-base` | User-overridable | `--bs-nav-link-font-size`, `--bs-nav-link-font-weight` |
| `--ds-dropdown-font-size` / `--ds-breadcrumb-font-size` / `--ds-pagination-font-size` | Typography | Overridable Derived | Menu, breadcrumb, and pagination font sizes | Derived from `--ds-font-size-base` / `--ds-font-size-sm` | User-overridable | `--bs-dropdown-font-size`, `--bs-breadcrumb-font-size`, `--bs-pagination-font-size` |
| `--ds-badge-font-size` / `--ds-badge-font-weight` | Typography | Overridable Derived | Badge font size and weight | Derived from `--ds-font-size-sm` / `--ds-font-weight-bold` | User-overridable | `--bs-badge-font-size`, `--bs-badge-font-weight` |
| `--ds-modal-title-line-height` / `--ds-offcanvas-title-line-height` | Typography | Overridable Derived | Modal and offcanvas heading line heights | Derived from `--ds-line-height-heading` / `--ds-line-height-base` | User-overridable | `--bs-modal-title-line-height`, `--bs-offcanvas-title-line-height` |
| `--ds-tooltip-font-size` / `--ds-popover-font-size` / `--ds-popover-header-font-size` / `--ds-toast-font-size` | Typography | Overridable Derived | Overlay/notification font sizes | Derived from `--ds-font-size-sm` / `--ds-font-size-base` | User-overridable | `--bs-tooltip-font-size`, `--bs-popover-font-size`, `--bs-popover-header-font-size`, `--bs-toast-font-size` |

### 3. Manual Overrides & Exclusions for Typography & Text

- **Manual overrides needed:**
- Headings (`h1`–`h6` / `.h1`–`.h6`), displays (`.display-1`–`.display-6`), `legend`, and `.fs-1`–`.fs-4` use RFS `calc(... + ...vw)` + `@media (min-width: 1200px)` rules with no `var()`; overriding base font size or heading scale requires manual rules (including the `1200px` breakpoint).
- Hardcoded font weights (`h1`–`h6` `500`, `.display-*` / `.lead` `300`, `dt` / `.nav-underline .nav-link.active` / `.alert-link` `700`) and `0.875em`/`rem` or `1.25rem` sizes on form controls, feedback, captions, and code blocks have no `var()`.
- `.link-<role>:hover, .link-<role>:focus` hardcode static `RGBA(...) !important` underline colours instead of referencing `-ds-<role>-hover-rgb`.
- `.fw-*`, `.lh-*`, `.fs-*`, `.fst-*`, `.text-*` (alignment/transform/decoration), and `.link-offset-*` utilities use `!important`, so any manual override on those classes must also use `!important` and match specificity/media queries.
- **Leave alone (functional / structural):** `sub, sup` (`0.75em`, `line-height: 0`, `vertical-align: baseline`), `.visually-hidden` (`1px` box / `nowrap`), `.carousel-indicators [data-bs-target]` (`text-indent: -999px`), `.text-truncate` (`ellipsis` + `nowrap`), `.form-floating > label` (`ellipsis` + `nowrap`), `.dropdown-toggle` / `.dropdown-item` / `.badge` (`nowrap`), and `.align-*` / `vertical-align` positioning rules.

# Category 8: Spacing

#### A. Core Theme Variables

| **Variable ID / Name** | **Design Category** | **Classification** | **Purpose** | **Default / Derivation Rule** | **Override Behaviour** | **Bootstrap Mappings** |
| --- | --- | --- | --- | --- | --- | --- |
| `--ds-space-unit` | Spacing | Core | Base spacing step used across computed scale/component defaults | User-configured (e.g., `0.25rem`) | Primary user input | Drives computed spacing steps & component spacing defaults |
| `--ds-gutter-x` | Spacing / Layout | Core | Horizontal grid & container gutter width | User-configured (e.g., `1.5rem`) | Primary user input | `--bs-gutter-x`, `.container*` horizontal padding, `.row` & `.row > *` horizontal spacing |
| `--ds-gutter-y` | Spacing / Layout | Core | Vertical grid row gutter spacing | User-configured (e.g., `0`) | Primary user input | `--bs-gutter-y`, `.row` & `.row > *` vertical spacing |

#### B. Computed Theme Variables

| **Variable ID / Name** | **Design Category** | **Classification** | **Purpose** | **Default / Derivation Rule** | **Override Behaviour** | **Bootstrap Mappings** |
| --- | --- | --- | --- | --- | --- | --- |
| `--ds-space-1` through `--ds-space-5` | Spacing | Computed | Standard spacing scale steps (`1` = `1x`, `2` = `2x`, `3` = `4x`, `4` = `6x`, `5` = `12x` of `--ds-space-unit`) | Derived from `--ds-space-unit` (defaulting to `0.25rem`, `0.5rem`, `1rem`, `1.5rem`, `3rem`) | Computed automatically | Manual utility overrides (if scaling spacing utilities) & default component spacing |
| `--ds-modal-footer-item-margin` | Spacing | Computed | Half-gap margin around modal footer action items | `calc(var(--ds-modal-footer-gap) * 0.5)` | Computed automatically | `.modal-footer > *` `margin` |
| `--ds-modal-footer-padding-bottom` | Spacing | Computed | Adjusted bottom padding for modal footer accounting for item gap | `calc(var(--ds-modal-padding) - var(--ds-modal-footer-gap) * 0.5)` | Computed automatically | `.modal-footer` `padding` |

#### C. Overridable Derived Variables

| **Variable ID / Name** | **Design Category** | **Classification** | **Purpose** | **Default / Derivation Rule** | **Override Behaviour** | **Bootstrap Mappings** |
| --- | --- | --- | --- | --- | --- | --- |
| `--ds-btn-padding-y` / `--ds-btn-padding-x` | Spacing | Overridable Derived | Vertical and horizontal button padding | Derived from `--ds-space-unit` (`0.375rem` / `0.75rem`) | User-overridable | `--bs-btn-padding-y`, `--bs-btn-padding-x` |
| `--ds-card-spacer-y` / `--ds-card-spacer-x` / `--ds-card-cap-padding-y` / `--ds-card-cap-padding-x` / `--ds-card-title-spacer-y` | Spacing | Overridable Derived | Card body, header/footer, and title vertical/horizontal spacing | Derived from `--ds-space-unit` / `--ds-space-3` | User-overridable | `--bs-card-spacer-y`, `--bs-card-spacer-x`, `--bs-card-cap-padding-y`, `--bs-card-cap-padding-x`, `--bs-card-title-spacer-y`, `--bs-card-group-margin` |
| `--ds-modal-padding` / `--ds-modal-header-padding` / `--ds-modal-margin` / `--ds-modal-footer-gap` | Spacing | Overridable Derived | Modal inner padding, header padding, outer margin, and footer button gap | Derived from `--ds-space-3` / `--ds-space-4` / `1.75rem` | User-overridable | `--bs-modal-padding`, `--bs-modal-header-padding`, `--bs-modal-header-padding-y`, `--bs-modal-header-padding-x`, `--bs-modal-margin`, `--bs-modal-footer-gap` |
| `--ds-offcanvas-padding-y` / `--ds-offcanvas-padding-x` | Spacing | Overridable Derived | Offcanvas header and body padding | Derived from `--ds-space-3` (`1rem`) | User-overridable | `--bs-offcanvas-padding-y`, `--bs-offcanvas-padding-x` |
| `--ds-dropdown-padding-y` / `--ds-dropdown-padding-x` / `--ds-dropdown-item-padding-y` / `--ds-dropdown-item-padding-x` / `--ds-dropdown-spacer` / `--ds-dropdown-divider-margin-y` | Spacing | Overridable Derived | Dropdown menu, item, spacer, and divider spacing | Derived from `--ds-space-unit` / `--ds-space-2` | User-overridable | `--bs-dropdown-padding-y`, `--bs-dropdown-padding-x`, `--bs-dropdown-item-padding-y`, `--bs-dropdown-item-padding-x`, `--bs-dropdown-header-padding-y`, `--bs-dropdown-header-padding-x`, `--bs-dropdown-spacer`, `--bs-dropdown-divider-margin-y` |
| `--ds-nav-link-padding-y` / `--ds-nav-link-padding-x` / `--ds-nav-underline-gap` / `--ds-navbar-padding-y` / `--ds-navbar-padding-x` / `--ds-navbar-nav-link-padding-x` / `--ds-navbar-brand-margin-end` | Spacing | Overridable Derived | Nav and navbar padding, underline gap, and brand spacing | Derived from `--ds-space-2` / `--ds-space-3` / `1rem` | User-overridable | `--bs-nav-link-padding-y`, `--bs-nav-link-padding-x`, `--bs-nav-underline-gap`, `--bs-navbar-padding-y`, `--bs-navbar-padding-x`, `--bs-navbar-brand-padding-y`, `--bs-navbar-brand-margin-end`, `--bs-navbar-nav-link-padding-x`, `--bs-navbar-toggler-padding-y`, `--bs-navbar-toggler-padding-x` |
| `--ds-alert-padding-y` / `--ds-alert-padding-x` / `--ds-alert-margin-bottom` | Spacing | Overridable Derived | Alert box padding and bottom margin | Derived from `--ds-space-3` (`1rem`) | User-overridable | `--bs-alert-padding-y`, `--bs-alert-padding-x`, `--bs-alert-margin-bottom` |
| `--ds-badge-padding-y` / `--ds-badge-padding-x` | Spacing | Overridable Derived | Badge padding | Derived from `--ds-space-unit` (`0.35em` / `0.65em` in BS) | User-overridable | `--bs-badge-padding-y`, `--bs-badge-padding-x` |
| `--ds-breadcrumb-padding-y` / `--ds-breadcrumb-padding-x` / `--ds-breadcrumb-item-padding-x` / `--ds-breadcrumb-margin-bottom` | Spacing | Overridable Derived | Breadcrumb bar padding, item separator spacing, and bottom margin | Derived from `--ds-space-2` / `--ds-space-3` | User-overridable | `--bs-breadcrumb-padding-y`, `--bs-breadcrumb-padding-x`, `--bs-breadcrumb-item-padding-x`, `--bs-breadcrumb-margin-bottom` |
| `--ds-pagination-padding-y` / `--ds-pagination-padding-x` / `--ds-list-group-item-padding-y` / `--ds-list-group-item-padding-x` | Spacing | Overridable Derived | Pagination link and list-group item padding | Derived from `--ds-space-unit` / `--ds-space-2` / `--ds-space-3` | User-overridable | `--bs-pagination-padding-y`, `--bs-pagination-padding-x`, `--bs-list-group-item-padding-y`, `--bs-list-group-item-padding-x` |
| `--ds-toast-padding-y` / `--ds-toast-padding-x` / `--ds-toast-spacing` | Spacing | Overridable Derived | Toast padding and vertical spacing between stacked toasts | Derived from `--ds-space-2` / `--ds-space-3` / `--ds-space-4` | User-overridable | `--bs-toast-padding-y`, `--bs-toast-padding-x`, `--bs-toast-spacing` |
| `--ds-tooltip-padding-y` / `--ds-tooltip-padding-x` / `--ds-tooltip-margin` / `--ds-popover-header-padding-y` / `--ds-popover-header-padding-x` / `--ds-popover-body-padding-y` / `--ds-popover-body-padding-x` | Spacing | Overridable Derived | Tooltip and popover padding/margin spacing | Derived from `--ds-space-unit` / `--ds-space-2` / `--ds-space-3` | User-overridable | `--bs-tooltip-padding-y`, `--bs-tooltip-padding-x`, `--bs-tooltip-margin`, `--bs-popover-header-padding-y`, `--bs-popover-header-padding-x`, `--bs-popover-body-padding-y`, `--bs-popover-body-padding-x` |

### 3. Manual Overrides & Exclusions for Spacing

- **Manual overrides needed (if scaling spacing globally via `-ds-space-unit`):**
    - Non-variable form, table, and element spacing (`p`, `ul`/`ol`, `hr`, `blockquote`, `figure`, `.form-control*`, `.form-select*`, `.form-floating*`, `.input-group*`, `.col-form-label*`, `.table > :not(caption) > * > *`, `.icon-link` gap `0.375rem`) do not read `-ds-space-unit` automatically unless overridden manually.
    - All `.m*`, `.p*`, `.gap*`, `.row-gap*`, and `.column-gap*` utility classes use `!important` across breakpoints (`sm`–`xxl`), so any manual override on those classes must include `!important` and respect the corresponding `@media (min-width: ...)` query.
- **Leave alone (layout geometry, resets & overlaps):**
    - `.offset-*` column percentages (`8.33333333%`–`91.66666667%`), `.container*` horizontal `margin: auto`, `.carousel-item` `margin-right: -100%`, `.visually-hidden` (`margin: -1px`), `.ratio::before` (`padding-top: var(--bs-aspect-ratio)`), nested list resets, and negative-margin overlap rules (`calc(-1 * var(--bs-border-width))`, `calc(-0.5 * ...)`).

Ready for the next audit category whenever you paste it!

# Cross-category consolidation review — 19 November 2026

**Scope:** Reviewed all eight category sections together to identify shared concepts, cross-category dependencies, naming inconsistencies, and decisions to resolve. This is a model-consolidation pass; it does not repeat or replace the existing Bootstrap source audits.

## 1. Overall structure

The eight categories form a sensible property-based organisation. Keep that structure for now. Categories 3 (Generated Content & Decoration) and 6 (Sizing) having no Core or Computed variables is not a problem: their current entries are optional component-level hooks rather than foundational theme inputs.

The category tables should remain the detailed specification, but the final document will also need a cross-category index. A property-based category alone does not show how a complete component is themed.

## 2. Cross-category overlaps to resolve

| Shared concept | Where it currently appears | Consolidation needed |
| --- | --- | --- |
| Focus appearance | Category 2: `--ds-outline-width`, `--ds-outline-offset`, `--ds-focus-ring-color`; Category 5: `--ds-focus-ring-width`, `--ds-focus-ring-alpha`, `--ds-focus-ring-shadow` | Define the distinction between a CSS outline and a box-shadow focus ring. Decide whether both are supported mechanisms, and document which tokens control each. Avoid two competing width controls unless they intentionally represent different things. |
| Backdrop scrim | Category 1: `--ds-backdrop-bg`; Category 5: `--ds-backdrop-opacity` | Define one unambiguous relationship. If the background token already includes alpha, applying a separate opacity can compound the transparency. Prefer a clear colour-plus-opacity model, or make one token the complete value and remove the redundant control. |
| Links | Category 1: `--ds-link-color`, role hover colours; Category 7: `--ds-link-underline-opacity` and link-related manual overrides | Keep the tokens in their property categories, but include them together in a link-family index so link colour, hover/focus treatment, and decoration are specified as one coherent component behaviour. |
| Focus and semantic colours | Category 1: primary and semantic colours; Category 2: focus and semantic border colours; Category 5: focus-ring shadow | Treat the primary/semantic colour tokens as shared source primitives. Border and shadow categories should consume them rather than define competing colour sources. |
| Component-level theming | Categories 1, 2, 5, 6, 7 and 8 contain card, button, modal, popover, tooltip, toast, navigation and other component tokens | Add a component-family index later. For example, a card's background, border, radius and spacing should be discoverable together even though each token remains in its property category. |

## 3. Naming conventions to standardise

The generic `<role>` notation is currently ambiguous beside the concrete names. For example, the document uses `--ds-color-primary`, `--ds-color-primary-rgb`, `--ds-on-primary`, and generic forms such as `--ds-<role>-hover`.

Before freezing names, define one explicit template for each family:

- Semantic colour source tokens: establish whether the canonical pattern is `--ds-color-{role}` and use it consistently.
- Semantic colour derivatives: define the exact pattern for RGB, hover, active, disabled, subtle, text-emphasis and on-colour tokens.
- Surface and text tokens: keep their separate `--ds-bg-*` and `--ds-text-*` namespaces, and state which derivative patterns apply to them.
- Component templates: clarify whether patterns such as `--ds-<component>-inner-radius` describe a token instantiated for a fixed supported component list, or a genuinely generic variable.

This is a naming decision, not a reason to rename everything immediately. First agree the templates, then apply them consistently.

## 4. Clarify the classification rules

The current **Overridable Derived** category contains two slightly different kinds of variable:

1. A value genuinely derived from another theme token, but which users may override.
2. A component-level setting with a standalone default copied from Bootstrap, such as a fixed maximum width or opacity, which users may override.

The second kind is not necessarily derived from another token. Decide whether to broaden the definition of Overridable Derived to include Bootstrap-defaulted component hooks, or distinguish those hooks with a separate classification. Apply the chosen definition consistently across all categories.

Also make the Computed classification precise: each computed variable should identify its source token(s), derivation rule, and whether the calculation can be expressed in CSS or requires runtime computation.

## 5. Build a cross-category dependency register

The next useful artefact is a dependency register, initially as a table rather than a diagram. Record each computed or overridable-derived variable, its source token(s), its derivation/default, and any cross-category dependency.

Important dependency groups already visible in the specification include:

- Semantic colours → RGB channels, interaction states, subtle surfaces, contrast text and semantic borders.
- Surface/text colours → emphasis and component surface treatments.
- Primary colour → focus-ring colour/shadow and other primary-accent states.
- Base shadow → small and large elevation shadows.
- Base font settings → component typography defaults and the type scale.
- Base spacing unit → spacing scale → component padding, gaps and margins.
- Component radius/border width → inner radius.
- Motion durations/easing → transition shorthands and component transitions.

This register will expose missing source tokens, circular dependencies, and values whose derivation is currently underspecified.

## 6. Recommended order of work

1. Agree the naming templates for semantic roles and component tokens.
2. Resolve the focus-ring/outline and backdrop colour/opacity overlaps.
3. Define exactly what Core, Computed and Overridable Derived mean.
4. Create the dependency register across all eight categories.
5. Create a component-family index so related tokens can be found together.
6. Once those are coherent, freeze the conceptual theme-variable API and move on to the runtime implementation design.

**Current conclusion:** The category breakdown is a good working structure. It does not need to be rebuilt. The next pass should make the relationships between categories explicit and settle the few shared design decisions before the variable model is treated as final.

# Theme Variable Dependency Model — 19 November 2026

**Purpose**

This model describes how the variables in Categories 1–8 relate to one another: which values are configured directly, which are derived from them, and which component-level hooks inherit or override those values.

It is a conceptual dependency model based on the current category specifications. It does not introduce new Bootstrap mappings or assume a particular runtime implementation. Where the source specification leaves a derivation open or ambiguous, that is called out rather than silently decided here.

## 1. Dependency layers

The model has four practical layers:

1. **Core inputs** — values configured by a theme author.
2. **Shared computed values** — reusable scales, colour/state derivatives, and compound values derived from core inputs.
3. **Component-level defaults and overrides** — component tokens that inherit from shared values but can be customised independently.
4. **Bootstrap application layer** — the Bootstrap custom properties and manual selector overrides that consume the theme tokens. This is the output of the model, not another source of theme values.

A component token may sit in layer 3 even if its classification is currently called Overridable Derived. The classification terminology still needs the separate decision recorded in the consolidation review.

## 2. High-level dependency map

The following map shows the main dependency paths. Arrows mean “is a source for” or “is used to derive”; they do not imply that every calculation is already fully specified.

```
CORE THEME INPUTS
│
├── Colour & surfaces
│   ├── Semantic colours: primary, secondary, success, info, warning, danger
│   ├── Surface colours: base, subtle, elevated
│   └── Text colours: base, muted, heading
│
│   ├──> RGB channels
│   ├──> Hover / active / disabled semantic states
│   ├──> Subtle semantic surfaces
│   ├──> On-colour foregrounds and semantic emphasis text
│   ├──> Text emphasis / tertiary text
│   ├──> Overlay / sunken surfaces
│   └──> Link, highlight, code, card, popover, tooltip, progress hooks
│
├── Borders & radii
│   ├── Border width / style / colour
│   ├── Base radius / pill radius
│   └── Outline width / offset
│
│   ├──> Border RGB and semantic subtle borders
│   ├──> Radius scale: sm / lg / xl / xxl
│   ├──> Component radii: button / card / modal / dropdown / tooltip / popover
│   └──> Inner radii from component radius and border width
│
├── Typography
│   ├── Font families, base size, base weights, base line heights, alignment
│   └──> Small / large text sizes, heading scale, link underline opacity
│       └──> Button / navigation / dropdown / badge / overlay typography
│
├── Spacing
│   ├── Space unit, horizontal gutter, vertical gutter
│   └──> Space scale: 1–5
│       └──> Component padding, gaps, margins and spacing adjustments
│
├── Shadows & opacity
│   ├── Base shadow, focus-ring width/alpha, disabled opacity
│   └──> Small / large shadows and focus-ring shadow
│       └──> Component elevation and focus-state treatments
│
└── Motion
    ├── Fast / base / slow durations and standard / out easing
    └──> Control and panel transition shorthands
        └──> Accordion and progress transitions; component motion overrides

COMPONENT TOKEN LAYER
    └──> Bootstrap CSS custom properties and manual CSS overrides
```

Generated content/decoration (Category 3) and sizing (Category 6) currently contain component-level hooks without a shared core/computed dependency chain. They are therefore represented as leaf settings in this model unless a later design decision gives them shared source tokens.

## 3. Dependency register

### 3.1 Colour and surface dependencies

| Source token(s) | Derived / downstream token(s) | Relationship described by current specification | Status / decision |
| --- | --- | --- | --- |
| `--ds-color-{role}` | `--ds-<role>-rgb` | Extract RGB channels from the matching semantic colour. | Defined conceptually; exact role-token naming template still to standardise. |
| `--ds-bg-base`, `--ds-bg-subtle`, `--ds-bg-elevated`, `--ds-text-*` | Corresponding `--ds-<role>-rgb` values | Extract RGB channels for alpha-based colour use. | Defined conceptually. |
| `--ds-color-{role}` | `--ds-<role>-hover`, `--ds-<role>-active`, `--ds-<role>-disabled` | Derive interaction states through mode-aware shifts or attenuation. Active is a stronger shift than hover; disabled may mix with a neutral surface. | Direction defined; colour-space and exact derivation need specification. |
| `--ds-color-{role}`, `--ds-bg-base` | `--ds-<role>-subtle` | Blend semantic colour into the base surface to create a subtle tinted surface. | Direction defined; blend ratio/colour space need specification. |
| `--ds-color-{role}` | `--ds-on-<role>` | Select a readable foreground (light or dark) for text/icons on a solid semantic background. | Contrast algorithm and target threshold need specification. |
| `--ds-text-base` | `--ds-text-emphasis`, `--ds-text-emphasis-rgb` | Derive a stronger-emphasis text colour and its RGB channels. | Concept defined; exact relationship needs specification. |
| `--ds-text-muted`, `--ds-bg-base` | `--ds-text-tertiary` | Derive a softer tertiary text step. | Concept defined; contrast/opacity rule needs specification. |
| `--ds-color-{role}`, `--ds-bg-base` / `--ds-border-color` | `--ds-<role>-text-emphasis` | Produce mode-aware semantic text suitable for subtle/light semantic surfaces. | Concept defined; contrast rule needs specification. |
| `--ds-<role>-hover` | `--ds-<role>-hover-rgb` | Extract RGB channels from the hover colour for link hover/focus treatment. | Defined conceptually. |
| `--ds-bg-base`, `--ds-bg-elevated` | `--ds-bg-overlay` | Choose/derive the raised surface used by overlays and floating components. | Source alternatives are listed; precedence needs deciding. |
| `--ds-bg-subtle`, `--ds-bg-base` | `--ds-bg-sunken` | Derive a recessed or muted surface for headers/footers. | Source alternatives are listed; exact rule needs deciding. |
| `--ds-color-primary` | `--ds-link-color` | Link colour defaults to primary, but has an independent user override. | Defined. |
| `--ds-color-warning`, `--ds-text-base` | `--ds-highlight-bg`, `--ds-highlight-text` | Defaults for highlighted text treatment. | Defined conceptually; retain independent overrides. |
| Semantic/accent colour role | `--ds-code-text` | Default inline-code colour derives from an accent/semantic role. | Source role is not yet explicit; decide which one. |
| `--ds-bg-base`, `--ds-bg-sunken` | `--ds-card-bg`, `--ds-card-cap-bg` | Card body and cap surfaces inherit from general surface tokens. | Defined; component overrides remain available. |
| `--ds-bg-overlay` / contrast fill | `--ds-popover-bg`, `--ds-tooltip-bg` | Popover and tooltip surfaces inherit from overlay/contrast treatment. | Defined as alternatives; specify exact defaults and whether tooltip differs. |
| `--ds-bg-subtle`, `--ds-color-primary` | `--ds-progress-bg`, `--ds-progress-bar-bg` | Track and fill inherit from neutral and primary colours. | Defined; component overrides remain available. |
| `--ds-color-primary` | `--ds-focus-ring-color` | Focus outline/border colour derives from primary, possibly softened. | Cross-category dependency; decide whether this colour is also the source for the shadow ring. |

**Important boundary:** `--ds-backdrop-bg` is currently listed as a computed translucent colour, while Category 5 separately defines `--ds-backdrop-opacity`. These must be reconciled. Do not apply both alpha in the colour and element opacity without explicitly intending the compounded result.

### 3.2 Border, radius and outline dependencies

| Source token(s) | Derived / downstream token(s) | Relationship described by current specification | Status / decision |
| --- | --- | --- | --- |
| `--ds-border-color` | `--ds-border-color-rgb` | Extract RGB channels for alpha border utilities. | Defined. |
| `--ds-color-{role}`, `--ds-bg-base` / `--ds-border-color` | `--ds-<role>-border-subtle` | Blend semantic colour with a neutral surface/border for subtle semantic borders. | Concept defined; exact blend rule needs specification. |
| `--ds-radius-base` | `--ds-radius-sm`, `--ds-radius-lg`, `--ds-radius-xl`, `--ds-radius-xxl` | Generate the radius scale using the documented multipliers, with special handling when base radius is zero. | Ratios proposed in Category 2; confirm as normative values. |
| `--ds-radius-base` | `--ds-btn-radius`, `--ds-card-radius`, `--ds-modal-radius`, `--ds-dropdown-radius` | Component radii default to the base radius. | Defined; independently overridable. |
| `--ds-radius-sm`, `--ds-radius-base` | `--ds-dropdown-item-radius`, `--ds-tooltip-radius`, `--ds-popover-radius` | Smaller or standard radius defaults for compact/floating components. | Defined; independently overridable. |
| Component radius and component border width | `--ds-<component>-inner-radius` | Inner radius = max(0, outer radius − border width). | Formula defined; requires a concrete component token-pair inventory. |
| `--ds-border-width`, `--ds-border-style`, `--ds-border-color` | Bootstrap border defaults and component borders | Shared structural border primitives feed standard borders and components. | Defined at conceptual level. |
| `--ds-color-primary` | `--ds-focus-ring-color` | Primary brand colour supplies the focus outline colour. | Shared with Colour & Surfaces; avoid a second unrelated focus colour source. |
| `--ds-outline-width`, `--ds-outline-offset` | Manual focus-outline rules | Control outline geometry where the chosen focus treatment uses CSS outline. | Keep distinct from box-shadow ring width unless one is deliberately made an alias. |

### 3.3 Typography dependencies

| Source token(s) | Derived / downstream token(s) | Relationship described by current specification | Status / decision |
| --- | --- | --- | --- |
| `--ds-font-size-base` | `--ds-font-size-sm`, `--ds-font-size-lg` | Small and large text steps derive by scale factors (currently proposed as 0.875 and 1.25). | Proposed factors; confirm as defaults. |
| `--ds-font-size-base` | `--ds-heading-scale-ratio` | Heading/display scale is driven by base size. | Current description is underspecified: a ratio of 1 at a 1rem base does not itself define a heading scale. Clarify whether this is a scale factor, a breakpoint/RFS control, or remove it. |
| `--ds-font-sans`, `--ds-font-size-base`, `--ds-font-weight-base`, `--ds-line-height-base` | Button typography tokens | Button typography inherits from global body typography, with a possible independent weight default. | Defined; document the exception/default weight clearly. |
| `--ds-font-size-base`, `--ds-font-weight-base` | Navigation link and dropdown/breadcrumb/pagination typography | Component text inherits global type settings or the small scale. | Defined. |
| `--ds-font-size-sm`, `--ds-font-weight-bold` | Badge typography | Badge size/weight inherit from shared type tokens. | Defined. |
| `--ds-line-height-heading`, `--ds-line-height-base` | Modal/offcanvas title line-height | Overlay title line-height inherits from heading/body line-height. | Defined; independent override available. |
| `--ds-font-size-sm`, `--ds-font-size-base` | Tooltip/popover/toast typography | Floating and notification text inherits from small/base type scale. | Defined; independent override available. |
| Link colour/emphasis settings | `--ds-link-underline-opacity` | Underline opacity derives from link emphasis/contrast settings. | Source relationship needs a precise rule; current default is 1. |

### 3.4 Spacing dependencies

| Source token(s) | Derived / downstream token(s) | Relationship described by current specification | Status / decision |
| --- | --- | --- | --- |
| `--ds-space-unit` | `--ds-space-1` through `--ds-space-5` | Scale multipliers are 1×, 2×, 4×, 6× and 12× the unit. | Defined; confirm whether the scale is fixed or configurable. |
| `--ds-space-unit`, space scale | Button, card, modal, offcanvas, dropdown, nav, alert, badge, breadcrumb, pagination, list-group, toast, tooltip and popover spacing tokens | Component spacing defaults draw from the shared unit/scale, with selected fixed Bootstrap defaults. | Broad relationship defined; exact per-token source is recorded in Category 8. |
| `--ds-modal-footer-gap` | `--ds-modal-footer-item-margin` | Half the footer gap is used as the item margin. | Formula defined. |
| `--ds-modal-padding`, `--ds-modal-footer-gap` | `--ds-modal-footer-padding-bottom` | Bottom padding = modal padding − half the footer gap. | Formula defined. |
| `--ds-gutter-x`, `--ds-gutter-y` | Grid/container gutter output | Horizontal and vertical grid gutters are independent core layout inputs. | Defined; do not derive these from the general spacing scale unless that is an intentional later decision. |

### 3.5 Shadow, opacity and filter dependencies

| Source token(s) | Derived / downstream token(s) | Relationship described by current specification | Status / decision |
| --- | --- | --- | --- |
| `--ds-shadow-base` | `--ds-shadow-sm`, `--ds-shadow-lg` | Smaller/larger elevation shadows scale from the base shadow. | Concept defined; CSS cannot generally scale an arbitrary shadow string, so derivation mechanism needs deciding. |
| `--ds-focus-ring-width`, `--ds-color-primary-rgb`, `--ds-focus-ring-alpha` | `--ds-focus-ring-shadow` | Compose a focus halo from ring width, primary RGB channels and alpha. | Defined conceptually; align its colour source with Category 2 `--ds-focus-ring-color`. |
| `--ds-disabled-opacity` | Disabled controls and labels | Shared opacity for disabled visual treatment. | Core input; preserve functional disabled behaviour separately from appearance. |
| `--ds-backdrop-opacity` | Modal/offcanvas backdrop appearance | Controls scrim opacity. | Reconcile with Category 1 backdrop colour alpha. |
| Mode / surface context | `--ds-btn-close-filter`, `--ds-carousel-control-filter` | Choose icon filter treatment suitable for light/dark surfaces. | Mode-aware behaviour described; exact mode mapping needs specification. |
| Opacity hooks | Tooltip, close button, placeholder, divider and carousel-control appearance | Standalone component-level opacity defaults. | These are override hooks; not all are derived from a common opacity scale. |

### 3.6 Motion dependencies

| Source token(s) | Derived / downstream token(s) | Relationship described by current specification | Status / decision |
| --- | --- | --- | --- |
| `--ds-duration-fast`, `--ds-ease-standard` | `--ds-transition-control` | Compose the standard interactive-control transition shorthand. | Defined conceptually; exact property list is documented in Category 4. |
| `--ds-duration-base`, `--ds-ease-out` / `--ds-ease-standard` | `--ds-transition-panel` | Compose the transition shorthand for panels/drawers/modals. | Defined conceptually; choose the easing per component or define a single rule. |
| `--ds-transition-panel` / `--ds-duration-base` | `--ds-accordion-transition` | Accordion transition inherits from the panel transition. | Defined; independently overridable. |
| `--ds-duration-slow`, `--ds-ease-standard` | `--ds-progress-bar-transition` | Progress fill transition derives from slow motion defaults. | Defined; independently overridable. |
| `--ds-accordion-icon-transform`, `--ds-icon-link-transform` | Accordion and icon-link visual states | Standalone transform values for component interaction states. | Leaf component hooks; not derived from duration/easing. |

### 3.7 Leaf component settings (no shared token parent currently specified)

These variables are still part of the theme API, but the current specification does not define a meaningful upstream token dependency for them. Their defaults are component-specific settings rather than outputs of a shared scale.

- **Generated content:** `--ds-breadcrumb-divider`, `--ds-blockquote-prefix`.
- **Sizing:** `--ds-modal-max-width`, `--ds-popover-max-width`, `--ds-tooltip-max-width`, `--ds-progress-height`, `--ds-offcanvas-height`, `--ds-offcanvas-width`, `--ds-dropdown-min-width`, `--ds-toast-max-width`, `--ds-nav-scroll-max-height`.
- **Standalone opacity/filter hooks:** `--ds-tooltip-opacity`, `--ds-btn-close-opacity`, `--ds-btn-close-hover-opacity`, `--ds-placeholder-opacity`, `--ds-divider-opacity`, `--ds-btn-close-filter`, `--ds-carousel-control-filter`.
- **Component-specific spacing hooks:** retain their Category 8 parent relationships where specified; any value whose default is a fixed literal rather than a shared spacing token should be treated as a leaf default until deliberately connected to the scale.

## 4. Component-family paths

These paths bring tokens from different property categories together. They are not new variables; they are views across the existing specification.

| Component family | Relevant token groups |
| --- | --- |
| Button | Semantic fill/hover/active/disabled colours and on-colour text; button radius; border; focus treatment; shadow; typography; padding; transition. |
| Form controls | Base/subtle/elevated surfaces; body/muted text; border and radius; focus outline/ring; disabled opacity; typography; spacing; validation semantic colours/borders; hardcoded SVG/state overrides. |
| Card | Card/body/cap surfaces; card border colour/width; card radius and inner radius; shadow; typography; card spacing. |
| Modal | Overlay surface and backdrop; modal radius/inner radius; border; shadow; focus behaviour; width; typography; title line-height; padding/footer gap; panel transition. |
| Offcanvas | Overlay surface and backdrop; border/radius; shadow; width/height; typography/title line-height; padding; panel transition. |
| Dropdown | Overlay surface; border/radius/item radius; min width; typography; padding/item/divider spacing; hover/active states; motion where applicable. |
| Navigation/navbar | Link colour and underline; typography; spacing; focus treatment; scroll max-height; toggler icon override. |
| Tooltip/popover | Overlay/contrast surface; text colour; radius; width; opacity; typography; padding; shadow; arrow/icon treatment. |
| Toast | Overlay surface; border/radius; shadow; max width; typography; padding/stack spacing. |
| Progress | Track/fill colours; height; opacity/striping treatment; fill transition. |
| Breadcrumb | Link colour/underline; typography; divider content; padding/item spacing/margin. |

## 5. Dependency decisions to settle before implementation

These are the places where the current specification does not yet define a sufficiently precise dependency rule:

1. **Colour derivation method:** choose the colour space and algorithm for hover/active/disabled, subtle surfaces, emphasis text and on-colour contrast. Define contrast targets and what happens when a colour cannot meet them.
2. **Backdrop composition:** choose whether backdrop colour is opaque plus a separate opacity, or an alpha colour with no second opacity control.
3. **Focus treatment:** decide whether outline and box-shadow rings are both first-class options, and how their widths/colours relate.
4. **Radius scale:** confirm the proposed multipliers and zero-radius behaviour; define which component radius/border-width pairs generate inner radii.
5. **Shadow scale:** choose whether small/large shadows are independently configured presets or generated from structured shadow parameters. Arbitrary CSS shadow strings are not reliably scalable with simple CSS calculations.
6. **Typography scale:** define the actual heading-size progression and clarify the purpose of `--ds-heading-scale-ratio`; specify whether responsive RFS behaviour is part of the theme system.
7. **Spacing scale:** confirm fixed multipliers and identify which component defaults intentionally remain fixed rather than scaling with `--ds-space-unit`.
8. **Mode resolution:** specify whether derived colours are generated independently for light/dark modes and how mode-specific user overrides take precedence.
9. **Runtime boundary:** decide which derivations can be expressed with CSS custom properties and which require JavaScript preprocessing/runtime calculation. In particular, colour contrast/colour mixing and arbitrary shadow transformations should not be assumed to be CSS-only.

## 6. How to use this model

Treat this as the relationship layer over the eight category specifications:

- The **category sections** remain the source of truth for each variable's purpose, classification, defaults, overrides and Bootstrap mappings.
- This **dependency model** is the source of truth for upstream/downstream relationships and cross-category component paths.
- When a variable changes, use the register to identify affected derived tokens and component families.
- When a new token is proposed, identify its parent source(s), derivation/default, override behaviour and downstream consumers before adding it to a category.
- Keep unresolved derivations explicitly marked until decided; do not treat illustrative examples as final algorithms.

**Model status:** First-pass dependency model based on the current eight category specifications. It is sufficiently structured to guide the next design-decision pass, but it is not yet a frozen implementation contract.
