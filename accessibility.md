# Accessibility

This document owns what makes the product operable by someone who is not looking at it, not using a mouse, not
seeing the colours the way we do, or not moving a finger with precision. It is not a compliance pass laid over
a finished screen: a name, a role, a landmark, a focus target, a hit area and a contrast ratio are part of what
a control *is*, decided when it is built. The feeling it protects is that the product works the same way for
everyone, at the same speed, with nothing announced twice and nothing announced not at all.

## Owns and defers

Owns: accessible names and roles, landmarks and heading order, focus order and focus custody, hit areas,
contrast minimums per theme, how an operating-system preference reaches the interface, what a screen reader is
told when something changes without a tap, and the gate's precedence over everything the app renders.

Defers: a field's focus and inline validation `[input-and-forms]`; showing focus once and the
overlay primitives that own the trap `[components]`; ring, hit-area and z-index token values `[tokens]`; how
reduced motion is choreographed `[motion]`; what reduced data costs a media surface `[speed]`; the type ramp
and the layouts that survive the largest text step `[type]`; alt text and captions `[media]`; the words inside a
name `[copy-and-voice]`; the translation of a name `[internationalisation]`.

## Rules

### Names and roles

1. **Every control says what it does.** A control's accessible name is the action, in the reader's language,
   resolved through the catalogue `[internationalisation]`. `IconButton` type-requires `aria-label` or `aria-labelledby`
   because it has no text to fall back on. Two controls a person can reach at the same time never answer to
   the same words: the story editor's overlay tool is `story.toolOverlay` precisely because its background
   panel already owns `ui.image` (`components/story/canvas-editor-constants.ts`).
2. **An icon-only control still announces itself.** When a label is hidden by width it becomes `sr-only`, as
   the rail's labels in `components/layout/sidebar.tsx` do below the widest desk; it is never dropped.
3. **A name is not a tooltip.** A tooltip is a fine-pointer affordance and cannot be the only source of a
   name; the accessible name exists whether or not a pointer can hover.
4. **A name changes when the state changes.** A like button reads "Like" or "Unlike", a mute reads "Mute" or
   "Unmute"; the label carries the state so nothing else has to announce it.
5. **A role is one the ARIA specification allows on that element**, and its required attributes come with it.
   No `aria-selected` on a button outside a tablist, no `aria-checked` outside a checkbox or radio role.
6. **ARIA attributes are declared under their standard hyphenated name.** A component prop spelled `ariaLabel`
   is unreachable for a caller writing `aria-label`, and TypeScript stays silent about it.
7. **A decorative glyph is `aria-hidden`.** An icon beside its own label, a chevron on a row and a divider
   glyph add nothing to the name and are hidden rather than read.
8. **A full-screen takeover behaves like a dialog.** It carries `role="dialog"`, a name, a focus trap, a
   scroll lock and Escape, or it renders one of the overlay primitives that already own all five `[components]`.

### Landmarks and headings

9. **One `main` per page.** `MainLayout` renders `<main id="main-content">` in both its branches; a route never
   renders a second one.
10. **One `h1` per page, and `PageHeader` is where it lives.** A route that uses the header primitive gets its
    `h1` from `PAGE_HEADER_TITLE` and writes no other.
11. **Headings descend without skipping.** A section under an `h1` is an `h2`; a level is never picked for its
    size, because size comes from the type ramp `[type]`.
12. **Navigation is a `nav` and names itself** when more than one exists on the page, the way the dashboard's
    section list does.
13. **The skip link is first in the document and reachable over everything.** `SkipToContent` in
    `components/layout/main-layout.tsx` targets `#main-content`, is `sr-only` until focused, and sits at
    `Z_INDEX.max`; it is the one surface allowed above the gate, because a keyboard user who cannot reach it
    has no way into the page at all.

### Focus

14. **Tab order follows reading order.** The DOM order is the visual order; nothing is repositioned by
    `order`, absolute placement or a positive `tabIndex` in a way that reorders the keyboard path.
15. **Focus is always visible for a keyboard user.** The global `:focus-visible` outline in `app/globals.css`
    is the floor. A control that suppresses it takes `FOCUS_RING` from `lib/styles/focus.ts`: a 2px accent
    ring offset 2px over the canvas, so it reads around an accent fill as well as beside one. A row inside a
    clipping list or menu takes `FOCUS_RING_INSET`, a control over a photo `FOCUS_RING_ON_MEDIA`, and a
    field keeps its shell ring `[components §5]`. `Button`, `IconButton`, `UnstyledButton`, `Pressable`,
    `Switch`, `Checkbox` and `Tabs` draw it, so a control built from them never spells a ring. The ring
    follows the control's border-radius, so a round target is a round box: the base rule in
    `app/globals.css` rounds any `a` or `button` whose only child is `rounded-full`, an avatar host keeps
    its box the avatar's square (`AUTHOR_AVATAR_LINK`), and a round target with more than one child, the
    profile avatar inside its story ring, carries `rounded-full` itself.
16. **A control that takes focus is a real control.** Focus never lands on a `div` standing in for a button,
    and a `role="button"` div is never the answer where `AuthorLink` or a primitive exists `[components]`.
17. **An overlay takes focus in and gives it back.** `useFocusTrap` moves focus to `[data-autofocus]`, then the
    container if it declares itself focusable, then the first match of `FOCUSABLE_SELECTOR`, and restores it to
    the trigger on close.
18. **A surface opened by thumb does not claim focus.** `useFocusTrap(active, ref, { autoFocus: false })`
    contains Tab without lighting a ring nobody asked for; the first Tab still enters.
19. **Only the topmost surface owns the keyboard.** A trap below another passes `enabled: false`, and a key
    listener exists only while its surface is on screen `[navigation-and-history]`.
20. **Focus is never left on a removed node.** When a row, sheet or dialog unmounts, focus returns to the
    element that opened it; it never falls back to `<body>`.

### Hit areas

21. **44px on a coarse pointer.** `--tap-target-min` is 44px and the `tap-target` utility paints it as an
    `::after` inside `@media (pointer: coarse)`. A range input gets it from `min-height` instead, because a
    replaced element renders no pseudo-element.
22. **`tap-target` goes on a real control, never a wrapper.** On a container its 44px pseudo-element covers the
    container's own children, and the taps that look like they land on a name or a link land on the div.
23. **`tap-target` is paired with a positioning class.** Without a positioned ancestor the hit area detaches
    and swallows a neighbour's taps; `local/no-unpositioned-tap-target` fails the lint.
24. **24px minimum on a fine pointer, and adjacent targets do not touch.** Two independent actions keep at
    least 4px of dead space between their hit areas, so a slip does the wrong thing rather than the
    destructive thing. `(not yet built)`: needs a `--tap-target-min-fine` token and a spec that measures
    rendered hit rectangles in a browser, since jsdom has no layout.

### Contrast

25. **4.5:1 for text, 3:1 for large text and control edges, in every theme.** `AA_BODY` and `AA_LARGE` in
    `scripts/theme/contrast-pairs.mjs` are the two numbers, and `CONTRAST_PAIRS` is the list of pairs each
    theme is scored on.
26. **A token that can carry text has a contrast pair.** The theme contract fails when a text or glyph token,
    or a category hue that can carry a label, has no pair scoring it; adding a token means adding its pair.
27. **`--text-faint` is chrome, never data.** It is held to the non-text minimum only, so it may not carry a
    number, a name, a timestamp or anything a person has to read.
28. **Contrast is measured over the real compositing chain.** A pair declares what it sits on, innermost last,
    because a tint over a card over the canvas is not the same ground as the canvas.

### Preferences

29. **A preference is honoured by token, not by branch.** Reduced motion, reduced data and text size change a
    variable that the whole ramp reads; a component never asks whether the preference is on and renders a
    different tree.
30. **Reduced motion shortens motion; it never teleports.** Functional motion keeps a real fade so
    `transitionend` still fires; status motion keeps looping, because a frozen bone reads as a hang `[motion]`.
31. **Reduced data lowers cost, not capability.** `prefersReducedData()` and `shouldConserveData()` in
    `lib/utils/media-query.ts` are the one read; autoplay stops and image DPR falls, and the person can still
    play the video `[speed]`.
32. **The viewer's text scale multiplies the whole ramp.** `--font-scale` (0.875 / 1 / 1.125) scales the px
    type tokens and `html[data-font-size]` moves the rem-based chrome with it; a layout survives the largest
    step without clipping, overlapping or a horizontal scroll `[type]`.
33. **A preference read in JavaScript comes from `MEDIA_QUERIES`.** `reducedMotion`, `reducedData`, `touch`,
    `fine` and `hoverCapable` are named there; nothing writes a media-query string by hand `[tokens]`.

### What is announced

34. **A change the person caused, on the control they touched, is not announced.** The control's own name
    already changed (rule 4); a live region on top of it says everything twice.
35. **A change the person caused, somewhere else on the page, is announced once.** The toast is that
    announcement: `components/ui/toast.tsx` is `role="alert"` for error and warning and `role="status"`
    otherwise, so severity picks the interruption level `[feedback]`.
36. **A change the person did not cause is announced politely or not at all.** Arriving messages, arriving
    posts and a changed call state are `aria-live="polite"`; nothing that arrives on its own is `assertive`.
37. **A message log is a `log`, and history is not new.** `chat-view.tsx` marks the scroller `role="log"`
    `aria-live="polite"` and its list `aria-live="off"`, then announces one keyed `sr-only` line, because a
    reverse-infinite list prepends old messages a naive live region would read out as if they had just arrived.
38. **A live region exists before the text does.** The node is in the DOM at mount and its content changes;
    a region mounted together with its message is not announced.
39. **Live regions do not nest.** A skeleton inside a skeleton that both declare `role="status"` announces
    twice; the outer one owns it.
40. **Determinate progress is readable, not hidden.** An upload, a publish or an export the person waits on
    carries `role="progressbar"` with `aria-valuenow`, or a polite text line beside it. `Meter` is the
    reference and carries both: `progressbar` while the fraction is knowable, `role="status"` once it is
    not. A surface that renders it owes neither role of its own. `components/compose/compose-progress-bar.tsx` is `aria-hidden` with no substitute, and is
    recorded as debt in `__tests__/fitness/baselines/silent-progress.json`.
41. **A validation error is announced where it is.** The field's message is `role="alert"` and bound to the
    field by `aria-describedby`, so focus moving into the field reads the reason `[input-and-forms]`.

### The gate

42. **Nothing the app renders outranks the gate.** The parental screen-time lock and the wellbeing limit paint
    at `Z_INDEX.gate`; the ladder runs modal < liveStage < floating < viewer < toast < gate < privacyOverlay,
    and a full-screen takeover uses `viewer`.
43. **The gate is a real boundary.** `FullScreenGate` is `role="dialog"` `aria-modal="true"`, names itself
    through `aria-labelledby`, locks scroll and traps focus, so the page behind it is not reachable by Tab.
44. **A gate names its reason and its one way out**, and where there is only waiting it says so; the screen-time
    gate keeps the "request more time" action available even while a request is pending `[permissions-and-trust]`.

## What is announced, and what is not

| Moment | Announced | How |
|---|---|---|
| Like, bookmark, repost, follow | No | The control's own name flips (rule 4) |
| Send a message or comment | No | The bubble is in the log; the log announces it (rule 37) |
| A message arrives | Yes, polite | The keyed `sr-only` line in `chat-view.tsx` |
| Older history pages in | No | The list is `aria-live="off"` |
| A mutation is refused | Yes, assertive | Toast at `role="alert"` naming what refused |
| A non-error result elsewhere on the page | Yes, polite | Toast at `role="status"` |
| A skeleton appears | Yes, once | `role="status"` `aria-busy="true"` on the outermost ghost only |
| Upload or publish progress | Yes | `Meter`: `progressbar` with `aria-valuenow`, then `status` |
| Call state change | Yes, polite | The call surface's own status line |
| Route change | No | The destination's `h1` and the restored focus carry it |

## Enforced by

- `__tests__/fitness/controls-are-operable.test.ts` covers rules 1, 8. No unnamed `Switch` or range input; the three
  takeovers (`editor-sheet`, `story-viewer`, `image-viewer`) carry a dialog role, a name and Escape, or render a
  primitive that does. It also holds glyph sizes to `ICON_SIZES` `[tokens]`.
- `__tests__/fitness/aria-props-use-the-standard-name.test.ts` covers rule 6, by AST over `components/` and `app/`.
- `__tests__/fitness/nothing-outranks-the-gate.test.ts` covers rules 13, 42. Asserts the rung order and allows
  `components/layout/main-layout.tsx` alone above the gate.
- `__tests__/fitness/every-control-has-a-test.test.ts` covers rules 1 and 16 indirectly: `scripts/control-coverage-map.mjs`
  censuses every control a person can press and ratchets the ones no spec drives, against
  `__tests__/fitness/baselines/untested-control.json`.
- `__tests__/fitness/a-focus-ring-is-the-recipe.test.ts` covers rule 15. No control ring is spelled outside
  `lib/styles/focus.ts`, every interactive primitive reads the recipe, and the recipe keeps its canvas offset.
- `__tests__/theme/theme-contract.test.ts` covers rules 15, 25, 26, 27, 28: the ring's accent is scored on the
  canvas, on a card and on a sheet. `npm run theme:contrast` prints the same
  table with headroom while a theme is being tuned.
- `local/no-unpositioned-tap-target` covers rule 23.
- `local/no-hardcoded-ui-text` covers rule 1, for `aria-label`, `title`, `placeholder`, `alt` and friends.
- `__tests__/fitness/a-tap-target-is-a-control-not-a-wrapper.test.ts` covers rule 22.
- `__tests__/fitness/a-decorative-glyph-is-hidden.test.ts` covers rule 7. Every Phosphor glyph with a
  text-bearing sibling and neither `aria-hidden` nor a name, ratcheted against
  `__tests__/fitness/baselines/glyph-beside-its-own-label.json`.
- `__tests__/fitness/a-page-has-one-h1-and-one-main.test.ts` covers rules 9, 10, 11. Resolves every
  `app/**/page.tsx` through its imports and its layouts: a second `<main>` in what the route renders, a second
  `<h1>` in what the page itself renders, and a level skipped inside a file or across a route. A ghost's or a
  not-found page's heading is not a second one, because each stands in for the whole screen `[loading-and-states]`.
- `__tests__/fitness/focus-follows-the-reading-order.test.ts` covers rules 14, 20. No positive `tabIndex`, no
  `order` utility painting a box away from its node, nothing but `lib/hooks/use-focus-trap.ts` taking Tab, and
  that hook still restoring focus to the element that opened the surface. Walking the rendered tab ring is
  `e2e/`'s.
- `__tests__/fitness/a-preference-is-a-token-not-a-branch.test.ts` covers rule 29, by AST: a reduced-motion or
  reduced-data read may pick a value, never a second JSX tree.
- `__tests__/fitness/a-live-change-is-announced-once.test.ts` covers rules 34, 35, 36, 38, 39. Censuses every
  `aria-live`, `role="status"`, `role="alert"` and `role="log"` site, then fails a region on the control that
  was pressed, an interruption that is not a refusal, a nested pair, and — against
  `__tests__/fitness/baselines/region-mounted-with-its-message.json` — a polite region that mounts with its
  first message. It also holds `components/ui/toast.tsx` to picking its role by severity.
- `__tests__/fitness/progress-a-person-waits-on-is-readable.test.ts` covers rule 40. A determinate fill driven
  by a job's percentage carries `role="progressbar"` with `aria-valuenow` or a polite line; a transport that can
  be seeked is a playhead, not a wait.

**Not yet enforced**

- Rule 24, `a-hit-area-clears-its-neighbour`: measures rendered hit rectangles in a browser and fails when two
  independent controls overlap or sit flush. Blocked: jsdom computes no layout, so this is `e2e/`'s to measure.

## Traps

- **22.** `tap-target`'s 44px `::after` on a wrapper draws over the wrapper's own children; the author pill's
  name and location taps landed on the pill, and the avatar only worked because it had been patched `z-10`.
- **21.** `scrollWidth` is unusable as an overflow signal once `tap-target` is in the tree: the pseudo-element
  inflates every ancestor, and 18 clean routes measured as overflowing.
- **19.** A global key handler mounted in a layout owns the key on every page; the live stage owned Escape
  with no session, so Escape on any dialog also walked the page back.
- **30.** The blanket reduced-motion `transition-duration: 0.01ms` swallowed the overlay panel, so every sheet
  teleported under OS Reduce Motion instead of fading.
- **42.** Three full-screen takeovers sat on `Z_INDEX.max`, so a child who hit the parental lock while the
  image viewer was open saw the lock render underneath it.
- **1.** A colour swatch shipped as an empty `<button>` and a Shorts publish toggle as a bare `Switch`, both
  announced as nothing but their role; the primitives that require a name were bypassed, not missing.
- **17.** `FOCUSABLE_SELECTOR` matches a control that is `visibility: hidden`, and nothing can take focus while
  it is; the trap filters on `checkVisibility()` before choosing.
