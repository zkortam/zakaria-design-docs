# Navigation and history

This document owns where the person is, how they got there, and how they get back: the URL of every
state, what Back leaves, what Escape closes, how a tab and a scroll offset survive a return, how a deep
link lands, and the keyboard map on desktop. The feeling it protects is that the product remembers.
Back always means one step, never two and never none; refreshing lands on the same view; returning to a
feed lands on the row that was under the thumb; and the chrome the person is holding never blinks.

## Owns and defers

Owns: state in the URL; Back and the history entry; Escape and key ownership; scroll restoration; the
active-tab gesture; swipe-back's contract; the desktop shortcut map; chrome persistence across routes;
warming a route on intent.

Defers: the page transition and swipe physics to `[motion]`; navigation budgets, prefetch cost and the
route ghost to `[speed]`; the route ghost's shape to `[loading-and-states]`; the `<Tabs>` primitive and
the overlay family to `[components]`; focus restore inside an overlay to `[input-and-forms]`; arrival on
a cold start and the install path to `[arrival]`; RTL gesture mirroring to `[internationalisation]`;
keyboard operability and focus order to `[accessibility]`.

## Rules

### Every state has a URL

1. **A state a person can be sent back to has a URL.** The route, the active tab, the open item, the
   filter and the query live in the location. Refresh lands on the same view. A fold whose rows lead to
   other pages is such a state: the settings index writes each of its folds through `useTabParam` under
   `SETTINGS_FOLD_QUERY`, so the entry Back returns to says the fold is open and the page mounts that way.

2. **A deep link lands directly.** The tab, item or offset named in the URL is the first thing painted.
   No default is applied first and corrected after.

3. **A tab is a search param, written with `replaceState`.** `useTabParam` is the one owner: it reads
   the param, falls back once, deletes the param when the fallback is chosen, and writes through
   `carriedHistoryState()`. `router.replace` is not used, because it re-renders the route on the server
   before the strip can paint its own selection.

4. **A tab change is not a history entry.** Ten taps across a strip leave one entry, so Back leaves the
   page rather than walking the strip backwards.

5. **History state has one owner.** `carriedHistoryState()` for a rewrite that changes the URL,
   `markedHistoryState()` for an entry whose URL does not move. Nothing else reads `history.state`.

6. **An overlay is not a URL.** A dialog, sheet or menu does not change the address; it owns one
   phantom history entry instead, so Back closes it and a refresh lands on the page beneath it.

6a. **A gate never answers the router with a redirect.** A verdict about the account (sign in, verify,
   onboard, the enforcement door) changes the moment the person acts, so it is never kept where a later
   navigation can replay it. The proxy answers a router fetch it would redirect with an empty 204, and the
   router loads the document, which meets the 307 fresh; finishing onboarding then lands on home.

### Back leaves one thing

7. **Back leaves exactly one thing:** the topmost open overlay if there is one, otherwise the page.
   Never two, never none.

8. **One overlay, one entry.** `useOverlayHistoryEntry` is the one owner — `Dialog` takes its entry from it
   like every non-dialog surface — pushing a single phantom entry carrying `markedHistoryState()` plus
   `dialog: true`, and popping it on close. A phantom that carried a bare `{dialog:true}` drops the router's
   own tree and Back reloads the document on it. A host that answers Back with a prompt instead of closing
   gets a fresh entry: that press already spent the old one, and without a replacement the next Back left the
   page out from under a composer that was still open.

9. **No close calls `history.back()` on its own.** A menu, drawer or sheet that walks history back on
   close undoes the navigation the tap just made.

10. **Back never spends a press on nothing.** A phantom buried by a navigation is a duplicate of the
    entry behind it, so landing on one continues back rather than leaving the person where they already
    were.

11. **A navigation out of an open overlay waits for its entry.** `closeOverlayThenNavigate` is the only
    correct order; doing it by hand races the phantom's pop against the push.

12. **Back is `useBackNavigation`, never `router.back()`.** `hasInAppHistory()` decides, and a deep link
    or cold PWA start falls back to the surface's own landing route rather than ejecting the person to
    whatever was in the tab before.

13. **Back answers in the frame it is pressed.** `announceBack` names the page behind as the pending
    destination, as a forward tap does, so the tab bar and the ghost answer before the router commits; the
    popstate listener in `HistoryDepthTracker` does the same for a traversal nobody here started. A kept root
    (rule 24a) is shown at once. `pathBehind` is how Back knows where it leads: the entries cannot be read, so
    each stamped one is remembered with its route. `[speed]` holds the budget.

### Escape closes the topmost only

14. **Escape closes one surface, the frontmost.** `useEscapeKey` is the owner; a nested overlay claims
    the key with `capture` and `stopPropagation` so the sheet underneath is not closed in the same press.

15. **A key listener exists only while its surface is on screen.** A handler mounted in a layout owns the
    key on every page, which is how Escape on a dialog also walked the page back.

16. **A surface with a dialog above it takes no key.** A handler calls `hasDialogAbove()` before acting,
    so the editor behind a picker does not answer for it. A `nonModal` sheet is excluded: the page under
    one stays live and keeps its shortcuts.

17. **Escape inside a field returns to the field's surface, not past it.** A focused editing target blurs
    first; the next Escape closes the surface.

### Scroll

18. **Back restores the row you left.** `useScrollRestoration(key)` records while scrolling, not on
    unmount: the offset on every frame, which costs nothing, and the first row marked `scrollAnchor(id)` with
    its distance from the top once the scrolling has stopped, or the moment a navigation or a traversal begins.
    The row is found by bisection, so the reads do not grow with the rows above it. On return it puts that row at
    that distance in the commit that mounts the page, and keeps it there while the document is still changing
    size (a `ResizeObserver`, so a frame in which nothing changed reads nothing) until it has held for
    `SETTLED_MS`, because the list is short on the first frame and the cards and media above the row keep settling
    after it; a pixel offset placed once landed 230px off. A place the page cannot reach yet is held for up to
    `HOLD_MAX_MS`, one idle slice, so a list still mounting its rows is waited for rather than abandoned at its
    end. Nothing is recorded while a return is being placed, so a list remounted mid-return still goes back to the
    row that was left. A page with no anchored row holds the offset the same way. `FeedList` marks every row.
    `history.scrollRestoration` is `manual`, set once by `HistoryDepthTracker`: the browser's own restore ran on
    its schedule against this one.

18a. **The list you return to is the list you left.** Nothing a return mounts may re-key it: the For You
    weights are one published snapshot per session (`useForYouWeights`), moved only by the model landing,
    a reset or a refresh, never by a remount sampling what reading has taught it since.

19. **Only a pop restores.** A fresh push lands at the top; `ScrollResetOnNavigate` does the reset in a
    layout effect so the incoming route never paints at the outgoing route's offset. That includes a push made
    from inside a sheet or the drawer: the scroll lock pins the body, where a reset moves nothing, so its release
    puts the page that arrived at the top (or at a kept root's own offset) and only puts back the page it found
    when the path did not change.

20. **A restore is not a scroll.** It calls `suppressScrollDirection(SCROLL_LOCK_SETTLE_MS)`, or the
    header and tab bar read it as scrolling down and hide themselves on arrival.

21. **The person wins.** A wheel, touch or key during a restore stops it immediately.

22. **A list that will restore mounts a window, not every row.** `restoringAnchor(key)` names the row the
    return is about to put at the top. `FeedList` mounts the `FEED_RESTORE_ROWS_AROUND` rows either side of it
    and holds every other row as a `RowWindow` box of the height it last measured, so the offset is reachable
    on the first frame and the rest mounts as the scroll comes near, in a transition. `willRestoreScroll(key)`
    still tells a staged list to skip its short first pass.

23. **Every scrolling route restores.** Every feed through `FeedList`, plus explore, trending,
    notifications and profiles, restore by a `useScrollRestoration` call of their own. A route that restores
    nothing of its own is restored all the same: `ScrollResetOnNavigate` records every route's offset under
    `ROUTE_SCROLL_KEY_PREFIX` and puts it back on a pop, standing down when a page on screen restores by a row.
    It places by a row wherever the page marks one: the settings index marks its folds with `scrollAnchor`, so a
    return holds the fold where it was however the rows above it settle.

24. **A tab switch keeps its offset.** Returning to a tab lands where it was left. `(not yet built)`:
    `useTabParam` writes with `replaceState`, so no pop occurs and `useScrollRestoration` never fires.
    This needs a per-tab key recorded on change and restored on the way back, inside `useTabParam` so
    every strip inherits it.

24a. **A tab root is kept, not remounted.** Home, Explore, Notifications and, on a phone, the messenger
    inbox stay mounted under React `<Activity>` once navigation has reached them (`KeepAliveRoot` hands the
    page, or the messenger's layout, to `KeptRoots`), at most `KEEP_ALIVE_MAX_HIDDEN` of them hidden. A
    tab switch or a Back to one is a reveal: its state, rows and scroll offset are where they were, its
    effects are off while it is hidden, the media inside it is paused, and `useKeepAliveScroll` swaps the window
    offset in the commit that swaps the roots. The destination a press names (`useOptimisticPathname`) is shown at
    once, with no ghost. The root the document was served with renders in place, where its HTML hydrates, and
    is kept from its next visit. A different account drops every root. Shorts is kept per feed (for you, following,
    a profile's, a tag's: `keptKeyOf`), and only when the link names no position in it; a parked Shorts keeps its
    place and its pictures and mounts no player, so it holds no decoder. A parked root is `inert` and
    `aria-hidden`, written to the DOM as it is hidden, because a hidden tree is the last to render.

### The tab bar and the rail

25. **Tap the active tab to go to the top, again to refresh.** `MobileNav` scrolls smoothly to 0 while
    `scrollY > 0`, then dispatches `CUSTOM_EVENTS.feedRefresh` for a refreshable destination.

26. **The desktop rail does the same.** `(not yet built)`: `Sidebar` has no active-tap handler, so the
    same gesture on the same product does nothing with a mouse. It needs `MobileNav`'s handler lifted
    into a shared hook both call.

27. **A refresh is one event, not a per-page hook.** Every refreshable route listens for
    `CUSTOM_EVENTS.feedRefresh`; nothing reaches into another page's refetch.

27a. **One way to say "you are here".** The active tab carries the travelling fill, the Profile tab
    included, and every glyph stays `regular`; on the desktop rail your profile is the account menu's
    first entry and the avatar lights while you are on it, and the messenger's rail and bar read `MESSENGER_DESTINATIONS` in one order. Off `/home`, the Home tab
    carries `NotificationUnreadDot` while notifications wait, from the bell's own count, so reading them
    clears both.

### Gestures

28. **Swipe-back follows the finger.** `useSwipeBack` tracks 1:1 from `SWIPE_BACK_EDGE_ZONE` (24px) at the
    inline-start edge `[internationalisation §21]`, locks
    horizontally past `DRAG_THRESHOLD_PX` (8px), commits past `SWIPE_BACK_THRESHOLD` (80px) and goes
    through `useBackNavigation`. The travel is written to the page itself, never to React state, over the
    ghost of the page it uncovers (`SwipeUnderlay`, named by `pathBehind`), so the finger never slides a page off
    a blank canvas. Compact only: in the regular size class the header's Back stays and the edge is the
    platform's. `[motion]` owns the release.

29. **The platform's own edge gesture is never overridden** by an app gesture on the same edge. An in-app
    horizontal drag (a section swipe, swipe to reply) starts outside `backEdgeZone`, the wider of
    `SWIPE_BACK_EDGE_ZONE` and the `--gesture-left`/`--gesture-right` an Android shell reports from the person's
    back sensitivity.

30. **A swipe-back that lands on the same pathname still comes home.** A query-only entry fires no route
    commit, so the page is released after `COMMIT_RELEASE_MS` rather than left off-screen.

### Chrome and warming

31. **Chrome never remounts across routes.** The rail, the tab bar, the right panel, the floating player
    and the live stage persist through navigation; only the page content changes.

32. **Nothing prefetches on viewport entry** is `[speed §8]`; a route warms on intent, below.

33. **A route warms on intent.** `useRouteWarm` returns `onPointerEnter`, `onPointerDown` and `onFocus`
    handlers and warms once per href, `usePreloadOnIntent` does the same for a lazy panel, and both stand
    down under `shouldConserveData()`.

34. **A tap never waits on a chunk.** The destination's ghost is already server-rendered and its chunk
    was warmed on the gesture that preceded the tap. An overlay the tap must answer in its own frame reads
    its warmed chunk through `warmChunk` and `use`, as the phone's create deck does; `lazyFromChunk` is that
    pair as a component. One that ships no fallback of its own is also warmed on idle by `usePreloadOnIdle`.

### Keyboard

35. **A shortcut never fires while the person is typing.** Every handler returns early for an `INPUT`,
    `TEXTAREA` or `contentEditable` target; `isTypingTarget` is the shared predicate.

36. **A shortcut binds on every pointer.** `useMessengerShortcuts` and `useEditorKeyboard` bind wherever a
    keyboard may be, because an iPad with a keyboard and no trackpad has no fine pointer; the typing guard
    (rule 35) and the accelerator guard (rule 38) are what protect a field and the browser. Nothing a touch
    device can reach depends on a key.

37. **A surface with more than two shortcuts publishes its roster.** `MESSENGER_SHORTCUTS` and
    `EDITOR_SHORTCUTS` are each one array that both the binding and `ShortcutsSheet` read, so what the sheet
    lists is what the key does.

38. **A shortcut leaves the browser's accelerators alone.** Anything with a modifier that the browser
    already owns is the browser's, and a handler that sees an unclaimed modifier returns rather than
    guessing.

39. **A document-level handler checks it owns the focus.** A listener on `document` that reads a field's
    key confirms `document.activeElement` is its own field first, or it swallows that key for the page.

## The desktop shortcut map

Every key the product claims, and the surface that must be on screen for it to fire.

| Keys | Does | Surface | Owner |
|---|---|---|---|
| `Escape` | Closes the frontmost overlay | Any overlay | `useEscapeKey`, `Dialog` |
| `/` | Focuses the search field | A page with one | `useSearchKeyboardShortcut` |
| Volume up, volume down | Records sound intent for autoplay | App frame | `useSoundIntent` |
| `Mod`+`K` | Opens the conversation switcher | Messenger | `MESSENGER_SHORTCUTS` |
| `Alt`+`↑` / `Alt`+`↓` | Previous / next conversation | Messenger | `MESSENGER_SHORTCUTS` |
| `Mod`+`F` | Searches inside the thread | Messenger | `MESSENGER_SHORTCUTS` |
| `Mod`+`Shift`+`U` | Marks the thread unread | Messenger | `MESSENGER_SHORTCUTS` |
| `Mod`+`/` | Shows the shortcut sheet | Messenger | `MESSENGER_SHORTCUTS` |
| `Mod`+`↑` / `Mod`+`↓`, `Enter`, `Shift`+`Enter` | Previous / next search hit | Thread search bar | `components/message/search-nav-bar.tsx` |
| `Mod`+`Enter` | Submits the composer | Any composer | `useComposeKeyboard` |
| `ArrowRight` (`ArrowLeft` in RTL) at the end, `Escape` | Takes the ghost text, puts it away | A composer showing ghost text | `GhostText` |
| `Ctrl`+`Space` | Opens Rewrite | Any composer field | `useRewriteShortcut` |
| `↑` / `↓`, `Enter`, `Escape` | Walks, accepts, closes the @/# list | A composer with one open | `useAutocompleteKeys` |
| `←` / `→` | Previous / next item | Image viewer, post carousel, highlight viewer | each component |
| `←` `↑` / `→` `↓`, `Escape` | Previous / next story, close | Story viewer | `useStoryGestures` |
| `↑` / `↓` | Pages the feed | Shorts | `components/shorts/shorts-feed.tsx` |
| `Space` | Play or pause | Shorts player | `components/shorts/shorts-player.tsx` |
| `Space` `k`, `m`, `←` / `→`, `f` | Play, mute, ±5s, fullscreen | Fullscreen video | `components/media/fullscreen-video-player.tsx` |
| `Space` | Play or pause | Shorts editor | `EDITOR_SHORTCUTS` |
| `j` / `k` / `l` | Shuttle backward, stop, shuttle forward; each repeat is faster | Shorts editor | `EDITOR_SHORTCUTS` |
| `Home` / `End` | First frame, last frame | Shorts editor | `EDITOR_SHORTCUTS` |
| `i` / `o`, `Mod`+`Shift`+`X` | Mark in, mark out, clear both | Shorts editor | `EDITOR_SHORTCUTS` |
| `Shift`+`L` | Loops playback over the marks, or the whole edit | Shorts editor | `EDITOR_SHORTCUTS` |
| `↑` / `↓` | Previous / next cut on any lane | Shorts editor | `EDITOR_SHORTCUTS` |
| `Mod`+`Z`, `Mod`+`Shift`+`Z`, `Ctrl`+`Y` | Undo, redo | Shorts editor, story canvas | `EDITOR_SHORTCUTS`, `useCanvasShortcuts` |
| `Mod`+`C` / `Mod`+`V` | Copy, paste the selected clip | Shorts editor | `EDITOR_SHORTCUTS` |
| `Mod`+`D` | Duplicates the selected clip; the browser keeps it when nothing is selected | Shorts editor | `EDITOR_SHORTCUTS` |
| `=` `+` / `-` `_`, `Shift`+`Z` | Zoom the timeline, zoom it to fit | Shorts editor | `EDITOR_SHORTCUTS` |
| `s` | Splits the selected clip | Shorts editor | `EDITOR_SHORTCUTS` |
| `←` / `→`, `Mod`+`←` / `→`, `Alt`+`←` / `→` | Step a frame, seek, step the clip (a video clip moves one place in the sequence, an overlay one frame) | Shorts editor | `EDITOR_SHORTCUTS` |
| `Delete` `Backspace` | Deletes the selection | Shorts editor, story canvas | `EDITOR_SHORTCUTS`, `useCanvasShortcuts` |
| `Escape` | Clears the selection | Shorts editor | `EDITOR_SHORTCUTS` |
| `Mod`+`/` | Shows the shortcut sheet | Shorts editor | `EDITOR_SHORTCUTS` |
| `t` `s` `d` `b` `f` `m` `g` | Picks a tool | Story canvas | `useCanvasShortcuts` |
| `j` `↓` / `k` `↑`, `Enter`, `x`, `Delete` | Moves, opens, selects, acts | Admin lists | `lib/hooks/admin/use-keyboard-list.ts` |

## Enforced by

- `__tests__/fitness/history-state-has-one-owner.test.ts` covers 5.
- `__tests__/supabase/onboarding-gate-one-round-trip.test.ts`, `__tests__/supabase/enforcement-door.test.ts`
  and `e2e/onboarding.spec.ts` ("a signup finished within minutes of signing in") cover 6a.
- `__tests__/fitness/a-tab-lives-where-its-strip-does.test.ts` covers 2 and 3.
- `__tests__/fitness/an-overlay-inherits-its-behaviour.test.ts` covers 6, 8 and 14.
- `__tests__/fitness/no-close-walks-history-back.test.ts` covers 9.
- `__tests__/fitness/a-navigation-waits-for-the-overlay-it-leaves.test.ts` covers 11: a handler that closes an overlay and navigates
  goes through `closeOverlayThenNavigate`, and a menu item that leaves the page declares `href`.
- `__tests__/fitness/back-goes-through-one-primitive.test.ts` covers 12.
- `__tests__/fitness/a-key-listener-outlives-nothing.test.ts` covers 15.
- `__tests__/fitness/a-surface-under-a-dialog-takes-no-key.test.ts` covers 16.
- `__tests__/fitness/a-scrolling-route-restores-its-offset.test.ts` covers 23.
- `__tests__/components/settings/settings-index-page.test.tsx` covers 1 and 23 for the settings index: a fold
  written into the address of its own entry, painted open on the first render of a return, and marked as a row.
- `__tests__/components/layout/keep-alive-roots.test.tsx` covers 24a: a hosted root keeps its state while
  hidden, runs no effect, takes its offset back in the commit that shows it, is shown the moment a press names it,
  and goes with the account.
- `e2e/a-left-tab-root-leaves-nothing-behind.spec.ts` covers 24a on a real build: inert and `aria-hidden` roots, one
  main landmark, no id twice, the offset and the rows put back, no video held by a parked Shorts.
- `__tests__/hooks/use-push-slide.test.tsx`, `__tests__/utils/push-slide.test.ts` and `__tests__/utils/kept-peek.test.ts`
  cover `[signature-motion §26]`.
- `__tests__/hooks/use-scroll-restoration.test.ts` covers 18, 19 and 22: no layout read while scrolling, a
  bisection when it stops, a hold that follows the document's size and lets go.
- `__tests__/utils/back-pending.test.ts` and `__tests__/components/shared/history-depth-tracker.test.tsx` cover 13.
- `__tests__/fitness/a-route-skeleton-must-server-render.test.ts` covers 34.
- `__tests__/fitness/a-shortcut-yields-to-a-field.test.ts` covers 35 and 36.
- `__tests__/fitness/a-shortcut-leaves-the-browser-its-keys.test.ts` covers 38.
- `__tests__/fitness/every-feature-has-a-way-in.test.ts` and
  `__tests__/fitness/a-way-in-survives-a-phone.test.ts` cover 1 by making every state reachable.

**Not yet enforced**

- 24: `a-scrolling-route-restores-its-offset` takes the per-tab key once `useTabParam` records one. A
  `replaceState` tab switch fires no pop, so today there is no restore to read.
- 26: `an-active-tab-behaves-the-same-on-both-postures` fails when `MobileNav` and `Sidebar` do not share
  the active-tap handler. Waits on that shared hook: neither posture holds one to compare against, so the
  spec would only baseline its own subject.

## Traps

- 6a: Next files a redirect its router followed under the URL it asked for, keeps it `staleTimes.dynamic`
  seconds, and `router.refresh()` keeps it too. A sign-in bounced to onboarding replayed that bounce when
  onboarding finished, and the last step stood blank with nothing to press.
- 6a: Next strips its own `rsc` header before the proxy runs, so a router fetch is known by
  `sec-fetch-dest: empty`, which on a page route nothing else sends.
- 1: a fold held in component state is shut on every return, so the offset being put back lies past the end
  of the shorter page and Back lands at its foot with the fold closed.
- 3: `<Tabs defaultValue>` eats every deep link. The tab is controlled from the URL where the strip lives.
- 5: Next syncs `usePathname` and `useSearchParams` only for an entry whose state is not its own, so a
  rewrite that carries `window.history.state` forward rewrites the address bar and changes nothing on
  screen.
- 8: a phantom entry Next does not recognise is one it reloads the document on when Back lands there.
- 10: `history.back()` is not guaranteed to produce a `popstate`; the coordinator's watchdog runs the
  parked actions after 300ms and absorbs a late pop by count and by time so a real Back is never eaten.
- 11: a phone-only `history.back()` on menu close undid every `router.push` that menu made.
- 15: the live stage owned Escape with no session, so Escape on any dialog also left the page.
- 18: an unmount sample reads 0, because the incoming page's scroll reset runs in a layout effect before
  the outgoing page's passive cleanup.
- 18: a hidden kept root keeps its rows in the document, zero-sized, and a query for "the row at the top"
  that does not exclude them finds theirs first.
- 24a: a row under a hidden root reports nothing intersecting at no size; read as a row that left, every row
  was released and the root came back as boxes.
- 24a: the wrapper around the routed page is the same element on a root and on a page, or moving between them
  remounts everything beneath it, the kept roots included.
- 18: React renders a popstate's route inside the router's own `popstate` listener, so a pop tracker
  registered after it (a session that began on another route) learns of the pop after the list has
  already chosen a four-row first pass; the tracker listens in the capture phase.
- 18a: every remount of the feed sampled the learning model afresh, so each Back moved the query key
  and the list the person returned to was re-ranked under them.
- 19: an overlay's own pop lands on the same URL and never reaches the pathname effect, so arming the
  pop flag for it eats the next real navigation's reset.
- 31: the whole app frame was once `dynamic(ssr: false)`, so the initial HTML held no nav, aside or
  composer at all.
- 39: a `document` listener that preventDefaults `Tab` swallowed it for the whole page and appended a
  suggestion to a draft nobody was looking at.
