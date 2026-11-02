# Quality gates

The rules every specialist in this suite applies before calling work done. Each
gate is a decision rule with a failure mode, not a checklist of aspirations.

Related: `source-map.md` (where these come from), `../nextora-ecosystem/SKILL.md` (routing).

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
  strings and mark the script as translated (`wp_set_script_translations()`)
  when it carries translatable literals.
- Pluralisation and count messages use `_n()`/`sprintf`, not string concatenation.

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
- `prefers-reduced-motion: reduce` disables non-essential motion, and the content
  is fully readable and operable with JavaScript disabled.
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
