---
name: nextora-ecosystem
description: Entry point for work on the Nextora block-theme ecosystem — child themes, Gutenberg blocks, design presets and the visitor colour/font switcher, template/part/pattern composition, and the intake and normalization of an external source template before it is ported. Use when a request mentions Nextora or a Nextora child theme, when the task spans more than one of those areas, when a request involves porting or converting an arbitrary source template or static site into a Nextora child theme, or when you need the source map and production quality gates for this ecosystem before writing code. Also use to decide which specialist skill a multi-part request needs.
---

# Nextora ecosystem — router

Coordinates six skills that share one source of truth and one set of production
gates. This entrypoint resolves the target theme, proves which specialists the
request needs, and hands over. It does not contain the implementation rules:
those live in the specialist skills and in this skill's two references.

## Skills in this suite

| Skill | Owns |
| --- | --- |
| `../nextora-child-themes/SKILL.md` | Creating or bootstrapping a Nextora child theme: headers, bootstrap, asset pipeline, block registrar. |
| `../nextora-gutenberg-blocks/SKILL.md` | Block metadata, editor UI, dynamic render, repeaters, settings panels, view scripts. |
| `../nextora-design-system/SKILL.md` | Palette/typography presets, style variations, the optional visitor colour/font switcher, token and CSS surfaces. |
| `../nextora-site-composition/SKILL.md` | Templates, template parts, patterns, query/archive composition. |
| `../nextora-template-porting/SKILL.md` | Intake, normalization and conversion planning for an arbitrary external source template: evidence classification, source manifest, token evidence and role mapping, section/interaction matrices, asset ledger, conversion dossier and phased roadmap. Analysis and planning only — it never implements. |

Shared evidence and gates for every specialist:

- `references/source-map.md` — where the rules come from, with provenance.
- `references/quality-gates.md` — what must be true before work is accepted.

## Ordered workflow

### 1. Resolve the target theme and its parent

Work from the live install, or from the repository you were given. Never invent a
path and never hardcode a machine path in output or notes.

Inside a WordPress install, the two roots are the template directory (the parent)
and the stylesheet directory (the activated theme, parent or child):

```php
get_template_directory();      // parent theme root
get_stylesheet_directory();    // active theme root (child when one is active)
get_stylesheet_directory_uri();
```

Outside an install, locate the theme by a `style.css` carrying a `Theme Name`
header and, for a child, a `Template:` header naming its parent. If a supplied
path is missing, rediscover the real one; never fabricate source evidence, never
silently pick a different theme, and never claim Nextora compatibility while the
Nextora source is unavailable — say the prerequisite is missing and finish the
work that does not depend on it.

### 2. Read local evidence before proposing code

Read these from the *target* theme, in this order, and prefer their live values
over anything quoted in a skill:

| File | What it settles |
| --- | --- |
| `style.css` | Theme name, Version, `Template:` (parent for a child), Requires at least, Requires PHP, Tested up to, Text Domain, License. |
| `functions.php` | Load order, what the theme requires at file scope, which feature files it pulls in. |
| `theme.json` | Palette, gradient, font-family, font-size, spacing slugs; custom templates and template parts; `version`. |
| `package.json` | Which build/verify commands actually exist, and which libraries are already dependencies. |
| `theme.js`/`.nvmrc`, build scripts | Node version and the real build entry points. |
| `AGENTS.md`, `README.md`, `docs/`, `.opencode/` | Project instructions. Treat them as claims to verify, not as authority. |

Then find what already exists before proposing anything new:

- `blocks/*/block.json` — registered blocks, their names, categories, supports.
- `parts/*.html`, `templates/*.html` — composition surfaces and their hierarchy.
- `patterns/*.php` — reusable section markup.
- `resources/css/` + `assets/css/` — which stylesheet actually ships and how it is built.
- `resources/ts/editor/` or equivalent — shared editor helpers to reuse instead of rewriting (preview shell, rich text, media, repeater).
- `inc/` — helpers for queries, navigation, templates.

Reuse beats invention: an existing block, pattern, helper or token that already
does the job is the correct answer, and a new block is only correct when the
requested behaviour or editing structure cannot be expressed otherwise.

### 3. Dispatch by intent

| Request | Specialists to load |
| --- | --- |
| Create, scaffold or bootstrap a child theme; child asset pipeline; child block registration | `nextora-child-themes` |
| Add/change a block: `block.json`, attributes, editor UI, `render.php`, view script, repeater, settings panel | `nextora-gutenberg-blocks` |
| Colours, fonts, spacing tokens, style variations, the visitor switcher, "must follow the palette" | `nextora-design-system` |
| Templates, template parts, patterns, page layouts, archives, post queries | `nextora-site-composition` |
| Analyze, audit or port an external source template — arbitrary HTML/static/framework source, screenshots, a brief; design-token extraction, section-to-block mapping, conversion roadmap | `nextora-template-porting` |

Combined requests load every applicable specialist. Three combinations are
obligatory:

- **External source template, in any form** → `nextora-template-porting` **first**,
  then every implementation specialist its dossier names, and no others. The
  template-porting dossier is frozen before implementation starts; it supplies the
  analysis and the roadmap and never overrides a specialist's implementation
  contract. `nextora-design-system` still owns every appearance decision the
  dossier hands it.
- **New child + any appearance requirement** → `nextora-child-themes` **and**
  `nextora-design-system`. A child that must follow preset or visitor switches
  cannot be finished by the child skill alone.
- **New or restyled block** → `nextora-gutenberg-blocks` **and**
  `nextora-design-system`. Block appearance is owned by the design system.

Because the specialists must agree, load them in the same session and let the
design-system skill win on any appearance question it covers.

### 4. Load references progressively

Each specialist is directly invocable; running this router is not a prerequisite.
When you do load a specialist, open its `SKILL.md`, then read only the reference
files its workflow names, by relative path. The two shared references above are
linked from every specialist, so cross-skill relative links
(`../nextora-ecosystem/references/...`) must keep resolving — do not move or
rename them.

### 5. Hand off

Load the specialist(s), follow their ordered workflow, and apply
`references/quality-gates.md` before declaring anything done. If the request is
purely a question about where a rule comes from, answer from
`references/source-map.md` and stop.

## Completion checks

- Target theme and parent identified from their own `style.css`/`theme.json`, not assumed.
- Only real, existing files cited; anything quoted as source was read in this session.
- Every applicable specialist loaded, including the mandatory pairings above.
- An external source-template request starts with `nextora-template-porting`, and
  its dossier is frozen before any implementation specialist is loaded.
- New code reuses existing blocks, patterns, helpers and tokens where they exist.
- `references/quality-gates.md` applied to the result.
- Nothing in the work depends on this router's own file layout beyond the relative links above.

## Boundaries

This suite is Markdown instruction and source-backed reference. It ships no
runtime scripts, generators, starter theme trees, package manager or installer.
Downstream build and browser verification commands belong to the target theme and
are listed in `references/quality-gates.md`.
