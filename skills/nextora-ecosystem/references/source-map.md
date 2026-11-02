# Source map

Where every rule in this suite comes from, and how to re-check it. **Source date:
2026-11-02.** Read this before authoring, then **reread the local sources
yourself before editing** — this file records anchors and provenance, not current
values.

## How to resolve the roots

Do not look for a fixed path. Resolve from the live install or the repository you
were handed:

- Parent (Nextora) = `get_template_directory()`, or the theme whose `style.css`
  declares `Theme Name: Nextora`.
- Child = `get_stylesheet_directory()`, or the theme whose `style.css` declares
  `Template: nextora`.
- Outside WordPress: find the theme root by the `style.css` header, then read
  `functions.php`, `theme.json` and `package.json` from that root.

All anchors below are **relative to a theme root**. Nothing here may be turned
into an absolute workstation path in shipped output.

Provenance tags used below:

- **[parent contract]** — enforced by Nextora code at runtime; matching it is required.
- **[sample convention]** — how the sample child happens to be built; copy the pattern, not necessarily every value.
- **[new-output requirement]** — a rule this suite imposes on new work.
- **[observed defect]** — a real inconsistency in the sources; do not reproduce it.

## Nextora (parent) anchors

| Anchor (relative) | Evidence |
| --- | --- |
| `style.css` | `Theme Name: Nextora`, `Version: 0.0.22`, `Requires at least: 6.4`, `Tested up to: 7.0`, `Requires PHP: 8.1`, `Text Domain: nextora`. **[parent contract]** |
| `functions.php` | Small bootstrap; feature code lives under `inc/`. |
| `theme.json` | Palette slugs `base`, `contrast`, `primary`, `paragraph`, `secondary`, `surface`, `transparent`; gradient slugs `gradient-primary`, `gradient-secondary`, `gradient-tertiary`, `gradient-subtle`, `gradient-soft`, `gradient-overlay`; font-family slugs `sans`, `mono`, `hanken-grotesk`, `josefin-sans`, `noto-serif-display`, `jetbrains-mono`; template parts `header`, `footer`, `footer-newsletter`; a `customTemplates` list. **[parent contract]** |
| `blocks/blocks.php` | `nextora_register_blocks()` globs `get_template_directory() . '/blocks'` and calls `register_block_type( $block_dir )` on `init` — the parent registrar does **not** check that `block.json` is readable. `nextora_register_block_categories()` (filter `block_categories_all`) adds slug `nextora`, title "Nextora Blocks", icon `layout`. `nextora_filter_block_type_args()` (filter `register_block_type_args`, priority 10, 2 args) forces `category = 'nextora'` for every `nextora/*` block. `nextora_enqueue_block_category_filter()` (action `enqueue_block_editor_assets`, priority 5) adds an inline `wp-blocks` script applying the same coercion through `blocks.registerBlockType`. Conditional `require_once` of per-block `register-styles.php` / `register-editor.php`. **[parent contract]** |
| `scripts/build-blocks.mjs` | Globs `./blocks/*/index.{ts,tsx}` for editor bundles and `./blocks/*/view.ts` for view bundles. A plugin intercepts **every** `@wordpress/*` import via `onResolve`/`onLoad` and rewrites it to a `window.wp.*` shim; generated dependency handles come from a fixed package list. An asset-writer plugin emits `index.asset.php`. The view build has **no** WordPress shim and bundles its own dependencies. **[parent contract]** |
| `scripts/gen-block.mjs` | Parent-only scaffold: `npm run gen -- --name <slug> --ns nextora` (space-separated; `--name=<slug>` is not the verified form). **[parent contract]** |
| `package.json` | Scripts include `build`, `build:css`, `build:blocks`, `typecheck`, `lint:php:all`, `test:wp`. Dependency ranges include `gsap ^3.15.0`, `swiper ^11.2.10`, `@tiptap/* ^3.22.2`, `lucide ^1.7.0` / `lucide-static ^1.7.0`, `chart.js ^4.5.1`, `three ^0.185.1`, `ogl ^1.0.11`. **[parent contract]** |
| `resources/css/app.css` | Tailwind v4 with Preflight disabled; `@theme` aliases map to `--wp--preset--*` with fallbacks; role variables `--nextora-font-body` (defaults to the `sans` preset), `--nextora-font-heading`, `--nextora-font-button`; `@source` globs; module import order base → components → prose → overrides. **[parent contract]** |
| `inc/features/color-scheme-switcher/scheme-data.php` | `nextora_glob_preset_files()`, `nextora_parse_color_preset()`, `nextora_get_color_presets()`, `nextora_get_font_registry()`, `nextora_get_font_presets()`, `nextora_parse_theme_preset()`, `nextora_get_themes()`, `nextora_get_switcher_config()`, `nextora_get_switcher_payload()`, `nextora_resolve_current_request_theme()`, `nextora_get_theme_inline_css()`. Discovery order, config merge and request resolution live here. **[parent contract]** |
| `assets/js/scheme-switcher.js` | Hand-maintained browser asset, **outside** the build pipeline. `applyColor()`, `applyFont()`, `applyTheme()`, the `reset*()` functions, `readStorage()`/`save()`, `bootstrap()`, `buildUI()`, `applySectionVisibility()`. **[parent contract]** |
| `styles/color-scheme/*.json` | `developer`, `lime`, `midnight`, `peach`, `warm-sand`. Shape: theme.json-style `title` + `settings.color.palette`. **[parent contract]** |
| `styles/font-preset/*.json` | `classic`, `elegant`, `modern`, `mono`. Shape: `title` + `body` + `heading` font slugs. **[parent contract]** |
| `styles/typography/*.json` | Per-family style variations. **[parent contract]** |
| `scheme-switcher.json` | `version: 1`; `initial: {theme, color, font}`; `sections: {themes, colors, fonts}` each with `enabled` / `showDefault`. Curates the presets and supplies first-visit defaults. **[parent contract]** |
| `blocks/team-section/` | Parent repeatable-data block: `edit.tsx`, `member-edit-form.tsx`, `member-utils.ts`, `render.php`, `register-styles.php`. Stable per-member IDs, immediate patching. **[parent contract]** |
| `blocks/advanced-icon/color-utils.ts` | The scoped-colour helper the design system describes: preset slug, palette-matching opaque hex → slug, literal custom/partial-alpha values. **[parent contract]** |
| `.opencode/skills/nextora-add-theme-block/SKILL.md` | Predecessor block workflow. It instructs `--name=<slug>` and a `theme` category for chrome blocks; both are contradicted by the code above. **[observed defect]** |
| `.opencode/skills/nextora-theme-styling-and-tokens/SKILL.md` | Predecessor token workflow; its `app.css` import list omits modules that exist today. **[observed defect]** |
| `AGENTS.md`, `docs/*.md` | Useful intent, but `AGENTS.md` points at `.cursor` paths that do not exist, and several docs predate the current preset slugs and categories. Verify against code. **[observed defect]** |

## Sample child (`nextsaas`) anchors

The sample is a reference for structure and conventions, not a template to clone
and not a statement about any specific child design.

| Anchor (relative) | Evidence |
| --- | --- |
| `style.css` | `Template: nextora`, `Version: 1.0.0`, `Requires at least: 6.4`, `Tested up to: 7.0`, `Requires PHP: 8.1`, `Text Domain: nextsaas`. Root stylesheet carries the header only. **[sample convention]** |
| `functions.php` | Documents that a child's `functions.php` loads **before** the parent's, so nothing at file scope may call a parent function or read a `NEXTORA_*` constant. Declares `strict_types`, guards `ABSPATH`, defines `NEXTSAAS_DIR` / `NEXTSAAS_URI`, `require_once`s `vendor/autoload.php` only when readable, then loads `inc/*` and `blocks/blocks.php`. **[sample convention]** |
| `inc/bootstrap/version.php` | Single `NEXTSAAS_VERSION` constant ('1.0.0'); the `style.css` header must be kept in sync with it. **[sample convention]** |
| `inc/assets/enqueue.php` | `nextsaas_asset_version()` returns a `filemtime()`-based version in development and the released theme version otherwise. Styles: `wp_enqueue_scripts` priority 25, dependencies on the parent's registered `global-styles` and `nextora-app`. Scripts: `wp_enqueue_scripts` priority 30, dependency `nextora-main`. Editor canvas: `after_setup_theme` priority 20 + `add_editor_style()`. Compiled file names deliberately avoid the parent's `app.css` / `main.js` because `add_editor_style()` resolves against the **stylesheet** directory. **[sample convention]** |
| `inc/blocks/query.php` | `nextsaas_query_term_ids()`, `nextsaas_posts_query_args()`, `nextsaas_search_available_types()`, `nextsaas_search_post_type()`, `nextsaas_search_url()`, `nextsaas_search_count_text()` — the sanitised query-attribute layer. **[sample convention]** |
| `inc/blocks/helpers.php`, `inc/navigation/setup.php`, `inc/template/body-class.php`, `inc/compat/beplus-visual-mega-nav.php` | Optional feature files, loaded only because that child uses those features. **[sample convention]** |
| `blocks/blocks.php` | `nextsaas_register_blocks()` globs `get_stylesheet_directory() . '/blocks'`, skips directories without a readable `block.json`, then `register_block_type()` and a conditional `require_once` of that block's `register-editor.php`. Category slug `nextsaas`, title "NextSaaS Sections", icon `art`. Server and client category coercion mirror the parent. `nextsaas_enqueue_editor_styles()` loads one shared compiled inspector stylesheet; `nextsaas_register_pattern_categories()` registers pattern categories. **[sample convention]** |
| `blocks/<slug>/block.json` | Blocks are named `nextsaas/<slug>` with `category: "nextsaas"`, `textdomain: "nextsaas"`, `apiVersion: 3`. **[sample convention]** |
| `resources/ts/editor/` | Shared editor helpers: `block-preview.tsx`, `rich-text-overlay.tsx`, `rich-text-control.tsx`, `colour-controls.tsx`, `media-control.tsx`, `repeater.tsx`, `icons.tsx` (imported by `repeater.tsx`), `types.ts`, `url-help.ts`. **[sample convention]** |
| `resources/css/` | `nextsaas.css` + `nextsaas-editor.css` entry points and ordered modules `00-tokens`, `10-base`, `20-components`, `25-sections`, `30-chrome`, `40-plugin`, `90-overrides`. `00-tokens.css` line 2 aliases `--nextsaas-*` onto `--wp--preset--*`, and pins sample fonts directly (`--nextsaas-font-display` → the `bodoni-moda` preset). **[sample convention]** |
| `theme.json` | Extends the parent palette with `border`, `accent-soft`, `mist`, `cobalt`; adds the `bodoni-moda` and `dm-sans` font families; declares its own `customTemplates`. **[sample convention]** |
| `assets/` | `css/nextsaas.css`, `css/nextsaas-editor.css`, `js/nextsaas.js`, `fonts/{bodoni-moda,dm-sans}/*.ttf`, `licenses/*OFL.txt`. **[sample convention]** |
| `package.json` | `build`, `build:css`, `build:ts`, `build:blocks`, `typecheck`, `lint:php:all`, `test:wp`. There is **no** `gen` script — the parent's generator is parent-only. **[sample convention]** |
| `patterns/*.php`, `templates/*.html`, `parts/*.html` | Composition conventions: PHP pattern headers, serialized block markup, thin templates, filesystem patterns. There is **no** `styles/` directory, so this child currently contributes no colour-scheme or font presets of its own. **[sample convention]** |

## Facts that are commonly mis-stated

| Claim | Reality |
| --- | --- |
| "Nextora blocks use the `theme` category." | `nextora_filter_block_type_args()` and the editor filter force `nextora` for every `nextora/*` block, whatever `block.json` says. |
| "Anything under `@wordpress/*` is externalised and safe to import." | The shim rewrites every `@wordpress/*` import, but only a fixed package list becomes a registered script dependency. An unlisted package can build and then read an undefined global. |
| "`npm run build` regenerates the switcher script." | `assets/js/scheme-switcher.js` is hand-maintained and outside the build pipeline. |
| "A rebuild gives a clean diff." | Parent `index.asset.php` versions are generated from `Date.now()`. |
| "`nextsaas` proves preset switching works." | The sample pins its own fonts and adds child-only palette entries; its tokens do not demonstrate that visitor switching reaches a child's roles. |
| "`test:wp` proves browser behaviour." | It is a server-side workflow; read the target's `scripts/wp-test.sh` and blueprint before promising a port, login or browser proof. |
| "WordPress 6.4 support is proven." | 6.4 / PHP 8.1 is a **declared** minimum while `theme.json` is v3 and modern package APIs are used. Treat it as unproven until checked at runtime. |

## If a path is missing

1. Rediscover the replacement from the live roots (`get_template_directory()`,
   `get_stylesheet_directory()`) or from the supplied repository.
2. If the parent source is genuinely absent, stop claiming Nextora compatibility:
   state the missing prerequisite, and complete only the source-independent work.
3. Never fabricate an anchor, never reuse a value from this file as if it were
   current, and never substitute a different theme silently.
