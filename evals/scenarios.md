# Evaluation scenarios

An instruction-suite evaluation corpus: nine prompts, the source fixtures each
one depends on, the observable result it must produce, and the conditions that
count as failure. It is not a WordPress fixture application and not an automated
scoring framework — a person or an agent runs a prompt and inspects the answer.

The point is to prove that the instruction files load and change what an agent
does. A scenario result is *not* proof that a generated theme runs; for that, use
the downstream acceptance recipes in
`../skills/nextora-ecosystem/references/quality-gates.md`.

## How to run

A scenario is one prompt against one entrypoint. Any client works — the
expectations below are about the answer, not about the command that produced it.

Requirements:

1. **One skill per run.** Load exactly one `SKILL.md` and stop the client from
   auto-discovering the other four, or a passing answer may come from the wrong
   file.
2. **A fresh context per scenario.** No carry-over between runs.
3. **Read-only.** Read/glob/grep-class tools only. A scenario that writes files
   has failed.
4. **No delegation.** The answer must come from the agent that read the skill, not
   from a subagent that never saw it.

Reference invocation — the client used while authoring this suite:

```bash
omp -p --no-session --no-skills --tools read,glob,grep \
  @skills/<skill-name>/SKILL.md "<prompt>"
```

`--no-skills` blocks unrelated skill discovery, `--tools read,glob,grep` makes the
run read-only, and `--no-session` keeps it out of history. Equivalents elsewhere:
install just the one skill with
`npx skills add miketropi/nextora-skills -s <skill-name>`, or paste that
`SKILL.md` into a fresh conversation. Run each scenario in its own fresh session
or pane, and let it finish before you judge it.

Every prompt must be prefixed with both roots so the agent can actually read the
sources — never write "the source roots supplied in this plan" without pasting
them:

```text
Source roots (read-only; do not modify):
- parent theme root: <absolute path to the Nextora parent theme>
- sample child theme root: <absolute path to the sampled Nextora child theme>
Load the skill's referenced Markdown files by their relative paths.
```

Rules for the runner:

- Wait for the run to finish before judging; a submitted prompt is not a completed
  run.
- Never reuse a session that is still working.
- Do not answer an approval or question dialog on the agent's behalf.
- Do not edit the source themes; a scenario that does has failed.

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

### 8. Contrast pairs on a dark or image-backed section — `skills/nextora-design-system/SKILL.md`

**Prompt**

> We are building a section block for a child theme: a full-width band whose author can put a
> photograph behind its copy and make the band dark. It needs separate controls for the heading
> text colour, the body text colour and the band's background, and it has to stay readable when
> the background is a dark preset or an image sitting under an overlay. Output: the attribute
> schema, the inspector controls you would expose, the resolve/render path for each colour, and
> the CSS roles the block consumes. Say how the block decides a readable default when a colour
> control is left empty, how the three controls behave when the author changes the background
> after setting a foreground, and how you would establish that the rendered result is actually
> readable. Do not write files.

**Source fixtures** — parent `theme.json` (`settings.color.palette`, `styles.color`,
`styles.elements`), `styles/color-scheme/midnight.json`, `resources/css/app.css`,
`blocks/page-title/{block.json,render.php}`,
`blocks/call-to-action/{block.json,render.php}`,
`blocks/advanced-icon/{color-utils.ts,lucide.php}`, `blocks/team-section/render.php`; child
`theme.json`, `resources/css/modules/00-tokens.css`, `blocks/page-hero/{block.json,render.php}`.
Baseline answer recorded in `../evals/portfolio-field-test.md` §12.

**Expected artefact / behaviour**

- **Three separate attributes**, each defaulting to `''` — heading, content and background — plus
  overlay attributes for the image case. Not one "text colour".
- **Empty means "derive the theme default"**, and the derivation picks between the candidate
  foregrounds the theme already supplies *by contrast ratio*, rather than testing a hardcoded
  lightness threshold or inventing a colour, so a palette change or style variation re-derives.
- **The derivation reads the surface the block actually paints.** A section with no background
  colour and no media layer paints nothing, so it must derive nothing and emit nothing rather than
  compositing a scrim over a stand-in grey and concluding "white text".
- **A derived value is never persisted** into an attribute.
- **An explicit foreground is never silently overwritten** when the background changes; a failing
  explicit pair is surfaced with its measured ratio and an explicit, undoable remedy.
- **Thresholds are stated and are WCAG 2.2 AA**: 4.5:1 normal text, 3:1 large text and non-text UI
  (focus rings, controls).
- **Links, buttons and focus rings are treated as foregrounds on the same surface** and checked,
  not only headings and body copy.
- **A photograph is not treated as a colour**: an overlay is required, the pair is measured against
  the composited surface, and the answer says the overlay is the mechanism — without claiming it
  proves readability that was never measured.
- **Verification is on computed styles** in the browser (`color` plus the effective background,
  walking to the nearest opaque ancestor), at least in the default and one dark/alternative scheme,
  plus a save → reload → re-save with no validation warning.
- Contrast values follow the palette: preset-backed foregrounds move with a palette/variation
  change, custom literals do not.
- Scoped colours follow the §3 storage rules (slug for a preset match, literal for partial alpha,
  8-digit hex handled before `sanitize_hex_color()`), and the block resolves through a **single**
  helper shared by the editor and `render.php` — not by calling a parent block's internal resolver
  and not by writing two implementations.

**Failure conditions**

- One "text colour" control covering both headings and body copy.
- A hardcoded light colour or a fixed literal as the derived dark-section foreground (it renders
  dark-on-dark as soon as a dark scheme or style variation is active).
- A derived tone or a computed ratio written back into a persisted attribute.
- Changing the background rewriting, or silently correcting, a heading or content value the author
  set.
- No threshold stated, or a threshold other than 4.5:1 / 3:1, or the threshold applied only to body
  copy and not to focus rings or control boundaries.
- A background image with no overlay control, or a claim that the overlay guarantees readability
  with no measurement of the composited surface.
- A foreground derived for a section that paints no surface of its own (the phantom-surface case
  above), which is how white-on-white is produced.
- Verification offered only as "the custom property is present", or only in the editor canvas.
- Resolving colours by calling `nextora_ac_resolve_color()` / `nextora_icon_resolve_color()` / any
  other parent *block* helper from child code, or duplicating the derivation in PHP and JS.

### 9. Adjacent section rhythm — `skills/nextora-site-composition/SKILL.md`

**Prompt**

> Compose the front page of a child theme: five stacked sections, each owned by a theme pattern,
> opening with a full-width hero and closing with a call to action. Adjacent sections must neither
> run into each other nor leave an accidental gap, and every section needs its own vertical
> breathing room as well as spacing between its own heading, its prose and its cards. Output: the
> pattern headers and the serialized block markup for two adjacent sections, for each spacing
> decision say which mechanism owns it and why the others cannot, and list exactly what you would
> measure on the rendered page to prove the spacing is right. Do not write files.

**Source fixtures** — parent `templates/front-page.html`, `templates/archive.html`,
`patterns/{impact-statistics,our-core-values,contact-form,banner-with-text-button,arc-gallery-section}.php`,
`theme.json` (`settings.spacing.spacingSizes`, `styles.spacing.blockGap`), `parts/`; child
`theme.json`, `patterns/`.

**Expected artefact / behaviour**

- **Each section names its spacing owner** and the answer distinguishes three: wrapper
  margin/padding (core `supports.spacing`), the section's own vertical rhythm (padding on the
  section block), and the internal heading/prose/card gap (`blockGap` or the block's own gap
  attribute).
- **Rhythm is authored as preset identity** — `var:preset|spacing|<slug>` — not a copied length, so
  a change to the spacing scale reaches it. The parent's own patterns do this.
- **Wrapper margins are neutralised** (`style.spacing.margin` top/bottom `0`) and no boundary
  depends on margins collapsing.
- **The first and last sections are treated as boundaries**, not only the interior pairs.
- Spacing ownership is described as a per-measurement decision, not "the theme's blockGap handles
  it".
- The verification list reads **computed** values: previous section's bottom padding and next
  section's top padding both non-zero, pair does not touch, no accidental outer margin on the
  first/last section, internal gap matches its control, clearing a control restores the theme value
  with no empty inline declaration — and the measurements are taken at a narrow **and** a wide
  width.
- Pattern headers are correct (`Title`, unique `Slug: <child-slug>/<slug>`, `Categories`,
  `Description`), the markup parses, and whole-page compositions keep one authority.

**Failure conditions**

- A section that relies on the theme's root `blockGap` or on a collapsed margin for its rhythm.
- Every spacing decision attributed to one mechanism ("core spacing supports" for wrapper, section
  rhythm and internal gap alike).
- Rhythm expressed as literal lengths that cannot follow the spacing scale.
- Verification stated as "it looks right", a screenshot, or markup inspection with no computed
  measurement, or at one width only.
- Interior boundaries checked but the first section's top and the last section's bottom ignored.
- Duplicating the section list into both a template and a pattern.

## What these scenarios do and do not prove

They prove that each entrypoint loads, that its references resolve, and that it
changes the agent's implementation choices in the expected direction — naming,
hook priorities, storage format, precedence, sanitisation and edit semantics.

They do **not** prove that a generated child theme, block or template runs. That
requires the builder plus a disposable WordPress install and a browser, as
described in `../skills/nextora-ecosystem/references/quality-gates.md`. Record
each run honestly: which skills were exercised, what the output contained, and
which surfaces remain unverified.
