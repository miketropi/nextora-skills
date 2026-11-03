---
name: nextora-child-themes
description: Create, bootstrap or audit a child theme of the Nextora parent block theme — the style.css header with Template nextora, the functions.php load order, the single version constant kept in sync with the header, the front-end and editor-canvas asset pipeline with its parent handle dependencies, the child block registrar and inserter category, PHP/i18n conventions, and the optional feature-file layout. Use when a request names a Nextora child, a new child theme name or slug, a child theme header or version bump, child CSS/JS enqueueing, editor canvas styles, a child block folder or block category, porting Nextsaas conventions into a new child, or asks why child code cannot call the parent at file scope. Pair with nextora-design-system for any child that must follow palette or visitor font/colour switching.
---

# Nextora child themes

Creates and audits child themes of Nextora: the bootstrap, the asset pipeline and
the child block registrar. Parent-level conventions live in the parent itself;
block internals belong to `../nextora-gutenberg-blocks/SKILL.md`, and anything
about palette, fonts or the visitor switcher belongs to
`../nextora-design-system/SKILL.md`. When a child is being rebuilt from an
external source template, `../nextora-template-porting/SKILL.md` supplies the
evidence-backed dossier and phased roadmap this skill implements against; it
supplies analysis and planning only and never overrides the child contract below.

## References to load

Read these before writing any file, by relative path:

- [`references/child-contract.md`](references/child-contract.md) — the slug/prefix
  substitution table, the exact directory recipe, the header, the bootstrap and
  enqueue code, the registrar, the PHP conventions, the sample's observed defects
  and the downstream WordPress acceptance recipe. **Load it at step 3 and do not
  write code before you have.**
- [`source-map.md`](../nextora-ecosystem/references/source-map.md) — where every
  rule comes from, with provenance tags and the facts that are commonly
  mis-stated.
- [`quality-gates.md`](../nextora-ecosystem/references/quality-gates.md) — the
  gates the finished child is measured against. Load it before declaring
  completion.

Reading the local sources outranks reading these files. Every value quoted in a
skill or reference is an anchor, not a current fact.

## Ordered workflow

### 1. Resolve the roots, do not assume them

In a live install the parent is the template directory and the child is the
stylesheet directory:

```php
get_template_directory();      // parent root
get_stylesheet_directory();    // active theme root (child when one is active)
```

Outside an install, find a `style.css` with `Theme Name: Nextora` for the parent
and one with `Template: nextora` for the child. If the parent source is genuinely
missing, say so and stop claiming Nextora compatibility; complete only the work
that does not depend on it. Never fabricate an anchor and never substitute a
different theme silently.

### 2. Read the evidence before proposing anything

From the **target** child — or the sample child, when the sample is what is being
ported:

1. `style.css` — the real header: `Template`, `Version`, `Requires at least`,
   `Requires PHP`, `Tested up to`, `Text Domain`, `License`.
2. `functions.php` — load order and what the theme requires at file scope.
3. `theme.json` — `version`, palette/font slugs, `customTemplates`,
   `templateParts`.
4. `package.json` and `.nvmrc` — which build scripts exist, and on which Node.
5. `inc/`, `blocks/`, `resources/`, `assets/` — what already exists.

From the **parent**: `style.css`, `functions.php`, `inc/assets/assets.php`,
`blocks/blocks.php`. These are what a child depends on, so their handle names and
registrar behaviour are the contract, not the documentation.

### 3. Fix the naming table before writing files

Load `references/child-contract.md` and fill its substitution table for the
requested slug. Every generated identifier — PHP functions, constants, text
domain, block names, CSS classes, custom properties, pattern slugs, handles and
file basenames — comes from that one slug. Parent-owned names (`nextora_*`,
`NEXTORA_*`, `nextora/*`, `global-styles`, `nextora-app`, `nextora-main`, the
parent palette and font slugs, the `--nextora-font-*` roles) are never renamed.

### 4. Lay out only the directories the child needs

The recipe is in `references/child-contract.md`. Required: root `style.css`,
`theme.json`, `functions.php`. Add `inc/`, `resources/`, `assets/`, `templates/`,
`parts/`, `patterns/` as applicable, `blocks/<block-slug>/` only when the child
actually ships custom blocks, and fonts plus licence files only when fonts ship.
Do not create empty feature scaffolding, and do not clone a whole theme tree.

### 5. Write the header, then the version constant, then the bootstrap

Order matters: `inc/bootstrap/version.php` defines the single
`<CHILD_PREFIX>_VERSION` constant, the `style.css` header `Version:` must equal
it, and the asset pipeline falls back to it, so the constant must exist before
anything requires the pipeline. In the header, declare only real minima — never
copy another theme's `Tested up to` value.

### 6. Write `functions.php` with child-before-parent in mind

WordPress loads a child's `functions.php` **before** the parent's, so nothing at
file scope may call a parent function or read a `NEXTORA_*` constant. Define the
child's own `_DIR`/`_URI` from the stylesheet directory, require the Composer
autoloader only when it is readable, and require the child's own files. Every
parent-dependent operation belongs inside a hook, and references parent handles by
name and checks they are registered first.

### 7. Wire the asset pipeline

Child stylesheet on `wp_enqueue_scripts` priority 25, depending on the parent's
`global-styles` and `nextora-app` when those are registered; child script on
priority 30 depending on `nextora-main` when registered; editor canvas styles on
`after_setup_theme` priority 20 through `add_editor_style()`. Guard each enqueue
with a readable-file check, and use non-colliding output basenames — a child
`assets/css/app.css` or `assets/js/main.js` can shadow the parent's asset.
Full code and hook table: `references/child-contract.md`.

### 8. Add the block registrar only when the child ships blocks

WordPress does not scan a theme's `blocks/` folder. Register each directory that
contains a readable `block.json` on `init` from `get_stylesheet_directory()`, add
the child inserter category, and mirror the category coercion client-side.
Block metadata, editor UI and renderers are `../nextora-gutenberg-blocks/SKILL.md`.
No `blocks/` directory means no registrar requirement.

### 9. Hand appearance to the design system

If the child must follow preset or visitor colour/font switching, load
`../nextora-design-system/SKILL.md` in the same session and let it decide the
role bindings, `theme.json` font families and preset files. A child's body,
headings and buttons resolve through the Nextora role variables; they are never
pinned directly to one sample design's fonts.

### 10. Verify against the running site

Run only the build commands the target child's own `package.json` declares, then
load a front-end page and the editor canvas and confirm the assets are actually
enqueued and the guard did not hide a missing build. The acceptance recipe in
`references/child-contract.md` is the browser-level proof. Mark anything you did
not exercise as unverified.

## Completion checks

- Parent and child resolved from their own `style.css`, not assumed or hardcoded.
- `Template: nextora` present; header `Version` equals `<CHILD_PREFIX>_VERSION`.
- `Text Domain:` equals the child slug, and every translatable string in child
  PHP/JS uses that domain.
- No parent function call and no `NEXTORA_*` read at child file scope.
- Every enqueue/editor-style registration is guarded by a readable-file check and
  a `wp_style_is()`/`wp_script_is()` check before adding a parent handle.
- Child asset basenames do not collide with the parent's `app.css` / `main.js`.
- Block registrar exists only if `blocks/` exists, uses the stylesheet directory,
  skips unreadable `block.json`, and registers the child category.
- `references/child-contract.md` followed for structure, and
  [`quality-gates.md`](../nextora-ecosystem/references/quality-gates.md) applied.
- Front-end and editor canvas actually load the child assets.

## Boundaries

This skill is Markdown instruction. It ships no generator, starter theme tree,
package manager or installer, and it never edits the parent or the sample child.
The parent's `npm run gen` scaffold is parent-only; a child theme is created by
writing the files described here.
