# ADR-0001 : Split a component into a contract and its values

- **Status** : `accepted`
- **Date** : 2026-11-07

## Validation

| Field        | Value |
|--------------|-------|
| Validated    | yes |
| Validated by | FortyTwo_Dev |
| Validated on | 2026-11-07 |

## Context

The package was extracted from infinity-qrcode with one file per family
grouped by kind (all button rules together, all form variants together). The
owner had started a parallel hand-built prototype of the same theme in the
website repository, with a different inner architecture: a component declares
the variables it consumes (`--button-height` and so on) and a separate values
file binds them.

Two needs drive the design: a rebrand must be able to restyle every component
without touching structure, and the file layout must let a reader find any
single component quickly.

Contracts and values earn their keep only where design values are numerous
enough: a heading with one rule and three direct token references gains
nothing from an indirection layer.

## Decision

- One file per component under `src/css/components/`, whatever the family
  size: `button.css`, `input.css`, `select.css`, `h1.css` to `h6.css`,
  `p.css`, `a.css`, and so on.
- High-variance families split into a **contract** and a **skin**:
  `components/N.css` holds the structure, the states and the variant classes,
  and consumes one knob namespace `--it-<family>-*`; `style/N.css` binds every
  knob to a token or a derived value. Both live in cascade layers: skins bind
  in `it.tokens`, contracts style in `it.base` and `it.variants`.
- Variant classes turn knobs; they never hold values. An `aria-invalid` state
  turns the same knob as the invalid class.
- Low-variance families consume the semantic tokens directly and ship no skin
  file.
- The scope of the initial conversion: `button` and the input family
  (`input`, `textarea`, `select`). A family joins the split when its variance
  and its derived values justify the layer, evaluated by the criteria below.

## Consequences

- **Positive**: a rebrand edits `style/` only; a structural change edits
  `components/` only; the knob list reads as the documented surface of a
  component; the completeness of bindings is checkable mechanically.
- **Negative**: every design decision is one indirection longer in the
  tools; the total CSS grows with the binding files (measured for the pilots:
  roughly plus 60 percent on the converted families); a forgotten binding
  degrades silently.
- **Trade-offs**: families below the variance threshold keep direct token
  consumption, accepting two coexisting patterns in exchange for not paying
  the indirection where it buys nothing.

## Rejected alternatives

- **The full split everywhere**: rejected; headings and paragraphs are single
  rules on direct tokens, the extra layer would only add aliases
  (the input pilot measured 28 bindings for 28 pure token aliases and zero
  derived value in the size section).
- **Pure direct tokens (no split at all)**: rejected; the button family
  measured 8 values out of 48 that exist nowhere in the token set (hover
  opacity, disabled states, variant bindings), and the rebrand scenario
  requires editing structure to change a colour.
- **One file per family (the extracted layout)**: rejected; a reader cannot
  find one component without opening a grouped file, and component growth
  would concentrate conflicts in the same files.
