# Platform grammar

Surf runs in a browser and must not feel like a document. This document owns the conventions the platform itself
supplies and the product either adopts, narrows or refuses: what the pointer does, what the scrollbar looks like,
what a person can select and long-press, how a scroll ends, what happens when the device turns or the window is
dragged, and how something leaves the app through the share sheet. The feeling it protects is that the surface
under the finger or the pointer behaves the way the device behaves, everywhere, with no screen that forgot.

## Owns and defers

Owns: the cursor vocabulary, scrollbar appearance and the reserved gutter, text selection and the long-press
callout, tap highlight and `touch-action`, overscroll chaining, orientation and every width between the postures,
desktop window resizing, sharing out of the app, and printing.

Defers: postures, breakpoint values, safe areas and the reading column to [layout]; press feedback, haptics and
the press ladder to [feedback]; gestures, springs and what the finger drags to [motion]; hit areas and pointer
capability to [accessibility]; the pull-to-refresh choreography to [motion]; `dir` and mirrored gestures to
[internationalisation]; tokens and their values to [tokens].

## Rules

### The pointer

1. **The arrow is the default and stays the default.** `app/globals.css` sets `cursor: default` on `button`,
   `[role="button"]`, `[type="button"]`, `[type="submit"]`, `[type="reset"]` and `a`. A control looks like a
   control; changing the pointer to say so again is the browser's habit, not the product's.
2. **The cursor only changes when its shape carries information the control does not.** The whole vocabulary is
   the table below. `cursor-pointer` is not in it and never ships.
3. **The primitive sets the cursor; the call site never does.** A disabled control's `cursor-not-allowed` comes
   from `button.tsx`, `icon-button.tsx`, `input.tsx`, `textarea.tsx`, `select.tsx`, `checkbox.tsx`, `switch.tsx`,
   `tabs.tsx`, `slider.tsx`, `choice-group.tsx`, `dialog.tsx`, `dropdown-menu.tsx` or the `disabled` recipe in
   `lib/styles/glass.ts`. A feature component that writes a cursor class is building a control that belongs in
   the primitive.
4. **Grab is on the thing that moves, and it becomes grabbing while it moves.** A draggable surface reads
   `cursor-grab` at rest and `cursor-grabbing` from pointerdown to release. `components/ui/sheet-grabber.tsx` is
   the shape every other drag handle follows.
5. **A cursor is a class, never an inline style,** for the same reason a blur is: an inline value cannot be shed
   or overridden by the tier, the theme or a state.

### Scrollbars

6. **One scrollbar, thin and theme-aware.** `styles/scrollbar.css`, imported once as the first line of
   `app/globals.css`, sets `scrollbar-width: thin` and a 6px WebKit bar on a transparent track, with the thumb at
   `--border-default`, its hover at `--text-muted`, and `--radius-full`. It never names a colour, so every theme
   inherits it. No surface styles its own.
7. **`.scrollbar-none` is the only way to hide one,** and it is for a horizontal strip whose overflow is already
   obvious: a chip row, a tab strip, a carousel. A vertical scroller never hides its bar, a text field aside (7b).
   A scrollbar class that `styles/scrollbar.css` does not define is dead and reads as an unstyled default.
7a. **A horizontal strip never shows a bar.** It fits first: a row of controls folds its rare actions into ···
    before it scrolls, the way the composer's tool row did. What must still scroll is `ScrollStrip`, which is
    `STRIP_SCROLLER` (its own axis, `overscroll-x-contain`, `scrollbar-none`, proximity snap) plus the
    `useScrollEdges('x')` fade on whichever edge has more. Staff tooling keeps the thin bar under a wide table.
7b. **A text field never shows a bar.** Every `textarea` hides its scrollbar in `styles/scrollbar.css`: it grows
    with its words to its ceiling and past that still scrolls by wheel, touch and caret.
8. **The root gutter is always reserved.** `scrollbar-gutter: stable` on `html`, so locking scroll for an overlay
   does not widen the viewport and lurch every fixed element sideways on open and close. Scroll locking must not
   also pad the body; the gutter already holds the width.

### Selection and long press

9. **Content is selectable; chrome is not.** Anything a person might quote, copy or read aloud selects with the
   native handles, highlighted with `::selection` at the accent token. `select-none` belongs on controls, labels,
   axis ticks, timestamps, drag handles and canvas surfaces, and nowhere else.
10. **A gesture surface that must be `select-none` gives its text back.** Where a whole surface suppresses
    selection so a drag or a long press can work, the readable content inside it carries `select-text`. A full
    screen that cannot be selected is a defect, not a decision.
11. **The long-press callout is suppressed only where the product owns the long press.** A media tile, a chat
    bubble, a sticker and the camera each open the app's own menu, so the native callout would double it; a link,
    a name and body text keep the platform's menu so a person can still save or copy. `touch-callout-none` is
    the utility, and `BUTTON_CHROME` (`select-none touch-callout-none`) rides `Button`, `IconButton` and
    `UnstyledButton`, so a long press on chrome raises no loupe and no Copy bar. A surface that owns a long press
    over an image adds `user-drag-none` too, so WebKit's drag does not start under the app's own menu. `(not yet
    built)`: the media tile, the sticker and the camera do not carry them yet, only the chat bubble and the
    full-screen video.
12. **The native tap highlight is off everywhere** (`-webkit-tap-highlight-color: transparent` on `html`), which
    makes the press ladder the only press feedback and makes it load-bearing on iOS [feedback].
13. **`touch-action` narrows, never widens.** `body` carries `manipulation` to drop the 300ms delay; a gesture
    surface declares exactly the axis it owns (`touch-none`, `pan-y`, `pan-x pan-y`) and never more, so the
    platform keeps every gesture the surface does not handle, swipe-back included.

### Scroll boundaries

14. **The document never bounces.** `overscroll-behavior-y: none` on `html, body`: pull-to-refresh owns the top
    pull, and the native rubber-band was the only moment the frosted header's sampling seam showed.
15. **Every scroller inside a surface contains its own overscroll.** A dialog body, a sheet, a menu, a panel, a
    chat list and an autocomplete list are `overscroll-contain`, so reaching the end scrolls nothing behind them.
    `overscroll-none` is only for a full-screen stage that owns every gesture on it.
16. **A horizontal scroller contains its axis** (`overscroll-x-contain`) so a carousel swipe never triggers the
    browser's back gesture.

### Orientation and width

17. **Rotating the device never costs anything.** Scroll position holds, an open overlay stays open, a playing
    clip keeps playing, and nothing relayouts into a scrollbar. The manifest asks for `any` on every host: a
    portrait lock is honoured by an installed app, and it kept the phone upright under a video it was turned
    sideways for. A landscape clip playing in its frame goes full screen on the turn and leaves on the turn back.
    A turn is the screen rotating on the same display (`isTurn`), never the viewport's aspect changing: folding,
    unfolding or resizing a split window never sends a clip full screen or closes a player. Turning a tablet keeps
    its size class, and turn-to-fullscreen happens only in compact.
18. **Landscape is the same layout at a shorter height, not a second design.** Full-height surfaces measure
    `--app-vh` rather than `100vh`, sheets cap at the visible height, and a control row that fits portrait scrolls
    rather than wraps [layout]. The orientation branches in the product are the two-person call, whose tiles sit
    side by side in landscape and stack in portrait, and the full-screen video player, whose strip tightens and
    clears the cutout [media §34c].
19. **Every width between 320 and 2560 is clean.** Nothing scrolls the body horizontally, no control row wraps to
    a second line on a phone, and no rung of `BREAKPOINTS` produces a layout that exists only at that rung. Windows
    from 320×320 are clean too: a split screen, a pop-up window and a flip's cover screen keep the header, the
    content and the tab bar on screen with the focused field above the keyboard. A cover screen is supported, not
    specially designed.
20. **A width question is asked once, by name.** `useMediaQuery(MEDIA_QUERIES.…)` and its named helpers
    (`useIsCompact`, `useIsTablet`, `useIsDesktop`, `useIsWider`); never a literal query string, and never a stored
    `window.innerWidth`. `compact` is "too narrow for the desktop layout", a rung inside one design. Which design a
    window gets is the size class, the one layout question that is not a media query, because no media query can
    ignore the keyboard `[layout §1]`; an interaction asks the pointer, never a width.
21. **Dragging a desktop window does not jank.** Layout changes because a media query changed, not because a
    resize handler ran. Anchored surfaces reposition on `resize` through a scheduled frame, measurement uses a
    shared `ResizeObserver` where one will do, and nothing reads layout inside the event.

### Leaving the app

22. **Share is one decision, made in one place.** `useShareOrCopy` for a URL, `shareOrDownloadBlob` for a file;
    both go through the `share` and `files` seams, whose web arms hold the choice between the native sheet and the
    fallback [shells]. A surface never re-implements it.
23. **Every share has a fallback and it is named.** A URL falls back to the clipboard with the toast the copy
    owns; a file falls back to a download. A share control with an empty catch is a dead control.
24. **A dismissed share sheet does nothing.** `AbortError` is the person saying no; copying the link behind their
    back is not the answer, and neither is a toast.
25. **A share control is only rendered where a share can happen.** `hasShareSheet()` and
    `shareSheetTakesFiles()` are the only honest answers, and `mediaShareMode` is where that answer is turned into
    which control appears.
26. **Sharing is a physical moment.** One `haptic('light')` as the sheet is asked for, from the shared hook, never
    from the call site [feedback].

### Printing

27. **Nothing prints.** No `@media print`, no `print:` variant, no `window.print()`. Surf is a place, not a
    document; a person who needs bytes out of it exports them through share or download, which are designed.

### Arriving from another app

28. **A share into Surf lands on `ROUTES.share` and sends nothing until a tap.** The web manifest's share target,
    Android's `ACTION_SEND` and the page all read the one `SHARE_IN` rule; the words are shown as plain text with
    two choices, the composer or chats, and the page never navigates anywhere a share names. Pictures and clips
    are held by the web service worker or the shell's staging until `receivedFiles` takes them once; they go to
    the composer only, never to a chat.

## The pointer's vocabulary

| Cursor | Means | Where |
|---|---|---|
| `default` | This is an app | everything, from the base rule; buttons and links included |
| `cursor-grab` / `cursor-grabbing` | This moves, and is moving | sheet grabber, PiP window, waveform scrubber, timeline playhead, canvas stickers, crop frame |
| `cursor-move` | This moves in two axes | image cropper, mask region |
| resize cursors | This edge resizes | crop frame handles, mask handles, timeline trim handles |
| `cursor-text` | This is an editable text region that does not look like a field | the story text tool while editing |
| `cursor-crosshair` | This point is the input | media tagging, canvas placement |
| `cursor-zoom-in` | Another magnification exists | image viewer |
| `cursor-wait` | The surface is busy and input is queued | a running cutout |
| `cursor-not-allowed` | Refused, with the reason one tap away | set by the primitive only |

## Where overscroll is contained

| Surface | Behaviour |
|---|---|
| document | `overscroll-behavior-y: none`; pull-to-refresh owns the top pull |
| dialog body, sheet body, menu, dropdown, popover, autocomplete | `overscroll-contain` |
| chat list, thread, feed panel, right panel, drawer, admin shell | `overscroll-contain` |
| carousel, waveform | `overscroll-x-contain` |
| full-screen editor stage | `overscroll-none` |

## Enforced by

- `__tests__/fitness/every-css-var-resolves.test.ts`: 6, because `styles/scrollbar.css` is in its file set.
- `__tests__/hooks/use-orientation-turn.test.ts`: 17, a rotation is a turn and an unfold or a resize is not.
- `__tests__/fitness/a-tap-target-is-a-control-not-a-wrapper.test.ts`: 3, for the wrapper half of it.
- `__tests__/fitness/arbitrary-value-duplicates-a-token.test.ts`: 5, for the inline-style half of it.
- `__tests__/fitness/the-pointer-has-one-vocabulary.test.ts`: 1, 2 and 3, ratcheting every cursor class outside
  the table and every `cursor-not-allowed` written anywhere but the primitive that refuses.
- `__tests__/fitness/a-grab-becomes-a-grabbing.test.ts`: 4, ratcheting a `cursor-grab` with no `cursor-grabbing`.
- `__tests__/fitness/a-scrollbar-class-exists.test.ts`: 7, ratcheting a `scrollbar-*` class `styles/scrollbar.css`
  does not define.
- `__tests__/fitness/a-horizontal-strip-hides-its-bar.test.ts`: 7a, for every horizontal scroller
  (`overflow-x-auto`, or its scroll twin) outside staff tooling.
- `__tests__/fitness/scroll-lock-does-not-pad-the-body.test.ts`: 8, both halves — `scrollbar-gutter: stable` on
  the root, and no script writing padding or margin onto `body`.
- `__tests__/fitness/a-button-is-chrome-not-content.test.ts`: 9 and 11, for the three button primitives.
- `__tests__/fitness/content-stays-selectable.test.ts`: 9 and 10, ratcheting a full-screen `select-none` whose
  file gives no text back with `select-text`.
- `__tests__/fitness/touch-action-narrows-only.test.ts`: 13, for the vocabulary half of it — every declaration,
  inline value and Tailwind utility is one of the five narrowing values.
- `__tests__/fitness/an-overlay-scroller-contains-its-scroll.test.ts`: 15, for the overlay primitives outright
  and for every height-capped scroller on a ratchet.
- `__tests__/fitness/a-media-query-is-named.test.ts`: 20, ratcheting `useMediaQuery` and `matchMedia` called with
  a literal query string.
- `__tests__/fitness/a-share-has-one-owner.test.ts`: 22, 23, 24 and 25, ratcheting `navigator.share` outside the
  two web arms and any caller that never tells `AbortError` from a refusal.
- `__tests__/fitness/nothing-prints.test.ts`: 27, ratcheting `@media print`, the `print:` variant and
  `window.print()`.

**Not yet enforced**

- `a-long-press-belongs-to-one-menu` (11): every surface with a long-press handler also suppresses the native
  callout, and no other surface does. Waits on the utility rule 11 asks for; there is nothing yet to read.
- `every-route-survives-a-rotation` (17, 18, 19): the browser sweep opens each route at 390×844 and 844×390 and
  asserts no horizontal document scroll and no control row on a second line. Belongs in `e2e/`; jsdom computes
  no layout, so a fitness spec could only pretend to measure it.

## Traps

- 7a: A strip written as `overflow-x-auto` alone looks right on a phone, where overlay scrollbars
  vanish at rest, and ships a grey rail under the row on every desk with a mouse.
- 8: Locking scroll without a stable gutter widens the viewport by the scrollbar's width, and every fixed
  element lurches sideways on both open and close. Reserving the gutter makes locking a no-op for layout.
- 14: The native rubber-band bounce was the only moment a 1px bright seam appeared under the frosted header:
  Chrome samples the warm tones at the header's edge mid-bounce.
- 19: `scrollWidth` cannot answer whether a layout overflows here. A `tap-target` pseudo-element and any
  descendant scroller inflate every ancestor. Measure the visible box or judge by screenshot.
- 19: A grid item without `min-w-0` measures its content: a 1928px item sat in a 382px track while the document
  reported no overflow.
- 21: Each `ResizeObserver` is its own entry in the browser's rendering step; one shared observer costs one.
- 24: A share that falls back on every rejection copies the link when the person deliberately dismissed the
  sheet. Only a non-`AbortError` failure falls through.
- 25: Only `canShare` can answer whether a file may be shared, and it needs the file; before the fetch, the mere
  presence of the function is the signal.
