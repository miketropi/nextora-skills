# Template intake and conversion roadmap

The detailed authority for `nextora-template-porting`. It defines how to intake an
arbitrary source template, how to normalize it, how to map it onto WordPress
surfaces, and the exact shape of the dossier and roadmap the skill must return.
The entrypoint's workflow summarizes this file; this file is what the dossier is
checked against.

Related: [`../../nextora-ecosystem/references/source-map.md`](../../nextora-ecosystem/references/source-map.md)
(root and provenance vocabulary),
[`../../nextora-ecosystem/references/quality-gates.md`](../../nextora-ecosystem/references/quality-gates.md)
(the gates the implementation is later measured against), and the four
implementation contracts this file hands to:
[`../../nextora-child-themes/SKILL.md`](../../nextora-child-themes/SKILL.md),
[`../../nextora-design-system/SKILL.md`](../../nextora-design-system/SKILL.md),
[`../../nextora-site-composition/SKILL.md`](../../nextora-site-composition/SKILL.md),
[`../../nextora-gutenberg-blocks/SKILL.md`](../../nextora-gutenberg-blocks/SKILL.md).
None of their rules are restated here — cite them.

## 1. Evidence intake contract

The source template is whatever the requester supplied. It may be a clean modern
tree, a tangled legacy export, or a mixture of files, a reachable site and
screenshots. Never assume its shape; classify every supplied input first, and keep
the classification visible in the dossier.

Classify each input into exactly one evidence class:

| Evidence class | Examples | What it can establish |
| --- | --- | --- |
| Source files | `*.html` templates, component files, `*.css`/`*.scss`/`*.less` output or sources, `*.js`/`*.ts`/`*.jsx`/`*.vue`/`*.svelte`, asset directories, `package.json`, lockfiles, build config, framework config | Structure, authored values, token candidates, component boundaries, build/runtime shape. |
| Rendered evidence | Reachable URL, screenshots, per-viewport captures, DevTools-style computed values captured by the requester | Visible hierarchy, approximate layout relationships, visible responsive states. |
| Written brief / content inventory | Supplied copy, sitemap, page list, CMS export, section list, brand notes | Intended content, purpose, ordering intent, brand constraints. |
| Missing or conflicting | A referenced file that is absent, an unreadable/private URL, two sources disagreeing on a value or a section | Nothing as fact. Recorded as `unverified` with the claim it blocks. |

Certainty rules:

- A value is a **fact** only if it was read from a source file, observed in a
  rendered capture supplied for this task, or stated in the brief.
- A screenshot establishes **visible hierarchy and approximate relationships**. It
  does **not** establish DOM semantics, element or landmark identity, token
  identity, interaction implementation or state, asset provenance, or
  implementation reuse.
- A value inferred by pattern or convention is **inference**, labelled as such, and
  must not be used as the basis for a normalize-and-ship decision on its own.
- Conflicting evidence is never silently resolved in favour of the tidier source;
  record both, name which is authoritative if the brief says so, otherwise mark the
  claim blocked.

Generated, minified, transpiled, transpiled-CSS and vendor bundle files are
evidence of **runtime behaviour**, not preferred authoring sources. Use them to
confirm what ships, then trace the value to its authored input; when no authored
input exists, say so and treat the value as a runtime observation.

## 2. Recursive source manifest

Produce a manifest before any recommendation. It is the inventory of what exists,
not an opinion about it. Cover, at minimum:

| Area | Record |
| --- | --- |
| Entry pages/routes | Every page, route or view file and how it is reached (file, router, build entry, static path). |
| Shared shells | Header, footer, nav, sidebar, layout wrappers, and which pages use each. |
| Section/component boundaries | Each repeated or named region and where it appears. |
| Stylesheets and scripts | Every stylesheet and script entrypoint, its load order and what it actually ships. |
| Framework / build system | Framework, CSS approach, bundler, package manager, build metadata, and whether outputs are committed. |
| Assets | Images, SVG/icons, video, fonts and their directories. |
| Forms and data sources | Forms, their endpoints or handlers, and any data the pages fetch or render from. |
| Third-party embeds | Analytics, tag managers, chat widgets, maps, video embeds, CDN scripts. |
| Interactive behaviour | Tabs, accordions, carousels, filters, modals, menus, animations, scroll effects. |
| Responsive breakpoints | Every breakpoint and what changes across it. |
| Repeated structures | Anything duplicated across pages (chrome, cards, CTA bands, marketing sections). |

Repeated structures are recorded with their repetition count and their differences;
those differences are where a shared WordPress surface usually wins.

## 3. Compare three systems, separately

The dossier compares three systems and never collapses them:

1. **Source-template reality** — what the source actually is, from §1 and §2.
2. **Live target capabilities** — what the Nextora parent and the target child
   already provide, read from their own `style.css`, `functions.php`, `theme.json`,
   `blocks/`, `patterns/`, `templates/`, `parts/`, CSS entries, editor helpers and
   build scripts. Resolve the roots without assuming a path
   (`../../nextora-ecosystem/references/source-map.md`). Record existing blocks,
   patterns, template parts, preset slugs, role variables and build commands as the
   **candidate pool** the ladder in §9 draws from.
3. **Proposed normalized child architecture** — what the port should become. It is
   only meaningful relative to (2): a proposal that ignores an existing pattern,
   block or preset is incomplete.

If the parent or child source is unavailable, the target capability inventory
states that plainly and every decision that depends on it is marked blocked; never
fabricate a capability.

## 4. Bounded dossier when source or render access is missing

Screenshots-only or brief-only requests are still answerable, but bounded. In that
case:

- Still produce the manifest of what is visible, the page inventory and the section
  matrix from the rendered evidence.
- Label DOM semantics, token identity, interaction implementation, asset
  provenance and any implementation-reuse claim `unverified`.
- List what can still be decided (section order, visible hierarchy, content,
  approximate responsive intent) versus what is blocked.
- State exactly which additional input would unblock each blocked decision: an
  authored stylesheet, a page URL, the source repository, a font licence file.

Never upgrade a screenshot inference into a fact to fill a table.

## 5. Fidelity decision

State the decision in the dossier, once, explicitly:

**Preserve** (when evidence supports it): source content, information architecture,
page and section sequence, layout relationships, responsive intent, and
recognizable brand cues (logo treatment, the dominant hue family, the display
typeface character).

**Normalize**: invalid or non-semantic markup, duplicated wrappers and depth,
utility and framework class noise, ad-hoc CSS variables, literal token copies
(hexes and lengths kept in place of preset identity), inaccessible interactions,
and source-specific runtime conventions (framework hydration, bundled JS, custom
breakpoints that duplicate the theme's).

**Not an acceptance criterion**: pixel-for-pixel visual parity. The port is judged
by preserved information architecture and content, a normalized design system that
follows the theme's roles, and a composition that is editable and accessible — not
by matching the source's markup or metrics.

## 6. Normalized design system

### 6.1 Token evidence table

One row per observed candidate. Columns:

| Column | Content |
| --- | --- |
| Source anchor | Selector / variable name / file path (and line when cheap). |
| Observed value(s) | The literal value(s), including every near-duplicate. |
| Usage count / surfaces | How many declarations or surfaces use it, and where. |
| Semantic role | The role it serves (page background, muted body text, primary action, card radius, section rhythm …). |
| State / responsive variants | Hover/focus/active/disabled variants, alpha overlays, per-breakpoint values. |
| Conflicts | Competing or contradictory values for the same role. |
| Proposed destination | The WordPress mechanism plus the target role/slug (§7). |

### 6.2 Clustering rules

- Cluster **by role**, not by numerical similarity. Two colours three units apart
  are one role if they serve one role; two identical values are two roles if they
  serve two.
- Preserve a source value only when it serves a stable role across the source.
- Merge accidental near-duplicates into one role and say which literals collapsed.
- Identify one-off exceptions explicitly as exceptions, and propose them as a
  scoped literal (or drop them) rather than promoting them to a preset.
- Cover, at minimum: palette and foreground/background role pairs; gradients;
  typography families, weights, sizes and line heights; spacing scale; container
  widths; radii; borders; shadows; breakpoints; motion durations and easings.

### 6.3 Mapping rules

- Reuse the live parent's existing preset identities and role variables. **Never**
  rename or re-point an inherited slug, and never copy a sampled child's palette.
- Additions receive **child-owned semantic slugs**; they are additions to the
  merged preset set, not a second token layer.
- Do not create a parallel token layer when `theme.json`, preset variables, core
  block supports or an existing Nextora role already express the need.
- Typography resolves through `--nextora-font-body` / `--nextora-font-heading` /
  `--nextora-font-button`; a surface reading a preset variable directly will not
  follow a visitor font change.
- Colour is a **readable surface pair** (foreground(s) versus the surface actually
  painted), and spacing has **exactly one owner per measurement**. The thresholds,
  the composited-surface rule and the ownership table are owned by
  `../../nextora-design-system/references/presets-and-switching.md` §3.1–§3.2:
  hand these decisions to `nextora-design-system`, and require a computed contrast
  and responsive-spacing check during implementation rather than asserting it here.
- A source hex, a screenshot or a sampled pixel is **evidence**, never authority.

## 7. Source → normalized role → mechanism → owner

The dossier carries this table. Every normalized token appears once.

| Source anchor | Normalized role | WordPress mechanism | Owner skill |
| --- | --- | --- | --- |
| … | … | one of the set below | … |

Mechanisms are **limited** to:

- `theme.json` preset / style (colour, gradient, font family, font size, spacing);
- a Nextora role variable (`--nextora-font-*`, or a role the parent already defines);
- a core block support (wrapper colour, spacing, typography);
- a child-scoped attribute resolved through the child's own helper;
- a style variation (`styles/*.json`);
- visitor-switcher data (`styles/color-scheme/*.json`, `styles/font-preset/*.json`,
  `scheme-switcher.json`) — data only, never an implementation;
- authored CSS, **only** where no native mechanism exists, and **only** for a
  scoped surface.

Any `authored CSS` row must state why the native mechanisms above cannot represent
it. "Faster to write" is not a reason. Storage, contrast and spacing rules are not
decided in this table — they are handed to `nextora-design-system`.

## 8. Page and section conversion matrix

### 8.1 Page/route inventory

One row per source page or route: source path/URL, purpose, its shell(s), its
sections in order, whether it is editor-owned or theme-owned content, and its
target WordPress surface (template, template part, pattern or editor content).

### 8.2 Section matrix

One row per section. Columns:

| Column | Content |
| --- | --- |
| Source page / anchor | Where the section lives in the source. |
| Purpose and retained content | What the section is for and the copy/structure that survives the port. |
| Source dependencies | Classes, framework components, scripts, embeds, data it relies on. |
| Repeated / reusable | Whether it recurs across pages, and how instances differ. |
| Target WordPress surface | Template, part, pattern, block or editor content. |
| Closest existing candidate | The nearest existing Nextora/child/core pattern or block, named from the live target. |
| Decision | `reuse`, `compose`, `extend`, `new block`, `replace` or `discard` (§9). |
| Editor ownership | Theme-owned or editor-owned, and who edits what (§10). |
| Dynamic-data needs | Query, taxonomy, custom fields, REST, none. |
| Responsive behaviour | What changes across breakpoints and how. |
| Asset needs | Which asset-ledger rows it consumes. |
| Handoff skill | The specialist that implements it. |

## 9. Decision ladder and classification

Apply in this order and stop at the first that fits:

1. Reuse an existing **filesystem pattern**.
2. Reuse an existing **parent/child or core block**.
3. Compose a **new child pattern** from existing blocks.
4. Assemble patterns through a **thin template** with the correct landmark and one
   authority.
5. **Override a template part** only when the child owns that chrome.
6. **Extend the closest existing block** when its editing model fits.
7. Create a **new child block** only when behaviour, data shape or editing
   structure cannot be expressed by choices 1–6.

Classification labels:

| Label | Meaning |
| --- | --- |
| `reuse` | An existing pattern, block or part already is the section. |
| `compose` | A new pattern assembled from existing blocks. |
| `extend` | The closest existing block is extended; its editing model fits. |
| `new block` | Nothing earlier in the ladder expresses the behaviour, data shape or editing structure. |
| `replace` | Source behaviour that WordPress/core/Nextora already provides more safely (forms, menus, lightboxes, query loops). |
| `discard` | Requires a concrete reason: duplicate chrome, inaccessible decoration, dead interaction, non-product demo content, tracking, or a section with no retained content. |

`new block` is the exception, not the default. A source `<div>` with classes is not
a reason for a block; a source framework's component is not a reason to keep the
framework.

## 10. Ownership model per full-page surface

Separate **content ownership** from **layout ownership**, and name the model for
every full-page surface so implementation does not invent it later:

- **Theme-owned fixed composition** — the section list lives in a hidden filesystem
  pattern (`Inserter: no`) referenced from a thin template, with one authority and
  no duplicated section list.
- **Editor-owned narrative / order** — `core/post-content` plus an insert-once
  starter pattern, for pages whose owner is expected to rewrite copy and reorder
  sections.

The composition contract (template hierarchy, pattern headers, part ownership,
the thin-template recipe) is owned by
`../../nextora-site-composition/SKILL.md`; this dossier only names which model
applies where.

## 11. Interaction matrix

One row per source interaction. Columns:

| Column | Content |
| --- | --- |
| Trigger / state / data | What starts it, the states it has, the data it needs. |
| Keyboard and no-JS behaviour | Whether it is operable by keyboard and what happens without JavaScript. |
| Closest existing component | The nearest core/Nextora block behaviour, or the nearest library already in the target. |
| Server / data dependency | Anything required beyond the browser. |
| Target | `core behavior`, `existing block`, block `viewScript`, plugin/service integration, or `omit`. |
| Reason | Why that target, and why not the next-closest option. |

Never preserve a source framework, its classes or its bundled JS solely because
they exist. Behaviour that core already provides (accordion, navigation, lightbox,
query filtering within supported controls) is `core behavior` or `replace`.

## 12. Asset ledger

One row per image, SVG/icon, video, font and third-party resource. Columns:

| Column | Content |
| --- | --- |
| Source path / URL | Where it comes from; flag remote demo hosts. |
| Used by | Pages/sections that reference it. |
| Dimensions / format | Intrinsic size and format where discoverable. |
| Apparent purpose | Decorative chrome, brand mark, content photo, icon, background. |
| Duplicate / hash relation | Duplicates and near-duplicates when discoverable. |
| License / provenance | Known, unknown or blocked. |
| Accessibility text | Whether it needs alt text and who authors it. |
| Disposition | `reuse locally`, `convert/optimize`, `replace`, `media-library content` or `exclude`. |

Rules:

- Remote demo URLs, tracking scripts, build outputs and vendor bundles are **never**
  proposed as production theme assets.
- Unknown license or provenance is a **blocked reuse decision**, not implicit
  permission.
- Distinguish **theme-owned design assets** (fonts, decorative chrome, block
  placeholders) from **editorial media** that belongs in the WordPress media
  library and content.
- Preserve an existing file only when it is reusable, licensed and compatible with
  the normalized architecture.

## 13. The dossier

Headings, in this exact order, with no implementation code beyond a tiny excerpt
needed to identify evidence:

1. **Evidence and confidence** — roots read, render states observed, missing and
   conflicting inputs, and the certainty classification (§1).
2. **Source inventory** — pages, shells, sections, build/runtime, interactions and
   assets (§2).
3. **Target capability inventory** — reusable parent/child/core blocks, patterns,
   helpers, presets and pipelines, read from the live target (§3).
4. **Normalized design system** — the token evidence table and the
   source → role → mechanism → owner table (§6–§7).
5. **Page and section conversion matrix** — every section classified by the fixed
   ladder (§8–§9).
6. **Asset and interaction disposition** — the interaction matrix and asset ledger
   (§11–§12).
7. **Target file/surface map** — proposed paths/surfaces only, with the owner skill
   beside each. A proposal, not a file manifest to create here.
8. **Conversion roadmap** — the phases and gates in §14.
9. **Unverified and blocked decisions** — every blocked claim with the input that
   would unblock it.

## 14. Conversion roadmap

Phases and gates are fixed. The roadmap must not skip a phase, reorder them, or
substitute its own gates.

| Phase | Work | Owner skill | Observable checkpoint (gate) |
| --- | --- | --- | --- |
| 0. Evidence freeze | Source manifest, render captures/routes, evidence classification | `nextora-template-porting` | Every claim is traceable to a read source or explicitly marked unverified. |
| 1. Architecture and token decisions | Normalized design system, page models, section classifications | `nextora-template-porting` (+ `nextora-design-system` for token/contrast/spacing sign-off) | No unresolved owner or mechanism: every token row and section row has a mechanism, an owner and a class. |
| 2. Child foundation | Bootstrap, presets/roles, build pipeline | `nextora-child-themes` (+ `nextora-design-system`) | Child activates and the merged presets and built assets are proven on a disposable install. |
| 3. Composition skeleton | Templates, parts, patterns from reusable surfaces | `nextora-site-composition` | Route hierarchy, landmarks, pattern parsing and edit ownership proven. |
| 4. Custom blocks / interactions | Only matrix rows classified `extend` or `new block` | `nextora-gutenberg-blocks` | Editor semantics, save/reload, no-JS and front-end parity proven. |
| 5. Content, assets and responsive refinement | Approved assets and retained content integrated | `nextora-site-composition` + `nextora-design-system` | No remote demo dependency; responsive intent preserved; computed design-system values pass. |
| 6. Runtime acceptance | Apply the shared quality gates in a disposable WordPress environment | as declared in `quality-gates.md` §8 | Requested routes, widths, accessibility, contrast, section rhythm, build reproducibility and every declared-version cell pass or are explicitly unverified. |

Each phase row must also state:

- **Prerequisites** — the earlier phase(s) and the specific dossier sections it
  consumes.
- **Consumed rows/surfaces** — the exact matrix rows or surfaces it implements.
- **Rollback boundary** — what can be reverted without touching the earlier phases
  (for example: a phase-4 block can be removed without disturbing phase-3 patterns,
  because the pattern references it by name only).

Do not estimate time. Do not add speculative features absent from the source or
brief; if a feature is not in the source or the brief, it is not in the roadmap.

## 15. Handoff map

| Dossier surface | Owner skill | Their authority for it |
| --- | --- | --- |
| Child bootstrap, header, asset pipeline, registrar | `nextora-child-themes` | `../../nextora-child-themes/SKILL.md` and its `references/child-contract.md` |
| Presets, role variables, contrast pairs, spacing ownership, style variations, switcher data | `nextora-design-system` | `../../nextora-design-system/SKILL.md` and its `references/presets-and-switching.md` |
| Templates, parts, patterns, hierarchy, section boundaries, full-page authority | `nextora-site-composition` | `../../nextora-site-composition/SKILL.md` and its `references/templates-and-patterns.md` |
| Block metadata, editor UI, render, repeaters, view scripts — only for `extend`/`new block` rows | `nextora-gutenberg-blocks` | `../../nextora-gutenberg-blocks/SKILL.md` and its references |

This skill cites those contracts and never restates or overrides them. Where a
dossier row names a mechanism, a slug, a threshold or a spacing owner, the owning
specialist's rule wins, and the dossier defers.
