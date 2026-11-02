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

## Using these skills

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

### Give the agent your theme roots

Every skill starts by locating the real sources, and it would rather be told than
guess. Put the paths in the request:

```text
Source roots (read-only; do not modify):
- parent theme: /path/to/wp-content/themes/nextora
- my child theme: /path/to/wp-content/themes/<child>
- sample child, structural reference only: /path/to/wp-content/themes/<sample>
```

If the parent is genuinely unavailable, say so. The skills are written to report
the missing prerequisite instead of inventing an anchor — and a child-theme
request cannot be answered responsibly without the parent source.

### Worked examples

Each command is a file plus a prompt. Add the source-roots block above, and add
`do not write files` when you only want a design to review.

**Plan a whole child theme** — the router dispatches to four specialists:

```bash
omp -p --no-session @skills/nextora-ecosystem/SKILL.md \
  "Create a preset-switchable Nextora child named nextfolio: an editable team block and a landing-page pattern. Output a file map and implementation outline only."
```

**Scaffold the child itself:**

```bash
omp -p --no-session @skills/nextora-child-themes/SKILL.md \
  "Output the style.css header, functions.php and the asset-hook snippets for nextfolio. Do not write files."
```

**Add a block with a repeatable list:**

```bash
omp -p --no-session @skills/nextora-gutenberg-blocks/SKILL.md \
  "Add a dynamic block nextfolio/team to my child: rich heading and body, and members managed by a list plus one modal, each member able to carry nested links. Show editor and render.php excerpts first."
```

**Make appearance follow the visitor's choice:**

```bash
omp -p --no-session @skills/nextora-design-system/SKILL.md \
  "My child must follow visitor colour and font switches. Show the theme.json font families, the :root role defaults and the preset files, and explain what changes when NEXTORA_COLOR_SCHEME_SWITCHER is absent."
```

**Compose a page from what already exists:**

```bash
omp -p --no-session @skills/nextora-site-composition/SKILL.md \
  "Compose a nextfolio landing template and an insertable pattern from existing blocks — do not create a new block."
```

**Review instead of build:**

```bash
omp -p --no-session @skills/nextora-gutenberg-blocks/SKILL.md \
  "Review my child's blocks/team/edit.tsx. Report only deviations from skills/nextora-ecosystem/references/quality-gates.md, each with file:line and the fix. Do not edit."
```

### Combining skills

Some requests need two entrypoints, and the router applies the pairing for you:

- New child **that must follow presets** → `nextora-child-themes` + `nextora-design-system`.
- New or restyled block **whose look matters** → `nextora-gutenberg-blocks` + `nextora-design-system`.

Load them in the same session rather than one after another, and let
`nextora-design-system` win on any appearance question. When in doubt, start from
`nextora-ecosystem`: it makes the same decision and links both shared references.

### Ask for the output shape you want

The entrypoints decide what is *correct*; your request decides how much comes
back. Useful endings:

| You want | Say |
| --- | --- |
| A plan to review first | "Output a file map and implementation outline only." |
| Code with no side effects | "Do not write files." |
| A diagnosis | "Report only deviations, each with file:line and the fix." |
| Options before committing | "Give the two or three viable shapes with tradeoffs, then recommend one." |
| Runtime proof steps | "Also give the exact commands and browser checks to prove this on a disposable install." |

### What the skills will and won't do

| They will | They won't |
| --- | --- |
| Read your actual theme and match its existing conventions | Edit the Nextora parent or the sampled child theme |
| Reuse an existing block, pattern, helper or token rather than inventing one | Fabricate a function, path or source anchor |
| Say which rule comes from the parent and which is this suite's own requirement | Promise WordPress-version compatibility from a `style.css` header |
| Report a missing prerequisite instead of guessing | Ship a starter theme, generator or installer |
| Hand you the downstream acceptance recipe and the target's own commands | Claim a build or browser result it did not observe |

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

## Troubleshooting

| Symptom | Why | What to do |
| --- | --- | --- |
| It invented a helper, slug or path that does not exist | It never saw the real source | Paste the parent (and child) roots. If the parent really is unavailable, expect an explicit "prerequisite missing" answer rather than code |
| It claims WordPress 6.4 is supported | `Requires at least` is a declaration, and both themes ship `theme.json` v3 | Ask for the runtime check in `skills/nextora-ecosystem/references/quality-gates.md` §7 and treat the minimum as unproven |
| It wrote files when you wanted a design | Nothing told it to stop | Add "output only, do not write files" |
| It copied the sample child's palette, fonts or scope classes | The sample was named without a boundary | Say "structural reference only — this child gets its own slug, palette and scope class" |
| It answered a broad request with a single specialist | The router was not loaded | Start from `skills/nextora-ecosystem/SKILL.md`, or name the specialists you want |
| It gave you a design where you wanted a running result | This suite ships no runtime and runs nothing for you | Point it at a disposable WordPress install plus the §8 acceptance recipes in `skills/nextora-ecosystem/references/quality-gates.md`, or accept the answer as a design |
| It changed an attribute shape in an existing block | The request did not say existing content must survive | Ask for a `deprecated`/migration entry and the save → reload → re-save check |

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
