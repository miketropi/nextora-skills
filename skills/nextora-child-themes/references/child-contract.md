# Child contract

The detailed rules behind `../SKILL.md` for a child theme of Nextora. Provenance
tags match the shared map:

- **[parent contract]** — enforced by Nextora code; matching it is required.
- **[sample convention]** — how the sample child (`nextsaas`) is built; copy the
  pattern, not every value.
- **[new-output requirement]** — a rule this suite imposes on new work.
- **[observed defect]** — a real inconsistency in the sources; do not reproduce it.

Shared sources: [`source-map.md`](../../nextora-ecosystem/references/source-map.md),
[`quality-gates.md`](../../nextora-ecosystem/references/quality-gates.md).
Reread the live `style.css`, `functions.php`, `inc/`, `blocks/blocks.php` and
`package.json` of both themes before editing; values below are anchors.

## 1. Slug substitution

One requested slug drives every identifier. The worked example is `nextfolio`.

| Surface | Rule | Example |
| --- | --- | --- |
| Child slug | kebab-case, matches the theme directory | `nextfolio` |
| PHP identifier prefix | slug with `-` → `_` | `nextfolio_` |
| Constants | `<CHILD_PREFIX>_VERSION`, `_DIR`, `_URI` | `NEXTFOLIO_VERSION`, `NEXTFOLIO_DIR`, `NEXTFOLIO_URI` |
| Text domain | the slug, everywhere | `nextfolio` |
| Block names | `<child-slug>/<block-slug>` | `nextfolio/team` |
| Block category | slug `nextfolio`, title `Nextfolio Blocks` | — |
| Pattern slugs | `<child-slug>/<pattern-slug>` | `nextfolio/page-home` |
| Pattern categories | `<child-slug>-sections`, `<child-slug>-pages` | — |
| CSS classes | `.<child-slug>-<block-slug>`, `__element`, `--modifier` | `.nextfolio-team__member--featured` |
| Custom properties | `--<child-slug>-*` | `--nextfolio-gutter` |
| Style handle | `<child-slug>-app` — the editor layer is a *file* passed to `add_editor_style()`, not an enqueued handle | `nextfolio-app` |
| Script handle | `<child-slug>-main` | `nextfolio-main` |
| Asset basenames | `assets/css/<slug>.css`, `assets/css/<slug>-editor.css`, `assets/js/<slug>.js` | `nextfolio.css` |

Never rename parent-owned names: `nextora_*`, `NEXTORA_*`, `nextora/*`, the
`global-styles` / `nextora-app` / `nextora-main` handles, the parent palette and
font slugs, and the `--nextora-font-body` / `--nextora-font-heading` /
`--nextora-font-button` role variables. A mechanically renamed parent API is a
broken child. **[parent contract]**

## 2. Structure recipe

Required for every child:

```text
style.css            header only; no rules
theme.json           child settings, palette/font additions, template declarations
functions.php        bootstrap
```

Add only what the child uses:

```text
inc/bootstrap/version.php     the single version constant
inc/assets/enqueue.php        front-end, script, editor-canvas pipeline
inc/blocks/helpers.php        block render helpers, when blocks exist
inc/blocks/query.php          query-attribute whitelists, when blocks run loops
inc/navigation/setup.php      only if the child registers its own menu location
inc/template/body-class.php   only if the design keys CSS off a body class
inc/compat/<plugin>.php       only if an optional plugin is integrated
resources/css/                source stylesheets (entry + ordered modules)
resources/ts/                 source scripts (entry + editor/runtime helpers)
assets/css/<slug>.css         compiled front end
assets/css/<slug>-editor.css  compiled editor layer
assets/js/<slug>.js           compiled script
blocks/blocks.php             registrar, only when blocks/ exists
blocks/<block-slug>/          one directory per block, with block.json
templates/  parts/  patterns/ only the surfaces the child owns
assets/fonts/  assets/licenses/  only fonts that actually ship
```

**[new-output requirement]** No empty feature scaffolding. The sample's
`inc/blocks/helpers.php`, `inc/blocks/query.php`, `inc/navigation/setup.php`,
`inc/template/body-class.php` and `inc/compat/beplus-visual-mega-nav.php` exist
because that design needs them; a new child loads a file only when it has that
file's job. **[sample convention]**

## 3. `style.css` header

```css
/*
Theme Name: Nextfolio
Theme URI: https://example.com/nextfolio
Description: <design intent; which surfaces it composes; what it inherits from Nextora>.
Author: <author>
Author URI: <url>
Template: nextora
Version: 1.0.0
Requires at least: 6.4
Requires PHP: 8.1
License: GNU General Public License v2 or later
License URI: https://www.gnu.org/licenses/gpl-2.0.html
Text Domain: nextfolio
Tags: block-patterns, block-styles, full-site-editing, wide-blocks, custom-colors, editor-style, translation-ready
*/
```

Rules:

- `Template: nextora` is what makes it a child. **[parent contract]**
- `Text Domain` must equal the slug; child PHP and JS never use the parent's
  domain. **[parent contract]**
- `Version` must equal `<CHILD_PREFIX>_VERSION`. The sample asserts this in
  `tests/ThemeHeaderTest.php` (`test_version_constant_matches_the_header()`), along
  with `Template`, `Text Domain`, `Requires at least`, `Requires PHP` and a
  "General Public License" licence string. Reusing that test shape is cheap and
  catches the release error the header alone cannot. **[sample convention]**
- Declare actual minima. The parent currently declares `Requires at least: 6.4`,
  `Tested up to: 7.0`, `Requires PHP: 8.1`, `Version: 0.0.22`; the sample declares
  the same WordPress minimum and `Tested up to: 7.0`. Those are **declarations,
  not proof** — the sources use `theme.json` v3 and modern package APIs, so
  quality gate 7 applies. **[parent contract + sample convention]**
- **[new-output requirement]** Do not copy a `Tested up to` value into a new
  child. Either omit it until the child has been exercised against that version,
  or set it after the runtime proof is recorded.
- The root stylesheet carries the header **only**. The sample's trailing comment
  is commented-out guidance, not a rule. **[sample convention]**
- Licence: the Nextora-compatible convention is GPL-2.0-or-later with the
  `https://www.gnu.org/licenses/gpl-2.0.html` URI. Do not invent a licence for
  the child, and do not add a licence to this skills repository on that basis.

## 4. The version constant

`inc/bootstrap/version.php`, required first by `functions.php`:

```php
<?php

declare( strict_types=1 );

if ( ! defined( 'ABSPATH' ) ) {
	exit;
}

if ( ! defined( 'NEXTFOLIO_VERSION' ) ) {
	define( 'NEXTFOLIO_VERSION', '1.0.0' );
}
```

It is read by the asset version fallback and by tests, so it must exist before
`inc/assets/enqueue.php` is required. **[sample convention]**

## 5. `functions.php` bootstrap

```php
<?php
/**
 * Nextfolio child theme bootstrap.
 *
 * WordPress loads a child's functions.php before the parent's, so nothing at file
 * scope here may read a NEXTORA_* constant or call a parent function.
 *
 * @package Nextfolio
 */

declare( strict_types=1 );

if ( ! defined( 'ABSPATH' ) ) {
	exit;
}

require_once __DIR__ . '/inc/bootstrap/version.php';

if ( ! defined( 'NEXTFOLIO_DIR' ) ) {
	define( 'NEXTFOLIO_DIR', get_stylesheet_directory() );
}

if ( ! defined( 'NEXTFOLIO_URI' ) ) {
	define( 'NEXTFOLIO_URI', get_stylesheet_directory_uri() );
}

$nextfolio_autoloader = NEXTFOLIO_DIR . '/vendor/autoload.php';

if ( is_readable( $nextfolio_autoloader ) ) {
	require_once $nextfolio_autoloader;
}

require_once NEXTFOLIO_DIR . '/inc/assets/enqueue.php';

// Add only the feature files this child actually ships.
// require_once NEXTFOLIO_DIR . '/inc/navigation/setup.php';
// require_once NEXTFOLIO_DIR . '/blocks/blocks.php';
```

Load order, and what follows from it:

1. The child's `functions.php` runs **before** the parent's. Core functions and
   `add_action`/`add_filter` exist; `nextora_*` functions and `NEXTORA_*`
   constants do not. **[parent contract]**
2. Therefore define the child's own paths and constants here, and reference
   parent handles **by name inside hooks** — never `NEXTORA_URI`, never
   `nextora_some_function()` at file scope.
3. Constants are wrapped in `! defined()` so a test bootstrap can pre-define them.
4. The child autoloader is required only when readable. The sample declares a
   PSR-4 `Nextsaas\` → `inc/` mapping, but its theme code is global prefixed
   functions; the mapping existing is not a reason to invent classes.
   **[sample convention]**
5. To run *after* a parent `after_setup_theme` callback, use a higher priority —
   the sample registers its nav menus at priority `11` for exactly that reason.
   Same-priority callbacks added earlier run earlier, and the child added its
   hooks first. The parent registers its own `primary` and `footer` locations from
   the same hook's default priority `10` (`inc/setup/theme-support.php`), so a
   child re-registering `primary` at `11` merges with it rather than replacing it;
   the sample also exposes the location through a filter
   (`nextsaas_primary_menu_location()`).
6. ABSPATH guard, `strict_types=1`, one prefixed namespace of global functions.

## 6. Asset pipeline (`inc/assets/enqueue.php`)

**[sample convention]** derived from `nextsaas_asset_version()`,
`nextsaas_enqueue_styles()`, `nextsaas_print_js_marker()`,
`nextsaas_enqueue_scripts()` and `nextsaas_add_editor_canvas_styles()`.

```php
<?php

declare( strict_types=1 );

if ( ! defined( 'ABSPATH' ) ) {
	exit;
}

/**
 * Development gets the file mtime; every other environment gets the released
 * constant, which is what the package artefact compares against.
 */
function nextfolio_asset_version( string $relative_path ): string {
	$path = NEXTFOLIO_DIR . $relative_path;

	if ( defined( 'WP_DEBUG' ) && WP_DEBUG && is_readable( $path ) ) {
		$mtime = filemtime( $path );

		if ( false !== $mtime ) {
			return (string) $mtime;
		}
	}

	return NEXTFOLIO_VERSION;
}

function nextfolio_enqueue_styles(): void {
	$relative_path = '/assets/css/nextfolio.css';

	if ( ! is_readable( NEXTFOLIO_DIR . $relative_path ) ) {
		return;
	}

	$deps = array();

	if ( wp_style_is( 'global-styles', 'registered' ) ) {
		$deps[] = 'global-styles';
	}

	if ( wp_style_is( 'nextora-app', 'registered' ) || wp_style_is( 'nextora-app', 'enqueued' ) ) {
		$deps[] = 'nextora-app';
	}

	wp_enqueue_style(
		'nextfolio-app',
		NEXTFOLIO_URI . $relative_path,
		$deps,
		nextfolio_asset_version( $relative_path ),
	);
}
add_action( 'wp_enqueue_scripts', 'nextfolio_enqueue_styles', 25 );

function nextfolio_print_js_marker(): void {
	wp_print_inline_script_tag(
		"document.documentElement.classList.add('nextfolio-js');",
	);
}
add_action( 'wp_head', 'nextfolio_print_js_marker', 1 );

function nextfolio_enqueue_scripts(): void {
	$relative_path = '/assets/js/nextfolio.js';

	if ( ! is_readable( NEXTFOLIO_DIR . $relative_path ) ) {
		return;
	}

	$deps = array();

	if ( wp_script_is( 'nextora-main', 'registered' ) || wp_script_is( 'nextora-main', 'enqueued' ) ) {
		$deps[] = 'nextora-main';
	}

	wp_enqueue_script(
		'nextfolio-main',
		NEXTFOLIO_URI . $relative_path,
		$deps,
		nextfolio_asset_version( $relative_path ),
		array(
			'in_footer' => true,
			'strategy'  => 'defer',
		),
	);
}
add_action( 'wp_enqueue_scripts', 'nextfolio_enqueue_scripts', 30 );

function nextfolio_add_editor_canvas_styles(): void {
	if ( is_readable( NEXTFOLIO_DIR . '/assets/css/nextfolio.css' ) ) {
		add_editor_style( 'assets/css/nextfolio.css' );
	}

	if ( is_readable( NEXTFOLIO_DIR . '/assets/css/nextfolio-editor.css' ) ) {
		add_editor_style( 'assets/css/nextfolio-editor.css' );
	}
}
add_action( 'after_setup_theme', 'nextfolio_add_editor_canvas_styles', 20 );
```

| Hook | Priority | Callback | Purpose |
| --- | --- | --- | --- |
| `wp_enqueue_scripts` | 25 | `*_enqueue_styles` | front-end sheet after `nextora-app` |
| `wp_head` | 1 | `*_print_js_marker` | `html.<slug>-js` before first paint |
| `wp_enqueue_scripts` | 30 | `*_enqueue_scripts` | child script after `nextora-main` |
| `after_setup_theme` | 20 | `*_add_editor_canvas_styles` | editor canvas CSS |
| `enqueue_block_editor_assets` | — | `*_enqueue_editor_styles` — registered in `blocks/blocks.php`, **not** in `enqueue.php`, because it only matters once the child ships blocks | inspector sheet (`<slug>-editor`) |

Why these choices:

- The parent registers its own stylesheet on `wp_enqueue_scripts` at the default
  priority `10` and its script at priority `20` (`inc/assets/assets.php`), so the
  child's `25` and `30` land after both; `global-styles` is enqueued by the parent
  at priority `1`, so the style dependency exists. **[parent contract]**
- The dependency is added only when the handle is registered/enqueued, so the
  child still loads if the parent changes its own gating. **[sample convention]**
- The front-end sheet is deliberately **not** on `enqueue_block_assets`: that hook
  also runs in `wp-admin`, where a bare `body` reset from the child stylesheet
  would repaint the whole editor chrome. The canvas receives the same file through
  `add_editor_style()`, inside the editor iframe. **[sample convention,
  see quality gate 2]**
- `add_editor_style()` is **necessary but not sufficient** for editor/front-end
  parity. Core prints the editor stylesheet links ahead of the `global-styles`
  inline block in the canvas, so a child rule whose specificity ties a core layout
  rule wins on the front end and loses in the editor; the same declaration then
  computes two different values (measured in one real child: a 720px rule at 720px
  on the front end and 540.96px in the canvas). Where a child rule deliberately
  override core layout — content widths, block margins, control sizing — raise its
  specificity deliberately and measure the computed value in **both** contexts.
  Quality gate 2 is a measurement, not an enqueue. Know what you are beating: the
  competing rule for a paragraph that is a direct child of the post content is
  core's editor-only `.block-editor-block-list__layout.is-root-container >
  :where(:not(.alignleft):not(.alignright):not(.alignfull))` at 0-2-0, so an
  element-qualified class pair (`p.<slug>-lede.wp-block-paragraph`, 0-2-1) wins in
  both contexts while a bare class rule ties and loses in the editor. Reach for the
  smallest bump that measures equal in both; `!important` is not the remedy, and
  deleting the measure is not either.
- `add_editor_style()` resolves against the **stylesheet** directory, so a child
  file named `assets/css/app.css` would replace the parent's canvas stylesheet.
  Never use `app.css` or `main.js` as a child output name; the sample's
  `scripts/build-ts.mjs` says so in a comment and writes `assets/js/nextsaas.js`.
- The JS marker is printed in `wp_head`: a deferred bundle runs after first paint,
  and a menu that collapses only for `html.<slug>-js` would visibly flash
  otherwise. Progressively enhance; a no-JS visitor must keep full navigation.
- A readable-file guard is a *visibility* measure, not success: if the guard
  returns early during verification, the build did not produce the file. Require
  a successful build and confirm the real request output.

## 7. Block registrar (`blocks/blocks.php`)

Only when the child ships blocks. WordPress does not scan a theme's `blocks/`
folder. **[sample convention + parent contract]**

```php
<?php

declare( strict_types=1 );

if ( ! defined( 'ABSPATH' ) ) {
	exit;
}

function nextfolio_register_blocks(): void {
	$blocks_dir = get_stylesheet_directory() . '/blocks';

	if ( ! is_dir( $blocks_dir ) ) {
		return;
	}

	$block_dirs = glob( $blocks_dir . '/*', GLOB_ONLYDIR );

	if ( false === $block_dirs ) {
		return;
	}

	sort( $block_dirs );

	foreach ( $block_dirs as $block_dir ) {
		if ( ! is_readable( $block_dir . '/block.json' ) ) {
			continue;
		}

		register_block_type( $block_dir );

		$register_editor = $block_dir . '/register-editor.php';

		if ( is_readable( $register_editor ) ) {
			require_once $register_editor;
		}
	}
}
add_action( 'init', 'nextfolio_register_blocks' );
```

Then the inserter category and the two coercion layers that keep a child block in
that category:

- `block_categories_all` → unshift
  `array( 'slug' => 'nextfolio', 'title' => __( 'Nextfolio Blocks', 'nextfolio' ), 'icon' => 'art' )`
  only if absent. The parent uses slug `nextora`, title "Nextora Blocks",
  icon `layout`; the sample uses `nextsaas` / "NextSaaS Sections" / `art`.
- `register_block_type_args` (priority 10, 2 args) → force `category` for every
  name starting with `nextfolio/`.
- `enqueue_block_editor_assets` priority 5 → `wp_add_inline_script( 'wp-blocks', … , 'after' )`
  applying the same coercion through `wp.hooks.addFilter('blocks.registerBlockType', …)`,
  because `block.json` values are applied client-side too.

The child's `block.json` follows the same field set the parent and sample use:
`apiVersion: 3`, `name: "nextfolio/<slug>"`, `category: "nextfolio"`,
`textdomain: "nextfolio"`, `editorScript: "file:./index.js"`,
`editorStyle: "file:./editor.css"`, `render: "file:./render.php"`, and
`viewScript: "file:./view.js"` when the block has front-end behaviour. Metadata
must never point at TypeScript. Editor UI, `save` semantics and view scripts are
covered by `../../nextora-gutenberg-blocks/SKILL.md`.

Build outputs: the sample's `scripts/build-blocks.mjs` globs
`./blocks/*/index.{ts,tsx}` and `./blocks/*/view.{ts,tsx}`, writes `index.js` +
`index.asset.php` (with a fixed WordPress handle list) and `view.js` +
`view.asset.php`. Do not generalise that to the parent: the parent's view build
emits no `view.asset.php`, and its `index.asset.php` version comes from
`Date.now()`, so a parent rebuild is not a deterministic diff. The registrar needs
those built files; they are generated, not editable source. **[sample convention]**

Failure surfacing: if `blocks/` exists but nothing registers, do not accept an
empty inserter. Check that each `block.json` is readable, that the bundle and its
`index.asset.php` were produced, and that the category filter actually ran.

## 8. `theme.json` for a child

- `version: 3` is what both sources ship. Treat the declared WordPress minimum as
  a declaration until checked (quality gate 7).
- Extend the parent palette/fonts **additively**: do not rename or re-point an
  inherited slug that existing content references, and read the replacement rule
  below before declaring a preset array at all. Parent palette slugs are `base`, `contrast`, `primary`,
  `paragraph`, `secondary`, `surface`, `transparent`; font slugs include `sans`,
  `mono`, `hanken-grotesk`, `josefin-sans`, `noto-serif-display`,
  `jetbrains-mono`. **[parent contract]**
- A shipping child font needs a `fontFamilies` entry with `fontFace` `src`
  arrays (`file:./assets/fonts/...`) and a licence file alongside — see the
  sample's `bodoni-moda` / `dm-sans` entries and `assets/licenses/*OFL.txt`.
- `templateParts` and `customTemplates`: declare what the child owns. The sample
  re-declares the parent's `about-us`, `page-full-width`, `contact-us`,
  `theme-terms-conditions`, `theme-privacy-policy`, `frequently-asked-questions`
  names, and also core names (`front-page`, `archive`, `single`, `search`, `404`),
  while shipping files for none of the first six. **[observed defect]** For new
  work, declare only names that are both non-core and have a file in the child's
  own `templates/` directory.

- **Declaring a preset array replaces the parent's array; it does not extend it.**
  `settings.color.palette`, `settings.typography.fontFamilies`,
  `settings.typography.fontSizes`, `settings.spacing.spacingSizes` and
  `settings.color.gradients` are sequential arrays, and core's theme.json merge
  replaces sequential arrays by numeric index
  (`wp-includes/class-wp-theme-json.php`). A child that declares
  `settings.color.palette` with one new slug therefore drops **all seven** inherited
  colours, and a child that declares `fontFamilies` drops all six inherited families
  and every preset, role and variation that resolves through them. So: list every
  inherited entry the child still needs in its own file, or omit the key entirely to
  leave the parent's list untouched — and assert the merged result (a
  `wp_theme_json_data_theme`-driven test, or a REST `theme.json` fetch) instead of
  assuming a merge. Never ship a child palette without checking that an inherited
  block still resolves `var(--wp--preset--color--…)` and a role font.

Colour roles, presets and the visitor switcher are owned by
`../../nextora-design-system/SKILL.md`; a child contributes preset data files, never
a second switcher.

## 9. PHP and i18n conventions

- `declare( strict_types=1 );` and an `ABSPATH` guard in every PHP file.
- Global functions prefixed `<child_prefix>_`; no unprefixed globals, no
  parent-prefixed names.
- Escape by context on output: `esc_html()`, `esc_attr()`, `esc_url()`,
  `wp_kses_post()` or a narrower allowlist for rich HTML.
- Every user-visible string goes through `__()`, `esc_html__()`, `_n()`,
  `_x()` with the **child** text domain. Scripts that carry translatable
  literals are localized (`wp_localize_script()`) or registered with
  `wp_set_script_translations()`.
- Avoid the reserved block-support attribute names for inner-surface styling;
  inner-element overrides use names such as `surfaceBackgroundColor`.
- PHP 8.1+ syntax is allowed, matching the declared minimum; do not raise the
  declared minimum silently to use a newer feature.

## 10. CSS surface

- Author in `resources/css/`, compile to `assets/css/<slug>.css` and
  `assets/css/<slug>-editor.css`. The sample's `postcss.config.mjs` runs
  Tailwind plus a `scopeRoots()` plugin; `npm run build:css` builds both entries.
- Keep the front-end sheet free of `wp-admin` selectors. Resets belong to the
  editor canvas and the front end, never the admin chrome.
- If the child defines design tokens, derive semantic tokens from parent roles
  (`var(--wp--preset--color--…)`) and keep any fixed token deliberate.

## 11. When the child is a port of the Nextsaas design

These are sample-design contracts, **not** requirements for every new child:

- `.nextsaas-scope` sits on the main wrapper and on every section root, and
  `scopeRoots()` additionally rewrites `.nextsaas-scope .section` to
  `.nextsaas-scope.section` for the root classes listed in
  `postcss.config.mjs` (a closed set — a new section class must be added there).
- `inner-page` is a **body** class added by `inc/template/body-class.php` for
  every request that is not the front page; the PostCSS plugin strips the scope
  from `.nextsaas-scope .inner-page` because the body can never be inside it.
- `resources/css/modules/00-tokens.css` pins sample fonts directly
  (`--nextsaas-font-display` → the `bodoni-moda` preset) and aliases the two
  child-only entries it uses, `border` and `accent-soft` (`mist` and `cobalt` are
  `theme.json` palette entries only and are not aliased in that file). That is a fixed
  design, **not** a switchable model: a new child that must follow visitor
  switching resolves through the `--nextora-font-*` roles and parent palette
  slugs instead.

A new child uses its own scope class/prefix and adds surface-specific body
classes only when its own CSS requires them.

## 12. Verification (downstream acceptance recipe)

Not run by this suite. Prerequisites: the child's own directory, its npm and
Composer dependencies installed as its `README` states, a **disposable**
WordPress site with Nextora and the child activated, and working admin access.

Run only commands the child's `package.json` declares. The sample has `build`,
`build:css`, `build:ts`, `build:blocks`, `typecheck`, `lint:php:all`, `test:wp`
and **no** `gen`. `.nvmrc` says `24`. `test:wp` is a server-side workflow — read
`scripts/wp-test.sh` and `blueprint.json` before promising a port, login or
browser proof.

1. Build, then load a front-end page. Confirm in the network panel that
   `assets/css/<slug>.css` and `assets/js/<slug>.js` are requested, and that the
   stylesheet loads **after** the parent's `nextora-app`.
2. Confirm the page still renders correctly with JavaScript disabled, and that
   the child's JS marker class is what the collapsed-navigation rules depend on.
3. Open the block editor and a page's canvas: the canvas carries the child
   stylesheet, and `wp-admin` chrome is unchanged.
4. Change a site style variation in the Site Editor and confirm the front end
   follows it — that is the persistent path, and it is the design-system proof.
5. If blocks ship, confirm each block appears under the child category, inserts,
   saves, reloads without a validation warning, and renders on the front end.
6. Bump `Version` in `style.css` and `NEXTFOLIO_VERSION` together; a header test
   (or a manual diff) must show them equal.
7. Check the requested minimum WordPress version **and** the current supported
   release before claiming compatibility.

If runtime is unavailable, say so plainly: the child's browser behaviour is
unverified. A header/bookkeeping check is not a substitute.
