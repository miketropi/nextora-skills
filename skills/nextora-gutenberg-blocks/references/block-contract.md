# Block contract

How a Nextora or Nextora-child block is registered, edited, rendered and built.
Every rule below was read from the live parent theme root and the child theme root;
provenance tags mark which is which.

- **[parent contract]** enforced by Nextora code at runtime — matching it is required.
- **[sample convention]** the child happens to be built this way — copy the pattern.
- **[new-output requirement]** a rule this suite imposes on new work.
- **[observed defect]** a real inconsistency; do not reproduce it.

Reread the local sources before editing: `blocks/blocks.php`, the closest existing
block folder, `scripts/build-blocks.mjs`, `package.json`, and the target
`theme.json`. Runtime metadata outranks documentation.

Related: `../../nextora-ecosystem/references/source-map.md` (anchors),
`../../nextora-ecosystem/references/quality-gates.md` (gates and acceptance recipes),
`editor-and-repeaters.md` (list editing), `../../nextora-design-system/SKILL.md` (appearance).

## 1. Resolve the target before writing anything

| Question | How to answer it |
| --- | --- |
| Parent or child block? | Parent registrar scans `get_template_directory() . '/blocks'`; the child registrar scans `get_stylesheet_directory() . '/blocks'`. A child block is invisible until the child registers it. |
| Existing block to extend? | `blocks/*/block.json` — names, categories, `supports`. Prefer changing the closest existing block over adding a near-duplicate. |
| Which namespace? | Parent `nextora/<slug>`, text domain `nextora`. Child `<child-slug>/<slug>`, text domain `<child-slug>`. |
| Is appearance in scope? | Then load `../../nextora-design-system/SKILL.md`; block appearance is owned there. |

## 2. The block folder

```text
blocks/<slug>/
  block.json          metadata: registration, attributes, asset pointers   [source]
  index.tsx           registerBlockType( metadata, { edit, save } )        [source]
  edit.tsx            editor UI: canvas + InspectorControls + Modal(s)     [source]
  render.php          front-end markup for a dynamic block                 [source]
  style.css           authored front-end CSS when the block bundles CSS    [source]
  editor.css          authored inspector/canvas-preview CSS                [source]
  save.tsx            only for blocks that persist InnerBlocks or static markup [source]
  view.ts             front-end behaviour source                           [source]
  types.ts            shared attribute types                                 [source]
  register-styles.php optional PHP integration (block styles, localize)    [source]
  register-editor.php optional editor-only PHP integration                 [source]
  index.js            build output                                         [generated]
  index.asset.php     build output (dependencies + version)                [generated]
  view.js / view.css  build output                                         [generated]
```

Never hand-edit a `[generated]` file; the next `npm run build:blocks` overwrites it
and the dependency manifest is regenerated with a `Date.now()` version
(`writeAssetFile()` in `scripts/build-blocks.mjs`), so a rebuild is never a clean diff.

## 3. `block.json` [parent contract]

Real keys from `blocks/team-section/block.json` and the child manifests:

```json
{
  "$schema": "https://schemas.wp.org/trunk/block.json",
  "apiVersion": 3,
  "name": "nextora/team-section",
  "title": "Our Team",
  "category": "nextora",
  "description": "…",
  "keywords": ["team", "nextora"],
  "textdomain": "nextora",
  "icon": "groups",
  "supports": {
    "html": false,
    "align": ["wide", "full"],
    "anchor": true,
    "color": { "background": false, "text": false, "link": false },
    "spacing": { "padding": true, "margin": true }
  },
  "attributes": { "members": { "type": "array", "default": [ /* 4 seeded member records */ ] } },
  "editorScript": "file:./index.js",
  "editorStyle": "file:./editor.css",
  "style": "file:./view.css",
  "viewScript": "file:./view.js",
  "render": "file:./render.php"
}
```

- `apiVersion: 3`, `editorScript: "file:./index.js"`, `render: "file:./render.php"` — every
  dynamic block in both themes.
- **Asset pointers name built files, never sources.** `viewScript` points at
  `view.js`, built from `view.ts`. Pointing metadata at a `.ts`/`.tsx` file ships a
  file the browser cannot run.
- **Front-end CSS can be bundled by the view script.** `team-section` authors
  `style.css` and imports it from `view.ts` (`import './style.css'`), next to
  `import 'swiper/css'` and its module styles; esbuild emits one `view.css`, which
  `block.json` points `style` at. Editing `style.css` (or the Swiper imports)
  without rebuilding leaves the front end on the old CSS.
- **Category.** Set `nextora` (or the child slug) in metadata, but know it is
  cosmetic: PHP and the editor both force it (section 4). The old advice to use
  `theme` for chrome blocks is contradicted by the code. **[observed defect]**
- **Attributes are typed and defaulted in metadata.** Array attributes with real
  defaults (members) keep the editor renderable before any author input.
- **`textdomain` in `block.json` is not readable back off the registered block type.** Core uses it
  for i18n-schema translation of the metadata fields and for the asset's script translations
  (`wp-includes/blocks.php:598,604-606`, `:321-323`) and never assigns it into the settings array, so
  `WP_Block_Type::$textdomain` stays `null` even though the manifest declares the domain — the
  property exists (`wp-includes/class-wp-block-type.php:110`) and is simply never populated from
  metadata. Declare the domain in `block.json` for the editor and the translation pipeline, and when
  you need to *assert* it, read the manifest (or the registered asset), not the block type.
  **[verified against WordPress 7.1.4, 2026-11-03]**
- **`supports.color` off + `PanelColorSettings` on** is the existing pattern where
  wrapper colours would fight the section design: the block exposes its own scoped
  colour attributes instead of core ones.
- **A section's colour is a set, not one control.** Heading text, non-heading content text and
  the background are separate attributes defaulting to `''`; links, buttons and focus rings are
  further foregrounds on the same surface. The live parent splits the roles this way inside
  individual blocks — `headingColor` / `descriptionColor` / `quoteColor` / `authorNameColor` /
  `linkColor` / `overlayBackgroundColor` / `buttonTextColor` in `expanding-cards`,
  `stacking-cards` and `testimonials` — but never documents the pattern as a contract, and adds
  no contrast requirement to it. Follow the closest block's naming, keep one content-role name
  inside the block, and treat the pair as the thing that must pass
  `../../nextora-design-system/references/presets-and-switching.md` §3.1.
- **A background-image attribute set is not complete without an overlay.** The parent exposes an
  image + overlay set in `page-title`, `advanced-container` and partly `slide-item`, but the set is
  **not uniform across blocks**: `call-to-action` has no `overlayColor`, `scrolling-image-strip` has
  no image field at all, and `overlayOpacity` is expressed on three different scales (`page-title`
  `0.3` float, `slide-item` `0.4`, `call-to-action` `40` integer percent). Pick one convention for
  the block and document
  it. Pass a focal point when the image is a background, and render the image on a decorative
  layer (`aria-hidden`) so the copy is never inside an `<img>`.
- **`supports.spacing` and an inner gap attribute are different owners.** `supports.spacing`
  writes `style.spacing` on the *wrapper* through `get_block_wrapper_attributes()`;
  `blockGap` (inside `supports.spacing`) is the space between children; a custom
  `columnGap`/`rowGap`/`itemGap`/`cardGap` is the block's own internal layout and belongs in
  its stylesheet. The parent uses all three inconsistently — of its 40 manifests, 39 declare
  `supports.spacing`, 20 also declare a `blockGap` key (19 enable it, one sets it `false`), and 16 declare their own gap attribute under
  14 distinct names; `expanding-cards` carries wrapper padding, wrapper margin, wrapper
  `blockGap` and its own `cardGap` on one block. Decide and write down which owner each space
  has, and do not add a `paddingTop`-style attribute that writes the property core already
  writes on the same element.

## 4. Registration [parent contract]

Parent `blocks/blocks.php`:

- `nextora_register_blocks()` on `init` globs `get_template_directory() . '/blocks'`
  with `GLOB_ONLYDIR` and calls `register_block_type( $block_dir )` for **every**
  directory. It does **not** check that `block.json` is readable, so an unfinished
  folder registers and fails at runtime rather than being skipped.
- `nextora_register_block_categories()` (`block_categories_all`) adds slug `nextora`,
  title "Nextora Blocks", icon `layout`.
- `nextora_filter_block_type_args()` (`register_block_type_args`, two args) sets
  `$args['category'] = 'nextora'` for any name starting `nextora/`.
- `nextora_enqueue_block_category_filter()` (`enqueue_block_editor_assets`,
  priority 5) adds an inline `wp-blocks` script applying the same coercion through
  `blocks.registerBlockType`, because the client registry validates categories too.
- Optional per-block PHP is pulled in conditionally at the bottom of the file
  (`register-styles.php` for several blocks, `register-editor.php` for others).

Child `blocks/blocks.php` is the stricter variant: `nextsaas_register_blocks()`
sorts the glob, **skips directories without a readable `block.json`**, registers the
block, then `require_once`s that block's `register-editor.php` when readable. It
also mirrors the category coercion server- and client-side and enqueues one shared
inspector stylesheet. **[sample convention]**

A child must load its own registrar: WordPress does not scan a child's `blocks/`
directory, and the parent registrar deliberately reads only the template directory.

## 5. Scaffolding [parent contract]

```bash
# parent theme root
npm run gen -- --name <slug> --ns nextora
```

Space-separated arguments only: `scripts/gen-block.mjs` reads flags with
`args.indexOf('--name')` plus the next item, so `--name=<slug>` is not the verified
form. The script writes `block.json`, `index.tsx`, `edit.tsx` and `render.php`, plus
`style.css` when `--ns nextora`. Its category default is `text`, remapped to `theme`
for the `nextora` namespace — **change it to `nextora` in the generated
`block.json`**. The generator exists only in the parent (`gen-block.mjs`); the child
`package.json` has no `gen` script, so a child block is authored by copying the
closest folder and renaming every identifier. **[sample convention]**

## 6. Build pipeline [parent contract]

`scripts/build-blocks.mjs`:

- Editor entry points: `./blocks/*/index.{ts,tsx}`. View entry points: `./blocks/*/view.ts`.
- `bundle: true`, `format: 'iife'`, `jsx: 'automatic'`, `jsxImportSource: 'react'`.
- `wpExternalsPlugin` intercepts **every** `@wordpress/*` import via `onResolve`
  and replaces the module with `module.exports = window.wp['<camelCaseName>']` —
  not raw esbuild `external: []`, which breaks CJS interop in IIFE output.
- The generated `index.asset.php` lists handles from a **fixed 14-package list**:
  `blocks`, `block-editor`, `server-side-render`, `components`, `element`, `i18n`,
  `hooks`, `data`, `core-data`, `compose`, `primitives`, `blob`, `notices`, `plugins`.
- The **view** build has no shim and bundles its own dependencies (that is where
  Swiper, GSAP, OGL and friends live in the shipped bundle).
- **Wire the block build into the theme's own aggregate build.** A theme whose
  `npm run build` builds only CSS and the front-end script leaves every
  `block.json` pointing at an `index.js` that a clean checkout never produced:
  the block then exists in the working tree and vanishes for the next person.
  Add `build:blocks` to the `build` chain (and to the build documentation), and
  check the built files are the ones `block.json` names.

Consequences that bite:

- **An unlisted `@wordpress/*` import builds successfully and then reads
  `window.wp['x']` that may never be loaded.** The shim resolves *any* package; the
  manifest only guarantees the 14 handles above. `@wordpress/icons` is the common
  trap, and neither theme imports it anywhere today. Before adding such an import,
  either avoid it (inline the SVG, use a core control) or enqueue the matching
  `wp-*` handle explicitly in the editor; the asset manifest will not do it for you.
- **There is a second, separate editor build.** `scripts/build-editor.mjs` compiles
  `resources/ts/editor/index.ts` → `assets/js/editor.js` + `editor.asset.php` with
  its **own** package list, which additionally includes `@wordpress/rich-text`,
  `@wordpress/edit-post` and `@wordpress/api-fetch`. Those imports are fine in that
  bundle (the parent's `resources/ts/editor/heading-inline-font.tsx` uses
  `@wordpress/rich-text`) and are not made safe by the block manifest. Do not lift
  the editor bundle's package list into `build-blocks.mjs` or assume a block entry
  inherits it.
- **Editor scripts are not a place for a second React — and the shim alone does not prevent one.**
  `wpExternalsPlugin` intercepts `@wordpress/*` only. With `jsx: 'automatic'` and
  `jsxImportSource: 'react'`, esbuild emits `import { jsx } from "react/jsx-runtime"`, and `react`
  is present in `node_modules` as a transitive dependency — so **both source themes bundle a full
  React runtime into every block bundle**: 40 of the parent's 40 `blocks/*/index.js` and 39 of the
  sample's 39 contain React's element symbol, with no `react` entry in the asset manifest, because
  the manifest only ever describes `@wordpress/*`. Do not reproduce that. Add an `onResolve` filter
  for `/^react(\/jsx-runtime|\/jsx-dev-runtime)?$/` that maps the module onto
  `window.wp.element` (which *is* React) and ships zero React — verified by grepping the built
  bundle for `Symbol.for("react.element")`, which must be absent. Do not add a second React, a
  blanket `@wordpress/scripts` pipeline, or a different bundler.
- **Generated bundles and manifests are not source.** No shipped skill, patch or
  review may treat `index.js`, `index.asset.php`, `view.js` or `view.css` as editable.

Libraries already available in the parent `package.json` (ranges, not installed
versions — read the lockfile before relying on an API): GSAP `^3.15.0`, Swiper
`^11.2.10`, `@tiptap/*` `^3.22.2`, Lucide `^1.7.0`, Chart.js `^4.5.1`, Three
`^0.185.1`, OGL `^1.0.11`. Do not bundle a library the block does not use.

Node version: read the target `.nvmrc` (the parent declares `24`) and the target's
own scripts; do not assume the two themes expose the same command set.

## 7. `save` semantics [parent contract]

| Block kind | `save` | Evidence |
| --- | --- | --- |
| Attributes rendered in PHP | `save: () => null` + `render: "file:./render.php"` | `blocks/team-section/index.tsx`, every child block |
| Persists nested blocks | `save: Save` returning `useBlockProps.save()` wrapping `<InnerBlocks.Content />` | `blocks/slide-wrapper/save.tsx`, `blocks/slide-item/save.tsx` |
| Persists static markup | `save` returns exactly the markup the parser expects | **No parent example.** All parent block directories ship `render.php`, and the nine that also ship `save.tsx` (`advanced-button`, `advanced-button-button`, `advanced-container`, `call-to-action`, `hero-paper-roll`, `hero-section`, `page-title`, `slide-item`, `slide-wrapper`) still render in PHP — a `save.tsx` beside `render.php` is legacy or validating markup, not a static block |

**`save: () => null` on a block that persists InnerBlocks silently discards the
author's nested content.** Decide which of the three the block is before writing
`index.tsx`. Changing an attribute's type or meaning later needs a `deprecated`
entry with `save`/`migrate`, or a one-time migration for the affected content —
never a silent shape change.

## 8. Editor contract

- **Prefer native fields.** `RichText` for visible text, `InnerBlocks` for nested
  block content, `MediaUpload`/`MediaPlaceholder` for attachments, core link UI for
  URLs. Plain `TextControl` is for identifiers, technical URLs and configuration —
  not the only editor for headings, body copy or repeated rich content.
- **Controls and panels.** `InspectorControls` + `PanelBody`; `SelectControl`,
  `RangeControl`, `ToggleControl`, `RadioControl`, `TextareaControl`,
  `PanelColorSettings`. Reuse the established panel vocabulary — `Members`,
  `Layout`, `Autoplay`, `Pagination`, `Navigation`, `Colors`, `Animation`,
  `Popup Drawer` in `team-section` — instead of inventing new titles for the same
  ideas. Keep `help` text where the existing block has it.
- **Expose empty as "theme default".** Scoped colour attributes default to `''` and
  an empty value emits no inline style, so the block follows the active preset.
- **Scoped inner-element overrides need non-reserved names.** A reserved core
  attribute name would be claimed by block supports; `surfaceBackgroundColor`-style
  names are not reserved. Follow the existing naming in the closest block
  (`sectionBackgroundColor`, `cardBackgroundColor`, `nameColor`, `roleColor`, …).
- **Editor CSS stays in the inspector and the canvas preview.** Never restyle
  `wp-admin` chrome, `.components-*` internals or global form controls from a theme
  stylesheet. `.nextsaas-block-preview__surface` deliberately disables pointer
  events inside the preview while the wrapper stays selectable — keep that split.
- **Visual editing beats a parallel overlay framework.** A new simple block edits
  its real canvas fields directly. Reuse the child's existing preview stack
  (`resources/ts/editor/block-preview.tsx`, `rich-text-overlay.tsx`,
  `rich-text-control.tsx`, renderer `data-nextsaas-rich="<dot.path>"` markers) only
  where the block already previews PHP through `ServerSideRender`; 1:1 fields go in
  the overlay, repeated fields go through the list editor
  (`editor-and-repeaters.md`).
- **No persisted UI state.** Open/closed editor state, draft values and status text
  live in React state; only author-meaningful values reach `setAttributes`.
- **An affordance must be *visible*, not merely present.** A control that renders as
  a 24×24 button with no glyph is a missing control to the person using it. Two ways
  this happens, both seen in a real delivery: an icon-only `Button` using a dashicon
  string renders blank because the **canvas iframe has no `dashicons` stylesheet**
  unless the theme enqueues it (`@wordpress/icons` SVGs avoid the problem but are not
  in the fixed handle list, so check before importing), and a shared label variable
  gives move-up, move-down and remove the *same* `aria-label`, so nothing — not the
  tooltip, not a screen reader — says which action is which. Therefore: give every
  control its own action-specific label (`Remove fact %d`, `Move fact %d up`), pass
  `showTooltip`, enqueue core `dashicons` for the editor when you use glyph strings,
  and make destructive actions survive a font that fails to load (a visible word
  beside the glyph). Then verify by measuring the rendered control — its box, its
  computed glyph and its label — in the real canvas, not by finding it in the DOM.

## 9. `render.php` contract

- Wrap the root with `get_block_wrapper_attributes( $wrapper_args )` — the parent
  does this in `blocks/team-section/render.php` and `blocks/advanced-icon/render.php`
  — and merge your own `data-*`/style into `$wrapper_args`, not into a hand-built
  `<div class>` and not into a second `style` attribute beside it.
- **Emit scoped colours as custom properties on the block's own styled element, and let the stylesheet
  consume them** — the inner surface when the wrapper already carries core spacing, so the wrapper's
  style attribute belongs to core supports alone (§3.2's one-element-one-owner rule). An empty
  attribute then emits no declaration at all (theme default), a preset slug resolves to
  `var(--wp--preset--color--<slug>)` so a palette change reaches the surface, and the stylesheet owns
  the fallback. Separate the owners while you do it: the section's own rhythm padding goes on that
  same block element, not on the wrapper core already pads.
- **One resolver per theme, not per block.** The parent has 25 `nextora_*_resolve_color()`
  implementations with real drift between them (`advanced-list` does no hex→slug mapping;
  `advanced-container` returns `''` for unknown input; `advanced-icon` returns `'currentColor'`
  for empty input and is the only one that accepts `has-*-color`). Duplicating that shape is
  what produced the drift. A child block resolves through the child's own shared helper; a new
  parent block should share one. The storage rules themselves are
  `../../nextora-design-system/references/presets-and-switching.md` §3, and the pair rules are
  §3.1.
- **Derive a foreground from the surface the block actually paints.** A section with no background
  colour and no media layer paints nothing: compositing a scrim over a stand-in grey and
  concluding "white text" produces white-on-white the moment the theme's own background is light.
  Return "nothing to derive", emit nothing, and let the theme default apply.
- **Escape by context**: `esc_html` for text, `esc_attr` for attribute values and
  inline style strings, `esc_url` for URLs, `wp_kses_post` (or the theme's rich
  helper) for stored HTML. The child's `nextsaas_rich()` runs `esc_html` on text
  nodes and then `wp_kses_post()` on the reassembled fragment; the parent escapes
  each field at the point of output.
- A **restricted editor toolbar is not a security boundary.** Sanitize stored HTML
  on output regardless of what the editor offered.
- Build repeated markup from a normalised array (`nextsaas_normalise_rows()`-style
  helpers, or the parent's `normalizeMembers()` on the JS side) so holes and
  scalars cannot leak into a `foreach`.
- Resolve scoped colour values through the theme helper rather than echoing raw
  input: `nextora_icon_resolve_color()` / `nextora_ac_resolve_color()` for parent
  blocks, the child's own contract for child blocks. See
  `../../nextora-design-system/references/presets-and-switching.md`.
- Every string goes through the **target theme's** text domain — `nextora` in the
  parent, `<child-slug>` in a child. Never the parent domain in child code.
- Optional per-block PHP (`register-styles.php` / `register-editor.php`) is
  `require_once`d by the registrar when readable. `register_block_style()` calls
  belong there; `team-section` registers `soft-band` and `contrast-band` style
  variations on `init` priority 20 and localises its editor placeholder URL through
  `wp_localize_script` on `enqueue_block_editor_assets`.

## 10. View-script contract

- One entry file per block (`view.ts`), built to `view.js`, declared as
  `viewScript: "file:./view.js"`.
- **Init must be idempotent.** The parent guards with per-element state attributes
  (`data-nextora-testimonial-swiper-inited`, `…-pending`) and returns early when
  either is set, so a re-run after an editor/preview re-render cannot duplicate
  listeners or markup.
- **Reserve layout and signal loading.** Emit a `--loading` class from the server
  render, then swap to `--ready` when init finishes; back it with an explicit
  `aspect-ratio`/`min-height` so nothing jumps.
- **Honour reduced motion** (`window.matchMedia('(prefers-reduced-motion: reduce)')`,
  used by `blocks/advanced-button/view.ts`, `blocks/advanced-icon/view.ts` and
  others) and keep the content readable and operable with JavaScript disabled.
- The parent sometimes enqueues a view script from PHP when the registry holds its
  handles (`nextora_icon_enqueue_view_script()` in `blocks/advanced-icon/lucide.php`).
  If you add that pattern, keep it conditional and idempotent — do not enqueue a
  carousel script on every page that never uses the block.

## 11. Acceptance

Block-specific steps to add on top of
`../../nextora-ecosystem/references/quality-gates.md` §8:

1. Build with the target's own script (`npm run build:blocks`, or `npm run build`)
   and confirm the block appears in the **right category** in the inserter — proof
   the PHP and editor category filters agree.
2. Insert, edit and re-select the block: the canvas fields and the inspector must
   both still work after a re-render (rules out duplicated init and stale preview).
3. Save → reload → re-save with no block-validation warning; confirm the front-end
   order and text match the editor.
4. Confirm any scoped colour left empty follows the theme default, and that a
   selected preset still tracks a palette change.
5. **The colour set, on computed styles, in more than one scheme.** For each of theme defaults, a
   light custom background, a dark preset background, an image plus overlay, a custom literal, a
   partial-alpha literal and an empty override: read the computed `color` for a heading and for a
   paragraph and the *effective* background (own `background-color`, else the nearest opaque
   ancestor, composited with the overlay), then assert 4.5:1 normal text / 3:1 large text and
   focus or UI boundaries. Change a Site Editor style variation and, separately, a visitor preset:
   preset-backed colours must follow, literal ones must not move, and every resulting pair must
   still pass. Confirm the heading selector consumes the heading colour and that paragraphs,
   lists, captions and metadata consume the content colour.
6. **Changing the background does not rewrite an explicit foreground.** Set a heading colour, then
   change the background, and confirm the stored heading value is unchanged — and that the block
   says so (a measured notice), rather than silently correcting it.
7. **Spacing ownership.** With the wrapper spacing controls and the block's own padding/gap
   controls, confirm each measurement lands on exactly one element; clear each control and confirm
   no empty inline declaration remains; confirm a preset-backed value follows a change to the
   spacing scale. Put the block in two adjacent sections and confirm the gap between them comes
   from padding on both sides, not from a collapsed margin.
8. With the block's `viewScript` present, test the no-JS and reduced-motion paths
   and confirm no layout jump at narrow and wide widths.
9. Confirm the block renders complete, readable content with JavaScript disabled — a dark or
   image-backed section included.
