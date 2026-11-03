# Evaluation scenarios

An instruction-suite evaluation corpus: eleven prompts, the source fixtures each
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
   auto-discovering the other skills, or a passing answer may come from the wrong
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

Every prompt must be prefixed with the roots it needs so the agent can actually
read the sources — never write "the source roots supplied in this plan" without
pasting them:

```text
Source roots (read-only; do not modify):
- parent theme root: <absolute path to the Nextora parent theme>
- sample child theme root: <absolute path to the sampled Nextora child theme>
- source template root: <absolute path to the template being ported>
Load the skill's referenced Markdown files by their relative paths.
```

The `source template root` line applies to the template-porting scenarios (10 and
11) and may be omitted for the rest; the two theme roots apply to every scenario.
A template-porting scenario still runs read-only — it inspects the source
template, it never edits or copies it.

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

### 10. Messy source to normalized design system — `skills/nextora-template-porting/SKILL.md`

**Prompt**

> Port the source template described by the dossier appended to this prompt into a
> Nextora child theme. Read the parent theme at the root above for its real preset
> slugs and `--nextora-font-*` roles. Output the fixed conversion dossier the skill
> defines — all nine headings, in order — plus the phase 0–6 roadmap with
> prerequisites, owner skill and an observable gate per phase. Every claim must be
> traceable to something in the dossier or the theme roots, or labelled unverified.
> Do not write files.

**Embedded source dossier (part of the prompt)**

The source is an unminified, half-migrated marketing site. There is no repository
history, no design documentation and no component library; `theme.css` and
`index.html` are the only authored files. Two screenshots exist, at 1440px and
390px, for the home hero only.

```css
/* theme.css — appended to and partially overridden during a redesign */
:root{
  --brand:#0b5cab; --brand-alt:#0a5cab; --main-color:#0B5CAB; --c-primary:rgb(11,92,171);
  --accent:#f2a104; --accent-2:#f3a205;
  --bg:#ffffff; --bg-soft:#fdfdfe;
  --text:#1c1f23; --text-soft:#1d2024;
  --gray-1:#f6f7f8; --gray-2:#f5f7f8;
  --radius:6px; --radius-lg:7px;
  --space-sm:13px; --space-md:26px; --space-lg:52px; --gap:25px;
  --speed:180ms;
}
.site-header{background:var(--bg-soft);padding:18px 24px}
.nav a{font-family:'Inter',system-ui,sans-serif;font-size:15px;color:var(--text)}
.hero{background:#0b5cab;color:#fff;padding:clamp(48px,7vw,104px) 24px;border-radius:6px}
.hero h1{font-family:'"Playfair Display"',Georgia,serif;font-size:clamp(2rem,5vw,3.4rem);line-height:1.08;font-weight:700}
.hero p{font-family:'Inter',system-ui,sans-serif;font-size:17px;line-height:1.6;color:rgba(255,255,255,.82)}
.hero .cta{background:#f2a104;color:#1c1f23;border-radius:6px;padding:12px 22px}
.hero__scrim{background:linear-gradient(180deg,rgba(11,92,171,0) 0%,rgba(11,92,171,.86) 100%)}
.card{border:1px solid #e6e8ea;border-radius:7px;box-shadow:0 1px 2px rgba(16,24,40,.06);background:#fdfdfe}
.card__title{font-family:'"Playfair Display"',Georgia,serif;font-size:22px;line-height:1.25}
.eyebrow{font:600 12px/1 'Inter';letter-spacing:.14em;text-transform:uppercase;color:#0b5cab}
.grid{display:grid;gap:25px;grid-template-columns:repeat(3,1fr)}
@media (max-width:960px){.grid{grid-template-columns:1fr}.hero{padding:40px 20px}.card__title{font-size:20px}}
/* carried over from the previous build; some rules dead */
.u-mt-13{margin-top:13px}.u-mt-sm{margin-top:12px}.flex-1{flex:1 1 0}.text-center{text-align:center}
/* the legacy minified bundle.css this override was appended after still declares --brand as #0a5cab */
@font-face{font-family:'Playfair Display';src:url('/demo/fonts/playfair.woff2') format('woff2')}
Header. Body copy uses the same 'Inter' stack the page also loads from a Google Fonts <link>.
```

```html
<!-- index.html (excerpt) -->
<header class="site-header js-header u-mt-13">
  <a class="brand" href="/">Acme Robotics</a>
  <nav class="nav"><a href="/platform">Platform</a><a href="/pricing">Pricing</a></nav>
</header>
<section class="hero js-hero" data-parallax="true">
  <h1 class="hero__title">Build faster with Acme</h1>
  <p>Ship production robotics workflows in days, not quarters.</p>
  <a class="cta" href="/signup">Start free</a>
</section>
<section class="features">
  <p class="eyebrow">Why Acme</p>
  <div class="grid">
    <article class="card"><h3 class="card__title">Fast</h3><p>…</p></article>
  </div>
</section>
```

Nothing else about the source is available: no license file for the `playfair.woff2`
font, no logo source, and the 390px screenshot is the only evidence of the mobile
breakpoint.

**Source fixtures** — parent theme `theme.json` (palette, font-family, font-size,
spacing slugs and `styles`), `resources/css/app.css` (`--nextora-font-body` /
`--nextora-font-heading` / `--nextora-font-button`), `styles/color-scheme/*.json`
and `styles/font-preset/*.json`; sample child `theme.json` and
`resources/css/modules/00-tokens.css`. The source dossier is embedded in the prompt;
no fixture application is added.

**Expected artefact / behaviour**

- All nine dossier headings, in order, and the phase 0–6 roadmap with prerequisites,
  owner skill and an observable gate per phase.
- Evidence and confidence separates what was read (the CSS/HTML excerpt, the two
  screenshots) from inference, and marks as unverified: DOM semantics not shown,
  the `--brand` conflict, the font's licence and provenance, the logo, and anything
  below the single captured mobile breakpoint.
- Token evidence table with one row per candidate, and **role-based clustering**:
  `#0b5cab`, `#0a5cab`, `#0B5CAB`, `rgb(11,92,171)` collapse to one brand role with
  the conflict recorded; `13px`/`12px` and `26px`/`25px` collapse into one spacing
  scale with exceptions named; `6px`/`7px` collapse into one radius role.
- The normalization maps roles onto the parent's **existing preset identities** and
  the `--nextora-font-*` roles — a source value is preserved only where it serves a
  stable role, and additions get child-owned semantic slugs rather than a parallel
  variable layer.
- Utility classes (`u-mt-13`, `.flex-1`, `.text-center`), the `js-*` hooks and the
  `data-parallax` attribute are classified as normalization noise, not as things to
  reproduce; the duplicated legacy `--brand` override and the dead utility rules are
  called out.
- The `rgba(255,255,255,.82)` body copy on the brand band and the
  `linear-gradient` scrim are treated as a **foreground/background pair** that needs
  a measured contrast result, handed to `nextora-design-system` — not asserted as
  readable.
- The `'Playfair Display'` `@font-face` from `/demo/fonts/` and the Google Fonts
  `<link>` are both **blocked**: unknown licence and provenance is not permission,
  and the decision is recorded in `Unverified and blocked decisions` with the input
  that would unblock it.
- Spacing and radius decisions are handed to `nextora-design-system`; the report
  states that pixel-parity is not an acceptance criterion.

**Failure conditions**

- A token table that copies literals (hexes, `13px`, `6px`/`7px`) into the child as
  a parallel variable set, or that clusters by numerical similarity and merges
  values that serve different roles.
- Renaming or re-pointing an inherited parent preset slug, or copying a palette.
- Claiming the brand band, the scrim or the font are fine without evidence; a
  readability claim with no measurement.
- Treating the `/demo/fonts/` font or the Google Fonts link as licensed to ship.
- Reproducing source class names, `js-*` hooks or utility classes as the port's
  markup convention, or proposing a pixel-perfect clone.
- Writing files, or claiming a runtime result.

### 11. Source sections to a conversion roadmap — `skills/nextora-template-porting/SKILL.md`

**Prompt**

> Plan the port of the source template described by the dossier appended to this
> prompt into a Nextora child theme, using the parent and sample child at the roots
> above as the capability inventory. Output the fixed conversion dossier — all nine
> headings, in order — with every supplied page, section, interaction and asset
> classified, plus the phase 0–6 roadmap with prerequisites, owner skill and an
> observable gate per phase. Do not write files.

**Embedded source dossier (part of the prompt)**

A 12-page marketing site plus a small journal. Authored source: `*.html` pages, one
`app.js` framework bundle, `styles/` and `assets/img/`. No component library and no
build config.

```
pages:  index.html, pricing.html, about.html, contact.html, platform.html,
        features.html, customers.html, careers.html, legal/privacy.html,
        legal/terms.html, journal/index.html, journal/post.html
shell:  the same <header> and <footer> markup is copy-pasted into all 12 pages;
        the footer differs on 3 pages (an extra newsletter form block)
shared sections repeated across pages:
  - "logo strip" (customers.html, index.html, platform.html, features.html)
  - "three-up feature cards" (index.html x1, features.html x2, platform.html x1)
  - "testimonial carousel" (index.html, customers.html)
  - "pricing table" (pricing.html, features.html)
  - "CTA band" (10 of 12 pages)
editorial:
  - journal/index.html -- a filterable post list (category chips re-render the list
    client-side from an inlined JSON array; no server filtering)
  - journal/post.html -- long-form article body, share buttons, related-posts strip
forms:   contact.html -- name/email/message form POSTing to /api/contact (the
         endpoint is a vendor script with a demo key); a newsletter form in 2 footers
interactions: mobile nav (hamburger + focus trap), testimonial carousel (autoplay,
         arrows, dots), FAQ accordion (about.html), category filter (journal),
         tabbed feature panel (platform.html), scroll-reveal animations on every section
assets:  assets/img/hero-*.jpg (remote demo URLs on 4 pages), a logo .svg, a
         sprite of 20 icons, a decorative animated Lottie JSON, 3 demo MP4 loops
runtime: app.js (a jQuery-era framework bundle with its own carousel/accordion/
         tabs plugins and a GSAP-like scroll-reveal), plus a Google Analytics tag
         and a chat-widget script in the footer of every page
licences: none shipped for the images, the Lottie file or the fonts
```

**Source fixtures** — parent `blocks/*/block.json` (including `nextora/contact-form`
and the section blocks), `patterns/*.php`, `templates/`, `parts/`, `theme.json`
(`customTemplates`, `templateParts`); sample child `patterns/*.php`, `templates/`,
`parts/`, `blocks/*/block.json`. The source dossier is embedded in the prompt; no
fixture application is added.

**Expected artefact / behaviour**

- All nine dossier headings, in order, and a roadmap with exactly phases 0–6, each
  carrying prerequisites, the matrix rows/surfaces it consumes, the owner skill, an
  observable checkpoint and a rollback boundary.
- A page/route inventory for all 12 pages, and a **section matrix** in which every
  supplied section has a row with source anchors, retained content, dependencies,
  repeated/reusable status, target surface, closest existing candidate, decision,
  editor ownership, dynamic-data needs, responsive behaviour, asset needs and
  handoff skill.
- The duplicated header/footer markup is one shared shell decision (reuse or one
  child-owned part) — not 12 copies of the same markup in the target.
- The **fixed decision ladder** is applied in order and stated: existing pattern →
  existing parent/child/core block → new child pattern → thin template → part
  override → extend a block → new block only for a behaviour/editing gap.
- Core/existing surfaces win where they fit: the contact form is
  `nextora/contact-form` rather than the vendor endpoint (`replace`); the FAQ
  accordion, tabs, mobile nav and lightbox-style behaviours are `core behavior` or
  existing-block behaviour rather than ports of `app.js`; the pricing table, feature
  cards, logo strip and CTA band become patterns composed from existing blocks.
- A **custom block** is proposed only for the one row whose editing model or data
  shape nothing existing expresses (for example the client-side filterable journal
  list), and the reason is written down.
- The **journal index** page model is chosen correctly: editor-owned narrative and
  order (`core/post-content` + an insert-once starter pattern) rather than a
  theme-owned fixed pattern, because its owner edits posts and copy.
- Full-page surfaces name exactly one ownership model each: hidden pattern + thin
  template for the fixed marketing pages, `core/post-content` for the editorial
  surfaces.
- The scroll-reveal animations and the framework bundle are not preserved because
  they exist; the interaction matrix records a reduced-motion-respecting decision
  or `omit`.
- Asset ledger rows for the remote demo images, the logo, the icon sprite, the
  Lottie file, the demo MP4 loops and the untracked fonts, with dispositions —
  remote demo URLs are excluded from production assets, and unknown licences are
  blocked reuse decisions.
- Third-party tracking (`analytics`, chat widget) is `exclude`/`replace`, never a
  theme dependency; editorial media is routed to the media library, not the theme.
- Handoffs dispatch to all applicable specialists: `nextora-child-themes` for
  bootstrap/assets, `nextora-design-system` for tokens, contrast pairs and spacing
  ownership, `nextora-site-composition` for templates, patterns and hierarchy, and
  `nextora-gutenberg-blocks` only for the custom-block row.

**Failure conditions**

- Writing files, editing either source theme or the source template.
- A generic checklist with no section-level decision, or a section matrix missing
  supplied sections, interactions or assets.
- Proposing custom blocks for sections that existing patterns or blocks already
  express, or skipping the ladder and jumping to "new block".
- Cloning the source's header/footer markup into many templates, or keeping the
  framework/`app.js` because its classes or plugins exist.
- The wrong page model for the editorial surfaces (a fixed theme pattern instead of
  `core/post-content`, or vice versa for the fixed marketing pages).
- Retaining the vendor/demo form endpoint, the tracking scripts or the remote demo
  media as production theme assets, or treating an unknown licence as permission.
- Inventing evidence not present in the dossier or the theme roots, or a roadmap
  that omits phases, gates, owners or rollback boundaries.

## What these scenarios do and do not prove

They prove that each entrypoint loads, that its references resolve, and that it
changes the agent's implementation choices in the expected direction — naming,
hook priorities, storage format, precedence, sanitisation and edit semantics.

They do **not** prove that a generated child theme, block or template runs. That
requires the builder plus a disposable WordPress install and a browser, as
described in `../skills/nextora-ecosystem/references/quality-gates.md`. Record
each run honestly: which skills were exercised, what the output contained, and
which surfaces remain unverified.
