---
name: nextora-template-porting
description: Analyze an arbitrary external source template before any Nextora WordPress implementation begins — a hand-written HTML site, a static export, a framework or React/Vue/Tailwind build, purchased-theme markup, a legacy site, or a set of screenshots plus a brief. Use when a request mentions porting, migrating or converting a source design, template or static site into a Nextora block child theme, auditing a source template, extracting its design tokens, mapping its sections to Gutenberg blocks and patterns, or producing a conversion roadmap. Read, analyze and plan only — it returns an evidence-backed conversion dossier and a phased roadmap, then hands implementation to the child-theme, design-system, site-composition and Gutenberg-block specialists — it never clones source markup, utility classes, framework conventions or pixels one-to-one.
---

# Nextora template porting

Reads an arbitrary source template — clean or tangled — and returns a fixed
conversion dossier plus a phased roadmap for rebuilding it as a Nextora WordPress
block child theme. It is a read/analyze/plan specialist: it inspects sources and
rendered evidence, normalizes their design system, maps their structure onto
WordPress surfaces, and dispatches the work. It does not create the child theme,
blocks, patterns, templates or runtime assets, and it never copies source markup,
utility or framework classes, ad-hoc variables or pixels one-to-one.

## References to load

Read these by relative path, in this order, before writing any dossier section:

| File | Read it for |
| --- | --- |
| `references/template-intake-and-roadmap.md` | The intake contract: evidence classification, the recursive source manifest, the three-system comparison, the token evidence table and role clustering, the section and interaction matrices, the fixed decision ladder, the asset ledger, the nine dossier headings and the phase-0-to-6 roadmap with gates. |
| [`source-map.md`](../nextora-ecosystem/references/source-map.md) | Provenance vocabulary and how to resolve the parent and active-theme roots from a live install or a repository. |
| [`quality-gates.md`](../nextora-ecosystem/references/quality-gates.md) | The gates the eventual implementation is measured against, plus the downstream acceptance recipes the roadmap's runtime phase hands to. |

Read the applicable specialist entrypoints — `../nextora-child-themes/SKILL.md`,
`../nextora-design-system/SKILL.md`, `../nextora-site-composition/SKILL.md`,
`../nextora-gutenberg-blocks/SKILL.md` — before naming their handoffs, and cite
their rules instead of restating them. Local sources outrank every value quoted in
these files.

## Ordered workflow

### 1. Freeze the evidence before recommending anything

Classify every supplied input by evidence type and certainty: source files
(HTML/templates, CSS or preprocessor output, JS/components, asset directories,
package/build metadata); rendered evidence (reachable URLs, screenshots,
viewport-specific states); a written brief or content inventory; and unavailable
or conflicting inputs, which are marked `unverified` rather than inferred. A
screenshot can establish visible hierarchy and approximate relationships; it can
never establish DOM semantics, token identity, interaction state, asset provenance
or implementation reuse.

### 2. Build the recursive source manifest

Before any recommendation, record entry pages/routes, shared shells, section and
component boundaries, stylesheet and script entrypoints, framework/build system,
assets, fonts and icons, forms and data sources, third-party embeds, interactive
behaviour, responsive breakpoints and repeated structures. Generated, minified and
vendor files are evidence of runtime behaviour, not preferred authoring sources —
trace them to authored inputs where those exist.

### 3. Read the live target; compare three systems, do not collapse them

Keep these separate in the dossier: (1) source-template reality; (2) live Nextora
parent and target-child capabilities, read from their own `style.css`,
`theme.json`, `blocks/`, `patterns/`, `templates/`, `parts/`, CSS entries, helpers
and build scripts; (3) the proposed normalized child-theme architecture. Root and
evidence resolution vocabulary is `../nextora-ecosystem/references/source-map.md`.

### 4. Fix the fidelity decision explicitly

Preserve, when the evidence supports it, the source content, information
architecture, section sequence, layout relationships, responsive intent and
recognizable brand cues. Normalize invalid semantics, duplicated wrappers,
utility/framework class noise, ad-hoc CSS variables, literal token copies,
inaccessible interactions and source-specific runtime conventions. Pixel-perfect
cloning is not an acceptance criterion; state this in the dossier rather than
leaving fidelity to interpretation.

### 5. Normalize the source into a trustworthy design-system proposal

Produce one token-evidence row per observed candidate (source selector, variable or
file; observed value(s); usage count and surfaces; semantic role; responsive and
state variants; conflicts; proposed WordPress destination) and **cluster repeated
values by role, not by numerical similarity**. Cover palette plus
foreground/background role pairs, gradients, typography families/weights/sizes/line
heights, spacing scale, container widths, radii, borders, shadows, breakpoints and
motion. Map the result onto the live parent's preset identities and the
`--nextora-font-body` / `--nextora-font-heading` / `--nextora-font-button` roles,
then emit the `source → normalized role → WordPress mechanism → owner skill` table.
The mechanism set, the contrast and spacing contracts and the storage rules are
owned by `../nextora-design-system/SKILL.md` and
`../nextora-design-system/references/presets-and-switching.md` §3.1–§3.2: hand
those decisions over, and treat a source hex or a screenshot as evidence, never as
authority.

### 6. Convert source structure into an editable architecture map

Produce a page/route inventory followed by a section matrix in which every section
carries its source page/anchor, purpose and retained content, source dependencies,
repeated/reusable status, target WordPress surface, closest existing
Nextora/child/core candidate, decision, editor ownership, dynamic-data needs,
responsive behaviour, asset needs and handoff skill. Apply the fixed decision
ladder in order — reuse an existing filesystem pattern, reuse an existing
parent/child or core block, compose a new child pattern from existing blocks,
assemble patterns through a thin template with the correct landmark and one
authority, override a template part only when the child owns that chrome, extend
the closest existing block when its editing model fits, and create a new child
block only when behaviour, data shape or editing structure cannot be expressed by
the earlier choices. Classify every section as `reuse`, `compose`, `extend`,
`new block`, `replace` or `discard`, and name the ownership model per full-page
surface: theme-owned fixed composition (hidden pattern + thin template) or
editor-owned narrative and order (`core/post-content` plus an insert-once starter
pattern). For interactions, add an interaction matrix (trigger/state/data, keyboard
and no-JS behaviour, closest existing library or component, server/data
dependency, target). Template hierarchy, part ownership and the reuse ladder belong
to `../nextora-site-composition/SKILL.md`; the custom-block contract belongs to
`../nextora-gutenberg-blocks/SKILL.md`. Never preserve a source framework because
its classes or bundled JS exist.

### 7. Inventory and dispose of assets without copying blindly

Give every image, SVG/icon, video, font and third-party resource an asset-ledger
row: source path or URL, pages/sections using it, dimensions and format, apparent
purpose, duplicate/hash relation when discoverable, license and provenance status,
accessibility-text requirement, and a disposition of `reuse locally`,
`convert/optimize`, `replace`, `media-library content` or `exclude`. Remote demo
URLs, tracking scripts, build outputs and vendor bundles are never proposed as
production theme assets, and unknown license or provenance is a blocked reuse
decision rather than implicit permission. Separate theme-owned design assets from
editorial media that belongs in the WordPress media library and content.

### 8. Emit the dossier under the nine fixed headings

Use these headings, in this order, with no implementation code beyond a tiny
excerpt needed to identify evidence: `Evidence and confidence`; `Source
inventory`; `Target capability inventory`; `Normalized design system`; `Page and
section conversion matrix`; `Asset and interaction disposition`; `Target
file/surface map`; `Conversion roadmap`; `Unverified and blocked decisions`.
Contents and required columns are in
`references/template-intake-and-roadmap.md`; the file/surface map names proposed
paths or surfaces only, with the owner skill beside each.

### 9. Emit the roadmap as phases 0–6 with a gate each

Phase 0 evidence freeze; phase 1 architecture and token decisions; phase 2 child
foundation; phase 3 composition skeleton; phase 4 custom blocks and interactions;
phase 5 content, assets and responsive refinement; phase 6 runtime acceptance.
Every phase lists its prerequisites, the exact matrix rows or surfaces it consumes,
the owner skill, an observable checkpoint and a rollback boundary. Phases,
required rows and gate wording are fixed in the reference. Do not estimate time and
do not add speculative features absent from the source or brief.

### 10. Hand off; never implement

Dispatch each roadmap phase to the specialist that owns it: child bootstrap,
assets and registrar to `nextora-child-themes`; canonical presets, role variables,
contrast pairs and spacing ownership to `nextora-design-system`; templates, parts,
patterns, hierarchy and section boundaries to `nextora-site-composition`; and
`nextora-gutenberg-blocks` only for behaviour or editing structure that fits none
of the earlier ladder choices. When one request asks for analysis and
implementation together, this dossier is frozen first, then the specialists run
against it. This skill never writes theme files.

## Hard limits

- Read/analyze/plan only. Never create or edit the child theme, blocks, patterns,
  templates, parts, styles or runtime assets, and never edit the source template.
- Never upgrade an inference into a fact: unavailable, conflicting or unread
  evidence is reported as unverified with the claim it blocks.
- Never propose a one-to-one copy of source markup, class names, framework
  conventions, bundled JS or pixel values; a value is preserved only when it serves
  a stable semantic role.
- Never create a parallel token layer or rename/re-point an inherited parent
  preset slug. Additions use child-owned semantic slugs.
- Never claim the conversion runs: the runtime verdict comes from the phase-6
  acceptance run on a disposable WordPress install.
- Never invent a path, asset, page or interaction that was not read or observed.

## Completion checks

- Every dossier input is classified by evidence type, and nothing unread is stated
  as fact.
- The source manifest covers pages/routes, shells, sections, build/runtime,
  interactions, breakpoints and assets, with generated files traced to authored
  inputs where they exist.
- The three systems are reported separately; the target capability inventory comes
  from the live parent and child files actually read.
- The token evidence table clusters by role, and the source → role → mechanism →
  owner table has no unresolved mechanism or owner.
- Every section has a matrix row and a classification from the fixed ladder; every
  full-page surface names its single ownership model.
- Every interaction and asset has a disposition; unknown provenance is blocked, not
  assumed.
- The dossier uses the nine fixed headings in order; the roadmap has phases 0–6,
  each with prerequisites, consumed rows, owner skill, checkpoint and rollback
  boundary.
- Implementation decisions are dispatched to the correct specialists, and any
  appearance, spacing or contrast decision is handed to `nextora-design-system`
  rather than settled here.

## Boundaries

Markdown instructions and source-backed references only: no runtime scripts,
generators, copied template trees, package management or installation automation.
This skill produces analysis and a roadmap; it never writes the child theme and
never modifies the parent, the target child or the source template. Downstream
build and browser verification belongs to the target theme and is listed in
`../nextora-ecosystem/references/quality-gates.md`.
