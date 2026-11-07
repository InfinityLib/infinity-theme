# @infinitylib/infinity-theme

An open, CSS-first design token and component theme.
No build step, no framework: the package publishes its CSS source as is.

It ships an editorial default taste (neutral light and dark modes, square
radii, status hues) that every consumer can rebind through the cascade.
Brand decisions, the accent, the typeface and accent-specific behaviours,
live in the consuming application.

## Install

```sh
bun add @infinitylib/infinity-theme
```

## Usage

Import the stylesheet, then scope the page with `.it-theme` and select the
mode and accent with data attributes:

```ts
import '@infinitylib/infinity-theme'
```

```html
<html class="it-theme" data-theme="light">
```

- `data-theme="light|dark"` selects the mode.
- The accent is not a library decision: `--it-accent` defaults to a neutral
  placeholder and each application binds its brand accent in its own
  stylesheet:

```css
/* application css */
:where(.it-theme) {
  --it-accent: #ffdf00;
  --it-accent-fg: #000000;
}
```

The library stays accent-agnostic both in values and in behaviour. If the
brand accent is too bright to work as text on light backgrounds, the
application carries that guard itself:

```css
/* bright accent: stay decorative on light backgrounds */
[data-theme='light'] .it-mark {
  color: inherit;
  text-decoration-line: underline;
  text-decoration-color: var(--it-accent);
  text-decoration-thickness: 2px;
  text-underline-offset: 2px;
}

[data-theme='light'] .it-nav-link[aria-current='page'] {
  color: var(--it-fg);
}
```

## Fonts

The library ships no typeface, no font file and no `@font-face` rule:
the visual identity of each application owns its font. `--it-font-sans`
defaults to a neutral system stack, and the consumer overrides it once the
font is loaded:

```css
@font-face {
  font-family: 'My Brand';
  src: url('./my-brand.woff2') format('woff2');
  font-display: swap;
}

:root {
  --it-font-sans: 'My Brand', ui-sans-serif, system-ui, sans-serif;
}
```

## Themes

A theme is a named bundle of token bindings scoped by the `data-theme`
attribute. The package ships two: `light` and `dark` (see
`src/css/tokens/themes/`). Declare additional themes in your own stylesheet
with the same mechanism; nothing else is needed, the components follow
through the token cascade:

```css
@layer it.tokens {
  :where([data-theme='midnight']) {
    color-scheme: dark;
    /* the token set below */
  }
}
```

A complete theme binds the full semantic set:

| Token | Role |
| --- | --- |
| `--it-bg`, `--it-fg` | Page background and text |
| `--it-surface`, `--it-surface-fg` | Raised surfaces: inputs, cards, pickers |
| `--it-muted` | Secondary text, placeholders, chevrons |
| `--it-border` | Borders and separators |
| `--it-disabled-bg`, `--it-disabled-fg`, `--it-disabled-border` | Disabled state ramp |
| `--it-accent`, `--it-accent-fg`, `--it-focus-ring` | Brand accent (yours, not the placeholder) |
| `--it-danger`, `--it-success`, `--it-warning`, `--it-info` and their `-fg` pairs | Status hues, only if your palette diverges |
| `color-scheme` | Native form and scroll styling follows the mode |

An incomplete theme degrades silently on the tokens it omits: inherited
values from the previous scope, or the `:root` default, remain in effect.

## Cascade contract

The library styles live in four cascade layers:

```
it.tokens, it.base, it.variants, it.overrides
```

When the consumer also uses Tailwind, declare the full order once, before
importing Tailwind. The first `@layer` statement wins, so this must come
before `@import 'tailwindcss'`:

```css
@layer theme, base, it.tokens, it.base, it.variants, components, utilities, it.overrides;

@import 'tailwindcss';
@import '@infinitylib/infinity-theme';
```

Optionally bridge Tailwind's semantic colours to the theme tokens:

```css
@theme {
  --color-background: var(--it-bg);
  --color-foreground: var(--it-fg);
  --color-primary: var(--it-accent);
  --color-border: var(--it-border);
  --radius-sm: var(--it-radius);
  --radius-md: var(--it-radius);
  --radius-lg: var(--it-radius);
}
```

## Tokens

Semantic tokens describe intent and are the ones components derive from:
`--it-bg`, `--it-fg`, `--it-surface`, `--it-surface-fg`, `--it-muted`,
`--it-border`, `--it-accent`, `--it-accent-fg`, `--it-danger`,
`--it-success`, `--it-warning`, `--it-info` and their `-fg` pairs,
`--it-focus-ring`, `--it-radius`, `--it-duration`, `--it-easing-standard`.

Primitives hold the raw scales: colours (`--it-color-*`), spacing
(`--it-space-0` to `--it-space-8`), font sizes, weights, line heights,
control heights and widths, z-index steps.

Structural tokens carry the values a page layout needs, so application
classes and any utility framework share the same scale:

| Token | Role | Default |
| --- | --- | --- |
| `--it-gutter` | Lateral gutter of a page container | `--it-space-4` |
| `--it-container-sm` | Narrow reading width | `40rem` |
| `--it-container-md` | Medium content width | `60rem` |
| `--it-container-lg` | Wide layout width | `80rem` |
| `--it-stack-gap` | Default gap inside a stack | `--it-space-5` |
| `--it-section-gap` | Vertical rhythm between sections | `--it-space-7` |

The library ships no layout components on purpose: the arrangement of a
page belongs to the application. Use the structural tokens from your own
classes:

```css
.app-main {
  inline-size: min(100% - 2 * var(--it-gutter), var(--it-container-lg));
  margin-inline: auto;
}

.hero {
  display: grid;
  gap: var(--it-stack-gap);
  padding-block: var(--it-section-gap);
}
```

## Components

Base HTML elements are styled inside `.it-theme` without classes, one file per
component under `src/css/components/`. Opt-in `it-*` classes carry variants,
sizes and states: buttons (`it-btn-primary`, `it-btn-sm`, `it-btn-ghost`,
...), form controls (`it-input-invalid`, `it-select-lg`, `it-field`,
`it-field-error`, ...), choice groups (`it-choice-group-2`, `it-choice`),
navigation (`it-nav-link`) and text helpers (`it-h5`, `it-p`, `it-small`,
`it-lead`, `it-muted`, `it-mark`).

High-variance families split into a **contract** and a **skin**:

- `src/css/components/button.css` is the contract: structure and states,
  where every design value is a `--it-button-*` knob;
- `src/css/style/button.css` is the skin: it binds each knob to a token or a
  derived value. A rebrand edits the skin only, never the contract
  (see `docs/skins/cold.css` for a rebind-only rebrand of a family).

Low-variance families (headings, paragraphs, links, tables, ...) consume the
semantic tokens directly and have no skin file.

A knob consumed by a contract must be bound by its skin: an unbound knob
silently falls back to nothing. Keep the two lists equal when either side
changes.

## Accessibility

- `it-sr-only` hides content from sighted users while keeping it available
  to screen readers.
- Focus rings and disabled states are built in.
- `prefers-reduced-motion` collapses transition durations.

## License

MIT. See [LICENSE](./LICENSE).
