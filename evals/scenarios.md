# Evaluation scenarios

An instruction-suite evaluation corpus: seven prompts, the source fixtures each
one depends on, the observable result it must produce, and the conditions that
count as failure. It is not a WordPress fixture application and not an automated
scoring framework — a person or an agent runs a prompt and inspects the answer.

The point is to prove that the instruction files load and change what an agent
does. A scenario result is *not* proof that a generated theme runs; for that, use
the downstream acceptance recipes in
`../skills/nextora-ecosystem/references/quality-gates.md`.

## How to run

Run each scenario in a **fresh** shell pane from this repository root, with the
skill file supplied explicitly and tools restricted so the run cannot write:

```bash
omp -p --no-session --no-skills --tools read,glob,grep \
  @skills/<skill-name>/SKILL.md "<prompt>"
```

`--no-skills` keeps the agent from picking up unrelated skills; `--tools
read,glob,grep` makes the run read-only. Every prompt must be prefixed with the
two roots so the agent can actually read the sources — never write "the source
roots supplied in this plan" without pasting them:

```text
Source roots (read-only; do not modify):
- parent theme root: <absolute path to the Nextora parent theme>
- sample child theme root: <absolute path to the sampled Nextora child theme>
Load the skill's referenced Markdown files by their relative paths.
```

Rules for the runner:

- Wait for the command to finish before reading the pane; a submitted command is
  not a completed run.
- Do not reuse a pane that is still working.
- Do not answer an approval dialog on the agent's behalf.
- The source themes are read-only. A scenario that edits them has failed.

## Scenarios

### 1. Router dispatch — `skills/nextora-ecosystem/SKILL.md`

**Prompt**

> Create a preset-switchable Nextora child named nextfolio; add an editable team
> block and compose a landing-page pattern. Read the source roots given above;
> output a file map and implementation outline only.

**Source fixtures** — parent `blocks/blocks.php`, `theme.json`,
`resources/css/app.css`; child `style.css`, `functions.php`,
`inc/assets/enqueue.php`.

**Expected artefact / behaviour**

- Routes to **all four** specialists: child themes, blocks, design system,
  composition.
- One consistent naming scheme throughout: theme slug `nextfolio`, PHP prefix
  `nextfolio_`, constants `NEXTFOLIO_VERSION` / `NEXTFOLIO_DIR` / `NEXTFOLIO_URI`,
  text domain `nextfolio`, blocks `nextfolio/<block-slug>`.
- The child declares the real parent (`Template: nextora`) instead of cloning it.
- The outline reuses the parent's reusable pieces (registrar pattern, tokens,
  existing blocks/patterns) rather than inventing parallel ones.
- Output is a file map and outline only; no files are written.

**Failure conditions**

- Cloning the parent theme into a new standalone theme, or proposing a
  plugin-shaped solution.
- Routing to a single specialist for this multi-part request.
- `nextsaas`-derived slug/prefix left in place, or parent APIs mechanically
  renamed (`nextora_*` → `nextfolio_*`).
- Writing files, or editing either source theme.

### 2. Child theme contract — `skills/nextora-child-themes/SKILL.md`

**Prompt**

> Using Nextsaas only as a reference, output the style.css header, minimal
> functions.php, and asset-hook snippets for nextfolio. Do not write files.

**Source fixtures** — child `style.css`, `functions.php`,
`inc/bootstrap/version.php`, `inc/assets/enqueue.php`.

**Expected artefact / behaviour**

- `style.css` headers: Theme Name, Version synchronised with the version
  constant, `Template: nextora`, `Text Domain: nextfolio`, Requires at least,
  Requires PHP, License and License URI; no `Tested up to` copied from the parent.
- `functions.php`: `strict_types`, `ABSPATH` guard, `NEXTFOLIO_DIR` / `NEXTFOLIO_URI`
  defined once, Composer autoloader required only when readable.
- **Nothing at child file scope calls a parent function or reads a `NEXTORA_*`
  constant**; parent handles (`global-styles`, `nextora-app`, `nextora-main`) are
  referenced only inside hooks.
- Correct hook priorities: styles on `wp_enqueue_scripts` 25, scripts on
  `wp_enqueue_scripts` 30, canvas styles via `after_setup_theme` 20 plus
  `add_editor_style()`.
- Output basenames do not collide with the parent's (`nextfolio.css`,
  `nextfolio-editor.css`, `nextfolio.js` — not `app.css` / `main.js`).
- Missing compiled files are guarded, and the guard is described as making the
  failure visible, not as success.

**Failure conditions**

- `Template: nextsaas`, or no `Template:` header at all.
- A parent function call or `NEXTORA_*` read at file scope.
- Compiling to `assets/css/app.css` or `assets/js/main.js`, or calling
  `add_editor_style()` with either name.
- Missing `ABSPATH` guard or `declare( strict_types=1 )`.
- An unconditional `require_once` of the Composer autoloader.
- Treating a `file_exists()` guard as proof the asset loaded.

### 3. Preset-aware defaults — `skills/nextora-design-system/SKILL.md`

**Prompt**

> Output the CSS role bindings and preset data for nextfolio with DM Sans body and
> Bodoni Moda headings by default. It must follow visitor font/color changes;
> explain behavior when the switcher constant is absent.

**Source fixtures** — parent `resources/css/app.css` (`@theme` role variables),
`theme.json`, `styles/font-preset/*.json`, `scheme-switcher.json`,
`inc/features/color-scheme-switcher/scheme-data.php`; child
`resources/css/modules/00-tokens.css`.

**Expected artefact / behaviour**

- Child CSS sets the default roles on `:root` by pointing the existing Nextora
  role variables at registered presets, e.g. body → the DM Sans family slug,
  heading → the Bodoni Moda family slug, button → the body role.
- Child semantic aliases consume those roles, and `theme.json` body/heading
  styles reference them.
- Fonts are registered in the merged `settings.typography.fontFamilies` with
  `fontFace` entries and real file paths — not variation-only declarations.
- The explanation states that the child **overrides** the parent's already-defined
  role defaults, and why adding fallback arguments instead would not work.
- `NEXTORA_COLOR_SCHEME_SWITCHER === true` is described as a strict gate: absent
  or false means no payload, no script, no page-theme CSS and no popover — and
  that the correct behaviour is to contribute data only, not to build a switcher.
- Colour roles stay the canonical parent roles.

**Failure conditions**

- Pinning new `--nextfolio-font-*` variables to literal font family names so the
  visitor font switch never reaches them.
- Defining a role in terms of itself.
- Claiming a new switcher, or a new fallback policy, is part of the work.
- Copying the sample's child-only palette entries (`cobalt`, `mist`) as the model
  for a switchable child.
- Treating the inert gate as "the feature still works, just without UI".

### 4. Block editing and repeaters — `skills/nextora-gutenberg-blocks/SKILL.md`

**Prompt**

> Output editor/renderer excerpts for a dynamic team block with visual
> heading/body and members managed by a list plus modal; members may have nested
> links. Explain Add/Done/Cancel/Escape, empty list, reorder and save/reload. No
> files.

**Source fixtures** — parent `blocks/blocks.php`, `scripts/build-blocks.mjs`,
`blocks/team-section/{edit.tsx,member-edit-form.tsx,member-utils.ts,render.php}`;
child `resources/ts/editor/{repeater.tsx,rich-text-control.tsx,media-control.tsx}`,
`blocks/blocks.php`.

**Expected artefact / behaviour**

- Metadata: `apiVersion: 3`, namespaced name, matching text domain, `editorScript`
  on the built `index.js`, `render: "file:./render.php"`, `save: () => null` for
  an attribute-rendered dynamic block; `viewScript` only if the block needs one.
- Heading and body use a visual rich-text field with `core/bold`, `core/italic`
  and `core/link`; not a plain text control as the only editor.
- One WordPress `Modal` per list; list shows compact labelled rows; add/edit and
  the modal are the only list UI.
- **Add/Duplicate insert the record before opening its editor**; Cancel and Escape
  discard *subsequent field edits* but keep the inserted item — they do not roll
  the list back. Editing an existing committed item and cancelling leaves it
  unchanged.
- Done commits one immutable array; no stored record is ever mutated in place.
- Row actions come from a single `DropdownMenu` in the order Move up, Move down,
  Duplicate, Remove, with boundary actions disabled and Remove disabled at the
  minimum.
- Nested item lists expand **inside** the open dialog; no second dialog is stacked.
- Modal open/draft state is component state, never persisted in attributes.
- Renderer uses `get_block_wrapper_attributes()`, `wp_kses_post()` (or a narrower
  allowlist) for rich values and `esc_html` / `esc_url` / `esc_attr` by context.
- Save → reload → re-save is stated as the compatibility check.

**Failure conditions**

- Introducing DataViews or another list framework instead of the existing model.
- Stacking a second `Modal` for nested items.
- A plain text control as the only editor for the heading/body.
- Persisting modal or draft state in block attributes.
- Claiming Cancel/Escape removes a just-added or duplicated item.
- `save: () => null` on a block that persists `InnerBlocks` content.
- Echoing stored rich HTML without server-side sanitisation.
- Assuming the parent's block generator exists in a child theme.

### 5. Scoped colour storage — `skills/nextora-design-system/SKILL.md`

**Prompt**

> For a scoped icon surface, show storage/render output for preset Secondary,
> #039590ff and custom #03959080; then switch palettes. Explain why settings
> controls do not sanitize stored HTML.

**Source fixtures** — parent `blocks/advanced-icon/color-utils.ts`,
`blocks/advanced-icon/lucide.php`, `blocks/advanced-container/render.php`,
`theme.json`, `resources/css/app.css`, `inc/features/color-scheme-switcher/`;
child `resources/css/modules/00-tokens.css`.

**Expected artefact / behaviour**

- A preset choice stores the **slug** (`secondary`), not the hex value it happens
  to have today.
- A full-opacity hex equal to a palette entry normalises to that entry's slug.
- `#039590ff` is treated as an opaque value; `#03959080` carries alpha and stays
  literal.
- PHP handles 8-digit hex **before** `sanitize_hex_color()`, which does not accept
  alpha, and resolves preset references to
  `var(--wp--preset--color--secondary)`. Palette switching then flows through
  automatically.
- The override attribute uses a non-reserved name for the inner surface; wrapper
  appearance stays with the core supports.
- The explanation separates editor affordance from security: the settings control
  restricts what the UI offers, and the stored value still passes server-side
  KSES/sanitisation on output.

**Failure conditions**

- Storing a literal hex for the preset case, which freezes the colour against
  future palette changes.
- Passing an 8-digit hex through `sanitize_hex_color()` and losing the value.
- Claiming the sidebar control sanitises stored HTML, or that a restricted toolbar
  is validation.
- Reusing a reserved wrapper attribute for an inner surface.

### 6. Composition — `skills/nextora-site-composition/SKILL.md`

**Prompt**

> Output a nextfolio landing template and insertable pattern using an existing
> Nextora team block and inherited header/footer; explain an unchanged frontend
> after a file edit.

**Source fixtures** — parent `templates/`, `parts/`, `patterns/`, `theme.json`
(`customTemplates`, `templateParts`); child `templates/front-page.html`,
`parts/header.html`, `patterns/page-contact-us.php`.

**Expected artefact / behaviour**

- Valid serialized block markup and a correct pattern header (Title, Slug,
  Categories, Inserter) with a unique pattern slug and the child text domain.
- The full-page composition has **one authority**: a hidden filesystem pattern
  (`Inserter: no`) referenced from a thin template with `wp:pattern`; its section
  list is not duplicated across files.
- Existing blocks are reused (for example the parent team block) and no new block
  is introduced for behaviour that existing blocks already express.
- Template-part references point at the theme that owns the part; a child-owned
  part is not pinned to the parent via a copied `theme` attribute.
- Block templates and parts are HTML; no classic PHP page template.
- The explanation of an unchanged frontend names a database-customised template or
  style in the Site Editor as the likely cause, and says to inspect or reset only
  a disposable environment — never to overwrite a real site's customisations.

**Failure conditions**

- A classic PHP page template, or redeclaring a parent-only custom template as a
  child-owned one.
- Duplicating the same section list into several files so they drift.
- Pinning a child part to the parent theme.
- Shipping mock posts as production content.
- Creating a custom block where the parent team block already fits.
- Recommending a destructive reset of a real site's customisations.

### 7. Switcher precedence and baseline — `skills/nextora-design-system/SKILL.md`

**Prompt**

> On the existing source, a saved bundled theme is active and the URL has only
> ?color=midnight; a child adds a duplicate preset title and a malformed preset
> file. Explain the actual outcome and the declared-versus-tested WordPress
> baseline.

**Source fixtures** — parent
`inc/features/color-scheme-switcher/scheme-data.php`,
`assets/js/scheme-switcher.js`, `scheme-switcher.json`, `styles/color-scheme/*.json`,
`style.css`; child `style.css`, `theme.json`.

**Expected artefact / behaviour**

- Bundled-theme resolution follows URL theme → direct page theme → valid stored
  theme → server-resolved cookie/page fallback → configured initial theme.
- When a bundled theme resolves, its colour **and** font pairing applies instead
  of the independent colour and font axes, so a bare `?color=midnight` does not
  override an already resolved bundled theme.
- The independent path is URL → saved choice → initial → defaults.
- A duplicate preset title is resolved by scan order — parent colour-scheme, child
  colour-scheme, parent top-level styles, child top-level styles — with the later
  entry winning on `sanitize_title( title )`; it is not an unconditional
  child-wins rule.
- A missing, unreadable or malformed preset file is skipped by the existing
  parser; unknown selections are validated against the available presets.
- The baseline answer separates *declared* (`Requires at least: 6.4`,
  `Requires PHP: 8.1`) from *proven* (v3 `theme.json` and current package APIs
  need runtime checks), and refuses to promise fixed behaviour across versions.
- No new switcher, persistence format or fallback policy is proposed.

**Failure conditions**

- "The colour URL parameter always wins."
- "The child's preset always wins because children load later."
- Claiming a malformed preset file is fatal or that it silently changes a palette.
- Claiming WordPress 6.4 compatibility is proven by the header.
- Inventing a new fallback or reset policy for the switcher.

## What these scenarios do and do not prove

They prove that each entrypoint loads, that its references resolve, and that it
changes the agent's implementation choices in the expected direction — naming,
hook priorities, storage format, precedence, sanitisation and edit semantics.

They do **not** prove that a generated child theme, block or template runs. That
requires the builder plus a disposable WordPress install and a browser, as
described in `../skills/nextora-ecosystem/references/quality-gates.md`. Record
each run honestly: which skills were exercised, what the output contained, and
which surfaces remain unverified.
