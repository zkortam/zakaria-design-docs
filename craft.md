# Craft

Craft is the difference between minimal and bare. This document owns how much room a thing gets, how deep the
hierarchy goes, what shape it is, where its glyph sits, which surface it stands on, and what a control looks like
at rest. Every rule here is measurable on a screenshot with a ruler. The feeling it protects: a person opens any
screen and finds it already familiar, because the same distances, the same corners and the same weights are used
everywhere, and nothing on it is there to decorate.

## Owns and defers

Owns: spacing as structure, the three gaps, hierarchy depth, radius nesting, circles, edges and seams, glyph
weight and optical centring, the surface ladder, a control's resting and active look, control heights, how
much chrome stands above a page's content, the one header control, where creativity is allowed, and the
contact-sheet standard with its reference pages.

Defers: token values and the no-arbitrary-value rule `[tokens]`; page gutters, columns, header and postures
`[layout]`; which primitive carries a concept `[components]`; ink tiers, accent, shadow and effect semantics
`[colour-and-effects]`; the type ramp and weights `[type]`; durations, easings and what moves `[motion]`; the
press ladder, haptics and toasts `[feedback]`; hit-area minimums and contrast `[accessibility]`; media framing
inside a player `[media]`; what a control says `[copy-and-voice]`.

## Rules

### Space is the structure

1. **Three gaps, then stop.** A page picks one gap between rows of a list, one between groups, one between
   sections, and uses only those three. A fourth distance on the same page is a mistake, not a nuance. Groups
   are separated by space, never by a line.
2. **The three gaps come from the scale.** Rows of one list sit 0 to 8px apart, groups 16 to 24px, sections 32px
   or more, every value a step on the spacing scale `[tokens]`. A gap chosen so a particular string fits is a
   layout bug, not a spacing decision.
3. **One gap per row.** Every glyph, avatar, label and trailing control in a row is separated by the same gap
   from its neighbour. `LIST_ROW_LAYOUT` is `gap-3`; `SETTINGS_CARD_ROW` is `gap-3.5`. A row that mixes two
   gaps reads as two rows badly joined.
4. **Density is set by the content.** A messages list is tight because messages are many; the settings hub is
   airy because choices are few. A row is 44px tall at minimum and 56px when it carries an avatar; the minimum
   is the hit area `[accessibility]`, the 56 is craft.
5. **Text never touches an edge.** Media may run edge to edge inside its card. Text, glyphs and controls keep
   the container's padding on every side, including the side the scrollbar is on.
6. **Padding is symmetric or it is deliberate.** A surface whose top and bottom padding differ states why in the
   recipe, the way `FEED_ROW_ACTIONS` does. An asymmetry nobody wrote down is drift.

### Hierarchy is three steps deep

7. **One title, one primary action, one accent per screen.** Everything else is secondary or muted. Two filled
   buttons on one screen means one of them is wrong: the primary is filled, every other action is text or a
   quiet icon button. A filled button inside a list row is N filled buttons, one per row, and the row's
   affordance is the row: a person row follows through `FollowButton quiet`, the profile's own Follow is
   the filled one.
8. **Size, weight and ink move together.** A title is larger, heavier and primary. A subtitle is body, regular,
   secondary. Meta is `sm`, regular, muted. A large muted heading or a small bold label is a step that points
   two ways at once, and the reader stops to work out which one wins.
9. **A row says one thing.** Icon, label, chevron. A subtitle appears only when it carries live state, never an
   explanation of what the row does `[copy-and-voice]`.
10. **Nothing is emphasised twice.** Selected is a fill, not a fill and a ring. Active is one change. Unread is
    one dot. Two signals for one state read as two states.
10a. **Facts on one line are parted by space, never by a glyph.** A handle, a time, a count and a status are each
    their own child of `MetaLine`, which parts them with the `META_LINE` gap; no dot, bullet, pipe or dash stands
    between them, and a hairline drawn between two facts is that pipe in another material. A value that must stay one string (a fold's summary, an accessible name, a title) joins its
    parts with `formatSummary`, in the reader's own list punctuation, and product copy joins with
    `formatPhrases`, which reads as one sentence: only its first phrase keeps a capital. A dot is never decoration: `UnreadDot`
    marks only a state nothing else on the row says, so an unread row is not also bold.

### Shape and edge

11. **Radius nests concentrically.** An element inside a rounded container has the container's radius minus the
    container's padding, from the table below. `nestedRadius(outer, padding)` in
    `lib/constants/design-tokens.ts` is that arithmetic; it has no call sites today, so the table is what a
    reviewer checks.
12. **A child is never rounder than the box that holds it.** Equal radii diverge visibly along the diagonal;
    a larger inner radius reads as a corner cut out of the container. Step down, every time.
13. **Circles are circles.** An avatar, a record button, a selection dot, an unread dot and an invite cell have
    equal height and width and `rounded-full`. `rounded-full` on an unequal box is a capsule, and a capsule is
    only ever a pill, a button or a badge.
14. **A pill is `full`, a control is `sm` or `md`, a card is `2xl` or `3xl`.** A sheet's top corners equal a
    card's. The theme's radius scale moves all of them together `[tokens]`.
15. **Edges are quiet.** A card's hairline comes from the glass recipe, which is inert in a flat theme; the
    component never decides whether it has a border. Shadows come from the elevation ladder
    `[colour-and-effects]`, never from a literal.
15a. **An edge is drawn once.** A surface has one edge at rest, the one its recipe gives it, or none. A ring beside
    a border is a double outline that reads as selected when nothing is; a ring answers a state (focus, selected)
    and draws nothing at rest.
16. **Hairlines are a budget.** A line is allowed only where two lists of unlike things meet. Between rows of
    one list there is space. The reference pages contain none, and a shared `divide-y` recipe is the habit
    promoted to a token rather than a licence to use it.
17. **Seams do not exist.** A frosted header sits under `TopEdgeCap`, the 1px `bg-canvas` strip at
    `Z_INDEX.topEdgeCap` in `components/layout/main-layout.tsx`, so the backdrop seam Chrome paints at the
    viewport's top edge never shows. Two halves of one grid meet at no line because they are one track.

### Glyphs

18. **One family, one weight at rest.** Phosphor, `regular`, with `fill` for selected or active. `bold`,
    `duotone`, `thin` and `light` are not part of the vocabulary, and a second icon family anywhere is a defect.
    The build ships only the weights `lib/config/glyph-weights.ts` names, so a stripped one draws nothing.
    A primitive that picks an off-vocabulary weight hands it to every surface that renders it, so the primitives
    are checked first.
19. **The box comes from `ICON_SIZES`.** Inside an `IconButton` the button sets the glyph size with
    `[&>svg]:h-*` and a `size` prop on the child does nothing, because CSS beats an SVG attribute. Outside one,
    a numeric literal on a Phosphor `size` prop is an arbitrary value `[tokens]`, and so is a local constant
    holding the same number under a different name.
20. **Glyphs are optically centred, not box centred.** A glyph whose mass sits off centre (a play triangle, a
    send arrow) is nudged so the ink looks centred; the nudge lives in the recipe that draws the control, never
    in a call site, and never as a `mt-px` on the icon itself.
21. **One glyph per concept.** From `lib/constants/icons.ts`. The repost loop is the repost loop on every
    surface, at every size, in every theme.
22. **A glyph that needs a tooltip is the wrong glyph.** If it needs a label to be understood it takes the label
    and loses nothing.

### Surfaces

23. **Two layers on the canvas, three in an overlay.** Canvas, card, and the overlay above them. Nothing sits on
    a card that is itself on a card. A list of cards is a list of rows drawn wrong.
24. **Chrome is glass, content is solid.** `GLASS_CHROME` belongs to the rail, the nav and the header and to
    nothing else, and the right panel is a solid card beside the glass rail; a reading column wearing it is a page pretending to be a frame. A card is
    `GLASS_CARD_SOLID`. A popover is `GLASS_POPOVER_SOLID` so it can scale. Content never blurs what is behind
    it, and one frosted layer per screen at rest is the ceiling.
25. **The chrome is quiet and the content is loud.** Chrome uses secondary and muted ink and carries no accent at
    rest. The person's photos, video and words are the only saturated things on the screen.
26. **Media is framed by its card, never by a frame.** No inner border, no second radius, no drop shadow on a
    photo. The card's inner radius is the media's radius, by rule 11.

### Controls

27. **A control looks like what it does.** A toggle is a switch. A choice among few is a chip strip, or a track of
    named stops in settings `[input-and-forms]`. A choice among many is a select or a sheet. A continuum is one
    shape, never a row of dots `[components]`. Only a slider with a few rigid stops wears a circle: a round handle
    that glides between them. `Slider` decides that from the range itself, `rangeStops` of its `min`, `max` and
    `step` against `STEPPED_MAX_STOPS` (seven, the most handles that stand apart on a phone's track), so named
    stops and a short count of whole numbers are a ladder and nothing needs a flag. Everything free-flowing, a
    slider, an editor's slider, a scrubber, a waveform, is its fill alone: no disc rides it, the fill's round
    leading edge is the value, the pointer maps to that edge from one end of the bar to the other, the whole bar
    is the target, and it swells under the finger and a fine pointer so it reads as something to drag rather than
    a meter. The fill is `RangeFill`, a bar slid by a transform under the track's own clip, so no value costs a
    layout. `RangeInput` draws nothing at the value; `RangeThumb` only seats what a host has ride its range, and a
    story sticker's emoji is the one such thing. Keyboard focus rings the bar. A
    centred continuum at rest keeps a stub at its centre, so zero is a place, not an empty bar. A track that only
    reports where a thing stands (a brand deal's stops) is `StepSlider` with `readOnly`: the same named stops and
    fill, no handle, so it never reads as something to drag. A track whose value was set by hand to no stop at
    all (the AI help level over a mix of its switches) is `StepSlider` with `mixed`: the handle rests between
    the two stops it lies between, neither reads as chosen, and the track's own line says the mix is the
    person's, in a sentence like each stop's rather than one word on a line alone. An arrow key from there
    moves to the neighbouring stop on that side; a pointer lands on the stop it touched.
28. **A control rests quiet and changes once.** No border unless the theme gives every field one, no shadow, no
    accent at rest. The active state is exactly one change: a fill, a ring or a knob.
29. **Heights come from the primitive.** The table below is the whole set. A control that needs a height not in
    it extends the primitive `[components]`; it does not write `h-[38px]`.
30. **A control's padding matches its neighbours'.** Two controls side by side in one row have the same height,
    the same radius step and the same horizontal padding, or the row reads as two rows.
31. **A row of controls scrolls; it never wraps.** On a phone a tab strip shrinks so it can scroll
    (`STRIP_SCROLLER`, `STRIP_ITEM`) and secondary actions move into the overflow menu. A second line of
    controls is a defect `[layout]`.

### Precision

32. **Numbers that change use tabular figures** so a count ticks without moving the text beside it `[type §18]`.
33. **A surface lands where it left.** A menu exits toward the corner it grew from, and placement state survives
    until unmount `[motion]`. Craft's part is that the anchor is a real geometric point on the trigger, not the
    screen centre.
34. **Nothing shifts after first paint.** A ghost is the content's exact shape, a control's width is fixed to its
    widest label, and a string arriving late lands in a box that was already its size `[loading-and-states]`.
35. **Alignment is a rule, not an eye.** Every row on a page starts on one vertical rule, including a section
    label above a card: `SETTINGS_GROUP_LABEL` stands on the card's own edge on the settings hub and the
    Sapphire hub alike, so a group's name and its card start on one rule.

### Where creativity lives

36. **In the content.** Full-bleed media, edge-to-edge shorts, a space's cover band, a profile's own background
    behind its whole page. The product is a stage and the person's work is the show.
37. **In one moment per flow.** The double-tap heart, the onboarding card's arrive, the story sticker's pop. One
    per flow, caused by something the person did, never repeated on the same screen `[motion]`.
38. **In precision.** Concentric corners, glyphs on one baseline, a sheet that lands exactly where it left, a
    count that ticks without a shift. Precision is what makes minimal read as premium instead of unfinished.
39. **In a drawn set, if there is one** `(not yet built)`. The app ships no illustration family today. When one
    arrives it is one family judged as a contact sheet, and every SVG filter in it sets
    `color-interpolation-filters: sRGB`, because the default is linearRGB and a shadow drawn in it reads
    lighter than the artist drew it.
40. **Never in ornament.** No gradient on a button, no glow on a card, no confetti, no mascot, no sparkle, no
    decorative dot, no "AI" purple.

### Chrome

41. **A page has at most one row of choices above its content.** Its sections, with the filter at the row's
    end `[components §27]` `[components §28]`. What a refinement is set to is one muted line under that row,
    never a second row of controls, and a header carries actions, not choices.
42. **One header control.** Every control in every header, home's and the profile's included, is `IconButton`
    at `PAGE_HEADER_ICON_SIZE`: a 48px box with a 26px glyph (`ICON_SIZES.chrome`), the size the tab bar and rail draw, so a glyph reads the same size on
    every screen. A header link wears `iconButtonClasses` at the same size, a glyph a badge wraps is sized
    `PAGE_HEADER_GLYPH` (the button reaches only a direct `svg`), a group of controls sits in
    `PAGE_HEADER_ACTIONS`, and home's wordmark is `PageHeader variant="root"`'s own title.

### Restraint

43. **A surface is not a wall.** More than five switches or three sliders written into one surface, rows that
    explain themselves, a paragraph where a gesture would say it: each is a decision or a reading the product
    owes the person. The answer is a default, one ladder, or one object that holds every value and is moved
    directly, and `components/settings/feel-pad.tsx` is the reference: one point dragged in one grid instead of
    ten sliders. Deleting the capability is not the answer; that is bare, not minimal.
44. **One door per action in one view.** Two controls on screen together that do the same thing are one too many:
    the header's plus and the empty state's Create, a glyph and a pill for one sheet, a link and a button both
    opening one place. Keep the one that is there in every state, usually the header's, and remove the other. An
    overflow menu is a second layer behind a press and repeats nothing on screen.

## The concentric radius table

Outer radius minus the container's own padding. Every pair below lands on a token step, which is why these are
the only pairs used; a padding that leaves no step is the wrong padding.

| Container | Padding | Child |
|---|---|---|
| `rounded-3xl` 32 | `p-1` 4 | `rounded-2xl` 28 |
| `rounded-3xl` 32 | `p-2` 8 | `rounded-xl` 24 |
| `rounded-3xl` 32 | `p-3` 12 | `rounded-lg` 20 |
| `rounded-3xl` 32 | `p-4` 16 | `rounded-md` 16 |
| `rounded-2xl` 28 | `p-1` 4 | `rounded-xl` 24 |
| `rounded-2xl` 28 | `p-2` 8 | `rounded-lg` 20 |
| `rounded-2xl` 28 | `p-4` 16 | `rounded-sm` 12 |
| `rounded-xl` 24 | `p-2` 8 | `rounded-md` 16 |
| `rounded-xl` 24 | `p-3` 12 | `rounded-sm` 12 |
| `rounded-lg` 20 | `p-2` 8 | `rounded-sm` 12 |
| `rounded-md` 16 | `p-1` 4 | `rounded-sm` 12 |
| `rounded-md` 16 | `p-2` 8 | `rounded-xs` 8 |

`SETTINGS_CARD` is the worked example: `GLASS_CARD` at `rounded-3xl` plus `p-1`, holding `SETTINGS_CARD_ROW` at
`rounded-2xl`. The theme radius scale multiplies the radii and not the padding, so at `--theme-radius-scale`
1.15 the pair drifts under a pixel; the exact fix is a `calc()` on the child radius and it is debt, not a
licence to pick the child by eye.

## Control heights

Read from the primitives, not from a posture. One control is one height at every width unless the primitive
itself steps it.

| Control | Height | Source |
|---|---|---|
| `Button` `sm` / `md` / `lg` | 40 / 44 / 48 | `components/ui/button.tsx` |
| `IconButton` `xs` / `sm` / `md` / `lg` | 32 / 40 / 44 (40 at ≥640) / 48 | `components/ui/icon-button.tsx` |
| Menu row | 48 minimum | `MENU_ITEM` |
| Slider | 44 hit box; stepped handle 36; continuous bar 36 at rest, 44 held, no handle | `components/ui/slider.tsx` |
| Creation tool | 44 minimum | `CREATE_TOOL_BUTTON`, paired with `ICON_SIZES.rail` |
| Player control pill | 40 | `VIDEO_CONTROL_PILL` |
| Centre play badge | 56 | `MEDIA_CENTER_PLAY` |
| Hero icon tile | 64 | `HERO_ICON_TILE` |
| Count badge | 16 | `COUNT_BADGE` |

`IconButton` `md` is the only control that changes size across a breakpoint, and it steps down because the
coarse-pointer minimum no longer applies, not because desktop wants a different look.

## The contact sheet

A change is judged beside the app, never alone. `npm run ui:probe` drives real Chromium at a phone and a desk
width and writes `tools/uiprobe/shots/<width>-<surface>.png`. Lay the new surface next to the reference pages at
the same width and the same theme. If it reads as a different product, it is not done.

The reference pages, in the order they are read:

| Page | Route | What it is the reference for |
|---|---|---|
| Home | `app/(main)/home` | A flat list on the canvas, no per-row card, no hairline |
| Explore | `app/(main)/explore` | One search pill and one row of tabs, nothing else |
| Profile | `app/(main)/[username]` | The person's background behind the whole page (`ProfileBackdrop`, under `--cover-veil`), and one identity block (`ProfileIdentity`, which Edit profile draws from the draft over a `COVER_BAND` miniature) |
| Settings hub | `app/(main)/settings` | Cards as groups with the label inside, the settings first and the destinations behind one More, rows as icon-label-chevron, zero prose |
| Shorts | `app/(main)/shorts` | Full-bleed content with on-media chrome and no frame |
| Sapphire | `app/(main)/sapphire` | The settings hub's geometry carrying commercial content |
| Inbox | `app/(main)/messages` | Density: many rows, one gap, one avatar size |

Sapphire is the route, not the theme. It reuses `SETTINGS_CARD_ROW` through `HUB_ROW`, which is why it is a
reference: it proves the hub's geometry survives a page that is selling something.

## Enforced by

- `__tests__/components/ui/slider-marks-its-value.test.tsx`: 27, for the handle a few stops wear and the stop
  count that decides it, the continuous slider that is its fill alone, and the bar that swells while held.
- `__tests__/components/ui/range-input.test.tsx` and `__tests__/utils/range-math.test.ts`: 27, for the pointer maths at
  both ends of a track, in both reading directions, the fill as a transform, and a range that draws nothing at
  its value.
- `__tests__/fitness/a-range-is-one-control.test.ts`: 27, for the one host allowed to seat something on a range.
- `__tests__/fitness/a-card-does-not-nest.test.ts`: 23. A card is an elevated recipe in `lib/styles`, and no
  element that names one stands inside another in the same tree.
- `__tests__/fitness/a-circle-has-an-equal-box.test.ts`: 13, on every `rounded-full` class list that fixes both
  an `h-` and a `w-` step.
- `__tests__/fitness/a-control-height-comes-from-a-primitive.test.ts`: 29, on a round control whose height is
  not one the table above lists.
- `__tests__/fitness/a-glyph-wears-one-weight.test.ts`: 18, both halves — the off-vocabulary weight and the
  second icon family — and a weight the build strips, on every surface with no baseline.
- `__tests__/fitness/a-meta-line-is-parted-by-space.test.ts`: 10a, on every string and JSX text in `app/`,
  `components/` and `lib/constants/`, every stylesheet `content`, and every locale; an unspaced middle dot joining
  two nouns in Japanese or Korean is that language's "&" and passes; and no `lib/styles` recipe draws a meta
  divider.
- `__tests__/fitness/a-header-control-is-one-size.test.ts`: 42, on every `IconButton` a `PageHeader` carries,
  every glyph weight inside one, and every wrapped glyph in a header-size `IconButton` anywhere.
- `__tests__/fitness/a-page-has-one-row-of-choices.test.ts`: 41, through the pill row it forbids as chrome.
- `__tests__/fitness/a-nested-corner-is-concentric.test.ts`: 11, 12. Walks each `rounded-*` element up to the
  container that rounds it, sums the padding between, and checks the table's arithmetic.
- `__tests__/fitness/a-page-picks-three-gaps.test.ts`: 1, 2, counted per page shell across `gap-*` and
  `space-*`.
- `__tests__/fitness/a-recipe-lives-in-styles.test.ts`: 3, 15, 24, 29; a class string used twice is a recipe.
- `__tests__/fitness/a-surface-is-not-a-wall.test.ts`: 43, counting switch and slider rows written into one file.
- `__tests__/fitness/an-edge-is-drawn-once.test.ts`: 15a, on every class string that sets a resting border and a
  resting ring together.
- `__tests__/fitness/one-door-per-action.test.ts`: 44, for an empty state (`EmptyState`, or a wrapper's `empty`
  prop) that repeats a handler, destination or label its page header already carries.
- `__tests__/fitness/a-text-size-is-not-a-colour.test.ts`: 8.
- `__tests__/fitness/an-empty-state-is-the-primitive.test.ts`: 23, 27.
- `__tests__/fitness/arbitrary-value-duplicates-a-token.test.ts`: 19, 29, and rule 11 wherever a radius is
  written as `rounded-[Npx]`.
- `__tests__/fitness/every-css-var-resolves.test.ts`: 14, where a radius var is read on a panel.
- `tools/uiprobe/`: the contact sheet. Reports rather than asserts, on purpose.

## Traps

- **11** `nestedRadius` exists and is called nowhere, so every nested corner in the app was picked by hand and
  the ones that agree do so by luck.
- **13** A card given a height and no width, whose children are all absolute, collapses to a sliver; the
  `rounded-full` on it then draws an oval.
- **17** Without `TopEdgeCap` Chrome paints a 1px warm line along the top of every sticky `backdrop-filter`
  header, and it only appears on a real browser at a real device pixel ratio.
- **19** Tailwind's scanner reads class strings statically and cannot resolve a constant, so a glyph box
  interpolated into a class ships no CSS. `IconButton` and `Avatar` write their size maps as literals for
  exactly this reason, and the numbers there must be kept in step with `ICON_SIZES` by hand.
- **20** A `tap-target` pseudo-element on a wrapper draws a 44px `::after` over the wrapper's own children, so a
  tap on a name inside an author pill lands on the pill.
- **24** Scaling a `backdrop-filter` layer re-rasterises it every frame, so a glass popover drops frames on open;
  the popover is solid and the frosted layer behind it glides away instead.
- **31** `scrollWidth` cannot be used to detect a wrapped or overflowing row: a `tap-target` pseudo-element and
  any descendant scroller inflate every ancestor. Judge it from the screenshot.
- **35** `<table className="text-start">` does not beat the user-agent sheet's `th { text-align: center }`, so a
  header sat off its own column while the class looked right.
- **41** Explore stacked a tab row and a row of five refine pills; notifications stacked a filled pill row under
  its header; bookmarks put two filter pills in its header and cut its own title to "Bookma…".
- **42** Home hand-rolled its bar's controls at 28px bold in a 44px box while `PageHeader` drew 24px regular in
  48px, so the icons on home read a size larger than the same icons on the profile.
