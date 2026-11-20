# Traps

Every line here is a mechanism that reached production once. The browser, the framework or the platform
behaved exactly as documented, and the documented behaviour was the surprise. This document exists so nobody
rediscovers them: it is the index a person opens when something looks wrong on a screen and the cause is not
in the code they are reading. It protects the feeling that the product is finished, because each of these was
invisible in review and obvious to a person holding a phone.

## Owns and defers

Owns: the one-line mechanism behind each experience failure, and the pointer to the rule that prevents it.

Defers: the rule itself, always, to the document that owns it. Traps whose rule lives in `[motion]` are
recorded in that document under their rule number, because their mechanism is only legible beside the rule.
Traps that no person can see, in the build, the type gate, the network edge, the query cache or a test
harness, live in `docs/engineering-traps.md`.

## Rules

1. **A trap is a mechanism, not a story.** One line: what the platform actually does, and the consequence.
   No dates, no incident, no before and after.
2. **A trap points at a rule, never replaces one.** If no rule prevents it, the rule is missing and is written
   in the owning document in the same commit.
3. **A trap earns its line by having fooled a reviewer.** Something caught in normal review is a bug. A trap
   is something that looked correct, passed the suite, and still shipped wrong.

## Tokens and values

- **A `:root` custom property cannot read a variable declared on a panel.** It computes to nothing at the root
  and every consumer silently loses the value; this is how every desktop dialog lost its height ceiling.
  Declare and read at the same level. `[tokens]`
- **`--canvas` is one colour and the canvas is not.** Six themes lay a ramp or a field over it, so a band of
  `--canvas` under a pinned control is seamless in Light and Dark and a dark slab everywhere else.
  `[colour-and-effects]`
- **`cn()` drops a custom `text-*` size it was never taught.** The count badge lost its 11px this way; use the
  token, never a one-off size. `[type]`

## Layout and shape

- **`tap-target`'s 44px `::after` covers the wrapper's own children.** Taps on the name and the location
  inside an author pill landed on the pill. It goes on a real control, never a container. `[layout]`
- **A grid item without `min-w-0` measures its content width.** A 1928px item sat in a 382px track while the
  document reported no overflow at all. `[layout]`
- **A card with a height and no width, whose children are all absolute, collapses to a sliver.** `[layout]`
- **A full-height layout keyed off the hostname gives a route the wrong column.** The studio inherited the
  600px reading column that way. Key off the route. `[layout]`
- **`display: inline-flex` on a name collapses the handle onto it with no gap.** A name and its handle stack
  in a flex column. `[type]`
- **`<table className="text-start">` does not beat the UA sheet's `th { text-align: center }`.** Style the
  cell, not the table. `[craft]`
- **A height query flips with the Android keyboard.** The page shrinks for it, so a design chosen by height
  changes mid-typing on a tablet in landscape; the size class reads the resting height and holds. `[layout]`
- **A safe utility overridden at `md:` drops the inset on a foldable or a tablet.** Both have bars at that
  width. `[layout]`
- **`ps-safe` reading `--safe-left` pads the wrong edge in Arabic.** `[layout]`
- **`regular:` sorts after every width rung.** A `regular:` value beside an `lg:` one on the same property wins
  on a desktop too, so a wider rung inside it is written `regular:lg:`. `[layout]`
- **The rail arrives at 600 and `lg` at 1024.** Between them a bar hidden by `lg:hidden` is still on screen: a
  `fixed inset-x-0` header ran under the rail's mark and its compose button on every tablet upright. `[layout]`
- **Two copies of a control sharing one ref leave it on the last one mounted.** The short's rail is drawn on the
  picture and beside it, one hidden, so a menu anchored to the ref opened at the hidden copy's zero rect. `[layout]`
- **A tooltip is the only word on a touch rail.** A finger never hovers, so a glyph-only rail reads as eight
  unlabelled icons on an iPad. `[layout]`
- **CSS-toggled twin composers hold two drafts.** Turning an iPad across `lg` swapped the visible one for its
  empty twin. `[layout]`
- **`(orientation: landscape)` flips on unfold and split-resize.** A turn is the screen's angle. `[platform]`
- **The iOS keyboard leaves the bottom `env()` at 34.** A composer kept a home-indicator band above it. `[layout]`

## Components and overlays

- **`next/dynamic` renders its `loading:` fallback even at `open={false}`.** A lazily imported sheet flashed a
  modal on every page load. A lazy overlay mounts nothing until its first open. `[components]`
- **A tab trigger does not inherit its strip's variant unless the strip passes it.** One trigger in a
  differently-styled strip is how a family drifts. `[components]`
- **A global Escape or key listener mounted in a layout owns that key on every page.** The live stage owned
  Escape with no session, so Escape on any dialog also left the page. A listener exists only while its surface
  is on screen. `[navigation-and-history]`
- **`<Tabs defaultValue>` eats every deep link.** The default wins over the URL on first render. Control the
  tab from the URL where the strip lives. `[navigation-and-history]`
- **A phone-only `history.back()` on menu close undoes the navigation the menu just made.** A sheet or drawer
  never buries its own history entry. `[navigation-and-history]`
- **A panel placed on the frame a fold starts opening is placed at the folded height.** `grid-template-rows`
  animates, so the short's More measured itself shut and then grew 80px past the bottom of the window.
  `[components]`
- **A flex row's glyph shrinks before its `nowrap` label does.** A speed glyph beside a label, a rate and a caret
  drew 4px wide in a 224px menu. `[components]`
- **`replaceState` written with Next's own state object skips the `useSearchParams` sync,** so the URL changes
  and nothing reading it updates. Carry the framework's state through. `[navigation-and-history]`
- **A fold held in component state is shut again by the remount a Back causes,** and the offset being restored
  is past the end of the page it shut. A fold whose rows lead away lives in the address. `[navigation-and-history]`

## Loading and first paint

- **A lazy fallback that does not match the final shape reads as a glitch.** A centred dialog standing in for
  an anchored menu was the entire perceived flash. `[loading-and-states]`
- **A skeleton with rows under a sheet whose body is pure UI shows bars where a form will be.** Reserve the
  shape, not a list. `[loading-and-states]`
- **A skeleton's outer margin collapses out to `<body>` and moves the whole document.** Pad the container
  instead. `[loading-and-states]`
- **A shimmer band sized as a percentage of its own bone sweeps at a different speed on every bone.** The band
  is an absolute width. `[loading-and-states]`
- **An app frame imported with `ssr: false` ships initial HTML with no nav, no aside and no composer.** The
  frame is server-rendered. `[loading-and-states]`
- **Hydration throws the server-painted list away and refetches** unless the server's rows are handed to the
  client as placeholder data. `[loading-and-states]`
- **A refusal thrown out of a server render reaches the browser as a minified React internal.** The message
  is withheld in production, so a suspended account's refused write and a stranger's refused staff read both
  read as one unnamable fault with a reference. Resolve a refusal where it is read. `[loading-and-states]`
- **A page that arrives by fading from opacity 0 paints a blank frame first.** On a throttled phone that frame
  is the whole screen under the nav, and the 200ms fade reads as the tap's delay. `[motion]`
- **A `loading.tsx` over a page with nothing to fetch still paints its ghost first.** React swaps the streamed
  page in on an animation frame the page's own scripts hold, so `/login` painted its form at hydration.
  `[loading-and-states]`
- **One deadline over the widget's answer and the person's answer tears a solvable challenge off the screen.**
  On production the Turnstile challenge appeared fourteen seconds in and was removed at twenty, so a person had
  six seconds to find a checkbox before sign-in said the check had not loaded. `[loading-and-states]`

## Media

- **A muted-autoplay retry cannot tell its own pause `AbortError` from a policy refusal,** so it restarts
  players that have already scrolled off screen: fifteen reels decoding at once, 29fps. Distinguish the abort
  you caused from the one you were given. `[media]`
- **An avatar requested at 2x on a DPR-2.6 phone is upscaled.** Request the density the element will actually
  be drawn at. `[media]`
- **A recorded blob's MIME type carries a `;codecs=` suffix,** which matches neither the client nor the edge
  allow-list, and every voice post fails validation. Compare the type, not the string. `[media]`
- **Seeking or re-sourcing the one element on screen is a freeze.** The stage holds the last frame until the
  decoder lands, and a new `src` blanks it and shows that file's first frame first. A cut swaps to an element
  already resting on the frame. `[media]`
- **A playing element presents about a frame ahead of the `currentTime` it reports,** so a cut timed on it
  shows a frame past the out-point. `[media]`
- **A `VideoEncoder` can describe its own output with a header no decoder reads,** and the muxer writes that
  description into `avcC` verbatim: zeroed reserved bits and a repeated NAL header byte published four shorts
  that never decoded a frame. Check the record before muxing it. `[media]`
- **Native video is drawn behind the webview,** so a background anywhere on the path to its slot, or the page
  painted under the call overlay, hides it while the call plays on. `[shells §19]`

- **WKWebView's own photo picker hands over Compatible copies:** a HEIC arrives as a JPEG and an HEVC clip as
  H.264, possibly smaller. `[media]`
- **An `<input capture>` on iOS records at medium quality,** and its photo is a JPEG at 0.8 with no EXIF. `[media]`
- **`takePhoto` with no size returns the smallest size on WebKit and the first-listed on Chromium,** never the
  largest the camera has. `[media]`
- **The Android WebView denies the pan-tilt-zoom permission,** so `zoom` never appears and every zoom is a crop of
  the frame the track delivers. `[media]`
- **WebKit on iOS ends the first camera's track when a second opens,** while listing both: dual camera offered
  there failed on every iPhone. It is hidden on iOS (`canHoldTwoCameras`). `[media]`

## Interaction

- **A pending signal read from inside the control it lights unmounts itself.** The tab bar swapped its link for
  an active-tab button the moment the link's `useLinkStatus` turned pending, the link's effect cleaned up, and
  no tab lit before the route committed. The control that reports pending stays mounted through it.
  `[loading-and-states]`
- **A native date picker opens on today.** A birthday is decades of wheel or calendar away from it, and
  `type="date"` raises no keypad to type one instead. A date of birth is typed. `[input-and-forms]`
- **A `pointerdown` that prevents default does not stop the `click` that follows.** A stage's own click
  cleared the selection the tap had just made. Decide on one event. `[feedback]`
- **`word-break: break-all` on one run of a caret overlay moves every glyph after it.** The composer painted a
  link with the reading link's `break-all` while its textarea wrapped by `overflow-wrap: anywhere`, so the URL
  broke mid-line under a field that had moved it whole to the next line. `[input-and-forms]`

## Speed

- **`<Link>` prefetches when it scrolls into the viewport, not on hover.** A nav rail, a tab bar and a right
  panel together pulled forty routes in during boot. `[speed]`
- **The router answers the destination in the first frame of a navigation, and paints it in the last.**
  `usePathname` and `useLinkStatus` both moved on ~13ms after a press that took 500ms to land, so the route
  ghost cancelled itself before its own grace had run and every navigation in the app showed a frozen page
  instead. What is still in flight is a transition this app holds, not anything the framework reports.
  `[loading-and-states]`
- **A frame with a suspended destination under it cannot re-render.** The ghost's reveal was a `setState` in
  the layout that owns the page slot; React took 300ms to commit it, by which time the wait was over. Anything
  that has to appear DURING a navigation is in the commit the press already made, revealed by CSS.
  `[loading-and-states]`
- **A lazy sheet plays its entrance twice.** The `loading:` fallback and the sheet that replaces it are both
  `<DialogContent>`s, so the panel rose, settled, and rose again when the chunk landed. The one taking over
  enters at rest. `[motion]`
- **A hook on a feed row is a request per row.** Every video in the feed asked for its own chapter list on
  mount — 2,400 reads a day for a control that only exists once the video is playing. `[speed]`
- **A CSS animation on a `display: contents` box does not run everywhere.** The navigation ghost and the page
  it replaces were swapped by two `visibility` animations, one on each; where the engine skipped them the page
  being left never hid and the two rendered on top of each other. A box that generates no box is flipped by
  inherited `visibility` off a class, never by an animation. `[loading-and-states]`
- **A custom property only JavaScript ever sets has no value on the first frame.** `var(--nav-ghost-grace)`
  with no fallback made the whole `animation` shorthand invalid until a layout effect ran, so the swap that
  depended on it never started. `[loading-and-states]`
- **A pending destination must not reshape the page it is leaving.** The shell took the destination's column
  and full-height branch from the press, while the page being left was still the page on screen for the whole
  grace: the feed stretched from 640px to 860px on the way to `/explore`, and to the whole 1368px beside the
  rail, clipped to the viewport, on the way to `/messages`. The frame is the route in flow; the ghost brings
  its own. `[layout]`
- **A store a press writes is a synchronous render of everything that reads it.** A zustand `set` reaches
  React through `useSyncExternalStore`, which renders on the urgent lane inside the click task, so the tab
  bar, the CSS-hidden rail and the whole layout were rendered before the first frame could paint, and the
  circle waited behind them. The press now writes the store, the two tabs it concerns select their own answer,
  the ghost arrives on the transition lane, and the push starts after the frame has painted. `[speed]`
- **`next/dynamic` suspends a chunk that has already landed.** Its lazy component is only initialised by
  its first render, and React then holds a fallback it has just shown for 300ms, so a warmed create deck
  still opened on a blank ghost. `[navigation-and-history]`
- **A progress line keyed to a phase says nothing for as long as the phase lasts.** An auto-edit read
  "Watching your footage" for over a minute while every read beneath it moved; the line is read from the reads
  and the stages themselves. `[ai]`
- **A grace tuned to a warm push ghosts every cold one.** At 120ms an ordinary tap — anything not already
  prefetched, which on a phone is most of them — threw a full-route skeleton up and tore it down again, which
  reads as a page glitching rather than loading. `[loading-and-states]`

## Enforced by

A trap is prevented by the rule it cites, and enforced by whatever that document's own **Enforced by** table
names. This document adds nothing of its own to the suite.

**Not yet enforced.** `every-trap-cites-a-living-rule`: every `[doc]` reference here resolves to a document in
`design-docs/`, and every document referenced still contains a rule on that subject.
