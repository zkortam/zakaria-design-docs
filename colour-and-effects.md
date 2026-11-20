# Colour and effects

Colour in this product is quiet. The chrome is neutral, the person's photos and words are the only
saturated things on a screen, and the accent marks exactly one thing at a time. Effects are a property
of the theme, never a decision a component makes: a card asks for a sheen and gets a gradient in Aero
and nothing in light, and it never asks whether the effect is on. Eight themes are first-class, which
means a screen that reads well in dark and badly in Aero is a screen that is not finished.

## Owns and defers

Owns: the ink tiers and what they mean, the two surfaces, the accent per theme and what it may mark,
semantic colour, the category ramp, on-media legibility, colour containment, the eight themes as one
contract, the high-contrast layer, theme switching without a flash, and the effects table with its
rules.

Defers: the token names, blur radii and shadow rungs to `[tokens]`; which shadow a surface picks and
how surfaces stack to `[craft]`; the durations and curves an effect transitions at to `[motion]`;
contrast minima, reduced transparency and the high-contrast setting as an accessibility contract to
`[accessibility]`; the theme-color meta tag, the icon plate and the pre-paint stamp as part of arrival
to `[arrival]`; media filters and blurhash to `[media]`.

## Rules

### Ink and surfaces

1. **Ink has four tiers.** `--text-primary`, `--text-secondary`, `--text-muted`, `--text-faint`. Pick
   by hierarchy, never by taste. A fifth tier is a bug.
2. **`faint` is chrome, never data.** A number, a name, a timestamp or a status a person acts on is
   `muted` at the lowest.
3. **`--text-inverted` is the ink on the inverted chip**, not a fifth tier and not a way to get white.
4. **Two surfaces on the canvas.** The canvas and a card. A card is `GLASS_CARD_SOLID` or
   `GLASS_CARD`; chrome is `GLASS_CHROME`; an overlay panel is `GLASS_OVERLAY_SOLID` and a popover is
   `GLASS_POPOVER_SOLID`. Nothing sits on a card that is itself on a card. The ground those solid
   recipes share, `--surface-overlay-strong`, is opaque in every theme: a 97% panel with no blur is a
   legible ghost of the post behind it.
5. **Chrome is glass, content is solid.** Content never blurs what is behind it.
6. **Never a raw white or black.** `text-white` says nothing about why. `on-media` and `on-accent`
   name the two reasons a foreground is fixed rather than themed, and `scrim` names a wash laid over
   media rather than a dark theme surface.
7. **An opacity modifier lands on a named tier.** The tiers cover the roles; a near-miss alpha is
   drift off one.

### Accent

8. **The accent marks the one thing that matters** on the screen: the primary action, the active tab,
   the live indicator, a selected item's fill. Never decoration, never two things at once.
8a. **Accent never colours a word the product wrote.** A text action, a link in product copy, "View all", a
    retry, a toast's Undo, a count or a section title is primary or secondary ink; its weight and its place say
    it can be pressed (`TEXT_ACTION`, and `INLINE_ACTION` inside a sentence). Accent on words belongs only to the
    links inside a person's own text, which `RichText` paints, and to the terms linked at a legal assent
    (`AssentNotice`), which must read as links at rest for the assent to hold. A glyph may still take accent to
    mark an on state.
9. **The accent is a theme's own hue,** not a brand constant. What it is per theme is the table below.
10. **The call to action is its own token.** `--cta` and `--on-cta` are declared per theme, because
    light and dark put the primary button on the high-contrast inverted slab while the look themes put
    it on the accent. A button reads `bg-cta text-on-cta`; it never reads `--accent` directly.
11. **Ink on a fill is theme-owned.** `--on-accent`, `--on-accent-secondary`, `--on-status` and
    `--on-cta` are declared by every theme, so a theme with a pale accent is forced to declare a dark
    foreground rather than inherit an unreadable white.
12. **A theme-following ink never sits on a fixed ground.** A media scrim, call chrome or a story
    overlay is dark in both polarities, so a themed ink token painted on it goes near-black on black
    in a light theme.

### Semantic and category colour

13. **Semantic colour is state, never a theme.** `success`, `warning`, `error`, `info`, at `/5` fills
    and `/40` borders as in `STANCE_COLORS`. A feature never adopts one as its look.
14. **Every status hue has two rungs.** The saturated step is the fill and reads on a dark canvas; the
    deep step is what stays legible as text or a glyph on a light one. The palette was written for the
    dark canvas, where shared with light `text-warning` scored 1.97:1.
15. **The category ramp says different, not meaningful.** `--cat-*` is a scale of distinguishable
    hues; each theme shifts it toward its own ink by `--cat-shift` rather than replacing it, so the
    same scale reads on any ground.
16. **Social hues are their own pair.** `--like-color`, `--repost-color` and their muted fills.

### On media

17. **On-media ink is fixed and tiered.** `--color-on-media` and its `--color-on-media-*` steps (strong,
    secondary, muted, faint) with `--color-on-media-fill-subtle`, `--color-on-media-fill-strong`,
    `--color-on-media-hairline` and `--color-on-media-border-strong`.
    They never flip with the theme, because the backdrop is a photo.
18. **The scrim has four weights.** `hint` 0.25, `subtle` 0.4, `soft` 0.55, `strong` 0.7, `heavy`
    0.85, always black. A light theme that lightened a scrim would erase the contrast it exists to
    create.
19. **Text on media carries its shadow.** `--shadow-on-media`, through `drop-shadow-text`, which
    traces the glyph's alpha rather than its bounding box. A caption card over a photo is
    `FEED_MEDIA_CAPTION_CARD`; a hairline on media is `border-on-media-hairline`.
20. **Colour is contained.** A tint, a gradient, a cover photo or a space's theme lives inside a
    bounded band, card or cover. Never a full-pane wash, and never behind chrome that has to read. The one
    full pane of colour is a light, not a tint: the screen lighting a face in the dark `[media §58]`.

### The eight themes

21. **Every theme declares every token.** `lib/theme/contract.ts` is the list; nothing inherits from
    another theme, so a theme can never be half-declared and never needs an override to correct
    another theme's value. Adding a theme is a registry entry, a copied block and a contract run.
22. **Dark is not inverted light.** Each theme declares its own ink opacities, shadows, canvas and
    inverted text. A theme's neutrals derive from a handful of `--theme-*-rgb` anchors, so retinting is
    a few numbers rather than fifty.
23. **A theme is a palette and a material.** The material half is `--surface-sheen`, `--bevel`,
    `--bevel-inset`, `--glow-hover`, `--gel-image`, `--gel-shadow`, `--gel-sheen-peak`, `--reflect`,
    `--theme-radius-scale`, `--theme-font-sans`, `--canvas-image`, `--canvas-overlay`, `--canvas-veil`
    and `--canvas-ornament`. A palette can only ever recolour.
24. **The high-contrast layer is the one sanctioned override.** It is keyed on the scheme, not a theme
    id, it declares no token a theme does not already own, and it derives from the active theme's ink
    anchor so it follows the palette instead of fighting it.
25. **A dark check sets both switches.** The account's theme and the OS colour scheme must both be
    dark, or the browser renders light and the screenshots have measured nothing.
26. **A change is checked in at least light and dark**, and any change to a material token is checked
    in Aero, which is the only theme that turns every material switch on.

### Switching without a flash

27. **The theme is on `<html>` before the first paint.** A blocking boot script reads the stored
    preference, resolves `system` against `prefers-color-scheme`, and stamps `data-theme` and
    `data-theme-scheme` together, alongside the perf tier, the reduce-motion class and the text size.
    Nothing applied in an effect: an effect paints the wrong frame first.
28. **Both attributes move together, always.** `applyTheme` and `themeScopeAttrs` write the id and the
    scheme in one call, so the `dark:` variant and the native `color-scheme` can never disagree with
    the palette.
29. **Nothing transitions on a theme change.** `body` sets its colour and background from tokens with
    no transition, so a switch is one repaint and never a smear. A surface that eases its own
    background colour smears the switch, and does so at whatever duration it typed.
30. **The `dark:` variant keys on the scheme, not the id,** so a new dark-polarity theme inherits every
    dark utility and the variant line never needs editing.
31. **A subtree may render in a theme the document is not using**, through `themeScopeAttrs`, which is
    what makes a theme preview the theme rather than a picture of it. A dialog pins its palette the
    same way when it opens from an always-dark surface such as the camera.

### Effects

32. **An effect is a theme property.** A theme declares it once as a token; a component reaches for
    the utility and never decides whether the effect is on. In a flat theme every one of these is
    inert, so the class is unconditional at the call site.
33. **Blur is a class, never an inline style,** so the low-end tier and the reduced-transparency
    preference can shed it. The one blur that survives both is `protect-blur`, which hides sensitive
    media.
34. **One frosted layer per screen at rest.** Chrome frosts; nothing else does.
35. **Never scale a blurred surface.** A `backdrop-filter` layer re-rasterises every frame it is
    scaled, so anything that animates on open is solid, and a frosted layer behind a morph glides away
    instead.
36. **Blur radii stay at their tuned values** `[tokens]`.
37. **Gel replaces sheen and bloom; it never joins them.** Both set `background-image` and the bloom
    replaces `box-shadow` on hover, which would drop the gel's lip and inner shade the moment the
    pointer arrived. The gel owns both properties including its own hover.
38. **Effects never carry meaning.** Selected, active, live and unread are shown by fill, ink and
    glyph weight, never by glow.
39. **Ambient motion is atmosphere, and it is shed.** The bubble field, Glow's colour field, the entry
    screens' rising feed and the sound disc stop under `perf-lite` and under reduced motion; a loader does not. Shedding under both
    is what buys the loop `[motion]`; the field itself stays, only its drift goes. While the page scrolls
    (`html[data-scrolling]`, set by `SceneRest`) or an overlay is open (`html[data-scroll-locked]`) the drift
    pauses, since every blurred surface over a moving backdrop re-blurs each frame; no bubble holds a permanent
    `will-change`, and `perf-lite` also drops Glow's `screen` blend.
40. **A blur needs a translucent ground.** `--surface-overlay-strong` and `--surface-solid` are opaque in
    every theme, so a blur over either paints nothing and still resamples every frame. Pinned chrome on the
    strong overlay (`MOBILE_STICKY_HEADER`, the header pill, the profile tab strip, `GLASS_STICKY`) takes
    `chrome-plate`, which keeps the layer and the containment and drops the blur.
41. **Nothing pinned over the canvas paints `--canvas`.** The canvas is a picture in the six look themes
    (`--canvas-image`, and Glow's `.glow-scene` blooming from the foot of the window), so a band of the flat
    colour under a pinned control is a slab cut across it, darkest exactly where Glow is brightest. A control
    pinned over a page floats on a plate of its own, as the header pill does: the feed panel's request field is
    `FeedSettings` with `dock="afloat"` on its page, and rests on a band only inside a dialog, whose ground is
    the one opaque `--surface-overlay-strong`.

## The eight themes

| Theme | Scheme | Family | Canvas | Accent | Call to action | Radius scale | Category shift |
|---|---|---|---|---|---|---|---|
| Light | light | mode | #f4f5f8 | #2563eb | the inverted chip | 1 | 46% |
| Dark | dark | mode | #050508 | #2563eb | the inverted chip | 1 | 0% |
| Glass | light | look | #a8d8f5 | #0b6fb8 | the accent | 1.15 | 66% |
| Aero | light | look | #a8d8f5 | #0b6fb8 | #095689 | 1.15 | 66% |
| Retro | dark | look | #080617 | #b52f78 | the accent | 0.96 | 0% |
| Celestial | dark | look | #040817 | #4d62bd | the accent | 1 | 0% |
| Glow | dark | look | #08070f | #6d4aee | the accent | 1 | 0% |
| Sapphire | dark | look | #04162f | #2f6fe0 | the accent | 1 | 0% |

Aero moves `--cta` off `--accent` onto the deeper rung because the gel's white peak composites over
the fill under the label. Glow is the only theme whose ground is not one colour: `.glow-scene` blooms
teal, blue, violet and magenta out of the foot of the window and drifts them past each other, so what
ink sits on is a range, which is why its tiers are heavier than the other dark looks'. Sapphire is the
only theme a tier gates, through `PREMIUM_THEME_FEATURE`.
`system` is a preference, not a theme: it resolves at apply time and is never written to `data-theme`.
The default for a first-time visitor is dark.

## Ink per theme

Every tier is an alpha on the theme's own `--theme-ink-rgb`, so a retint moves them together.

| Theme | Ink anchor | secondary | muted | faint | inverted |
|---|---|---|---|---|---|
| Light | 11, 11, 18 | 0.66 | 0.57 | 0.45 | white |
| Dark | 255, 255, 255 | 0.6 | 0.46 | 0.35 | #060a1e |
| Glass | 6, 38, 56 | 0.72 | 0.69 | 0.54 | white |
| Aero | 6, 38, 56 | 0.85 | 0.8 | 0.62 | white |
| Retro | 242, 244, 255 | 0.68 | 0.54 | 0.4 | #0b071c |
| Celestial | 242, 245, 255 | 0.7 | 0.56 | 0.42 | #070b20 |
| Glow | 244, 243, 255 | 0.76 | 0.63 | 0.47 | #0a0818 |
| Sapphire | 237, 244, 255 | 0.7 | 0.56 | 0.42 | #050f26 |

The high-contrast layer replaces `secondary` with 0.92 and `muted` with 0.8 on the same anchor, and
raises `--glass-border` to 0.62, because that token carries 282 of the borders on screen while
`--border-default` carries almost none.

## The effects table

| Effect | Utility or token | Where it is on | Where it is inert |
|---|---|---|---|
| Frost | `.glass-chrome`, `backdrop-blur-*` | The translucent rail and right panel, overlay surfaces | `perf-lite`, `prefers-reduced-transparency` |
| Plate | `chrome-plate`, `MOBILE_STICKY_HEADER` | The tab bar, sticky headers, the header pill, the profile tab strip: pinned chrome on an opaque ground | Never; it has no blur to shed |
| Sheen | `surface-sheen`, `--surface-sheen` | Glass, Aero, Retro, Celestial, Glow, Sapphire | Light, Dark |
| Hover bloom | `glow-hover`, `--glow-hover` | The same six, under `(hover: hover)` only | Light, Dark, every coarse pointer |
| Recess | `bevel-inset`, `--bevel-inset` | The same six, on a field carved into its panel | Light, Dark |
| Gel | `gel`, `--gel-image`, `--gel-shadow`, `--gel-sheen-peak` | Aero only, on a filled control | Every other theme |
| Reflection | `reflect`, `--reflect` | Aero only, on the brand marks of the entry screens | Every other theme, and Firefox |
| Edge treatment | `--bevel`, composed into `--sh-glass` | Per theme | |
| Canvas image | `--canvas-image`, `--canvas-overlay`, `--canvas-veil`, `--canvas-ornament` | The six look themes | Light, Dark, which read `none` and paint nothing |
| Ambient | `aero-bubble-drift`, `glow-orb-drift`, the sound disc spin | Aero's bubbles, Glow's colour field and player status | `perf-lite`, reduced motion |
| Scrim | `--scrim-media`, `--scrim-media-strong`, `bg-scrim-*` | Behind on-media text and behind every overlay | Never; fixed in every theme |
| On-media legibility | `drop-shadow-text`, `text-on-media-*`, `border-on-media-*` | Text and hairlines over photos and video | Never |
| Elevation | `--shadow-xs` to `xl`, `--shadow-glass`, `--shadow-glass-heavy` | Per theme | |
| Shimmer | `.skeleton-shimmer`, `--shimmer-band`, `--duration-shimmer` | Bones only, and it keeps running under reduced motion | Never; a frozen bone reads as a hang |
| Privacy | `protect-blur` | Sensitive media and the app-switcher cover | Never; the one blur that survives `perf-lite` |
| Calm colours | `html.calm-colors img, video` | The wellbeing setting, on media elements only | Off by default |
| Edge cap | `Z_INDEX.topEdgeCap` | The 1px cap that hides the backdrop seam under a frosted header | Never |
| AI listening edge | `--ai-edge-stops`, `--ai-edge-width`, `--ai-edge-radius`, masked per layer with `--blur-strong`, `--blur-media`, `--blur-overlay` | A perimeter ribbon while an AI surface listens: three stops from the theme's accent, its complement and white at 0.9, 0.7 and 0.5 alpha, drifting; thickness from the input level | Flat themes keep the ribbon and drop the drift `(not yet built)`: no material token marks a theme flat; `perf-lite` reduces it to a 2px solid accent edge |
| Live activity material | `GLASS_POPOVER_SOLID` while it morphs, `GLASS_CHROME` at rest (not yet built) | The capsule in the app frame | Never |
| AI rewrite shimmer | `skeleton-shimmer` band over live text, then `state-fade-in` (not yet built) | Text an AI is rewriting | Reduced motion: the dissolve only |

## Enforced by

- `__tests__/fitness/accent-never-colours-a-word.test.ts`: rule 8a, on every class string that sets accent ink
  together with a size, a weight or an underline, and on the two shared text-action recipes.
- `__tests__/theme/theme-contract.test.ts`: rules 4 (the opaque overlay ground), 21, 22 and 24, plus the contrast gate, which scores
  every text tier, status fill, social hue and category step in all eight themes and fails a pair
  below its minimum. `node scripts/theme/report-contrast.mjs` prints the table with headroom.
- `__tests__/theme/glow-field-legibility.test.ts`: Glow's field, composited at the brightest point its
  four orbs can reach at any window shape, with every ink tier scored there as well as on `--canvas`.
- `__tests__/theme/ink-context.test.ts`: rules 6 and 12, for a themed foreground on a fixed scrim, a
  media-fixed foreground on a themed panel, and a theme-following foreground on an accent fill.
- `__tests__/theme/token-utilities.test.ts`: every theme-token class the app writes compiles.
- `__tests__/theme/boot-script.test.ts`: rule 27, pinning every branch of the pre-paint script.
- `__tests__/theme/cover-gradients.test.ts`: the cover presets and their geometry across both views.
- `__tests__/fitness/a-theme-switch-is-one-repaint.test.ts`: rule 29, for a colour transition on the
  document surface, and for a chrome surface that eases its colour with no state variant beside it.
- `__tests__/fitness/one-frost-per-screen.test.ts`: rule 34, for a screen that writes its own
  `backdrop-blur` instead of taking a recipe that already frosts.
- `__tests__/fitness/a-surface-that-moves-is-solid.test.ts`: rule 35, for a blurred surface that
  scales, fades, translates or transitions. Owned as `[motion §41]`.
- `__tests__/fitness/a-blur-needs-a-translucent-ground.test.ts`: rule 40, for a class list that pairs an opaque
  plate with a backdrop blur.
- `__tests__/components/settings/feed-settings.test.tsx` and `__tests__/components/settings/feed-settings-page.test.tsx`:
  rule 41, for the request field afloat on its own plate on the page, with nothing painted across the canvas, and
  banded only in a dialog.
- `__tests__/fitness/an-effect-carries-no-meaning.test.ts`: rule 38, for a glow or a shadow token
  switched on a selected, active, live or unread flag.
- `eslint design/no-raw-neutral-literal`: rule 6.
- `eslint no-restricted-syntax`, the arbitrary-colour selector: rule 6, for a `bg-[#…]`, `text-[rgba(…)]` or
  `ring-[hsla(…)]` in any class string, which the palette-name guard never matched.
- `eslint design/no-theme-flip-on-dark-surface`: rule 12.
- `eslint design/no-untiered-opacity`: rule 7.
- `eslint design/no-off-token-style`: rules 33 and 36, for blur radius, elevation and near-miss accent
  in class strings and inline style objects.

**Not yet enforced**

- Rule 8: nothing counts accents per screen. `one-accent-per-screen` would need a rendered tree, so it
  belongs in the browser check rather than in jsdom.
- Rule 34: the second frost a route assembles out of two files is invisible to source — every route's
  import closure is most of the app — so counting frosted layers per screen belongs in the browser
  check too. Source holds the half that matters: a screen never writes its own frost.

## Traps

- Rule 25: a dark sweep that reads light on light has measured nothing. Both the account theme and the
  OS scheme must be dark.
- Rule 27: a hook that reads a value in an effect instead of during render paints the wrong frame
  first, which is why the theme, the perf tier and the text size are stamped by a blocking script.
- Rule 27: `localStorage` throws in a partitioned or cookie-blocked context rather than returning
  null, so the boot script uses three independent `try` blocks and one failure never takes the others
  down.
- Rule 33: an inline `backdropFilter` cannot be shed, because both shed switches key off the
  `backdrop-blur` class.
- Rule 41: `--canvas` matches the page in Light and Dark, where `--canvas-image` is `none`, so a band of it
  looks seamless on the two themes most screens are checked in.
- Rule 33: the minifier treats `-webkit-backdrop-filter` and `backdrop-filter` as one property and
  keeps only the last, so the standard half lives inside an `@supports` block that cannot be merged.
  Shipping only the prefixed half left Chromium with no frost and made both shed switches no-ops.
- Rule 35: a backdrop blur behind a morphing sheet smears the morph and drops frames; the tab bar
  glides away instead.
- Rule 37: `glow-hover` replaces `box-shadow` wholesale on hover, so composing it with `gel` dropped
  the gel's edge the instant the pointer arrived.
- Rule 32: `bevel-inset` and `gel` each carry their own `0 0 #0000` fallbacks in every Tailwind shadow
  slot, because without them the whole `box-shadow` is invalid whenever Tailwind has not emitted the
  `@property` defaults, and the effect silently resolves to `none`.
- Rule 4: a single shared `--glass-bg-strong` once left a near-black card carrying theme-following
  labels, and a whole header went black on black in light mode. It is declared per theme.
- Rule 20: `filter` on `<html>` or `<body>` re-anchors every `position: fixed` element to the document
  box, so the calm-colours filter lands on the media element itself.
- Rule 18: seventeen distinct `bg-scrim/N` steps shipped for what reads as four, and fourteen
  hand-picked `text-on-media/N` alphas for what is four roles. The tiers name the roles.
