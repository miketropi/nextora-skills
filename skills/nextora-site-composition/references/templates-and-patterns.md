# Templates and patterns

The composition contract behind `../SKILL.md`. Provenance tags match the shared
map: **[parent contract]**, **[sample convention]**, **[new-output requirement]**,
**[observed defect]**.

Shared sources: [`source-map.md`](../../nextora-ecosystem/references/source-map.md),
[`quality-gates.md`](../../nextora-ecosystem/references/quality-gates.md).
Reread the live `templates/`, `parts/`, `patterns/`, `theme.json` and `blocks/*/block.json`
before editing; every file list below is an anchor.

## 1. What each surface is

| Surface | Path | Resolves against |
| --- | --- | --- |
| Template | `templates/<name>.html` | stylesheet directory first, then template directory |
| Template part | `parts/<name>.html` | same |
| Pattern | `patterns/<slug>.php` | the theme that registers it; a child can be selected by `theme` on the reference |
| Declaration | `theme.json` `customTemplates` / `templateParts` | the declaring theme, merged child over parent |
| Query helper | `inc/blocks/query.php` (child) | the active theme |

A child's file with the same relative name replaces the parent's — that is how
`nextsaas/parts/header.html` (one line: `<!-- wp:nextsaas/site-header /-->`)
replaces the parent's header part everywhere it is referenced. **[sample
convention]**

## 2. Template hierarchy, with the real names

The active theme is consulted first; a file the child does not ship falls through
to the parent.

| Request | First match | Fallback |
| --- | --- | --- |
| Front page | `front-page.html` | `home.html`, then `index.html` |
| Posts index (page for posts) | `home.html` | `index.html` |
| Category / tag / taxonomy | `archive.html` — the shipped generic archive | `index.html` |
| Single post | `single.html` (and `single-<post_type>.html`) | `index.html` |
| Search | `search.html` | `index.html` |
| Page | a page-specific template, then `page.html` | `index.html` |
| 404 | `404.html` | `index.html` |
| Custom template | `templates/<name>.html` for a `<name>` declared in `customTemplates` | the hierarchy above |

Shipped names, as read:

- Parent `templates/`: `404`, `about-us`, `archive`, `contact-us`,
  `frequently-asked-questions`, `front-page`, `home`, `index`, `page`,
  `page-full-width`, `search`, `single`, `theme-privacy-policy`,
  `theme-terms-conditions`. **[parent contract]**
- Sample child `templates/`: `404`, `archive`, `front-page`, `home`, `index`,
  `page`, `page-about-us`, `page-contact-us`, `page-faq`, `page-pricing`,
  `page-product`, `search`, `single`. **[sample convention]**

`home.html` is the posts index, not a synonym for the front page: the sample's
`home.html` header comment says it holds the same block list as `archive.html`
because both render the journal surface, and it is reached *after*
`front-page.html` for the site front page. **[sample convention]**

`index.html` is the required fallback; a theme without one is incomplete.

## 3. Template parts and ownership

Reference syntax as it appears in the sources:

```html
<!-- child:  --> <!-- wp:template-part {"slug":"header","tagName":"div"} /-->
<!-- parent: --> <!-- wp:template-part {"slug":"header","tagName":"header","area":"header"} /-->
<!-- child footer: --> <!-- wp:template-part {"slug":"footer","tagName":"div"} /-->
```

Rules:

- `slug` is the file name in `parts/`, without `.html`. **[parent contract]**
- Because resolution goes to the stylesheet directory first, a child part is a
  real override for every template that references that slug — including the
  parent's templates. Override a part only when the child genuinely owns it.
- Do not write a `theme` attribute to pin the parent's copy. A reference inside a
  child template that was copied from the parent and left pointing at the parent
  is a defect: the child's own part is bypassed and later edits diverge.
- `area` (and `tagName`) are optional metadata; the parent uses `area: "header"`
  / `"footer"`, the child omits it. Match the target theme's own convention rather
  than mixing both.
- Parent `parts/`: `header`, `footer`, `footer-newsletter`, all declared in
  `theme.json` `templateParts`. The parent header part composes
  `nextora/scrolling-promotion` and `nextora/header`; the parent footer part is
  plain group markup. **[parent contract]**
- The sample child's `parts/header.html` and `parts/footer.html` are each a single
  block reference. **[sample convention]**

## 4. Patterns

### 4.1 Registration comes from the file header

Neither source theme calls `register_block_pattern()`. Patterns in `patterns/`
are registered by WordPress from the PHP file header; the themes register only
*pattern categories* in PHP:
`inc/setup/pattern-categories.php` in the parent and
`nextsaas_register_pattern_categories()` in the child's `blocks/blocks.php`.
**[parent contract + sample convention]**

### 4.2 Header fields, as authored

Section pattern (sample):

```php
<?php
/**
 * Title: Built with intention
 * Slug: nextsaas/about-us-built-with-intention
 * Categories: nextsaas-sections
 * Description: Concept statement and related links
 * Keywords: about, editorial, section
 * Viewport Width: 1440
 */

?>
<!-- wp:nextsaas/about-us-built-with-intention /-->
```

Hidden full-page pattern (sample):

```php
<?php
/**
 * Title: NextSaaS — Contact us
 * Slug: nextsaas/page-contact-us
 * Categories: nextsaas-pages
 * Description: Contact us, composed in design order. Hidden from the inserter; use the template.
 * Keywords: contact-us
 * Inserter: no
 * Viewport Width: 1440
 */

?>
<!-- wp:nextsaas/page-hero {…} /-->
<!-- wp:nextsaas/contact-us-contact -->
<!-- wp:nextora/contact-form {"showHeading":false,"showDescription":false,"enablePhoneField":false,…} /-->
<!-- /wp:nextsaas/contact-us-contact -->
```

Requirements for new patterns:

- `Slug: <child-slug>/<pattern-slug>`, unique across the theme; the parent uses
  `nextora/<name>`. **[parent contract]**
- `Title`, `Description` ("when to use it"), `Categories`, optionally `Keywords`
  and `Viewport Width`.
- `Inserter: no` for a full-page composition that a template references; leave it
  out of a section pattern meant to be inserted.
- The body after the header is the serialized block markup. It must parse, use
  only registered blocks (`nextora/*`, the child's own `<child-slug>/*`, and core
  blocks), and stay editable after insertion.
- **Declare the section's vertical rhythm in the pattern, and say which mechanism owns it.** A
  section pattern is where the rhythm is authored, so an omission here is invisible until the
  sections are stacked. The convention the live parent already follows is: neutralise the wrapper
  margins (`style.spacing.margin` `top` and `bottom` of `0`) and own the rhythm with
  `style.spacing.padding`, stored as preset identity (`var:preset|spacing|<slug>`) so a change to
  the spacing scale reaches it — see `patterns/impact-statistics.php`,
  `patterns/our-core-values.php`, `patterns/contact-form.php` and `templates/front-page.html` in
  the parent. **[parent contract, by observation]**
- **Never build a rhythm out of collapsing margins.** Two adjacent sections whose spacing comes
  from margins have a boundary that cannot be seen in the inspector, cannot be measured per
  section, and disappears entirely inside a flex or grid parent. Padding on both sides is the rule;
  the boundary is then two non-zero measurements, which is what the acceptance step below reads.

### 4.3 Pattern categories

Both themes register categories in PHP and reference them from pattern headers:

- Parent: `nextora-page-title` ("Page Title"), `nextora-core-values`,
  `nextora-newsletter`, plus core/other categories used by headers such as
  `featured`, `team`, `testimonials`, `contact`, `gallery`, `about`, `banner`,
  `cta-ready-to-grow`. **[parent contract]**
- Sample child: `nextsaas-sections` ("NextSaaS Sections") and `nextsaas-pages`
  ("NextSaaS Pages"). **[sample convention]**

A child registers its own categories before its patterns reference them, guarding
with `function_exists( 'register_block_pattern_category' )`. Reuse a parent
category only when the pattern genuinely belongs to it; a new category slug must
be prefixed with the child slug.

## 5. Full-page compositions: one authority

**[new-output requirement]** A new full-page composition has exactly one
authority: a hidden filesystem pattern, referenced from a thin template.

```html
<!-- templates/page-landing.html -->
<!-- wp:template-part {"slug":"header","tagName":"div"} /-->

<!-- wp:pattern {"slug":"nextfolio/page-landing"} /-->

<!-- wp:template-part {"slug":"footer","tagName":"div"} /-->
```

```php
<?php
/**
 * Title: Landing page
 * Slug: nextfolio/page-landing
 * Categories: nextfolio-pages
 * Description: Whole landing surface, composed in design order. Hidden from the inserter; use the template.
 * Inserter: no
 * Viewport Width: 1440
 */

?>
<!-- wp:nextfolio/hero /-->
<!-- wp:nextfolio/features /-->
<!-- wp:nextfolio/closing-cta {"heading":"…"} /-->
```

`wp:pattern` resolves the slug against the pattern registry at render time instead
of storing a copy of the markup, so editing the pattern file is the single edit.
The slug must match the pattern's `Slug:` header exactly.

**A thin template still has to provide the page's landmarks.** Moving the section list into a hidden
pattern removes the markup from the template — including, if you are not careful, the `<main>`
landmark, because a pattern of sections is not a page. Wrap the reference in the template:

```html
<!-- wp:group {"tagName":"main","anchor":"content","className":"<child-slug>-scope","layout":{"type":"default"}} -->
<!-- wp:pattern {"slug":"nextfolio/page-landing"} /-->
<!-- /wp:group -->
```

Keep the landmark in the **template**, not in the pattern: the template owns page chrome and the
pattern owns the section list, so a second template that ever references the same pattern does not
inherit a duplicate `<main>`. Then assert it — exactly one `<main>` and exactly one `<h1>` on every
route — because a theme whose ordinary templates carry the landmark while its front page does not is
the failure this rule exists to prevent. **[new-output requirement, verified 2026-11-03]**

### 5.1 The second whole-page model — editable page content

A whole-page composition serves one of two needs, and they are different models,
not two spellings of the same one:

| The layout is… | Model |
| --- | --- |
| owned by the theme; the owner edits structured fields inside it | hidden pattern referenced from a thin template with `wp:pattern` |
| the owner's own copy — narrative text, section order, sections they add | thin template rendering `core/post-content`, plus a hidden **starter** pattern (`Inserter: no`) inserted once into the page by the demo setup |

In the second model the page content is the single authority *after* insertion:
the starter pattern is insert-time material, not a live reference, so it is not a
second copy that can drift. The trade-off is the mirror image of §5 — a later edit
to the pattern file no longer changes an existing page.

Rules for it: do not seed the same copy from both a template and the page; do not
keep a duplicate hardcoded full-page file beside it; do not hide a page's editable
content behind a template that renders literal markup; and state the chosen model
in the theme's own `README`, because it decides who can edit what. A page's own
content is empty until the starter is inserted, so the setup step and the
substitutions it needs (a parent ID in a query, for example) belong in that
README too.

When setup code inserts a pattern's content itself, read it from the registry and
**slash it before storing**: `wp_insert_post( array( 'post_content' =>
wp_slash( $pattern_content ) ) )`. Core registers a pattern through
`serialize_block_attributes()` (`wp-includes/blocks.php`), which encodes `&` as
`\u0026`, and `wp_insert_post()` unslashes its input — so an un-slashed write
stores the literal characters `u0026` in the page and the defect only shows up on
the rendered front end. The same applies to any later content patch (a regex
replacement over stored markup included). Verify by reading the stored content
back and rendering the page, not by assuming the pattern file's own text survives
the round trip.

Choose the model from what the brief says the *owner* must be able to change. A
brief that asks for page content a non-developer edits is asking for the second
model, and no pattern-reference recipe satisfies it.

### 5.2 Section boundaries are part of the composition

A page composed of sections has a rhythm that belongs to the *sequence*, not to any one section,
and this suite previously said nothing about it. Three rules make it checkable:

1. **Every section names its spacing owner.** Wrapper margin/padding is core block supports; a
   section's vertical rhythm is its own padding; the space between the section's own heading, prose
   and cards is a `blockGap`/gap attribute. One owner per measurement — the ownership table is
   `../../nextora-design-system/references/presets-and-switching.md` §3.2.
2. **Adjacency is authored, not incidental.** Two rules that both "look fine" apart can touch when
   stacked, and no per-section check can see it.
3. **The first and last sections are boundaries too.** A section that pads its top because it
   *used* to follow another section leaves unwanted space above the header when it becomes the
   first section.

Which is why the acceptance step reads measurements rather than eyeballing the page:

| Boundary | What to read (computed) | Passes when |
| --- | --- | --- |
| Header → first section | the section's own top padding | non-zero and what the pattern authored; no extra outer margin |
| Section → section, every pair | previous section's bottom padding **and** next section's top padding | both non-zero and matching the authored presets; the pair does not touch |
| Last section → footer | the section's own bottom padding | non-zero, and no accidental outer margin after it |
| Inside a section | the heading/prose/card gap | matches the control, and clearing the control restores the theme value with no empty inline declaration |

Repeat at a narrow and a wide width. A `clamp()`-based preset whose lower bound collapses is a real
rhythm defect at the narrow end, and the wide end is where an over-large preset reveals itself.
Both are invisible in a single-width screenshot.

Conventional layout templates solve this with padding on a single element and no margins; a block
theme's equivalent is the section block owning it, which is why a *core* group used as a section
needs its spacing set explicitly in the pattern rather than left to the theme's root `blockGap`.

Observed today, and not to be copied into new work: **[observed defect]**

- The sample holds the same section list twice — `patterns/page-home.php`
  (`Inserter: no`) and `templates/front-page.html` — and repeats that pairing for
  its other nine hidden patterns: `page-archive`, `page-search`, `page-post`
  (paired with `templates/single.html`), `page-about-us`, `page-contact-us`,
  `page-faq`, `page-pricing`, `page-product` and `page-error-404` (paired with
  `templates/404.html`).
- The parent inlines pattern markup directly into its templates and tags it with
  `metadata.patternName` (for example
  `"metadata":{"categories":["nextora-page-title"],"patternName":"nextora/page-title-with-breadcrumbs","name":"Page title with breadcrumbs"}`
  in `templates/archive.html`).

When you are merely editing an existing block in one of those surfaces, keep the
existing duplication and update the other copies only if the section content
actually changed.

## 6. Queries

### 6.1 Inherit the main query

Archives, the posts index, search and post detail delegate to `core/query` with
`"inherit":true` so WordPress's own request parsing supplies the context. Read
from `nextsaas/templates/search.html`:

```html
<!-- wp:query {"queryId":2,"query":{"perPage":10,"pages":0,"offset":0,"postType":"post","order":"desc","orderBy":"date","author":"","search":"","exclude":[],"sticky":"","inherit":true},"className":"nextsaas-search-loop","layout":{"type":"default"}} -->
<div class="wp-block-query nextsaas-search-loop">
<!-- wp:group {"className":"search-results","layout":{"type":"default"}} -->
<div class="wp-block-group search-results">
<!-- wp:post-template {"layout":{"type":"default"}} -->
<!-- wp:group {"className":"log-row","layout":{"type":"default"}} -->
<div class="wp-block-group log-row">
<!-- wp:post-terms {"term":"category","className":"meta"} /-->
<!-- wp:post-title {"level":3,"isLink":true} /-->
<!-- wp:post-excerpt {"moreText":"","excerptLength":24} /-->
<!-- wp:read-more {"content":"Read","className":"round-link"} /-->
</div>
<!-- /wp:group -->
<!-- /wp:post-template -->
</div>
<!-- /wp:group -->

<!-- wp:query-pagination {"layout":{"type":"flex","justifyContent":"space-between"}} -->
<!-- wp:query-pagination-previous /-->
<!-- wp:query-pagination-numbers /-->
<!-- wp:query-pagination-next /-->
<!-- /wp:query-pagination -->

<!-- wp:query-no-results -->
<!-- wp:paragraph -->
<p>Nothing matched your search. Try a shorter phrase, or one of the starting points above.</p>
<!-- /wp:paragraph -->
<!-- /wp:query-no-results -->
</div>
<!-- /wp:query -->
```

Requirements:

- Use `core/post-template` plus `core/post-*` blocks inside `core/query`. Never
  hand-write a PHP loop in a template, and never ship mock posts.
- Provide `wp:query-pagination` and `wp:query-no-results` on every archive-like
  composition.
- Use `wp:query-title` for archive/search titles and `core/post-title` for the
  entry title. On the single surface the sample composes `core/comments`,
  `core/comment-template`, `wp:comments-pagination` and `wp:post-comments-form`
  in `nextsaas/templates/single.html`, and renders the article body through its
  own `nextsaas/post-article-content` block, whose `render.php` applies
  `the_content` instead of using `core/post-content`. `core/post-content` appears
  in `nextsaas/templates/page.html`.
- A link literal (like the sample's `"No stories have been published here yet."`)
  is a hardcoded English string; new compositions should route it through the
  theme's own translated markup path.

### 6.2 A block that starts its own loop

Reuse the child's `inc/blocks/query.php` helpers when they exist; they are the
sanitising layer for attribute-driven queries:

| Helper | Rule it encodes |
| --- | --- |
| `nextsaas_query_term_ids( $value )` | accepts scalar or list, keeps positive integers in saved order, drops zero (which `category__in`/`tag__in` treat as a real term and would silently filter everything) |
| `nextsaas_posts_query_args( $attributes, $exclude_id )` | page size defaults to the literal `3` and a non-numeric value resets to `3` regardless of the calling block's declared default, then is clamped with `max( 1, min( 24, … ) )`; an empty filter list adds no clause |
| `nextsaas_search_available_types()` / `nextsaas_search_post_type( $filter )` | whitelist `page`, `post` and `product` when `post_type_exists( 'product' )`; an unknown filter falls back to "all types" |

A new child adapts the same shape with its own prefix. Prepared SQL is only for a
genuinely custom query and does not make an attacker-chosen `post_type` or
`taxonomy` safe — validate against allowed values first.

## 7. Reusing the parent contact form

The parent ships the form backend as a block; the child composes it. The sample's
contact pattern wraps `nextora/contact-form` in the child's own section block and
only overrides display attributes and labels:

```html
<!-- wp:nextsaas/contact-us-contact -->
<!-- wp:nextora/contact-form {"showHeading":false,"showDescription":false,"enablePhoneField":false,"fullNameLabel":"Full name","emailLabel":"Work email","messageLabel":"What's on your mind?","buttonLabel":"Preview your message"} /-->
<!-- /wp:nextsaas/contact-us-contact -->
```

The parent also ships a full `nextora/contact-form` pattern
(`patterns/contact-form.php`, `Slug: nextora/contact-form`) that layers contact
details and social links next to the block. Reuse that backend; never copy its
submission handling into a child.

## 8. `theme.json` declarations

| Theme | `customTemplates` declared | Files shipped |
| --- | --- | --- |
| Parent | `about-us`, `page-full-width`, `contact-us`, `theme-terms-conditions`, `theme-privacy-policy`, `frequently-asked-questions` | matches |
| Sample child | the same six, plus `front-page`, `page-about-us`, `page-contact-us`, `page-faq`, `page-pricing`, `archive`, `single`, `search`, `404`, `page-product` | only the second group exists in `templates/` |

**[observed defect]** The sample re-declares six names it does not ship a file for
(they resolve to the parent's files) and also lists core template names
(`front-page`, `archive`, `single`, `search`, `404`) as custom templates.

**[new-output requirement]** Declare a `customTemplates` entry only when the name
is not a core template and the declaring theme actually ships
`templates/<name>.html`. Declare a `templateParts` entry only for a part the theme
ships in `parts/`. Parent-only custom templates stay parent-owned; do not
re-declare them as child-owned.

## 9. Media and content

**[observed defect]** Parent templates embed absolute external image URLs in block
attributes — for example
`"backgroundImageUrl":"https://pub-…r2.dev/alonepro/cgl-bg-slider-2.jpg"` with
`"backgroundImageId":850` in eight parent templates — `about-us.html`,
`archive.html`, `contact-us.html`, `front-page.html`, `home.html`, `index.html`,
`search.html` and `single.html` — plus placeholder addresses, phone numbers and an
email in `patterns/contact-form.php`. A new composition must use the site's own
attachments (an id from the media library, or an empty value), not a CDN path
from the demo environment.

Sample child templates, by contrast, contain no remote URLs. **[sample
convention]**

## 10. Filesystem versus Site Editor

Templates, parts and styles edited in the Site Editor are stored in the database
and take precedence over the files on disk. Consequences:

- A file edit that changes nothing on the front end means a database copy exists.
- The block editor reports a modified template; the theme's file is still the
  fallback, so deleting the customisation restores it.
- Inspect or reset only a disposable environment. Never delete a real site's
  customisations as a side effect of shipping a theme file.

Also confirm which theme actually resolves the surface: a child's copy of
`templates/<name>.html` or `parts/<name>.html` overrides the parent's, which is
why an "unchanged front end" after a *parent* file edit usually means the child
owns that surface.

## 11. Verification (downstream acceptance recipe)

Not run by this suite. Prerequisites: a **disposable** WordPress site with the
parent and the target child activated, the child's dependencies installed as its
`README` states, and working admin access. Run only commands the target's
`package.json` declares (the sample has `build`, `build:css`, `build:ts`,
`build:blocks`, `typecheck`, `lint:php:all`, `test:wp`; `.nvmrc` says `24`).

1. Visit each composed surface and confirm the intended template resolves —
   including that a child-owned part is used, not the parent's.
2. Insert each section pattern from the inserter: it must be offered in its
   category, insert, parse, and stay editable. Hidden (`Inserter: no`) patterns
   must not be offered.
3. Check the archive/search/index loops for pagination and a real no-results
   state, with the main query inherited.
4. Edit the hidden full-page pattern file on disk, reload the front end, and
   confirm every template referencing it changed — then confirm a *template* file
   edit that shows no change is explained by a database customisation.
5. Confirm no remote demo media appears on the front end.
6. **Read every section boundary on computed styles** — header→first, each adjacent pair, and
   last→footer — at a narrow and a wide width, using the table in §5.2. Clearing a spacing control
   must restore the theme default and leave no empty inline declaration.
7. Check the requested minimum WordPress version **and** the current supported
   release before claiming compatibility. Both source themes declare 6.4 / PHP
   8.1 while shipping `theme.json` v3 — a declaration, not proof.

If runtime is unavailable, report the composition as unverified rather than
substituting a markup-only check.
