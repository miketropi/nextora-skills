---
name: nextora-site-composition
description: Compose the surfaces of a Nextora or Nextora child block theme — HTML templates and template parts, PHP block patterns and their categories, page templates and the template hierarchy, front-page versus posts-index, filesystem patterns referenced from thin templates, core query/archive/search loops and explicit query context, header and footer ownership, and filesystem-versus-Site-Editor customisation. Use when a request asks for a landing page layout, a page template, an archive or search composition, a reusable pattern or section, a header/footer part, a query loop, or asks why a template file edit shows no change on the front end.
---

# Nextora site composition

Owns the composition layer of a Nextora-sourced block theme: `templates/`,
`parts/`, `patterns/` and the `theme.json` declarations that describe them. Block
internals are `../nextora-gutenberg-blocks/SKILL.md`; appearance and preset data
are `../nextora-design-system/SKILL.md`; child bootstrap and assets are
`../nextora-child-themes/SKILL.md`. When a page or section is ported from an
external source template, `../nextora-template-porting/SKILL.md` supplies the
section conversion matrix, its reuse classification and the page ownership model
this skill implements against; it supplies analysis and planning only and never
overrides the composition contract below.

## References to load

- [`references/templates-and-patterns.md`](references/templates-and-patterns.md) —
  the template hierarchy with the real file names, part ownership and override
  rules, the pattern header and category conventions, the hidden-pattern-plus-thin
  template recipe, query composition and the sanitising helpers,
  filesystem-versus-database customisation, the observed defects, and the
  downstream WordPress acceptance recipe. **Load it before step 3.**
- [`source-map.md`](../nextora-ecosystem/references/source-map.md) — provenance
  for every anchor named there.
- [`quality-gates.md`](../nextora-ecosystem/references/quality-gates.md) — the
  gates that decide whether the composition is finished.

Local files outrank every value quoted in these references.

## Ordered workflow

### 1. Resolve the theme roots and read the local surfaces

The parent is the template directory, the active theme is the stylesheet
directory:

```php
get_template_directory();      // parent root
get_stylesheet_directory();    // active theme root (child when one is active)
```

Read, in this order: `theme.json` (`customTemplates`, `templateParts`),
`templates/`, `parts/`, `patterns/`, the target's `blocks/*/block.json` names,
and `inc/` for query or template helpers. A child's copy of a template or part
overrides the parent's file of the same name, so what already exists decides most
of the answer.

### 2. Find what already exists before composing anything

Reuse beats invention, in this order:

1. An existing pattern that already is the section.
2. An existing block (parent `nextora/*` or the child's own) that already renders
   the section.
3. A new child pattern composed from existing blocks.
4. A thin template that assembles those patterns.
5. A child part override — only for a surface the child genuinely owns.
6. A new block — only when the requested behaviour or editing structure cannot be
   expressed by any existing block.

Never ship mock posts as production content, and never clone the parent's contact
form backend: the parent already ships the `nextora/contact-form` block, and the
sample's contact page composes it inside a child wrapper group.

### 3. Place the composition in the right template

Apply the hierarchy from `references/templates-and-patterns.md`:

- `front-page.html` governs the front page **when present**; `home.html` is the
  posts index, not a synonym for the front page; `index.html` is the fallback.
- Archive, single, search and page-specific templates precede the index fallback.
- A name declared in `theme.json` `customTemplates` must have a matching
  `templates/<name>.html` in the theme that declares it.

Read the theme's own `templates/` listing rather than guessing file names: the
parent ships `front-page`, `home`, `index`, `archive`, `single`, `search`, `page`,
`page-full-width`, `about-us`, `contact-us`, `frequently-asked-questions`,
`theme-terms-conditions`, `theme-privacy-policy`, `404`; the sample child ships
its own, differently named, set.

### 4. Decide part ownership, then reference the part

A `wp:template-part` reference resolves against the **stylesheet** directory
first, so a child `parts/header.html` replaces the parent's header everywhere it
is used.

- Keep the parent header/footer if the child has no reason to own them: do not
  add a copied part.
- If the child owns a part, give it a child-owned body and say so in the
  reference; do not leave a `theme` attribute that pins the parent's copy.
- The sample child's `parts/header.html` and `parts/footer.html` are one block
  reference each (`nextsaas/site-header`, `nextsaas/site-footer`); the parent's
  header part composes `nextora/scrolling-promotion` and `nextora/header`.
- The reference syntax in the sources is `{"slug":"header","tagName":"div"}` in
  the child and `{"slug":"header","tagName":"header","area":"header"}` in the
  parent. Match the target's own convention; never hardcode a filesystem path.

### 5. Author patterns as PHP files with a real header

Patterns live in `patterns/*.php`. Their registration comes from the file header,
not from a `register_block_pattern()` call — neither source theme calls it; both
register only *categories* in PHP. Required header fields and slug rules are in
`references/templates-and-patterns.md`:

- `Title`, `Slug: <child-slug>/<pattern-slug>` (unique), `Categories`, and a
  `Description` that says when to use it.
- **A section pattern owns its own vertical rhythm, and says which mechanism provides it.** The
  live parent's own patterns do this — `patterns/impact-statistics.php`,
  `patterns/our-core-values.php`, `patterns/contact-form.php` and
  `patterns/banner-with-text-button.php` all set `style.spacing.margin` `top`/`bottom` to `0` and
  own the rhythm with `style.spacing.padding`, stored as `var:preset|spacing|<slug>` — but no suite
  file states the requirement, so a new pattern can omit it and look acceptable in isolation while
  every boundary changes when the sections are stacked. Give each section deliberate top and
  bottom padding, neutralise the wrapper margins, and never rely on margins collapsing between two
  sections to create or remove a gap. Ownership table:
  `../nextora-design-system/references/presets-and-switching.md` §3.2.
- The pattern body is the serialized block markup the editor inserts, starting
  after the closing `*/` of the header comment.
- It must parse, use only registered blocks, and stay editable after insertion.

### 6. Give a full-page composition exactly one authority

For a new full-page composition, put the section list in a hidden filesystem
pattern — `Inserter: no` in the header — and reference it from a thin template:

```html
<!-- wp:template-part {"slug":"header","tagName":"div"} /-->

<!-- wp:pattern {"slug":"nextfolio/page-landing"} /-->

<!-- wp:template-part {"slug":"footer","tagName":"div"} /-->
```

Do not repeat the section list into the template as well. The sample currently
holds the same list twice (`patterns/page-home.php` and
`templates/front-page.html`), and the parent's templates inline pattern markup
carrying `metadata.patternName`; that duplication is what this rule replaces for
new work. When merely editing an existing block, keep the existing duplication —
update the other copies only if the section content actually changed.

### 7. Compose queries with core blocks and explicit context

- Archives, the posts index, search and post detail delegate to `wp:query` with
  `"inherit":true`, so the main query provides the context; the sample's
  `index.html`, `home.html`, `archive.html` and `search.html` all do this.
- Build the loop from `core/post-template` plus `core/post-*` blocks, add
  `wp:query-pagination`, and always provide `wp:query-no-results`.
- Use `wp:query-title`, `core/post-title`, `core/post-content`,
  `core/comments` + `core/comment-template` + `wp:comments-pagination` +
  `wp:post-comments-form` for the single-post surface rather than a classic loop.
- A block that starts its **own** loop passes sanitised attributes: reuse the
  child's `inc/blocks/query.php` helpers where they exist, otherwise clamp page
  sizes and validate post types, taxonomies and terms against allowed values.
  Prepared SQL is not a substitute for that validation.

### 8. Keep `theme.json` declarations accurate

Declare `customTemplates` and `templateParts` entries only for surfaces the theme
owns, and only for names that are not core templates. See
`references/templates-and-patterns.md` for the observed re-declaration in the
sample and the rule for new work.

### 9. Account for the Site Editor

Filesystem templates and parts can be masked by database customisations saved in
the Site Editor. If a file edit does not change the front end, look for a saved
template or style before touching the file. Never silently overwrite a real
site's customisations; inspect or reset only a disposable environment.

### 10. Verify against the running site

Resolve each template and part in the front end, confirm patterns insert and stay
editable, confirm the requested WordPress minimum and the current supported
release, and run the browser recipe in `references/templates-and-patterns.md`.
Check the composition at its **boundaries**, not only per section: for every adjacent pair read
the previous section's computed bottom padding and the next section's computed top padding,
confirm both are non-zero and match what was authored, confirm the first and last sections gain no
accidental outer space, and confirm no pair touches. Repeat at a narrow and a wide width, because a
preset that collapses at one width is a rhythm defect the per-section view cannot show.
Mark anything you did not exercise as unverified.

## Completion checks

- Every template and part used resolves to the intended theme; no stray `theme`
  attribute pins a parent part from a child template.
- No unnecessary new block, no classic PHP page template, no mock production
  content.
- New full-page composition: one authority in a hidden pattern, referenced from a
  thin template; no duplicated section list.
- Patterns parse, use registered blocks, keep unique slugs, and their categories
  are the ones the theme registers in PHP.
- Query loops use explicit context, sanitised attributes and a no-results state.
- Every section names its own spacing owner, none depends on margins collapsing, and each measured
  boundary (first, adjacent, last) was checked on computed styles at both a narrow and a wide
  width.
- `theme.json` declares only owned, non-core template names.
- [`quality-gates.md`](../nextora-ecosystem/references/quality-gates.md) applied,
  including its composition acceptance section.

## Boundaries

Markdown instruction only: no runtime scripts, generators, copied theme trees or
installers, and no edits to the parent or the sample child. The parent's pattern
library and the sample's compositions are evidence to read, not files to fork.
