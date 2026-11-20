# Input and forms

This document owns every surface a person types into: the field, the keyboard that rises for it, what
happens when they paste or drop something, the composer, where a validation message goes, and how a
setting is changed. The feeling it protects is that typing costs nothing. The right keyboard is already
up, the right key is already in the corner, the first character lands the frame it is pressed, nothing
the person typed is ever lost, and no screen makes them press Save to find out whether it worked.

## Owns and defers

Owns: field primitives and the attributes each field declares; focus and the visual viewport; paste,
drop and their previews; the composer's typing behaviour; inline validation placement; settings,
defaults and live preview.

Defers: a field's one ring to `[components]` and a control's ring to `[accessibility]`; input type size and the
16px floor value to `[tokens]`; the keyboard-rise easing and the composer's growth transition to
`[motion]`; draft persistence across navigation, refresh and crash to `[resilience]`; label,
placeholder and error wording to `[copy-and-voice]`; camera, microphone and contacts prompts to
`[permissions-and-trust]`; upload validation and alt text to `[media]`; accessible names and live
announcements to `[accessibility]`; RTL and locale keyboards to `[internationalisation]`.

## Rules

### The field

1. **One field primitive.** `components/ui/input.tsx`, `components/ui/textarea.tsx`,
   `components/ui/select.tsx`, `components/ui/switch.tsx`, `components/ui/code-input.tsx`,
   `components/ui/password-input.tsx`, `components/ui/money-input.tsx` and `components/ui/birth-date-input.tsx` are the fields. `local/no-raw-form-control` fails a bare
   `<input>` or `<textarea>`; a disable comment states which property of the primitive is in the way.

2. **Every field declares its keyboard.** A field carries `type`, `inputMode`, `enterKeyHint`,
   `autoComplete`, `autoCapitalize`, `autoCorrect` and `spellCheck` from the row in the keyboard table
   that matches what it holds. A field with none of them is a defect, not a default.

3. **The primitive carries the attribute, not a wrapper.** The attributes ride on the `<input>` or
   `<textarea>` element itself, spread through `Input`, `Textarea` or `PasswordInput`, so a caller that
   forgets one is visible as a missing prop rather than as an inert attribute on a `div`.

4. **A handle, an email and a code never autocorrect.** `autoCapitalize="none"` (or `"characters"` for
   a code that is uppercase), `autoCorrect="off"` and `spellCheck={false}` together. The keyboard that
   capitalises the first letter of a username produces a name the server refuses.

5. **`type="number"` never ships.** It brings a spinner, changes value on scroll, drops leading zeros
   and refuses a partial entry. A numeric field is `type="text"` with `inputMode="numeric"`. Money is
   `MoneyInput`: `inputMode="decimal"`, the currency on the side the reader's prices wear it, and
   `parseMoneyToCents` reading the reader's own decimal mark, so "12,50" is never 1250 and an amount that
   cannot be read is refused rather than guessed.

5a. **A date of birth is typed, never picked.** A native date picker opens on today, and a birthday is decades
   of calendar or wheel from it, with no keypad offered instead. `BirthDateInput` is the one field for it, in
   onboarding, the identity check and a moderator's correction: three numeric parts in the order the reader
   writes a date (`datePartOrder`), each named in their language (`datePartName`) and by `bday-day`, `bday-month`
   and `bday-year` for autofill. A part hands on to the next when it is full, when no second digit can follow its
   first, or at the mark typed between parts; Backspace in an empty part steps back; a pasted date fills all
   three; an Arabic keypad's digits are read as digits. It answers with an ISO day, and only once the parts name
   a real one that is not in the future nor more than `MAX_BIRTH_AGE_YEARS` back, so a year on its way to the
   right one is never judged; its `onBlur` fires when focus leaves all three parts, never between them. A date
   near today (a schedule, a deadline, a filter) keeps `type="date"` and the platform's own picker.

6. **A field never zooms the page.** Every text field pairs its resting size with
   `pointer-coarse:text-base`, because iOS zooms the viewport for anything under 16px and the page
   never zooms back. `[tokens]` holds the value.

7. **Enter does the obvious thing.** `enterKeyHint` names it and the key performs it: `send` on a
   composer, `search` on a search field, `next` on a field with another after it, `done` or `go` on the
   last. A form whose Enter does nothing has the wrong hint. A composer asks `shouldSubmitOnEnter`, the one
   policy: a key an input method is still composing never sends, a single line sends, and a multi-line field
   sends on Return only while no on-screen keyboard is up (`data-keyboard` absent), so a tablet's hardware
   keyboard sends and a soft keyboard's Return stays a newline beside the send button.

8. **A limit is shown as what is left, late.** `CharacterLimitRing` reveals its number only inside the
   last tenth of the budget and turns to `text-error` past it. The ring itself follows the same rule —
   neutral until the numeral appears, `stroke-warning` with it, `stroke-error` past the limit — because
   a ring that goes amber at half a post is the same countdown the numeral is forbidden to be. Every frame
   a post is written or edited in carries it beside the way the post is sent: `ThreadToolbar` for the sheet
   and the phone deck, the inline composer, every reply variant and the edit sheet, never a hand-built
   `n/max`; `__tests__/components/compose/every-post-composer-shows-what-is-left.test.tsx` fails when
   one loses it.

### Focus and the keyboard

9. **Focus goes where the person is going to type.** An overlay containing one editing surface marks it
   `[data-autofocus]`, which `useFocusTrap` prefers over the first focusable, so a composer sheet does
   not open with the close button focused. An overlay with no editing surface takes no autofocus. Focus lands at
   once with `preventScroll`, so the first key is not lost; the scroll to the field waits `DIALOG_MOTION_MS`, until the
   sheet has stopped sliding in.

10. **The keyboard's height is read once.** `lib/viewport/visible-viewport.ts` is the one listener of
    `visualViewport` and the one receiver of a shell's keyboard frame. In one scheduled frame it publishes
    `--app-vh` (the visible height `100svh` gets wrong inside an iOS standalone window), `--app-vv-top` and
    `html[data-keyboard='open']`, under which the keyboard pays the bottom inset `[layout §16b]`. The keyboard
    is open when it overlaps the page by more than `KEYBOARD_HEIGHT_THRESHOLD` (100px), so a browser chrome
    shift never reads as one, or when the page itself shrank that far below its resting height while a field
    has focus, as Android does; `useKeyboardHeight` and `useKeyboardAware` read its store, and
    `AppViewportHeight` keeps it running. A shell's frame carries the height, duration and curve ahead of the
    viewport, so the store holds them (`keyboardFrame`) and publishes the keyboard from the frame, not from the
    late resize; `html[data-keyboard]` follows the lift, so the bottom inset drops after the bar has risen.

11. **Chrome rides the keyboard, it does not jump.** A sheet, a composer or a send button that sits above
    the keyboard rises with it on `translate` and never relayouts under it: `useKeyboardLift`, over the frame's
    duration and curve `[motion §30]`. The header above it does not resize when a field below it focuses. One
    owner per surface: inside a `Dialog` it is `DialogContent`, and a surface outside one calls the hook itself. Two compensations on the same
    toolbar lift it twice and leave an empty band the height of the keyboard. A surface that has to be sized
    by the keyboard (the editor's text composer centres its field in what the keyboard leaves) takes the final
    height once and glides its parts from where they were with `useFlipPlace` over `keyboardTiming`; it never
    animates a layout property.

12. **The caret is never under the keyboard.** A focused field scrolls itself into the visible viewport
    before the first character lands, and stays there as the field grows.

13. **The query debounces, the field never does.** A search field writes each keystroke straight to its
    own value and debounces only the query it sends, at `TIMING.debounce` (300ms). `[speed]` holds the
    budget a keystroke is held to.

### Paste and drop

14. **One paste and drop owner.** `useMediaDropPaste` provides `dropProps` for every composer.
    `extractMediaFiles` decides what counts as media, from a `ClipboardData` or a `DataTransfer` alike.

15. **Paste binds to the composer, never to the window.** A screenshot pasted into a different field is
    not attached here.

16. **A mixed paste keeps its text.** `preventDefault` runs only after the clipboard is known to carry
    media, so a paste that carries both an image and text still inserts the text.

17. **Drag state counts, it does not toggle.** Entering a child fires `dragleave` on its parent first, so
    the overlay is driven by a depth counter. `MediaDropOverlay` is the one drop affordance, and it
    appears only for a drag that carries `Files`.

18. **A pasted link becomes a card, not a chore.** `useLinkPreview` asks the server for the first URL in
    the text after `LINK_PREVIEW_DEBOUNCE_MS`, and `ComposerLinkCard` opens `LinkPreviewCard` under the words
    with its corner X; a removed card is published as the author's "no card". A direct image or video URL is
    rendered as media instead of scraped. Removing the URL removes its card in the same flush.

19. **How the text arrived is recorded at the field.** `causeFromInput` classifies each change as typed,
    pasted, autocorrected or `aiGhost` (`lib/domain/content/input-cause.ts`). A paste leaves `pasteCauseOf`'s
    answer for exactly the next change: a clipboard holding one address and nothing else is a `link`, not
    borrowed words, and an image-only paste that is prevented leaves nothing behind. `recordEdit` credits a
    model's whole-text rewrite only with the words it changed (`rewriteSpans`), counts a revision once per
    place the author went back to, never per key, and never counts the keyboard or a model as the author
    reworking their words; text the author cut and pasted back keeps the history it had.

### The composer

20. **The composer grows synchronously.** `useTextareaResize` sizes the field to its content with no
    transition; a transition on height reads as lag. Where `field-sizing` exists the browser grows the field and
    nothing is measured per key; elsewhere it is measured once per change and watched for width once. `ComposerPill` holds the field and its send on the
    last line.

21. **The composer shows focus with its caret.** No ring on a composer, a textarea or any large editing
    surface. `[components]` owns the one-ring rule.

22. **Markdown is painted, never shown raw.** `ComposerRichTextarea` paints the source, read by
    `parseRichSource`, in an `aria-hidden` overlay under a transparent-text textarea: every character
    stays where it was typed, the markers in faint ink, and emphasis is drawn with ink, stroke, fill and
    decoration only, never a weight, slant or padding that would widen a glyph and slide the caret off it.
    A link, a handle and a tag are accent ink on those same glyphs, through `RichText`'s `field` variant:
    no anchor, no break rule and no mask of their own, and markdown never splits a link. Both layers carry the same type classes, the visible placeholder is mirrored as the field's
    `aria-label`, and every composer, the reply bar included, draws this one field.

23. **Cmd or Ctrl and Enter submits.** `useComposeKeyboard` owns it. A composer inside a `Dialog` passes
    `ownEscape: false` so the dialog keeps Escape and no second document listener competes for it. An open
    @/# list is lent the keys first: `useAutocompleteKeys` holds the highlighted row, the field walks it with
    ↑ and ↓, Enter accepts that row instead of inserting a newline, and Escape closes the list rather than the
    composer behind it. The list is a `role="listbox"` of `role="option"` rows named to the field through
    `aria-activedescendant`, so what a reader hears is the row Enter would take.
    A composer sends on Enter only through `shouldSubmitOnEnter` (a multi-line field: on touch Return is a
    newline and the button sends) or `shouldSubmitLineOnEnter` (a single line), and neither fires on the Enter
    that commits an input-method candidate (`isImeEnter`: `isComposing` or `keyCode` 229).

24. **Attaching never deletes what is already attached, and what is attached belongs to the entry
    it was attached to.** Pictures and a voice take ride together; a clip is the one thing that
    cannot join either, because it carries sound and a frame of its own. `useComposerMedia` and
    `startVoice` are the only doors, and each refuses the pair it cannot hold by naming what to
    remove — `removePhotosFirst`, `removeVideoFirst`, `removeVideoForVoice`, `removeVoiceForVideo` —
    rather than discarding it. A take recorded on the third post of a chain publishes as the third
    post, and what belongs to the chain's first post alone — its poll, its people, its location —
    is drawn under that entry and nowhere else.

### Validation

25. **A field has one message slot and it is for errors.** `Field` renders one node under the control at
    `FIELD_MESSAGE_ERROR`, wired by `aria-describedby`, with `role="alert"` so it is announced when it
    appears. A caption beneath a field is banned by `[craft]`; the error is not a caption, it is the
    answer to what refused, and it is the only thing that may sit there. A refusal with a way through
    carries it in that same slot as an `INLINE_ACTION` inside the sentence — `UsernameField` opens the
    claim flow from the line that says the name is held — never as a second control below the field.

26. **`hint` is not used.** `Field` accepts one, and helper prose under a label is what `[copy-and-voice]`
    removes. If a field needs an explanation, the placeholder or the label is wrong.

27. **An error appears on submit or on blur, never mid-word.** Forms run react-hook-form's default
    `onSubmit` mode. A message that appears while the person is still typing the first value tells them
    they are wrong before they have finished being right.

28. **The message names what refused.** `[copy-and-voice]` owns the wording. The field, not the form,
    carries it: a form-level banner for a field-level fault makes the person hunt.

29. **A field that is invalid says so without moving.** `aria-invalid` tints the border through the
    primitive; the message slot is the only thing that grows, and it grows below every other field's
    baseline so the form does not reflow around it.

### Settings and defaults

30. **A setting exists only when two reasonable people differ.** Anything with one right answer is the
    behaviour, not a switch. A settings screen that has to explain a row does not need the row.

31. **A setting applies on change.** No Save button, no Apply, no dirty state. Device-local preferences
    go through `useLocalSettings`, account preferences through `useUpdateUserField`, which patches the
    auth store optimistically and rolls back with a toast when the write refuses.

32. **A setting previews live.** Theme, text size, calm colours, haptics and locale take effect on the
    screen the person is standing on, in the frame they choose. A change the person cannot see is a
    change they cannot judge.

33. **One save bar, for a policy set for someone else.** `FamilySaveBar` is the only Save in settings,
    because a guardian's limits are one policy and must commit together. Nothing else grows one.

34. **A setting has one control shape.** `SettingsToggleRow` for a boolean, `SettingsStepRow` for a short
    choice, ordered or not, because named stops on one track read quieter than a row of pills;
    `SettingsSegmentedRow` only where an option carries a glyph, `SettingsSelectRow` (`SettingSelect`) for a
    long one, `SettingsActionRow` for a destination (its `summary` states what is set there, through `RowDestination`), `SettingsFoldRow` for a group revealed in place under a
    one-line summary (`reveal` scrolls it into view where it opens at the foot of a short sheet). A row does not invent an eighth.
    A fold's or a destination's summary names only what departs from the defaults, through `departureSummary`:
    nothing at the defaults, the one change, or Custom for several, so it fits its row instead of truncating.
    Every row wears `SETTINGS_FLUSH_MARK`, so `SettingsSection` seats a run of them on the edge the hub's
    `SETTINGS_CARD` rows keep, and a card of rows carries no more air above its first row than the hub does.

35. **A locked option is offered, not hidden.** `SettingSelect` marks an entitled option `locked` and
    routes its tap to `onLockedSelect`, so the gate names itself instead of the choice disappearing.

36. **Defaults are read before the first paint.** A screen renders its controls invisible until
    `useLocalSettings` reports `hydrated`, rather than painting the default state and flipping.

37. **A control something else holds says so.** It shows the value in force, holds still, and names what
    holds it with the one press that frees it (`ReasonNote`).

38. **One concept, one control, one place.** A value a simple control sets is never written by a second control
    somewhere else. A precise control for it is the same state drawn finer: `FeelPad` and the Energy and Depth
    sliders read and write one recipe (`readVibe`, `withVibe`), so the pad and a slider can never disagree. A
    typed request writes only what a control on screen shows, folded onto that control (`foldVibes`), and a
    retired control's stored value is mapped onto the control that replaced it before the control disappears.
39. **A refinement is set in a setting's shapes.** The `FilterButton` sheet `[components §28]` holds
    `SettingsStepRow` for named stops, ordered or not; `FilterChecks`, the pick list's own rows, for a set, or
    for one of a set with `single`; and `SettingsToggleRow` for a yes-or-no. Each applies on change, as a
    setting does, and the sheet's bar carries the one Reset. A refinement lives where the page keeps its
    state, the URL for a route, and a Reset that clears several params writes them once through
    `useClearParams`, because a write per param starts from the same query and undoes the one before.

## The keyboard table

Every field kind, and what it declares. `enterKeyHint` is `next` instead of the value below whenever
another field follows in the same form.

| Field holds | `type` | `inputMode` | `enterKeyHint` | `autoComplete` | `autoCapitalize` | `autoCorrect` | `spellCheck` |
|---|---|---|---|---|---|---|---|
| Email address | `email` | `email` | `go` | `email` | `none` | `off` | `false` |
| Existing password | `password` | n/a | `go` | `current-password` | `none` | `off` | `false` |
| New password | `password` | n/a | `done` | `new-password` | `none` | `off` | `false` |
| Sign-in field offering a passkey | `email` | `email` | `go` | `username webauthn` | `none` | `off` | `false` |
| One-time code | `text` | `numeric` | `done` | `one-time-code` | `none` | `off` | `false` |
| Recovery or invite code | `text` | `text` | `done` | `off` | `characters` | `off` | `false` |
| Case-sensitive code | `text` | `text` | `done` | `off` | `none` | `off` | `false` |
| Backup passphrase or recovery key | `password` | n/a | `done` | `off` | `none` | `off` | `false` |
| Handle | `text` | `text` | `done` | `username` | `none` | `off` | `false` |
| Display name | `text` | `text` | `done` | `name` | `words` | `off` | `false` |
| Search query | `search` | `search` | `search` | `off` | `none` | `off` | `false` |
| Web address | `url` | `url` | `done` | `url` | `none` | `off` | `false` |
| Whole number | `text` | `numeric` | `done` | `off` | `none` | `off` | `false` |
| Money | `text` | `decimal` | `done` | `off` | `none` | `off` | `false` |
| Numeric passcode | `password` | `numeric` | `done` | `off` | `none` | `off` | `false` |
| One part of a date of birth | `text` | `numeric` | `done` | `bday-day`, `bday-month` or `bday-year` | `none` | `off` | `false` |
| Message or post body | n/a | n/a | `send` | `off` | `sentences` | `on` | `true` |
| Caption, bio, alt text | n/a | n/a | `done` | `off` | `sentences` | `on` | `true` |

Two rows exist only because uppercasing would corrupt what they hold. A case-sensitive code is the
device-link offer in `components/settings/device-link-section.tsx`, whose payload is base64, so the
`characters` of the row above it would make every typed code invalid. The backup secret box takes either
a passphrase or a recovery key and declares `off` rather than `current-password`, because a manager
filling the account password there would spend one of the restore's limited attempts.

The passkey row is the sign-in email field: `usePasskeyAutofill` starts a conditional-mediation ceremony
wherever the browser offers one, so a saved passkey appears in that field's autofill, and `PasskeyButton`
stays beside it for every runtime that does not.

## Enforced by

- `__tests__/fitness/a-field-shows-focus-once.test.ts` covers 21.
- `__tests__/fitness/the-keyboard-is-read-once.test.ts` covers 10: `visualViewport`, `--app-vh` and
  `data-keyboard` belong to the one listener; `__tests__/viewport/visible-viewport.test.ts` drives it.
- `__tests__/components/compose/markdown-is-painted-in-the-composer.test.tsx` and `__tests__/domain/rich-text.test.ts`
  cover 22: every character kept, the markers faint, and no element or class that changes a glyph's width.
  `__tests__/ui/a-caret-overlay-measures-like-its-textarea.test.tsx` holds a link to the same, and
  `tools/uiprobe/composer-caret.probe.ts` measures every glyph of both layers in a real browser.
- `eslint-rules/no-raw-form-control.mjs` (`local/no-raw-form-control`) covers 1.
- `__tests__/fitness/controls-are-operable.test.ts` covers hit area and naming for 1 and 34.
- `__tests__/fitness/every-control-has-a-test.test.ts` ratchets that a field's controls are driven by a
  spec before a person drives them.
- `__tests__/fitness/a-field-declares-its-keyboard.test.ts` covers 2, 4, 5, 7, and reads any field that declares
  `inputMode="search"` or `enterKeyHint="search"` (`SearchInput`, the emoji and Explore bars) as the search row. It compiles the keyboard
  table: it resolves each field to the row its `type`, `autoComplete`, tag or `inputMode` names, ratchets
  every cell of that row it does not declare, ratchets a field that declares none of the seven, and
  ratchets `type="number"` on its own.
- `__tests__/domain/money-amount.test.ts` covers 5 for money, in every separator the nine locales write.
- `__tests__/components/ui/birth-date-input.test.tsx`, `__tests__/domain/birth-date.test.ts` and
  `__tests__/utils/format-date.test.ts` cover 5a: the keypad and the autofill names on every part, the reader's
  order, the hand-on, the paste, the days it refuses, and the one blur for the whole field.
- `__tests__/fitness/a-text-field-cannot-zoom-ios.test.ts` covers 6, reading the ramp and the 16px floor
  out of `app/globals.css` and resolving a size hidden behind a `lib/styles` recipe.
- `__tests__/fitness/a-paste-binds-to-its-composer.test.ts` covers 15.
- `__tests__/fitness/a-field-message-is-only-an-error.test.ts` covers 25, 26, for the `hint` no `Field`
  may be given and the `FIELD_MESSAGE_ERROR` node that has to announce itself.
- `__tests__/fitness/a-validated-field-surfaces-its-refusal.test.ts` covers 28, resolving every
  `register('…')` against the `errors` the form's own file, or a file importing it, renders.
- `__tests__/fitness/a-setting-applies-on-change.test.ts` covers 31, 33, reading the commit labels out of
  `lib/i18n/locales/en.json` so a renamed key cannot walk a Save past it.
- `__tests__/fitness/a-setting-waits-for-its-value.test.ts` covers 36.
- `__tests__/components/settings/reason-note.test.tsx` and `__tests__/map/map-components.test.tsx` cover 37:
  the reason a held control names, and the one press that frees it.
- `__tests__/feed-algorithm/feed-feel.test.ts` and `__tests__/components/settings/feed-settings.test.tsx` cover 38:
  every ranked dimension owned by exactly one vibe, every position read back as the one that wrote it, and no
  second writer of a field on the panel.
- `__tests__/components/shared/filter-button.test.tsx`, `__tests__/components/explore/search-refine.test.tsx`
  and `__tests__/components/notification/notification-tabs.test.tsx` cover 39: each shape writes its own axis
  back, and Reset and Clear let every refinement go at once.

Rules 11, 12, 20 and 32 are layout, easing and paint: no source read decides them, and
`[definition-of-done]` holds the browser evidence they need.

## Traps

- 1: `Input` renders its control inside `w-full min-w-0`. As a bare flex child the `<input>`'s intrinsic
  `size` width won, and option rows came out two thirds wide.
- 3: `Field` owns the control's `id`, so a caller-supplied `id` on a field inside one unlinks the label
  from the control. A field drawn as several controls wraps each one after the first in `FieldFollower`, or
  all of them take the one id.
- 5a: a blur between two parts of one field arrives with no `relatedTarget` on some engines, so the field asks
  where focus landed a microtask later; judged on the event alone, tabbing from month to day closed the age gate.
- 9: `useFocusTrap` restores focus to the previously active element only when it is still connected; a
  trigger removed by the action the overlay performed takes focus nowhere.
- 10: iOS scrolls the visual viewport under a focused input without resizing it, so the one listener
  hears `scroll` as well as `resize`. Android resizes the page for the keyboard and overlaps nothing, so
  an overlap alone never sees its keyboard.
- 16: a clipboard that carries both an image and text is common from design tools and screenshot
  utilities; swallowing the event before inspecting it drops the text half with no sign.
- 17: a naive boolean flickers on every child drag-enter event, so the overlay flashes across a drag.
- 19: an image-only paste is prevented upstream and fires no `onChange`, so a flag set instead of
  assigned makes the next typed key count as pasted.
- 22: the overlay must occupy exactly the space the textarea does, or the caret drifts away from the text
  it belongs to. A trailing zero-width space keeps the last empty line at full height.
- 26: react-hook-form's `mode` is `onSubmit` by default and a form that sets `onChange` to "help" turns
  every keystroke of a half-typed email into a red line.
- 27: a field the schema can refuse and the `Field` renders no `error` for does not merely go unexplained
  — `handleSubmit` stops at the resolver, so the whole Save does nothing and says nothing.
