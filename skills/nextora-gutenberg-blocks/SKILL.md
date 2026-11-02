---
name: nextora-gutenberg-blocks
description: Build or change Gutenberg blocks in the Nextora parent theme or a Nextora child theme — block.json registration, typed attributes, native visual editing with RichText and media, InspectorControls settings panels, repeatable list editors with modal item editing, render.php output, view scripts, block style variations, and the esbuild block pipeline. Use when a request mentions a block, block.json, edit.tsx, index.tsx, render.php, ServerSideRender, InnerBlocks, a repeater or list editor, a modal item form, an inspector panel, a block category, or "add/restyle a section block" for Nextora.
---

# Nextora blocks

Owns the code contract for a block: metadata, registration, editor UI, dynamic
render, view script, build output consumption. Appearance rules are owned by
`nextora-design-system`; templates and patterns by `nextora-site-composition`;
child-theme scaffolding by `nextora-child-themes`. Load the design-system skill
whenever a request touches how a block looks, and load both whenever a block must
follow presets or the visitor switcher.

## Load these references first

Read only what the task needs, by relative path from this file:

| File | Read it for |
| --- | --- |
| `references/block-contract.md` | Folder layout, `block.json` keys, registrar behaviour, category coercion, the generator, the esbuild/`window.wp` shim and its 14-handle dependency list, `save` semantics, render and view contracts. |
| `references/editor-and-repeaters.md` | The two real list editors (`team-section` and the child `repeater.tsx`), their exact commit/cancel semantics, rich text and media helpers. |
| `../nextora-ecosystem/references/source-map.md` | Where every rule comes from and how to re-check it; root resolution vocabulary. |
| `../nextora-ecosystem/references/quality-gates.md` | Data, parity, asset, security, i18n, accessibility gates and the shared downstream acceptance recipes (§8). |

Never treat a value quoted in a reference as current: reread the target theme's own
files before editing.

## Ordered workflow

### 1. Resolve the target and the block's host theme

Identify the parent theme root and the active (child) theme root from the live
install or the supplied repository — `get_template_directory()` and
`get_stylesheet_directory()`, or the `style.css` headers. A child block only exists
if the child registers it; the parent registrar reads the template directory only.
Never claim Nextora compatibility without the real Nextora source.

### 2. Read the target's own evidence

In this order, preferring live values over anything written here:

1. `blocks/blocks.php` — the registrar, its glob, whether it checks readability,
   category filters, and which per-block PHP files it pulls in.
2. `blocks/*/block.json` — existing names, categories, `supports`, asset pointers.
3. The closest existing block's `edit.tsx`, `render.php`, optional `view.ts`,
   `save.tsx`, `register-styles.php`/`register-editor.php`.
4. `scripts/build-blocks.mjs` — the exact entry globs and the shim.
5. `package.json` + `.nvmrc` — the commands and Node version that actually exist.
6. `theme.json` — palette, font, gradient, font-size and spacing slugs the block
   should reference.

### 3. Decide reuse before creation

Extend an existing block when the requested editing structure already exists
(another heading field, another list, another toggle). Create a block only when the
requested **behaviour or editing structure** cannot be expressed by an existing
block, core block, pattern or block style variation. State which existing block you
reused, or why nothing fit — and state it **in the deliverable**, not only in the
conversation: a docblock in the block's editor entry plus a line in the theme's
`README` (naming the closest candidates you rejected and what each one lacks). A
recorder who reads only the repository has to be able to find that decision.

### 4. Write the metadata first

`block.json` with `apiVersion: 3`, namespace/textdomain/category (`nextora` +
`nextora` in the parent; `<child-slug>` in a child), typed attributes with real
defaults, `supports` for wrapper appearance, and asset pointers that name **built**
files: `editorScript: "file:./index.js"`, `render: "file:./render.php"`,
`viewScript: "file:./view.js"`. Category in metadata is cosmetic — PHP and the
editor both force the namespace category — so never rely on `theme` or another slug
to place the block.

### 5. Decide `save`, then build the editor

- Dynamic (PHP renders from attributes): `render` + `save: () => null`.
- Persists nested blocks: `save` must return `useBlockProps.save()` wrapping
  `<InnerBlocks.Content />`.
- Static: save exactly what the parser expects.

Then write `edit.tsx` with native fields — `RichText` for visible text,
`InnerBlocks` for nested content, media APIs for attachments, core link controls
for URLs — and `InspectorControls`/`PanelBody` for settings, reusing the existing
panel vocabulary. Repeated data follows `references/editor-and-repeaters.md`. No
persisted UI state.

### 6. Write `render.php` (dynamic blocks)

`get_block_wrapper_attributes()` on the root; your own `data-*`/style merged into
its argument array; escaping by context (`esc_html`, `esc_attr`, `esc_url`,
`wp_kses_post`); scoped colours resolved through the theme helper; every string
translated in the **target theme's** domain. Keep `render.php`'s class names
aligned with the block stylesheet.

### 7. Wire behaviour and styles

Add `view.ts` only if the block needs front-end behaviour: idempotent init guarded
by state attributes, a loading → ready swap, reserved layout space, reduced-motion
and no-JS paths. Front-end CSS is authored in the block's `style.css` (or the view
script's CSS import) and referenced through `style`/`editorStyle`; inspector and
canvas-preview CSS stays scoped to the block, never to `wp-admin` chrome.

### 8. Build, then verify on the real surfaces

Run the target's own script from the theme root (`npm run build:blocks`, or
`npm run build`, or the child's `npm run build:blocks`). Never hand-edit
`index.js`, `index.asset.php`, `view.js` or `view.css`. Then apply
`../nextora-ecosystem/references/quality-gates.md` and its §8 block recipe plus the
block-specific steps in `references/block-contract.md` §11.

## Hard limits

- No second React runtime, no blanket `@wordpress/scripts` pipeline, no second
  bundler. Reuse the theme's esbuild setup.
- Adding an unlisted `@wordpress/*` import builds but can read an undefined
  `window.wp.*` global: the asset manifest only lists a fixed 14-package handle set.
  Check the global and its handle before importing, or avoid the import.
- Do not add a library the block does not use; libraries already in
  `package.json` are ranges, so read the lockfile before relying on an API.
- Do not change a persisted attribute shape without a `deprecated` entry or a
  one-time migration, and never convert an index-keyed list schema to an ID-keyed
  one (or back) to make an editor helper convenient.
- Do not add REST routes, caching, telemetry, admin pages or unrelated
  abstractions to satisfy a block request.
- Do not modify the source themes while delivering skills or documentation.

## Completion checks

- Target theme and block host identified from their own files; nothing claimed
  stands on unread source.
- Reuse decision stated, or a concrete reason the block had to be new.
- `block.json` names a real namespace/category/textdomain pair and points only at
  built assets; the block appears under the namespace category in the inserter.
- `save` matches the block's storage kind, verified by save → reload → re-save with
  no block-validation warning.
- Visible text, nested content, media and URLs use native editing controls; the
  item editor is a single dialog; no UI state is persisted.
- `render.php` escapes by context and translates in the target domain; scoped
  colours resolve through the theme helper and empty values follow theme defaults.
- View behaviour is idempotent, reserves layout, and respects reduced motion and
  no-JS.
- Build output regenerated with the target's own script; generated files untouched
  by hand.
- Relevant §8 acceptance steps executed, or the unverified surface reported plainly.

## Boundaries

Markdown instructions and source-backed references only. This skill does not ship
runtime scripts, generators, copied theme trees, package management or installation
automation, and it does not modify the parent or child theme sources.
