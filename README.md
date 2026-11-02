# Nextora skill suite

Agent skills for the Nextora WordPress block-theme ecosystem: child themes,
Gutenberg blocks, the design system and its optional visitor colour/font
switcher, and template/part/pattern composition.

This repository ships **Markdown instructions and source-backed references only**.
It contains no runtime scripts, no generators, no starter theme, no package
manager and no installer.

## Layout

```text
README.md
evals/
  scenarios.md
skills/
  nextora-ecosystem/
    SKILL.md                          # router: resolve, dispatch, shared gates
    references/source-map.md          # where every rule comes from
    references/quality-gates.md       # what must be true before "done"
  nextora-child-themes/
    SKILL.md
    references/child-contract.md
  nextora-gutenberg-blocks/
    SKILL.md
    references/block-contract.md
    references/editor-and-repeaters.md
  nextora-design-system/
    SKILL.md
    references/presets-and-switching.md
  nextora-site-composition/
    SKILL.md
    references/templates-and-patterns.md
```

Each `SKILL.md` is a self-contained entrypoint under 250 lines with an ordered
workflow, reference-loading instructions and completion checks. The longer
contracts live in the `references/` files next to it.

## Which skill to load

| Request | Skill |
| --- | --- |
| Create or bootstrap a child theme, child asset pipeline, child block registration | `skills/nextora-child-themes/SKILL.md` |
| Add or change a block: `block.json`, attributes, editor UI, `render.php`, view script, repeater, settings panel | `skills/nextora-gutenberg-blocks/SKILL.md` |
| Colours, fonts, spacing tokens, style variations, the visitor switcher, "must follow the palette" | `skills/nextora-design-system/SKILL.md` |
| Templates, template parts, patterns, page layouts, archives, post queries | `skills/nextora-site-composition/SKILL.md` |
| Anything spanning several of the above, or unknown scope | `skills/nextora-ecosystem/SKILL.md` (router) |

Two combinations are obligatory: a new child theme that must follow presets needs
`nextora-child-themes` **and** `nextora-design-system`; a new or restyled block
needs `nextora-gutenberg-blocks` **and** `nextora-design-system`. The router in
`nextora-ecosystem` applies these rules and links the two shared references.
Specialists are directly invocable — loading the router first is optional.

## How to use a skill

Point the agent at the `SKILL.md` you want and let it load the files that
entrypoint names, by relative path. From this repository root:

```bash
omp -p --no-session @skills/nextora-ecosystem/SKILL.md \
  "Create a preset-switchable Nextora child theme named nextfolio with an editable team block and a landing-page pattern."
```

For a narrow task, load the specialist directly — for example:

```bash
omp -p --no-session @skills/nextora-design-system/SKILL.md \
  "Explain how a scoped colour override survives a palette switch, and what the switcher's feature gate changes."
```

Any client that can load a Markdown file and follow relative links can use these
skills; the commands above are `omp` examples because `omp` is what was exercised
here.

### Install with the skills CLI

The repository is laid out the way the `skills` CLI expects
(`skills/<name>/SKILL.md`), so it installs directly from GitHub:

```bash
npx skills add miketropi/nextora-skills -l                     # list the five skills, install nothing
npx skills add miketropi/nextora-skills                        # install to the detected agent(s)
npx skills add miketropi/nextora-skills -s nextora-ecosystem    # install just the router
npx skills use miketropi/nextora-skills@nextora-design-system    # prompt for one skill, no install
```

`-a <agents>` targets specific agents, `-g` installs user-level instead of
project-level, `--copy` copies instead of symlinking, and `-y` skips the
confirmation prompt. Run `-l` first if you only want to see what is on offer.

Two limits worth stating plainly:

- **Discovery from a bare checkout is not automatic.** Pointing a client at this
  repository root is not the same as installing it — supply the entrypoint file
  explicitly, or install it with the CLI above.
- Only `omp` (direct file load) and the `skills` CLI (`-l`, listing) were
  exercised while authoring this suite. No other client was tested, and the
  repository itself installs nothing.

## How the skills resolve a theme

No skill embeds a machine path. Each one resolves the roots at run time:

- Parent (Nextora) = `get_template_directory()`, or the theme whose `style.css`
  declares `Theme Name: Nextora`.
- Child = `get_stylesheet_directory()`, or the theme whose `style.css` declares
  `Template: nextora`.
- Outside a WordPress install, the theme root is the directory containing the
  `style.css` that carries the theme header.

`skills/nextora-ecosystem/references/source-map.md` records the anchors and the
provenance of each rule (parent contract, sample convention, new-output
requirement, observed defect). It is a map, not a cache: every skill tells the
agent to reread the live sources before editing.

## Evaluation

`evals/scenarios.md` is an instruction-suite evaluation corpus: seven prompts
with their source fixtures, expected outputs and explicit failure conditions. It
is not a fixture WordPress application and not an automated scoring framework — a
human or agent runs the prompt and checks the observable result.

## Verification

What was actually exercised while authoring this suite (2026-11-02):

- **Structure.** Five `skills/*/SKILL.md` entrypoints with directory-matching
  `name` values, descriptions inside 1–1024 characters, every entrypoint under 250
  lines, every relative Markdown link (including cross-skill references) resolving,
  and exactly the layout above present with no stray files.
- **Source accuracy.** Every anchor, function name, slug, hook, priority, handle and
  preset path quoted in the references was checked against the live Nextora parent
  and the sampled child theme in two independent read-only reviews, one per source
  root. Fourteen inaccurate or unsupported statements were found and corrected
  before shipping.
- **Instruction loading.** All seven prompts in `evals/scenarios.md` were run
  against the real entrypoint files — `omp -p --no-session --no-skills
  --tools=read,glob,grep @skills/<name>/SKILL.md …` from this repository root — and
  each answer was inspected against its stated expectations. The two
  design-system scenarios were re-run after the corrections above.
- **Distribution.** `npx skills add miketropi/nextora-skills -l` against the
  published repository discovers all five skills with their authored names and
  descriptions — listing only, with no install into any agent directory.

**Not verified here:** nothing was built, installed or loaded in WordPress. No
generated child theme, block, template or pattern was rendered in a browser, and no
switcher or preset behaviour was observed at runtime. The acceptance recipes in
`skills/nextora-ecosystem/references/quality-gates.md` §8 exist to close that gap on
a disposable install.

## Non-goals

- No generated test fixture application, starter theme or theme installer.
- No edits to the Nextora parent or to the sampled child theme.
- No client-specific duplicate skill trees; the skills are portable Agent Skills
  Markdown.
