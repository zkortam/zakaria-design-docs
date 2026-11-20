# Layout

One product in two postures. The phone is the primary posture and the desktop is the same product
with room, never a second design. This document owns the shape of a page: which posture a width
belongs to, how wide the reading column is at each rung, where the gutters are, how tall the header
is, what clears the notch and the home indicator, and how the shell rearranges without ever
remounting. A person moving between their phone and their laptop recognises the same product
immediately, and a person turning their phone or dragging a window edge never sees a page break.

## Owns and defers

Owns: the postures and their boundaries, the two column measures and their rungs, the page gutter,
the header's geometry, safe areas, honest widths, the shell modes, landscape, thumb reach, and window
resizing on desktop.

Defers: the token values these rules read to `[tokens]`; spacing rhythm, density and hierarchy to
`[craft]`; what the header and rail look like to `[colour-and-effects]` and `[craft]`; the primitives
that draw them to `[components]`; navigation, Back and scroll restoration to `[navigation-and-history]`;
cursor, scrollbar and the widths between postures at the platform level to `[platform]`; image `sizes`
consumption to `[media]`.

## Rules

### Postures

1. **Two designs, chosen by size class.** Compact is a phone-shaped window: narrower than 600, or narrower than
   1024 and shorter than 480 at rest, so every phone in either orientation, a folded foldable and a narrow iPad
   window. Regular is everything else: every full-screen iPad and Android tablet in both orientations, an unfolded
   foldable and every desktop window. `sizeClassOf(width, restingHeight)` is the one function; the boot script sets
   `html[data-size-class]` before first paint and the viewport store (`[input-and-forms §10]`) updates it in the
   frame it publishes `--app-vh`, holding it while `data-keyboard='open'`, because a tablet whose page shrinks for
   its keyboard must not change design mid-typing. CSS asks with the one variant, `regular:`; script asks
   `useSizeClass()`, and list-and-detail asks `useSideBySide()`. Inside regular the width rungs only resize:
   the right panel from `xl` (an upright tablet has the rail and the page alone: three columns crowd it), the desk shapes of the shorts frame and the post from `lg`,
   below which the post is one column with its composer docked on that column, clear of the rail. Chrome only a
   phone wears is asked of the size class too, never of `lg`, or a tablet upright wears it beside the rail:
   `PageHeader`'s pinning and hide-on-scroll, the plate under a bar the phone pins (`COMPACT_HEADER_PLATE`, home's
   and the profile's), Explore's fixed search header (`EXPLORE_HEADER_BAR`), the profile's edge-to-edge tab strip
   (`TAB_STRIP_PIN`), the comment sheet a reply opens, and home without its composer (`INLINE_COMPOSER_SLOT`). A
   handler or a warm-up that only samples asks `currentSizeClass()`. A one-pane surface in
   compact centres on the reading measure rather than stretching across a turned phone, as the messenger does
   through `MESSENGER_FRAME`. `regular:` sorts after every width rung, so a property it sets keeps its wider
   rungs inside it (`regular:3xl:ms-64`).
2. **Layout asks width, interaction asks pointer.** A layout choice reads `compact` or `desktop`; an
   interaction affordance reads `phone`, `touch`, `fine` or `hoverCapable`, so a tablet gets the
   compact layout without inheriting phone-only gestures. A hold, a right-click and the selection a
   hold suppresses read the event's `pointerType` or `pointer-coarse:`/`pointer-fine:`, never a width
   hook: an iPad is as wide as a laptop and has no right button.
3. **Overlay presentation switches at 640.** `useIsXSmall` decides whether a dialog presents as a
   bottom sheet or a centred panel, and every anchored menu, the context menu and a picker become popovers
   at the same 640, so the sheet boundary is `sm`, never the size class.
4. **No hover-only affordance.** Anything a hover reveals is reachable by tap. Hover states live under
   `(hover: hover)` alone, because a stuck hover after a tap reads as broken.
5. **Every feature has a way in below 1024.** A destination reachable only from the desktop rail does
   not exist on a phone.

### The two measures

6. **A page picks one of two column measures**, from `lib/constants/layout.ts`, never a literal
   `max-w-`. `READING_COLUMN` is the default; `WIDE_COLUMN` is for a page whose content is a grid.
   A literal pins one rung and drifts from the column at every rung above it. The reading measure
   stops at 700 because 700px of body text is already 95 characters and a rung above that makes a 4K
   desk worse, not better; the wide measure climbs to `5xl`, because a grid answers a bigger screen
   with another column rather than a longer line. A grid that gains a column keeps the tile count, the
   run columns and the media block on ONE ladder — `GRID_RUNGS` in
   `components/explore/discovery/explore-grid-ghost.ts` — or a block that is not whole rows of the
   track leaves holes beside the run after it. `(not yet built)`: past 2560 the grid stops growing,
   because a `repeat(auto-fill, …)` track publishes no column count for a block size to be dealt
   against, so a 3840 panel still shows a 1280px grid in a 3574px area.
7. **The wide measure is a route list, not a prop, and the route is the one in flow.**
   `isWideColumnRoute(pathname)` holds the exact matches, and a subpath is a normal page unless it
   says otherwise there. `pageColumn(pathname)` hands that measure — or the full-height shell's whole
   area — to the page column, and the `pathname` it reads is always the route on screen, never a
   pending one: a frame that took the destination's shape at the press stretched the page still being
   read, for the length of the grace `[loading-and-states §18]`. The ghost carries its own
   destination's frame instead `[loading-and-states §19]`.
8. **Image `sizes` derive from the measure.** `READING_COLUMN_SIZES`, `READING_COLUMN_HALF_SIZES`,
   `WIDE_COLUMN_HALF_SIZES` and `EXPLORE_TILE_SIZES` are computed from the column string by
   `columnImageSizes`, so a request can never ask for 700px inside a 560px card, or 220px for a tile
   drawn at 287.

### Gutters and edges

9. **The gutter is one width per posture, and it is a constant.** `PAGE_GUTTER` is 10px of canvas either
   side of the column below `lg` and nothing above it, and `pageColumn` is its one reader, so a card, a
   field and the header pill all stand on one edge and no text meets the glass. A phone's bar whose chrome runs
   to the screen's edge (home's bar, the profile's) undoes it with `PAGE_BLEED`, in compact alone: beside the rail
   the bar has no chrome and its row stands on the column; the tab bar pill stands on the same 10px. A page's own inset is its recipe, not a number
   typed on the container: the feed's is `ROW_GUTTER`, a card's is `CARD_PADDING` (16px) and an
   overlay's is `[components §14a]`, so text sits the same distance from its surface's edge on every screen.
10. **The full-bleed surfaces supply their own inset.** Messages is full-bleed rather than centred, so
    its headers carry `FULL_BLEED_HEADER`, `px-2.5 shrink-0`, the same 10px as `PAGE_GUTTER`, so a
    messenger sub-page's pill stands where every other page's does.
11. **Content touches the edge only when it is media.** A photo may run to the card's inner radius.
    Text never does.

### The header

12. **One header everywhere.** `PageHeader` with the `PAGE_HEADER_*` recipes from
    `lib/styles/page-header.ts`. The title is `PAGE_HEADER_TITLE`; actions sit in the header pill,
    `PAGE_HEADER_PILL`, each one control `[craft §42]`. A tab's first screen (home, the inbox, spaces,
    the studio) is `PageHeader variant="root"`: flat, no back, the same title box, with `leading` where a
    page would carry Back; its ghost reads `pageHeaderRootRow`, so the two cannot drift. A root title never
    truncates: beside it sit the one primary action and a filter at most, and every rarer door folds
    into one overflow menu, whose trigger carries any count those doors hold (the inbox header).
    Home's wordmark is the one centred title (`centreTitle`, `PAGE_HEADER_TITLE_CENTRED`): the brand
    stands on the bar's centre line, not after the controls that lead it.
13. **The header's height is one token.** `--page-header-row` is 3.5rem and `--page-header-h` is what
    anything pinned beneath it offsets by. Writing the padding and the offset separately is how the
    header once sat on top of the profile tab strip.
14. **Chrome never resizes under its content.** A header does not shrink when a field inside it
    focuses, a rail does not shift when a selection changes. A bar that responds to scroll moves on
    transform and keeps its height.
15. **A chat bar is the same row.** `CHAT_HEADER_HEIGHT` reads `--page-header-row`, so the messenger's
    richer bar stands exactly where every other header stands, and the loading bar matches it.

### Safe areas

16. **Never `env()` directly.** Every inset is read through `--safe-top`, `--safe-right`,
    `--safe-bottom`, `--safe-left`, or through the `*-safe` utilities, so a host can declare the insets
    the hardware would report and get the app's own spacing logic unchanged. iOS reads `env()`, which WebKit
    keeps live through every turn and resize; on Android the shell injects them at every WebView version
    `[shells §14]`.
16a. **A side is logical.** `--safe-inline-start` and `--safe-inline-end` resolve to the left and right
    insets and swap under `[dir=rtl]`; every `ps-`, `pe-`, `start-` and `end-safe*` utility reads them, and
    `--safe-left`/`--safe-right` are read only by physical positioning. An Arabic page pads the edge the
    cutout is on.
16b. **The keyboard pays the bottom inset.** Under `html[data-keyboard='open']` `--safe-bottom` is `0px` and
    the physical value stays in `--inset-bottom`, so a composer sits flush on the keyboard rather than on a
    home-indicator band. No surface writes its own keyboard conditional around an inset: it reads the
    `*-safe` utility and the token does the rest `[input-and-forms §10]`. A sheet the keyboard lifts stops at
    `--sheet-h-lifted`, and the editor's panels read the same token, so neither rises under the status bar or
    the Dynamic Island; `--sheet-h-full` is capped the same way for a landscape phone or a split window.
16c. **Every width pays.** An iPad, a foldable's inner screen, an Android tablet and DeX all have bars, so a
    wider rung changes the step (`md:top-safe-min-3`) and never drops the inset. Where the element leaves the
    screen's edge at that rung, for a card or a centred dialog, its class list says so (`sm:relative`) or scopes
    the inset to the rungs on the edge (`max-md:top-safe`). The rail and the right panel pay
    `pt-safe-min-*` and `pb-safe-min-*`, and the live activity capsule rests at `lg:bottom-safe-min-4`.
16d. **An inset band holds background only.** Media, a scrim, a sheet's or a bar's glass may run under the status
    bar, the home indicator, a cutout or the 3-button bar, and says so with `data-edge-bleed`; text and every
    control inside it still clear the top and bottom insets, the side insets in landscape and the display's corner.
    Only media whose own words are part of the picture, a story's stickers, says `data-edge-bleed="media"`, and
    its words go where the author put them. `e2e/fixtures/edge-probe.ts`
    checks it in a page against every class in `e2e/fixtures/device-classes.ts`.
16e. **Landscape pays the sides.** Every full-screen row and rail pays the side insets through `*-safe-min-*`; an
    overlay that runs to the screen's sides takes `OVERLAY_INLINE_SAFE`, or `OVERLAY_INLINE_SAFE_AS_SHEET` where it
    is a sheet only below `sm`; a floating window rests at `--fw-start` and `--fw-end`, never closer than the side
    inset; and the tab bar's pill is never wider than the reading column's 560, so a turned phone's cutout and
    3-button bar stay clear of its glass. A phone on its side (`data-short-window`, a compact window shorter than
    480 at rest) sets `--page-header-row` to 2.75rem, still the 44pt tap minimum, and its sticky header slides
    away on transform while the keyboard is up, so the thread keeps its lines.
16f. **Scrolled content is judged where its scroller ends.** Content a scroller carries rests under the tab bar
    and the home indicator at the top of a feed; that is the page working. What must hold is that it can scroll
    clear: at the end of every scroller the last control and word sit above the bottom inset and above every
    pinned bar, and a strip's last item clears the side inset. A pinned box and a snapped page are always at rest,
    so they are judged at rest. `e2e/the-edge-probe-judges-scrolled-content.spec.ts` holds the probe to both verdicts.
17. **Three intents, three utility families.** The inset alone, the inset plus a step, and at least a
    step but never less than the inset. Picking the wrong one is what produced sixty-one hand-written
    spellings of the same three ideas.
18. **The nav reserve is applied once.** `NAV_RESERVE` is applied by `MainLayout`; a page inside it
    adds no bottom padding of its own, and a surface that opts out applies `NAV_RESERVE` rather than
    inventing a number.
19. **The full-height shell measures the visible viewport.** `h-app` reads `--app-vh`, which has the
    keyboard subtracted, falling back to a stable `100svh` before JavaScript runs. No CSS viewport unit
    tracks an iOS keyboard, so a `100svh` shell scrolls the composer away inside an installed app.

### Honest widths

20. **Every grid child that holds text or media carries `min-w-0`.** Without it the item measures its
    content width, and `document.scrollWidth` reports no overflow while a 1928px item sits in a 382px
    track.
21. **A box with a height and no width, whose children are all absolute, gets a width.** Otherwise it
    collapses to a sliver.
22. **Nothing scrolls the body horizontally.** A row of controls that cannot fit scrolls or overflows
    into a menu; it never wraps onto a second line on a phone. A strip is `ScrollStrip`, or `STRIP_SCROLLER`
    and `STRIP_ITEM`, plus `STRIP_GUTTER` on the page or `CARD_STRIP_BLEED` in a card, because a snap ignores
    padding and lands the first item on the edge. A control that belongs with a strip rides inside it
    (the period calendar is `ChoiceGroup`'s `trailing`); nothing sits beside a strip to cover its last item.
23. **`scrollWidth` is not an overflow signal here.** The `tap-target` pseudo-element and any
    descendant scroller inflate every ancestor. Measure the visible box or judge by screenshot.
24. **The scrollbar gutter is always reserved.** `scrollbar-gutter: stable` on `<html>`, so locking
    scroll for an overlay changes no width and no fixed element lurches sideways on open or close. A
    scroll lock therefore never also pads the body.

### The shell

25. **The rail is the navigation in regular, the tab bar in compact.** The rail is `hidden regular:flex`,
    fixed to the inline start, at `z-sticky`, `SIDEBAR_RAIL_WIDTH`: a column of glyphs at `lg`, and
    from `3xl` — a 16-inch screen, where words beside glyphs cost the content nothing — wide enough
    for every row to say what its glyph is. Below `3xl` the word is `sr-only` and a fine pointer
    reads it as a `Tooltip`; from `3xl` the tooltip is off, or it would repeat the word beside it. A device that
    cannot hover shows the glyphs alone, as the tab bar does, the label staying each one's accessible name: a caption
    under every glyph read as clutter. On a window too short for every row (an iPhone Duo's inner screen on its side)
    the rail's middle scrolls and the account stays pinned at its foot. The tab bar is `regular:hidden`, in two
    forms: a pill at `--mobile-nav-pill-h` on a normal page, a full-width bar at `--mobile-nav-overlay-h` over
    video. There is no drawer in regular. The server sends the rail and the right panel with the page unless the
    last visit left `surf-size-class` saying compact; the window then mounts only the chrome its size class uses, so
    a phone's taps never re-render a rail it cannot see.
26. **One width, four utilities.** `SIDEBAR_RAIL_WIDTH`, `SIDEBAR_RAIL_OFFSET` for an in-flow column,
    `SIDEBAR_RAIL_INSET` for a fixed full-viewport surface, and `SIDEBAR_RAIL_REMAINDER` for an
    in-flow box that breaks out of its column onto the whole area beside the rail. The offsets are
    logical, `ms-` and `start-`, so they follow the writing direction in RTL. A surface that writes
    the rail's own pixels keeps the `lg` rung at `3xl` and lands 92px under the 256px rail.
27. **The right panel belongs to the social pages.** Home, notifications, bookmarks, lists, trending,
    radar and every other social reading page carry it, so the column stands at the same place on every
    tab. The account pages go without: settings, monetization, premium, Sapphire, family, brand deals,
    business, identity checks and donate are one calm reading column centred in the canvas, because who
    to follow beside "Delete account" is clutter. `isAccountRoute(pathname)` is that list, and the tab bar
    reads the same one. So does every surface that owns the whole area beside the rail (a full-height shell, a wide
    grid, the shorts and map surfaces, a post's split, a capture or live stage). `isRightPanelRoute(pathname)`
    is that rule, read by the shell and by the pending ghost, which reserves the gutter the panel holds;
    `RIGHT_PANEL_WIDTH`
    and `RIGHT_PANEL_OFFSET` move in lockstep across `xl`,
    `2xl`, `3xl` and `4xl` or the feed and the panel overlap. Below `xl` a regular window has the rail and no panel,
    so the rail's account menu carries the panel's one door, Trending, there.
28. **The tab bar hides on deep utility pages**, the ones reached by a back header rather than a tab
    switch. `isBackHeaderRoute(pathname)` is that rule: every account page, the dashboard and the go-live
    stage, one registry with the right panel's, and it is social-only.
29. **Full height is keyed off the route, never the host.** `isFullHeightRoute(pathname)` decides, and
    the ghost that stands in for the route reads the same `pageColumn`, or the loading state is a
    different shape from the page.
30. **Studio, admin and messenger are modes of one shell.** `Sidebar` switches its items by app mode.
    There is never a second sidebar beside the rail.
31. **The side column is one token.** `SIDE_COLUMN` is the whole pane in one-pane layouts, 320 once list and
    detail stand side by side and 380 from `lg`; `SIDE_COLUMN_CHROME` is the border and glass it only wears once
    it is a column. The ghost reads the same token, so it cannot be a different width.
32. **Chrome persists across routes** and is `[navigation-and-history §31]`; the shell only fixes its widths. One
    `AppFrame` in `app/(main)/layout.tsx` draws every signed-in screen, a profile included, so opening one never
    tears the frame down or drops a minimised live stream; only a signed-out visitor on a profile reads the guest shell.

### Landscape, reach and resizing

33. **Landscape is the same layout with the landscape insets paid.** A phone in landscape is still the
    compact design; content clears the cutout with `ps-safe-min-*` and `pe-safe-min-*` `[layout §16e]`. Every
    full-screen capture and viewer answers the turn: the call stage and the full-screen video player with
    `landscape:`, the rest with `short:`, the variant on `data-short-window` (a phone on its side, read from
    the resting height so a keyboard never flips it): the story and highlight cards stand nearly full height,
    the live chat keeps to the start of the frame, go-live's form moves to the end, and the camera and image
    viewer chrome give the picture its height back.
34. **The thumb owns the bottom of the phone.** Primary navigation, the composer's send, a sheet's
    confirm and a drag handle sit in the bottom third. A destructive action never sits under the
    resting thumb, and the last item in a menu sheet is the one the thumb reaches first, so what
    deletes is last.
35. **A window resizes without a break.** Dragging from 1440 to 900 to 400 crosses `xl`, `lg`, `md`, the size
    class and `sm`, and at each boundary the column changes measure, the rail becomes the tab bar and the dialog
    becomes a sheet. Folding, unfolding, splitting and turning cross them the same way. Nothing unmounts, nothing
    scrolls horizontally and no state is lost; a stateful input is mounted once and moved by layout, never
    rendered twice and toggled by a breakpoint, or the half-typed text stays in the hidden copy.
36. **A pinned strip publishes its height.** `--top-strip-h` is set on `<html>` while the phone voice
    strip is up, and the header and main content read `TOP_STRIP_OFFSET` and `TOP_STRIP_RESERVE`
    rather than each guessing.
37. **A reserved slot is measured, not estimated.** `INLINE_COMPOSER_SLOT_HEIGHT` and
    `STORY_STRIP_MIN_HEIGHT` exist because the composer arrives in three steps after hydration and the
    story rail changes height between empty, loading and populated, and each step moved the feed.
38. **A panel's fixed rows never scroll away behind a list that grows.** The phone drawer pins
    Sapphire and Settings; the Spaces list, the one list the person can grow without bound, takes the
    height left and scrolls inside itself under `scroll-edge-fade`, and the body scrolls only when the
    list's floor of one and a half rows cannot fit.

39. **List and detail stand side by side in regular from `md`.** Messages, Surf Messenger and Spaces show the
    list at `SIDE_COLUMN` and the open item beside it at `regular:md`, with the empty state in the detail pane
    when nothing is open; selecting is a navigation, the chat header's Back hides, and crossing the boundary
    keeps the conversation mounted where it was, with its draft, caret and scroll. In compact and below `md`
    the list and the item are separate screens.

## The measures

| Measure | Rungs | Who uses it |
|---|---|---|
| `READING_COLUMN` | 560 base, 560 at `lg`, 600 at `xl`, 640 at `2xl`, 680 at `3xl`, 700 at `4xl` | Every page that is a single column: home, profile, post detail, notifications, settings |
| `WIDE_COLUMN` | 700 base, 760 at `lg`, 820 at `xl`, 860 at `2xl`, 1000 at `3xl`, 1120 at `4xl`, 1280 at `5xl` | Explore, premium, the circles dashboard, minigames settings |

| Shell measure | Value |
|---|---|
| `SIDEBAR_RAIL_WIDTH` | 72 in regular, 256 at `3xl` |
| `SIDEBAR_RAIL_REMAINDER` | `calc(100vw - 72px)` in regular, `calc(100vw - 16rem)` at `3xl` |
| `SIDE_COLUMN` | `w-full`, 320 at `regular:md`, 380 at `regular:lg` |
| `SIDE_RAIL_WIDTH` | `w-[300px]` |
| `RIGHT_PANEL_WIDTH` | 320 at `xl`, 368 at `2xl`, 392 at `3xl`, 416 at `4xl` |
| `FLOATING_MEDIA_PHONE_WIDTH` | `w-[42vw]` |
| `CHAT_MEDIA_WIDTH` | `w-[min(20rem,78vw)]` |
| `FLOATING_BAR_BOTTOM` | `--safe-bottom` + `--bottom-chrome-h` + 1.25rem; the chrome var is the nav pill in compact and `0` in regular, where the rail replaces it |

## Enforced by

- `__tests__/fitness/a-press-is-decided-by-the-pointer.test.ts`: rule 2, for a width hook read inside a
  handler or expression that decides a long-press, a right-click or `select-none`.
- `__tests__/fitness/a-way-in-survives-a-phone.test.ts`: rule 5, for a destination behind a viewport-gated
  shell, for the phone drawer carrying every rail destination, and for the rail reaching every right-panel
  destination in a regular window narrower than `lg`.
- `__tests__/fitness/a-stateful-input-is-mounted-once.test.ts`: rule 35, no file mounts a stateful input
  twice; `useMovedByLayout` moves the one instance, and `__tests__/hooks/use-moved-by-layout.test.tsx` proves
  the text survives the move.
- `__tests__/fitness/the-size-class-is-one-variant.test.ts`: rules 1, 25 and 39, no shell, list-and-detail,
  compose, swipe-back or phone-chrome question decided by a width, in the files it names and in every file that
  wears the phone's plate; one writer of the attribute and one `regular:` variant.
- `__tests__/device-fit/size-class.test.ts` and `__tests__/theme/boot-script.test.ts`: rule 1, every device
  class gets its design from `sizeClassOf`, and the boot script paints the same one.
- `__tests__/fitness/every-feature-has-a-way-in.test.ts`: rule 5 at the route level.
- `__tests__/fitness/a-route-skeleton-must-server-render.test.ts` and
  `__tests__/fitness/a-ghost-is-written-once.test.ts`: rule 29, since the ghost must branch the same
  way the page does.
- `__tests__/theme/stylesheet-token-gate.test.ts`: rules 13 and 16, through the single declaration of
  the header and tap-target tokens.
- `eslint local/no-off-scale-sheet-height`: rule 19's sibling, a hand-written `svh` height.
- `eslint local/no-physical-inline-axis`: rule 26, a physical `left`/`right` where the logical
  property is required.
- `__tests__/components/layout/home-drawer.test.tsx`: rule 38, the footer outside the drawer's scroller
  and the Spaces list scrolling inside its own.
- `e2e/the-rail-says-what-it-is.spec.ts`: rule 25, the rail's two widths in a real browser, the word
  on screen from `xl` with no tooltip repeating it, and the tooltip carrying it below.
- `e2e/the-desk-scales.spec.ts`: rules 25 and 26, by measuring the post detail's split pane and
  anything the rail hangs out beside its own words against the rail that is actually there, at 1024,
  1280, 1440, 1920 and 3840.
- `__tests__/fitness/an-edge-element-pays-its-inset.test.ts`: rule 16d, at the source: an element fixed to a
  screen edge pays a safe utility or is marked `data-edge-bleed`; its baseline is the inventory and only shrinks.
- `__tests__/fitness/an-inset-holds-at-every-width.test.ts`: rule 16c, a safe utility overridden by a bare
  value on the same property at a wider rung while the element still stands on the edge.
- `__tests__/fitness/a-side-inset-is-logical.test.ts`: rule 16a, the logical utilities, the right-to-left
  swap, and no logical side reading a physical inset anywhere in the source.
- `__tests__/fitness/the-rail-width-is-written-once.test.ts`: rule 26, at the source: no surface
  measures itself against the rail in the rail's own pixels.
- `__tests__/fitness/a-shell-centres-on-a-measure.test.ts`: rule 6, for the four files that centre a
  whole route — a literal there pins every page inside them to one rung.
- `__tests__/fitness/a-measure-answers-every-desk.test.ts`: rule 6, for the wide measure reaching the
  top of the scale, staying wider than the reading one, and dealing whole rows at every rung.
- `scripts/smoke-deploy.mjs`: rules 13 and 24 on production, by opening the tallest dialog and
  asserting its height and the core routes' paint.
- `__tests__/fitness/a-page-picks-a-measure.test.ts`: rule 6 one level below the shells, at every
  section under `app/(main)`; the two explore panels that pin themselves to a rung are its baseline.
- `__tests__/fitness/a-grid-child-declares-its-width.test.ts`: rule 20, for the track item that holds
  text or media and carries no `min-w-0`, so it measures its content instead of its column.
- `__tests__/fitness/a-control-row-scrolls-rather-than-wraps.test.ts`: rule 22, for a row of three or
  more controls that wraps unconditionally and offers no scroller.
- `__tests__/fitness/a-full-screen-surface-answers-landscape.test.ts`: rules 16e and 33, for a full-screen
  surface that captures or plays a frame, through the layout modules it draws with: an answer to the turn,
  and the side insets paid. Both baselines are empty.

**Not yet enforced**

- Rule 9: `a-page-gutter-is-one-token` would fail any page container that writes its own `px-*`
  rather than reading `PAGE_GUTTER`, and any bleed written as a literal rather than `PAGE_BLEED`.
- Rule 35: nothing drives a resize, and no source-level read can. A Playwright sweep that walks 1440
  to 375 in steps and asserts no horizontal document overflow and no remount at each boundary would
  cover it.

## Traps

- Rule 29: full-height layouts keyed off `window.location.host` gave every studio room the 560px
  reading column on the social domain. Key off the route.
- Rule 20: a grid item without `min-w-0` measured 1928px in a 382px track while the document reported
  no overflow at all.
- Rule 23: `tap-target`'s 44px `::after` and any descendant scroller inflate every ancestor's
  `scrollWidth`, so eighteen routes that read as overflowing at 375px were clean.
- Rule 24: without a stable gutter, a scroll lock widened the viewport by the scrollbar and every
  fixed element lurched sideways on open and again on close.
- Rule 21: a card with a height and no width, whose children were all absolute, collapsed to a sliver.
- Rule 6: a literal `max-w-[760px]` pinned only the `lg` rung and drifted from the column at `xl` and
  above; the hand-written image `sizes` beside it asked for 700px inside a 560px card.
- Rule 12: `overflow-x: hidden` on the page wrapper creates a scroll container and breaks every sticky
  page header. Use `overflow-x-clip`.
- Rule 37: the shorts feed reserved 4rem for a bar that renders at 4.25rem, so the last 4px of every
  short, where the scrub bar lives, sat underneath it.
- Rule 26: the post detail's split pane wrote `calc(100vw - 72px)` and centred on the parent, so from
  1280 to 1596 it was up to 184px wider than the area it stood in — half of that under the rail, half
  clipped off the end — and `left-1/2` centred it 134px off in Arabic, because a relative box wider
  than its parent starts at the parent's inline END under `dir=rtl`.
