# Portfolio field test — do the Nextora skills guide a real delivery?

A real project was run end to end against the **frozen** skill suite: a Nextora child theme portfolio for
a fictional Product Designer / Creative Developer, built by a separate agent, reviewed by a second agent,
accepted by a third, with the coordinator (this author) owning scope, acceptance and skill corrections.

This is an **observational field test**, not a causal A/B experiment. The builder ran with its own global
instructions and its own model; only the evidence below is claimed.

Status: **complete**. Acceptance ran against candidate v6 (theme digest `33053e3c…`, 72 files), and the
delivered tree was then extended to **v7** (`9f80cabe…`, 80 files) by a follow-up block-capability probe
that added the theme's first child block and its registrar; the matrix is §7, the skill verdicts §8 and the
overall verdict §11. `$RUN` below is `/tmp/nextora-portfolio-field-test-20261102T150910Z`.

## 1. What was frozen, and what ran

| Item | Value |
| --- | --- |
| Skill baseline (authority for the delivery run) | copy of `skills/` at `$RUN/baseline/skills`, hashes `baseline/skills-baseline.sha256` (12 files, 5 skills + 7 references) |
| Parent source hash baseline | `baseline/parent-sample.sha256`, 4397 files (`nextora` + `nextsaas`, excluding VCS, `node_modules`, caches) |
| Coordinator | 9router/cmc/deepseek/deepseek-v4-flash |
| Builder | `omp`, `deepseek-v4-flash`, pre-existing pane `w4:p1` (renamed `portfolio-builder`, session preserved), cwd the empty target theme |
| QA | `omp`, pane `w3:p6`, created for this run |
| Reviewer | `omp`, pane `w3:p7`, created for this run |
| `omp` version | 18.13.1 (18.15.0 offered, not applied) |
| Runtime | node 24.8.0, npm 11.6.0, host php 8.4.7; `@wp-playground/cli` 3.1.62 |
| Live site | `dev.local` — WordPress 7.1.4, stylesheet `nextsaas`, 8 pages, occupied and never touched |

**Confounders, recorded not controlled:** the builder runs with the user's global model roles and MCP
config; the global instruction/skill set on this machine already contains the same five skills, so a
"skill-guided" behaviour cannot be separated from an installed-skill effect. One declared MCP server
(`open-design`) fails to start (`MCP subprocess closed stdout`) and was deliberately not fixed or relied
on. The live install's occupied state is why every runtime claim below comes from disposable Playground
instances.

## 2. Supplied prompt and read evidence

- Product brief: `$RUN/brief.md` (builder discovery) — product surfaces, naming/versioning table, palettes,
  font pairs, editorial visual brief, the seven-section first-response contract, "pause before writing".
- Later briefs, all referenced by absolute path: `qa-brief.md`, `review-brief.md`, `gate-1.md`,
  `qa-decisions.md`, `handoff-qa.md`, `handoff-review.md`, `gate-3.md`, `v2-signal-qa.md`,
  `delta-review-v2.md`, `rerun-skill-test.md`.
- The builder's first answer is preserved verbatim-ish at `baseline/builder-discovery-response.txt`; the
  stable handoff at `baseline/builder-handoff-1.txt`; the post-review fix report at
  `baseline/builder-gate3-report.txt`.

Files the builder cited as read in its discovery pass (from its own report, not verified line by line by
the lead): parent `style.css`, `functions.php`, `theme.json`, `blocks/blocks.php`, `blocks/post-grid/*`,
`blocks/hero-section/block.json`, `inc/` layout, `patterns/`, `templates/`; sample `theme.json`,
`scripts/wp-test.sh`, `blueprint.json`, `scripts/build-blocks.mjs`, `templates/index.html`. The reviewer
independently re-read the parent contracts it judged against, and the QA agent independently re-derived
the theme.json merge rule and the Playground version behaviour from core and CLI sources.

## 3. Team, communication and intervention ledger

Herdr topology: builder in its pre-existing pane `w4:t1`; QA and reviewer as background siblings in the
coordinator's tab (`w3:p6`, `w3:p7`); the primary WordPress instance in its own shell pane (`w3:p8` in tab
`w3:t2`); the skill-rerun agent in `w3:p9`. No workspace, worktree or tool update was created; no
pre-existing pane was closed.

Every handoff went through `herdr agent prompt … --wait` with the brief on disk. **Every** `--wait` call
returned `timeout` (turn still live) except the final acknowledgements; nothing was ever resent, and each
time the agent was inspected instead.

| # | Intervention | Class |
| --- | --- | --- |
| 1 | Custom post type rejected; hierarchical Pages + `core/query` `parents` required | skill omission (see §8 S1) |
| 2 | Home selected-work made curated rather than a date-ordered query | product requirement |
| 3 | Default font pair inverted to Hanken/Hanken | product requirement |
| 4 | Harness paths reassigned from builder to QA | plan hygiene |
| 5 | Hidden starter patterns required for About/Contact as well as Home/Work | product requirement |
| 6 | Build must not depend on optional host tooling (ImageMagick) | plan hygiene |
| 7 | Playground CLI pinned in `devDependencies` (plan requirement) | plan compliance |
| 8 | Page excerpt support added from child code (page ships without it) | real defect found by lead |
| 9 | QA self-inflicted script error; "internal subagent" reading corrected — it was a supervised process | process |
| 10 | QA harness accepted as the authority on which build the CLI actually serves | environment |

Interventions 1-8 are builder-affecting; 9-10 are team-side. The count is the headline number of the
field test: a strong first pass, then eight concrete corrections before the first screenshot.

## 4. Runtime and isolation

WordPress Playground only. Parent copied to `$RUN/runtime/nextora` (3916 files; VCS, `node_modules` and
caches excluded; built assets and `vendor` retained) and mounted **that copy**; the child mounted from its
real path; evidence directory mounted so captured artefacts are readable from the host. The live install,
its uploads and its database were never mounted or written. Parent and sample source hashes were re-checked
after delivery: unchanged (4397/4397).

Two runtime facts the plan got wrong and the run corrected, both from measurement rather than assumption:

- In `@wp-playground/cli` 3.1.62, `--wp=` / `--php=` do **not** select the served build; only the
  blueprint's `preferredVersions` does, and the CLI banner reports the flags rather than reality. The
  matrix cells therefore live in the blueprints, the flags are passed to match them, and
  `tests/smoke/assertions.php` fails hard unless the instance self-reports the expected versions.
  Evidence: `evidence/prep/runtime-cell-finding.md`.
- Node 24.8.0 / npm 11.6.0 sit below the CLI's declared `>=24.18.0` / `>=11.16.0`: `EBADENGINE` warnings
  only, no upgrade, recorded as an environment limitation.

### One unresolved isolation observation on the live site

The plan required `dev.local` to be left exactly as it was found. It was not: at the end of the run the
live install's `stylesheet` option reads `test-skills` (baseline, captured at 15:16Z: `nextsaas`) while
`template` is still `nextora`, and the page count went from 8 to 9 — page 27 "Home" created 16:00Z and
page 29 "Contact" created 16:09Z, both authored by user 1 (`adminj`), with page 27's content carrying this
child's case-study pattern markup. **No evidence ties that to this run**: every agent used Playground for
runtime work; the builder's session transcript contains no live-site write command (its only `wp --path`
and `theme activate` mentions are inside the sample's own `scripts/wp-test.sh` and `blueprint.json`, which
it read); QA's and the reviewer's reports record only read-only GETs against `dev.local`; and the lead's
own live-site commands were `option get` / `post list` reads. The change is therefore attributed to the
user or another session outside this project, and it was **not reverted** — the live database was never a
write target here, and reverting someone's activation is not this run's call. Also relevant: the child
theme directory and the Playground mount make the live install reference `test-skills` URLs even when the
parent is active, so a URL-scan alone cannot settle which theme is active; the option value is the
evidence. Parent and sample **sources** are byte-identical to the frozen baseline (`4397/4397`), so no
source cutover happened.

## 5. Candidate history

| Candidate | Contents | Manifest |
| --- | --- | --- |
| v1 | first complete child theme | `evidence/candidate-1.sha256`, 73 files, digest `97862358…` |
| v2 | v1 + F1-F6 from the independent review | `evidence/candidate-2-theme.sha256`, 71 theme files, digest `045f5501…`; `evidence/candidate-2-all.sha256`, 78 files with QA harness |
| v3 | v2 + the three delta-review items (chrome alignment, restored lede/footer measures with a measured specificity bump, corrected README variation passage) | `evidence/candidate-3-theme.sha256`, 71 theme files, digest `12dc272c…`; `evidence/candidate-3-all.sha256`, 78 files |
| v4 | v3 + gate-5 (README UI-vs-REST clarification, `Tested up to: 7.1.4` with the proof named, About pattern `&` cleanup, contract test updated) | `evidence/candidate-4-theme.sha256`, 71 theme files, digest `aa6cfc14…` |
| v6 | v4 + gate-6 (skip-link focus target, duplicate footer navigation removed, README variation passage corrected) | `evidence/candidate-6-theme.sha256`, 72 theme files, digest `33053e3c…` |
| v7 | v6 + the block-capability probe's block (`test-skills/fact-list`), the child registrar, the block build script, and the review findings closed | `evidence/candidate-7-theme.sha256`, 80 theme files, digest `9f80cabe…`; `evidence/candidate-7-all.sha256` |

The full browser matrix ran against **v6**, served live from the mounted target in the lead's pane and
re-seeded under `evidence/instance-primary-final`; the block probe then added the theme's first block and
was itself proven live on the same instance, giving **v7** as the delivered tree. v1's browser results were
invalidated rather than reported, because the F1 fix changes every page width; QA marked them under
`evidence/V1-INVALIDATED.md`, and the v2 attempts under `evidence/V2-PARTIAL.md`. Lead-run verification at
each freeze: `npm ci`, `npm run typecheck`, `npm run build`, `php -l` on every authored PHP file, plus
`composer phpstan` / `php-cs-fixer` / `phpunit` where they exist. Raw output:
`evidence/lead/build-verify.txt`, `build-verify-v2.txt`, `build-verify-v3.txt`, `build-verify-v4.txt`,
`build-verify-v7.txt`. The dashboard of lead-run browser observations is §7.

## 6. Build and ledger of what the delivered theme actually is

- 80 theme files: `style.css` (header only), `theme.json` v3, `functions.php`, `inc/{bootstrap,assets,patterns,setup,template}`,
  `resources/css` + 6 modules, `resources/ts/main.ts`, 5 hand-authored SVG sources, 8 templates, 2 parts,
  10 patterns, 5 WebP exports, **one child block** (`blocks/fact-list/`), the child block registrar
  (`blocks/blocks.php`), `README.md`, `CHANGELOG.md`, real npm/Composer tooling.
- Blocks: **one**, added after acceptance by the block-capability probe below — a dynamic, Gutenberg-native
  block (`apiVersion: 3`, `save: () => null`, `render.php`), registered by a child registrar that reads the
  stylesheet directory, skips an unreadable `block.json` and forces the `test-skills` category in PHP and
  through the client-side coercion. No second React runtime: the child has no React dependency, so the
  editor entry uses `createElement` and the build shims every `@wordpress/*` import onto `window.wp.*`.
- Build: `npm run build` = PostCSS + esbuild (script) + `node scripts/build-blocks.mjs` →
  `assets/css/test-skills.css`, `assets/js/test-skills.js`, `blocks/fact-list/{index.js,index.asset.php}`;
  `build:images` stays separate and guarded.
- Assets: style handle `test-skills-app` (deps `global-styles`, `nextora-app`, priority 25), script
  `test-skills-main` (dep `nextora-main`, priority 30, deferred), `wp_head` marker at priority 1, editor
  canvas through `add_editor_style()` only.

## 7. Verification matrix

Filled from `$RUN/qa-report.md` (QA, acceptance run), the reviewer's sections in `$RUN/review-report.md`,
and the lead's own browser observations. Every runtime value comes from the instance's own self-report or a
browser measurement; Playground cells are pinned by the blueprint's `preferredVersions`, never by the CLI
banner.

| Cell | Instance | Self-reported | Server-side assertions |
| --- | --- | --- | --- |
| primary | 9420, lead's pane, re-seeded | WP 7.1.4 / PHP 8.3.36, switcher gate on | 87 checks / 0 failures |
| minimum | 9423, QA | WP **6.6.11** / PHP **8.1.34**, gate off | 87 / 0 |
| bare (ordinary activation) | 9422, QA | WP 7.1.4 / PHP 8.3.36, gate off | 68 / 0 |
| anonymous (contact path) | 9429, QA, blueprint `login:false` | WP 7.1.4 / PHP 8.3.36, gate on | seeded, 87 / 0 |
| capture-disabled (honest failure) | 9427, QA | WP 7.1.4 / PHP 8.3.36 | n/a |

| Scenario | primary | minimum | bare | anonymous |
| --- | --- | --- | --- | --- |
| 1 browse/contact journey, 404, search, publish/unpublish | **pass** | **pass** (routes) | n/a | **pass** (skip link, keyboard order) |
| 2 Page editing is real | partial — hero edit with bold/link/cleared paragraph matched the front end with no invalid-block warning, undo/redo and restore verified; image replace via toolbar and publishing the draft pattern page not exercised | not exercised | n/a | n/a |
| 3 inherited team block behaviour | **pass** — one modal, Escape leaves the immediate patch in place, add/reorder/bounds/minimum/undo correct, save→reload→re-save matched the front end, collaborator photo/alt edited and rendered; serialized empty list renders safely (front-end half) | not exercised | n/a | n/a |
| 4 persistent vs visitor design | partial — visitor path verified on computed styles and persistence; applying a Site Editor variation was not exercised | computed palette and font roles pass | switcher absence asserted only | n/a |
| 5 contact truthfulness | **parent defect P4** (logged-in visitor cannot submit) | n/a | n/a | **pass** — valid submit 200 with intercepted admin+visitor messages, wrong nonce 403, empty submit rejected; capture-disabled shows an honest error |
| 6 responsive/a11y/resilience | partial — no overflow at 1440/768/390/320 and 200% zoom, contrast 8.0:1 paper / 11.3:1 ink, reduced motion honoured, no failed asset requests; no-JS and anonymous keyboard verified on the anonymous cell | not exercised | partial (admin chrome) | **pass** — no-JS renders cards/H1/nav with no marker and no false success |
| 7 isolation and reproducibility | partial — parent/sample unchanged, ordinary activation creates nothing, fresh boot usable; a persisted-DB seed re-run was not exercised | n/a | partial | n/a |

Lead-run browser observations on the live primary (independent of both workers):

| Check | Result |
| --- | --- |
| Front page | HTTP 200; body `wp-theme-nextora wp-child-theme-test-skills test-skills-home`; one `<main>`, one `<h1>` "Alex Morgan"; child stylesheet after `nextora app.css`; child script after `nextora main.js`, deferred; three theme WebP images; no donor copy |
| Paper palette (computed, v1 and v2) | body `rgb(246,244,238)`, text `rgb(75,75,67)`, `--wp--preset--color--primary #315C3B`, both `--nextora-font-*` roles `"Hanken Grotesk", sans-serif` |
| Visitor switch | `?color=portfolio-ink&font=portfolio-editorial` on clean storage: body `rgb(21,25,22)`, text `rgb(203,208,196)`, heading `"Noto Serif Display", serif`, roles inline on `<html>`, preference persisted |
| `/work/` listing | three cards linking `/work/fieldnotes/`, `/work/forma/`, `/work/atlas/`, one `<h1>`; v2 widths 1200px listing / 384px cards inside a 1264px shell |
| Case study | `/work/forma/` one `<h1>`, prose paragraphs 720px |
| Chrome alignment (v3+) | shell and header inner both at left 88px with a 1200px content box; lede `max-width` 540.96px; footer brand 419px; hero role 610px |
| Journeys (final) | Home → Work → Forma → About → Contact each render exactly one `<main>` and one `<h1>` with a working skip link and the three navigation links; unknown URL renders the 404 page with recovery links; `?s=Forma` and a no-match query both render a results surface |
| About copy after the harness fix | "Motion & prototyping" and "Content strategy & editorial", zero `u0026` on the page |
| Skip-link target (v6) | live markup carries `<main tabindex="-1" id="content">` |
| Block registered (v7) | editor on the live instance lists 156 block types, including `test-skills/fact-list`, with the `test-skills` category present and no editor notice |
| Block round trip (v7) | inserted into a fresh page, attributes edited, saved; reopening the editor parses the block back with `heading`, `layout: stacked`, `dividers: true` and both rows intact and **no invalid-block notice**; the published page renders `<dl class="test-skills-fact-list__facts has-dividers">` with `<dt>` labels, `<dd>` values, the `<h3>` heading and `data-layout="stacked"` |
| Block build (v7) | `npm run build` builds the block too (`blocks/fact-list/index.js` 5,748 B + `index.asset.php` declaring the 14-handle list); the bundle contains zero `require("@wordpress…")` and reads only `window.wp.{blocks,blockEditor,components,element,i18n}` |

Not exercised, with reasons, in `$RUN/qa-report.md`: applying and persisting a Site Editor variation and
the Reset control; image replace/alt through the block toolbar; publishing the pattern draft; the team
block's empty state *inside* the editor; the persisted-database seed re-run; link/button contrast; wp-admin
chrome in v3+; the minimum cell's editor legs; and the visual quality of the five images (the vision model
was unavailable, so they were verified by dimension and ink coverage only).

## 8. Skill assessment

Per skill, against the frozen baseline. Verdicts are the *observed* consequence for this project.

| Skill | Verdict for this project | Evidence |
| --- | --- | --- |
| `nextora-ecosystem` | Useful | routing was correct (child + design system + composition + inherited-block work), the ordered workflow and read-locally-first rule produced a genuinely source-grounded first answer, and the parent/sample lines it quotes were re-verified. It did not prevent the architecture error, because that error was outside every specialist's written guidance (§8 S1). |
| `nextora-child-themes` | Useful, one misleading claim | the exact hook table (25 / 30 / `wp_head` 1 / `add_editor_style()` 20), the non-colliding basename rule, the version-constant order and the "nothing at child file scope may touch the parent" rule were all followed and all survived review. The claim that `add_editor_style()` delivers parity is too strong (S5). |
| `nextora-design-system` | Useful, one missing trap | the mechanism table, the `:root` role-default contract, the data-not-code rule for the switcher and the gate description produced a child whose visitor path works on first measurement and whose compiled CSS holds **zero** literal colours. It does not state that preset data files under `styles/**` are also registered as Site Editor variations (S4), nor that declaring `settings.color.palette` replaces the parent's array (S2, in the child skill). |
| `nextora-site-composition` | Useful and incomplete | the part-ownership rule, the pattern-header rules, the `inherit:true` query rule and the "one authority" rule were all applied correctly. The recipe only covers the theme-owned-layout model, so the brief's editability requirement had no documented answer and the builder invented a CPT (S1). |
| `nextora-gutenberg-blocks` | Useful, with three misses only a real build exposed | the inherited-block semantics it documents (ID-keyed members, immediate `patchMember`, one modal, no rollback, minimum one) matched the real block and kept the child from inventing a rollback contract or a team-block clone. A separate time-boxed probe then built the theme's first **new** block against the frozen skill (§8.1): the contracts that carry the most risk landed correctly on the first attempt — `apiVersion: 3` metadata, dynamic `save: () => null` with `render.php`, escaping by context with the child textdomain, a registrar that reads the stylesheet directory, and the `window.wp` handle trap (zero unshimmed `require`, only the five globals its bundle needs, all inside the contract's 14-handle list). What it missed: the skill's "state the reuse decision" requirement was skipped, the editor's list mutations were announced to nothing (no live region), and the block build was not wired into the theme's own `npm run build`, so a clean build left `block.json` pointing at a missing file. |
| `references/quality-gates.md` §5 | Partially unusable as written | the i18n gate cannot be satisfied by a block theme's template/pattern copy and names no mechanism; it produced one medium review finding and a lead ruling (S3). |

### Skill defects corrected

Each had an exact baseline quote and a concrete consequence:

| ID | Baseline text | Consequence in this run | Patch |
| --- | --- | --- | --- |
| S1 | `templates-and-patterns.md` §5: "A new full-page composition has exactly one authority: a hidden filesystem pattern, referenced from a thin template." | The brief required owner-editable page content; the recipe leaves page content empty, so the builder chose a custom post type with a theme-switch data-loss risk it recorded itself — and the lead had to overrule it (intervention 1). | New §5.1 "the second whole-page model": thin template + `core/post-content` + hidden **starter** pattern, with the trade-off, the rules and "choose from what the brief says the owner must change". |
| S2 | `child-contract.md` §8: "Extend the parent palette/fonts; do not replace inherited slugs that existing content references." | Directionally right, mechanically wrong: declaring a preset array **replaces** the parent's array, so a child that adds one slug silently drops seven colours and six families. The child happened to re-list all seven, but only because the builder knew the core behaviour. | Added the replacement rule, the affected keys, and "list every inherited entry or omit the key; assert the merged result". |
| S3 | `quality-gates.md` §5: "Every user-visible string goes through `__()` … with the **target theme's** text domain". | Unachievable for copy that ships inside `templates/*.html` and pattern bodies, with no mechanism named; the theme passed review while shipping untranslatable copy (medium finding), and the lead had to rule. | Split code strings from starter content: code paths are covered in full; template/pattern markup is starter content, state it explicitly, and never rewrite HTML templates into PHP to wrap copy. |
| S4 | `presets-and-switching.md` §6: `styles/font-preset/<name>.json` described as switcher data only. | Core registers **every** JSON under `styles/**` as a Site Editor variation, so preset data also lands in the persistent list — an unavoidable side effect that was undocumented and caused a review disagreement. | Added the side-effect note with the resolver source, "give every file a distinct title", keep the allowlists tight, and "do not invent a second directory layout just to hide a variation". |
| S5 | `child-contract.md` §6: parity asserted from `add_editor_style()` alone. | Core prints editor styles ahead of the `global-styles` inline block, so a child rule at equal specificity won on the front end and lost in the canvas (measured 720px vs 540.96px) — the child did exactly what the skill said and still failed the gate. | Added "necessary but not sufficient": name the competing rule and its weight (0-2-0 for core's editor root-children rule), the smallest bump that measures equal in both (an element-qualified class pair, 0-2-1), and that neither `!important` nor deleting the measure is the remedy. |
| S6 | `templates-and-patterns.md` §5.1 (new) / §4: nothing said about writing a pattern's registered content into a page from code. | The project's demo setup inserted a registered pattern with `wp_insert_post( array( 'post_content' => $pattern_content ) )`. Core registers a pattern through `serialize_block_attributes()`, which encodes `&` as `\u0026`; `wp_insert_post()` unslashes, so the backslash was stripped and `/about/` rendered "Motion u0026 prototyping" for three candidate rounds. The lead first misclassified this as a theme defect and asked for a pattern edit that fixed nothing; QA's fixture was the actual cause. | Added a paragraph to §5.1: read the content from the registry and `wp_slash()` it before storing, the same for later content patches, and verify by reading the stored content back and rendering the page. |

### Parent defects found (reported, never fixed)

The child degrades gracefully around these; none was worked around with a shim.

| ID | Defect | Minimal reproduction | How the child copes |
| --- | --- | --- | --- |
| P1 | `nextora/contact-form`'s editor preview is hand-built and differs from its own renderer (`blocks/contact-form/edit.tsx:314` vs `render.php:268`), so consumer styling of the rendered classes is invisible in the canvas. | Insert the block; the canvas shows `__fields` with faux `span[role=textbox]` fields while the front end renders `__form` with real inputs. | Child-only canvas-scoped rules mirror the preview markup; the residual difference is documented in the child README. |
| P2 | A member photo supplied by `photoUrl` with `photoId: 0` has no editable alt: the field is guarded by `photoId > 0` (`blocks/team-section/member-edit-form.tsx:100`). | Set a member photo by URL only; the editor shows the photo but no alt field. | The theme's About starter ships `photoUrl` as a fallback and the demo seed assigns real attachment IDs so the field exists; documented. |
| P3 | The parent's own header/footer parts ship donor copy and placeholder links (`parts/header.html` marquee "Join 50,000+ supporters worldwide", `support@gmail.com`; `parts/footer.html` `href="#"` policy links). | Render the parent theme's header/footer. | The child owns both parts and composes them from core blocks — recorded as deviation 6 with this reproduction. |
| P4 | The parent's contact form cannot be submitted by a **logged-in** visitor: the client posts only `_wpnonce` from `form.dataset.nonce` and never an `X-WP-Nonce` header (`blocks/contact-form/view.ts:384-400`), while WordPress treats a cookie-authenticated REST request without that header as anonymous — so a nonce minted for the logged-in user can never verify (403 `invalid_nonce`). | Log in, open the front-end contact page, submit a valid message: 403. | Not fixed and not shimmed; QA re-ran the journey on a no-login instance and the logged-in 403 is retained as this reproduction. The child adds no nonce workaround for a parent defect. |
| P5 | The team block's canvas preview renders `alt=""` for URL-sourced member photos while the front end renders the stored alt. | Select the team block in the canvas and inspect the preview image. | None possible child-side; recorded as editor parity drift belonging to the parent block. |

Also recorded from the review, not patched as skill text: the child's handling of `Tested up to` matched
gate 7 exactly — left absent until the primary matrix proved 7.1.4, then set to it with the proof named in
the README, and never copied from the parent's unproven `7.0`.

### 8.1 Block-capability probe (the one path the project itself never touched)

The delivery shipped no block, so the skill's authoring path was untested by it. After acceptance, a
time-boxed probe (two Herdr agents: one builder, one reviewer, both started from the frozen skill) built
`test-skills/fact-list` — a dynamic case-study "facts" strip — in a throwaway clone, then moved it into the
delivered theme with the review findings closed.

| Item | Result |
| --- | --- |
| Reviewer verdicts on the first pass | 6 pass (metadata, `save`, `window.wp` handles, `render.php`, registrar, styles), 2 partial (no live region for list mutations; `build:blocks` missing from `npm run build`), 1 fail (reuse decision never stated; `core/table` never addressed) |
| Lead verification (independent) | built artefacts present and regenerable, `index.asset.php` declares the contract's 14 handles, bundle has zero `require("@wordpress…")`, registrar uses the stylesheet directory and skips unreadable metadata |
| Runtime proof on the live instance | block registered in the editor (156 block types), inserted, saved, **reloaded with attributes intact and no invalid-block notice**, and the published page rendered `<dl>`/`<dt>`/`<dd>` with `has-dividers` and `data-layout="stacked"` |
| After the fixes | reuse decision stated (README + the editor entry's docblock, naming `core/table`'s read-back row model and `nextora/advanced-list`'s lack of label/value pairing as the reasons nothing existing fit), an unconditional `role="status"` `aria-live="polite"` region with five translated messages, and `build:blocks` inside `npm run build` with README/CHANGELOG parity |
| Cost, and what that says about the skill | the probe needed **~9 minutes** for one small block, because the skill requires reading `SKILL.md` plus two references (~680 lines) before writing, and the child had no block infrastructure at all — the registrar and the esbuild pipeline had to be created as well. The three misses above are the skill's own gaps, not the agent's invention, and all three are cheap to state in the contract. |

A fresh read-only agent (`skill-rerun`, `w3:p9`) was given the **same** architecture question the original
builder answered with a CPT, pointed at the **patched** suite. It chose hierarchical Pages + `core/query`
with `"parents":[<work id>]`, a thin `templates/page.html`, and a hidden starter pattern as insert-time
material — quoting the new §5.1 verbatim and independently re-deriving `wp-includes/blocks.php:3046-3047`,
page's missing excerpt support and `excludeCurrent` being `@since 7.1.0`. Raw answer:
`baseline/rerun-response.txt`. **This is a post-correction result and never replaces the baseline**: the
delivered theme is still the one produced under the frozen skills and corrected by the lead.

## 9. Not exercised / limitations

- **Not exercised in this project:** the child repeater's draft/Done/Cancel model
  (`resources/ts/editor/repeater.tsx`) — the new block uses canvas-native `RichText` rows with immediate
  patching instead, so the component and its Add/Duplicate-before-open semantics remain unproven here;
  Duplicate and Done/Cancel on `team-section` (the block has neither — correctly marked not applicable);
  WCAG assistive-technology passes; subdirectory installs; multisite; and any WordPress version outside the
  two matrix cells. Block authoring, the block registrar and `build:blocks` were exercised after acceptance
  by the probe in §8.1, on the frozen skill, not during the graded matrix.
- **Required checks still unexercised at handover** (from `$RUN/qa-report.md`, each with its reason):
  applying and persisting a Site Editor variation and the Reset control; image replace/alt through the
  block toolbar; publishing a draft built from the section pattern; the team block's empty state *inside*
  the editor (its front-end half passed); the persisted-database seed re-run (`QA_PERSIST_DB=1` produced no
  fixture report, so idempotency across boots stays unproven); link and button contrast; wp-admin chrome in
  v3+; the minimum cell's editor legs. These are coverage gaps, not observed defects.
- `Tested up to: 7.1.4` is claimed on the primary matrix only; 6.6 and 8.1 are exercised at runtime but the
  host has no 8.1 binary, so PHP syntax was linted under 8.4 and the declared minima remain declarations.
- The matrix's version cells are proved by each instance self-reporting its WordPress and PHP versions —
  never by the CLI banner, which reports the flags instead of what it serves.
- Two low residuals accepted with justification rather than fixed, both documentation-level: the README's
  Site-variation passage says "the editor's own reset option" where the editor actually shows two default
  entries (one card row, one colour-pill row), and `inc/template/skip-link-target.php` documents core's
  skip-link id behaviour without noting that on the declared 6.6 minimum the child's `tabindex="-1"` is at
  worst inert. Neither affects behaviour.
- The five concept images were verified by dimension and ink coverage only; no visual judgement was made.

## 10. What the corrected skills change for the next project

The six patches are all small and all evidence-backed. Together they close the classes of failure this
project actually hit: an undocumented content model (S1), a merge rule that silently drops inherited
presets (S2), an i18n gate with no mechanism for block themes (S3), an unmentioned Site Editor side effect
of the preset layout (S4), an over-strong parity claim (S5) and a content-write trap for seeding from
patterns (S6). The rerun in §8 shows the first of them changing a real decision.

## 11. Verdict

**Useful with documented correction.** For this project the frozen suite was neither sufficient alone nor
mere decoration:

- **What held without intervention.** The enqueue contract, the load-order rule, the non-colliding asset
  naming, the version-constant discipline, the inherited-block reuse rules (no schema change, no invented
  rollback, no team-block clone), the design-system role/data split, the switcher gate and the `:root`
  role-default contract, the pattern-header and one-authority rules, the query composition rules and the
  quality gates' data/parity/security/i18n framing were all followed and all survived an independent review
  and a browser matrix. Six of the seven candidate fixes were implementation defects of the ordinary kind,
  not misunderstandings of the suite.
- **What the suite cost when it was silent.** One undocumented architecture question produced a custom post
  type with a theme-switch data-loss risk (S1) and needed the lead to overrule it. Four further gaps only
  surfaced because a real delivery stress-tested them (S2-S5), and one harness-side failure was misdiagnosed
  by the lead first and needed the registry content read to settle (S6).
- **Numbers.** 10 leader interventions, 8 of them builder-affecting; 6 skill defects corrected (S1-S6),
  one of which surfaced through a harness bug the lead first misdiagnosed as a theme defect; 7 candidate
  revisions (v1→v7), each independently reviewed or measured — v1→v6 by the review loop, v7 by the
  out-of-band block probe; the matrix rows, assertion runs and the block round trip in §7 are the
  verification count that matters — the screenshots and logs behind them are not.
- **What is not claimed.** No causal comparison against a no-skills run, no claim that the corrected skills
  would have prevented every correction, no claim of compatibility outside the two matrix cells, and no
  claim that the delivered theme is production-ready: it is a disposable-instance-verified child theme with
  the parent defects in §8 intact and the unexercised checks listed in §9.

For this suite, "useful" means: with these corrections applied, the same project can be run again with a
materially smaller intervention count — and the specific gaps that made the difference are now written
down rather than remembered.
