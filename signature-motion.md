# Signature motion

The grammar in `[motion]` makes every animation correct. This document makes the product feel the way the best
Apple software feels: what the person remembers after they put the phone down. A photo that becomes the bubble.
A card that becomes its page and folds back into its place. A short that docks beside its comments in one
breath. A capsule that inflates to show what is still happening. A glow that listens. Each is one primitive,
tuned once, with the physics Apple uses, so it is never a version per feature and never a cheap imitation. If a
surface cannot carry one of these at this quality, it carries the plain sentence from `[motion]` instead.

## Owns and defers

Owns: the physics of springs in the terms Apple's designers use and the mapping onto Surf's springs; response,
redirection and momentum; shared-element morphs; numeric rolls and glyph effects; large-title and chrome
response; route transitions; the live activity capsule; the AI listening edge; the haptic and sound rules for
these moments.

Defers: the duration ladder, the curves, the compositor rules and reduced motion to `[motion]`; the primitives'
files to `[components]`; the capsule's material and the ribbon's colours to `[colour-and-effects]`; the haptic
rungs and the sound catalogue to `[feedback]`; the chunk warming that lets a morph start on time to `[speed]`;
gesture thresholds to `[motion §34]`.

## The physics, once

Apple describes a spring by two perceptual numbers, and so does this document: **duration**, the time the motion
reads as taking, and **bounce**, from 0 (no overshoot, a smooth settle) to 1. Duration is perceptual on purpose: a
spring never truly stops, so nothing waits for it to settle; a change that follows a spring fires at the
perceptual duration. Bounce above 0.4 reads as exaggerated on a control. Bounce 0 is the general-purpose
spring; 0.15 reads brisk rather than bouncy; 0.3 is noticeably playful and belongs to gestures with momentum,
because a gesture that ends with momentum and no overshoot feels broken.

A spring's other virtue is that it takes any starting velocity: the velocity the finger had when it let go, or
the velocity the element already had when its target changed. That is why every motion here is a spring and
why nothing here is a tween: a tween cannot be caught, redirected or handed a fling.

Surf's springs, in `lib/styles/motion.ts`, expressed in those terms (mass 1; duration is 2π over the square
root of stiffness; bounce is 1 minus damping over twice the square root of stiffness):

| Spring | Stiffness / damping | Duration | Bounce | Use |
|---|---|---|---|---|
| `SPRING_SHEET` | 400 / 35 | 0.31 s | 0.12 | Content: every morph, the capsule, a sheet released, a viewer settling |
| `SPRING_SNAPPY` | 400 / 34 | 0.31 s | 0.15 | Indicators and small chrome: a pill, an underline, a docked player, a swipe row snapping, the like pop |
| `SPRING_GENTLE` | 200 / 25 | 0.44 s | 0.12 | A large surface settling after a slow gesture |

Apple's own presets for reference, so a tuning conversation has a shared vocabulary: smooth is 0.5 s at bounce
0, snappy is 0.5 s at 0.15, bouncy is 0.5 s at 0.3. Surf runs shorter (0.31 s) because a web page has no
system-wide motion to stay in step with and the owner's bar is calm.

Two formulas, used wherever a gesture ends:

- **Projection.** Where a fling would stop if nothing caught it: position + velocity × rate ÷ (1 − rate), with
  velocity in pixels per millisecond and rate 0.998 for a list (Apple's normal deceleration) or 0.99 for a
  surface that should stop sooner (a docked player, a carousel). The rest position is chosen from the
  projected point, never from the release point.
- **Rubber band.** Displacement past an edge: x × d × c ÷ (d + c × x), with d the dimension being pulled (the
  height of a sheet, the width of a photo) and c 0.55, Apple's constant. The band is symmetric on release and
  springs back on `SPRING_SHEET`.

## Rules

### Response and redirection

1. **The first frame answers.** The pressed state is CSS `[feedback]`; the element the gesture will move is
   already under the finger when it moves. Latency is a defect, not a tuning.
2. **Highlight on touch down, confirm on touch up.** A press that drifts outside the control cancels; a press
   that drifts back re-highlights. The hit area is wider than the drawn control so a slight drift does not
   cancel `[accessibility]`.
3. **A swipe declares itself at 10 px** of travel (hysteresis), then tracks one to one from that point. Before
   it, the tap and the swipe both stay possible; the surface never waits on a timer to decide.
4. **One to one, always.** While the finger is down, the element is where the finger says, with no easing, no
   lag and no minimum step. The moment tracking breaks the person feels it, so the only break allowed is the
   rubber band at an edge, which is itself a message.
5. **Everything is redirectable mid-flight.** A sheet closing can be caught and reopened; a page pushing can be
   pulled back; a card zooming can be pinched down. A push that is interrupted becomes a pop, never a cancel.
   Nothing is disabled "while animating".
6. **Respond to the change in motion, not to time.** A pause in a drag is read from the finger's acceleration,
   not a timer; a fling is read from the last several samples `[motion §33]`.

### Shared element

7. **Whatever becomes something else travels there.** The two surfaces carry one `layoutId`, named once in
   `lib/constants/shared-elements.ts` (not yet built) as `<concept>-<id>`, so one node's position, size and
   corner radius interpolate on `SPRING_SHEET` and nothing is unmounted and redrawn.
8. **The pixels stay; only the box moves.** During a morph the image, the text and the ink never fade, blur or
   crossfade. A crossfade inside a morph is two elements pretending to be one.
9. **The morph starts in the frame of the cause.** The send, the tap, the release of a drag. It never waits
   for a chunk, a request or a mount; the destination is warmed on intent `[speed]` and the destination's
   data is the optimistic copy `[resilience]`, so the morph lands on something real.
10. **The list makes room as the element lands.** Rows on either side glide on transform while the morph
    settles, on the same spring, so arrival and reflow are one motion.
11. **A morph reverses along its own path.** Back, Escape, a drag down or a tap outside carries the element to
    the frame it came from, and that frame is still there because placement state survives to unmount
    `[motion §28]` and the list is at its offset `[navigation-and-history]`.
12. **Aspect changes are cropped, not stretched.** When source and destination differ in aspect (a square
    thumbnail, a portrait viewer) the image is clipped to the morphing box with its centre held, never scaled
    non-uniformly.
13. **Corners are continuous through the morph.** The radius interpolates from the source rung to the
    destination rung on the same spring, and a corner that lands at 0 goes there through the ramp, never in a
    step.

### Momentum, projection and edges

14. **A fling lands on a rest position, never in between.** A docked player, a reorder drop, a carousel page,
    a sheet detent, a swipe row: the release velocity is projected (formula above) and the nearest rest to
    the projected point is chosen, then reached on the spring with the release velocity as its start.
15. **Every edge rubber-bands.** The end of a list, the top of a sheet, the edge of a zoomed photo, an
    over-drag on a swipe row, with `SHEET_DRAG` elastic values where the drag is a sheet and the formula above
    elsewhere. A hard stop is a cut.
16. **Paths are symmetric.** What enters from the right leaves to the right; what rises from the bottom sinks
    to the bottom; a sheet dragged past its threshold leaves in the direction of the drag.
17. **A surface hints its destination.** A sheet being dragged grows toward the size it will be; a card being
    pulled grows toward the page. The motion under the finger is the first frames of the motion after it.
18. **Weight matches meaning.** A photo is light (lower damping, quicker settle); a page is heavy (higher
    damping, no overshoot). Two surfaces of the same kind never carry two weights.
19. **Reorder by long press lifts the row.** `press-firm` scale with `--shadow-lg`, one `medium` haptic when it
    lifts; siblings part on transform as it moves; it settles into the gap on `SPRING_SHEET` and the same
    haptic marks the commit. (not yet built)

### Presence and continuity

20. **A changing value rolls.** The digit that changes rolls out along one axis while the new digit rolls in
    at `normal` on `--ease-dialog`; digits that did not change stand still, in tabular figures; a count that
    grows by more than one rolls each column once, not once per increment. Never a repaint of the whole number.
    (not yet built)
21. **A glyph performs its change once, on its cause.** Four kinds, after SF Symbols: a **discrete** effect
    plays once on an event (the heart's fill beat, the bell's ring on a new notification, the bookmark's
    drop); an **indefinite** effect holds while a state is true and ends when the state ends (the mic glyph
    breathing with the level, a variable-fill "connecting" that reverses); a **transition** effect brings a
    glyph in or out of the layout; a **content transition** replaces one glyph with another in place (play
    to pause). One effect per glyph per cause; never on mount; never on a loop unless it is status.
22. **A large title condenses into the bar.** On a page that opens with a large title the title folds into the
    header on transform, driven by scroll position, and unfolds on the way back; the bar never changes
    height. (not yet built)
23. **Chrome responds to what passes under it.** The frosted header and tab bar strengthen their frost and
    their hairline as content scrolls beneath and relax at rest; `[colour-and-effects]` holds the material.
    (not yet built)
24. **Text an AI rewrites shimmers, then dissolves.** A band sweeps the block at `--duration-shimmer` while the
    rewrite is in flight; the new text crossfades in at `fast`. The block is never replaced in a frame, and
    the person can leave the rewrite while the band is sweeping `[ai §6]`. `AiShimmer` is the one treatment
    `[ai §8]`.
25. **Two things never arrive together toward the same attention.** A morph and a toast, a roll and a burst:
    the second waits for the first's perceptual duration `[motion §59]`.

### Route transitions

26. **Push and pop are one surface over another.** On a phone a push into a detail page (a post, a profile,
    a thread, a settings subpage) slides the page in from the inline end at `slow` on `--ease-dialog`, as one
    `transform` string on the compositor (`slidePageIn`), over the kept root it covers, which recedes
    `PUSH_PARALLAX_PERCENT` of the screen and dims to `PUSH_DIM_BRIGHTNESS`. A swipe back drives the page off one
    to one over the kept root it uncovers (`peekRoot`), or over the ghost of a page that is not kept
    (`SwipeUnderlay`), written to the page directly so it never waits on a render. A tab switch is never a push
    but a reveal `[navigation-and-history §24a]`; reduced motion, a traversal and a wait long enough for the
    destination's ghost to stand in all cut. A page leaving by the Back button or the shell's Back is a reveal of
    what is beneath, not a slide off `(not yet built)`: the page is already gone when the route commits.
27. **A row zooms into its page when the page is a larger version of the row.** A post card into the post, a
    photo into the viewer, a conversation row into the thread, a space card into the space, a sound into its
    page. The row is the shared element `[signature-motion §7]`; the page grows from the row's frame; Back
    shrinks it home. Where the page is not a larger version of the row (settings, a form) it pushes instead.
    On the web this is the View Transitions API: `view-transition-name` on the row and on the page's header,
    `startViewTransition` around the navigation. (not yet built)
28. **A zoom is interactive from its first frame.** A drag down or a pinch during the zoom reverses it with the
    finger's velocity; an interrupted open becomes a close. The identity captured for the transition is the
    row's id, never the row's node, because the node is recycled by the list.
29. **Desktop never travels horizontally.** Content swaps under the route ghost with `animate-reveal`; only a
    shared element moves, and it moves the same way at every width.

### The live activity capsule

30. **What is still happening lives in one capsule.** An upload, a render, a recording, a call, a live stream
    being watched, a scheduled send, a download: each is a live activity shown in one persistent capsule in
    the app frame, `components/layout/live-activity-capsule.tsx`. Never a spinner in a card, a toast and a
    floating window that each invent their own shape. A job registers with `startLiveActivity`, names its own
    glyph and the screen it belongs to (`href`), and calls `end` from its screen's cleanup when it dies with that
    screen. On its own screen the screen draws the wait and takes over how it ended (`useLiveActivityOutcome`);
    the capsule speaks for it everywhere else. One activity per thing: a job started again at the same place
    replaces the one there. A job that needs the person opens the capsule by itself; where it has a screen,
    Open is the primary action and the job's own (Dismiss) sits beside it. A minigame build, a Studio run
    (`useStudioRunActivities`, which also looks for runs the server starts on its own) and a short's export
    (`useShortExport`) are on it; captions are not yet. An export has no `href`: one for publishing lives and
    dies with the edit it was started from, and a saved copy outlives it, waiting in the capsule for the tap
    that shares it.
31. **Three presentations, one surface.** Compact: a glyph and one value (a ring, a timer, a level), as narrow
    as the content allows and snug to its anchor. Minimal: when two activities are live, the second is a
    detached circle beside the first. Expanded, on long press: the controls (pause, stop, end, go to it) and
    the one detail, hugging its content with no empty band at the top. The three are one surface that inflates
    on a `clip-path` opening out of its old size and deflates on a scale into its new one, both on `SPRING_SHEET`
    from sizes cached at the last change (`useFlipSize`); the detached circle glides on `translate`
    (`useFlipPlace`). It never disappears and reappears, and nothing measures per frame.
32. **A tap goes to the thing; a long press expands.** Nothing else opens from the capsule, and its expanded
    state never covers the toast or the gate `[feedback]`, `[accessibility]`.
33. **It is a canvas of foreground elements.** No background imagery, no wash; a glyph, a value, a ring, in the
    theme's accent for identity and the ink tiers for the rest. Concentricity is the test: blur the content
    and the resulting shape must sit centred inside the capsule with even margins.
34. **Placement holds between presentations.** The glyph that is on the left when compact is on the left when
    expanded; the value that is on the right stays on the right, so the eye never re-finds anything.
35. **Numbers roll, elements replace.** A timer or a progress value uses the roll `[signature-motion §20]`; a
    glyph that changes uses a content transition; a row that moves in a list moves alone while the others
    fade, never overlapping.
36. **An alert expands the capsule; it does not toast.** When an activity needs attention (an upload refused,
    a call waiting) the capsule expands on its own with the reason and the one action, and the element that
    caused the alert is emphasised in the expansion.
37. **Completion collapses it, once.** The capsule deflates with one `success` haptic and the result lands
    where it belongs (the post at the top of the list, the clip in drafts, the call in the log), morphing out
    of the capsule when the destination is on screen. An ended activity is gone within a few seconds; it never
    lingers as a badge.
38. **Solid, with continuous corners** `[motion §41]`: it moves on a spring more often than it rests, and a
    blur that scales smears. The capsule's corner radius scales with its height so the shape stays concentric
    at every size, and the radius interpolates with the size on the same spring.

### The AI listening edge

39. **Listening is shown at the edge of the screen, once, everywhere.** Voice search, "Ask about this post",
    the editor's agent, eye-scroll calibration, the live Ask, the verification reading: the same
    `components/shared/ai-listening-edge.tsx`, fed the input level through a ref so it never re-renders. The
    reading and voice search draw it, and the eye-scroll calibration while the camera reads the face, swelling with
    the nod it asked for; the others still draw their own indicator and move to it `(not yet built)`. Never a
    spinner, a pulsing dot, a waveform or a bespoke glow: the calibration's lit edge bands are gone.
40. **The ribbon is four layers, from the outside in.** A wide soft bloom at low alpha and a large blur, a
    mid band at moderate alpha and blur, a narrow core at high alpha and slight blur, and a hairline centre
    with no blur. Their widths step down roughly 1.5, 0.8, 0.3 and 0.1 of the ribbon width; their alphas step
    up roughly 0.15, 0.3, 0.7 and 0.95. The bloom fades inward to nothing; there is no hard inner edge.
41. **The ribbon follows the screen's own corner.** It sits inside the safe area and turns the corner on the
    device's continuous radius, so it reads as the screen lit from its edge, never as a rectangle drawn on it.
    `[colour-and-effects]` holds the radius token and the colours. No platform tells a page its display's corner
    radius, so `--ai-edge-radius` is a fixed approximation inside the safe area: a deliberate choice, not a
    measurement.
42. **The colour drifts along the perimeter; it never spins.** The gradient's stops travel around the edge at
    `--duration-gradient-drift` for a full circuit, always in one direction, by moving the gradient's anchor
    points rather than rotating anything, so it costs nothing per frame.
43. **The level is visible in the same frame.** The ribbon's width and the core's brightness follow the input
    level unsmoothed, so the person sees themselves heard as they speak; silence lets it rest at its base
    width, never at zero.
44. **It appears at `fast` and collapses into the answer.** On listening start the ribbon fades and grows in
    from the edge; on listening end it stops drifting the same frame and collapses toward the surface that
    carries the response, which morphs out of the edge `[signature-motion §7]`. It never loops after
    listening ends, never appears on mount, never plays for an error.
45. **It is quality or it is absent.** The ribbon has its own reference frames in the contact sheet `[craft]`;
    under `perf-lite` it becomes a two-pixel solid accent edge with no blur; under reduced motion it holds at
    its base width and drifts nothing; a surface that cannot host it at that quality shows the static glyph
    and no glow.

### Haptics and sound at these moments

46. **Causality.** A haptic or a sound at a signature moment plays on the frame the visual lands, never before
    the motion starts and never after it settles, so the source of the feedback is obvious.
47. **Harmony.** The three modalities agree: a small element that lands makes a small tick, a large surface
    that settles makes a soft thud or nothing; a sound and a haptic for one moment start in the same frame.
48. **Utility.** These moments earn feedback because something physical happened: a lift, a commit, a landing,
    a completion. The morph itself, the roll and the ribbon are silent; `[feedback]` names the rungs.

## Phone and desktop

One vocabulary, two postures `[layout]`. The phone is touch, one hand, a screen that is the whole world; the
laptop is a pointer, a keyboard and a window inside a larger screen. A signature that ignores the difference
is wrong on one of them.

| Signature | Phone (touch, under 768) | Desktop (fine pointer, 1024 and up) |
|---|---|---|
| Response | Highlight on touch down; hysteresis 10 px; one-to-one under the finger | Highlight on pointer down; hover at rest `[colour-and-effects]`; a drag is one-to-one under the pointer; every action also has a key `[navigation-and-history]` |
| Shared element | Morphs travel the full screen; the reverse is a drag as often as a tap | Morphs travel inside the reading column and the viewer; the reverse is Escape, Back or the close control; no drag-to-dismiss on a mouse, a trackpad swipe-back is the platform's own |
| Send morph | The composer sits over the keyboard; the bubble lands at the bottom of the thread, and the list scrolls to keep it under the composer | The composer is docked; the bubble lands in place and the list does not scroll unless it was at the end |
| Card to page, zoom | Full-screen page grows from the card; drag down or Back returns it | The page grows into the reading column beside the rail; the rail and right panel do not move; Escape or Back returns it. Never a full-window takeover for a post |
| Thumbnail to viewer | Full-screen viewer; drag down dismisses; pinch zooms | Viewer centred over a scrim within the window; scroll wheel zooms; Escape and the close control dismiss; arrows step |
| Short to comments | The short docks above the sheet; one drag drives both | The comments are a side panel; the short stays at its size; nothing docks |
| Reorder | Long press lifts; the finger carries the row | Press and hold with the pointer lifts; the row follows the pointer; the keyboard moves a focused row with modifier and arrows |
| Push and pop | The page slides over the page; swipe-back drives it | Never: content swaps under the route ghost; only the shared element moves |
| Large title | Folds into the bar on scroll | The header bar holds one size; no large title |
| Chrome response | The tab bar and header respond to the list beneath | The rail and the reading column's header respond; the right panel does not |
| Live activity capsule | One capsule at the top of the frame beneath the safe area; compact by default; long press expands; it never covers the tab bar | One capsule docked at the foot of the rail (or the bottom of the reading column on the messenger shell); hover expands it, a click goes to the thing; Escape collapses it |
| AI listening edge | The ribbon runs the whole screen's perimeter inside the safe area, on the device's continuous corner | The ribbon runs the app window's perimeter inside the viewport, on the window's corner radius; if the window is narrower than the reading column it still runs the whole viewport, never a panel |
| Haptics | Every moment in `[feedback]` | None: there is no engine. The sound and the visual carry the moment alone, so the visual landing must be unambiguous without the tick |

Rules that follow from the table:

49. **A signature is written for both postures or it is not written.** Every catalogue row names its phone
    and its desktop behaviour before the primitive is built; a primitive that takes a `posture` from
    `MEDIA_QUERIES` rather than reading the width itself.
50. **Touch drives, pointer clicks.** On a phone the person's finger is the animation's source of truth
    during a gesture. On a desktop a drag is rare and a click is the cause, so the spring starts from rest
    with no velocity and the perceptual duration is what the person sees; it is the same spring.
51. **Hover is a desktop signature and never a phone one.** The capsule expands on hover, a card blooms on
    hover, a control's tooltip arrives on hover, all under `(hover: hover)` only; on touch each has a press
    or a long press instead, and nothing depends on hover alone `[platform]`.
52. **The keyboard is a desktop gesture.** Every signature that a drag reverses on a phone is reversed by
    Escape on a desktop in the same motion, and every rest position a fling can reach, an arrow key can
    reach.
53. **Desktop never takes the window.** A phone signature may own the whole screen; a desktop signature
    stays inside the reading column and the viewer, so the rail, the right panel and the person's other
    windows are undisturbed.
54. **The in-between width is the phone.** Between 768 and 1023 every row reads as the phone column with a
    pointer's hover added, never a third design `[layout]`.

## The catalogue of signatures

Every row is one primitive and one spring. `[motion]`'s choreography catalogue carries the same rows so a
reader of either document finds them.

| Signature | Trigger | What travels | Spring | Reverse |
|---|---|---|---|---|
| Send morph | The send | The composer's thumbnail or text becomes the bubble | `SPRING_SHEET` | Undo or failure carries it back into the composer |
| Card to page | Tap on a card | The card's frame becomes the page; the page's header is the card's header | `SPRING_SHEET` | Back, or a drag down, shrinks it home to the list at its offset |
| Thumbnail to viewer | Tap on media | The image's box grows to the viewer; the scrim fades in with it | `SPRING_SHEET` | Drag down past threshold or tap; the image returns to its card |
| Avatar to profile | Tap on an avatar or name | The avatar becomes the profile header's avatar | `SPRING_SHEET` | Back |
| Story ring to story | Tap on a ring | The ring expands to the stage; between people the stage turns as one surface | `SPRING_SHEET` | Drag down or tap collapses to the ring |
| Short to comments | Tap on comment | The short docks to its small frame while the sheet rises, one progress value, one drag | `SPRING_SHEET` | Drag down or close restores the short in the same motion |
| Reorder | Long press | The row lifts, siblings part, the row settles in the gap | `SPRING_SHEET` | Release outside a gap returns it home |
| Numeric roll | A value changes | The changed digit | `normal`, `--ease-dialog` | n/a |
| Glyph effect | An event or a state | The glyph, once | `SPRING_SNAPPY` | n/a |
| Large title | Scroll | The title folds into the bar | scroll-driven | Scroll back |
| Push and pop | Navigation on a phone | The page over the page | `slow`, `--ease-dialog`; swipe-driven | Pop |
| Zoom | Tap on a row whose page is a larger version of it | The row's frame becomes the page | `SPRING_SHEET` | Drag down, pinch or Back |
| Live activity capsule | An activity starts, is long-pressed, alerts, completes | The one capsule between compact, minimal and expanded | `SPRING_SHEET` | Deflates |
| AI listening edge | Listening starts and ends | The ribbon; the answer morphs out of it | `fast` in; drift at `--duration-gradient-drift` | Collapses into the response |

## Enforced by

- `framer-ships-once` (7, 30, 39): the shared-element, capsule and ribbon primitives ship through the one
  Framer root.
- `a-press-reads-one-ladder` (1, 19): the lift is the press ladder, never a private scale.
- `every-css-var-resolves` (40 to 43): the ribbon's tokens resolve where they are read.
- `a-shared-element-is-named-once` (7): a morph's name is never spelled out where it is used, so the two
  ends of it cannot drift apart.
- `a-morph-carries-no-crossfade` (8): a node that is travelling animates no opacity and no blur.
- `a-spring-stays-under-the-bounce-ceiling` (physics, 7): every spring is declared in `lib/styles/motion.ts`
  and settles at or under 0.4, and every morph runs on one of them.
- `one-listening-edge` (39): a surface that is hearing the person draws no spinner, pulse or waveform.
- `one-capsule-per-frame` (30): the app frame mounts the one capsule, once in each of its branches, and every
  job reaches it through `startLiveActivity`, never by writing its store.

**Not yet enforced.**
- `a-fling-projects` (14): every release handler that chooses a rest position calls the one projection helper.
  Nothing in the tree carries the projection formula yet.
- `a-value-rolls` (20): a count rendered from a number that can change goes through the rolling number. The
  rolling number is the primitive rule 20 asks for.
- `a-glyph-performs-once` (21): no glyph effect on mount and no looping effect off a status prop. No glyph
  carries an effect yet; the looping half is `a-loop-is-a-status`.

## Traps

- **7** The short's dock was keyed to the comment sheet's mount, and the sheet was a lazy chunk, so the short
  moved half a second after the tap and after the sheet. Key the morph to the tap; warm the chunk on intent.
- **9** A morph that waits for the server's row lands on nothing and then jumps when the row arrives. The
  optimistic copy is the destination.
- **11** Placement state cleared on close made the reverse morph aim at screen centre. Keep it to unmount.
- **14** A docked player that sprang to the corner nearest the release point, not the projected point, went the
  wrong way on every fast flick.
- **21** A bounce effect wired to a prop's presence replayed on every re-render. Trigger on a value change.
- **31** A capsule that unmounted between compact and expanded lost its velocity and flashed. One node, one
  `layoutId`, three layouts.
- **42** Rotating a gradient to make it drift re-rasterised the whole edge every frame. Move the anchor points.
- **45** An edge glow drawn with a rectangular corner over a rounded screen read as a frame, not as light.

## Sources

Apple's own material, distilled: "Designing Fluid Interfaces" (WWDC 2018), "Animate with springs" and
"Design dynamic Live Activities" and "Animate symbols in your app" (WWDC 2023), "Enhance your UI animations
and transitions" (WWDC 2024), "Practice audio haptic design" (WWDC 2021), the Human Interface Guidelines on
motion, Live Activities and playing haptics, and the scroll-view deceleration and rubber-band constants from
UIKit.
