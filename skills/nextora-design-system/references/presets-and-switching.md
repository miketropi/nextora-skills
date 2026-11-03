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

Related: `../../nextora-ecosystem/references/source-map.md`,
`../../nextora-ecosystem/references/quality-gates.md`,
`../../nextora-gutenberg-blocks/references/block-contract.md`.

## 1. Canonical preset identity

Parent palette slugs are `base`, `contrast`, `primary`, `paragraph`, `secondary`,
`surface`, `transparent`. Parent font-family slugs are `sans`, `mono`,
`hanken-grotesk`, `josefin-sans`, `noto-serif-display`, `jetbrains-mono`, and the
gradient slugs are `gradient-primary`, `gradient-secondary`, `gradient-tertiary`,
`gradient-subtle`, `gradient-soft`, `gradient-overlay`. All of these are
**identifiers**: a preset's value comes from the target's active
`theme.json`/variation, never from a value copied out of a skill or a sample design. The parent's
spacing-size slugs are `10`, `20`, `30`, `40`, `50`, `60` (`settings.spacing.spacingSizes`, with
`defaultSpacingSizes: false`).

**Reading the merged presets in PHP is not a flat list.** `wp_get_global_settings()` returns a preset
key (palette, fontFamilies, spacingSizes, gradients …) **bucketed by origin** —
`default`, `theme`, `custom`, `user` — not as one array of entries:

```
wp_get_global_settings( array( 'spacing', 'spacingSizes' ) )
→ array( 'default' => array( … core's 20–80 … ),
         'theme'   => array( array( 'slug' => '10', 'size' => 'clamp(…)', … ), … ),
         'custom'  => …, 'user' => … )
```

So `array_column( $value, 'slug' )` over that returns nothing, and an assertion built on it fails
while the presets are perfectly present. Flatten the buckets first, and assert the `theme` bucket
when the point is "the parent's own presets survived a child's `theme.json`". Do **not** treat a
populated `default` bucket as a defect on its own — it carries core's defaults and is expected even
when a theme sets `defaultSpacingSizes: false`. **[verified against WordPress 7.1.4, 2026-11-03]**

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

Reuse this **behaviour** — slug storage, hex→slug on a full-opacity match, literal retention for
partial alpha, 8-digit hex handled before `sanitize_hex_color()`, `''` for empty — but implement it as
**one helper shared by every block in the target theme**. In a child theme that helper belongs to the
child; a new parent block should share one too. Copying the shape block-by-block is what produced the
drift listed above, and calling a parent *block's* internal resolver from child code is not reuse at
all: it is a dependency on a private implementation that is free to change without notice. The sample child does
**not** demonstrate this pattern: `resources/ts/editor/colour-controls.tsx` exists
there but nothing imports it, and no `block.json` declares the
`sectionBg`/`sectionHeading`/`sectionText` attributes it drives — so no sample
block currently stores a scoped colour as a preset slug. Copying the sample
therefore proves nothing about switching; follow the parent helper pattern above.
**[observed defect]**

Note what none of this covers: **a colour control never sanitises stored HTML.** The
toolbar/format restriction is a UX boundary; output still passes `wp_kses_post()` or
a narrower allowlist.

## 3.1 Contrast pairs: foreground, background and the surface they meet

§3 is about storing and resolving *one* colour. A surface that can receive a background needs a
**pair**, and the pair is the acceptance unit.

### 3.1.1 The thresholds

External standard, not a house preference — WCAG 2.2 AA, and it is the number an acceptance
report has to quote:

| Foreground | Minimum | Applies to |
| --- | --- | --- |
| Normal-size text | **4.5:1** | headings, body copy, lists, captions, metadata |
| Large text | **3:1** | ≥ 24 px, or ≥ 18.66 px when bold |
| Non-text UI, boundaries, state | **3:1** | focus rings (`:focus-visible`), button borders, input outlines, icons that carry meaning |

Colour must not be the only state cue, and a failing pair is a defect even when the design looks
fine in one scheme.

**A focus indicator is its own foreground, measured against the surface it is drawn on.** Do not
inherit it from the body text colour by default: the moment an author sets that colour close to the
background, the ring falls below 3:1 (a light `surface` value on a white section is roughly 1.1:1)
and the failure is invisible in the panel that reports the *text* ratios. Give the ring a
surface-derived token, and report its measured ratio in the editor alongside the heading, content
and button pairs. The same reasoning applies to control borders and any other boundary the author
can recolour. The parent ships no contrast helper at all — `grep -ri
'luminance\|contrast_ratio\|wcag'` over its `blocks/`, `inc/` and `resources/` returns nothing —
so this is a **[new-output requirement]**, not something to copy. **[verified, 2026-11-03]**

### 3.1.2 Three attributes, not one

Heading text, non-heading content text and the background are three separate attributes, each
defaulting to `''`. Links, buttons and focus rings are further foregrounds on the same surface
and are checked independently. Naming follows the closest existing block's vocabulary and must
avoid reserved core support names (§3): the live parent splits the roles across blocks as
`headingColor` / `descriptionColor` / `quoteColor` / `authorNameColor` / `linkColor` /
`overlayBackgroundColor` / `buttonTextColor` (`blocks/expanding-cards/block.json`,
`blocks/stacking-cards/block.json`, `blocks/testimonials/block.json`). There is no parent-wide
`contentColor`; pick one name and use it consistently inside the block.
**[parent contract, by observation]**

### 3.1.3 Empty means "derive the theme default", and one derivation serves both paths

- An empty foreground emits **no** inline declaration.
- With the foreground empty, the readable value is derived from the *background actually in
  effect*: resolve the background to a comparable value, compare the candidate foregrounds the
  theme already supplies (`base`, `contrast`, `paragraph`, …) by WCAG contrast ratio, and keep
  the best one. Picking between the theme's own anchors — rather than testing a hardcoded
  lightness threshold or inventing a colour — is what makes a palette change and a style
  variation re-derive correctly.
- **Derive from the surface the section actually paints.** A section with no background colour and
  no media layer paints nothing: it sits on the page canvas, so the derivation must not composite
  a scrim over a stand-in grey and conclude that white text is right. Return "nothing to derive"
  and emit nothing, which is when the theme default applies.
- **Every surface derives its own pair — nesting included.** A card, callout or inset panel inside a
  section is its own surface: if the section is dark and the card keeps a light background, the card's
  copy must be derived against the *card's* surface, not inherit the section's. A foreground inherited
  across a surface boundary is the `1.10:1` white-on-light failure, and it is invisible whenever the
  two surfaces happen to agree. Derive per surface, or make the outer rule also set the inner surface's
  background so the pair is deliberate.
- **Composite the background's own alpha before measuring.** A semi-transparent background colour is
  not the surface: composite it over the surface beneath (the page canvas, or the parent surface) and
  derive against the result, or the decision is made against a colour the browser never paints — a
  `rgba(18,52,86,0.5)` band derives a foreground for `#123456` while rendering over
  `rgb(136,153,170)`, and content that passes on paper measures 2.68:1 on screen.
- The **editor and the renderer must call the same derivation.** Two implementations (a PHP one
  for `render.php`, a JS one for an inspector notice) are the parity defect in gate §2 waiting to
  happen: they drift, and the editor starts telling the author something the front end does not do.
- A derived value is **never persisted back** into an attribute. It is computed state, not author
  intent, and writing it would make an empty control stop meaning "follow the theme".
- An explicit author choice is **never silently overwritten**. When the background changes, stored
  foregrounds stay byte-identical; if the resulting explicit pair fails the threshold, surface it
  (a notice naming the measured ratio) and offer an explicit action the author can undo. Changing
  a background is not consent to change a foreground.

Derivation requires parsing the colour the theme actually emits. Accept at least 3/6/8-digit hex,
`rgb()`/`rgba()`, `hsl()`, and whatever form the target's own palette uses; composite alpha over
the surface behind it before measuring; and return "underivable" — never a guess — for anything
else, then fall back to the page-canvas pair and say so in the editor.

### 3.1.4 A background image stops being a colour

An overlay is **required**, not an advanced option, whenever a background image can carry text,
and contrast is measured against the **composited** result:

1. The pair is measured against the surface the text actually sits on: background colour,
   composited with the overlay at its authored opacity, over the image.
2. The image's own pixels are unknown server-side, so the overlay is the readability mechanism.
   That is a statement about *where* the guarantee comes from, not a substitute for checking: the
   worst-case region of the real image must be measured (sample the rendered pixels behind the
   text box, not the image average), and the overlay opacity raised or the pair changed until it
   passes.
3. A background image with an empty overlay colour and an opacity of `0` is the case to look for
   and reject: the live parent ships exactly that in `patterns/banner-with-text-button.php` —
   `backgroundType: image`, `overlayOpacity: 0` — over a remote photograph with headings on top.
   **[observed defect]**
4. Never promise a universal automatic foreground for an arbitrary image. "Unverifiable" is a
   reportable state; "the overlay makes it readable" without a measurement is not.
5. The parent's overlay convention is **not uniform, and neither is the attribute set**: `page-title`
   uses `overlayOpacity` as a `0.3` float, `call-to-action` as a `40` integer percentage and
   `slide-item` as `0.4` — three scales among the blocks that have the attribute at all. Only
   `page-title` and `advanced-container` carry the full image-plus-`overlayColor`-plus-`overlayOpacity`
   set; `call-to-action` has no `overlayColor`, and `scrolling-image-strip` has no image field. Pick
   one convention for the block, document it, and never assume the set is complete elsewhere.
   **[observed inconsistency, verified 2026-11-03]**

### 3.1.5 Store foregrounds exactly as §3 stores any scoped colour

Preset-slug storage, hex→slug normalisation on a full-opacity match, literal retention for partial
alpha, 8-digit hex handled before `sanitize_hex_color()`,
`var(--wp--preset--color--<slug>)` on output, `''` for "theme default" — all of it applies to
foregrounds unchanged. A foreground stored as a literal hex is a foreground that will not follow a
palette change, which is exactly the failure the pair is supposed to prevent.

### 3.1.6 These are the checks, on computed styles

Per surface, in the browser, on the real page:

1. Read `getComputedStyle(el).color` for the heading and for body copy, and the **effective**
   background — the element's own `background-color`, or the nearest opaque ancestor when it is
   transparent, composited with any overlay layer.
2. Compute the ratio and assert §3.1.1's thresholds hold.
3. Repeat for: theme defaults (empty controls); a light custom background; a dark preset
   background; an image with an overlay; a custom literal; a partial-alpha literal; and an empty
   override.
4. Change a **Site Editor style variation** and, separately, a **visitor preset**: preset-backed
   colours must follow, custom literals must stay fixed, and every resulting pair must still pass.
   Roles are positional — their values invert per scheme — so a section must select an
   **offsetting pair drawn from the same roles**, never "a light value for dark sections". Check
   the target's own scheme files rather than assuming: the parent ships five
   (`styles/color-scheme/*.json`), and in the dark ones (`midnight`, `developer`) `base` *and*
   `surface` are the dark values while `contrast` is the light one — the reverse of the light
   ones.
5. Confirm descendant selectors split the roles: headings consume the heading colour and
   paragraphs, lists, captions and metadata consume the content colour.
6. Check links, buttons and focus rings independently, and confirm colour is not the only cue.

A colour that is present as a custom property but was never measured is not a passed check.

## 3.2 Spacing: one owner per measurement, stored by identity

Three different measurements get called "spacing", and the live parent ships all three on a single
block, so the ownership question is real rather than theoretical. Of the parent's **40** block
manifests, **39** declare `supports.spacing`, **20** declare a `blockGap` key inside it (**19** of
them enable it; one sets it `false`), and **16**
declare their own gap attribute under **14** distinct names (`cardGap`, `columnGap`,
`elementGap`, `gap`, `gridColumnGap`, `gridGap`, `gridRowGap`, `iconTextGap`, `imageGap`,
`itemGap`, `numberLabelGap`, `rowGap`, `stepHorizontalGap`, `stepVerticalGap`).
`blocks/expanding-cards/block.json` alone carries wrapper `padding`, wrapper `margin`, wrapper
`blockGap` **and** its own `cardGap`. `grep -rn blockGap` over this suite returned nothing before
this section existed. **[verified, 2026-11-03]**

### 3.2.1 Who owns what

| Space | Owner | Emitted as | Never |
| --- | --- | --- | --- |
| Space *around* the block, authored per instance | Core supports: `supports.spacing.margin` / `.padding` | `style.spacing` on the wrapper, via `get_block_wrapper_attributes()` | A hand-built inline `margin`/`padding` on the wrapper |
| A section's **vertical rhythm** | The section block's own padding, on its own element | preset identity: `var:preset\|spacing\|<slug>`, or a sanitised custom length when the inspector offers one | The wrapper, when core already emits padding there |
| Space between elements **inside** the block | The block's own gap attribute (`blockGap`, `columnGap`, `rowGap`, `itemGap`) | a block-scoped custom property read by the block's stylesheet | A copied literal length that cannot follow the spacing scale |

### 3.2.2 Rules

- **One element, one owner.** If a wrapper carries core `style.spacing.padding`, the section's
  rhythm padding belongs on a different element. Two controls that write the same property on the
  same element is a defect, not a bonus.
- **Store by identity, not by value.** A spacing control that offers the theme's spacing scale
  stores `var:preset|spacing|<slug>` (emitted as `--wp--preset--spacing--<slug>`) so a change to
  the scale reaches the surface — the same argument as §3 for colours. The live parent does exactly
  this throughout its own patterns (`style.spacing.padding` of `var:preset|spacing|50`, `margin`
  of `0`), so this is the existing convention rather than a new one. **[parent contract, by observation]**
- **Sanitise custom lengths, and only where offered.** Where the inspector allows a custom value,
  accept a CSS length (a number with a unit, or a `clamp()`/`min()`/`max()`/`calc()` form built
  from lengths) and reject everything else — including anything containing `;`, `{`, `}`, quotes,
  `url(`, `@` or a comment. Never pass an arbitrary string into a style attribute. Note the
  parent's two spacing resolvers disagree on their fallback signature
  (`nextora_sis_resolve_spacing( $raw, '' )` vs
  `nextora_sw_resolve_spacing( $raw, 'var(--wp--preset--spacing--40, 1rem)' )`) — pick one
  behaviour for the block and document it rather than inheriting both.
- **Empty emits nothing.** No `padding-top:` with an empty value, no `--x:;`. An empty control
  means "the theme's own value", and the stylesheet — not an empty inline declaration — is where
  that value lives.
- **`settings.spacing` is a sequential array like the palette.** Declaring
  `settings.spacing.spacingSizes` in a child **replaces** the parent's array. Either list every
  inherited entry (the parent's are `10`, `20`, `30`, `40`, `50`, `60`, with
  `defaultSpacingSizes: false`) or omit the key entirely and inherit it. Omitting is the cheaper
  correct answer. **[parent contract]**
- **No rhythm by margin collapse.** Adjacent sections must not rely on collapsing margins to
  produce or remove a gap: a collapsed margin is invisible in the inspector, disappears in a
  flex/grid parent, and cannot be measured per section. Neutralise the margins
  (`style.spacing.margin` `top`/`bottom` of `0`) and own the rhythm with padding, then verify the
  computed padding at both sides of every boundary.

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

- **Every JSON file under the child's `styles/` tree is also a Site Editor style
  variation.** `WP_Theme_JSON_Resolver::get_style_variations()` scans `styles/**`
  recursively for both themes (child wins on basename) and registers any file
  without `blockTypes` as a variation
  (`wp-includes/class-wp-theme-json-resolver.php`). So `styles/color-scheme/*.json`,
  `styles/font-preset/*.json` and `styles/*.json` all appear in the Site Editor's
  Styles list as well as in the visitor popover — and a font-preset file, which
  carries no styles, appears there as a no-op variation. This layout makes that
  unavoidable: give every file a distinct, meaningful `title`, keep the switcher's
  `presets.*` allowlists tight so the popover stays curated, and tell the owner
  which entries belong to which mechanism. Do not invent a second directory layout
  just to hide a variation.

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
