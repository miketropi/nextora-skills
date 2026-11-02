---
name: nextora-design-system
description: Own colours, typography, spacing tokens, style variations and the optional visitor colour/font switcher for the Nextora parent theme and its child themes. Use when a request mentions a palette, colour role, preset slug, font family or font preset, theme.json settings, Tailwind @theme aliases, CSS custom properties, a style variation, "follow the palette", "must switch with the visitor's choice", scoped block colours, or appearance for a new child theme.
---

# Nextora design system

Single authority for appearance in this suite. Child-theme code, block code,
templates and patterns all defer here for palette, typography, tokens, style
variations and the switcher. Load this skill together with `nextora-child-themes`
for any new child, and with `nextora-gutenberg-blocks` for any restyled or new block.

## Load these references first

| File | Read it for |
| --- | --- |
| `references/presets-and-switching.md` | Canonical preset identity, role variables, scoped colour storage and PHP resolution, the two style mechanisms, the feature gate, what a child contributes, discovery/merge rules, `scheme-switcher.json` semantics, runtime precedence, the new-child `:root` role defaults, and the anti-claims list. |
| `../nextora-ecosystem/references/source-map.md` | Provenance and root-resolution vocabulary. |
| `../nextora-ecosystem/references/quality-gates.md` | Gates plus the shared downstream acceptance recipes (§8). |

Values quoted in the reference are anchors, not current data. Reread the target's
own `theme.json`, CSS entry, `styles/` and switcher files before editing.

## Ordered workflow

### 1. Resolve the target and identify the live appearance sources

Identify the parent theme root and the active theme root. Then read, in order:
`theme.json` (palette, gradients, font families, sizes, spacing, `styles`,
`customTemplates`), the CSS entry that ships (`resources/css/app.css` for the
parent, the child's own entry), the `styles/` tree, and `scheme-switcher.json`.
Never take a palette value from a skill, a screenshot or a sample child.

### 2. Decide which mechanism is correct

| Need | Use |
| --- | --- |
| Author picks a colour/typography/spacing for a block wrapper | Core block `supports` + `get_block_wrapper_attributes()` |
| A colour inside the block's own surface | A scoped attribute, stored as a preset slug, resolved to a preset variable on output (§5) |
| A site-wide alternative look the owner selects and keeps | A theme.json style variation (`styles/*.json`, `styles/typography/*.json`) |
| A visitor-facing preview choice | The switcher, and only behind its gate (§6) |
| A brand default for a whole child theme | A child palette/font contribution plus the `:root` role defaults (§7) |

Prefer native supports first. Add a scoped override only when global styles cannot
express it, and never invent a second appearance system beside these.

### 3. Change presets additively and by identifier

Add slugs; do not rename or re-point a parent slug inherited content already uses.
Every surface reads `var(--wp--preset--color--<slug>)`,
`var(--wp--preset--gradient--<slug>)` or `var(--wp--preset--font-family--<slug>)`,
so a literal hex in theme CSS opts that surface out of palette changes and out of
the switcher. Mirror Tailwind `@theme` aliases only where the CSS build consumes
them, and verify what each alias actually points at rather than copying an alias
block from an older guide — the parent's `--color-secondary` currently resolves to
the `paragraph` preset.

### 4. Route typography through the Nextora roles

Body, heading and button typography resolve through `--nextora-font-body`,
`--nextora-font-heading` and `--nextora-font-button`. `theme.json` uses
`var(--nextora-font-body)`; the switcher writes those three properties inline on
`<html>` with a concrete family stack. A surface that reads a preset variable
directly will not follow a visitor font change. Add or change a font family in
`theme.json` (with `fontFace` assets when it is not a system stack) and load it
before offering it as a preset choice.

### 5. Store scoped colours as presets, resolve them on output

Follow the existing helper pattern in `blocks/advanced-icon/color-utils.ts`
(`normalizeColorForStorage`, `colorValueForPicker`) and its PHP counterparts
(`nextora_icon_resolve_color`, `nextora_icon_hex_to_preset_slug`,
`nextora_ac_resolve_color`): a chosen preset stores its slug; a full-opacity hex
matching a palette entry normalises to that entry's slug; custom colours and
partial-alpha 8-digit hex stay literal; gradients keep preset identity the same way.
On output, a stored slug becomes `var(--wp--preset--color--<slug>)` so a palette
change reaches the surface, and 8-digit hex is handled **before**
`sanitize_hex_color()`. An empty value means "theme default" and emits nothing.
Reuse the real signatures; do not write a parallel resolver. Remember that a colour
control is not a sanitiser: stored HTML still passes `wp_kses_post()`.

### 6. Respect the switcher gate, and contribute data rather than code

The visitor switcher exists only when `NEXTORA_COLOR_SCHEME_SWITCHER` is strictly
`true`. Away from that, there is no payload, no script, no page-theme CSS and no
popover — and no child-side substitute should be written to fill the gap. A child
contributes **data**: `styles/color-scheme/*.json`, `styles/font-preset/*.json`,
optionally `styles/*.json` for a bundled variation, and an optional
`scheme-switcher.json` allowlist. When authoring a new child, set the child's own
default roles on `:root` in its later-loaded CSS (§7 of the reference) instead of
pinning a preset font. Never add a switcher implementation to a child.

### 7. Verify on the real surfaces

Build with the target's own script (`npm run build:css` or `npm run build`) — never
edit `assets/css/*` output, and never state that a build regenerates
`assets/js/scheme-switcher.js`, which is hand-maintained and outside the pipeline.
Then apply `../nextora-ecosystem/references/quality-gates.md` and its §8
design-system recipe plus the preset-specific steps in
`references/presets-and-switching.md` §10.

## Hard limits

- Do not hardcode a brand palette. Values come from the target's active presets.
- Do not propagate stale fallback hexes from older docs into new theme CSS.
- Do not re-implement the switcher, and do not claim it works because a sample child
  looks right: the sample pins its own fonts and adds child-only palette entries.
- Do not promise that a colour URL parameter overrides an already resolved bundled
  theme, or that Reset erases Site Editor styles.
- Do not restyle `wp-admin` chrome from a theme stylesheet; editor resets stay inside
  the editor canvas.
- Do not raise the declared WordPress/PHP minimum silently — surface the requirement
  instead.
- Do not modify the parent or sample child theme while delivering this work.

## Completion checks

- Target theme resolved; every preset, font, gradient and token referenced exists in
  that theme's own `theme.json`/variations.
- The mechanism table in step 2 was followed; native supports used where they
  suffice.
- Typography resolves through the `--nextora-font-*` roles and follows a preset
  change.
- Scoped colours store a preset slug where the colour matches a preset, keep custom
  and partial-alpha values literal, and render through preset variables.
- The switcher gate is respected; a child contribution is data only; any new-child
  default pair is declared on `:root` through the roles.
- Style variations (persistent) and the switcher (runtime preview) are described and
  tested as different things.
- The §8 acceptance steps were executed against **computed** styles, or the
  unverified surface is reported plainly.

## Boundaries

Markdown instructions and source-backed references only: no runtime scripts,
generators, copied theme trees, package management or installation automation, and
no edits to the parent or child theme sources. Downstream build and browser
verification commands belong to the target theme and are listed in
`../nextora-ecosystem/references/quality-gates.md`.
