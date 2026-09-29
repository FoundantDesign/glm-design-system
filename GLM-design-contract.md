# GLM Design Contract v1

*The single source of truth for prototype generation and design-system governance.*
*Ratifies the production dialect found in the live Request Summary build and its shipped CSS.*
*Replaces: style-brief-v2.md. Retires: all `ft-*` invented classes and the `--primary` style of hardcoded tokens.*

---

## 0. How to read this document

This contract has two kinds of statement, kept rigorously separate so "what is true now" and "what we intend to fix" never blur together:

**RATIFIED** means the value or pattern is exactly what production ships today. Prototypes must reproduce it verbatim, because a prototype built from ratified values is liftable Razor markup, not a thing engineering has to re-derive.

**GAP** means production is incomplete, internally inconsistent, or using a stock default where a Foundant value belongs. Prototypes should follow the *resolution* noted, and the design system should close the gap in the real codebase. Gaps are listed in Section 9 and flagged inline where relevant.

The governing principle of the whole system: **style components by remapping MDB's own `--mdb-*` variables onto Foundant tokens. Never write new component CSS, never hardcode a color.** This is how production already works, and it is the reason the dialect is liftable.

---

## 1. Stack and load order

Tech stack: .NET 8, MVC, Razor, MDB UI Kit, Font Awesome Pro, AG Grid, minimal vanilla JavaScript.

Production loads CSS in this exact order, and each layer depends on the ones before it. Prototypes must preserve the order:

1. `mdb.min.css` — MDB UI Kit defaults
2. `mdb-patches.css` — small surgical fixes
3. `foundant-tokens.css` — Foundant tokens, `:root` only, no component rules
4. `mdb-customizations.css` — `--mdb-*` remaps onto Foundant tokens, plus component overrides
5. Page-shared CSS as needed (`GLM.Summary.css` for entity Summary pages)

**RATIFIED.** The ordering is load-bearing: `mdb-customizations.css` references Foundant tokens that must already be defined, and overrides MDB defaults that must already be loaded.

**`mdb_min.css` is now available in this project** (added after the checkbox/radio/switch color investigation), so MDB's actual default rules, variable names, and specificity can be checked directly going forward instead of inferred or flagged as unconfirmed. Earlier sections written before this was available (Text Inputs/Select border-focus treatment, Section 6h) have been retroactively checked against it and updated where the evidence changed the reasoning.

---

## 2. Tokens (Foundant vocabulary)

These are the canonical names. Everything else in the system points at these. Values are RATIFIED from `foundant-tokens.css`.

### 2a. Brand and primary

| Token | Value | Notes |
|---|---|---|
| `--colors-brand-primary-dark-blue` | `rgb(45, 59, 74)` | The primary. Text, primary buttons, headings, borders-brand. |
| `--colors-brand-secondary-light-blue` | `rgb(213, 226, 250)` | Active/selected backgrounds (side panel active, info subtle). |
| `--colors-brand-tertiary-white` | `rgb(255, 255, 255)` | Surface. |
| `--colors-brand-orange` | `rgb(242, 107, 34)` | Brand accent. |
| `--colors-brand-yellow` | `rgb(246, 184, 69)` | Brand accent. |
| `--colors-brand-light-yellow` | `rgb(254, 251, 220)` | Brand accent, subtle. |

The `--colors-primary-shades-0..90` ramp (from `rgb(45,59,74)` at 0 to `rgb(234,235,237)` at 90) is the tint ramp off the primary. `shades-30` = muted text via `.text-muted`, `shades-40` = `--mdb-secondary-color`, `shades-90` = navbar brand-badge background (paired with `--colors-brand-primary-dark-blue` text — the real, shipped light-bg/dark-text pairing pattern).

**Now surfaced in the kitchen sink Colors section (previously existed in `foundant-tokens.css` but was never displayed).** Reverse-engineered the exact formula from the real values: step `N` = a straight `N%` linear blend of the base color toward white, verified independently across all three RGB channels (not assumed) — e.g. step 10 is consistently ~10% white-blended on R, G, and B. This is a different convention from the danger/success/warning/info 50–900 scale (different step numbering, no dedicated "500 = base" identity), and that inconsistency is intentional, not an oversight: `primary-shades` already ships in production, so it wasn't touched or renumbered to match danger's convention.

**New: a matching secondary ramp, generated (not yet ratified).** Applied the identical N%-toward-white formula to `--colors-brand-secondary-light-blue` (`#D5E2FA`) as the step-0 anchor:

| Step | Hex |
|---|---|
| 0 | `#D5E2FA` (existing token, unchanged) |
| 10 | `#D9E5FB` |
| 20 | `#DDE8FB` |
| 30 | `#E2EBFC` |
| 40 | `#E6EEFC` |
| 50 | `#EAF1FD` |
| 60 | `#EEF3FD` |
| 70 | `#F2F6FE` |
| 80 | `#F7F9FE` |
| 90 | `#FBFCFF` |

**Caveat, stated plainly:** because the secondary base is already very light, the entire ramp lives in a narrow band between `#D5E2FA` and `#FBFCFF`. A contrast matrix across these steps will show mostly Fail — that's an accurate property of a light base color, not a defect in the ramp or the math. There's also no dark "step-0-equivalent" of secondary's own to use as label/text color the way primary's own darkest step serves its lighter end; labels/text on secondary fills borrow `--colors-brand-primary-dark-blue` directly, matching how secondary is actually paired with text in production today (e.g. `.hero-status-badge` uses an info/light-blue background with `--colors-brand-primary-dark-blue` text). Not yet in `foundant-tokens.css` — same status as the success/warning/info ramps in Section 2c: proposed, pending team review, needs engineering sign-off before ratifying.

Contrast matrices (every foreground × background pairing, WCAG 2.2) now exist for **all six** color families — danger, primary, secondary, success, warning, info — using the same reusable JS matrix component.

### 2b. Neutral ramp

`--colors-neutral-0` (#000) through `--colors-neutral-100` (#fff), eleven stops. RATIFIED. Common usage: `30` muted/secondary text, `40` secondary icons + empty-state text, `50` disabled borders, `60` dividers, `70` hover backgrounds, `80` app background, `90` subtle backgrounds, `100` surface.

### 2c. Semantic colors

| Role | `-0` (foreground) | `-50` (subtle bg) | Status |
|---|---|---|---|
| Success | `rgb(103, 207, 157)` | `rgb(225, 245, 235)` | RATIFIED at 0/50 |
| Danger | `rgb(189, 59, 57)` (= `-500`) | `#FAEAEA` (= `-50`) | RATIFIED, full ramp exists |
| Warning | `rgb(244, 200, 98)` | `rgb(255, 244, 209)` | RATIFIED at 0/50 |
| Info | `rgb(83, 110, 138)` | `rgb(213, 226, 250)` | RATIFIED at 0/50 |

**Danger is the only fully-validated semantic ramp** (`-50` through `-900`, WCAG-checked). Use the full ramp for danger states: `-500` anchor, `-600` hover, `-700` active/pressed, `-50` subtle bg.

**Text on any `-50` subtle fill is always `--colors-brand-primary-dark-blue`.** This holds for badges, alerts, toasts, and row highlights. RATIFIED.

### 2d. State layers

| Token | Value |
|---|---|
| `--colors-state-layers-dark-hover` | `rgb(66, 79, 92)` |
| `--colors-state-layers-dark-focused` | `rgb(66, 79, 92)` |
| `--colors-state-layers-dark-pressed` | `rgb(87, 98, 110)` |
| `--colors-state-layers-dark-link` | `rgb(57, 110, 196)` |
| `--colors-state-layers-light-hover` | `rgb(234, 235, 237)` |
| `--colors-state-layers-light-focused` | `rgb(234, 235, 237)` |
| `--colors-state-layers-light-pressed` | `rgb(213, 216, 219)` |

RATIFIED. Never invent hover/focus/pressed colors. Dark layers for filled controls, light layers for outline/ghost controls. `dark-link` is the in-content link color used on record pages.

### 2e. Borders

`--colors-borders-divider` `rgb(224,224,224)` is the workhorse: cards, rows, table cells, dividers. `--colors-borders-enabled` `rgb(117,117,117)` for input borders, `--colors-borders-disabled` `rgb(189,189,189)`, and per-semantic border tokens (`-danger`, `-warning`, `-success`, `-info`) for state borders. `borders-hover/focused/brand/pressed` all resolve to the primary `rgb(45,59,74)`. RATIFIED.

### 2f. Links and radius

**Correction:** this section previously stated link colors as `rgb(25,118,210)`/`rgb(35,82,124)` and marked them RATIFIED. That was wrong — verified directly against `mdb-customizations.css`, production actually remaps `a { --mdb-link-color-rgb: var(--colors-brand-primary-dark-blue); }` (navy, not blue). The prototype's own remap had been using `--colors-link-default`/`--colors-link-hover` (the values above) instead, producing generic blue links — a real rendering bug, now fixed to match `--mdb-link-color-rgb: var(--colors-brand-primary-dark-blue)` exactly.

`--colors-link-default`, `--colors-link-hover`, `--color-primary`, `--color-primary-hover` remain defined in `foundant-tokens.css` but are unused anywhere in current production CSS — leftovers from the older `foundant.css` this file mirrors (per its own header comment). Candidates for removal; see Section 9.

Radius scale: `--radius-none` 0, `--radius-xs` 2px, `--radius-sm` 4px, `--radius-md` 8px, `--radius-round` 50px. RATIFIED, all in `foundant-tokens.css`.

> **GAP — `--radius-card` exists only in the prototype, not yet in `foundant-tokens.css`.** Real cards and the hero render at `6px` (hardcoded in page CSS); the prototype named this `--radius-card: 6px` in `glm-prototype-foundation.css` so components stop using a magic number. It has never been round-tripped into the actual token file — a prototype-only patch masquerading as a named token. See Section 9.

---

## 3. Typography

| Property | Value | Source |
|---|---|---|
| Font family | `Roboto, sans-serif` via `--body-font` | RATIFIED (`foundant-tokens.css`) |
| Base size | `16px` | RATIFIED |
| `h1` | `24px` | RATIFIED (`mdb-customizations.css` sets `h1{font-size:24px}`) |
| `h1.OrganizationSelector` | `32px`, weight `700`, color `primary-shades-50` | RATIFIED, special case |
| Body weight | `400` | RATIFIED |
| Heading weight | `500` (MDB default, kept) | RATIFIED |
| Button font size | `14px` | RATIFIED (`.btn{--mdb-btn-font-size:14px}`) |

Record page titles use Bootstrap's `.fs-2` utility on an `<h1>` plus a page-specific class (e.g. `.request-header-title`), not a bare `h1`. Card titles use `.card-title` inside `.card-body`. RATIFIED from the live page.

> **GAP — heading scale undocumented.** Only `h1` is pinned in shipped CSS. `h2`–`h6` fall through to MDB defaults. The v2 brief invented an `h1..h4` scale that production does not use; it is retired. If a heading scale is wanted, it must be added to `foundant-tokens.css` and validated, not assumed. See Section 9.

> **Note — font swap seam.** v2 kept a `--font-heading`/`--font-body` split so a heading face could be introduced later. Production uses a single `--body-font`. The contract follows production: one variable. If a heading face is later introduced, add `--heading-font` to the tokens file as a deliberate, ratified change.

---

## 4. Icons — Font Awesome Pro

**RATIFIED and CHANGED from v2.** Production uses Font Awesome Pro. The live page renders the `fa-light` family predominantly, with `fa-regular` for a few controls (e.g. toggles, kebab `fa-ellipsis-vertical`). Markup is the standard FA pattern: `<i class="fa-light fa-circle-user"></i>`, which FA's JS upgrades to an inline `<svg class="svg-inline--fa fa-circle-user">`.

Prototype rule: **use Font Awesome Pro classes, default family `fa-light`, `fa-regular` only where production does.** The v2 inline-SVG-symbol mandate is retired; it produced icon markup that could never be lifted. Load FA via the production CDN/kit reference in the shell.

Sizing uses FA utility classes (`fa-lg`, `fa-xl`) and Bootstrap spacing utilities (`me-2`, `me-md-3`) exactly as the live page does, not inline width/height.

---

## 5. Layout patterns

### 5a. Entity Summary page (the dominant record pattern)

This is what the Request/Organization/User/Foundation Summary pages use. RATIFIED from `GLM.Summary.css` and the live page. The structure:

```
main#MainContent
  .page-header-container (d-flex justify-content-between)
    .flex-grow-1            ← title block
    #HeaderButtons          ← page-level actions
  .panel-layout (d-flex flex-wrap, height:100%)
    .side-panel (275px)     ← entity nav rail + info card
      .request-info-card    ← the contact/identity card
      .list-group.list-group-flush  ← nav items (Summary, Comments, Documents…)
    .main-content-body (flex:1)
      #ContentContainer
        .hero-card          ← amounts + status + stage tracker
        .row.g-3            ← card grid (col-12 col-xl-6 pairs)
          .card  .card  …
```

Key facts: the side panel is **275px**, contextual to the record (not a global app sidebar), and **collapses to a horizontal scroll-tab bar under 768px** (full responsive logic in `GLM.Summary.css`). The card grid is Bootstrap's grid: `.row.g-3` with `.col-12.col-xl-6` children, cards get `.h-100.mb-0` so paired cards match height.

### 5b. Card composition

Cards are MDB/Bootstrap `.card` > `.card-body`, header via a `.d-flex.justify-content-between.align-items-center.mb-3` containing an `<h5 class="card-title mb-0">` and an actions cluster. Card header and footer borders are removed (`mdb-customizations.css`: `.card-header{border-bottom:none}`, `.card-footer{border-top:none}`). RATIFIED.

Inside cards, repeating label/value rows use the `*-row` / `*-row-label` / `*-row-value` pattern (`award-detail-row`, `fin-row`, both flex space-between, 8px vertical padding, divider border-bottom, last-of-type no border).

> **GAP — duplicate row components.** `award-detail-row` and `fin-row` are byte-for-byte identical styling under two names. They should collapse to one shared label/value row component. Prototypes should use a single canonical class; see Section 9.

### 5c. Stat tiles

Two patterns coexist: `.data-card` / `.data-card-row` (auto-fit grid, `minmax(225px,1fr)`, 10px radius) in `GLM.Summary.css`, and the hero amount blocks (`.hero-amount-block` / `-label` / `-value`) inline on the page. RATIFIED both, but see Section 9 — these overlap conceptually.

---

## 6. Components — the remap pattern

**Ratified styling principle (confirmed with engineering).** Components are reskinned by remapping MDB's own `--mdb-*` variables onto Foundant tokens, not by writing bespoke override classes that reach around MDB. The `--mdb-*` layer is the indirection: a class reads MDB's variables, and those variables hold Foundant token values. The goal is fewer one-off classes, achieved by leaning on MDB's variable system. Prototypes follow this: compose from MDB/Bootstrap classes with `--mdb-*` remaps, and do not invent parallel component classes. (This is why the invented `ft-*`/`.btn-t` vocabularies were retired.)

Every styled component below follows that rule. Values RATIFIED from `mdb-customizations.css`.

### 6a. Buttons

Base `.btn` sets `--mdb-btn-border-radius: var(--radius-round)`, `--mdb-btn-font-size: 14px`, `text-transform: initial`.

| Variant | Class | Key remaps |
|---|---|---|
| Primary (filled) | `.btn.btn-primary` | bg `--colors-brand-primary-dark-blue`, hover `state-layers-dark-hover`, pressed `dark-pressed`, no box-shadow |
| Outline dark | `.btn.btn-outline-dark` | the workhorse on record pages, used with `.btn-sm.fw-bold` |
| Outline secondary | `.btn.btn-outline-secondary` | bg `neutral-100`, border + text `primary-dark-blue`, hover bg `light-hover` |
| Dark (filled alt) | `.btn.btn-dark` | used with `.btn-sm.fw-bold.shadow-none` for a card primary action |
| Danger | `.btn.btn-danger` | bg `danger-500`, hover `-600`, active `-700`, white text |
| Outline danger | `.btn.btn-outline-danger` | transparent bg, `danger-500` text/border, hover bg `danger-50` |
| Link | `.btn.btn-link` | all states `primary-dark-blue` |

Focus is global: `.btn:focus-visible{outline:solid 2px var(--colors-brand-primary-dark-blue);outline-offset:2px}`. RATIFIED.

**The record-page default action button is `.btn.btn-outline-dark.btn-sm.fw-bold`.** Card-level primary action is `.btn.btn-dark.btn-sm.fw-bold.shadow-none`. This replaces v2's `.btn-ft-sm`/`.btn-ft-default`.

**Icon usage in buttons — RATIFIED, evidence-backed.** Checked every real `.btn` in the production HTML sample: 19 total, 1 with an icon — and that one (`.kebab-btn`) is icon-*only*, not a text+icon combo. Every button that has a label is text-only in production today. The rule: **default to text-only**, with one deliberate exception — primary "create new" actions (`+ New`, `+ Add`), where the `+` is a near-universal affordance and these buttons recur often enough across list/table views to earn a scan-target. Everything else (Edit, Save, Cancel, Delete, Confirm, Submit, Download) is text-only, no exceptions. `.kebab-btn` remains its own separate icon-only pattern, never paired with a label. Applies to the Modals rebuild (Section 6c) in particular: footer buttons get no icons.

**Demo cleanup:** the kitchen sink originally showed three buttons here — `+ New` (the exception), a text-only `Edit` (illustrating the default), and the kebab (icon-only). The `Edit` counter-example read as a mistake rather than an illustration, so it was removed; the section now shows only the two real patterns (icon+text exception, icon-only), with the "everything else is text-only" rule stated in prose instead of demonstrated with a button.

**Frequency rule, added on direct instruction — decision, not evidence:** an icon+text button should appear **at most once per page**, reserved for a single primary action rather than a general-purpose style. The `+` specifically should only be used when the action is genuinely additive (creating something new) — not as decoration on an already-text-only button. Icon-*only* buttons (`.icon-btn`/`.kebab-btn`) are explicitly exempted from this restriction and can be used far more liberally (row menus, repeated table actions), since a compact icon-only control doesn't compete for attention the way a repeated icon+text combo would. Documented as a yellow/warning-toned callout in the kitchen sink to visually distinguish it from the ratified (green) exception-list decision right above it.

**Token-consistency correction:** an audit of every property value in `glm-prototype-foundation.css` (not just the button rules) found 6 instances of hardcoded `#fff` — the disabled text color on `.btn-primary` and every state color on `.btn-danger`. These were fixed to `var(--colors-neutral-100)`, which already resolves to the same white and is the established token, rather than a second, untracked way of writing white. Two other hardcoded values found in the same audit were deliberately left alone: `color: #000000` in the Validation section (Section 6n) is intentionally unmodified, since it's documenting real production CSS exactly as it ships; and the "OVERRIDDEN" status-tag colors are kitchen-sink reference-page chrome, not a GLM component, so they're outside the token system's scope entirely.

### 6a-i. `.ratified-note` — new documentation convention

New CSS class alongside the existing `.gap-note`/`.gap-zone` pair, for when the kitchen sink documents a decision that's settled rather than drafted. `.gap-note` is red/amber and hazard-striped by design — using it (even color-overridden) for a ratified decision was semantically backwards. `.ratified-note` is a plain success-green callout, no striped wrapper, since ratified content isn't a hazard. Added to `glm-prototype-foundation.css`.

### 6b. Badges

Base `.badge` sets `--mdb-badge-color: var(--colors-brand-primary-dark-blue)`. Pill badges use `.badge.rounded-pill` plus a semantic modifier and often `.fw-normal`.

`GLM.Summary.css` defines `.badge-success` / `-warning` / `-danger` / `-secondary` as pill badges (subtle `-50` background, primary or neutral text, uppercase, letter-spaced). RATIFIED.

> **GAP — badge family fragmentation.** v2 invented ten `ft-badge--*` status names (approved, manual, loi-draft, overdue, etc.). Production has four semantic `badge-*` classes. The *domain statuses* (LOI Draft, Overdue, Pending…) still need a home: they should map onto the four semantic backgrounds via a documented status→semantic table, not a parallel class family. This mapping is the badge governance task. See Section 9.

> **GAP — intentional shape/border departure, team-requested.** Prototype now ships badges rectangular (`--radius-sm`, not `.rounded-pill`) with uppercase/letter-spacing removed and a saturated `-0`/`-200` border added per variant, since pills read too similar to pill buttons and needed more contrast against white. This is a real, deliberate change to ratified production behavior, not a bug fix — flagged, not silent. Text stays uniform navy for now; colored text per variant is a separate open decision tied to the proposed WCAG ramps (Section 2c, backlog #4). Needs engineering review before it round-trips into `GLM.Summary.css`.

### 6c. Tables, forms, modals, toasts, dropdowns

- **Table**: `.table` sets `--mdb-table-bg: var(--colors-neutral-100)`. Header background white, divider borders. (Note: complex data grids use AG Grid, not `.table` — see Section 7.)
- **Form control**: `.form-control`/`.input-group` bg `neutral-100`. Input border = `--colors-borders-enabled`, focus border = primary. RATIFIED.
- **Modal**: header and footer border widths set to `0` (`--mdb-modal-header-border-width: 0`). Legacy Bootstrap-3 `.close` button hidden; use a Cancel action. RATIFIED.
- **Toast**: the global pattern is `#GlobalToast` — success-50 bg, 4px success-0 left border, 8px radius, transparent header/body. RATIFIED.
- **Dropdown**: standard MDB dropdown, `.dropdown-menu.dropdown-menu-end`; nested via `.dropstart` + `.dropdown-item.dropdown-toggle`. Interactions are `data-mdb-*` driven. RATIFIED from live page.

### 6d. Side panel nav

`.side-panel` 275px, `.list-group-item.list-group-item-action.ripple`, active = `--colors-brand-secondary-light-blue` background + bold. Section headers `.nav-group-header` (12px, 700, uppercase, neutral-30). `.ft-label` is the small-caps identity label used in the info card. RATIFIED, confirmed against the live Request Summary markup: `.panel-layout` > `.side-panel` (`.nav-fade-left`/`-right` mobile-collapse overlays, `.request-info-card`, `.list-group.list-group-flush`) beside `.main-content-body`. The captured page's nav is flat (Summary/Interactions/Documents/Follow Ups/Financials & Compliance); `.nav-group-header` exists in CSS for grouped variants but isn't exercised on this specific page.

> **GAP — `.request-info-card` background is inline, not a class rule.** The card's `--colors-neutral-70` background is set via an inline `style` attribute in the Razor markup rather than a CSS rule, unlike every other component in this contract. Token reference is correct, but the implementation bypasses the class layer. See Section 9.

### 6e. Navbar

RATIFIED structure, confirmed from the live Request Summary page markup (not inferred from CSS alone): `header#MainHeader.fixed-top` > `nav.navbar.navbar-expand-lg` > `container-fluid`, containing brand logo, a left-side icon link, a right-side cluster, and a collapsible `#MainNavigation` mega-menu. Standard MDB5 throughout (`data-mdb-collapse-init`, `data-mdb-dropdown-init`) — no bespoke navbar classes.

**Three access tiers share one shell, subtracting rightward:**
- **Super admin:** left = Dashboard icon. Right = `#roleToggle` Role dropdown (switches Administrator/Applicant view) + avatar dropdown. `#externalLinks` is suppressed whenever `#roleToggle` is present.
- **Client/admin:** same shell, no Role dropdown. `#externalLinks` (Compass / Idea Lab / Release Notes / Resources icons) shows in its place. `#TopLinks` mega-menu present.
- **Applicant:** simplified to a couple of left-side icon links plus the avatar dropdown only. No `#TopLinks`, no Role dropdown, no external-resources row.

**Header background is an org-brandable slot, not a fixed value.** Foundant ships `header, footer { background-color: var(--colors-neutral-100) }` as the default (confirmed in `mdb-customizations.css`). Individual foundations then inject their own background/text color via a per-tenant `<style>` block loaded *after* the standard CSS stack — the live capture used for this contract shows one org overriding to a custom brand blue with white nav text. This is a legitimate, working pattern, but it is ad hoc CSS per tenant today rather than a documented token or mechanism. See Section 9 backlog.

User dropdown (`#UserDropdown`, all tiers): avatar icon, org-name header row, Edit Profile, Preferences, Edit Organization, divider, Sign Out. Stock MDB dropdown, no override.

### 6f. Alerts

GAP — no `.alert` override exists anywhere in production; ships as pure stock MDB today. Prototype foundation CSS drafts `.alert-success`/`-warning`/`-danger`/`-info`, mirroring the badge convention (Section 6b): a pale tint background with `--colors-brand-primary-dark-blue` text, since only the danger ramp is WCAG-validated past `-50` (Section 9 backlog #4). `alert-success`/`-warning`/`-danger` use `-50` fill + `-0`/`-200` border. `alert-info` uses `-90` fill + `-50` border instead — `info-50` is noticeably more saturated than `success-50`/`warning-50` and duplicates `--colors-brand-secondary-light-blue` (the sidenav active-state color), so `-90` was used to match the paleness of its siblings and avoid borrowing that color's "selected" meaning.

**Confirmed cascade finding, generalizable beyond alerts:** the initial draft only set `--mdb-alert-bg`/`-color`/`-border-color`, which had zero visual effect — MDB's stock `.alert-*` rules paint `background-color`/`color`/`border-color` directly rather than solely through custom-property indirection, so overriding only the `--mdb-*` variables silently did nothing. Confirmed by inspecting MDB 7.1.0's source directly. Fix was to set the rendered properties directly in addition to the custom properties. **Any future component override should verify the rendered output actually changed, not just assume `--mdb-*` variable overrides took effect** — this may not be alert-specific.

Needs engineering review before adoption.

### 6g. Tabs & pills

GAP — no `.nav-tabs`/`.nav-pills` override exists anywhere in production; ships as pure stock MDB today. Prototype foundation CSS drafts active/hover states matching the state-layer + primary-dark-blue convention already ratified for `.kebab-btn` and the navbar hover fix (Section 6e backlog #11), rather than inventing a new hover language. Needs engineering review before adoption.

**Update:** engineering has now shipped tabs on upgraded pages, built from this draft. One discrepancy confirmed by screenshot — production still renders MDB's stock uppercase `text-transform`, which this draft explicitly removes. Direction confirmed: production should match the kitchen sink, not the reverse. Awaiting confirmation the fix has landed before flipping this section's flag to OVERRIDDEN.

### 6h. Text inputs & select

GAP — confirmed structural pattern: GLM uses a plain top-label field (`.form-label` above `.form-control`), not MDB's notched floating-outline (`.form-outline`/`.form-notch`). The only production remap confirmed today is `.form-control` background (`--colors-neutral-100`). Border and focus color are proposed here, using the dedicated `--colors-borders-enabled`/`--colors-borders-focused` tokens (Section 2e) rather than `--colors-brand-primary-dark-blue` directly, since they're the more semantically correct remap target even though the values currently match.

**Label color, specifically:** deliberately lighter than body text, not matched to it. `mdb-customizations.css` sets `--mdb-body-color` to `--colors-brand-primary-dark-blue` globally, so all unstyled text — including what gets typed into the field — already renders navy by inheritance. Production already uses this exact light-label/dark-value hierarchy in two real places: `.detail-label` + `.detail-value`, and `.summary-card dt` + `dd`, where the label gets an explicit gray (`--colors-neutral-30`) and the value is left to inherit navy. This label follows that same established pattern rather than inventing a fourth caption treatment, and was sent to engineering directly as the answer to their own question about `.form-label` looking "faded" under stock MDB.

**Font-weight and margin** are intentionally not set in the CSS rule at all — they're applied as `fw-normal`/`mb-1` utility classes in markup instead, per the already-ratified "spacing uses Bootstrap `m-*`/`p-*` utilities" convention (Section 5). **`font-size: 0.75rem`** has no token to point at — `foundant-tokens.css` defines zero font-size tokens today (backlog #7), and production's own `.detail-label` hardcodes this identical raw value with no token either, so matching it is evidence-accurate rather than a shortcut.

Confirmed against `mdb_min.css` (now in this project): the base border does use a variable (`--mdb-border-color`), but it's shared across many unrelated components, so scoping the override to `.form-control`/`.form-select` directly is safer than remapping that variable globally. The winning focus rule in `mdb_min.css` is a hardcoded hex (`border-color:#3b71ca`), not a variable, at the same specificity as this override — confirming the direct-property approach (following the Alerts/Tabs precedent) rather than a guess. `.form-select` has no production evidence of its own — no override exists, and the one production HTML sample has no `<select>` to check — so it's folded into the same border/focus rule purely to keep the field family visually unified, not because a select-specific value was confirmed. Needs engineering review before either round-trips into `mdb-customizations.css`.

**Resolved:** validation is confirmed live and split across two legacy classes — `.field-validation-error` (per-field) and `.validation-summary-errors` (bottom-of-form summary) — confirmed directly with engineering ("field level and bottom summary"), not just inferred from CSS. See Section 6n for the full writeup, including a proposed modernization, since the team is planning to modernize form completion rather than just port the legacy system forward. Backlog #22 is resolved by this; see backlog #25 for the new proposal's status.

### 6i. Checkbox, radio & switch

GAP — no `.form-check-input` override exists anywhere in production; checked/focus state ships as pure stock MDB blue today. This is a standalone proposal, not an extension of a sibling remap the way Select was — the first customization proposed for this component family. Note: the prior kitchen sink draft's claim that "checked color follows theme primary" was not supported by any evidence and has been corrected, not carried forward. One CSS rule covers checkbox, radio, and switch together since MDB's switch reuses `.form-check-input` with a `.form-switch` parent class rather than a distinct input class.

**Corrected against `mdb_min.css` (now in this project):** a first attempt at this fix was wrong on two counts, both about specificity, not load order. **Checkbox's** real checked-state rule is `.form-check-input[type=checkbox]:checked` — the attribute selector makes it more specific than a plain `.form-check-input:checked` rule, so MDB's blue won outright regardless of which stylesheet loaded last. **Radio's** visible dot isn't the base element's background at all — it's a `::after` pseudo-element (`.form-check-input[type=radio]:checked:after`), untouched by any rule on the base element. Both are now matched at the correct selector. **Switch** was already correct from an earlier pass — MDB colors the knob via `::after` at higher specificity, not the track; the focus ripple around a checked switch knob is a separate hardcoded hex, overridden the same way. **Border width** is 2px in MDB's own later, Material-style `.form-check-input` rule (`border: .125rem solid`) layered on top of the Bootstrap base; overridden to 1px here at equal specificity, per design direction. An earlier version of this fix guessed `accent-color` as the mechanism; confirmed via `mdb_min.css` that MDB doesn't use `accent-color` anywhere, so that guess was removed rather than left in as dead code.

**Retested after deploying — found two more of the same specificity pattern, this time on the focus state specifically.** MDB has separate, more specific rules for the checked+focus combination that the plain `:checked` fixes above never reached: `.form-check-input:checked:focus{border-color:#3b71ca}` (radio's outer ring) and `.form-check-input[type=checkbox]:checked:focus{background-color:#3b71ca}` (checkbox's fill). Since clicking a control focuses it automatically, this was visible immediately, not an edge case — worth remembering for any future MDB component work: check `:focus` combos specifically, not just the base interactive state, since MDB frequently ships a more-specific rule for the two combined. Both now matched at the same specificity. Confirmed via targeted `grep` against every `:checked` + `:focus` combination in `mdb_min.css` for this component, not just the two that were visibly wrong — switch's knob and ripple were re-verified and confirmed already correct, and no separate track-background rule exists for switch that needed a matching fix. Needs engineering review before it round-trips into `mdb-customizations.css`.

### 6j. Input groups

GAP, with real ratified pieces — radius-joining (`.input-group > :not(:first-child) > .form-control`) and sizing (`.input-group > div > .form-control`) are confirmed real overrides in `mdb-customizations.css`, not drafted.

**Bug found while building this section:** MDB's own `.input-group > .form-control:focus` selector is more specific than the plain `.form-control:focus` rule proposed in Section 6h (Text Inputs), so that rule's navy focus color never actually reached an input sitting inside an input-group — it silently kept showing MDB's hardcoded blue there. Matched at the same specificity to fix it, using the `inset` box-shadow style MDB itself uses in this context (rather than the outer-ring style used for standalone inputs). This is the same class of bug as the checkbox/radio specificity issue in Section 6i — a reminder to check every context a base-level rule needs to reach, not just the most obvious one.

**Separate, still-open leftover:** `.input-group > .autocomplete > .form-control:focus` hardcodes `border-left: 1px solid #3b71ca !important`, specific to autocomplete nested in an input-group. Not fixed here — belongs with the Autocomplete section once that's built, since it needs the same specificity treatment plus confirmation of the autocomplete markup itself. Needs engineering review before either round-trips into `mdb-customizations.css`.

### 6k. Datepicker — direction ratified, real markup still needed

**Confirmed:** production loads `bootstrap-datepicker.js` (the older eternicode jQuery plugin), not MDB5's native Datepicker — verified via the production HTML's script tags. Zero CSS customization of it exists anywhere in `mdb-customizations.css` or `GLM.Summary.css`. Separately confirmed: MDB5 Pro genuinely ships its own, entirely different native Datepicker component (`.datepicker-cell`, `.datepicker-header`, `.datepicker-main`, `.datepicker-footer`, etc. — all real, present in `mdb_min.css`), sharing no markup or class names with bootstrap-datepicker.

**Decision:** theme what's actually live — bootstrap-datepicker — rather than migrate to MDB5's native component right now. The native component is logged here as a future option, not a current plan; if a migration happens later, it's a separate rebuild from scratch, not an extension of the bootstrap-datepicker work, since the two share nothing.

**Still blocked, and this is the reason the kitchen sink section isn't built yet:** bootstrap-datepicker has been forked/modified enough across versions and projects that its exact rendered class structure in this specific build can't be safely assumed from memory or general knowledge of the library. The prior kitchen sink draft was a fully custom-styled fake calendar matching neither library — replacing it with a guess at bootstrap-datepicker's structure would repeat the same mistake with different wrong details. Needed before rebuilding: a screenshot or DOM/rendered-HTML capture of the picker actually open somewhere in the app.

### 6l. Autocomplete

GAP, with real ratified pieces — unlike Datepicker, this component is genuinely confirmed in production use: the header search box (production HTML: `id="HeaderSearchBox" class="form-control ... autocomplete-input"`) and a dedicated org-search instance, confirmed by a real, ID-scoped override (`#autocomplete-dropdown-organizationSearch .autocomplete-dropdown { --mdb-autocomplete-dropdown-box-shadow: none; }`) plus the global `.autocomplete-loader { display: none; }`.

Everything past those two confirmed rules is proposed, not confirmed: item hover/active background and label color are stock MDB blue (`#3b71ca`) today, remapped here to established Foundant tokens rather than left alone or guessed at with new values.

**Markup note:** the one real example (header search) uses MDB's notched floating-outline structure (`.form-notch`) — but that's a navbar-specific context, not the standard form pattern. A form-context autocomplete (e.g. searching for an organization inside a normal create/edit form) follows the already-ratified plain top-label pattern from Section 6h, same as every other field, rather than copying the navbar's notch style.

**Also fixed here:** the input-group-nested-autocomplete focus-border leftover flagged back in Section 6j (`.input-group > .autocomplete > .form-control:focus`, hardcoded `#3b71ca !important`) — matched at the same specificity plus `!important` to override it, since the original rule already forces `!important` and a later, non-`!important` rule can't beat it regardless of specificity. Needs engineering review before any of this round-trips into `mdb-customizations.css`.

### 6m. Stepper

GAP, DRAFT — real MDB5 component, confirmed present in `mdb_min.css`, but with two things worth being explicit about. First: **no evidence GLM uses this anywhere in production** — nothing in `mdb-customizations.css`, `GLM.Summary.css`, or the production HTML sample references it. This section documents the correct native component to reach for if a multi-step flow gets built, not a confirmed existing pattern — same posture as the Ghost button before it was removed, except this one's worth keeping since it's a real MDB component, not an invented class.

Second, and more structurally significant: **MDB5's Stepper is vertical-only.** There's no `.stepper-horizontal` class anywhere in `mdb_min.css`. The prior kitchen sink draft was a horizontal row of connected icons — that layout has no equivalent in the real component at all, so the mismatch wasn't just wrong colors, it was a fundamentally different structure. The closest real variant is `.stepper-mobile`, a condensed horizontal progress-bar look, but it's a small-screen responsive fallback, not a general desktop layout choice.

State-icon colors (`.stepper-completed`, `.stepper-active`, `.stepper-invalid`, `.stepper-disabled`, each targeting `.stepper-head-icon`) are proposed, remapped from MDB's own stock `-bg-subtle`/`-text-emphasis` tokens to Foundant equivalents. `--colors-semantic-success-700` isn't a ratified token yet (danger has the full 50–900 ramp; success is still proposed, Section 2c) — used with a fallback to `--colors-brand-primary-dark-blue`. `--colors-semantic-danger-700` is real and ratified, used directly. Needs a team decision on whether to build this at all before it's a real candidate for `mdb-customizations.css`.

### 6n. Validation — confirmed current system, plus a proposed modernization

**Confirmed live, directly with engineering** (not inferred from CSS alone): *"I would just go into an application and look at it — it's field level and bottom summary."* This maps exactly to the two legacy classes already found in `mdb-customizations.css`: `.field-validation-error` (per-field) and `.validation-summary-errors` (bottom-of-form list). Both share identical box styling — `border: 2px solid #cd0a0a`, `color: #000000 !important` — hardcoded, not tokenized at all. Documented as-is in the kitchen sink; not smoothed over or partially tokenized to look better than it is.

**Also confirmed real and already tokenized:** `.form-group .fa-circle-check { color: var(--colors-semantic-success-0) }` and `.form-group .fa-circle-exclamation { color: var(--colors-semantic-danger-0) }` — validation-adjacent icon classes that already exist and already use Foundant tokens, just not currently paired with a modern message treatment.

**Engineering, same conversation:** *"the form completion is something we're planning to modernize not just port to modern so open to change."* This changes the job here from "document and tokenize the legacy system" to "propose a real alternative for discussion." The proposal in the kitchen sink:

- Per-field: real Bootstrap `is-invalid` + `.invalid-feedback`, remapped to `--colors-semantic-danger-0` instead of MDB's stock red.
- Summary: reuses `.alert-danger` exactly as already proposed in the Alerts section (Section 6b) — not a separate invention, so if Alerts gets ratified, this summary treatment is already consistent with it.
- Icons: reuses the real, already-tokenized `fa-circle-exclamation`/`fa-circle-check` classes rather than inventing new iconography.

This is explicitly a discussion-starter, not a decision. Needs actual team input on the modernization direction, then engineering review, before any of it becomes a candidate for `mdb-customizations.css`.

### 6o. Modals — confirmed stock, no gap to propose

Confirmed via a real modal's full outerHTML plus its DevTools Styles panel checked side by side (the Collaborate dialog — inviting someone to a grant record). Every rule on `.modal-content`, `.modal-header`, `.modal-footer`, and `.btn-close` traces to `mdb.min.css`. None trace to `mdb-customizations.css`. Modals are entirely stock MDB in production today — this section documents that fact rather than proposing a remap, since there's genuinely nothing to remap.

**One real nuance worth keeping:** the checked modal adds `.border-bottom` back onto its `.modal-header` as a utility class. The global default (confirmed separately in `mdb-customizations.css`: `--mdb-modal-header-border-width: 0`) is borderless, but this specific modal opts back into a divider at the markup level. Both facts are true at once — there's no contradiction, just a per-instance choice layered on top of a global default. Worth remembering when building new modals: the borderless default doesn't mean every modal has to be borderless.

**Confirms three things already established or found here, not separate inventions:**
- Footer buttons in the real modal are text-only ("Close"/"Invite") — already matches the ratified icon-usage rule (Section 6a) without needing a fix.
- The real modal's hidden `#collaborateError` validation box uses `class="alert alert-danger"` — the exact same class already proposed in Alerts (6b) and reused in the Validation modernization proposal (6n). One consistent pattern across all three, not three separate inventions.
- **Footer layout:** the real modal uses `class="modal-footer flex-body justify-content-between"` — `justify-content-between` splits Cancel to the left and the primary action to the right, overriding stock `justify-content: flex-end` (which would otherwise cluster both buttons together on the right). Applied to the kitchen sink's Confirm Award example. `flex-body` is also in the real markup but isn't defined anywhere in `mdb.min.css` or `mdb-customizations.css` — kept in the code sample for fidelity to what production actually ships, but confirmed to do nothing.

**Decision, not evidence, layered on top of the above:** Modal titles and Card titles both now use a real `<h2>` tag, unsized down. Real production actually renders `<div class="modal-title h5 w-100">` in the checked Collaborate modal, and Cards previously used `<h5 class="card-title">` in the kitchen sink, matching the common Bootstrap convention — this changes both, on direct instruction, not new evidence. Neither `.modal-title` nor `.card-title` sets its own font-size in MDB (only margin-bottom, line-height, color), so swapping the tag is enough — no additional CSS needed, and none was added.

**Interactive demo, upgraded from a static preview:** the Modals section now has a real "Pop the modal" trigger plus a working `.modal-backdrop` scrim, toggled with a small helper script since this reference page doesn't load MDB's JS bundle — the code sample still shows the real `data-mdb-dismiss="modal"` markup you'd actually ship, kept deliberately separate from the demo's `onclick` handlers. Because the demo now uses the real `.modal > .modal-dialog > .modal-content` structure instead of skipping straight to `.modal-content`, `--mdb-modal-padding` (defined on `.modal` itself) cascades down to `.modal-body` correctly without any manual workaround.

**Dependency worth flagging:** H2's current value (`1.25rem`/20px, `font-weight: 500`) is itself part of the same disputed v2 typography scale flagged in Section 2 (Typography) — only H1's real winning override (24px) has been separately confirmed; H2–H6 haven't been through that same confirmation pass. Modals and Cards will inherit whatever H2 ends up being once that's resolved, which is the point of using the real tag instead of a local override — but it means this decision's visual result isn't fully settled yet either.

### 6p. Icons — usage reference, plus a standardized icon button

**Icon usage reference, built from real evidence, not recollection.** Font Awesome's Kit JS leaves the original `<i class="...">` as an HTML comment immediately next to the SVG it renders — that's how every entry in the kitchen sink's reference table was pulled, from one production page (the Event Request record view). Over 30 distinct icons confirmed, grouped by context: global nav, user/account menu, left nav tools menu, record nav rail tabs, a recurring row-action pair (`fa-arrow-up-right-from-square` + `fa-file-pdf`, always together as "View" + PDF download), stage/progress indicators, and the already-ratified semantic-color icon rules (`fa-circle-check`/`fa-circle-exclamation`/`fa-triangle-exclamation`/`fa-trash-can:hover`).

**Explicitly flagged as partial, not exhaustive:** this covers one page sample. Gaps in the table mean "not yet confirmed," not "doesn't exist" — update it as more pages get checked, same as any other evidence-based section here.

**Real find: `.kebab-btn` has no CSS rule anywhere.** Confirmed via the actual production markup: the class name is a JS/dropdown hook only. Every real instance repeats the identical look through a copy-pasted inline `style=""` — `background: none; border: 1px solid var(--colors-borders-divider); border-radius: 4px; padding: 0px 5px; color: var(--colors-brand-primary-dark-blue)`. That's the standardization gap the icon-button request was actually about: the values are real and consistent today, but nothing enforces that consistency going forward.

**Correction to an earlier guess:** `.kebab-btn` already had a CSS rule in `glm-prototype-foundation.css` from the Buttons work (Section 6a), but it was an unverified guess at the time — no border, `--radius-sm`, `--colors-neutral-40` text. That didn't match the real inline-style values once actually checked. Corrected in place, both in the foundation CSS and the Buttons section's code sample, rather than left standing alongside the new correct version.

**Proposed: `.icon-btn` as the general-purpose name.** Same real values as `.kebab-btn`, exactly — nothing invented in the base rule. Proposed as a second, more general class name (not a replacement) so a future icon-only button for something other than a dropdown trigger (delete, edit, download) has a real class to reference instead of a sixth copy-pasted inline style. `:hover` matches an already-established pattern (`--colors-state-layers-light-hover`); `:disabled` is genuinely new, since no disabled instance exists yet to confirm against — flagged as such. Needs engineering review before either rolls into `mdb-customizations.css`, since adopting it broadly means replacing inline styles at every existing call site.

**Second pass: WCAG 2.2 AA target-size compliance, requested directly.** The version above had no explicit width or height — sizing fell out incidentally from `padding: 0px 5px` plus whatever font-size and line-height happened to be inherited from context. In the one real instance checked (kebab-btn in a card header) that likely cleared 24px, but nothing guaranteed it: the same markup dropped into a denser context with smaller inherited text could fall under the WCAG 2.2 SC 2.5.8 (Target Size Minimum, Level AA) floor of 24×24 CSS pixels with no warning. None of the narrow exceptions to that criterion (inline text, essential presentation, an equivalent control elsewhere, adequate spacing) apply to a standalone icon button.

**Fix, second pass:** explicit `width: 32px; height: 32px` as the default — comfortably above the 24px floor, not just coincidentally at it — plus `display: inline-flex; align-items: center; justify-content: center` to keep the icon centered regardless of its own intrinsic size. A `.icon-btn-sm` modifier provided the exact 24×24px legal minimum for genuinely dense contexts. `:focus-visible` was also added — reusing the exact outline ring already ratified for regular buttons (Section 6a) rather than inventing a second focus treatment — since the prior version had no visible focus state at all, a separate accessibility gap (WCAG 2.4.7/2.4.11).

**Third pass, corrected on direct instruction:** 32px read too heavy in dense UI, table rows specifically. Default is now 24×24 everywhere — the real WCAG floor itself, not a comfort margin above it — and `.icon-btn-sm` is gone, since there's no longer a larger default to shrink down from. The border from production's real `.kebab-btn` instance was also removed on direct instruction: WCAG 2.5.8 governs the tappable *area*, not visual styling, so a visible border was never actually a compliance requirement — it was just what one real production instance happened to look like. Hover (`--colors-state-layers-light-hover`) and `:focus-visible` now carry the visual affordance at rest and on keyboard focus instead of a static border.

**Also documented as a hard requirement, not a suggestion:** every icon-only button needs a real `aria-label`. There's no visible text for assistive tech to announce, so without one the control has no accessible name (WCAG 1.1.1, 4.1.2, both Level A) — `title` alone doesn't reliably substitute. Every example in the kitchen sink now includes one; the two pre-existing `.kebab-btn` demo instances (Buttons section) were retroactively fixed to match rather than left inconsistent with the new standard.

**`.kebab-btn` and `.icon-btn` still share one rule**, so this fix applies to both without needing two copies to keep in sync. The Buttons section's own code sample now points to this section rather than repeating the CSS a third time, to avoid the exact kind of drift this whole exercise was meant to eliminate.

---

## 7. AG Grid

**GAP — this is the largest un-Foundanted surface in the system.** The shipped AG Grid theme is **stock Alpine**: `--ag-foreground-color: #000`, `--ag-background-color: #fff`, `--ag-font-family: 'Helvetica Neue'`, `--ag-active-color: #2196f3`, `--ag-border-radius: 0px`. The only Foundant remap is a single `--ag-font-family: var(--body-font)` in `mdb-customizations.css`, which the stock theme then overrides downstream.

So data grids — the pages where you said AG Grid is central — currently render in colors and a typeface that match nothing else in GLM. This is the clearest, highest-impact instance of the consistency problem the design system exists to solve.

**Prototype resolution.** Until a Foundant AG Grid theme exists, prototypes that need a grid should apply an explicit Foundant theme override block that remaps the core `--ag-*` variables onto tokens: foreground → `primary-dark-blue`, font → `--body-font` (enforced), header background → `neutral-90`, borders → `borders-divider`, active/selected → `brand-secondary-light-blue`, row hover → `state-layers-light-hover`, font-size 14px, radius from the scale. This block becomes the draft of the real production theme. It is the natural second deliverable after this contract, and it is worth its own working session because the variable surface is large (200+ props) and several need WCAG validation the way the danger ramp got.

---

## 8. JavaScript conventions

MDB handles modal, toast, dropdown, accordion, and tab via `data-mdb-*` attributes — no custom JS. Custom JS uses `var` and classic `for` loops, no arrow functions, no frameworks. RATIFIED (matches both v2 and production constraints). Font Awesome's JS upgrades `<i>` to inline SVG at runtime; prototypes rely on this rather than hand-writing SVG.

---

## 9. Completion backlog (the governance to-do)

These are the live gaps and inconsistencies the production files reveal. The design system's job is to close them in the real codebase, not to copy them forward. Listed roughly by leverage.

1. **AG Grid is unthemed (stock Alpine).** Build and validate a Foundant AG Grid theme. Highest impact; see Section 7.
2. **Only the danger semantic ramp is complete.** Success, warning, info need full 50–900 ramps through the same WCAG process. `foundant-tokens.css` flags this in a comment. **Now also: a generated secondary ramp** (`--colors-brand-secondary-light-blue` tinted 0–90, matching primary's existing formula) is proposed alongside these — see Section 2a. All four (success/warning/info/secondary) share the same status: drafted in the kitchen sink, not yet in `foundant-tokens.css`, pending team review.
3. **`--colors-semantic-success-30` is used but never defined.** `GLM.Summary.css` `.settings-overview-overridden` references it; it resolves to nothing. Either define it (part of the success ramp work) or change the reference.
4. **No `6px` radius token, yet cards/hero use 6px.** Page CSS hardcodes `border-radius:6px` while the scale offers only 4/8/round. Cards (6px), `.page-container` (4px), `.data-card` (10px) all differ. Reconcile to a documented card-radius token. **Discrepancy found while adding CSS examples to the kitchen sink (Cards/Spacing sections):** searched all three production evidence files in this project — `mdb-customizations.css`, `GLM.Summary.css`, and the raw `Event_Request_...html` — for a literal `border-radius: 6px` and found zero matches anywhere. The only place 6px is actually applied is `.hero-card` in the prototype's own foundation CSS; generic `.card` has no radius override and renders on MDB's stock 8px. This may mean the original "cards hardcode 6px" claim came from a live DevTools/screenshot session that predates these evidence files, or it may mean the claim was never quite right for generic `.card`. Not resolved here — flagging so it gets reconciled with real evidence rather than carried forward silently. The Cards and Spacing sections of the kitchen sink have been corrected to state only what's actually verifiable today (hero = 6px via token, card = stock 8px, unremapped).
5. **Duplicate row components.** `award-detail-row` and `fin-row` are identical; collapse to one. Same for the conceptual overlap between `.data-card` and `.hero-amount-block`.
6. **Badge status mapping is undefined.** Domain statuses (LOI Draft, Overdue, Pending, Approved, Cancelled…) need a documented status→semantic-background table on top of the four `badge-*` classes, replacing the retired `ft-badge--*` family.
7. **Heading scale `h2`–`h6` undocumented.** Falls through to MDB defaults. Decide and ratify, or explicitly accept MDB defaults.
8. **Two icon families render (`fa-light` + `fa-regular`).** Confirm this is intentional (it likely is — regular for filled-state controls) and document when each is used.
9. **`success-30` aside:** several one-off hex values appear inline in the hero/page CSS (e.g. `#536E8A`, `#F4C862`, `#2D3B4A` in `.content-card`) that duplicate existing tokens. Replace with token references.
10. **~~Disabled buttons fall back to MDB default colors.~~ RESOLVED.** The disabled remaps (`--mdb-btn-disabled-bg`, `-color`, `-border-color` per variant) were added to the prototype foundation CSS and are being adopted in production's `mdb-customizations.css`. Disabled buttons now fade the Foundant color, not MDB's stock blue.
11. **Navbar top-level hover uses hardcoded hex (proposed resolution, not yet adopted).** `.navbar-nav > li > a:hover` is a gray gradient (`#eee`→`#e4e4e4`) with a 5px solid `#222` border-bottom — the last hardcoded-hex hover in the system, and inconsistent with the stock-MDB dropdown-item hover one level down in the same menu. Prototype foundation CSS now applies `background-color: var(--colors-state-layers-light-hover)` + `--radius-sm` instead, matching the already-ratified `.kebab-btn` treatment. Needs engineering confirmation before ratifying.
12. **Navbar dropdown toggles carry a redundant caret.** Markup pairs MDB5's native `.dropdown-toggle::after` arrow with a leftover Bootstrap-3 `<span class="caret">`. `.caret` has no rule anywhere in production CSS, so this likely renders a duplicate indicator. Needs confirmation and, if confirmed, removal of the dead `<span class="caret">` markup.
13. **Header/footer brand color is ad hoc per-tenant CSS, not a documented mechanism.** The org-brandable header slot (Section 6e) works today via each foundation injecting its own `<style>` override after the standard stack. There's no token, variable, or documented contract for what's themeable (background only? text too? logo sizing?) or how an implementer should apply it consistently.
14. **`.request-info-card` background is set inline, not via a class rule.** Unlike every other ratified component, its `--colors-neutral-70` background lives in a `style` attribute on the Razor markup itself (Section 6d). Correct token, wrong layer — should move into a real CSS rule.
15. **~~`.btn-ghost` draft.~~ REMOVED, not pursued.** Was proposed as a pill-shaped, transparent-at-rest borderless button matching `.kebab-btn`'s hover treatment. Pulled from the kitchen sink, foundation CSS, and this contract at the team's request — not carried forward as a backlog item.
16. **Kitchen sink is running the retired v2 heading scale — flagged, not yet fixed, pending team decision.** Section 3 already documents the real state: `h1` = 24px (RATIFIED, `mdb-customizations.css`) plus `h1.OrganizationSelector` = 32px (RATIFIED special case); `h2`–`h6` are an undocumented gap, falling through to MDB defaults. Despite that, `glm-prototype-foundation.css` still ships the retired v2 `--h1-size` through `--h6-size` scale (h1 = 1.75rem/28px), and the kitchen sink's Typography section presented it as settled fact until this entry. Cross-checked against three real surfaces: Request Summary page renders h1 at 24px (matches ratified), an Organization record page renders h1 at 32px (pending confirmation that this is the ratified `.OrganizationSelector` case, not a fourth value), and the kitchen sink itself renders 28px (the retired scale, not a real production value). **Team decision needed:** formally design and ratify a full h2–h6 scale into `foundant-tokens.css`, or strip the invented scale out entirely and leave h2–h6 on MDB defaults until one is deliberately designed. Do not resolve by simply picking one of the three numbers — h1 itself is already answered; the open question is whether a broader scale should exist at all.
17. **`.alert` variants are a draft, not lifted from production evidence.** No alert customization exists anywhere in production — ships as stock MDB today. Section 6f drafts `-success`/`-warning`/`-danger`/`-info` mirroring the badge color convention. Needs engineering review before it round-trips into `mdb-customizations.css`.
18. **`.nav-tabs`/`.nav-pills` are a draft, not lifted from production evidence.** No tab/pill customization exists anywhere in production — ships as stock MDB today. Section 6g drafts active/hover states matching the `.kebab-btn`/navbar hover convention. Needs engineering review before it round-trips into `mdb-customizations.css`.
19. **`--radius-card` exists only in the prototype, never round-tripped into `foundant-tokens.css`.** Real cards and the hero hardcode `6px`; the prototype named it but the actual token file was never updated to match. Section 2f.
20. **Four unused legacy tokens in `foundant-tokens.css`: `--colors-link-default`, `--colors-link-hover`, `--color-primary`, `--color-primary-hover`.** Leftover from the older `foundant.css` this file mirrors. Not referenced anywhere in current production CSS. One of them (`--colors-link-default`) had been mistakenly used in the prototype's own link-color remap instead of the correct `--colors-brand-primary-dark-blue` — now fixed. Candidates for removal from the token file entirely. Section 2f.
21. **`glm-prototype-foundation.css` contains a manually-copied mirror of `foundant-tokens.css`'s values, not a live reference.** The deployed kitchen sink (`index.html`) only links `glm-prototype-foundation.css` — it never loads `foundant-tokens.css` directly. All base `--colors-*` tokens are duplicated into the foundation file's own `:root` block alongside the component remaps, for single-stylesheet deployment simplicity. This means the two files can silently drift: if production's `foundant-tokens.css` changes, nothing in the prototype updates automatically. No build step currently keeps them in sync.
22. **`.form-control`/`.form-select`/`.form-label` are a draft, not lifted from production evidence — and validation system is unresolved.** No border/focus/label customization exists anywhere in production; only `.form-control` background is confirmed, and `.form-select` has no evidence of its own at all. Section 6h drafts the rest, and separately flags that two validation-messaging systems exist in `mdb-customizations.css` (`is-invalid`/`invalid-feedback` vs. legacy `.field-validation-error`/`.validation-summary-errors`) with no confirmation yet on which is canonical. Blocks the Validation section and the invalid/valid states within Text Inputs. Needs engineering review before either round-trips into `mdb-customizations.css`.
23. **`.form-check-input` checked/focus color is a draft, not lifted from production evidence.** No customization exists anywhere in production — checked state ships as stock MDB blue today, not brand navy. Section 6i proposes the remap. The prior kitchen sink draft incorrectly claimed checked color already followed theme primary; that claim has been corrected. Needs engineering review before it round-trips into `mdb-customizations.css`.
24. **`.input-group > .form-control:focus` needs its own specificity-matched fix, separate from Text Inputs.** MDB's own selector at this nesting level is more specific than the plain `.form-control:focus` rule, so the Text Inputs focus-color fix (backlog #22) never reached inputs inside an input-group. Fixed in Section 6j at matching specificity. A separate, still-unfixed leftover exists for autocomplete nested in an input-group (`.input-group > .autocomplete > .form-control:focus`, hardcoded `#3b71ca`) — deferred to whenever Autocomplete gets built. Needs engineering review before either round-trips into `mdb-customizations.css`.
25. **Validation modernization proposal (Section 6n) needs actual team discussion, not just engineering review.** Unlike most GAP items, this one isn't "lift the evidence, propose the obvious remap" — it's a genuinely open design question the team explicitly wants to reconsider (engineering: form completion is being modernized, not ported forward). The current legacy system (`.field-validation-error`/`.validation-summary-errors`) is accurately documented and confirmed live; the proposed replacement (`is-invalid`/`.invalid-feedback` + `.alert-danger` summary) is one starting point, not the only option. Don't treat this as settled just because a proposal exists in the kitchen sink.
26. **`.icon-btn` standardization (Section 6p) needs engineering buy-in on rollout, not just the class itself.** Icon color is a direct lift from real production values. Sizing (24×24, the WCAG floor exactly) and the removed border are direct design decisions, not evidence — production's real `.kebab-btn` instance has a border; it was removed on the basis that WCAG 2.5.8 governs tappable area, not visual styling. `:focus-visible` is new, added specifically for WCAG 2.2 SC 2.5.8/2.4.7 compliance, since the original inline-style pattern had no explicit dimensions or focus state at all. What needs review is the *adoption*: formalizing this into `mdb-customizations.css` implies replacing inline styles — and removing the border — at every existing icon-only-button call site, a real migration, not a drop-in addition. `aria-label` is required on every instance per WCAG 1.1.1/4.1.2 — flagged as a hard requirement, not a nice-to-have, in the kitchen sink documentation.

---

## 10. What changed from v2 (migration summary)

| v2 (retired) | v1 contract (ratified) |
|---|---|
| `ft-bento`, `ft-kv`, `ft-grid-*`, `ft-sidebar*` invented classes | Real classes: `card`/`card-body`, Bootstrap grid `row`/`col-*`, `side-panel`, `*-detail-row` |
| `--primary: #2D3B4A` hardcoded tokens | Foundant `--colors-*` tokens, consumed via `--mdb-*` remaps |
| `.btn-ft-filled` / `.btn-ft-sm` / `.btn-ft-default` | `.btn.btn-primary` / `.btn.btn-outline-dark.btn-sm.fw-bold` / `.btn.btn-dark…` |
| `ft-badge--*` (10 invented statuses) | `.badge.rounded-pill.badge-*` + status→semantic mapping (to be built) |
| Inline SVG symbol icons, FA banned | Font Awesome Pro, `fa-light` default |
| `ft-sidebar` global rail | `side-panel` 275px contextual rail with responsive collapse |
| `--font-heading` + `--font-body` split | single `--body-font` |
| Invented `h1..h4` scale | `h1: 24px` ratified; rest is a documented gap |

The litmus test for any future prototype: **could an engineer paste a card or a button straight into a Razor view and have it match production?** With v2 the answer was no. With the contract, for ratified components, the answer is yes.
