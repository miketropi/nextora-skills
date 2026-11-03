# Quality gates

The rules every specialist in this suite applies before calling work done. Each
gate is a decision rule with a failure mode, not a checklist of aspirations.

Related: `source-map.md` (where these come from), `../../nextora-ecosystem/SKILL.md` (routing).

## 1. Data compatibility

- **Never change the shape of a persisted attribute without a migration.** New
  attributes need no legacy alias; an attribute whose meaning or type changes
  needs either a `deprecated` entry on the block (with `save`/`migrate`) or an
  explicit one-time migration for the affected content, and both the old and new
  serialized forms must be verified.
- **Adding an `id` to an existing index-based list schema is a breaking change.**
  Keep index-keyed schemas index-keyed; keep ID-keyed schemas ID-keyed. Do not
  convert one into the other to make an editor helper convenient.
- **Pick `save` by what the block stores.** A block that renders from attributes
  in PHP is dynamic (`render`/`render.php`, `save: () => null`). A block that
  stores block markup is static and must save exactly what the parser expects. A
  block that persists nested blocks must save `InnerBlocks.Content` — `save: () =>
  null` there silently discards the user's content.
- **After any attribute change, test save → reload → re-save** and confirm no
  block validation warning appears in the editor.

## 2. Editor and front-end parity

- Anything the editor previews must come from the same source of truth the front
  end uses: the same attributes, the same tokens, the same CSS surface. A second
  hardcoded preview copy is a defect waiting to drift.
- Editor styles stay scoped to the block canvas and inspector. Never restyle
  `wp-admin` chrome, global form controls or `.components-*` internals from a
  theme stylesheet.
- Verify the rendered front end, not only the editor: a dynamic block that renders
  correctly in `ServerSideRender` can still be enqueued wrongly on the front end.

## 3. Assets

- Conditional and lazy loading must be **idempotent**: initialisation runs once
  per element, guarded by an init attribute or a pending/inited state, and a
  second pass must not rebuild listeners, rebind observers or duplicate markup.
- Missing build output must be visible, not silent. Guard a `file_exists()` read,
  then make the failure surface during verification — an empty guard that hides a
  missing compiled asset is not a fix.
- **A generated artefact and a test fixture must not leak into the shipped tree.** Test doubles,
  temporary block folders, throwaway manifests and build scratch files belong in a temp directory
  that the code under test is pointed at — never inside the theme, and never "cleaned up in a
  `finally`", because an aborted run leaves the artefact behind. The consequence is not cosmetic:
  a theme registrar that globs `blocks/*` registers any directory holding a readable `block.json`,
  so a leftover fixture becomes a real block in the inserter and in every runtime cell, and any
  test that iterates `blocks/*/block.json` starts failing for a reason the next person cannot see.
  If a README or a completion check claims a suite writes nothing into the theme, verify that by
  reading the tests rather than asserting it.
- **Assert what the composition actually guarantees, not a marker only some of its parts emit.** A
  front page assembled from a mix of custom blocks and core blocks cannot assert one custom-block
  data attribute per section — the core-block sections legitimately carry none — and a wrapper
  regex that depends on a single attribute order or a single newline fails for a reason that has
  nothing to do with the theme. Count the thing the composition promises (the rendered sections, the
  resolved pattern, the composed structure), and parse markup with a parser rather than pattern
  matching on attribute order.
- Reuse the target's own pipeline (its `build:blocks` / `build:css` / `build:ts`
  scripts and its esbuild and PostCSS configuration). Never add a second React
  runtime, a blanket `@wordpress/scripts` pipeline, or an unrelated bundler.
- Do not assume a parent bundle exposes a global, and do not bundle a library the
  feature does not use.
- Defer/reserve layout space for anything that hydrates (carousels, canvases) so
  the content does not jump: an explicit `aspect-ratio`/`min-height` plus a
  loading → ready state.

## 4. Security

- **Escape on output, by context**: `esc_html()` for text, `esc_attr()` for
  attributes, `esc_url()` for URLs, `wp_kses_post()` or a narrower allowlist for
  rich HTML. Sanitize on input as well when the value is consumed outside a
  single render path.
- **A restricted toolbar is not a security boundary.** Limiting the editor to
  bold/italic/link changes what the UI offers; PHP must still sanitize the stored
  HTML on output, because stored content can be written by other means.
- **Privileged writes need both a capability check and a nonce** (`check_admin_referer()`
  / `wp_verify_nonce()`). A capability check alone does not stop CSRF.
- **Every REST route needs a `permission_callback`.** If the endpoint is meant to
  be public, `'__return_true'` is a deliberate, documented decision — never an
  omission.
- **Use a prepared statement only when a custom query is genuinely needed.** Most
  data needs go through `WP_Query`/`get_posts()` with sanitized arguments. When a
  block takes query attributes, clamp page sizes and validate post types,
  taxonomies and terms against allowed values; a prepared statement does not make
  attacker-chosen `post_type` or `taxonomy` safe.
- Never print a raw stored value because the control that produced it is
  restricted.

## 5. Internationalisation

- Every user-visible string goes through `__()`, `_e()`, `_n()`, `_x()` or
  `esc_html__()` with the **target theme's** text domain — never a hardcoded or
  parent domain in child code.
- Block metadata declares `textdomain`; editor scripts use the translated
  strings. **Check whether core already did the registration before calling
  `wp_set_script_translations()` yourself:** for a block registered from `block.json`, core calls it
  automatically when the manifest declares a non-empty `textdomain` **and** the built asset's
  dependency list contains `wp-i18n` (`wp-includes/blocks.php:321-323`), and `i18n` is one of the
  14 handles the block build emits. So a block bundle that imports `@wordpress/i18n` is already
  covered, and a manual call there is redundant. Call it explicitly for an **editor** script that
  is *not* a `block.json` asset (the theme's own `resources/ts/editor/` bundle, a standalone
  `enqueue_block_editor_assets` script) or when the dependency list does not include `wp-i18n`.
  Neither source theme calls it at all, which is consistent with this rule — not evidence that the
  gate is optional. **[verified against core source, 2026-11-03]**
- Pluralisation and count messages use `_n()`/`sprintf`, not string concatenation.
- **In a block theme, separate code strings from starter content before applying
  this gate.** Copy that ships as block markup in `templates/*.html` and pattern
  bodies is content: core translates only a pattern's `title`/`description` header
  fields (`wp-includes/block-patterns.php`) and nothing at all in template HTML, so
  there is no mechanism that makes a block-theme template's literal copy
  translatable — and rewriting HTML templates into PHP to wrap them is not the fix.
  Everything a **code path** emits is a code string and is covered in full: PHP
  output, editor-script and view-script literals, localized data objects, REST
  payload text, labels created in JavaScript. If a project needs translatable
  starter content, say so explicitly and name how; never claim this gate is met for
  copy that ships untranslated.
- Pattern bodies are PHP, so `esc_html__()` belongs on chrome-like literals a
  pattern must render (a button label, a caption) — not on page copy the owner is
  expected to rewrite.

## 6. Accessibility and resilience

- Semantic elements and a sane heading order; a heading is a heading, not a
  styled paragraph.
- Decorative images get empty `alt`; meaningful images get an author-editable
  `alt` that survives selection, replacement and removal.
- Keyboard: every interactive control is reachable and operable by keyboard, focus
  is visible, and focus returns to the trigger when a dialog closes.
- Dialog/list mutations announce their outcome through an ARIA live region that
  is rendered from the first render, so assistive technology is already observing
  it when the text changes.
- **Contrast is measured, not assumed.** WCAG 2.2 AA is the threshold: **4.5:1** for normal-size
  text, **3:1** for large text (≥ 24 px, or ≥ 18.66 px bold) and for non-text UI — focus rings,
  control borders, input outlines, meaningful icons. It applies to the *composited* surface: a
  background colour, a background image plus its overlay, or the nearest opaque ancestor. Roles are
  positional and their values invert per scheme, so a pair must be verified in the default scheme
  **and** in a dark or alternative one, and a section must select an offsetting pair of roles
  rather than "a light colour for dark sections". Colour must never be the only state cue. A
  passing pair for a photograph cannot be asserted from a colour string: the overlay is the
  mechanism, and the worst-case region behind the text has to be measured. Rules and the
  per-surface recipe:
  `../../nextora-design-system/references/presets-and-switching.md` §3.1.
- `prefers-reduced-motion: reduce` disables non-essential motion, and the content
  is fully readable and operable with JavaScript disabled.
- **Every space has exactly one named owner.** Space around a block belongs to core block supports
  (wrapper margin/padding); a section's vertical rhythm belongs to the section's own padding; the
  gap between elements inside it belongs to the block's own gap attribute. Two controls writing the
  same property on the same element is a defect. An empty spacing control emits nothing, and a
  spacing value that can follow the theme's scale is stored as preset identity
  (`var:preset|spacing|<slug>`) rather than as a copied length. Ownership table:
  `../../nextora-design-system/references/presets-and-switching.md` §3.2.
- **Section rhythm survives being stacked.** No boundary may depend on margins collapsing, and the
  boundary is verified as two non-zero computed paddings (previous bottom, next top) plus a
  non-touching pair — first and last sections included. A rhythm that holds at one width and
  collapses at another is a defect; see
  `../../nextora-site-composition/references/templates-and-patterns.md` §5.2.
- Layout holds at narrow and wide widths; long and empty strings do not break it.

## 7. Declared versus tested baseline

- A theme's `Requires at least` / `Requires PHP` / `Tested up to` headers are
  **declarations**, not proof. Both source themes declare WordPress 6.4+ / PHP 8.1+
  while using `theme.json` v3 and recent package APIs.
- Before using a newer core API or metadata key, check what the declared minimum
  actually supports. On an older supported target, use the equivalent stable API
  or a compatible metadata form.
- If the requested feature genuinely needs a higher minimum, surface that
  requirement to the user — never raise the declared minimum silently.
- Never copy the parent's `Tested up to` value into a new theme as if it had been
  tested. Set it after real runtime proof.
- Read the target's `.nvmrc` and `package.json` for the Node version and the
  commands that actually exist.

## 8. Downstream acceptance recipes

These prove a *generated* theme, block or composition. They are not run by this
instruction suite; in this repository no theme is installed, seeded or built.

**Prerequisites.** The target theme's own working directory, its npm and Composer
dependencies installed (`npm ci` / `composer install` as its README states), a
**disposable** WordPress site with the parent and the target activated, and
working admin access. Use only commands present in the target's `package.json`:
the parent has `build:blocks`, `build:css`, `build`, `typecheck`, `lint:php:all`
and `test:wp`; the sample child has `build`, `build:css`, `build:ts`,
`build:blocks`, `typecheck`, `lint:php:all` and `test:wp`, but **no** `gen`.
`test:wp` is a server-side workflow: read the target's `scripts/wp-test.sh` and
`blueprint.json` before promising a port or a login. Then verify in a browser —
build success and a server workflow are not proof of runtime behaviour.

**Blocks and repeaters.**

1. Insert the block in the editor and visually edit rich content: apply bold, add
   a link, then clear the field entirely.
2. Add two items; open one, press **Cancel**, and confirm the record that was
   already committed is unchanged.
3. Open another, press **Done**, and confirm the new values are committed.
4. Duplicate, move up, move down at the boundaries (actions disabled), and remove
   down to the minimum (removal disabled at the minimum, message announced).
5. Nest a sub-list and its link editing inside the open dialog and confirm no
   second dialog appears.
6. Undo, redo, save, reload the editor, and compare the front-end order and text
   with the editor. Re-save and confirm no validation warning.

**Design system and the switcher.**

1. Change a site style variation in the Site Editor and confirm the front end
   follows it (this is the persistent path).
2. Separately, with no bundled theme or page choice and clean storage/cookies,
   visit with a colour/font query (for example `?color=midnight&font=elegant`) and
   assert the **computed** colour values and `font-family` of real elements — not
   merely that a custom property exists. Then reload and confirm persistence, and
   confirm the Reset control restores the defaults.
3. Test a bundled-theme URL and a direct page-theme choice separately; they follow
   their own precedence, not the independent colour/font path.
4. With the switcher feature gate disabled, confirm its script, global and
   popover are all absent.
5. Confirm a custom alpha colour survives to the rendered output, inherited
   blocks are still styled, child font files actually load, and the `wp-admin`
   chrome is unchanged.

**Colour contrast and section rhythm (computed values, not declarations).**

1. For each surface that can take a background, read `getComputedStyle(el).color` for a heading, for
   body copy, for a link, for a button label, and for the focus ring, plus the **effective**
   background — the element's own `background-color`, or the nearest opaque ancestor when it is
   transparent, composited with any overlay layer. Assert the thresholds in §6. Do this for: theme
   defaults (every control empty), a light custom background, a dark preset background, an image
   with an overlay, a custom literal colour, a partial-alpha literal, and an empty override.
2. Repeat after changing a **Site Editor style variation** and, separately, a **visitor preset**.
   Preset-backed colours must follow; literal ones must not move; every resulting pair must still
   pass.
3. For an image-backed surface, measure the worst-case region behind the text (sample the rendered
   pixels, not the image average) and confirm the overlay's opacity and colour are what make it
   pass. If pixel sampling is not run, report that surface as unverified rather than claiming the
   overlay guarantees readability.
4. Read the computed padding at every section boundary — header→first, each adjacent pair, and
   last→footer — at a narrow and a wide width, plus the internal heading/prose/card gap. Each
   measurement must be non-zero where authored, must match the authored preset, and must come from
   exactly one owner. Clear a spacing control and confirm the theme default returns with no empty
   inline declaration left behind.

**Composition.**

1. Confirm each template and part resolves to the intended theme (a copied
   `theme` attribute can pin the wrong one).
2. Confirm patterns insert, parse and remain editable.
3. Edit a template file on disk and reload; if nothing changes, look for a
   database-customised template or style that is masking it. Inspect or reset only
   the disposable environment — never silently overwrite a real site's
   customisations.
4. Check both the requested minimum WordPress version and the current supported
   release before claiming compatibility.

If runtime is unavailable, say so plainly: report that generated-theme and
browser behaviour remains unverified. A metadata check is not a substitute.
