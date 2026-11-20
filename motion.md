# Motion

Motion is the product's body language. It says where a thing came from, where it went, what the person just
did, and what is still happening. This document owns every duration, curve, spring, gesture physic and
choreography in the interface, and the discipline that keeps all of it on the compositor. The feeling it
protects: one continuous material that folds, slides and settles. Nothing pops into existence, nothing
vanishes, nothing bounces for fun, and nobody ever waits for an animation.

## Owns and defers

Owns: the four jobs, the duration ladder, the curves and springs, the nine sentences, the choreography of
every surface, gesture physics and thresholds, compositor discipline, reduced motion, what never moves.

Defers: token values and the mirror-parity rule to `[tokens]`; which overlay primitive a surface uses and what
it owns to `[components]`; blur, scrim and sheen values to `[colour-and-effects]`; when a skeleton is allowed
and how long the hold lasts to `[loading-and-states]`; frame budgets and how motion is measured to `[speed]`;
haptics and sound to `[feedback]`; Back, Escape and history to `[navigation-and-history]`; the motion
preference as an accessibility setting to `[accessibility]`; mirroring direction for RTL to
`[internationalisation]`; playback, scrub and picture-in-picture behaviour to `[media]`.

## Rules

### The jobs motion does

1. **Four jobs, nothing else.** A motion orients, shows causality, carries continuity, or reports status. A
   motion that does none of the four is removed, not shortened.
2. **Status is the only loop, and atmosphere is the one exception.** `.animate-spin`,
   `skeleton-shimmer`, `typing-bounce`, an indeterminate progress ring and the playing `sound-disc-spin`
   may repeat, because each reports something. Nothing else repeats, because a loop with no job never
   stops asking for attention — except atmosphere, `aero-bubble-drift`, `glow-orb-drift` and the entry
   screens' `entry-feed-rise`, which earns the loop by being shed under BOTH `prefers-reduced-motion` and
   `html.perf-lite`. One of the two is not enough.
3. **One burst per flow.** At most one deliberate moment of delight in a journey, answering something the
   person did. The feed's is the double-tap heart. Never for something the app did, never on chrome.
4. **Absence is a decision.** A state that changes in place with no motion reads as a glitch beside a
   neighbour that animates. If one member of a family animates, all of them do.

### The ladder and the curves

5. **Every duration is a rung.** A key of `TIMING`, a key of `ANIMATION_DURATION`, a
   `--transition-duration-*` variable, or one of the named per-element constants below.
   `motion/no-literal-motion` fails the build on a literal duration, delay or curve in `app/**`,
   `components/**` and `lib/**`.
6. **The larger and further, the longer, and never past 400ms.** The one exception is the burst at
   `ANIMATION_DURATION.extraLong`. A person never waits for motion; they only ever notice it.
7. **Exit is 0.8 of enter.** `EXIT_DURATION_RATIO` in TypeScript, `calc(var(--ovl-dur) * 0.8)` in CSS. Leaving
   gets out of the way faster than arriving asked to be looked at.
8. **Arrivals decelerate, departures accelerate.** An arriving element takes `--ease-dialog` or
   `--ease-enter`; a leaving one takes `--ease-exit`. Nothing arrives on an accelerating curve.
9. **In-place change takes the standard or gentle curve.** `--ease-smooth` for colour, opacity and a size
   carried on transform; `--ease-gentle` for a quiet crossfade such as the skeleton swap.
10. **Every transition names its curve.** A Framer `transition` with a duration and no `ease` silently takes
    Framer's default, which is in neither table. A CSS `animation` shorthand with a bare `ease` does the same.
11. **A gesture releases into a spring, never a curve.** A spring inherits the finger's velocity; a curve
    throws it away. `SPRING_SHEET` for sheets, `SPRING_SNAPPY` for rows and indicators, `SPRING_GENTLE` for a
    large surface settling. A tap hands off no velocity, so a tap-driven move may use a duration.
12. **The cinematic curves are rationed.** `EASE_ARRIVE` settles a long pull or a ceremony. `EASE_BACK`
    overshoots, so it takes one element at a time and never a group. Nothing a person touches overshoots
    past a brisk settle `[signature-motion]`.
13. **A private easing set is a second motion language.** No component defines its own curves or its own
    timing table. Curves live in `lib/styles/motion.ts` and `app/globals.css`, durations in
    `lib/constants/timings.ts`, gesture numbers in `lib/constants/gestures.ts`.
14. **Rendered video is not interface.** The scene DSL under `lib/editor/scene-templates/` resolves its own
    names through `resolveEase`. That vocabulary is exported content and never reaches a component.

### The nine sentences

15. **Every animation is one of nine sentences.** Arrive, leave, change in place, move, morph, roll, reveal in
    sequence, follow, burst. Pick the sentence, then the rung, then the curve. Anything else is a new sentence and needs
    this document changed first.
16. **Nothing arrives from nowhere.** An arrival is opacity 0 to 1 plus a short travel on transform from the
    side the surface belongs to. A scale from 0 is the burst and nothing else.
17. **One travel per kind of arrival.** A dialog rises 12px, an in-page reveal rises 8px, an anchored popover
    nudges 4px from its corner. A surface does not invent a distance. `(not yet built)`: there is no travel
    token, and eight different rises are in the tree; `MOTION_RISE` beside `TIMING` would end it. (not yet built)
18. **Change in place is opacity only.** Two states of one element crossfade at `--transition-duration-fast`
    on `--ease-gentle` through `state-fade-in`. Never a slide, never a scale, so an icon swapping outline for
    fill and a count ticking never move the row.
19. **Move is one element, not two.** A thing that travels between two positions keeps its identity: one node
    that is never unmounted at the old place and redrawn at the new one. Inside a strip it is `SlidingFill`,
    which animates a transform string on `SPRING_SNAPPY` from cached geometry; across two surfaces it is
    `layoutId`, which measures during the commit and runs in JavaScript.
20. **A sequence steps once and stops.** Rows that arrive together arrive in order through
    `animate-cascade-up` with `--reveal-delay`, one step of `ANIMATION_STAGGER`, `both` fill so a row holds
    its start state through its delay, capped so the last row never waits on the first.
21. **A server-rendered reveal is a class.** `animate-reveal`, never a Framer `initial`, which serialises
    `opacity: 0` into the HTML and gates first paint on hydration. A client surface that mounts with its
    parent passes `initial={false}` or sits under `<AnimatePresence initial={false}>`.
22. **Follow is 1:1 and unsmoothed.** During a drag nothing eases; the element is exactly under the finger,
    with the transition removed imperatively.
23. **A burst answers a person.** Scale 0 to 1.2 to 1 and fade over `ANIMATION_DURATION.extraLong`, matching
    `DOUBLE_TAP_HEART_DURATION_MS` so the timer and the visual cannot drift (`--burst-ms` mirrors it). Under
    reduced motion the scale becomes an opacity pulse. It is `HeartBurst`, a CSS keyframe animation, in the feed
    and on a short alike: one primitive, no script per frame.

### Overlays and presence

24. **Overlays open on CSS transitions, not on JavaScript.** The main thread is busiest at the instant an
    overlay mounts, because React has just committed the portal, and a JS-driven open drops its first frames.
    `useCssPresence` plus the `.ovl-panel` and `.ovl-scrim` rules are the path.
25. **One duration source per overlay.** The component writes `--ovl-dur` from `DIALOG_MOTION_MS` or
    `MENU_MOTION_MS` and hands the same number to `useCssPresence`, so the unmount clock cannot drift from the
    visual. `design-token-mirror-parity` asserts the CSS default equals `DIALOG_MOTION_MS`.
26. **The mode chooses the travel.** `data-mode` is `dialog` (fade and a 12px rise), `sheet` (from the bottom
    edge), `side` (from the right edge) or `pop` (scale 0.96 from the anchor corner with a 4px nudge on
    `--ovl-pop-y`). A surface picks a mode; it never writes its own keyframes.
27. **Pointer events stop the instant the close begins.** `data-interactive='false'` is set synchronously
    while the node stays mounted for its exit, so a leaving surface never swallows the next tap.
28. **Placement state survives to unmount.** An anchored surface keeps its anchor, transform origin and
    measured max-height until the exit finishes, so the leave lands where the entrance began instead of
    jumping to the centre of the screen.
29. **The exit starts in the commit that closes, and the unmount clock with it.** `useCssPresence` derives
    `state` closed in that very render and arms its timeout in the same layout effect, with no
    `requestAnimationFrame` between the tap and the exit, and `useExitPresence` holds the retained value for the
    same window, so an overlay never renders a frame of empty content on its way out.
30. **The measured property is never transitioned.** An anchored popover measures its room before it animates
    and excludes `max-height` from its transition. A sheet rides the keyboard on `translate`, never on `bottom`
    or `max-height`, and changes detent at once: a rise is its top gliding up on a transform, and a fold sinks to the
    short rest before the height changes under it.

### Gestures and physics

31. **The finger owns the transform.** A live drag writes `transform` imperatively with the transition
    removed, and restores the exact transition string once the release's spring lands, so what follows is the
    system's motion and not a leftover inline style.
32. **A drag writes a MotionValue, not React state.** A gesture that sets state renders the tree on every
    `pointermove`, which is what costs frames on a mid-range phone.
33. **Velocity is averaged over real samples.** Never one, and never one that includes the pointerdown. A
    single sample read a 29px drag as 29,000px per second, and a repeated final sample read every flick as
    motionless.
34. **A threshold is distance or velocity, whichever comes first.** A short fast flick commits; a long slow
    pull commits. Distance is a share of the surface's own travel, never a fixed pixel count: 30px closed a
    two-row menu and a full form alike. Every number lives in `lib/constants/gestures.ts`, never in the
    component.
35. **One swipe vocabulary.** A row, a bubble and a viewer arm at the same proportion of travel, reveal the
    same glyph ramp through `SWIPE_GLYPH`, and fire the same haptic once, at the arming point, while the
    gesture is still live. Three thresholds sets for one gesture is two too many.
36. **Rubber-band asymmetrically.** Elastic 0.1 against a hard edge and 0.4 toward the dismiss direction, so
    the direction that does something feels available and the direction that does nothing feels closed.
37. **A gesture never fights the platform's own.** Swipe-back at the screen edge belongs to the platform, and
    a sheet drag is bound to its chrome, the grabber and the header, so it never fights the body's inner scroll.
    A control that owns one axis waits to learn which way the finger is going: `lockedAxis` reads the first
    `DRAG_THRESHOLD_PX` of travel, and a tie is the page's. `RangeInput` takes a finger or a pen only once that
    reads horizontal, takes a tap from the browser's own `click` rather than from the press, and puts back the
    value it found when the platform cancels the touch, so a thumb that lands on a slider while scrolling changes
    nothing. A mouse scrolls no page by dragging, so its press takes the track at once.
38. **An abandoned surface returns to rest, never to a parked transform.** A dismissed sheet parks
    `translateY(100%)` inline; the reopen clears it, and a vetoed close clears it on the next frame, or the
    panel sits off-screen holding the scroll lock and the focus trap.

### Compositor discipline

38a. **Nothing that must survive a busy main thread animates in script.** A sheet's release, a pill's move, the shorts
    morph, a toast and a dip are WAAPI animations or CSS transitions on a whole `transform` string, or on
    `opacity`, `clip-path` or `translate`. `animateSpring` and `springTiming` turn a spring into a `linear()`
    easing that carries the finger's velocity; motion's `x`, `y`, `scale`, `layout` and `animate(value, { onUpdate })`
    run in JavaScript, and only a gesture's own frames may write the DOM from script.
39. **Only `transform` and `opacity` animate.** `height`, `width`, `top`, `left`, `inset`, `bottom`, `margin`,
    `padding`, `border-width`, `aspect-ratio`, `box-shadow` on a large surface, `filter` and `mask-image` do
    not. A bar that shows a level scales on `transform: scaleX` from a fixed track, the way
    `audio-controls.tsx` and `STORY_TRACK_FILL` already do.
40. **A disclosure is the one sanctioned layout animation.** `Collapse` transitions `grid-template-rows` from
    `0fr` to `1fr` so nothing measures height in JavaScript. It is the only one, and it is never hand-rolled.
41. **A surface that moves is solid.** Scaling, fading or masking a `backdrop-filter` layer re-rasterises it
    every frame, and translating one re-samples the backdrop behind it. An overlay that animates uses
    `GLASS_OVERLAY_SOLID` or `GLASS_POPOVER_SOLID`; a frosted layer behind a morph glides away instead of
    smearing through it.
42. **Blur is a class, never an inline style.** An inline `backdropFilter` cannot be shed by the low-power
    path, which keys off the `backdrop-blur` class.
43. **`will-change` lives only as long as the movement.** It is set on a node mounted only for the duration of
    its motion, or set and cleared around the animation. Left on a permanently mounted element it holds a
    compositor layer for the whole session.
44. **Framer ships once.** `LazyMotion` with `domAnimation` and `strict` at the root, `<LayoutMotionFeatures />` beside
    it loading layout projection and drag (`domMax`) as a chunk of their own, `m as motion` at every call site, and only
    for shared-element moves, springs and gestures. `framer-ships-once` fails a plain `motion` import.
    Everything that can be CSS is CSS.
45. **A page container is never transformed to fade it.** A transform there makes the container the containing
    block for every `position: fixed` child for the length of the animation. The reduced-motion stand-in for a
    section swipe is opacity only.
46. **Nothing animates while the person is driving something else.** No motion during fast scroll, during a
    drag, or while a field is being typed into, other than the thing the person is driving.

### Reduced motion

47. **Functional motion is shortened, never removed.** Overlays keep a real crossfade at
    `--transition-duration-fast`, shorts paging shortens to `SHORTS_TRACK_TRANSITION_REDUCED`, pinch-zoom to
    `ZOOM_TRANSITION_REDUCED`, and the nav bar keeps its glide at `--transition-duration-nav-glide`.
    `--duration-reduce-motion` is 0.01ms and is the blanket kill value, not a rung any surface asks for.
48. **Travel is removed, presence stays.** `sheetPresence(reduceMotion)` returns `SHEET_FADE`, so the sheet
    fades in place rather than teleporting. A Framer surface whose only animated property is `y` must carry an
    opacity frame, because `reducedMotion: 'always'` snaps transforms and leaves nothing to animate.
49. **Decorative motion is removed.** No cascade stagger, no burst, no ambient drift, no hover bloom. A
    surface never opts a decorative loop back in past the person's preference.
50. **Status motion stays.** `.animate-spin` and `skeleton-shimmer` opt back in by class specificity. A frozen
    bone reads as a hang, not as calm.
51. **A drag under reduced motion writes `important`.** The blanket kill sets `transform: none !important` on
    `.ovl-panel` and crushes every transition to 0.01ms, so an imperative gesture transform that does not
    match it is discarded frame by frame.
52. **Press becomes opacity.** `--press-opacity` replaces the scale, because on touch there is no hover and
    the platform tap highlight is already suppressed, so without a substitute a press is invisible.
53. **Both switches are honoured, at first render.** The OS query and the in-app `reduce-motion` class are
    read together, and the boot script puts the stored choice on `<html>` before React runs so `MotionConfig`
    captures it when it constructs its first component.

### What never moves

54. **Body text.** Colour and size never change on hover, focus or press. The focus ring is drawn, not
    animated.
55. **Chrome under its content.** A header does not shrink when a field focuses. A bar that changes apparent
    size scales on transform; it never relayouts.
56. **A card whose control was pressed.** The press ladder excludes an element with a pressed descendant
    control, so the card stays still and only the control answers.
57. **Content, on scroll.** No parallax. Depth comes from the surface ladder, not from scroll-linked travel.
58. **A live badge, an unread dot, a status blip.** Live is a state, not an event. It fades in and fades out.
59. **Two things toward the same attention.** Never a burst and a toast, never a cascade and a dialog.
60. **A surface that cannot be made smooth.** If a change cannot be made smooth, it does not ship at all.

### Signature motion

61. **The signatures are `[signature-motion]`.** Shared-element morphs, projection and rubber-banding, the
    numeric roll and glyph effects, the large title, chrome response, route transitions, the live activity
    capsule and the AI listening edge are one document with Apple's physics and one primitive each. This
    document's catalogue carries their rows so a reader here finds them; the rules live there.

## The duration ladder

Two tables exist and both are live. `TIMING` in `lib/constants/timings.ts` is milliseconds and mirrors the CSS
scale; `ANIMATION_DURATION` in the same file is seconds and is what Framer transitions read.

| `TIMING` | ms | CSS variable | Job |
|---|---|---|---|
| `micro` | 50 | none | An icon fill swap, a checkbox tick |
| `instant` | 75 | `--transition-duration-instant` | The release after a press, a toggle knob, a stroke tick |
| `fast` | 150 | `--transition-duration-fast` | Colour, opacity, hover, a crossfade between two states of one element |
| `normal` | 200 | `--transition-duration-normal` | A small reveal, a context card, an in-place tab crossfade |
| `slow` | 300 | `--transition-duration-slow` | A page-level reveal, a cascade row, shorts paging, a toast |
| `nav-glide` | 320 | `--transition-duration-nav-glide` | Chrome gliding away, slower than a panel on purpose |
| `slower` | 400 | `--transition-duration-slower` | A full-screen takeover, an image fading over its blurhash |
| `debounce` | 300 | none | Not motion: an input debounce that shares the number with `slow` |
| `feedback` | 2000 | none | Not motion: how long an inline confirmation stays |
| `undo` | 5000 | none | Not motion: how long an undo toast waits |

| `ANIMATION_DURATION` | s | Job |
|---|---|---|
| `micro` | 0.05 | The seconds twin of `TIMING.micro` |
| `instant` | 0.1 | Framer's instant, which is **not** `TIMING.instant` |
| `fast` | 0.15 | The workhorse crossfade |
| `normal` | 0.2 | The workhorse arrival |
| `medium` | 0.25 | A panel slide-in, `TRANSITION_SLIDE_UP`, desktop shorts paging |
| `slow` | 0.3 | A toast, a cascade |
| `slower` | 0.4 | A takeover, the feed re-rank dip |
| `page` | 0.5 | A route-level move, the ceremony root |
| `extraLong` | 0.8 | The one burst, matching `DOUBLE_TAP_HEART_DURATION_MS` |
| `loop`, `loopMedium`, `loopSlow` | 1, 1.5, 2 | Status loops driven from Framer |

Durations that live outside both tables because a component feeds them per element:

| Value | Where | ms |
|---|---|---|
| `DIALOG_MOTION_MS` | `--ovl-dur` on a dialog, sheet or side panel | 240 |
| `MENU_MOTION_MS` | `--ovl-dur` on a desktop menu or popover | 160 |
| `TOOLTIP_FADE_MS` after `TOOLTIP_DELAY_MS` | the tooltip's fade and its hover delay | 160 after 400 |
| `NAV_SWIPE.commitMs` / `.releaseMs` | a section swipe committing or abandoning | 190 / 220 |
| `CAMERA_MOTION.pressMs` / `.photoWashMs` / `.lensSwitchMs` | shutter press, capture wash, lens freeze | 80 / 90 / 260 |
| `--duration-shimmer` | the skeleton sweep across `--shimmer-band` | 1800 |
| `--duration-refresh-spin` | the pull-to-refresh spinner | 1100 |
| `--duration-typing-bounce` | the typing indicator | 600 |
| `--duration-gradient-drift` | an animated gradient the person picked | 6000 |
| `--sound-disc-spin-ms` / `SOUND_DISC_SPIN_MS` | the shorts sound disc | 10000 |
| `--duration-reduce-motion` | the blanket kill value, never a rung | 0.01 |

### The ladder's debt

Documented as it is, and named so a future pass retires it rather than rediscovering it.

- `instant` means 75ms in `TIMING` and 100ms in `ANIMATION_DURATION`. One name, two values, and only the
  reader's choice of table decides which the surface gets.
- `micro` exists in both TypeScript tables and in no stylesheet, so no CSS rule can reach the shortest rung.
  Nothing in the app currently uses either spelling of it.
- `medium`, `page`, `loop` and `loopMedium` exist only in `ANIMATION_DURATION`; `nav-glide`, `debounce`,
  `feedback` and `undo` exist only in `TIMING`. Neither table is a superset of the other.
- 160ms is a real rung, used by every desktop menu and every tooltip, and it has a name in neither table.
- `debounce` duplicates `slow` at 300ms inside the same object.
- Loop durations live in three places at once: `--duration-*` variables, `ANIMATION_DURATION.loop*`, and named
  millisecond constants.
- `EXIT_DURATION_RATIO` manufactures off-table values from on-table inputs: 120, 128, 160 and 192ms are all
  real durations no table names.

## The curves

Five curves are mirrored between CSS and TypeScript; four are Framer only. `design-token-mirror-parity` asserts
every pair, and a rename on either side is recorded there in the same commit.

| CSS | TypeScript | Curve | Character | Use |
|---|---|---|---|---|
| `--ease-smooth` | `EASE_STANDARD` | 0.4, 0, 0.2, 1 | Balanced | Anything changing in place: colour, opacity, a size on transform |
| `--ease-dialog` | `EASE_SMOOTH` | 0.32, 0.72, 0, 1 | Fast start, long soft landing | The house arrival: dialogs, sheets, popovers, cascades, the nav glide |
| `--ease-enter` | `EASE_OUT` | 0, 0, 0.2, 1 | Decelerate | An element arriving from off-screen: slide-up panels, toasts, an image over its blurhash |
| `--ease-exit` | `EASE_LEAVE` | 0.4, 0, 1, 1 | Accelerate | Anything leaving |
| `--ease-gentle` | `EASE_ENTER` | 0.25, 0.1, 0.25, 1 | Soft | A quiet in-place fade, the skeleton crossfade, the typing bounce |
| none | `EASE_SMOOTH_OUT` | 0.25, 0.46, 0.45, 0.94 | iOS deceleration | Shorts paging and the release after a drag |
| none | `EASE_ARRIVE` | 0.16, 1, 0.3, 1 | Expo settle | A long pull settling, a ceremony reveal |
| none | `EASE_BACK` | 0.34, 1.56, 0.64, 1 | Overshoot | One capture or story element popping in |
| none | `EASE_IN_OUT` | 0.4, 0, 0.6, 1 | Symmetric | A motion that reverses along its own path |

| Spring | Stiffness / damping | Use |
|---|---|---|
| `SPRING_SHEET` | 400 / 35 | A dragged sheet released, and the picture-in-picture dock; damped so no gap shows beneath it |
| `SPRING_SNAPPY` | 400 / 34 | Shared-element moves: the nav indicator, the tab pill, a swiped row, a story cube face, the like pop |
| `SPRING_GENTLE` | 200 / 25 | A large surface settling after a gesture |

### The curves' debt

The two naming schemes are cross-wired: `--ease-smooth` is `EASE_STANDARD`, `--ease-dialog` is `EASE_SMOOTH`,
`--ease-gentle` is `EASE_ENTER`, and `--ease-enter` is `EASE_OUT`. Both names are load-bearing today, because
Tailwind's `ease-smooth` utility has to stay the Material curve while the sheet curve has to be reachable from
a stylesheet. The pairing is the contract until one side is renamed, and it is renamed on both sides in one
commit or the mirror spec fails. `EASE_ARRIVE`, `EASE_BACK` and `EASE_SMOOTH_OUT` have no CSS twin, so a
stylesheet that wants one hand-rolls a `cubic-bezier` string from the array.

## The nine sentences

| Sentence | Shape | Rung | Curve | Carried by |
|---|---|---|---|---|
| Arrive | Opacity 0 to 1 plus a short travel from the surface's own edge | 240 dialog and sheet, 160 menu, `normal` in-page | `--ease-dialog` | `.ovl-panel[data-mode]`, `POP_MENU`, `animate-reveal` |
| Leave | The arrival reversed at 0.8 of its duration | `EXIT_DURATION_RATIO` | `--ease-exit` | `.ovl-panel[data-state='closed']`, `POP_MENU` exit |
| Change in place | Opacity crossfade between two states of one element | `fast` | `--ease-gentle` | `state-fade-in`, `TRANSITION_COLORS` |
| Move | One element travels and keeps its identity | spring | `SPRING_SNAPPY` | `layoutId` |
| Morph | One element changes position, size and corner radius at once and becomes the next surface, never overshooting | spring | `SPRING_SHEET` | `layoutId` shared across the two surfaces `[signature-motion §7]` |
| Roll | A digit or a glyph is replaced by rolling the old one out and the new one in along one axis | `normal` | `--ease-dialog` | a rolling number primitive `[components]` (not yet built) |
| Reveal in sequence | Rise 12px and fade, stepped, `both` fill | `slow` or `normal` | `--ease-dialog` | `animate-cascade-up`, `--reveal-delay` |
| Follow | Transform tracks the finger 1:1, then releases | none, then spring | none, then `SPRING_*` | `useSheetDrag`, `SHEET_DRAG`, `useSwipeAction`, `SHORTS_TRACK_TRANSITION` |
| Burst | Scale 0 to 1.2 to 1 and fade | `extraLong` | `EASE_OUT` | `HeartBurst`, used by `double-tap-like.tsx` and `shorts-player.tsx` |

## Choreography catalogue

The exact motion for every surface. A surface that is not here uses the nearest sentence above and is added
here in the same commit.

| Surface | Enter | Leave | Notes |
|---|---|---|---|
| Desktop dialog | Fade and rise 12px, `DIALOG_MOTION_MS`, `--ease-dialog`; the scrim fades in parallel | Reverse at 0.8 | CSS-driven through `.ovl-panel[data-mode='dialog']`, so it runs while React commits the portal |
| Phone sheet | Slide up 100%, `DIALOG_MOTION_MS` | Slide down at 0.8 | Grabber and `DialogCompactHeader` drag through `useSheetDrag` (a press on a control inside is the control's): it takes the grab from the sheet's live offset, mid-enter included, then 1:1 down, rubber-banded up through `followSheet`; `releaseDismisses` commits past `DRAG_DISMISS.commitFraction` of the sheet's own height, or on a flick past `flickFraction` at `flickVelocity`, arming haptic at the commit share. The release, either way, is `SPRING_SHEET` carrying the finger's velocity as a `linear()` easing on a WAAPI transform animation (`animateSpring`), so it runs on the compositor and the close commit never lands in its frames; under reduced motion the panel fades from where the finger left it |
| Side panel, desktop | Slide in from the right edge, `DIALOG_MOTION_MS` | Reverse at 0.8 | `data-mode='side'`; the panel's content crossfades, the panel does not re-enter |
| Menu, popover, context menu | Scale 0.96 to 1 with a 4px nudge on `--ovl-pop-y` from the anchored corner, `MENU_MOTION_MS` | Reverse at 0.8 | Solid surface only. The Framer twin is `POP_MENU`, `POP_ANCHORED`, `POP_MENU_UP`, mirroring the CSS exactly |
| Menu sheet on a phone | The same menu as a bottom sheet at `DIALOG_MOTION_MS` | Reverse at 0.8 | `MenuSheet` hugs its items; the tier is a ceiling, not a floor |
| Phone create deck | Slide up 100%, `DIALOG_MOTION_MS` | Slide down at 0.8 | `.ovl-panel[data-mode='sheet']` on a fullscreen takeover that is not a `Dialog`; `ComposeRouter` holds it mounted for the exit. A warm chunk mounts the deck in the press's own commit through `warmChunk`; a cold one stands `MobileCreateFlowGhost` in on the same slide and the deck takes over at rest. Tabs slide on one 100%-wide track, so no resize listener drives the deck |
| Keyboard rising under a sheet, a dialog or a composer | `useKeyboardLift`: the `translate` property rises by the keyboard's height over the shell's own frame duration and curve (`SurfKeyboard`), from the height it announces rather than the one `visualViewport` catches up to; with no shell it rises over `normal` once the viewport settles | Reverse | `translate` sits beside `transform`, so a sheet that enters and drags on transform needs no wrapper. A centred dialog rises by half. The safe inset drops after the lift (`html[data-keyboard]`), never before it, and returns at once when the keyboard goes. A sheet that is too tall for what is left is capped by `max-height` once, not eased |
| Toast | Rises from the dock with its opacity, `ANIMATION_DURATION.slow`, `EASE_OUT`, on a whole `translate()` string the compositor runs (`DockItem`, `useDockPresence`); the slot around it glides on a compositor `translate` from its old place, once per change of who is on screen (`useFlipPlace`), when a neighbour comes or leaves | Along its swipe, which follows the finger 1:1 (`useSwipeAway`) and springs back on `SPRING_SHEET` through `animateSpring` short of `releaseDismisses` on its own width; else down into the dock on a phone, off the inline end on a desk | Off a solid surface, because fading a blur pops it; under reduced motion only the fade. The undo window drains on `toast-drain`, a transform that keeps running under reduced motion because it is a timer |
| Tooltip | Fade at `TOOLTIP_FADE_MS`, after `TOOLTIP_DELAY_MS` of rest | Fade | Fine pointers only; focus opens it with no delay |
| Tab and choice fill | `SlidingFill`: one fill sized to the chosen item and scaled from the old item's box, on a WAAPI `transform` along `SPRING_SNAPPY`; geometry is cached by a `ResizeObserver`, so a choice reads no layout. `FillSeed` is its twin in the chosen item for the server's HTML and the first paint | none | Reduced motion places it without travel |
| Tab and nav indicator | The rail's `NavActiveIndicator` moves by `layoutId` on `SPRING_SNAPPY`, `layout="position"` so only the position travels. The phone bar's `NavBarTravelIndicator` springs one whole `transform` string on `SPRING_SNAPPY`, which motion runs as a Web Animation with a `linear()` easing, so it keeps moving while React is busy | The phone circle fades in where its tab is and cuts away with the page; it never sweeps in from the previous tab. Under reduced motion it cuts to the tab and keeps the fade | Tab offsets are measured when the bar appears or is resized (`useTabOffsets`), never in a press. `x` is never animated: an independent transform runs in JavaScript |
| Optimistic active tab | The indicator and the two tabs it concerns change in the frame the tab is pressed; `router.push` starts after that frame has painted (`afterNextPaint`) | | Driven from the pending pathname by `useActiveNavHref`, which selects the answer, so a press re-renders those tabs and not the bar. The bar's shape and presence follow the committed path |
| Tab content | Crossfade at `fast` | | Never a horizontal slide on desktop; on a phone a section swipe follows the finger |
| Section swipe between tab roots | Surface follows at `NAV_SWIPE.follow`, capped at `maxTravelFraction` | Commits at `commitMs`, abandons at `releaseMs` | The page is what travels, so the swipe's navigation asks for no ghost (`{ ghost: false }`) and warms its neighbours with `FULL_PREFETCH`. Commit needs `commitFraction`, or `flickFraction` plus `flickVelocity`, and horizontal must beat vertical by `dominance` |
| Page arrival with no gesture | A cut | | The tab indicator's travel and the destination's ghost are the motion. A fade from opacity 0 painted a blank page for a frame and read as delay. A tab root that `KeptRoots` holds is revealed in the frame it is pressed, with no ghost, at the offset it was left at |
| Swipe back | Follows the finger 1:1 from `SWIPE_BACK_EDGE_ZONE`, commits at `SWIPE_BACK_THRESHOLD`, over the kept page it uncovers, or the ghost of one that is not kept (`SwipeUnderlay`) | Settles at `ANIMATION_DURATION.fast` | The travel is written to the page and its edge directly, never through React state; the leading edge shadow rides the same clock; `will-change` exists only while the gesture is active |
| Route ghost | The destination's ghost stands in only after `NAV_GHOST_GRACE_MS`, stays `NAV_GHOST_MIN_MS` once it has, is gone in the commit that lands the page, and never outlasts `NAV_GHOST_MAX_MS` | | The swap itself is a cut, deliberately: the ghost is the destination's own shape, so a fade would animate between two drawings of the same thing |
| Nav bar hide on scroll | `.mobile-nav-bar` glides on transform at `--transition-duration-nav-glide`, `--ease-dialog`; returns on any upward scroll | | Also glides away while the shorts comment sheet is open, so nothing blurred sits behind the morph |
| Sticky header on scroll | Translates out on transform at `normal`; the bar never changes height | Reverse | |
| List insert | `(not yet built)` Row fades and rises 8px at `normal`, `--ease-dialog`; rows below glide on transform | | The feed, the thread and the notification list currently snap. It needs `AnimatePresence` around the row primitive, or a CSS reveal on first commit |
| List remove | `(not yet built)` Row fades at `fast`; rows below close the gap on transform | | A swiped row leaves under the finger first |
| Swipe row actions | Follows the finger past `SWIPE_ACTION.activate`, yields toward `maxTravel`, arms at `threshold`; the glyph ramps through `SWIPE_GLYPH` | Settles on `SPRING_SNAPPY`, or exits the row's own width when committed | One arming haptic, fired while the finger is still down |
| Swipe to reply on a bubble | Right only, 1:1, clamped to `SWIPE_DISMISS_THRESHOLD` plus `SWIPE_DISMISS_OVERSHOOT` | Settles at `ANIMATION_DURATION.normal` on `--ease-exit` | The reply glyph reads the same `SWIPE_GLYPH` ramp as the row |
| Skeleton to content | `state-held` holds the layout unpainted for `SKELETON_DELAY_MS`, then bones | Content crossfades in at `fast` through `state-fade-in` | Opacity only, and the bones are the content's shape, so nothing moves |
| Cascade of results | `animate-cascade-up`, one `ANIMATION_STAGGER` step per item, `both` fill, capped | | With no `--reveal-delay` it is identical to a plain reveal |
| Server-rendered panel | `animate-reveal` at `normal` | | Never a Framer `initial` |
| Context card | `animate-context-appear`: scale 0.98 and rise 8px at `normal`, `--ease-dialog`, `forwards` | | The explore surface's arrival |
| Image | Blurhash paints first; `.img-fade` raises the image at `--transition-duration-slower` on `--ease-enter` | | Opacity only; a photo never zooms, and one the client mounts never pops: a cached one re-fires the fade from `complete`. A picture in the server's HTML is not held for hydration `[media §1]` |
| Video controls | Fade at `normal` from `initial={false}`; auto-hide by fade | | Scrub follows the finger; progress is one continuous shape |
| Progress and level bars | `transform: scaleX` on an `origin-left rtl:origin-right` fill | | `STORY_TRACK_FILL` and `audio-controls.tsx` are the reference; a waveform reveals its lit copy with `clip-path` |
| Shorts paging | Transform at `SHORTS_TRACK_TRANSITION` (`TIMING.slow`, `EASE_SMOOTH_OUT`), written imperatively with `important` | | Reduced motion shortens to `SHORTS_TRACK_TRANSITION_REDUCED`. An interrupted snap resumes from the visible pixel, read off the live matrix |
| Shorts comment sheet | The playing short's media morphs on transform to `SHORTS_COMMENTS_VIDEO_RATIO`; its chrome is gone by a third of the travel; the tab bar glides away | Reverse, driven by the same progress value | One full-height sheet translated between the peek and full detents: the grabber and header follow the finger 1:1 (`followSheet`), only the top edge rubber-bands, and a release lands on `releaseCommentsSheet`'s detent on `SPRING_SHEET` carrying the velocity. The sheet's drag carries the video with it through `CommentsMorph`: the finger writes every layer frame by frame, and a release gives the sheet, its composer, the short (a `transform` and a `clip-path` round, never a `border-radius`), its chrome, the feed's tabs and the backdrop keyframes on one spring, so they part at no frame and none waits on script. Never a relayout, and the sheet surface is solid |
| @ask turn arriving | The question and its answer fade and rise 8px at `normal`, `--ease-dialog`; the trailing note below glides on `layout="position"` | | The composer is pinned under the list, so the conversation scrolls beneath it rather than pushing it down |
| Lightbox | Fades in from the page; the chunk's ghost has already painted the scrim, so it enters at rest | A pull down from the picture moves it 1:1 and fades the scrim under it (`useViewerGestures`), locked to the vertical after a few pixels and ignored while zoomed or pinching; a release past its distance, or a flick, closes it, else the picture springs home on the compositor | A sideways swipe steps. Distance or velocity, whichever comes first |
| Story viewer | Fades in from the page at `normal`, the stage's blurhash in its first frame. On a cold chunk `StoryViewerFallback` makes that same fade and the viewer takes over at rest through `useTakesOverFallback` | Fades out at `normal` | Never a solid stand-in cut in before the fade: black, then the page, then the viewer is three frames of one arrival. A highlight opens the same way |
| Story progress | `transform: scaleX`, ticked at `STORY_TIMER_TICK_MS` across `STORY_DURATION_MS`; only the active bar transitions | | A highlight is the same object and reads the same bar |
| Story advance | The incoming image raises through `.img-fade`; the leading `STORY_TAP_PREV_FRACTION` of the stage goes back | | `(not yet built)` for video: a video-to-video advance is a hard cut today |
| Story cube | The rotor turns 1:1 with the finger, releases into `SPRING_SNAPPY` at `COMMIT_FRACTION` or `COMMIT_VELOCITY`; only the neighbouring face mounts | | The face swaps inside a synchronous flush so the rotor never resets a frame late |
| Story pull to dismiss | Opacity falls across `STORY_DISMISS_FADE_PX` to `STORY_DISMISS_FADE_OPACITY` | Springs back on `SPRING_SNAPPY` | Follows the finger, rubber-banding sideways when there is no neighbour |
| Sticker and text widget | Placement is a MotionValue; the drag is 1:1 with no inertia and no spring | | Deliberately unphysical: a sticker lands where the finger left it, snapping to centre, then edges, then siblings |
| Picture-in-picture | The dock settles on `SPRING_SHEET` from opacity 0, rise 12px, scale 0.96, `initial={false}` | Reverse, then the node goes invisible only once the exit lands | One player, mounted across every route, still playing throughout. `FloatingWindow` follows 1:1, releases into the projected corner on `SPRING_SHEET` and folds the offset into the corner's anchor in the same task, so nothing jumps |
| Re-rank dip | The feed recedes to `scale(0.92) translateY(28px)` at 0.3 opacity on `ANIMATION_DURATION.slower`, the new order swaps in behind it, and it resurfaces on `SPRING_GENTLE`, both as `transform` strings so the dip survives a busy main thread | | Every inline style is cleared on resurface: a lingering transform would make the feed the containing block of every fixed child |
| Like, repost, share | `GlyphResponse` remounts the glyph on activation: like from scale 0.6 on `SPRING_SNAPPY`, repost from rotate −90 on `SPRING_SNAPPY`, share from a 2px lift | | Fires only on the affirmative direction, in the frame of the tap |
| A count that changes | `(not yet built)` Crossfade at `fast` with tabular figures so the width holds | | Counts snap today; nothing crossfades a digit |
| Double-tap like | `HeartBurst`: scale 0 to 1.2 to 1 and fade over `ANIMATION_DURATION.extraLong` on `EASE_OUT`, as a CSS keyframe animation | | The one burst, in the feed and on a short; reduced motion keeps an opacity pulse |
| Follow, join, subscribe | The label crossfades; the control's width is fixed to the wider label so nothing shifts | | |
| Send a message or comment | `(not yet built)` The bubble arrives with the list-insert motion in the frame of the tap | | Delivery state changes by icon crossfade, never a spinner |
| Message highlight on jump | Ring holds for `MESSAGE_HIGHLIGHT_MS`, then fades | | A jump says which row it landed on, then gets out of the way |
| Reaction chips | Scale 0.92 and fade in at `normal` on `--ease-gentle`, under `AnimatePresence initial={false}` | Reverse at 0.8 on `--ease-exit` | The bubble's own menu grows from the bubble's corner on `SPRING_SNAPPY` |
| Typing indicator | `typing-bounce` at `--duration-typing-bounce`, `--ease-gentle`, looping | Fades out | The one looping status in messaging; it stops itself at `TYPING_INDICATOR_TIMEOUT_MS` |
| Incoming call | Scrim fades at `normal`, the card follows from scale 0.92 after `ANIMATION_DURATION.instant` | | A solid scrim, so nothing blurred fades |
| Composer growing | Height follows content with no transition | | A transition here reads as lag |
| Composer and toolbar swap | `popLayout` under `AnimatePresence initial={false}`, rise from 20px at `fast` | Reverse | The outgoing bar leaves the flow so the incoming one does not wait for it |
| Pull to refresh | A solid wash over the feed follows the finger 1:1 on opacity alone, converges to a committed cover on `EASE_ARRIVE`, then lifts off in the bloom; the spinner runs at `--duration-refresh-spin` | | No blur and no mask: opacity is composited. The converge holds at least `FEED_REFRESH_MIN_MS` so the arrival reads before the bloom |
| Notification peek | `POP_ANCHORED` from the bell's corner at `fast` | Reverse at 0.8 | Dwells `NOTIFICATION_PEEK_DURATION_MS`; hover holds it |
| Empty state | `animate-reveal` after the data settles | | Never during the hold |
| Unread dot, badge | Fades in at `fast` | Fades out | Never pulses |
| Live badge | Static fill, no dot, no glow | | Live is a state, not an event |
| Live stage | The route's own hold draws the identical split, so the handoff moves nothing; stage and picture-in-picture swap on a `layout` FLIP armed for `TIMING.slow` | | The FLIP is armed only by a presentation change, never by incidental relayout |
| Selection fill on a chip or cell | The fill travels as `SlidingFill` at `SPRING_SNAPPY` | | One fill means selected; never a ring and a fill |
| Hover, desktop | Surface tint and `--glow-hover` bloom at `fast`, under `(hover: hover)` only | | Never on touch; a stuck hover reads as broken |
| Press | Lands in the same frame (`transition-duration: 0s` on `:active`), releases at `--transition-duration-fast` on `--ease-smooth` | | Three rungs: `--press-subtle` 0.98, `--press` 0.96, `--press-firm` 0.93, each excluding a pressed descendant control |
| Focus | The `:focus-visible` outline is drawn, not animated | | |
| Disclosure | `Collapse` eases `grid-template-rows` at `normal` on `--ease-smooth` | Reverse | Zero border while collapsed, or a hairline shows through the closed box |
| A post hidden in place | `useTombstone`: the card's `Collapse` closes as the strip's opens, in one frame, so the rows below glide rather than jump | Undo plays it backwards | A row never hidden stays `idle`, the same boxes with no clip, so hiding it later remounts nothing |
| Auth screens | `auth-card-enter` rises the solid card at `normal`. The frosted entry card does not move, since a moving blur re-rasterises every frame; the posts behind it move as atmosphere, `entry-feed-rise` at a constant `linear` speed like the sound disc, so no column stalls at its loop's seam | | Switched off under reduced motion by name |
| Error page | The card reveals once through `EmptyState`'s `animate-reveal` under a header that holds still; `global-error.tsx`, with nothing of the app left, uses `error-page-enter` at `slow` | | Arrives like a panel; never shakes, never flashes red |
| Onboarding | The chrome holds still; the step travels `STEP_TRAVEL_PX` under `mode="wait"` and `initial={false}`, so the first step paints at rest, entering on `EASE_ENTER` and leaving at 0.8 on `EASE_LEAVE`; the progress bar is `scaleX` with `initial={false}` | | Only the content moves in a multi-step flow |
| Ceremony, welcome | Staged reveals on `EASE_ARRIVE`, leaving on `EASE_LEAVE`, each stage baking its final frame into inline style so the resting state is exact | Aborts on unmount | Under reduced motion every stage is set to its final frame and the ceremony is skipped, not slowed |
| Capture and story pop-ins | `EASE_BACK`, one element at a time; the focus reticle is the reference | | Never a group |
| Shutter, countdown, capture wash | `CAMERA_MOTION.pressMs` press, `photoWashMs` wash, `lensSwitchMs` freeze; the countdown is `CountdownTick` wherever something counts in (the camera's timer, a voiceover take), its tick scaling up on the way out under `popLayout` | | The shutter animates transform and border-radius only, and never changes identity; the take ring's fill is a `MotionValue` the arc reads, so a recording costs no render per frame |
| Music ticker | `MarqueeText` carries a label wider than its window through one pass of `animate-marquee` on `--ease-smooth`, settling on the frame it started from | | One pass, not a loop `[motion §2]`; paused rather than removed off-screen, and static under reduced motion |
| Sound disc, shorts | `sound-disc-spin` at `--sound-disc-spin-ms`, linear, infinite, paused rather than removed | | Deliberately off `.animate-spin`, which is pinned to 1s for loaders |
| Skeleton shimmer | `skeleton-shimmer` sweeps at `--duration-shimmer` across `--shimmer-band` | | An absolute band width, so a small bone and a large one shimmer alike |
| Send morph | The composer's thumbnail or text becomes the bubble in the thread: one `layoutId`, `SPRING_SHEET`, in the frame of the send; rows above glide up as it lands | Undo or failure carries it back into the composer | `[signature-motion §7]` (not yet built) |
| Card to page, page to card | The tapped card is the shared element; the page grows from its frame and Back shrinks it home to the same scroll offset | Reverse on the same spring | `[signature-motion §27]` (not yet built) |
| Thumbnail to viewer | The photo grows from its card frame into the viewer; a drag down shrinks it back and dismisses when past threshold | Reverse, driven by the drag | `[signature-motion §7]`, `[motion §34]` (not yet built) |
| Avatar to profile | The row's avatar becomes the profile header's avatar | Reverse | (not yet built) |
| Story ring to story | The ring expands into the story stage; between people the stage rotates as one surface | Collapse to the ring | (not yet built) |
| Short to comments | The short morphs to its docked size in the same frame the sheet rises, one spring, one drag; replaces the two-motion version above | Reverse | `[signature-motion §9]` (not yet built) |
| Reorder by long press | Lift on `press-firm` with `--shadow-lg`; siblings part on transform; settle into the gap on `SPRING_SHEET` | | `[signature-motion §19]` (not yet built) |
| Numeric roll | The changed digit rolls out and in along one axis at `normal` on `--ease-dialog`; other digits hold in tabular figures | | `[signature-motion §20]` (not yet built) |
| Glyph state | Heart fill beat, bell ring, bookmark drop, mic breath: one effect, once, on the cause | | `[signature-motion §21]` (not yet built except the heart) |
| Large title | Folds into the bar on transform with scroll; unfolds on the way back; the bar's height holds | | `[signature-motion §22]` (not yet built) |
| Chrome response | Frost and hairline strengthen as content passes under the header and tab bar | | `[signature-motion §23]` (not yet built) |
| AI text rewrite | A shimmer band sweeps the block, then the new text crossfades in at `fast` | | `[signature-motion §24]` (not yet built) |
| Push and pop (phone) | A push into a detail page slides the page in from the inline end at `slow` on `--ease-dialog`, one `transform` on the compositor; the kept root beneath recedes `PUSH_PARALLAX_PERCENT` and dims to `PUSH_DIM_BRIGHTNESS` | Swipe-back, above. Back by button or shell is a reveal, not a slide | `[signature-motion §26]`. A tab switch, a traversal, reduced motion and a wait the ghost stood in for are all cuts |
| Zoom (row to page) | View Transitions API: the row and the page header share a name; the page grows from the row | Reverse | `[signature-motion §27]` (not yet built) |
| Live activity capsule | Inflates from the compact pill to the expanded card on `SPRING_SHEET`; collapses on completion with a `success` haptic | Deflates into the pill | `[signature-motion §30]` |
| AI listening edge | A perimeter ribbon appears at `fast`, drifts at `--duration-gradient-drift`, follows the input level per frame, collapses into the response | Collapses the frame listening ends | `[signature-motion §39]` |

## Enforced by

| Test or rule | Rules |
|---|---|
| `__tests__/utils/compositor-spring.test.ts`, `__tests__/ui/overlay-motion.test.ts` | 11, 31, 38a: a release is a `transform` animation on a `linear()` spring that takes its start from the live offset |
| `__tests__/hooks/use-keyboard-lift.test.tsx`, `__tests__/viewport/visible-viewport.test.ts` | 30: the keyboard lifts on `translate` over the shell's frame, and the safe inset drops after it |
| `__tests__/hooks/use-shorts-comments-morph.test.ts` | 38a: the shorts morph is keyframes on one timing |
| `__tests__/fitness/a-release-runs-on-the-compositor.test.ts` | 38a: the surfaces that own a release import no MotionValue spring and pass no `onUpdate` |
| `eslint-rules/no-literal-motion.mjs`, wired at `eslint.config.mjs` | 5, 8, 9, 13; its burndown list names 20 files still exempt |
| `__tests__/config/design-token-mirror-parity.test.ts` | 5, 8, 9, 25, and the whole curve table |
| `__tests__/fitness/framer-ships-once.test.ts` | 19, 44 |
| `__tests__/fitness/a-press-reads-one-ladder.test.ts` | 52, 56, and the ban on `whileTap` under 5 |
| `__tests__/fitness/an-overlay-inherits-its-behaviour.test.ts` | 24, 26 |
| `__tests__/fitness/arbitrary-value-duplicates-a-token.test.ts` | 5 |
| `__tests__/fitness/every-css-var-resolves.test.ts` | 25, 30 |
| `__tests__/fitness/a-gesture-writes-a-motion-value.test.ts` | 32 |
| `__tests__/utils/axis-lock.test.ts`, `__tests__/components/ui/range-input.test.tsx`, `__tests__/components/ui/step-slider.test.tsx` | 37, for the axis a finger commits to, the scroll that writes nothing, the tap, and the cancelled drag put back |
| `__tests__/fitness/a-loop-is-a-status.test.ts` | 2, 49, 50 |
| `__tests__/fitness/a-reveal-is-a-class-on-the-server.test.ts` | 21 |
| `__tests__/fitness/a-stagger-steps-once-and-stops.test.ts` | 5, 20 |
| `__tests__/fitness/a-surface-that-moves-is-solid.test.ts` | 41 |
| `__tests__/fitness/a-transition-names-a-curve.test.ts` | 10 |
| `__tests__/fitness/a-transition-names-a-property-not-a-layout.test.ts` | 39, 40 |
| `__tests__/fitness/an-easing-is-a-token.test.ts` | 8, 13 |
| `__tests__/fitness/an-exit-is-shorter-than-its-enter.test.ts` | 7 |
| `__tests__/fitness/one-swipe-has-one-threshold-set.test.ts` | 34, 35 |
| `__tests__/fitness/will-change-outlives-nothing.test.ts` | 43 |
| `__tests__/components/ui/dropdown-menu.test.tsx` | 25, 27 |
| `__tests__/components/shared/context-menu.test.tsx` | 27, 29 |

**Not yet enforced.** Each needs the test named, in the house style, added with the rule it covers.

- `a-travel-is-a-token` (17): every `y`, `x` and `translate` offset in an enter or exit variant reads a named
  travel constant. Blocked: there is no travel token to read, and rule 17 marks the one it asks for as not yet
  built, so the spec would have nothing to compare a rise against.

## Traps

- **7** A transition reads its duration from the style it moves *to*, so an exit-only duration has to be
  declared on the closed state, after the mode rules whose `transition` shorthand would reset it.
- **17** One undefined custom property invalidates a whole `transition` shorthand, so a curve variable that was
  never declared silently removes the animation it was meant to shape.
- **21** A `'use client'` component still server-renders, so a Framer `initial` on it writes `opacity: 0` into
  the HTML and the panel is invisible until hydration.
- **24** A JS-driven open drops its first frames because React has just committed the portal on the same
  thread. A transition needs a style recalculation, not a paint, to capture its start value, so
  `flushPendingStyle` (`document.getAnimations()`, which flushes style in every engine and reads no geometry)
  before flipping the state is what makes the enter cost nothing. A `getBoundingClientRect` there is a layout
  of the whole document, mounted feed rows included.
- **27** A scrim that is invisible but still hit-testing swallows the tap that follows a close.
- **28** Clearing placement state on close makes the exit jump to the centre of the screen, because the node is
  still mounted for its exit.
- **29** A never-opened instance that arms an exit timer costs a timer per row, and one feed page mounts
  hundreds of them.
- **31** A dismissed sheet parks an inline `translateY(100%)` that outranks every class rule, and reopening
  within the unmount window reuses the same node, so without an explicit clear the sheet returns invisible.
- **33** A real pointerup repeats the last move's position and timestamp, which is a zero-length interval and
  reads every flick as motionless. Fall back to the sample before it.
- **38** A host that vetoes a close leaves the panel parked off-screen but still open, holding the scroll lock
  and the focus trap over a page nothing can reach.
- **39** Tailwind's bare `transition` utility includes `filter` and `backdrop-filter`, so writing it beside a
  blurred surface animates the blur without ever naming it.
- **41** A backdrop blur behind a morphing sheet smears the morph and drops frames.
- **42** An inline `backdropFilter` cannot be shed by the low-power path, which keys off the class.
- **43** A later `transition-*` utility replaces `transition-property` wholesale and drops `transform`, which
  snaps the press on every control that pairs the two. The press ladder is declared unlayered for that reason.
- **47** The blanket reduce-motion `transition-duration: 0.01ms` swallowed the overlay panel, so every sheet
  teleported. The two kill switches sit at different specificity, so the opt-back is declared twice.
- **48** `reducedMotion: 'always'` snaps transforms, so a variant whose only animated property is `y` blinks in
  and out with nothing left to animate.
- **50** Zeroing an animation's duration without also zeroing its delay leaves a `both`-filled staggered panel
  invisible for as long as its stagger.
- **52** `-webkit-tap-highlight-color: transparent` plus WebKit gating `:active` on a touch listener means zero
  press feedback on iOS while every `press` class looks complete on desktop.
- **53** Framer captures `reducedMotion` when it constructs a motion component, so a value that only arrives
  after a settings hook hydrates never reaches the app shell.
- **58** A loader class pinned to a fixed duration for spinners will also whip a decorative disc around if the
  disc reuses it.
