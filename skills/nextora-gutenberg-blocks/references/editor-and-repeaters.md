# Editor UI and repeaters

The two list editors that exist in the sources, their exact commit semantics, and
the rules for new repeated-data fields. There is **no DataViews dependency** in
either theme; both editors are built from `@wordpress/components` primitives.

Read the sources before changing them:

| Implementation | Files (relative to the theme root) |
| --- | --- |
| Parent team section — ID-keyed, immediate patching | `blocks/team-section/edit.tsx`, `blocks/team-section/member-edit-form.tsx`, `blocks/team-section/member-utils.ts`, `blocks/team-section/types.ts`, `blocks/team-section/social-utils.tsx` |
| Child reusable list — positional, draft/Done/Cancel | `resources/ts/editor/repeater.tsx`, `resources/ts/editor/rich-text-control.tsx`, `resources/ts/editor/icons.tsx` |
| Child shared field helpers | `resources/ts/editor/media-control.tsx`, `types.ts`, `url-help.ts`, `block-preview.tsx`, `rich-text-overlay.tsx` — all imported by blocks |

`resources/ts/editor/colour-controls.tsx` also exists in the sample but nothing
imports it and no `block.json` declares the `sectionBg`/`sectionHeading`/
`sectionText` attributes it drives. Treat it as unused sample code, **not** as a
pattern to copy.

Related: `block-contract.md`, `../../nextora-ecosystem/references/quality-gates.md`.

## 1. Parent `team-section`: stable IDs, every keystroke persists

`edit.tsx` holds `editingMemberId` (a string or `null`) and derives the list from
attributes on every render:

```ts
const members = normalizeMembers(attributes.members);
const editingMember = editingMemberId
  ? members.find((m) => m.id === editingMemberId)
  : undefined;
```

| Action | Implementation | Semantics |
| --- | --- | --- |
| `setMembers(next)` | `setAttributes({ members: next })` | The list is the persisted attribute; there is no separate committed copy. |
| `patchMember(id, patch)` | `members.map((m) => (m.id === id ? { ...m, ...patch } : m))` | **Immediate patching** — each field change is written to the block attribute as it happens. |
| `addMember()` | `createMemberId()` → push a full default record → `setEditingMemberId(id)` | Inserts first, then opens the editor for the new record. |
| `removeMember(id)` | early `return` when `members.length <= 1`; filter by id; clear the editor if it was open | One member is the implicit minimum; removal is also disabled in the UI at that point. |
| `moveMember(id, delta)` | `findIndex`, bail when out of bounds, swap in a cloned array | Order is list position; there is no order field or drag-and-drop. |
| Duplicate | not implemented | Do not invent one for this block; it would change the list contract. |

`createMemberId()` uses `crypto.randomUUID()` when available and falls back to
`member-${Date.now()}-${random}`. `normalizeMembers()` preserves any non-empty
string `id` and otherwise falls back to `String(index + 1)`, so legacy content
saved before ids existed gets positional ids — which is why the row `key` and
`patchMember` both key off `id` in current content, and why a new list should not
copy this fallback.

The **Members** `PanelBody` in the inspector is the list: one row per member with a
photo thumbnail, the label `member.name || sprintf(__('Member %d'), index + 1)`, and
`Edit`, `Move up` (disabled at index 0), `Move down` (disabled on the last row) and
`Remove` (disabled when `members.length <= 1`, `isDestructive`) buttons, plus an
"Add member" button and a help line when the list is empty. Rows are keyed by
`member.id` — not by index.

The editor is **one** `Modal`, keyed off `editingMember`:

- `title`: `Edit: <name>` or `Edit team member`.
- `headerActions`: a single primary **Done** button that clears `editingMemberId`.
- `onRequestClose` clears `editingMemberId` too.

Because patching is immediate, closing *is* the commit — there is no draft, no
Cancel and therefore no rollback of field edits. `onRequestClose` and **Done**
behave identically. Do not add a Cancel button here expecting it to undo edits.

`member-edit-form.tsx` owns the field layout and calls `onPatch(patch)` for every
change; it never holds its own state:

- Photo: `MediaUploadCheck` + `MediaUpload` with `allowedTypes` from
  `TEAM_SECTION_MEDIA_TYPES`; `onSelect` patches `photoId` and `photoAlt` together
  (`media.alt ?? member.photoAlt`). The URL is derived at render time by
  `resolvePhotoUrl()` (attachment id first via a `useSelect` media lookup, then a
  stored `photoUrl`) — it is not written on selection. "Remove photo" patches
  `{ photoId: 0, photoAlt: '' }`.
- The **Photo alt text** field is only rendered when `photoId > 0`.
- Tags (`default` template only): an array of plain strings with "Add tag",
  per-row `Remove`, keyed `${member.id}-tag-${tagIndex}`.
- Social links behind a "Show social links" toggle: rows of `{ platform, url }`
  with a platform `SelectControl`, a URL `TextControl` using
  `getSocialPlaceholder()`, per-row `Move up`/`Move down` (disabled at the bounds)
  and a destructive "Remove link", plus quick-add preset buttons. Keys are
  `${member.id}-social-${linkIndex}`.
- Template-dependent fields: the bio and line clamp are hidden for the
  `overlay-social` template, and the clamp is hidden for `template-02`.

Each `Move up`/`Move down`/`Remove` clones the array before mutating
(`const tags = [...member.tags]`), so no stored record is mutated in place.

## 2. Child `repeater.tsx`: positional keys, one immutable draft

`RepeaterProps` is the whole contract: `label`, `items`, `fields`, `onChange`,
`createItem`, and optional `getItemLabel`, `addLabel`, `emptyLabel`,
`minimumItems` (default `0`), `renderItemExtras`. The fields it can render are
`text` (default), `textarea`, `select`, `toggle`, `rich` and `number` (a
`RangeControl` with `min`/`max`/`step` defaulting to `0`/`100`/`1`).

Keys stay **positional indexes**. The file's own rationale: items are plain
attribute records with no identity of their own, and adding an `id` field would
change the persisted attribute shape of every block already using the component.
The editor therefore keeps an immutable draft and writes it back **exactly once**,
on `Done`.

State: `status`, `openIndex`, `draft`, `insideModal`; plus `nested` from
`ItemEditorContext` and `inlineEditor = nested || insideModal`.

| Function | Behaviour |
| --- | --- |
| `openEditor(index, item)` | Records `openIndexRef.current = index`, `setDraft({ ...item })`, `setOpenIndex(index)`. |
| `updateDraft(patch)` | `setDraft((current) => current === null ? current : { ...current, ...patch })` — immutable merge. |
| `closeEditor()` | `setOpenIndex(null)` + `setDraft(null)`. **Cancel, Escape and the modal's close button all route here and never touch the committed list.** |
| `commitDraft()` | Guards `openIndex === null \|\| draft === null \|\| openIndex >= items.length` → `closeEditor()`; otherwise clones the array, replaces `next[openIndex] = { ...draft }`, calls `onChange(next)` once, then closes. |
| `move(index, offset)` | Bounds check → clone → swap → `onChange` → status. |
| `duplicate(index)` | `copy = { ...items[index] }`, `splice(index + 1, 0, copy)`, `onChange`, status, then `openEditor(index + 1, copy)`. |
| `remove(index)` | Filter by position, `onChange`, status. |
| `add()` | `createItem()`, append, `onChange`, status, then `openEditor(next.length - 1, item)`. |

### The Add/Duplicate boundary — copy this, do not "fix" it

**`add()` and `duplicate()` insert the new record *before* opening its draft
editor.** Cancel, Escape or the close button discard the subsequent field edits
but **retain that inserted item**; they do not roll back the list mutation. The
item is already in the attributes by the time the editor opens, so "cancel"
means "stop editing this record", not "undo creating it". Existing-item
**Edit → Cancel** leaves the committed record completely unchanged.

The stale-draft guard in `commitDraft()` matters: if undo removes the item under
the open editor, `openIndex >= items.length`, the draft is discarded and
`closeEditor()` runs instead of writing a sparse or wrong record.

### Chain of custody for the UI

- The status region is `<p className="nextsaas-repeater__status" role="status"
  aria-live="polite">`, rendered from the first render so assistive technology is
  already observing it. It shows either the minimum-items message (`_n()`,
  pluralised, sprintf'd with the list label and `minimumItems`) or the last
  mutation message (`added`, `duplicated`, `movedUp`, `movedDown`, `removed`).
- `atMinimum = minimumItems > 0 && items.length <= minimumItems`.
- One row button (`.nextsaas-repeater__row-main`) carries the sequence number and
  the label from `itemLabel()`, which prefers `getItemLabel`, then strips tags and
  decodes entities from `label`/`heading`/`title` via `plainLabel()`, then falls
  back to `Item %d`.
- **One `DropdownMenu` per row**, `label` = `sprintf(__('Actions for %s'), rowLabel)`,
  with `controls` in this fixed order: **Move up, Move down, Duplicate, Remove**.
  Bounds actions are `isDisabled: listLocked || index === 0 | last`; Remove adds
  `atMinimum`; every action also respects `listLocked`.
- **`listLocked = inlineEditor && openIndex !== null`**: while an inline nested
  draft is open, list mutations are locked to protect the positional target —
  including `Add`, which is `disabled={listLocked}`.
- Focus returns to the row: an effect keyed on `openIndex` reads the index out of
  `openIndexRef` and focuses `.nextsaas-repeater__row-main` at that position, which
  also covers the nested in-place editor. `Modal` returns focus on its own as well;
  doing it explicitly keeps the target deterministic.
- Nesting: `renderItemExtras(draft, update, index)` is wrapped in
  `<ItemEditorContext.Provider value>`, so a repeater rendered inside an item
  editor works inline instead of opening a **second dialog** (WordPress' `Modal`
  dismisses every other modal of its React context and modalises the document).
  A repeater dropped into somebody else's modal is detected with
  `root.closest('.components-modal__frame')`. **Never stack a second dialog.**
- The item `Modal` uses `size="medium"`, `focusOnMount="firstContentElement"` and
  `headerActions` holding **Cancel** (secondary) and **Done** (primary). The
  in-place nested editor renders the same two actions in its own footer row because
  it has no dialog header.
- Empty `items` plus `emptyLabel` renders the empty-state text; the Add button is
  always present below the list.
- `repeater.tsx` imports `./icons` and `./rich-text-control`. Adapting it means
  bringing those files (or equivalents) with it — do not copy the component alone.

## 3. Which model for new work

- **New repeated-data field** → the child draft model: compact rows with labels
  and one `DropdownMenu`, one `Modal` per item, immutable draft, single commit on
  **Done**, Cancel/Escape as discard-only, explicit minimum only when the content
  genuinely requires it, empty-state Add, focus return, accessible action names,
  pluralised live-region feedback.
- **Editing `team-section`** → keep its ID-keyed immediate-patch model. Do not
  convert it to positional keys, and do not convert a positional list to ids:
  quality-gate §1 makes both directions a breaking data change. When editing an
  existing block, **preserve its serialization and commit semantics**; a list that
  silently changed from "commit on close" to "commit on Done" changes what authors
  lose when they close a dialog.
- **Never persist UI state** — open index, draft, status text and lock flags stay in
  React state.
- All new objects are brand-new: `onChange` receives a new array of new item
  objects so no caller and no undo snapshot ever sees a stored record mutated in
  place.

## 4. Rich text, media and URLs

- `RICH_TEXT_FORMATS` is `['core/bold', 'core/italic', 'core/link']`.
  `RichTextControl` filters any caller list through that set, so a caller may
  **narrow** the formats and never widen them.
- `normaliseRichValue(value, multiline)` converts `\r\n?|\n` to `<br>` when
  multiline, collapses whitespace/newlines to a single space when not, and returns
  `''` for anything with no text after stripping tags and `&nbsp;`. Empty means
  empty: a cleared field never stores `<p></p>` or a lone `<br>`.
- The control uses `RichText` from `@wordpress/block-editor` with
  `inlineToolbar` (a `BlockControls` slot is unreachable from inside the block
  canvas or a modal), `disableLineBreaks={!multiline}`, and a `BaseControl`
  wrapper that supplies the visible label plus `aria-label`/`aria-describedby`.
  Keep core blocks' paste, undo and keyboard behaviour — do not build a bespoke
  editor.
- **The toolbar is a UX boundary, not a sanitisation policy.** There is no
  client-side KSES; PHP must still sanitize stored HTML on output
  (`nextsaas_rich()` or `wp_kses_post()`). A comment claiming otherwise is wrong.
- **Media**: reuse `resources/ts/editor/media-control.tsx` when it exists.
  Selecting or replacing writes `{ id, url, alt }` **together** from the attachment
  record so the three can never drift; alt stays individually editable through
  `onChangeAlt`; removal clears all fields. `kind` switches image/video, and an
  optional caption field is hidden when omitted. Labels are passed in translated by
  the caller, so the component owns no block attributes.
- **URLs**: reuse `resources/ts/editor/url-help.ts` (`urlHelp`,
  `structuralLinkHelp`, `urlHelpWithStructuralNote`) for any field whose value
  reaches `nextsaas_resolve_url()` in `inc/blocks/helpers.php`; the grammar is
  documented once so it cannot drift.
- **Types**: reuse `resources/ts/editor/types.ts` (`EditorEditProps`,
  `RepeaterFieldSpec`, `MediaValue`, `CtaValue`, `NavItemValue`) rather than
  redeclaring editor prop shapes per block.

## 5. Acceptance, on top of quality-gate §8

Add these checks to the shared blocks/repeaters recipe:

1. New list: **Add → Cancel** leaves the inserted record in the list and **Add →
   Done** commits the typed values. Duplicate behaves the same way.
2. Existing item: **Edit → Cancel** and **Escape** leave the committed record
   unchanged, and reopening shows the original values.
3. Undo while an item editor is open: confirm the draft is discarded and no
   malformed record is written.
4. Open an inline nested draft: confirm Add and every per-row action are disabled
   for the duration, and that no second dialog opens.
5. Confirm `Move up`/`Move down` are disabled at the list bounds and `Remove` is
   disabled at the minimum, with the pluralised live-region message announced.
6. Confirm focus lands back on the edited row after Done, Cancel and Escape.
