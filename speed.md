# Speed

Speed is a design property, not an engineering afterthought. This document owns the person's time: how long
anything is allowed to take, what they see at each moment while it takes it, and what the product spends its
first second on. The feeling it protects is that the interface is never the bottleneck: intent lands the frame
it is formed, the page they asked for is already there, and the network is something the app handles on their
behalf without asking them to watch.

## Owns and defers

**Owns.** The response ladder. Every performance budget and the rule that budgets only fall. Boot sequence and
what the server paints. Navigation cost. Data fetching, freshness, caching and invalidation. Interaction cost.
Media loading cost. Device tiers and graceful degradation. What is measured and how a number becomes true.

**Defers.** What a skeleton looks like and when it appears, `[loading-and-states]`. What happens with no network
at all, `[resilience]`. Durations, easings and the compositor rules that keep motion at 60fps, `[motion §39]`.
Image formats, capture, playback and codecs, `[media]`. Scroll restoration and Back, `[navigation-and-history]`.
Toast grammar for a rollback, `[feedback]`. Harness rules such as HEAD versus GET, the `app.` redirect and
DOM-reading observers, `[engineering-traps]`.

## The response ladder

What is on screen at each moment after a person acts. Every interaction is designed against this, and a rung
that cannot be met is a defect in the interaction, not a reason to move the rung.

| After | What has happened |
|---|---|
| 0ms | The control shows its pressed state, from CSS, with no JavaScript in the path. |
| 1 frame | The result is on screen: the heart filled, the row gone, the bubble in the list, the sheet opening. Optimistic, before any request leaves. |
| ≤100ms | A navigation has painted the destination's real chrome, and either its cached content or its ghost. |
| `NAV_GHOST_GRACE_MS` | A navigation still uncommitted stands its destination's ghost where the old page was; under it a warm push has already landed and a ghost would only flash. A revealed ghost stays `NAV_GHOST_MIN_MS`, so a commit just after it never reads as one. |
| `SKELETON_DELAY_MS` | The hold ends and pending data earns bones `[loading-and-states §15]`. |
| ≤1s | Content has painted on a warm device. On a cold one the frame, nav and composer are already interactive. |
| `TIMING.feedback` | Inline confirmations are gone. |
| `TIMING.undo` | An undo toast has expired. Anything still loading names what it waits for, and blocks nothing else on the page. |
| `NAV_GHOST_MAX_MS` | A navigation that never resolved gives the page back, so the app is never permanently mid-load. |
| Failure | The optimistic state rolls back with a crossfade and names what refused. Never a silent revert. |

## Rules

### Budgets

1. **Budgets only fall.** Every number below is a ceiling. When a measurement improves, lower the ceiling in the
   same commit, so the headroom cannot be spent again by accident.
2. **A budget is never raised to make a test pass.** Raising one is a product decision with a commit message that
   says what the person gets for the milliseconds.
3. **The measurement device is a mid-range phone at 4x CPU throttle on throttled 4G.** A feature that is smooth
   only on the developer's laptop is not smooth.

| Measure | Ceiling | What holds it |
|---|---|---|
| Input to visible response | 1 frame | Optimistic mutation, CSS press |
| Interaction to Next Paint | 100ms at p75 | Nothing heavy in a tap handler (§23) |
| Rail or tab click to painted content | 200ms warm | Route ghost, intent-warmed chunk, cached query |
| Overlay open to first frame of motion | 1 frame | CSS-driven presence `[motion §24]`, chunk warmed on intent |
| Layout shift after first paint | 0 | Ghosts at content height, reserved chrome `[loading-and-states §7]` |
| Total Blocking Time at boot | ~0 desktop, < 200ms mid phone at 4x | Idle effects, no sync decrypt or decode on boot |
| Largest Contentful Paint | < 1.5s warm, < 2.5s cold on 4G | Server-rendered frame, seeded pages, persisted cache |
| First data request leaves | < 1.5s after navigation start | One boot document, no prefetch storm |
| Dropped frames on the feed | < 10%, worst frame < 50ms | Bounded mounts and decoders, no blur animated on scroll |
| Shared shell JavaScript | 540 KB brotli | `route-javascript-stays-under-budget` |
| Route JavaScript above the shell | `/home` 201 · `/shorts/create` 220 · `/[username]` 184 · `/explore` 196 · `/notifications` 130 · `/shorts` 96 · `/settings` 58 · `/lists` 40 · `/login` 38 · `/signup` 32 · `/messages` 30 KB brotli | same test |
| Requests during boot | Falling; no CORS preflight on the critical path | Same-origin data path, one boot read |
| Round trips per common action | Send a DM ≤ 14 and falling; like, follow, bookmark: 1 | Batched extras, one RPC per action |
| Server round trip | ~100-150ms per RPC; a page chains at most two before it can paint | Parallel queries, seeds |
| Image bytes | The drawn size at the device's real DPR | `next/image` sizes from the layout `[media]` |
| Video decoders alive | Only what is on or adjacent to the screen | §33 |

### Boot

4. **The server paints the real frame.** Rail, nav, header, composer and the first rows are in the HTML.
   Nothing in the app frame is `dynamic(ssr: false)` except a surface with no first-paint job of its own.
5. **One boot document answers the shell's questions.** `get_viewer_boot` returns standing, entitlement, wellbeing,
   unread counts, muted words and the rest in one read, inlined into the page at `VIEWER_BOOT_SLOT` and seeded
   into the query cache by `applyViewerBootSeed` before first render. No hook re-asks a question it answered.
6. **The persisted cache is restored as soon as it can be, which is after first paint, never blocking it.**
   `readQueryCache` hydrates from IndexedDB inside an effect, keyed by viewer and busted by `ENV.buildId`, so a
   returning open paints, then fills, then refreshes behind that. It is a way to reach content sooner, **not** a
   way to avoid a layout shift: an entry that arrives a commit after first paint still moves whatever is under it.
   A surface that must not move decides on the server `[loading-and-states §7]`.
   Only successful queries under `PERSIST_PREFIXES` persist, only their first page (`keepFirstPage`), and only
   for `QUERY_CACHE_MAX_AGE_MS`. Each list is its own record: `persistQueryCacheOnChange` saves in idle time at
   most once per `CACHE_PERSIST_WINDOW_MS`, and a save writes the lists that changed and no others.
7. **Stateless and public pages are seeded on the server**, and hydration keeps the seed as `placeholderData`
   rather than blanking and refetching `[loading-and-states §21]`.
7a. **A drill-in never waits on its own seed.** A thread's page starts its reads on a
   `createStreamingSeedClient` and returns without awaiting them, so the route renders from the client cache the
   moment the router asks and the server's answer streams into the same queries.
8. **Nothing prefetches on viewport entry.** `AppLink` fixes `prefetch={false}` and does not expose the prop, so
   chrome cannot opt back in. Routes warm on intent through `useRouteWarm`, panels through `usePreloadOnIntent`,
   and both bail under `shouldConserveData()`. A route's warm fetches its ghost's chunk too, through
   `preloadRouteGhost`, so the pending frame never waits on code `[loading-and-states §22]`.
9. **Speculative warming waits for boot to finish.** `useNavPrefetch` starts `NAV_WARM_DELAY_MS` after mount, then
   waits for the launch to load, paint and go quiet on the network (`whenLaunchSettled`), and warms one route per
   idle slice (`IDLE_SLICE_TIMEOUT_MS`, so WebKit's missing idle callback costs a frame, not seconds), because warming
   inside the boot window is what turned a nav rail into forty route requests. The order is the order a person opens
   them: the tab bar (`MODE_TAB_BAR_ROUTES`), compose, the viewer's profile, then the header's routes; the route on
   screen is skipped until the person leaves it. Progress lives in the warm registry, so a navigation neither restarts
   the queue nor its delay, and an entry is warmed again `ROUTE_WARM_RENEW_AFTER_SECONDS` after it was placed, inside
   the `FULL_PREFETCH_FRESH_SECONDS` the router keeps it. On a cellular link or a shell's `expensive` path
   (`nativeNetworkFacts`) it keeps at most `CELLULAR_PREFETCH_ROUTE_LIMIT` routes warm; it holds a slice while a press
   is navigating and does nothing in a hidden page. A press is the person's own intent: only the data saver stops
   `useRouteWarm`, never a slow link, and a finger warms the route tree alone so the tap that follows reuses it
   instead of rendering the page twice.
10. **Heavy subsystems load after first interaction is possible**: end-to-end encryption, WebAuthn, the editor,
    charts, emoji data, and the schema a submit is validated against — `deferredResolver` reads it on the form's
    first focus, so zod is not in the paint of a screen with an email field and a button, and `validateInput`
    reads the post schemas when something is published. A module a list reads keeps no schema beside its
    constants. The shell reserves their space; it does not ship them.
11. **Boot work runs in idle time.** `useIdleEffect` with `IDLE_TASK_TIMEOUT_MS` for analytics, device check-in,
    cache warming and key rotation. Nothing that is not on screen runs before what is.
12. **Fonts do not block.** One family per theme, Latin subset, `display: swap`, a real system fallback stack,
    and only the default family preloaded.
12a. **The browser bundle holds none of the English catalogue, and no string ever draws blank.** The server inlines, beside
    each flush, the strings that markup was drawn with (`ServedLabels`) and the head preloads the whole catalogue from a
    content-addressed, immutable address (`english-asset`), downloaded beside the scripts. The app below `RuntimeProviders`
    does not mount until `whenEnglishReady` resolves, so everything it draws finds its words; a device that holds the build's
    copy (`persisted-english`) starts at once. A string asked for outside that gate (a lite page's handler) draws blank for the
    moment and redraws, never as its key. A message group is read when asked, never spread at import, or it freezes blank.
    `scripts/perf/cold-strings.mjs` samples a cold first visit with a slow catalogue and fails on any blank control or key.

### Navigation

13. **A route paints its own ghost from the server** at the page's real chrome and content height, so a click
    lands on something in the same frame `[loading-and-states §8]`.
14. **A returning list keeps every page it had.** `keepFirstPage` bounds only what is persisted to disk; in
    memory an infinite list is never refetched from page one on the way back `[navigation-and-history §13]`.
15. **A tab switch spends no request.** Moving between tabs of one page and back does not refetch: both tabs'
    queries stay resident for `QUERY_GC_TIME`, and their stale times decide the rest.

### Data

16. **Freshness is decided by a named stale time, never by a mount.** A query reads a rung of
    `QUERY_STALE_TIME` `[tokens]` or a named constant whose declaration says why. The provider default is
    `medium`, and a bare number is never a stale time.
17. **The window's focus never refetches.** `refetchOnWindowFocus` is off for the whole app: coming back to a
    tab must not replace what is under the person's thumb. Freshness on return comes from an explicit signal
    `[resilience §12]`.
18. **Every mutation names the reader it refreshes.** An invalidation aimed at a key nothing observes is not a
    slow update, it is no update ever. `invalidateFeedCaches` and the `queryKeys` helpers are the vocabulary;
    a mutation that invalidates nothing and writes no cache is a defect.
19. **An event carries the row.** Realtime patches the cache with what changed rather than triggering a refetch,
    and a settled device checks in rather than republishing.
20. **One call per page of extras.** A feed page reads its likes, bookmarks, poll votes and follow states in one
    batched call, and a checked post is never re-asked. No list issues a request per row.
21. **Queries that can run in parallel do**, and placeholder data beats a blank: the previous page, the server
    seed, or the persisted cache, so the person sees content a second old instead of a skeleton.
21a. **The proxy asks the database nothing a layout beside it can ask.** `proxy.ts` runs in the visitor's
    nearest region, a continent from the database, while the render runs next to it. A gate a layout above
    every Suspense boundary can answer is handed there and still answers with a 307: the enforcement door by
    `ENFORCEMENT_DOOR_HEADER`, and whether `/onboarding` is already finished by the (onboarding) layout itself.
    The proxy keeps the token refresh and the verdicts it has to write cookies for.
22. **Pagination fetches within a screen of the end**, never at it, and lives in a hook rather than in a list.

### Interaction

23. **A tap handler does no work.** It writes state and returns. Formatting, encryption, upload, analytics and
    navigation preparation happen after the frame, in a transition or on idle.
24. **Every mutation is optimistic.** `onMutate` writes the cache and the request follows; the rollback restores
    the snapshot and says what refused `[feedback]`. Offline, a mutation fails fast rather than pausing:
    `networkMode: 'always'` is set globally so no button spins against a network that is not there. A control is
    never dimmed to wait for its write: a switch turns on the tap and turns back, with a toast, if the write is
    refused, and writes that must land in order share a `scope`, so three taps on an arrow are three moves. The
    write reaches every observer in the tap's own task: `notifyObserversInTheTapsTask` replaces TanStack's
    `setTimeout(0)` hand-off, which delivered an optimistic state a task after the tap. A second tap on a toggle
    during its round trip is queued and carried to the state it asked for, never dropped.
25. **A read retries twice and no more.** `MAX_QUERY_RETRIES` is 2, and a deterministic refusal or an auth error
    is never retried, because retrying a refusal only delays the message that explains it.
26. **Typing never lags.** The field itself is never debounced; derived work is. Search debounces at
    `TIMING.debounce`, and a surface with a different rhythm names its own constant with a reason. A request the
    next key or the next slider position supersedes is cancelled (the query hands its `signal` to the call) and
    the last answer stays up until the next one lands (`placeholderData`), so nothing folds, reopens or flickers
    between keys. A post being written renders once per key: the caret travels in the keystroke's own batch, the
    markdown paints a render behind past `RICH_PAINT_DEFER_CHARS`, and the classifier reads a deferred copy. The
    composer hands its controls (`composer.controls`) a stable identity through typing, with methods that always
    call the newest closure (`useStableMethods`), so a control that reads no words is not rendered by them.
27. **Heavy work leaves the main thread.** Encryption, image resize, waveform analysis, video frame extraction
    and hashing run in workers or idle slices, never synchronously during boot or scroll. The backup key's scrypt
    is derived by `scrypt-worker`; a page with no Worker derives it in slices of `SCRYPT_SLICE_MS` that yield to
    the browser, so the first Messages open never holds a frame.
28. **A long list bounds what is mounted.** The feed shares one `IntersectionObserver` through
    `FeedVisibilityProvider`, and `RowWindow` swaps a row more than `ROW_KEEP_VIEWPORTS` viewports away for a box
    of the height it last measured, on every engine, in the feed, the profile tabs and the conversation list:
    `content-visibility` skips painting a row but keeps its effects and observers. `MAX_AUTO_PAGES` and
    `MAX_AUTO_SHORTS` bound an endless scroll; shorts mount a player only within `MOUNT_WINDOW` of the active
    index and a picture one further out, and a playing short renders nothing: its clock reaches the bar and the
    readouts by subscription. Mounting never happens inside a scroll handler.
28a. **A page of rows mounts a slice at a time, and each slice yields.** `useStagedMount` commits
    `FEED_COMMIT_ROWS` rows, waits for boot to go idle, and then adds `FEED_COMMIT_ROWS` more per `yieldToMain`,
    each in a transition so an input can land between two slices. Committing the rest of a page in
    one render is one task, and on `/home` it was the longest task of the whole boot.
28b. **A card subscribes to nothing the list already knows.** The viewer's muted words, spoiler topics,
    moderated spaces and restricted users are read once by `ViewerListsProvider`, and how they read translations
    once by `TranslationReaderProvider`; a card reads them through context and mounts no mutation. A lookup mounts
    only where it is used: a quote when the post quotes, a subscription check when it is subscriber-only, a
    translation when one is wanted. An overlay host mounts when first reached: `PostModals` after the first
    sheet opens, the context menu's panel at the first pointer or touch.
    `__tests__/components/post/a-feed-card-mounts-only-what-it-uses.test.tsx` counts them.
28c. **A component selects the field it reads from the profile, never the profile.** Every settings toggle
    writes one field; a whole-profile selector re-renders for all of them. `useUserFieldWriters` is the hook
    for a caller that only writes, and `__tests__/fitness/a-selector-reads-a-field-not-the-profile.test.ts`
    holds the screens that hand the whole profile on.
29. **A gesture runs on the compositor.** During a drag the transform is written imperatively and React is out of
    the loop until release `[motion §31]`.
30. **Nothing polls where an event exists.** The bounded polls are named constants with a stated reason:
    `NEW_POSTS_POLL_INTERVAL_MS` because `posts` is deliberately not in the realtime publication,
    `MODERATION_STATUS_POLL_INTERVAL_MS` because a ban lands in the database and not in this client,
    `STORY_RING_POLL_INTERVAL_MS` because moderation approval is not an insert, and
    `POST_PROCESSING_POLL_INTERVAL_MS` because a worker finishing a scan or a transcript writes rows the
    author's client cannot subscribe to. A new poll needs the same sentence.
31. **Uploads are progressive and cancellable.** An attachment the person removed stops spending their data:
    an `AbortSignal` runs from `useMediaPrepare`'s `forget` through `useMediaUpload` to the XHR in
    `presignedUpload`, a cancel is never retried and never reported as a failure, and a file that shares a
    bounded pool with siblings still wanted keeps going until the last of them is dropped too.
    The post is on screen before its insert leaves the browser,
    progress is drawn inside the card, and the composer closes without waiting for bytes. A short is the same:
    the tap seats the reel and navigates to it, and the upload, the cover, the sound and the INSERT all run
    after the person has left the create screen `[loading-and-states §41]`. The person waits only for what the
    row cannot exist without: the scan of the published bytes, the transcript, the description, the embeddings
    and the fingerprints are queued work with their own retries, never something a publish stands in front of.
    A generation is the same: the typo check starts as the writer reaches for Post (`usePolishIntent`: a
    hover that lingers `POLISH.approachDwellMs`, a focus or a press, and on a short also leaving the caption),
    never while they type or a pointer only crosses Post, and a warm-up never reads as busy on anything the
    writer can press. A submit that stays on the page waits `POLISH.submitWaitMs` for a check still running and
    then publishes the words the author actually wrote; a tap that navigates, as a short's does, waits for
    nothing and takes only a check that has already answered, because the seated row is the answer to the tap
    and a model cannot be in front of it. So on a short every approach waits out the dwell (`tapLeaves`), and
    its tap aborts any check still running.

### Media cost

32. **Media never blocks the first frame.** What paints is a placeholder; no decode, fetch or measurement of a
    real asset sits between a navigation and the page's first paint `[media]`.
32a. **A picture that can open the first screen is asked for first.** `priority` on `FeedImage` is eager,
    preloaded and `fetchpriority=high`; next/image's own `priority` only preloads. A feed spends it on the first
    row that paints an image or a poster among its first `FEED_FIRST_SCREEN_ROWS`, the most rows a first screen
    can open with, and on no row after them, because the page's own scripts load at low priority.
33. **A decoder is a budgeted resource.** The feed's live-decoder budget is what holds the dropped-frame
    ceiling, so widening it is widening a budget and takes rule 2's commit message. `[media]` decides which
    element holds one.
34. **The link decides whether bytes are spent.** `shouldConserveData()` is true under data saver,
    `prefers-reduced-data`, or an effective type of `slow-2g`, `2g` or `3g`, and it gates autoplay, route
    warming and panel warming alike, because none of them is worth a megabyte the person is paying for.

### Device tiers

35. **The product degrades; it never disappears.** Every feature still works on the low tier. Degrading means
    fewer effects, not fewer capabilities.
36. **One capability test, read before paint.** `isLowEndDevice()` is the single answer: under 4 GB of reported
    memory, under 4 cores, a Safari that reports neither on an iOS at or below `IOS_LOW_TIER_MAX_MAJOR`, or a
    stated data-saving preference. `PERF_LITE_CLASS` is stamped on the root by the blocking boot script and kept
    in sync in React, so nothing decides per component.
37. **The low tier sheds cost, not meaning.** `PERF_LITE_CLASS` is what every shed effect keys off
    `[colour-and-effects]`, and worker concurrency halves with it. Nothing a person can do disappears.
38. **On-device model size is a tier decision.** `canAffordLargeModel()` picks the smaller model; the feature
    stays.
38a. **Background work sizes itself to the tier.** The history completeness walk reads
    `HISTORY_WALK_PAGE_SMALL` rows per page instead of `HISTORY_WALK_PAGE` on the low tier, because one page
    is a ratchet load and an archive write per row. It still covers the whole account, only slower.

### Measuring

39. **A speed claim without a measurement is a guess.** Measure on production, with a real signed-in session,
    one sweep at a time, cold and warm separately, and never compare runs from different times of day. The
    harness traps that make these numbers lie are in `[engineering-traps]`.
40. **Time the tap, not the route.** Record from pointerdown on the real trigger to the frame the result paints.
    A scripted navigation cannot see a lazy fallback or an intent-warmed chunk.
41. **Count requests and waves, not only bytes.** A boot that is small but sequential is slow.
42. **Read the frame timeline, not the JavaScript profile alone.** On the feed, most of the scroll cost was paint
    and decode.
43. **Every budget is checked on every ship.** `route-javascript-stays-under-budget` runs against the build —
    `npm run test:budget`, immediately after `npm run build` in the release, because it measures that build's
    own manifests — and `scripts/smoke-deploy.mjs` drives the core paths on production after deploy; a new
    budget is added to one of them in the same commit. A working tree that has never built is the only thing
    that spec may skip for, and it says so in its own name `[engineering-traps]`.
44. **The field is read per route, weekly.** `npm run vitals` prints LCP, INP, CLS and TTFB at p75 per route
    from what Vercel Speed Insights collects off `app/layout.tsx`, against Google's published thresholds and
    with every route under ten sessions dropped, because a p75 over a handful of sessions is noise. A route
    outside `good` is named at the foot of the report, so a regression is seen before a person reports it.

### Never ship, for speed

The review checklist. Each line is an observable defect; the rule it violates is beside it.

| Defect | Rule |
|---|---|
| A tap whose result waits on a request, a chunk or a decrypt | §23, §24 |
| A `<Link>` that prefetches on viewport entry inside chrome | §8 |
| A lazy panel that mounts, or a `loading:` fallback that renders, before its first open | `[components]` |
| A hook that re-asks a question the boot document answered | §5 |
| A list that issues a request per row | §20 |
| A page that chains three round trips before it paints | §21 |
| A query with a literal stale time | §16 |
| A mutation that invalidates nothing | §18 |
| A poll where an event exists | §30 |
| A scroll handler that mounts, measures or reads layout | §28 |
| An inline `backdropFilter`, or a blur that animates | `[motion §41, §42]` |
| A blur over an opaque plate | `[colour-and-effects §40]` |
| An overlay that reads geometry before its first frame, or a script spring that animates what the compositor can | `[motion §24, §38a]` |
| An image requested larger or smaller than it is drawn | `[media §4]` |
| A synchronous crypto, resize or decode call during boot or scroll | §27 |
| A budget raised to make a test pass | §2 |

## Enforced by

- `__tests__/fitness/route-javascript-stays-under-budget.test.ts`: rules 1 and 2, and the two JavaScript budget rows.
  Measures brotli at maximum quality from `.next/app-build-manifest.json`, JavaScript only; the shell is the
  root and polyfill chunks plus the intersection of every `(main)` route.
- `__tests__/fitness/invalidation-reaches-a-reader.test.ts`: rule 18. Fails when an invalidation names a key that
  no surface observes.
- `__tests__/fitness/a-lazy-overlay-waits-for-its-first-open.test.ts`,
  `__tests__/fitness/a-lazy-sheet-waits-to-be-opened.test.ts`: the lazy-panel row of the checklist.
- `__tests__/fitness/discovery-feed-has-one-query.test.ts`: rule 20, for the feed's own retrieval path.
- `__tests__/supabase/enforcement-door.test.ts` and `__tests__/app/auth-layout-lets-a-sign-in-finish.test.tsx`:
  rule 21a. They fail when a signed-out document pays the enforcement read in the proxy, or the layout drops it.
- `__tests__/fitness/a-gate-layout-sits-above-every-suspense.test.ts`: rule 21a. Fails when a root loading file,
  a Suspense or a suspending `dynamic()` wraps the root layout's children, which would stream a gate's 307 as a 200.
- `__tests__/supabase/onboarding-gate-one-round-trip.test.ts`: rules 21 and 21a, for the proxy's onboarding gate.
  Runs the real auth-js client and fails when the onboarding read waits behind GoTrue's, or when `/onboarding`
  pays it in the proxy.
- `__tests__/hooks/use-staged-mount.test.tsx` and `__tests__/components/post/feed-list.test.tsx`: rule 28a.
  Both fail on a feed that goes from its first commit to the whole page in one render.
- `__tests__/components/post/feed-list.test.tsx` and `__tests__/components/media/feed-image.test.tsx`: rule
  32a, for the slot reaching a picture in the last row a first screen can show and none after it, and for
  `priority` asking for high priority.
- `__tests__/fitness/no-undefined-query-option.test.ts`: rule 16, for options silently dropped.
- `__tests__/fitness/the-typing-surfaces-compile.test.ts`: rule 26, for the composer's hooks and deck the React
  Compiler must not skip; `__tests__/hooks/use-composer-text.test.ts` (one render a key),
  `__tests__/hooks/use-feed-preview.test.tsx` and `__tests__/hooks/use-text-autocomplete.test.tsx` (cancel and keep), and
  `__tests__/hooks/use-thread-composer.test.tsx` (the controls hold and a memoised control skips every key).
- `__tests__/factories/a-scoped-write-queues-and-refetches-once.test.tsx`: rule 24, for the queued writes.
- `__tests__/hooks/use-shorts-playback-sound.test.tsx` and `__tests__/hooks/use-shorts-scroll.test.ts`: rule 28,
  for the playing short that does not render and the flick read from the finger's last 100ms.
- `__tests__/fitness/chrome-never-prefetches-on-sight.test.ts`: rule 8. Fails when a module in the app frame
  imports `next/link`, and when `AppLink` stops fixing the prop off.
- `__tests__/utils/presigned-upload.test.ts`, `__tests__/hooks/use-media-upload.test.tsx` and
  `__tests__/hooks/use-media-prepare.test.ts`: rule 31, for the cancel that reaches the bytes.
- `scripts/smoke-deploy.mjs`: rule 43. Seven checks on production after every deploy.
- `__tests__/fitness/a-budget-names-the-device-it-was-measured-on.test.ts`: rules 3 and 39. Reads the budget
  table above and fails on a row that states a duration without the device, network or cache state it holds
  on, and when the table stops naming the measurement device at all.
- `__tests__/fitness/a-hook-does-not-re-ask-the-boot-document.test.ts`: rule 5. Reads the legs of
  the boot document and fails when a query calls one of them under a key
  `applyViewerBootSeed` does not fill.
- `__tests__/fitness/a-stale-time-has-a-name.test.ts`: rule 16. Fails on a `staleTime` written as a number,
  and when the provider stops defaulting to `QUERY_STALE_TIME.medium`.
- `__tests__/fitness/a-mutation-is-optimistic-or-says-why.test.ts`: rule 24. Fails when a mutation names the
  caches it refreshes and writes none of them before the request, unless it declares itself background.
- `__tests__/fitness/the-entry-screen-validates-after-it-paints.test.ts`: rule 10, for the entry flow. Fails when
  a signed-out screen imports a zod schema or `@hookform/resolvers/zod` into its own graph, and when anything
  reaches `lib/validations/auth-resolvers.ts` other than through an `import()`.
- `__tests__/fitness/a-poll-states-its-reason.test.ts`: rule 30, both halves — the interval is a named
  cadence rather than a number or a bare `TIME_MS` unit, and that cadence's declaration says why an event
  will not do.

- `scripts/vitals-report.mjs`: rule 44. Reads the field per route; run it, do not guess.

## Traps

- **4** The whole app frame was once `dynamic(ssr: false)`, so the initial HTML held no nav, aside or composer.
- **7** Hydration threw away the server-painted feed and refetched it from page one.
- **9** Speculative warming inside the boot window competes with the first data request for the same connection,
  and a rail that warms on mount is indistinguishable from one that prefetches on sight.
- **16, 18** With a default stale time, a query that no mutation invalidates and that no observer re-mounts
  stale can go a whole session without ever updating.
- **21** auth-js holds one lock across a bare `getUser()` round trip, and every PostgREST call on that client
  reads its token through the same lock, so a `Promise.all` beside it still ran one read after the other.
- **21a** The enforcement read in the proxy cost `/login` 469ms at p75 on a warm instance, 1s from Singapore
  and Sydney, while every other anonymous page paid 12-20ms. The onboarding read cost `/onboarding` 344ms at
  p75 on a warm instance, 1.5s from Mumbai, for a question its own layout asks again beside the database.
- **28** `content-visibility` skips paint but keeps every row's state and observers mounted, so an unbounded
  scroll grows memory even while nothing is being painted.
- **28a** Mounting the rest of the page on one idle callback is still one task: 26 cards rendered together
  held the main thread for 267-317ms on `/home`, which is where the 67ms of input delay on a tap came from.
- **33** Fifteen reels decoding at once took the feed to 29fps and a worst frame of 3,883ms; the budget is what
  the frame ceiling actually rests on.
- **8** Registering every route's ghost eagerly put forty ghosts and everything they import into the shell
  of every route. The shell carries the registry and the tab bar's ghosts; the rest are chunks of their own.
- **10** A named import carries its whole module: a ghost that wanted one class string shipped twenty-two
  notification glyphs in the shell, and a post-id check shipped the image normaliser beside it.
- **43** A budget with no test is a number in a document. Both budget rows here exist because a test reads the
  build manifest on every run.
