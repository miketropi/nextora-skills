---
name: nextora-design-system
description: Own colours, typography, spacing tokens, style variations and the optional visitor colour/font switcher for the Nextora parent theme and its child themes. Use when a request mentions a palette, colour role, preset slug, font family or font preset, theme.json settings, Tailwind @theme aliases, CSS custom properties, a style variation, "follow the palette", "must switch with the visitor's choice", scoped block colours, or appearance for a new child theme.
---

# Nextora design system

Single authority for appearance in this suite. Child-theme code, block code,
templates and patterns all defer here for palette, typography, tokens, style
variations and the switcher. Load this skill together with `nextora-child-themes`
for any new child, and with `nextora-gutenberg-blocks` for any restyled or new block.
A port from an external source template starts at
`../nextora-template-porting/SKILL.md`, which supplies the token evidence and
source-to-role mapping but hands every preset, contrast-pair, spacing-ownership and
storage decision to this skill — and never overrides it.

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
| A section that must stay readable on any background it can be given, including a photograph | A coordinated foreground/background **role pair** — separate heading, content and background attributes, one derivation shared by the editor and the renderer, and a contrast check on the *composited* surface (§3.1) |
| Deciding who owns a space: the wrapper's margin/padding, a section's vertical rhythm, or the gap between elements inside it | Spacing ownership — exactly one owner per measurement, named in the block and in the pattern (§3.2) |

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
Reuse this **behaviour**, but implement it as one helper shared by every block in the target theme —
a child's own helper, never a call into a parent block's internals. Remember that a colour
control is not a sanitiser: stored HTML still passes `wp_kses_post()`.

### 6. Pair every foreground with the background it actually sits on

A colour control that sets a background is half a decision. Whenever a surface can receive a
background — a preset, a literal, or a photograph under an overlay — its foregrounds are a
**set**, and the set is what has to be readable:

- **Separate roles, separate controls.** Heading text, non-heading content text and the
  background are three attributes, each defaulting to `''`. Links, buttons and focus rings are
  further foregrounds on the same surface and are checked too (`:focus-visible` is a non-text
  boundary; §3.1). Never fold heading and body copy into one "text colour".
- **Empty means "theme default", derived.** With a control left empty, the readable value is
  derived from the background actually in effect, by one helper shared by the editor and the
  renderer so the two cannot disagree. A derived value is never persisted into an attribute.
- **An explicit choice is never silently overwritten.** Changing the background must not rewrite a
  heading or content value the author set; when an explicit pair fails the threshold, surface it
  and offer an explicit action instead.
- **A photograph is not a colour.** Derivability stops at the overlay: measure the *composited*
  worst-case surface, which means an overlay control is not optional, and never claim the overlay
  guarantees readability without that measurement (§3.1).
- **Verify on computed styles, in more than one scheme.** The pair must survive a palette change,
  a Site Editor style variation and a visitor preset. Roles are positional and their *values*
  invert per scheme, so the rule is "an offsetting pair drawn from the same roles", never "a light
  value for dark sections".

### 7. Give every space exactly one owner

Three different things are routinely all called "spacing", and the live parent ships all three on
one block (a single manifest can declare `supports.spacing`, `blockGap` and its own `cardGap`).
Name the owner before writing either side:

| Space | Owner |
| --- | --- |
| Space around the block, chosen by the author | Core block supports — `supports.spacing.margin` / `.padding`, emitted through `get_block_wrapper_attributes()` |
| A section's vertical rhythm | The section's own padding, on the block's own element |
| Space between elements *inside* the block | A custom gap attribute of the block (`blockGap`, `columnGap`, `rowGap`, `itemGap`, …) |

The emission form, the custom-length sanitiser, the `settings.spacing` replacement hazard and the
no-margin-collapse rule are owned by
`references/presets-and-switching.md` §3.2 — do not restate them here.

Rules that follow: do not let one element carry two owners; emit nothing for an empty value;
prefer the preset's identity over a copied length so the rhythm follows a change to the spacing
scale; and never rely on margins collapsing between siblings to produce a gap.

### 8. Respect the switcher gate, and contribute data rather than code

The visitor switcher exists only when `NEXTORA_COLOR_SCHEME_SWITCHER` is strictly
`true`. Away from that, there is no payload, no script, no page-theme CSS and no
popover — and no child-side substitute should be written to fill the gap. A child
contributes **data**: `styles/color-scheme/*.json`, `styles/font-preset/*.json`,
optionally `styles/*.json` for a bundled variation, and an optional
`scheme-switcher.json` allowlist. When authoring a new child, set the child's own
default roles on `:root` in its later-loaded CSS (§7 of the reference) instead of
pinning a preset font. Never add a switcher implementation to a child.

### 9. Verify on the real surfaces

Build with the target's own script (`npm run build:css` or `npm run build`) — never
edit `assets/css/*` output, and never state that a build regenerates
`assets/js/scheme-switcher.js`, which is hand-maintained and outside the pipeline.
Then apply `../nextora-ecosystem/references/quality-gates.md` and its §8
design-system recipe plus the preset-specific steps in
`references/presets-and-switching.md` §10.

## Hard limits

- Do not hardcode a brand palette. Values come from the target's active presets.
- Do not promise a universal automatic foreground for an arbitrary image. The luminance of
  unknown pixels cannot be derived from a colour string: require an overlay or a verified
  author-selected pair, and report the unverifiable case as unverified instead of asserting it.
- Do not resolve a child block's colours by calling a parent block's internal helper
  (`nextora_ac_resolve_color()`, `nextora_icon_resolve_color()`, `nextora_si_resolve_color()`,
  …). Those are per-block, they already disagree with one another, and they are not an API. A
  child resolves through its own shared helper, and a new parent block should share one too.
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
- Every background-capable surface has separate heading, content and background controls, and
  its derived pair comes from one helper shared by the editor and the renderer.
- Contrast was measured on composited computed styles at the thresholds in
  `references/presets-and-switching.md` §3.1, in the default scheme plus at least one
  dark/alternative scheme, including an image-overlay case.
- Every space in the deliverable has exactly one named owner (§3.2 of the reference); no gap
  depends on margins collapsing, and an empty spacing control emits nothing.
- The §8 acceptance steps were executed against **computed** styles, or the
  unverified surface is reported plainly.

## Boundaries

Markdown instructions and source-backed references only: no runtime scripts,
generators, copied theme trees, package management or installation automation, and
no edits to the parent or child theme sources. Downstream build and browser
verification commands belong to the target theme and are listed in
`../nextora-ecosystem/references/quality-gates.md`.
