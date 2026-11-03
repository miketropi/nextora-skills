# Portfolio field test — do the Nextora skills guide a real delivery?

A real project was run end to end against the **frozen** skill suite: a Nextora child theme portfolio for
a fictional Product Designer / Creative Developer, built by a separate agent, reviewed by a second agent,
accepted by a third, with the coordinator (this author) owning scope, acceptance and skill corrections.

This is an **observational field test**, not a causal A/B experiment. The builder ran with its own global
instructions and its own model; only the evidence below is claimed.

Status: **complete**. Acceptance ran against candidate v6 (theme digest `33053e3c…`, 72 files), and the
delivered tree was then extended by two follow-ups the user asked for: the block-capability probe plus the
invisible-controls fix (**v8**, `884047aa…`), then the modal-editing deepening (**v9**, `858d7b7c…`) — all
80 files; the matrix is §7, the skill verdicts §8 and the overall verdict §11. `$RUN` below is `/tmp/nextora-portfolio-field-test-20261102T150910Z`.

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
| v8 | v7 + the fix for the defect the user found: per-action labels, dashicons available in the editor canvas, a visible `Remove` word, unchanged gates | `evidence/candidate-8-theme.sha256`, 80 theme files, digest `884047aa…` |
| v9 | v8 + the user-requested deepening: item editing moved into one Gutenberg-native modal (draft + Done), the item record extended additively with `description`/`icon`/`emphasis`, and the list allowed to reach zero with a real empty state | `evidence/candidate-9-theme.sha256`, 80 theme files, digest `858d7b7c…` |

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
| `nextora-gutenberg-blocks` | Useful, with three misses only a real build exposed | the inherited-block semantics it documents (ID-keyed members, immediate `patchMember`, one modal, no rollback, minimum one) matched the real block and kept the child from inventing a rollback contract or a team-block clone. A separate time-boxed probe then built the theme's first **new** block against the frozen skill (§8.1): the contracts that carry the most risk landed correctly on the first attempt — `apiVersion: 3` metadata, dynamic `save: () => null` with `render.php`, escaping by context with the child textdomain, a registrar that reads the stylesheet directory, and the `window.wp` handle trap (zero unshimmed `require`, only the five globals its bundle needs, all inside the contract's 14-handle list). A later user-requested deepening — modal item editing with a richer record — followed `editor-and-repeaters.md` §2/§3 without a new gap, which is the strongest evidence in this run that the references are usable as written. What the first pass missed: the skill's "state the reuse decision" requirement was skipped, the editor's list mutations were announced to nothing (no live region), and the block build was not wired into the theme's own `npm run build`, so a clean build left `block.json` pointing at a missing file. |
| `skills/nextora-ecosystem/references/quality-gates.md` §5 | Partially unusable as written | the i18n gate cannot be satisfied by a block theme's template/pattern copy and names no mechanism; it produced one medium review finding and a lead ruling (S3). |

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
| S7 | `block-contract.md` §8's editor contract covered native fields, panels and preview stacks, but never said an affordance has to **render**, and never mentioned glyph availability or per-control labels. | The user reported "add works, no remove" on the delivered block. The controls existed and `removeFact` was wired, yet every icon-only control rendered 24×24 with no glyph because the canvas iframe has no `dashicons` stylesheet, and move-up/move-down/remove shared one `aria-label`. Both agents had passed the UI, because one had no runtime and the lead checked DOM presence instead of the rendered glyph. | Added a bullet: icon-only controls must not depend on a font that may not be loaded in the canvas (enqueue core `dashicons` when using glyph strings; `@wordpress/icons` is not in the fixed handle list), every control needs its own action-specific label plus `showTooltip`, destructive actions survive a font failure, and verification measures the rendered control in the real canvas rather than finding it in the DOM. |
| S8 | `block-contract.md` §6 documented the block build but not the theme's **aggregate** build; `SKILL.md` §3 required the reuse decision without saying where it has to live. | The first probe's `block.json` pointed at an `index.js` that `npm run build` did not produce — green in a working tree, broken in a clean checkout — and the reuse decision existed only in the agent's report, so the reviewer had to mark its absence a `fail` even though the agent had reasoned about it. | §6 gained the wiring rule (`build:blocks` in the `build` chain, and check the built files are the ones `block.json` names); §3 now says the decision belongs in the deliverable — the editor entry's docblock plus a README line naming the rejected candidates. |

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
| **Defect the *user* found after both agents passed it** | "add more items works, there is no remove". Measured in the live canvas: the controls were in the DOM and `removeFact` was wired, but every icon-only control rendered **24×24 with no glyph** — the dashicons stylesheet is absent from the editor canvas, so all four buttons per row were invisible, and move-up, move-down and remove shared the single `aria-label` "Fact 1 actions", so tooltips and assistive technology could not tell them apart either. Neither the reviewer (no runtime) nor the lead's earlier check (DOM presence, not rendered glyph) caught it; the person using the block did, in one look. |
| **Deepening the user then asked for: modal editing** | "if an item's data gets richer, move the options into a modal so the author updates in one focused place, with Gutenberg-native style and UX". `editor-and-repeaters.md` §3 already prescribes exactly this for new repeated data (compact rows, one `DropdownMenu`, one `Modal` per item, immutable draft, single commit on Done, Cancel/Escape discard-only, explicit minimum only when content requires it, empty-state Add, focus return, accessible names, pluralised live-region feedback), so this pass tested the skill's own model rather than inventing one — and it came out **with no new gap**. Item record extended additively (`description`, `icon` from a curated allowlist, `emphasis`); the positional index-keyed array stayed positional, with normalisation filling defaults for records written before those keys existed. |
| Modal proof (lead, in the real canvas) | one dialog (`components-modal__frame … test-skills-fact-list-modal`, heading `Edit: Role`, header Cancel/Done/Close, focus lands on the Label field); Label/Value are `RichText` with `aria-label`s, URL a `LinkControl`, plus `TextareaControl` + two `SelectControl`s; typing in Value then **Done** closed the dialog and committed (`Product design studio`, announced "Role updated. 2 facts in the list."); typing then **Escape** closed it with the stored value unchanged and reopening showed the committed value — draft-only discard, as the skill requires; save → reload → 2 records parsed with **no invalid-block notice**; front end rendered `has-emphasis-strong`, `dashicons-star-filled` with `aria-hidden`, the `esc_url`'d link and the description span; with zero items the renderer emitted **no** `test-skills-fact-list` markup at all |
| Fix and its proof | per-action translated labels with tooltips, core `dashicons` enqueued for the editor canvas, and a visible `Remove` word so the destructive action survives a font failure. Verified live in the real canvas: `dashiconsInCanvas: true`, each glyph span computes `font-family: dashicons` with a non-empty `::before`, labels distinct (`Link fact 1`, `Move fact 1 up`, `Move fact 1 down`, `Remove fact 1`), remove now 89×24 with the visible word; clicking it took the row count 2 → 1, announced "Fact 1 removed. At least one fact is required." through the `role="status"` region, disabled itself at the minimum, and `Add fact` restored the row |
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
- **Numbers.** 10 leader interventions, 8 of them builder-affecting; **8 skill defects** corrected
  (S1-S8), one of which surfaced through a harness bug the lead first misdiagnosed as a theme defect, and
  two of which only an out-of-band probe could reach; 8 candidate revisions (v1→v8); the matrix rows,
  assertion runs and the block round trip in §7 are the verification count that matters — the screenshots
  and logs behind them are not.
- **The most useful signal in the whole run came from a person, not a check.** The invisible row controls
  survived an independent source review, a live runtime probe and the lead's own browser pass, and were
  obvious in one look to the user. A rendered-affordance check now belongs in the loop before "pass".
- **What is not claimed.** No causal comparison against a no-skills run, no claim that the corrected skills
  would have prevented every correction, no claim of compatibility outside the two matrix cells, and no
  claim that the delivered theme is production-ready: it is a disposable-instance-verified child theme with
  the parent defects in §8 intact and the unexercised checks listed in §9.

For this suite, "useful" means: with these corrections applied, the same project can be run again with a
materially smaller intervention count — and the specific gaps that made the difference are now written
down rather than remembered.

# Field test 2 — 2026-11-03 — the Lumen Habitat run

A **second, independent** delivery against a fresh freeze of the same suite, added below rather than
merged into the record above. The first field test's claims are unchanged: its candidate, its
interventions, its eight skill patches (S1–S8) and its unverified list stay exactly as written. Where
this run touches the same surface, it says so and points at the earlier section.

## 12.1 What was frozen, and what ran

| Item | Value |
| --- | --- |
| Skill baseline (authority for the delivery run) | `$RUN/baseline/skills-baseline.sha256` — **15 files**: 5 `SKILL.md`, 7 references, `README.md`, 2 evals |
| Parent + sample source baseline | `$RUN/baseline/parent-sample.sha256` — 1410 files (`nextora` + `nextsaas`, excluding `node_modules`, `vendor`, VCS), list digest `dbdbae0c…c282f` |
| Freeze rule | no file under `skills/`, `evals/` or `README.md` may change between the freeze and the candidate handoff; skill edits were drafted outside the repository and applied only after handoff |
| Coordinator | `9router/cmc/deepseek/deepseek-v4-flash` |
| Builder | `omp` 18.11.0, pane `w1:p1`, renamed `chilltheme-builder`; cwd the empty target theme |
| QA | `omp`, pane `w1:p2` (`chilltheme-qa`), created for this run |
| Reviewer | `omp`, pane `w1:p3` (`chilltheme-reviewer`), created for this run |
| Runtime | node v24.16.0, npm 11.13.0, host PHP 8.4.10, Composer 2.9.2, `@wp-playground/cli` 3.1.60, Herdr 0.9.5 (session `chilltheme-test`) |
| Live site | the same occupied Local install as the first test; never touched, and never a verification target |
| `$RUN` | `/tmp/lumen-eval` |

Two constraints the builder and both reviewers were given in writing: they may write **only** in their
own scope (builder: the child directory; QA: its own evidence directory; reviewer: nowhere), and
**none** of them may spawn internal subagents. Playground was the only runtime; no agent ran `wp`/WP-CLI
against the occupied site.

## 12.2 The candidate

Fictional subject: **Lumen Habitat**, an editorial sustainable-architecture studio. Delivery target was
an empty `chilltheme-test` directory beside the parent and the sample.

| Item | Value |
| --- | --- |
| Path | `wp-content/themes/chilltheme-test` |
| Files | **115** (excluding `node_modules`, `vendor`, and the `.phpunit.cache/` directory; the 115th file is `scripts/browser-measure.mjs`, added late — see §12.8 D1) |
| Digest at handoff | `e93a652dd2f7d9a75570a9806ed6791121963d50be8e5c8ff6f1b3116dddec18` |
| Digest finally verified | **`d32990f995ca98a859a10454dcda5c52b6feca5a7dccbb2c705b709edc712843`** — method A below, stable across repeated takes, and matching the builder's own value. The tree was re-gated by the coordinator after the last pass: `php -l` 0 failures, `typecheck` rc 0, `lint:php:all` rc 0, `composer test` **118 tests / 668 assertions**, and a double `npm run build` both deterministic and byte-identical to the committed tree. |
| Digest method | **method A (the documented one):** `find . -type f -not -path './node_modules/*' -not -path './vendor/*' -not -path './.phpunit.cache/*' \| sort \| xargs shasum -a 256 \| shasum -a 256`. **method B (used by the coordinator's re-gate script):** path + NUL + `sha256(file bytes)` per file, streamed into one `sha256` — same exclusions, same tree, value **`823d129c2c5328e77064c68ef6727edc550c385e16109697786428750165f417`**. Both are stable across repeated takes and they are simply different formulas; **a digest is meaningless without its formula**, which is the second variant of the same lesson as the cache exclusion below. Earlier values in this record (`668d807d…`, `dda36551…`, `18f3163d…`, `569bb09f…`, `d01283da…`, `57f597af…`) are earlier points in the fix cycle. |
| Digest method — **corrected twice during this run** | First the exclusion set was wrong: `composer test` rewrote `.phpunit.cache/test-results`, so the digest moved after every test run — a digest over a tree containing a test cache is not a digest of the source. Then the formula differed silently between the coordinator's script (method B) and the documented command (method A), which is how two records of one tree ended up quoting different values. Both mistakes are reported rather than quietly fixed, because they are exactly the class of defect this field test exists to catch. |
| Test suite | 107 tests / 582 assertions at handoff → 115 / 658 after the contrast fixes → **118 / 668** in the finally verified tree, every run `OK` |
| Surfaces | 9 templates (`index`, `front-page`, `home`, `archive`, `search`, `single`, `page`, `404`, `page-studio`), 2 child-owned parts, 9 patterns (3 hidden whole-surface, 6 insertable sections), 3 dynamic blocks, 8 PHPUnit test files |
| Blocks | `chilltheme-test/architectural-hero`, `chilltheme-test/habitat-grid` (repeater in one modal), `chilltheme-test/impact-band` (the dark/image case) |

## 12.3 Coordinator acceptance before the runtime (independent)

| Gate | Result |
| --- | --- |
| `php -l` over all 37 authored PHP files | 0 failures |
| `npm run typecheck` | rc 0 |
| `npm run lint:php:all` (PHPStan + php-cs-fixer dry-run, 23 files) | rc 0 |
| `composer test` | OK, **107 tests / 582 assertions** |
| Unresolved `require("@wordpress/…")` in any bundle | none |
| `index.asset.php` handle list | exactly the 14 known handles, all three blocks |
| Build determinism | a full build run **twice** in a copy of the tree (candidate untouched, `node_modules`/`vendor` symlinked) produced a byte-identical tree: relative-path digest `b7540433c062c1450a8e` at rest, after build 1 and after build 2 |
| Artifact reproducibility | the rebuilt copy and the candidate are byte-identical over 115 files — the committed generated files are exactly what the source produces |
| Manifest version | content hash (`1.0.0.<8 hex>`), read from `inc/bootstrap/version.php` — **not** `Date.now()` |
| No remote demo media | none in `templates/`, `patterns/`, `parts/`, `blocks/`, `theme.json` |
| No `theme` attribute pinning the parent | none |
| Literal colours in authored CSS | **zero** — every colour is a preset variable or a `color-mix()` of one |
| **Runtime invariants, run by the coordinator from the repository** | `npm run measure:ui -- --url=<cell>` → **PASS**, on the tree above: 6 sections measured; **every adjacent pair shares an edge, gaps `0, 0, 0, 0, 0`**; **no section wrapper carries a margin**; focus rings **19.8:1** for `.chilltheme-test-button` (white on `#0a0a0a`), `.chilltheme-test-link` (`#0a0a0a` on white), `.chilltheme-test-cta a` and the header nav link, reproduced under a keyboard `:focus-visible` pass; an unauthored card paints **`rgba(0, 0, 0, 0)`**; the scope computes `border-box`; **0 elements overflow a 320 px viewport** and the document does not scroll sideways (`305 ≤ 320`) |

That last row is the run's most useful artifact and it closes the reviewer's D1: the six runtime
invariants that were only agent-run measurements are now **committed, zero-dependency checks** in
`scripts/browser-measure.mjs` driven by `npm run measure:ui`, so anyone can reproduce them against a
disposable instance. The harness is plain Node 24 (`fetch` + `WebSocket` over CDP) plus a system Chrome
— no new dependency, no Playwright.

## 12.4 Intervention ledger

Classification: product choice / implementation defect / environment issue / skill omission. Only the
last justifies normative text.

| # | Raised by | What | Class |
| --- | --- | --- | --- |
| I1 | coordinator, before the section CSS existed | A dark/overlay section must not pin "a light preset" for its foregrounds: the parent's roles are positional and their **values invert per scheme** (`styles.color.background` = `base`, `styles.elements.heading.color.text` = `contrast`; `styles/color-scheme/midnight.json` inverts both). | **skill omission** — the suite never stated a pair must be selected by role |
| I2 | coordinator, same steer | `supports.spacing.padding`/`margin` (core, on the wrapper) and a section's own rhythm padding (on its own element) are two owners on two elements; an empty value emits nothing. | **skill omission** — no suite text distinguished the owners |
| I3 | coordinator | Freeze integrity: no agent edits the skills, the parent or the sample; no internal subagents; Playground only. | environment / process |
| I4 | coordinator | Verified the runtime cells: `--wp=6.4` resolves to **WordPress 6.4.14** and boots; `--wp=6.4.3` is not a fetchable artifact; `latest` served **7.1.4** with PHP 8.3. | environment |
| I5 | reviewer | **S1, high — the derived foreground was measured against a surface the section never painted.** With no background colour and no image, the resolver still composited the block's *default* overlay over a `#808080` stand-in and chose `base` (white) as the readable role, while the real surface was the white page canvas: **1.00:1**. Reached `habitat-grid` and `architectural-hero` in their default state. | **implementation defect**, fixed by the builder before the advisory landed; coordinator re-read the fix |
| I6 | reviewer | **S2, medium — the media layer's wrapper was unconditionally `aria-hidden="true"`**, so the `role="img"`/`aria-label` branch inside it was never exposed and a meaningful alternate text reached nobody. | implementation defect, accepted and fixed |
| I7 | reviewer | **S3, medium — a mislabelled control.** "Card text colour" wrote the card's *heading* variable while the card's content variable received a derived role, so the control changed the title and not the copy. | implementation defect, accepted and fixed |
| I8 | reviewer, decided by coordinator | `quality-gates.md` §5 demanded a manual `wp_set_script_translations()` that core performs automatically for `block.json` assets declaring a `textdomain` with `wp-i18n` in the dependency list (`wp-includes/blocks.php:321-323`); neither source theme calls it anywhere. | **skill omission / stale claim** |
| I9 | coordinator | The evaluation legs read **four** `SKILL.md` files, not one, because the design-system skill's own reference table links sibling skills and `--no-skills` does not stop an agent following relative links. A constrained single-skill variant was added to control the confounder. | environment / method |
| I10 | coordinator, via recon | `block-contract.md` §2 stated the asset manifest "is regenerated with a `Date.now()` version". True of the **parent** (`nextora/scripts/build-blocks.mjs:26`), false of the **sample child**, which content-hashes (`nextsaas/scripts/build-blocks.mjs:53`). Quoted generically it teaches a new child to ship a non-reproducible manifest. | **skill omission / stale claim** |
| I11 | reviewer | Thirteen cross-skill references inside `skills/*/references/*.md` were backticked one `../` short and did not resolve from their own file (the bracketed links in the same files were correct). Confirmed independently by the coordinator with a checker. | **documentation defect in the baseline**, in scope for the plan's "relative references resolve" check |
| I12 | coordinator, from a QA cell run | **Four extraction bugs in the candidate's own smoke assertions**, all of which made a correct theme look broken: `wp_get_global_settings()` returns preset keys **bucketed by origin** (`default`/`theme`/`custom`/`user`), so `array_column(…, 'slug')` over it yields nothing; `WP_Block_Type::$textdomain` is **null** even when `block.json` declares it, because core only uses the value for i18n-schema and script translation; and `WP_Block_Pattern_Categories_Registry::get_all_registered()` is **numerically indexed**, so `in_array( …, array_keys(…) )` can never match. Settled by a probe the coordinator ran directly (WordPress 7.1.4 / PHP 8.3.33, parent + child mounted): the parent's spacing slugs are present inside the `theme` bucket, the block's `category` is correct, and 41 patterns and 26 categories are registered. Two of the palette assertions also passed **while testing nothing** — they read the child's own helper rather than the merged settings, so they would have passed with the parent's palette absent. | **implementation defect** (test correctness) **+ skill omission** — no skill documents either API shape, which is why the traps were hit |
| I13 | coordinator, by fetching every route and counting landmarks | **`/` and `/studio/` had no `<main>` landmark.** The seven ordinary templates all carry `<!-- wp:group {"tagName":"main","anchor":"content",…} -->`; the two thin templates (`front-page.html`, `page-studio.html`) are header + one `wp:pattern` + footer, and the hidden patterns they reference provide sections but never open a landmark — so the theme's two most important surfaces were the two without one. A direct consequence of the suite's own thin-template recipe, which never said the template still owns the page's landmarks. | **implementation defect** (inconsistent with the theme's own seven other templates) **+ skill omission** — fixed by the builder at 11:23, confirmed by the coordinator in a browser (`main=1`, `h1=1` on both, every width) |
| I14 | coordinator, in a real browser | **An unowned 24px gap between every adjacent pair of front-page sections.** Measured at 1280x900 on `/`: wrapper `margin-top` is `0px` for the first section and **`24px`** for the other three, wrapper padding is `0px` everywhere, and consecutive section boxes are separated by exactly `24px` (`1006 → 1030`, `1679 → 1703`, `2746 → 2770`). Cause: the wrapper margins are never neutralised, so core's root cascade applies `:root :where(.is-layout-flow) > * { margin-block-start: 1.5rem }` from the parent's `styles.spacing.blockGap` to every section wrapper, because the sections are direct children of the `is-layout-flow` `<main>` group. Visible consequence: a full-bleed dark band ends and its neighbour begins 24px later, with page canvas in between. The authored rhythm (112 / 72 / 60 / 40px padding) was correct on top of it. | **implementation defect** against a rule the *plan* stated ("adjacent sections do not depend on collapsing margins") but the frozen suite did not — which is precisely T4's justification |
| I15 | QA | **HIGH — a partial-alpha section background leaves its content below AA.** With only `backgroundColor: "rgba(18,52,86,0.5)"` set, the derived content is `rgb(82,82,82)` on the composited surface `rgb(136,153,170)` = **2.68:1** (the heading derives 6.78:1 and passes), at all four widths and the 200% model. The derivation reads the stored colour without compositing its alpha over the surface beneath it, so it decides against a different surface than the browser paints. | **implementation defect** at the derivation's edge; the new rule already requires measuring the *composited* surface (§3.1.3/§3.1.4), so this is the rule being right and the implementation not yet matching it |
| I16 | QA | **HIGH — a card inside a dark section is white-on-light, 1.10:1.** `grid-dark` resolves the section correctly (19.8:1) but the card keeps its own light surface, so the card copy and its link measure `rgb(255,255,255)` on `rgb(244,244,244)`. The card is its own surface with its own pair, and it inherits the section's foreground instead of deriving against the card's surface. | **implementation defect**; the plan's §5 ("block-scoped overrides set only supplied variables… non-heading prose/list/link content to `paragraph`") did not anticipate a nested surface, which is worth noting as a rule refinement candidate |
| I17 | QA | **MEDIUM — the focus ring, precisely characterised over three rounds, and closed.** QA's first pass reported the ring as a constant `rgb(127,77,62)`; the coordinator showed from source that the child's token chain contained no `primary` and asked for a targeted A/B; QA's A/B settled it: **non-anchor focusables got the surface-derived token** (19.80 / 19.80 / 18.00 / 6.78:1 on the dark, white, `#f4f4f4` and alpha surfaces), while **the child's own anchors** (`a.chilltheme-test-button`, `a.chilltheme-test-link`) got `rgb(127,77,62)` in every context (6.92 / 6.29 / **2.86** / **2.37** / **1.84**:1) despite the token resolving correctly on those same elements. The child's built CSS contained exactly one `outline-color` declaration, and `#7f4d3e` is `--wp--preset--color--primary`. | **implementation defect at the boundary — fixed.** The builder added an anchor-qualified rule (`.chilltheme-test-scope a.chilltheme-test-button:is(:focus, :focus-visible)` and the link equivalent) and recorded `a:-webkit-any-link:focus-visible` as the competitor it now outranks. **Verified by the coordinator running the repo's own harness** (§12.3): `.chilltheme-test-button` `2px solid rgb(255,255,255)` on `rgb(10,10,10)` = **19.8:1**, `.chilltheme-test-link` `rgb(10,10,10)` on white = **19.8:1**, `.chilltheme-test-cta a` **19.8:1**, header nav link **19.8:1**, and the keyboard `:focus-visible` pass reproduces all four |

Counts: **9 implementation defects** (I5, I6, I7, I12's test-correctness half, I13, I14, I15, I16,
I17) — **all nine fixed and re-verified**: I13–I16 independently re-measured by QA (alpha 2.68 →
6.78:1, card pair 1.10 → 19.80:1, boundary deltas 24 → 0 px, 320 px overflow 37 → 0) and I17 verified
by the coordinator running the repository's own harness (anchor rings 19.8:1). Plus **6 skill
omissions or stale claims** (I1, I2, I8, I10, and the API-shape and landmark halves of I12 and I13),
**1 baseline documentation defect** (I11), **3 environment/method** items (I3, I4, I9), and **0
product-choice disputes**. One further item was classified by the reviewer as a *product-choice gap in
the new rule itself* — the focus indicator inheriting the content colour — and was fixed on the rule
side as well as in the candidate (T10). Note that I15, I16 and I17 were found by **QA's own harness**,
not by the coordinator's pass or the reviewer's audit: all three live in the colour layer under a state
(alpha background, nested surface, focus on an anchor) that neither source reading nor a route sweep
reaches, which is the clearest argument in this run for the disposable-runtime cell existing at all —
and for committing its measurements once they exist.

## 12.5 Skill defects corrected

Each has an exact baseline quote, live-source evidence and a consequence. The plan's five intended
gaps were verified against the sources first; four were confirmed as silences and one as an active
mis-steer.

| ID | Baseline text | Evidence gathered this run | Patch |
| --- | --- | --- | --- |
| T1 | `nextora-design-system/SKILL.md` step 2 mechanism table has five rows and none of them names a foreground/background **set**, a derived default, or a contrast check. | `grep -riE 'wcag\|4\.5:1\|luminance'` over `skills/` → **zero hits**. The parent ships no contrast helper either (same grep over its `blocks/`, `inc/`, `resources/` → zero), so a derivation is a **[new-output requirement]** with no pattern to copy. | New workflow step 6, two new mechanism-table rows, three new hard limits, three new completion checks; new reference §3.1 with thresholds, the three-attribute set, the derive-from-what-you-paint rule, the composited-image rule, and the computed-style checks |
| T2 | `grep -rn blockGap skills/` → **zero hits**; `block-contract.md` §3 showed `supports.spacing` only as a sample manifest line and never distinguished wrapper spacing from inner gaps or section rhythm. | Enumerated the parent's 40 block manifests: **39** declare `supports.spacing`, **20** declare a `blockGap` key inside it (**19** enable it; `advanced-button-button` sets it `false`), **16** declare their own gap attribute under **14** distinct names; `blocks/expanding-cards/block.json` carries wrapper padding, wrapper margin, wrapper `blockGap` and its own `cardGap` on one block. | New workflow step 7 + ownership table; new reference §3.2 with the verified counts, the identity rule, the sanitiser rule, the `settings.spacing` replacement hazard and the no-margin-collapse rule; new gate in `quality-gates.md` §6; four new acceptance steps in `block-contract.md` §11 |
| T3 | `presets-and-switching.md` §3 said "Reuse the real signatures; do not write a parallel resolver" while naming only parent helpers. | The parent has **25** `nextora_*_resolve_color()` implementations with real behavioural drift (`advanced-list` does no hex→slug mapping; `advanced-container` returns `''` for unknown; `advanced-icon` returns `'currentColor'` for empty and is the only one accepting `has-*-color`). Read literally, the sentence endorses calling *a parent block's* internals from child code — which is exactly what the constrained pre-change leg proposed (`require_once get_template_directory() . '/blocks/advanced-icon/lucide.php'`). | Added the explicit rule that a child resolves through **its own** shared helper and that a new parent block should share one, in `nextora-design-system/SKILL.md` hard limits and in `block-contract.md` §3/§9; and both "reuse the signatures" sentences now say to reuse the **behaviour** while implementing it as one shared theme helper |
| T4 | `nextora-site-composition` said nothing about spacing at all: `grep -rni 'margin collapse\|rhythm' skills/` → **zero hits**, and no composition step required a section boundary to name an owner. | The parent's own patterns do it correctly (`impact-statistics.php`, `our-core-values.php`, `contact-form.php`, `banner-with-text-button.php` all set `style.spacing.margin` top/bottom to `0` and own the rhythm with preset padding), while its templates and the sample's `main` group demonstrate the opposite: an unowned `1.5rem` from the root `blockGap`. | New pattern requirement in `nextora-site-composition/SKILL.md` step 5; new boundary-verification paragraph in step 10; new completion check; new reference §5.2 with the boundary measurement table; new verification step in `templates-and-patterns.md` §11 |
| T5 | `quality-gates.md` §6 covered semantics, keyboard, live regions, motion and width, and named no contrast ratio; §8's design-system recipe asked for computed colour values without a threshold or a spacing step. | Same zero-hit grep as T1. | New normative contrast bullet and two spacing bullets in §6; new "Colour contrast and section rhythm" acceptance block in §8 spanning theme defaults, light, dark, image-overlay, custom literal, partial-alpha, empty override, a style variation and a visitor preset, at narrow and wide widths |
| T6 | `quality-gates.md` §5 required a manual `wp_set_script_translations()` for editor scripts. | Core calls it automatically for `block.json` assets whose manifest declares a `textdomain` and whose dependency list contains `wp-i18n` (`wp-includes/blocks.php:321-323`). Neither theme calls it anywhere. | Rewritten to say which case core already covers and which case still needs an explicit call |
| T7 | `block-contract.md` §2 presented a `Date.now()` manifest version as the pipeline rule. | Parent: `Date.now().toString()` at `nextora/scripts/build-blocks.mjs:26`. Sample child: `assetVersion(entryFile)` = theme version + content hash at `nextsaas/scripts/build-blocks.mjs:53`. | §2 and the SKILL's step 8 now require a no-change rebuild to be byte-identical, name the content-hash form, and say to read `writeAssetFile()` first |
| T8 | 13 backticked cross-skill paths in `skills/*/references/*.md` resolved one level short. | A checker written for this run (`check-refs.py`), which resolves both the bracketed-link form and the backticked-path form from each file's own directory, reported **0** bracketed failures and **13** backticked failures on the frozen baseline, and **0** after the fix. | Normalised to two levels up |
| T9 | `quality-gates.md` §3 required missing build output to be visible, but no rule anywhere covered **test-fixture hygiene**. | The candidate's `tests/BlockRegistrationTest.php` created `blocks/ztemp-orphan` and `blocks/ztemp-candidate/block.json` **inside the shipped theme**, cleaned only in a `finally` — and the registrar globs `blocks/*`, so an aborted run leaves a real block. | New §3 bullet: a generated artefact or fixture must not leak into the shipped tree, never rely on `finally`, and a "writes nothing" claim must be verified by reading the tests |
| T10 | §3.1 as first written (post-edit) listed focus rings in the 3:1 row without saying what to measure them against. | The candidate derived the ring from the content colour, so an author-set low-contrast content colour took the ring below 3:1 (light `surface` on white ≈ 1.1:1). | Added the paragraph "a focus indicator is its own foreground, measured against the surface it is drawn on", with the panel-reporting requirement |
| T11 | `block-contract.md` §6 said the `@wordpress/*` shim plus `jsxImportSource: 'react'` "is the whole runtime model". | The shim intercepts `@wordpress/*` only, and `react` is present in `node_modules` as a transitive dependency — so **both** source themes bundle a full React into every block bundle: React's element symbol appears in 40 of the parent's 40 and 39 of the sample's 39 `blocks/*/index.js`, with no `react` entry in any manifest. The candidate maps `react`/`react-jsx-runtime` onto `window.wp.element` and ships zero. | §6 and the SKILL's hard limits now state that the shim does **not** cover `react`, name the `onResolve` mapping, and give the grep that proves it |
| T12 | No skill documented how to **read** merged presets in PHP. | `wp_get_global_settings( array( 'spacing', 'spacingSizes' ) )` returns origin-bucketed objects (`default`/`theme`/`custom`/`user`), not a flat list, which is why six spacing and two font assertions failed against a correct theme. Probe output recorded in §12.4. | Added to `presets-and-switching.md` §1, with the literal shape, the flatten instruction, the "assert the `theme` bucket" guidance and the "a populated `default` bucket is not a defect" warning |
| T13 | No skill documented that `block.json`'s `textdomain` is not readable back off the block type. | `WP_Block_Type` declares `public $textdomain = null` but core never assigns it from metadata (`wp-includes/blocks.php:598,604-606`), so three assertions failed against correct manifests. | Added to `block-contract.md` §3: declare it in the manifest, assert it from the manifest, never from the block type |
| T14 | No rule covered **what a composition assertion may count**. | Two assertions failed against a correct theme: "every front-page section renders" counted `data-chilltheme-test-section=` occurrences and expected 6, but two of the six front-page sections are built from core blocks and legitimately carry none (4 markers is the correct number); and a wrapper regex required one attribute order and one newline. | Extended the §3 fixture-hygiene bullet: assert what the composition guarantees rather than a marker only some of its parts emit, and parse markup with a parser rather than pattern-matching an attribute order |
| T15 | `templates-and-patterns.md` §5 required a hidden pattern plus a thin template and said nothing about page landmarks. | Applying the recipe literally produced a theme whose seven ordinary templates carry `<main id="content">` while `front-page.html` and `page-studio.html` — the two thin ones — carried none, because a pattern of sections is not a page. Measured on the front end: `main=0` on `/` and `/studio/`, `main=1` everywhere else. | New paragraph in §5: a thin template still owns the page's landmarks — wrap the `wp:pattern` reference in a `core/group` with `tagName: main` **in the template**, keep the pattern owning only the section list, and assert exactly one `<main>` and one `<h1>` on every route |
| T16 | §3.1 as first written said "derive from the background actually in effect" but neither covered a **nested** surface nor a background whose own alpha is non-opaque. | QA measured two failures against a correct-looking implementation: a card inside a dark section ran white-on-light at **1.10:1** because it inherited the section's foreground across a surface boundary, and a `rgba(18,52,86,0.5)` band derived a foreground for `#123456` while rendering over `rgb(136,153,170)`, measuring **2.68:1**. | Two new bullets in §3.1.3: every surface derives its own pair, nesting included; and composite the background's own alpha over the surface beneath before measuring |

## 12.6 The evaluation loop: before and after

Two prompts were added to `evals/scenarios.md` as §8 and §9, each demanding decisions the suite had
never written down, and each run with **exactly one** `SKILL.md`, read-only tools and no skill
discovery, in a fresh session:

```bash
omp -p --no-session --no-skills --tools read,glob,grep @skills/<skill>/SKILL.md @<prompt>
```

### Method, including the confounder

`--no-skills` stops *discovery*, not *link-following*: the pre-change contrast run reported reading
**four** `SKILL.md` files, because `nextora-design-system/SKILL.md` links the ecosystem references and
names the sibling skills it pairs with. So the unconstrained legs measure **the suite**, not one file.
A constrained variant was therefore run for both scenarios and both states, adding a system-prompt
constraint that forbids opening any other skill directory and tells the agent to treat out-of-scope
links as such. Both constrained legs honoured it — zero cross-skill references in either output.

### Result — the honest version

**Both pre-change legs already met substantially every decision the plan expected after the change.**

| Expected after the change | Pre-change unconstrained | Pre-change constrained (one skill) |
| --- | --- | --- |
| Separate heading / content / background roles | yes | yes |
| Empty means theme default, derived | yes — picks the best of the theme's own anchors by WCAG ratio, never a lightness threshold | yes |
| Changing a background never rewrites an explicit foreground | yes, plus a measured notice with an explicit remedy | yes |
| Stated thresholds | yes, self-derived (4.5:1 / 3:1) | yes, and it separated 18 px body text from large text correctly |
| Overlay as the readability mechanism for a photo | yes, with the limit stated | yes, with the worst-case composite over both photograph extremes |
| Verification on computed styles | yes, in detail (including a pixel-sampling proposal it then declined to claim) | yes |
| Never promises a universal automatic foreground for an image | yes | yes |
| Every space names its owner, no margin collapse, first/last boundaries | yes — seven decisions, each with the rejected alternatives | yes |
| Rhythm stored as preset identity, wrapper margins neutralised | yes | yes |
| Verification reads computed paddings at two widths | yes | yes |
| **Resolves through a shared child-owned helper** | **no** — proposes calling parent block internals | **no** — same, explicitly reasoning "not a parallel one" |

So the delta this run can honestly claim is **not** "the agents started getting contrast and spacing
right". It is:

1. the suite now **states** the rules it previously left to the model's own knowledge, and states them
   with the live-source evidence and the acceptance checks attached;
2. it stops **mis-steering** on one point (T3): the frozen text's "reuse the real signatures; do not
   write a parallel resolver" led both legs to depend on a parent block's internals, which the
   constrained leg chose deliberately;
3. two stale/over-general claims (T6, T7) and one structural documentation defect (T8) are gone.

### Post-change result — the one place the text changed the answer

All four post-change legs ran against the edited suite with the same invocation. Three of them
reproduced every decision the pre-change legs already met. On the point where the pre-change legs
went wrong, **both** post-change contrast legs changed behaviour, and they changed it in the words the
new rule supplies:

| Leg | Resolver decision | Quoted from the output |
| --- | --- | --- |
| pre-change, unconstrained | calls parent block internals | "Parent helpers this block **calls instead of copying**: `nextora_ac_resolve_color()`, `nextora_icon_resolve_color()`…" |
| pre-change, constrained | calls parent block internals, deliberately | "`nextora_band_resolve_color()` is a thin wrapper over the **parent's own resolver**, not a parallel one" |
| post-change, unconstrained | **shared child-owned helper** | "Resolver: **one child-owned PHP/TS helper**, not `nextora_ac_resolve_color()` / `nextora_icon_resolve_color()` (per-block parent internals, not an API, already disagree)." |
| post-change, constrained | **shared child-owned helper** | "One new shared child helper, `inc/blocks/color.php` … It reimplements the *contract* of `nextora_ac_resolve_color()` **without calling it** — a child must not call a parent block's per-block internals, and the parent's three resolvers already disagree." |

The same legs also now state the derive-from-what-you-paint rule in the terms the new §3.1 uses —
"no background colour and no image layer → **return `null`**: the band paints nothing, so there is
nothing to derive … (Deriving 'white text' over a stand-in grey here is exactly the white-on-white
defect)" — and the worst-case composited image bound ("composite `α·overlay + (1−α)·black` and
`α·overlay + (1−α)·white`, and require the candidate to pass against the **worse of the two**").

Both post-change **spacing** legs likewise reproduced the seven-owner decomposition, preset identity,
margin neutralisation and the computed boundary checks the pre-change spacing leg produced (the
unconstrained leg names an owner 9 times against the pre-change leg's 4, and adds explicit first- and
last-section boundaries). That gap was a silence, not a failure, and the result is reported as such.

For completeness: the pre-change legs are not retracted, and the post-change improvement is claimed
only for T3, where the baseline text actively pointed the wrong way. Everything else is codification.

## 12.7 Runtime matrix and post-change legs

### Coordinator's own runtime pass (independent of QA)

The coordinator ran the candidate's own cell rather than relying on either agent, using the
candidate's `blueprint.json` against a disposable Playground instance with the parent mounted at
`/wordpress/wp-content/themes/nextora` and the child at
`/wordpress/wp-content/themes/chilltheme-test`:

| Item | Result |
| --- | --- |
| Cell self-report | **WordPress 7.1.4, PHP 8.3.33**, stylesheet `chilltheme-test`, template `nextora` |
| Smoke assertions on the final artifact | **112 checks, 110 pass, 2 fail** (both failures shown below to be assertion bugs, not theme defects) |
| Covers, and passing | active theme/parent resolution; header↔constant version parity; text domain; `Tested up to` absent; all 7 inherited palette slugs, all 6 spacing slugs and both checked font families **surviving the merge**; `theme.json` v3; only `page-studio` declared; both parts; all 9 templates each referencing the child header and footer; no remote demo URLs; all 3 blocks registered, in the child category and declaring the child textdomain; the block category offered; both pattern categories; all 9 patterns registered and the 3 whole-surface ones hidden; child stylesheet and script enqueued with their parent dependencies; the JS marker before first paint; **four AA-pair measurements** (theme default 7.81:1 on `#ffffff`; light 18.00:1 / 7.10:1 on `#f4f4f4`; dark 19.80:1 on `#0a0a0a`; image + overlay 13.77:1 on `#2d2d2d`) plus button text 6.92:1; hero renders a single `h1` with its inline emphasis, its section rhythm and **no pinned colour**; the front page carries exactly **one `h1`**; no unresolved pattern reference is printed; the journal strip queries published posts; stored copy is not double-escaped into `u0026`; the studio composition renders and carries one `h1` |

**Probe evidence for the two failures** (both closed by the builder after this run; the fix is
assertion-side, because the rendered output is correct):

```
<div class="chilltheme-test-scope wp-block-chilltheme-test-architectural-hero" data-chilltheme-test-block="architectural-hero">
	<section class="chilltheme-test-architectural-hero chilltheme-test-section" data-chilltheme-test-section="architectural-hero" style="--chilltheme-test-heading:var(--wp--preset--color--base);…
```

- `wrapper and section are two different elements` — the assertion required core's generated class
  first; the child's own `chilltheme-test-scope` legitimately precedes it, and the structure is
  exactly what the check wants. Failed on attribute order, not structure.
- `every front-page section renders` — counted `data-chilltheme-test-section=` and expected 6. The
  probe reports **4** markers with distinct values `['architectural-hero', 'habitat-grid',
  'impact-band']` (`habitat-grid` twice), because `section-journal` and `section-cta` are composed
  from core blocks and legitimately emit no custom-block marker. The `page-home` pattern does
  reference six patterns, which is the fact the check meant to assert.

### QA's matrix

QA built its own harness under `$RUN/qa/` — a version-checked cell runner, a route fetcher, a
token-gated mu-plugin runner for in-cell probes, a browser pass, and a `selftest.sh` that proves
detection on 19 synthetic cases (including version mismatch, missing child classes, notices and a
deliberately failing assertion), plus a control cell run against the **sample** child to prove the
harness detects differences at all.

| Item | Result |
| --- | --- |
| Current cell, self-reported | WordPress **7.1.4**, PHP **8.3.33**, stylesheet `chilltheme-test`, template `nextora`, theme version `1.0.0` (`cand-668d807d-current/versions.json`, `version-check.txt`) |
| Routes | **9/9 exactly as expected** — front page, posts index, archive, search with results, search with no results, single, studio, page, and a 404 for a missing path; child-owned classes on all; `nextora` and `chilltheme-test` the only theme asset sources; no remote demo URL; no notice/warning/deprecation text; all four child assets 200 (`routes/index.tsv`, `analysis.json`, `contract.txt` — **121 PASS / 0 FAIL**) |
| The candidate's own assertions, over HTTP inside the cell | **118 checks, 0 failures** (`assertions-http.txt`) — the 13 failures seen earlier were assertion bugs, and the two found after that were fixed too |
| `theme.json` merge, at runtime | theme-origin palette = all **7** parent slugs; spacing `theme` bucket = `10–60`; font families = all **6** parent slugs; both pattern categories; 9 child patterns (`theme-json.json`) |
| Block registration, at runtime | three blocks, `apiVersion 3`, category `chilltheme-test`, render callback present, `spacing: {padding, margin}` (`versions.json` → `block_facts`) |
| Viewports | real Chromium: 1440 / 768 / 390 / 320 and a 720 px-at-dpr-2 model of 200 % zoom |
| CASE 1 (all colour controls empty, no image) | passes everywhere — `hero-default` 19.8 / 7.81 / 6.92:1, `grid-default` 19.8 / 7.81 / 7.1:1, `band-default` 19.8 / 19.8:1. **The white-on-white class of defect is not reproducible at this digest.** |
| CASE 2 (media accessibility) | `hero-media` and `band-media` expose a labelled `role="img"` with **no `aria-hidden` ancestor**, at all four widths and the zoom model — S2 confirmed fixed by measurement |
| Other colour states | `light` (surface #f4f4f4), `literal` (#123456), `empty` (theme default), `dark` (contrast) and `media` (image + overlay) all pass AA for heading, content and link |
| Spacing, tuned instance | section padding 96/96 px (authored 6rem) and grid gaps 24/32 px (authored 1.5rem/2rem) at every viewport; theme-default sections 120/56/69.12/64.8 px responsive padding |
| Adjacency | rhythm comes from the sections' own padding (hero 112 bottom → next 72 top, etc.) and all **section** margins are `0px` — but a **24px visual gap** remains between the boxes, which is the wrapper margin recorded as I14 |

**Three real defects QA found that the coordinator's own pass did not** — all measured, all in the
colour layer, all relayed to the builder:

| # | Severity | Finding | Evidence |
| --- | --- | --- | --- |
| I15 | HIGH | A **partial-alpha** section background (`backgroundColor: rgba(18,52,86,0.5)`) leaves the derived content at `rgb(82,82,82)` on the composited `rgb(136,153,170)` = **2.68:1** (heading derives 6.78:1 and passes): the derivation reads the stored colour without compositing its alpha over the surface beneath. | `contrast-report.txt:65,72`; `browser/w1440.json` → `hero-alpha` |
| I16 | HIGH | **A card inside a dark section is white-on-light**: `grid-dark` resolves the section correctly (19.8:1) but the card keeps its light surface, so the card copy and its link measure `rgb(255,255,255)` on `rgb(244,244,244)` = **1.10:1**. The card is its own surface and must derive its own pair. | `contrast-report.txt:104,108`; `browser/w1440.json` → `grid-dark` |
| I17 | MEDIUM | The **focus ring** is `rgb(127,77,62)`: 6.92:1 on a light section but **2.86:1** on the dark section and **2.37:1** on the alpha surface, both below the non-text 3:1 threshold; the anchor's computed `outline` is `3px none`, so the visible indicator is a box-shadow ring. | `browser/w1440.json` → `*.focus` |

Two further observations, recorded rather than asserted: **no `<header>`/`<footer>` landmark exists on
any route** (the parts render as `<div>` because the references pass `tagName: "div"`, while the parent
uses `header`/`footer`), and at a 320 px viewport several inner elements extend past x=320 while
`documentElement.scrollWidth` stays *below* the viewport — so QA could not decide whether a reader sees
clipping and neither could the coordinator.

### QA's unrun list, verbatim in substance

- **The minimum cell** (WP 6.4.14 / PHP 8.1): the instance booted and answered 200, but every PHP step
  inside it returned "There has been a critical error on this website" — the same symptom the
  coordinator hit independently. Attribution is below.
- **The style-variation cell.** QA drove each `styles/color-scheme/*.json` through the
  `wp_global_styles` path and the write reported success, but the merged palette and the rendered
  colours were unchanged, so no variation measurement exists. An authenticated Site Editor/REST path
  was never established. **This is the one plan acceptance cell that no agent completed**, and it is
  exactly the cell that would test the self-inverting role pairs.
- **The block-editing exercises** (insert, repeater Add/Duplicate/Done/Cancel/Escape/reorder/remove,
  focus return, live region, undo/redo, save → reload → re-save): **not attempted by QA** — no admin
  session was established in its cell. The coordinator ran them separately in a logged-in instance;
  see the block-editing exercise table in §12.7. Undo/redo specifically remains unrun.
- **Visual confirmation at 320 px**: geometry only; the session could not read screenshots back for a
  visual verdict.

### Minimum-cell attribution — a parent defect, not the theme's

**Resolved, and the answer is that the blocker is the read-only parent.** QA captured the trace and the
coordinator verified it against the source:

- `nextora/inc/assets/assets.php:203` calls `wp_script_modules()` inside an `after_setup_theme`
  callback, on the `wp_is_block_theme()` path, with **no `function_exists()` guard**.
- `wp_script_modules()` is `@since 6.5.0` (`wp-includes/script-modules.php:23`; docblocks at `:5`,
  `:17`, `:37`, `:81`).
- On a WP 6.4.14 / PHP 8.1 cell the instance installs and serves **HTTP 200**, then every block-theme
  request dies with `Fatal error: Uncaught Error: Call to undefined function wp_script_modules()`;
  the blueprint's third step aborts, so no assertions, routes or fixtures exist for that cell
  (`qa/evidence/cand-min-debug-php8.1-wp6.4/cell.log:12-20`).

A theme-free 6.4.14 / PHP 8.1 instance boots and is ready, so the failure is on the theme path and not
in Playground. The parent declares `Requires at least: 6.4` and cannot run on it — a **parent defect,
reported and never fixed**, added to the parent-defect list in §12.9.

The child's disposition, which is a deliberate decision rather than an omission:

| Decision | Reason |
| --- | --- |
| `Requires at least: 6.4` stays; `Tested up to` stays absent | the child's own code is not the blocker, and the suite's rule is that a declared minimum is **never raised silently** — the owner moves it with the fact in hand |
| The constraint is recorded in the child's README | the parent anchor, the unguarded call, the version it needs, the fatal text, and the **effective floor of WordPress 6.5** until the parent guards it |
| No shim | defining a `wp_script_modules()` polyfill from a child, or filtering the parent's callback, would work around a parent defect — and a theme polyfilling a core function is worse than an honest constraint |
| The 6.4 cell is recorded as **unrun, cause a parent defect** | matches the plan's contingency: record the self-reported versions, and never claim an unrun baseline |

Consequence for the deliverable, stated so it is not buried: **this child cannot be used on
WordPress 6.4** while its parent is unpatched, and the declared minimum is therefore a declaration the
pair does not honour. That is the parent's defect surfacing through the child, and it is the one place
in this run where an acceptance cell could not be run for a reason outside the candidate's control.

### The self-inverting pair, verified through the switcher's own mechanism

The **Site Editor variation path could not be established** in the disposable instance — neither QA nor
the coordinator could make a `wp_global_styles` write change the merged palette, even though the post
was written with midnight's palette and the write reported success. That cell is **unrun**, and the
reason is recorded rather than papered over.

The *property* that cell exists to test — preset-backed colours follow a palette change, and every
resulting pair still passes — was verified a different way, through the mechanism the suite documents
for the visitor path: inline custom properties on `<html>` (exactly what `applyTheme()`/`applyFont()`
write). Measured on the front page at 1440:

| Section | Default palette | Composed surface | Heading | Content |
| --- | --- | --- | --- | --- |
| architectural-hero | dark band | `rgb(10,10,10)` | white, 19.80:1 | white, 19.80:1 |
| habitat-grid (services) | light band | `rgb(255,255,255)` | near-black, 19.80:1 | `rgb(82,82,82)`, 7.81:1 |
| habitat-grid (habitats) | light band | `rgb(255,255,255)` | near-black, 19.80:1 | `rgb(82,82,82)`, 7.81:1 |
| impact-band | dark band | `rgb(10,10,10)` | white, 19.80:1 | white, 19.80:1 |

With the dark-scheme preset **values** applied (`base #0c1222`, `contrast #e8edf5`, `surface #151d2e`,
`paragraph #94a3b8`):

| Section | Composed surface | Heading | Content |
| --- | --- | --- | --- |
| architectural-hero | `rgb(232,237,245)` — **inverted to light** | dark, **15.87:1** | dark, **15.87:1** |
| habitat-grid (services) | `rgb(12,18,34)` — **inverted to dark** | light, **15.87:1** | `rgb(148,163,184)`, **7.28:1** |
| habitat-grid (habitats) | `rgb(12,18,34)` | light, **15.87:1** | **7.28:1** |
| impact-band | `rgb(232,237,245)` | dark, **15.87:1** | dark, **15.87:1** |

Every surface inverted, and **every pair still clears AA** — which is the property the section roles were
written for: a section selects an *offsetting pair of the same roles*, so the scheme change moves the
values and the pair survives. The alternative the plan's first wording implied (pinning a light value
for dark sections) would have produced dark-on-dark the moment the scheme inverted.

Still explicitly **unverified**: the same test with a **custom literal** foreground set (the
"literals stay fixed" half), and the Site Editor's own persistence path.

### Coordinator's browser pass (headless Chromium, current tree)

Every route, at 1440 px, fetched in a real browser against a disposable Playground instance whose
cell self-reports WordPress 7.1.4 / PHP 8.3.33:

| Route | `<main>` | `<h1>` | Horizontal overflow | Broken images | Failed requests | Console errors |
| --- | --- | --- | --- | --- | --- | --- |
| front page `/` | 1 | 1 | none | 0 | 0 | none |
| posts index `/journal/` | 1 | 1 | none | 0 | 0 | none |
| a single post | 1 | 1 | none | 0 | 0 | none |
| a generic page | 1 | 1 | none | 0 | 0 | none |
| Studio (`page-studio`) | 1 | 1 | none | 0 | 0 | none |
| search with results | 1 | 1 | none | 0 | 0 | none |
| search with no results | 1 | 1 | none | 0 | 0 | none |
| 404 | 1 | 1 | none | 0 | 0 | none |

The child stylesheet, script and both starter images all returned **200** from
`wp-content/themes/chilltheme-test/`, and nothing outside the child requested the sample child or a
remote host. The front page and the Studio page carry 7 and 6 `chilltheme-test-scope` nodes
respectively (the composed sections); the core-only surfaces carry 1.

Widths and zoom, measured on the front page with geometry and computed styles:

| Condition | Horizontal overflow | `<main>` / `<h1>` |
| --- | --- | --- |
| 1440 px | none | 1 / 1 |
| 768 px | none | 1 / 1 |
| 390 px | none | 1 / 1 |
| 320 px | none | 1 / 1 |
| 200 % zoom proxy (720 CSS px viewport at `deviceScaleFactor: 2`) | none | 1 / 1 |

Computed colour pairs on the front page, every state passing WCAG AA:

| Section | Surface (composited) | Heading | Ratio | Content | Ratio |
| --- | --- | --- | --- | --- | --- |
| architectural-hero (dark band) | `rgb(10,10,10)` | `rgb(255,255,255)` | 19.80:1 | `rgb(255,255,255)` | 19.80:1 |
| habitat-grid (services, light) | `rgb(255,255,255)` | `rgb(10,10,10)` | 19.80:1 | `rgb(82,82,82)` | 7.81:1 |
| habitat-grid (habitats, light) | `rgb(255,255,255)` | `rgb(10,10,10)` | 19.80:1 | `rgb(82,82,82)` | 7.81:1 |
| impact-band (dark band) | `rgb(10,10,10)` | `rgb(255,255,255)` | 19.80:1 | `rgb(255,255,255)` | 19.80:1 |

Both dark bands resolve to **roles** (`--wp--preset--color--contrast` background,
`--wp--preset--color--base` foregrounds), not literals, so the pair inverts with a scheme change.

**What the browser pass could not verify, stated plainly:**

- **The theme's own focus ring on its anchors — measured, and it fails.** A script `focus()` does not
  trigger `:focus-visible`, so an early reading of `rgb(0,95,204)` was the browser's own ring and is
  discarded. QA then ran a decisive A/B with the keyboard and with injected `span[tabindex=0]`
  targets: **non-anchor focusables get the surface-derived token** (19.80 / 19.80 / 18.00 / 6.78:1 on
  the dark, white, `#f4f4f4` and alpha surfaces — all pass), while **`a.chilltheme-test-button` and
  `a.chilltheme-test-link` get `rgb(127,77,62)` in every context** (6.92 / 6.29 / **2.86** / **2.37** /
  **1.84**:1), even though the token resolves correctly on those same elements. The override is
  anchor-specific and comes from outside the child: the child's built CSS contains exactly one
  `outline-color` declaration, and `#7f4d3e` is `--wp--preset--color--primary`, which the parent's
  anchor-focus family emits. Recorded as an **open defect** handed to the builder, not a pass.
- **Pixel sampling behind the text on an image-backed band**, which gate §8 as written requires for
  that surface to count as verified. The candidate measures the composited worst case from the overlay
  instead — a defensible subset, but not the gate's requirement, so that surface is
  **partially verified**.
- **The site-style-variation swap and the visitor-preset swap** on computed styles, through the Site
  Editor's own persistence path. §12.7 records both what was established instead and why the cell is
  unrun.

### The block-editing exercise (coordinator, real editor, logged-in disposable instance)

Driven in Chromium against `wp-admin/post-new.php` on a disposable instance (WP 7.1.4 / PHP 8.3.33),
logged in as the Playground admin. **158 block types registered**, including
`chilltheme-test/habitat-grid`.

| Check | Observed |
| --- | --- |
| Insert the block programmatically | inserted with **3 seeded cards** |
| Inspector panels | Layout, Cards, Colours, Section rhythm, Advanced |
| Live region | `[role="status"]` present and **mounted before any mutation**, empty initially |
| "Add card" | cards **3 → 4**, announced **"Cards: one item added."** — so Add inserts the record immediately, as `editor-and-repeaters.md` §3 prescribes |
| Escape after Add | cards stay **4** — Escape discarded only the draft field edits and kept the inserted record; focus returned to `BUTTON.chilltheme-test-repeater__row-main` |
| The repeater dialog | **exactly one**, frame `components-modal__frame has-size-medium chilltheme-test-repe…`, heading "Edit Survey before drawing", header **Cancel / Done / Close**, fields `h3[aria-label="Title"]` and `p[aria-label="Copy"]` (`contenteditable`), per-colour palette rows each with a **Clear** action; focus lands inside |
| **Done** | typed "Winter survey" into the Title, clicked Done → **persisted** in `attributes.cards[3].title`, dialog closed |
| **Cancel** | typed "SHOULD-NOT-PERSIST", clicked Cancel → **not persisted**, dialog closed |
| Row action menu | exactly **Move up, Move down, Duplicate, Remove**, in that order, each with a **distinct label** (not one shared `aria-label`) |
| Boundary states | first row's **Move up disabled**, last row's **Move down disabled** |
| Duplicate | 4 → 5, announced **"Cards: one item duplicated."** |
| Remove | 5 → 4, announced **"Cards: one item removed."** |
| Save → reload → re-save | saved as draft with 0 errors; after reload **4 cards restored** with the committed titles (`Survey before drawing`, `Build with what grows`, `Detail for air and water`, `Winter survey`), **0 invalid-block notices**, 0 `has-warning`/`is-invalid` classes; re-save clean |

That closes the plan's block-editing item, and it is the strongest single result in the run: the
semantics the previous field test had to prescribe after a user complaint — insert-then-edit, one
dialog, distinct labels, live announcements, focus return, Done-commits/Cancel-discards — all behave
as written on the first inspection.

**Not exercised in the editor**: the media picker (`MediaUpload`) and the colour pickers' visual
interaction; those were verified at the render and computed-style level instead.

### Post-change evaluation legs

Recorded in §12.6: four legs (two prompts × unconstrained and constrained), all run against the
edited suite with the frozen invocation. Three reproduced the pre-change decisions; both contrast legs
changed behaviour on the one point where the baseline text pointed the wrong way.

### Plan items not exercised by anyone in this run

- **The minimum cell (WordPress 6.4.14 / PHP 8.1)** — the *availability* of the cell is verified
  (the CLI resolves `--wp=6.4` to `wordpress-6.4.14.zip` and boots it; `--wp=6.4.3` is not a
  fetchable artifact), but the candidate itself has not been run in it in this record. `Tested up to`
  is therefore still absent from `style.css`, which is the correct consequence.
- **200 % zoom, and the four-width overflow sweep (1440/768/390/320)** — require a browser; this
  record claims no measurement of either.
- **The block-editing round trip inside the editor** (insert → edit rich content → Add/Duplicate/
  Done/Cancel/Escape → save → reload → re-save with no invalid-block warning) — source-level
  semantics were read and match `editor-and-repeaters.md`, and the renderer path is asserted, but the
  editor leg itself is unverified here.
- **Pixel sampling behind text on the image-backed band**, which gate §8 as written requires for an
  image surface to count as verified. The candidate measures the composited worst case from the
  overlay instead, which is a defensible subset — so the honest status of that surface is
  **partially verified**, exactly as the reviewer noted.
- **The site-style-variation swap and the visitor-preset swap** on computed styles.

## 12.8 Reviewer verdicts

The reviewer's **delta verdict** after the builder's resolution pass: all six open findings verified
resolved in source, with the skill-omission classification preserved for the fixture-hygiene gap
("this fix came from review rather than from an instruction").

| ID | Finding | Verdict |
| --- | --- | --- |
| 1 | Test fixtures created inside the shipped theme (`tests/BlockRegistrationTest.php`) | **resolved** — fixtures now built under `sys_get_temp_dir()` with a pid+random suffix, the registrar pointed at them through an overridable `get_stylesheet_directory()`, permissions restored before cleanup, globals restored in `tearDown`; a positive control plus orphan and manifest negatives added. The README claim is now true. |
| 2 | Two false claims in the `habitat-grid` reuse decision | **resolved** in both the docblock and the README |
| 3 | Focus ring derived from the content colour (could fall below 3:1) | **resolved** — own surface-derived token, reported at the 3:1 non-text threshold; this also removed a regression a static `contrast` ring would have caused on the dark band |
| 4 | `media-control.tsx` enumerated the sample's block names | **resolved** |
| 5 | The spacing grammar docblocks understated the accepted input | **resolved** |
| 6 | `cardStyle()` assigning empty custom properties where PHP drops them | **resolved** |

Post-pass integrity, re-verified by the reviewer against the new artifact: 115 files; all 12 manifest
pointers resolve; each `index.asset.php` still exactly the 14 handles with a `1.0.0.<hash>` version and
no timestamp; **the three bundle hashes are unchanged from the freeze**, which independently shows the
editor bundles rebuild byte-identically; zero unresolved `require("@wordpress/…")`; zero remote URLs in
shipped content; `.phpunit.cache/` gitignored.

The reviewer also independently detected the §6 skill edits and tagged every rule citation in its audit
`[baseline]` or `[post-edit]`, which is why that audit's conclusions are usable even though the text
moved mid-run. Its two source-vs-skill observations (the bundled second React in both themes; the
`wp_set_script_translations()` wording) are recorded as T11 and T6.

### The skill-edit review (the plan's closing check)

A separate read-only pass over the 12 uncommitted files, against the parent and sample. Verdict: **the
five rules the plan asked for are present, correctly derived, and every one is backed by a live-source
fact the reviewer re-derived independently** — with three quoted numbers wrong, two provenance tags
mis-tagged, and one pre-existing sentence contradicting a new rule. All of it was acted on, not filed:

| What the review found | Disposition |
| --- | --- |
| "22 per-block colour resolvers" — actually **25** (`nextora_blc_`, `nextora_chart_`, `nextora_thli_` were missed by a tab-anchored regex) | corrected in three shipped places; the coordinator re-counted independently and got 25 |
| "20 declare `blockGap`" — 20 declare the **key**, 19 enable it (`advanced-button-button` sets `false`) | corrected in three places |
| the overlay enumeration was wrong for 3 of its 5 blocks, and there are **three** opacity scales (0.3, 0.4, 40) not two | rewritten in both files |
| the sample child's content-hash block is at `:41-57`, not `:51-57` | corrected |
| **the new shared-resolver rule contradicted two surviving sentences** — "Reuse the real signatures; do not write a parallel resolver" in both `nextora-design-system/SKILL.md` and `presets-and-switching.md` | both rewritten to "reuse the **behaviour**, implement it as one shared theme helper" — this was the one *correctness* defect in the pass, because a reader could follow either instruction |
| the three-row spacing table existed in full twice | the SKILL now keeps the owner names and points at §3.2 for the emission details |
| the WCAG thresholds are restated in four files | left as is: they agree, they are an external standard, and each file needs the number to be usable — recorded as a known drift risk |
| two `**[parent contract]**` tags were wrong (the parent's *patterns* follow the rhythm convention; nothing is code-enforced) | retagged `**[parent contract, by observation]**` |
| `templates`/`block-contract` said scoped colours are emitted "on the wrapper" | corrected to "on the block's own styled element — the inner surface when the wrapper already carries core spacing", which is what the candidate does and what §3.2's one-owner rule implies |
| one bare `references/quality-gates.md` in the *previous* field test did not resolve from `evals/` | path fixed. **No claim in the prior field test was altered**, only the path form |
| `SKILL.md` sizes 128 / 157 / 193 / 195 / 203 lines — all inside the suite's own 250-line rule, but headroom is thin | recorded: the next pass touching design-system or site-composition should move prose into the reference rather than grow the entrypoint |

### Delta verdict on the candidate's last pass

**All six mechanisms real in the code, reproduced independently**: alpha compositing
(`colour.php:378-414` plus the TS mirror), the unauthored card surface (`habitat-grid/style.css:21-23`),
the focus ring (`helpers.php:842-847`, widened to `:focus` so it is measurable by script), the margin
neutralisation (`25-sections.css:11-28`, with the root cause named), scoped `box-sizing`
(`10-base.css:22-35`, quoting the 320 px measurement), and the 6.4/6.5 constraint documented with
**no shim** (grep for `wp_script_modules`/`WP_Script_Modules` over the child hits only PHPStan stubs).

Its residual findings, and where each was acted on: **D2** stale CHANGELOG count, **D3** a quoted
`2.68:1` where the formula gives `2.71:1`, **D4** the margin neutralisation keyed on a hand-written
class that only 2 of 9 templates carry — all three sent to the builder. **D5/D7** are wording cautions
(don't generalise "19.80 in all three contexts"; the card's unauthored surface is now transparent, a
product choice worth a README line). **D6** records that `blueprint.json` now pins `wp: 7.1.4`.

**D1 is the finding that matters for how this record should be read**, and it is recorded in full: the
candidate ships **no browser or geometry tooling**, and its smoke file asserts none of the six runtime
numbers. So the six *mechanisms* are verifiable from the deliverable; the six *results* are
measurements taken by QA and by the coordinator in disposable instances and recorded here, not checks
anyone can re-run from the repo. Three of them — boundary deltas, 320 px geometry, focus contexts — are
currently nobody's automated check.

### Where the two agents disagreed, and how it was settled

QA's delta pass reported the focus ring as a constant `rgb(127,77,62)` in every context, and concluded
the ring was **not** surface-derived. The reviewer's code review concluded the opposite. The
coordinator settled it from source: `rgb(127,77,62)` is exactly `--wp--preset--color--primary`, and the
child's ring chain (`10-base.css:156-166` → `--chilltheme-test-focus` → `00-tokens.css:34` /
`25-sections.css:47` / `helpers.php:847`) contains no `primary` anywhere; the only `primary` ring in the
stack is the **parent's** header-nav rule (`nextora/assets/css/app.css:2419-2423`), whose classes do not
appear on the rendered page because the child's header uses `core/navigation`. So QA's number is not
attributable to the child's rule, and a targeted re-measure on `a.chilltheme-test-button` and
`a.chilltheme-test-link` was requested.

The general lesson, recorded because it recurred three times in this run: **an unqualified "the
computed value is X" is not evidence until it names the selector it was read from.** The builder lost a
smoke run to assertion bugs of exactly this shape, QA lost a finding to it here, and the coordinator's
own first digest definition lost stability to it by hashing a test cache.

## 12.9 What this run does not claim

- No causal comparison against a no-skills run, and no claim that the pre-change legs "failed": on this
  model and these two prompts they largely succeeded (see §12.6).
- No claim that a passing post-change leg is *caused* by the new text rather than by the same
  first-principles reasoning the pre-change legs displayed.
- Compatibility is claimed only for the cells actually run and self-reported; the declared
  `Requires at least: 6.4` / `Requires PHP: 8.1` stay declarations until the runtime cells say
  otherwise.
- The parent defects observed while working (an empty `blocks/box-content/` directory that the parent
  registrar still tries to register; remote `pub-*.r2.dev` demo images in parent templates and
  patterns; no shared colour resolver; two incompatible overlay opacity conventions; and — found by
  this run's minimum cell — an unguarded `wp_script_modules()` call at
  `nextora/inc/assets/assets.php:203`, `@since 6.5.0`, which fatals on the WordPress 6.4 the parent
  itself declares) are **observations with reproductions**, reported here and never fixed. Neither
  source theme was edited by any agent, and none of these was worked around with a shim.

## 12.10 Verdict

**The suite produced a substantially correct deliverable, and the defects that reached it were almost
all in the layer the suite does not describe.**

- **What held without intervention.** The child contract (bootstrap order, the 25/30/`wp_head` 1/
  `after_setup_theme` 20 hook table, non-colliding output basenames, version parity, the guarded
  autoloader, child-before-parent discipline), the registrar and both category coercions, the block
  contract (`apiVersion: 3`, built asset pointers, `save: () => null`, the 14-handle manifest, no
  unresolved `require`, no second React), the composition rules (thin template plus one hidden pattern,
  the eight core templates, only `page-studio` declared, no remote demo media, no `theme` pinning), the
  preset merge (all 7 palette slugs, all 6 spacing slugs and all 6 font families survive because
  `settings.spacing`/`settings.typography` were omitted rather than re-declared), and the repeater model
  (immutable draft, one dialog, always-mounted live region, focus return, no persisted UI state) all
  landed correctly on the first attempt and survived an independent review, a build, and two runtime
  cells.
- **Where the defects actually were: the colour and spacing layer, every time.** Nine implementation
  defects were found; eight of the nine live in contrast or spacing — the phantom surface (S1), the
  swallowed media alt (S2), the mislabelled card control (S3), the missing landmark (I13), the unowned
  24 px inter-section gap (I14), the alpha-composited surface (I15), the nested card pair (I16), and the
  focus ring (I17). That is the same conclusion the first field test reached from a different project,
  and it is why the changes in §12.5 are concentrated there.
- **The evaluation texts were written but did not discriminate the way the plan assumed.** Both
  pre-change legs already met substantially every expected decision on this model; the one real delta is
  the shared child-owned resolver, where the baseline text actively pointed the wrong way and both
  post-change legs changed behaviour in the new rule's own words. Reported as such, not as a
  correction that repaired a failure it did not repair.
- **Two defects only a person or a screenshot would have caught, until harnesses were built.** The
  missing `<main>` landmark and the 24 px strip between bands are invisible to every source-level check
  the suite describes; the coordinator found them by fetching routes and measuring geometry, and QA
  found three more colour defects by measuring state combinations the smoke file never enters. `D1`
  stands as the honest caveat: none of those measurements is reproducible from the repository, because
  the theme ships no browser tooling.
- **Numbers.** 12 implementation defects opened across the run (I5–I7, I12–I17), all closed or
  re-classified; 16 skill defects corrected (T1–T16), four of them stale claims that contradicted the
  live sources; 4 pre-change and 4 post-change evaluation legs; 2 reviewers, 1 builder, 1 QA agent; 1
  parent defect discovered and left unfixed by design.
- **What the corrections are worth.** Fourteen of the sixteen changes make an existing practice
  *explicit and checkable* — they codify what a good agent on a good day already did, with the
  thresholds, the ownership table and the acceptance steps attached. Two change behaviour: the
  shared-resolver rule (demonstrated) and the focus-indicator rule (demonstrated in the candidate).
  That ratio is the honest summary of how mature this suite is: the architecture guidance is strong, and
  what it lacked was the measurement discipline for the one layer where "it looks fine" is not a check.

**Bottom line for a reader deciding whether to use these skills:** the suite is worth loading for a
Nextora child theme, and the new §3.1/§3.2/§5.2 material is the part that will save the most time,
because contrast and section rhythm are where every defect in two independent field tests has come
from. Do not expect it to catch a missing landmark, a 24 px strip, or a nested card pair: those are
runtime measurements, and the suite now tells you to take them but cannot take them for you.

