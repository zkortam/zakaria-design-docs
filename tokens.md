# Tokens

Every number the product renders comes from a named token, and every token has one declaration. This
document is the register: what each scale is called, what it resolves to, which file declares it and
which mirror JavaScript may read. Nobody sees a token, but everybody feels one. A size picked by hand
is the size that does not line up with the size beside it; a duration typed into a class is the thing
that moves at a different speed from the thing it moves with; a corner written as a literal is the
corner that stops being concentric the moment a theme scales the radius around it.

## Owns and defers

Owns: the scales and their verified values, which file declares each, the mirror-parity contract, the
no-arbitrary-value rule, the recipe rule, and the places where two scales carry one idea.

Defers: which token a surface picks to `[craft]`; which duration does which job, and the grammar of
motion, to `[motion]`; what a theme makes of a colour token, and the effects switches, to
`[colour-and-effects]`; the typographic rules on the ramp to `[type]`; the column, header and shell
geometry that reads these tokens to `[layout]`; hit area and contrast minima to `[accessibility]`.

## Rules

1. **One declaration, one value.** `app/globals.css` is the source for every scale a stylesheet can
   express. `lib/constants/design-tokens.ts`, `breakpoints.ts`, `timings.ts` and `lib/styles/motion.ts`
   are checked mirrors JavaScript may read. Nothing else defines a value.
2. **No arbitrary value that a token already names.** No `text-[13px]`, no `rounded-[14px]`, no
   `duration-[180ms]`, no `z-[45]`, no `h-[62svh]`, no hex or `rgb()` outside the theme blocks and the
   colour and gradient constant files.
3. **A token is read, not retyped.** A class writes `duration-fast`, not `150ms`; a style object
   writes `var(--blur-sheet)`, not `blur(14px)`. A value that appears in two files is one token that
   was copied.
4. **Every CSS variable resolves where it is read.** A property declared at `:root` cannot read a
   variable declared on a panel; it computes to nothing and every consumer silently loses it. Declare
   and read at the same level.
5. **Blur, elevation, radius, duration and easing are class-level.** An inline `backdropFilter`,
   `boxShadow` or `transition` string cannot be shed by `perf-lite`, cannot follow reduced motion and
   cannot be retuned in one place.
6. **A class string that appears twice is a recipe**, and a recipe lives in `lib/styles/`, named by
   domain: `glass.ts`, `page-header.ts`, `feed-post.ts`, `motion.ts`, `camera.ts`, `admin.ts`,
   `content-info.ts`, `story-sticker.ts`, `eye-scroll.ts`, `liveness.ts`. A recipe module for one
   screen is a private design system and is not one of these.
7. **The mirror is a subset, never a variant.** A key that exists on both sides carries the same
   number in both. A JavaScript-only rung is allowed and is marked as such; a JavaScript rung that
   contradicts its CSS twin is a defect.
8. **A count badge never sets its own size.** `text-badge` is the one exception to the 14px floor, and
   `cn()` drops a `text-*` size it was never taught, so a one-off size on a badge resolves to nothing.
9. **Two scales that mean one thing are debt, not a choice.** The asymmetries below are recorded so a
   reader picks the right side today and a future rename retires the wrong one.

## Type

Declared in `@theme` in `app/globals.css`. Every step multiplies by `--font-scale`, so the viewer's
Text Size moves the whole ramp at once.

| Token | Size | Line height | Tracking | Job |
|---|---|---|---|---|
| `--text-badge` | 11 | 1 | | Count badges, the one exception to the 14px floor |
| `--text-sm` | 14 | 20 | | Secondary and meta, the reading floor |
| `--text-body` | 15 | 22 | -0.005em | Reading |
| `--text-base` | 16 | 24 | | Inputs; iOS zooms a focused field below 16 |
| `--text-lg` | 18 | 28 | -0.01em | Titles |
| `--text-xl` | 20 | 28 | -0.01em | Titles |
| `--text-2xl` | 24 | 32 | -0.02em | Page titles |
| `--text-3xl` | 30 | 36 | -0.02em | Page titles |
| `--text-4xl` | 36 | 40 | -0.02em | Display |
| `--text-5xl` | 48 | 1 | -0.02em | Display |
| `--text-7xl` | 72 | 1 | -0.02em | Display |

Weights: `--font-weight-normal` 400, `medium` 500, `semibold` 600, `bold` 700.

Text Size: `--font-scale` is 0.875, 1 or 1.125, set from `html[data-font-size]`, paired with a root
`font-size` of `--root-font-small` 14px, `--root-font-default` 16px, `--root-font-large` 18px. Both
axes move together, because the app measures in `rem` for chrome and in scaled `px` for the ramp.

Families: `--font-sans` and `--font-mono` in `@theme`; a theme may replace the UI face through
`--theme-font-sans`, which only Aero does today.

## Spacing

`--spacing-*` in `@theme`, on a 4px base. The step name is the multiplier; `_5` is a half step.

| Token | px | Token | px | Token | px |
|---|---|---|---|---|---|
| `--spacing-0` | 0 | `--spacing-4` | 16 | `--spacing-12` | 48 |
| `--spacing-0_5` | 2 | `--spacing-5` | 20 | `--spacing-14` | 56 |
| `--spacing-1` | 4 | `--spacing-5_5` | 22 | `--spacing-15` | 60 |
| `--spacing-1_5` | 6 | `--spacing-6` | 24 | `--spacing-16` | 64 |
| `--spacing-2` | 8 | `--spacing-7` | 28 | `--spacing-20` | 80 |
| `--spacing-2_5` | 10 | `--spacing-8` | 32 | `--spacing-22` | 88 |
| `--spacing-3` | 12 | `--spacing-9` | 36 | `--spacing-24` | 96 |
| `--spacing-3_5` | 14 | `--spacing-10` | 40 | `--spacing-28` | 112 |
| | | `--spacing-11` | 44 | `--spacing-32` | 128 |

The scale continues `--spacing-36` 144, `--spacing-40` 160, `--spacing-44` 176, `--spacing-48` 192,
`--spacing-52` 208, `--spacing-56` 224, `--spacing-60` 240, `--spacing-64` 256, `--spacing-72` 288 and
`--spacing-78` 312. Every step equals its name times four, and the mirror spec asserts it.
One step is named rather than numbered: `--spacing-overlay`, 16 below `sm` and 20 from it, the overlay
inset `[components §14a]`.

## Radius

`--radius-base-*` is declared once in `:root`; the `[data-theme]` block multiplies each rung by
`--theme-radius-scale` into `--radius-*`, which is what a utility resolves. `RADII` in
`lib/constants/design-tokens.ts` is the numeric mirror for inline styles.

| Token | px | Typical user |
|---|---|---|
| `xs` | 8 | Small chips, swatches |
| `sm` | 12 | Controls |
| `md` | 16 | Controls, popup surfaces |
| `lg` | 20 | Tiles |
| `xl` | 24 | Cards |
| `2xl` | 28 | Cards, sheets |
| `3xl` | 32 | Overlay panels, feed cards |
| `full` | 9999 | Pills, circles |

`nestedRadius(outer, padding)` returns the inner rung so corners stay concentric, floored at 0.

## Elevation, ink and colour tokens

Shadows resolve per theme: `--shadow-xs`, `sm`, `md`, `lg`, `xl` read the theme's `--sh-*`, and
`--shadow-glass`, `--shadow-glass-heavy` read `--sh-glass`, `--sh-glass-heavy`. `--drop-shadow-text`
reads `--sh-text`, which every theme maps to the fixed `--shadow-on-media`. Ink is four tiers,
`--text-primary`, `--text-secondary`, `--text-muted`, `--text-faint`, plus `--text-inverted`; a fifth
is a bug. Every theme declares the full contract in `lib/theme/contract.ts`, and their values are
`[colour-and-effects]`.

## Blur

`--blur-*` in `@theme`. Backdrop cost scales with the square of the radius and frost composites under
every overlay on every frame, so these are tuned down on purpose and none rises without a measurement.

| Token | px | Where |
|---|---|---|
| `--blur-overlay` | 4 | The lightest wash behind a floating layer |
| `--blur-media` | 8 | A pill, badge or scrim sitting on video or a photo |
| `--blur-header` | 12 | `GLASS_HEADER`, glass media controls |
| `--blur-chrome` | 14 | `.glass-chrome`: the translucent rail and right panel |
| `--blur-sheet` | 14 | `GLASS_OVERLAY`, `GLASS_SHEET`, `GLASS_DROPDOWN` |
| `--blur-glass` | 16 | `GLASS_CARD`, the blurred standalone card |
| `--blur-strong` | 24 | Full-screen gates and the sensitive-media guard, above the tuned range and under review |
| `--blur-hero` | 80 | The auth hero wash and the publish sheet, one decorative surface |

## Z-index

`Z_INDEX` in `lib/constants/design-tokens.ts` is the whole ladder. Use `above(rung)` or `below(rung)`
for a companion surface, never `+ 1`.

| Rung | Value | CSS twin | Job |
|---|---|---|---|
| `base` | 0 | yes | The floor everything sits over |
| `editorOverlay` | 45 | | The band of the editor's hit targets over the stage the compositor paints |
| `inPageChrome` | 50 | | A page's own sticky bar, a new-posts pill, a menu's dismiss catcher |
| `topEdgeCap` | 60 | | The 1px cap masking the backdrop-filter seam under a frosted header |
| `sticky` | 200 | yes | The app frame: rail, mobile nav, `PageHeader` |
| `risingSurface` | 250 | | A page surface lifting over the app frame for its own transition |
| `overlay` | 300 | yes | An overlay scrim |
| `dropdown` | 350 | yes | In-flow dropdown surfaces |
| `livePip` | 360 | | The floating live picture-in-picture window, below modals |
| `modal` | 400 | yes | A dialog panel |
| `liveStage` | 450 | | The full-screen live takeover, below toasts |
| `floating` | 500 | yes | Anchored menus, popovers and sheets opening over a takeover |
| `viewer` | 520 | | A full-screen media takeover, which opens from a sheet |
| `tooltip` | 550 | yes | Transient, pointer-following, never interactive |
| `toast` | 600 | yes | The app's own voice; nothing the app renders covers it |
| `gate` | 700 | yes | The parental lock and the wellbeing limit |
| `max` | 9999 | | Reserved; a surface that needs it needs a rung instead |
| `privacyOverlay` | 99999 | | The app-switcher blur, the only thing above the gate |

The CSS mirror `--z-index-*` carries nine rungs: `base`, `sticky`, `overlay`, `dropdown`, `modal`,
`floating`, `tooltip`, `toast`, `gate`. Everything else is JavaScript-only, for a surface that sets
`zIndex` inline. A fixed element never writes a raw `z-<n>`.

## Sizes

| Scale | Where | Values |
|---|---|---|
| `ICON_SIZES` | `design-tokens.ts` | `xxs` 10, `tiny` 12, `xs` 14, `sm` 16, `md` 18, `lg` 20, `rail` 22, `xl` 24, `chrome` 26 (header, tab bar, rail), `xxl` 28, `2xl` 32, `3xl` 48 |
| `BADGE_SIZES` | `design-tokens.ts` | `xs` 14, `sm` 18, `md` 22, `lg` 26 |
| `DIALOG_SIZES` | `design-tokens.ts` | `xs` 360, `sm` 460, `md` 560, `default` 620, `lg` 720, `xl` 920 |
| `TOUCH_TARGET` | `design-tokens.ts` | `size` 44, `offset` -16, for timeline resize handles |
| `--tap-target-min` | `:root` | 44px, the coarse-pointer minimum, read by the `tap-target` utility and by `input[type=range]` |
| `POPOVER` | `lib/constants/layout.ts` | `edgeMargin` 8, `gap` 4, `minDropHeight` 200 |
| `STORY_RING` | `lib/constants/layout.ts` | `outer` 104, `circle` 92, `avatar` 73, `strokePx` 3 |

Inside an `IconButton` the size prop on the glyph does nothing: the button sets `[&>svg]:h-*` and CSS
beats an SVG size attribute. Size through the button.

## Sheet, dialog and chrome geometry

Declared in the plain `:root` in `app/globals.css`, because `@theme` cannot carry an `env()` call.

| Token | Value | Job |
|---|---|---|
| `--sheet-h-xs` | 30svh | A two-row sheet, and the space a `rows={0}` fallback holds |
| `--sheet-h-sm` | 45svh | A short form; the phone sheet's resting minimum |
| `--sheet-h-md` | 60svh | A list a person scrolls a little |
| `--sheet-h-lg` | 75svh | A menu sheet's ceiling, and a long form |
| `--sheet-h-full` | `min(92svh, 100svh − --safe-top − --sheet-top-gap)` | A takeover sheet's ceiling, short of 100 so it still reads as a sheet, and never under the status bar |
| `--sheet-top-gap` | 0.5rem | What a sheet at its ceiling keeps below the status bar |
| `--sheet-h-lifted` | `--app-vh − --safe-top − --sheet-top-gap` | The ceiling of a sheet the keyboard lifts |
| `--dialog-gutter` | 1.5rem | The room a centred dialog keeps at the viewport edge |
| `--dialog-h-cap` | `min(--sheet-h-full, --dialog-max-w, 100vw - 2 × gutter)` | Height never exceeds width; redeclared on the panel beside `--dialog-max-w` |
| `--page-header-pad` | 0.25rem, 0.75rem in the regular size class | The pill's breathing room; beside the rail there is no notch to spend it on |
| `--page-header-row` | 3.5rem | The one header height |
| `--page-header-h` | `max(pad, --safe-top) + row + pad` | What anything pinned beneath the header offsets by |
| `--mobile-nav-overlay-h` | 4.25rem | The nav in its bar form |
| `--mobile-nav-pill-h` | 4.5rem | The nav in its pill form |
| `--top-strip-h` | 0px | The phone voice strip's live height, set on `<html>` while it is up |
| `--floating-video-inset` | 0.5rem | The floating window's frame |
| `--floating-media-max-h` | 34dvh | The room under the right panel a clip may take |
| `--app-vh` | published by `lib/viewport/visible-viewport.ts` | The visible viewport with the keyboard subtracted; falls back to `100svh` |

A rung is a ceiling, not a floor: a sheet hugs its content and stops at its rung. Ninety-five
hand-written `svh` literals across twenty-four distinct values preceded these five, which is one
decision nobody was making rather than ninety-five decisions. A route whose pinned chrome a server
render cannot measure declares its own geometry the same way, as `--explore-pill-h`,
`--explore-chip-row-h` and `--explore-header-h` do.

Safe areas: `--safe-top`, `--safe-right`, `--safe-bottom`, `--safe-left` indirect over `env()`, and
every consumer reads the variable, never `env()` directly, so a host without a cutout can declare the
insets a device would report. The utilities are `pt-safe`, `pb-safe`, `ps-safe`, `pe-safe` for the
inset alone; `pt-safe-<n>`, `pb-safe-<n>` for the inset plus a step; `pt-safe-min-<n>`,
`pb-safe-min-<n>`, `ps-safe-min-<n>`, `pe-safe-min-<n>` for at least a step and never less than the
inset; and `top-safe`, `bottom-safe`, `start-safe-min-<n>`, `end-safe-min-<n>` for insets.
`--safe-inline-start` and `--safe-inline-end` are the side insets by writing direction `[layout §16a]`.
`--inset-bottom` is the physical bottom inset and `--safe-bottom` reads it, except under
`html[data-keyboard='open']`, where the keyboard pays it and `--safe-bottom` is `0px`. `--gesture-left`,
`--gesture-right` and `--gesture-bottom` are the system's gesture edges, `0px` until a shell injects them. An element
whose background runs under an inset carries `data-edge-bleed` `[layout §16d]`.

## Breakpoints

`BREAKPOINTS` mirrors `--breakpoint-*`: `sm` 640, `md` 744 (the narrowest full-screen tablet, which no phone
reaches in portrait), `lg` 1024, `xl` 1280, `2xl` 1440, `3xl` 1680, `4xl` 1920, `5xl` 2560. `MEDIA_QUERIES` names
every query the app is allowed to ask:

`xSmall` under 640, `phone` under 744, `compact` under 1024, `aboveXSmall` 640 and up, `tablet` 744
and up, `desktop` 1024 and up, `wide` 1280 and up, `touch` `(pointer: coarse)`, `fine`
`(pointer: fine)`, `hoverCapable` `(hover: hover)`, `phoneOrTouch`, `reducedData`, `reducedMotion`.
`fine` and `hoverCapable` are separate axes: a stylus is fine and cannot hover, so anything that only
unwinds on `pointerleave` asks `hoverCapable`. Which design a window gets is not a width: it is the size class,
`SIZE_CLASS_RUNGS` read by `sizeClassOf` and published as `data-size-class` `[layout §1]`; `data-short-window`
marks a phone on its side, where `--page-header-row` is 2.75rem, and `short:` is its variant.

## Durations

Two tables exist. The first is the shared speed ladder; the second is Framer's, in seconds. Which
duration does which job is `[motion]`.

| Name | TIMING (ms) | CSS | Notes |
|---|---|---|---|
| `micro` | 50 | none | `TIMING` only; there is no CSS twin |
| `instant` | 75 | `--transition-duration-instant` | |
| `fast` | 150 | `--transition-duration-fast` | Also the reduced-motion overlay fade |
| `normal` | 200 | `--transition-duration-normal` | |
| `slow` | 300 | `--transition-duration-slow` | |
| `slower` | 400 | `--transition-duration-slower` | |
| `nav-glide` | 320 | `--transition-duration-nav-glide` | Off the generic ladder on purpose |
| `debounce` | 300 | none | Not a motion value |
| `feedback` | 2000 | none | How long an inline confirmation stays |
| `undo` | 5000 | none | How long an undo toast waits |

Overlay durations are fed from JavaScript so the unmount timer cannot drift from the visual:
`DIALOG_MOTION_MS` 240 and `MENU_MOTION_MS` 160 in `lib/constants/gestures.ts` are written onto the
panel as `--ovl-dur`, whose stylesheet default is 240ms. `EXIT_DURATION_RATIO` is 0.8 in
`lib/styles/motion.ts` and is mirrored by `.ovl-panel[data-state='closed']`.

Ambient and status durations, all in `@theme`: `--duration-reduce-motion` 0.01ms, the blanket kill
value that keeps `transitionend` firing; `--duration-typing-bounce` 600ms; `--duration-refresh-spin`
1100ms; `--duration-gradient-drift` 6000ms; `--sound-disc-spin-ms` 10000ms; `--duration-shimmer` 1.8s
with `--shimmer-band` `clamp(9rem, 26vw, 26rem)`.

`ANIMATION_DURATION` in `lib/constants/timings.ts` is the seconds table Framer reads: `micro` 0.05,
`instant` 0.1, `fast` 0.15, `normal` 0.2, `medium` 0.25, `slow` 0.3, `slower` 0.4, `page` 0.5,
`extraLong` 0.8, `loop` 1, `loopMedium` 1.5, `loopSlow` 2. `ANIMATION_STAGGER` is `fast` 0.03,
`normal` 0.05, `slow` 0.1.

Other named waits a surface may read rather than type: `SKELETON_DELAY_MS` 150,
`NAV_GHOST_GRACE_MS` 100, `NAV_GHOST_MIN_MS` 200, `NAV_GHOST_MAX_MS` 10000, `NAV_WARM_DELAY_MS` 3000, `IDLE_TASK_TIMEOUT_MS`
3000, `IDLE_SLICE_TIMEOUT_MS` 200, `TOOLTIP_DELAY_MS` 400, `TOOLTIP_FADE_MS` 160, `LONG_PRESS_TIMEOUT_MS` 500,
`TOAST_DURATION_MS` 4000, `MAX_VISIBLE_TOASTS` 3,
`DOUBLE_TAP_HEART_DURATION_MS` 800, `QUERY_STALE_TIME` short 30s, medium 1m, long 5m, day.

## Easings

The names are cross-wired between the two files and both names are load-bearing, because
`ease-smooth` is a Tailwind utility and must stay the Material curve. The pairing, not the name, is
the contract, and the mirror spec fails when either side moves.

| CSS | `lib/styles/motion.ts` | Curve |
|---|---|---|
| `--ease-smooth` | `EASE_STANDARD` | 0.4, 0, 0.2, 1 |
| `--ease-dialog` | `EASE_SMOOTH` | 0.32, 0.72, 0, 1 |
| `--ease-enter` | `EASE_OUT` | 0, 0, 0.2, 1 |
| `--ease-exit` | `EASE_LEAVE` | 0.4, 0, 1, 1 |
| `--ease-gentle` | `EASE_ENTER` | 0.25, 0.1, 0.25, 1 |
| none | `EASE_IN_OUT` | 0.4, 0, 0.6, 1 |
| none | `EASE_SMOOTH_OUT` | 0.25, 0.46, 0.45, 0.94 |
| none | `EASE_ARRIVE` | 0.16, 1, 0.3, 1 |
| none | `EASE_BACK` | 0.34, 1.56, 0.64, 1 |

Springs, all in `lib/styles/motion.ts`: `SPRING_SNAPPY` 400/34, `SPRING_SHEET` 400/35,
`SPRING_GENTLE` 200/25.

## Press

Declared in `:root`, read by the `press-subtle`, `press` and `press-firm` utilities, which carry
their own `transition` because `transition-colors` does not include `transform`.

`--press-subtle` 0.98 for a wide surface, `--press` 0.96 for a compact control, `--press-firm` 0.93
for a small glyph, `--press-opacity` 0.7 as the motion-free stand-in under reduced motion.

## The asymmetries, as debt

- **Two duration tables.** `TIMING` is milliseconds and mirrors the CSS; `ANIMATION_DURATION` is
  seconds and does not. `instant` disagrees across them, 75ms against 0.1s, and `medium` 0.25 and
  `page` 0.5 exist only in the seconds table. `TIMING.micro` exists only in TypeScript. A rename
  retires the seconds table onto the ladder; until then a CSS transition reads `TIMING` and a Framer
  transition reads `ANIMATION_DURATION`, and neither invents a third.
- **Two menu durations.** The Framer popover recipes use `ANIMATION_DURATION.fast`, 150ms; the
  CSS-driven anchored popover uses `MENU_MOTION_MS`, 160ms. They describe one animation and are
  one number once the rename lands.
- **Cross-wired easing names.** `--ease-smooth` is not `EASE_SMOOTH`, and `--ease-enter` is not
  `EASE_ENTER`. Read the table above, never the name.
- **Two z-index ladders.** The CSS mirror carries nine of the twenty rungs. A surface that needs one
  of the other eleven sets `zIndex` from `Z_INDEX` inline, which is the sanctioned exception to rule 5.

## Enforced by

- `__tests__/config/design-token-mirror-parity.test.ts`: rules 1 and 7, the radius derivation, the
  spacing multiplier, the easing pairing, the `--ovl-dur` and `--sound-disc-spin-ms` defaults.
- `__tests__/theme/stylesheet-token-gate.test.ts`: rule 2 inside stylesheets, for font size, weight,
  radius, duration and blur, and the single declaration of `--tap-target-min`.
- `__tests__/fitness/arbitrary-value-duplicates-a-token.test.ts`: rule 2 in class strings.
- `__tests__/theme/css-var-resolution.test.ts` and
  `__tests__/fitness/every-css-var-resolves.test.ts`: rule 4.
- `__tests__/theme/token-utilities.test.ts`: every theme-token class the app writes compiles.
- `__tests__/theme/z-index-ladder.test.ts`: the ladder is strictly ordered and the toast, the gate and
  an anchored menu keep their rungs.
- `__tests__/fitness/a-recipe-lives-in-styles.test.ts`: rule 6, including the recipe-export and
  chrome-module baselines.
- `__tests__/fitness/a-text-size-is-not-a-colour.test.ts`: rule 8.
- `__tests__/fitness/a-duration-is-read-not-retyped.test.ts`: rule 3 for the JavaScript ladders — a
  literal in a duration, a `setTimeout` or a `setInterval` that a `TIMING` or `ANIMATION_DURATION`
  rung already names, in a file that reads one of them.
- `__tests__/fitness/one-ladder-answers-a-duration.test.ts`: rule 9's first asymmetry — a rung both
  duration tables name carries one speed, with `instant` recorded as the debt.
- `__tests__/fitness/a-menu-opens-at-one-speed.test.ts`: rules 3 and 9's second asymmetry —
  `MENU_MOTION_MS` against `POP_MENU`, recorded as the debt, and one transition across `POP_MENU`,
  `POP_ANCHORED` and `POP_MENU_UP`.
- `eslint design/no-off-token-style`: blur radius, elevation and near-miss accent, in class strings
  and inline style objects.
- `eslint motion/no-literal-motion`: rules 3 and 5 for duration and easing.
- `eslint local/no-off-scale-sheet-height`: the five sheet rungs.
- `eslint local/no-raw-app-stacking`: a raw `z-<n>` on a fixed element.
- `eslint design/no-untiered-opacity`: an opacity modifier off the named tiers.
- `eslint local/no-press-without-transform`: rule 5 where a transition list drops `transform`.

## Traps

- Rule 4: `--dialog-h-cap` at `:root` read `--dialog-max-w`, which is declared on the panel, so it
  computed to nothing and every desktop dialog lost its ceiling. The panel now declares both.
- Rule 8: `cn()` drops a custom `text-*` size it does not know about, which is how the count badge
  lost its 11px.
- Rule 5: an inline `backdropFilter` cannot be shed by `perf-lite`, which keys off the
  `backdrop-blur` class.
- Rule 1: `@theme` cannot carry an `env()` call. Declared there, every safe-area inset silently
  vanishes and every `max(1.5rem, var(--safe-top))` becomes invalid at computed-value time.
- Rule 2: Tailwind v4 maps `duration-*` to the `--transition-duration-*` namespace, so a bare
  `--duration-*` never compiles into a utility.
- Rule 7: the CSS may hold keys the TypeScript omits. The mirror is a subset by design, and a spec
  that asserted equality in both directions would fail on every JavaScript-only rung.
