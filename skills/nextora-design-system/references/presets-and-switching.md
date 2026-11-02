# Presets, role variables and switching

The appearance contract everything else defers to: canonical theme.json presets,
the Nextora role variables, scoped colour storage, style variations, and the
optional visitor colour/font switcher. Read the target's own sources first and
prefer their live values over any value quoted here.

| Read (relative to a theme root) | What it settles |
| --- | --- |
| `theme.json` | Palette, gradient, font-family, font-size and spacing slugs; `styles`; custom templates/parts. |
| `resources/css/app.css` (parent) or the child's CSS entry | `@theme` aliases, role variables, module import order, `@source` globs. |
| `styles/color-scheme/*.json`, `styles/font-preset/*.json`, `styles/*.json`, `styles/typography/*.json` | Preset and style-variation data. |
| `scheme-switcher.json` | Switcher curation: allowlists, labels, first-visit defaults, section visibility. |
| `inc/features/color-scheme-switcher/load.php`, `scheme-data.php`, `enqueue.php` | The feature gate, discovery, parsing, merge and precedence. |
| `inc/features/heading-inline-font/fonts.php` | `nextora_resolve_heading_font_family_stack()` and `nextora_get_heading_inline_font_families()` — the concrete font registry that the switcher's font presets and bundled themes resolve families through. |
| `assets/js/scheme-switcher.js` | The browser side: apply/reset, persistence, precedence, UI. |
| `blocks/advanced-icon/color-utils.ts`, `blocks/advanced-icon/lucide.php`, `blocks/advanced-container/render.php` | The scoped-colour helper pattern on both sides. |

Related: `../nextora-ecosystem/references/source-map.md`,
`../nextora-ecosystem/references/quality-gates.md`,
`../nextora-gutenberg-blocks/references/block-contract.md`.

## 1. Canonical preset identity

Parent palette slugs are `base`, `contrast`, `primary`, `paragraph`, `secondary`,
`surface`, `transparent`. Parent font-family slugs are `sans`, `mono`,
`hanken-grotesk`, `josefin-sans`, `noto-serif-display`, `jetbrains-mono`, and the
gradient slugs are `gradient-primary`, `gradient-secondary`, `gradient-tertiary`,
`gradient-subtle`, `gradient-soft`, `gradient-overlay`. All of these are
**identifiers**: a preset's value comes from the target's active
`theme.json`/variation, never from a value copied out of a skill or a sample design.

Rules:

- Reference presets by their emitted variables
  (`var(--wp--preset--color--<slug>)`, `var(--wp--preset--gradient--<slug>)`,
  `var(--wp--preset--font-family--<slug>)`) so a palette or variation change flows
  through. A literal hex in theme CSS opts that surface out of every palette and
  switcher change.
- A child may add slugs (`nextsaas` adds `border`, `accent-soft`, `mist`, `cobalt`
  and the `bodoni-moda` / `dm-sans` families) but must not rename or re-point a
  parent slug that inherited content already uses.
- Keep the identifiers the switcher needs: fonts are addressed by registry slug,
  and preset files reference them by slug.

## 2. Role variables — the switching seam

The parent `resources/css/app.css` `@theme` block defines three role variables and
they are the only font handles the switcher writes:

```css
--nextora-font-body: var(--wp--preset--font-family--sans, Arial, Helvetica, sans-serif);
--nextora-font-heading: var(--nextora-font-body);
--nextora-font-button: var(--nextora-font-body);
```

Everything else must resolve through those roles:

- `theme.json` `styles.typography.fontFamily` is `var(--nextora-font-body)`.
- `applyFont()` sets `--nextora-font-body`, `--nextora-font-heading` and
  `--nextora-font-button` as inline styles on `document.documentElement`, using the
  **concrete family stack** from the font registry (`FONTS[preset.body].family`),
  with `heading` falling back to `body` when the preset omits or breaks it.
- `applyTheme()` (a bundled colour+font preset) sets the same three roles plus the
  palette and gradients, and enables the `#nextora-page-theme-vars` style element.
- The reset helpers remove exactly those inline properties.

`resources/css/app.css` also mirrors Tailwind `@theme` aliases onto presets
(`--color-base`, `--color-contrast`, …) so utilities stay in step. Mirror an alias
only where the CSS build actually consumes it, and check what it really points at:
the parent's `--color-secondary` currently maps to
`var(--wp--preset--color--paragraph, …)`, so a `text-secondary` utility is **not**
the `secondary` preset. Copying alias blocks from an old doc propagates that kind of
drift. **[observed defect]**

## 3. Scoped colour overrides: store the preset, resolve on output

When a block needs a colour inside its own surface (not the block wrapper), follow
the parent helper pattern. On the editor side, `blocks/advanced-icon/color-utils.ts`
(`normalizeColorForStorage`, `colorValueForPicker`, `normalizeGradientForStorage`,
`gradientValueForPicker`):

| Stored input | Stored output | Later palette/variation change |
| --- | --- | --- |
| `secondary` (a known slug) | `secondary` | Follows. |
| `var:preset\|color\|secondary` or `var(--wp--preset--color--secondary)` | `secondary` | Follows. |
| `#039590` equal to a palette entry's colour | that entry's slug | Follows. |
| `#039590ff` (opaque 8-digit) equal to a palette entry | that entry's slug | Follows. |
| `#03959080` (partial alpha) | `#03959080` literal | Stays literal — intended. |
| Any other hex / `rgb()` / `hsl()` | literal | Stays literal. |

Hex comparison normalises `#abc` → `#aabbcc` and strips alpha for matching
(`normalizeHex`, `stripHexAlpha`, `paletteColorMatches`). Gradients behave the same
way: a stored value whose normalised CSS equals a registered gradient stores the
slug; a custom `linear-gradient(...)`/`radial-gradient(...)`/`conic-gradient(...)` is
kept literal.

On the PHP side the equivalent resolvers are `nextora_icon_resolve_color()` and
`nextora_icon_hex_to_preset_slug()` (`blocks/advanced-icon/lucide.php`) and
`nextora_ac_resolve_color()` (`blocks/advanced-container/render.php`). They:

- accept `var:preset|color|<slug>`, a raw slug, a literal
  `var(--wp--preset--color--<slug>)`, `transparent`, `rgb()`/`hsl()` and 3/6/8-digit
  hex — and, in the icon resolver **only**, `has-<slug>-color`; the container
  resolver has no `has-*-color` branch (it also accepts `rgba(0, 0, 0, 0)`);
- return `var(--wp--preset--color--<slug>)` for anything that maps to a preset
  **so a palette change reaches the surface**;
- **handle 8-digit hex before `sanitize_hex_color()`**, which does not accept
  alpha: `preg_match('/^#([0-9a-f]{8})$/i', …)` first, map it through
  `nextora_icon_hex_to_preset_slug()`, and only if that fails keep the literal
  lowercased value. `nextora_icon_hex_to_preset_slug()` normalises with
  `sanitize_hex_color()` and, on a slug tie, picks the most frequent slug.
  Because that helper only accepts the 3- or 6-digit forms, the remap cannot
  recover an **8-digit** value: a stored `#039590ff` that never passed through the
  editor renders as the literal and stops tracking the palette. Only the editor's
  normalised output can be relied on.
- Note the editor guard is case-sensitive: `normalizeColorForStorage()` tests
  `trimmed.endsWith( 'ff' )` on the untrimmed-case value, so `#039590ff` normalises
  to the preset slug while the same colour written `#039590FF` stays literal.
  **[observed defect, minor]**
- return `'currentColor'` (icon) only for empty or whitespace input; a non-empty
  but unrecognised icon value falls through to the raw value unchanged. The
  container resolver returns `''` for unrecognised input. The resolver's own
  allowlist — not the picker — is therefore the boundary.

Reuse these signatures; do not write a parallel resolver. The sample child does
**not** demonstrate this pattern: `resources/ts/editor/colour-controls.tsx` exists
there but nothing imports it, and no `block.json` declares the
`sectionBg`/`sectionHeading`/`sectionText` attributes it drives — so no sample
block currently stores a scoped colour as a preset slug. Copying the sample
therefore proves nothing about switching; follow the parent helper pattern above.
**[observed defect]**

Note what none of this covers: **a colour control never sanitises stored HTML.** The
toolbar/format restriction is a UX boundary; output still passes `wp_kses_post()` or
a narrower allowlist.

## 4. Two different mechanisms — do not conflate them

| | Site Editor style variations | Visitor switcher (optional) |
| --- | --- | --- |
| What it is | Persistent site-wide styles saved to the database / `styles/*.json` | Runtime overrides in the browser for a preview/demo experience |
| Persistence | Site styles | `localStorage` + cookie |
| Gate | None | `NEXTORA_COLOR_SCHEME_SWITCHER === true` |
| Mechanism | WordPress global styles | Inline custom properties on `<html>`, plus one served `<style>` for a resolved page theme |
| Reach | Everything that reads the preset | Everything that reads the preset variables or the `--nextora-font-*` roles |

The switcher can only move a surface that already resolves through presets or roles
— that is exactly why the child contract in §7 matters.

## 5. Feature gate

`inc/features/color-scheme-switcher/load.php` returns immediately unless the
constant is **strictly** `true`:

```php
if ( ! defined( 'NEXTORA_COLOR_SCHEME_SWITCHER' ) || true !== NEXTORA_COLOR_SCHEME_SWITCHER ) {
	return;
}
```

Absent, `false` or `'1'` (string) means: no payload, no `assets/js/scheme-switcher.js`,
no `#nextora-page-theme-vars` inline CSS, no popover skeleton, no page-theme meta
box. When the gate is off, **do not build a child-side switcher to compensate** —
report that visitor switching is disabled and make the child's defaults correct
instead.

## 6. What a child contributes: data, not code

A child adds appearance by shipping files, not by re-implementing the feature:

| File | Shape | Notes |
| --- | --- | --- |
| `styles/color-scheme/<name>.json` | theme.json-shaped `title` + `settings.color.palette` (+ optional `settings.color.gradients`) | Palette entries with empty `slug` or `color` are skipped; slugs are kebab-cased. |
| `styles/font-preset/<name>.json` | `title`, `body`, optional `heading` | `body` must be a registered font slug or the file is skipped; `heading` falls back to `body`. |
| `styles/<name>.json` | A bundled variation carrying **both** a palette and resolvable font roles | Registered as a bundled *theme* as well as a colour preset. |
| `styles/typography/<name>.json` | Per-family style variations | Parent-side only today. |
| `scheme-switcher.json` | Curates the lists and supplies first-visit defaults | Merged parent → child. |

Discovery is `nextora_glob_preset_files( $rel_glob )`: glob the parent theme root,
then the child theme root, `array_unique` the results. Colour and bundled-theme
presets iterate `styles/color-scheme/*.json` **then** `styles/*.json` (non-recursive,
so colour-scheme files are not read twice) and key each entry by
`sanitize_title( $preset['title'] )` — so **later sources win on a slug collision**,
in this order:

1. parent `styles/color-scheme/*.json`
2. child `styles/color-scheme/*.json`
3. parent `styles/*.json`
4. child `styles/*.json`

A collapsed duplicate title therefore silently replaces the earlier preset rather
than erroring. Give presets distinct titles. If a `title` is missing the runtime
falls back to the file basename (still queryable), but authored files should declare
one. A malformed or unreadable file is skipped (`is_readable`, `json_decode`,
empty palette, unusable font all return `null`), not fatal.

Fonts: `nextora_get_font_registry()` only exposes fonts that actually exist on the
site (it reuses the merged heading-font list, so the returned stacks are concrete).
`nextora_get_font_presets()` requires `title` + a valid `body` slug, falls back
`heading` → `body`, and **if no valid font-preset file survives it synthesizes one
preset per registered font** (title = the font's name). A font that is only
declared inside a style variation is still not a visitor choice: declare and load it
in the merged `settings.typography.fontFamilies` (with `fontFace` assets where
needed) before offering it.

Bundled themes additionally require resolvable font **roles**:
`nextora_parse_theme_preset()` resolves `styles.typography.fontFamily`,
`styles.elements.heading.typography.fontFamily` and
`styles.elements.button.typography.fontFamily` through
`nextora_resolve_heading_font_family_stack( $var, $font_map )`; a value that resolves
back to a `var(...)` is rejected, and the file's own
`settings.typography.fontFamilies[0]/[1]` are only a fallback. No resolvable body
font means the file becomes a colour-only preset, not a bundled theme. Button colours
default to `var(--wp--preset--color--primary)` / `base`, with `:hover` on
`secondary` / `base`, and header-CTA overrides come from
`styles.blocks["nextora/header"].elements.button`.

### `scheme-switcher.json` semantics

```json
{
  "version": 1,
  "initial": { "theme": "", "color": "", "font": "" },
  "sections": {
    "themes": { "enabled": true, "showDefault": true },
    "colors": { "enabled": true, "showDefault": true },
    "fonts":  { "enabled": true, "showDefault": true }
  },
  "presets": { "themes": [], "colors": [], "fonts": [] }
}
```

- `presets.themes` / `.colors` / `.fonts` are **allowlists that replace** the
  parent list wholesale (not merged by index). Each rule is `{ slug, enabled?, label? }`;
  an unknown slug is skipped, `enabled: false` excludes it, `label` renames it.
  `null`/absent keeps auto-discovery. Order in the file becomes the displayed order.
- `initial.{theme,color,font}` are validated against the **filtered** sets; an
  invalid slug becomes `''` (no forced default).
- `sections.<axis>.enabled: false` removes that section from the popover;
  `showDefault: false` removes the synthetic "Default" option (which maps to the
  reset action). A themes section with zero bundled themes is removed as well.
- The parent merge reads parent then child, so a child's file wins per key;
  `presets.*` arrays are replaced rather than concatenated.

## 7. New-child role defaults on `:root` (child CSS contract)

A child that pins its fonts straight at a preset variable is switching-proof in
appearance only until the visitor changes the font. Instead, the child's
later-loaded canvas/frontend CSS binds the Nextora roles to **its own** default pair:

```css
:root {
  --nextora-font-body: var(--wp--preset--font-family--dm-sans);
  --nextora-font-heading: var(--wp--preset--font-family--bodoni-moda);
  --nextora-font-button: var(--nextora-font-body);
}
```

Rules for this contract:

- Set the roles, not the presets: `applyFont()`/`applyTheme()` write these three
  properties inline on `<html>`, so an inline visitor choice overrides a normal
  `:root` declaration, and the child keeps its authored pair when no choice exists.
- The slugs used here must be real font-family slugs **in the child's merged
  theme.json**, with `fontFace` assets when they are not system stacks — otherwise
  the default resolves to nothing.
- A child's own aliases (`--nextsaas-font-body`, `--nextsaas-font-display`, …) must
  resolve **through the roles**, e.g. `var(--nextora-font-body)`. An alias that points
  at `var(--wp--preset--font-family--bodoni-moda)` looks right but ignores every font
  preset — which is exactly the sample's situation, so a child cannot prove
  switching works by copying it. **[observed defect]**
- Colour defaults follow the same principle: child palette aliases read
  `var(--wp--preset--color--<slug>)` with a render-safe fallback, as
  `00-tokens.css` line 2 does.
- Where a bundled theme narrows the visible font set, the switcher injects
  `#nextora-scheme-font-fallbacks`, which re-points `.has-<slug>-font-family` and
  inline `--*-font-family: var(--wp--preset--font-family--<slug>)` occurrences that
  are no longer available back to the body/heading roles. Keep font slugs registered
  so this fallback does not have to fire for ordinary content.
- The served page theme wins on specificity: `nextora_get_theme_inline_css()` emits
  `:root:root { … }` (plus button override selectors) into
  `#nextora-page-theme-vars`. The child's plain `:root` block is intentionally weaker.

## 8. Runtime precedence (from source, not from the summary doc)

Bundled theme selection, in `bootstrap()` order:

1. `?theme=<slug>` in the URL (and it is persisted).
2. Direct page theme (`OPTIONS.isDirectPageTheme` from PHP resolution).
3. A saved `prefs.theme` in `localStorage`.
4. `pageTheme` resolved server-side for this request (cookie / page meta fallback).
5. `config.initial.theme`.

**If a bundled theme resolves, its colour *and* font pairing applies** — the
independent colour/font axes are skipped for that request. Only when no theme
resolves do colour and font take their own precedence, each axis independently:
URL (`?color=`, `?font=`) → saved choice → `config.initial` → theme defaults.

Persistence: `save()` writes `localStorage['nextora-scheme-preferences']` as
`{ v: 3, theme, color, font }` and the cookie `nextora_active_theme`
(`max-age=2592000; SameSite=Lax`), writing the literal `default` when no theme is
active. A one-time migration reads and removes the legacy
`nextora-color-scheme` key. Persisting happens when the request came from a URL
parameter or when a theme was resolved directly.

Server side, `nextora_resolve_current_request_theme()` resolves the theme for the
request: a `nextora_active_theme` cookie of `default` short-circuits to "no theme";
then URL `?theme=`; then front page (`_nextora_theme` on `page_on_front` when
`show_on_front === 'page'`, else the cookie, else the configured initial); then
singular (`_nextora_theme` on the queried post, then the post slug, then the
unhyphenated slug, then ancestors); then the cookie for inner pages; then the
initial fallback. `_nextora_theme` is registered post meta on `page` with
`sanitize_title` and an `edit_posts` auth callback, and its meta box write path
checks a nonce plus `current_user_can( 'edit_page', $post_id )`.

**Reset** calls `resetTheme()`: it clears colour, gradient, font and button
overrides and disables `#nextora-page-theme-vars`, then saves the reset state. It does
**not** erase Site Editor styles, and it does not "unresolve" a bundled theme that
the page itself designates — a subsequent request resolves that theme again. Do not
promise otherwise. Likewise, a `?color=` parameter does not override an already
resolved bundled theme (§8, item 4 of the JS chain).

## 9. Never claim these

- That `npm run build` regenerates `assets/js/scheme-switcher.js`. It is a
  hand-maintained parent asset outside the build pipeline, versioned with
  `filemtime()` at enqueue time.
- That a colour URL parameter always wins. It loses to a resolved bundled theme.
- That a duplicate preset title is an error, or that the first file wins. Later
  sources win on `sanitize_title( title )`.
- That a style variation is enough to offer a visitor font choice.
- That the switcher creates a persistent site style. It is a runtime preview.

## 10. Acceptance, on top of quality-gate §8

Add these to the shared design-system recipe:

1. Assert **computed** values, not the presence of a custom property: read the real
   element's `color`/`background-color`/`font-family` before and after a palette or
   preset change, and after a child adds its own palette entries.
2. Confirm a scoped override stored as a preset slug follows a palette change, a
   custom hex does not, and a partial-alpha 8-digit value survives to the rendered
   output unchanged.
3. Confirm the child's `:root` role defaults are what renders with no choice stored,
   then confirm a visitor choice overrides them and that Reset restores them.
4. Confirm a style variation change in the Site Editor is reflected on the front end
   independently of the switcher (the persistent path is separate).
5. With the gate disabled, confirm the script, the `NEXTORA_THEME_OPTIONS` global,
   the page-theme style element and the popover are all absent.
6. Confirm inherited block content whose font class is no longer in the active
   theme's `fontSlugs` still renders through the body/heading roles.
