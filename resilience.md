# Resilience

This document owns the person's day at its worst: the tunnel, the lift, the two bars, the phone call that
interrupts them mid-sentence, the three hours away, the second device, the tab that has been open since
Tuesday. The feeling it protects is trust. Nothing they typed is ever lost, nothing they sent quietly fails,
nothing they read jumps out from under them, and the app never asks them to think about the network. Where the
product cannot do the thing, it says so in one line, in the frame they are already looking at, and keeps
everything else working.

## Owns and defers

**Owns.** Offline behaviour and what each action does with no network. Flaky links, retries and backoff.
Reconnection and catch-up. Queued sends. Backgrounding and foregrounding. Returning after hours. Session expiry
without interruption. Drafts across navigation, refresh and crash. Continuity across devices. Concurrent edits.
The service worker's contract with the person.

**Defers.** The shape of a loading, empty or error surface, `[loading-and-states]`. Budgets, stale times,
caching for speed and the polls' intervals, `[speed]`. The words in any of these messages, `[copy-and-voice]`.
Toast placement, priority and the undo window, `[feedback]`. What a lock icon promises about encryption,
`[permissions-and-trust]`. Retry as a control's label, `[input-and-forms]`.

## Rules

### Offline

1. **The connection is verified, never assumed.** `navigator.onLine` is a hint only: a false reading is
   confirmed with a real `HEAD /api/health` before anything changes on screen, and a true reading is trusted at
   once. `useOnlineStatus` is the one answer; no surface reads the browser flag directly.
2. **Offline is one banner, in one place.** `OfflineBanner` stands in the toast dock on a solid ground,
   "Offline, showing what's saved", so it never lies across the header's controls `[feedback §18]`. It is the
   only ambient offline signal; nothing else announces it, and it costs no layout at all.
3. **Nothing is disabled for being offline.** Every control stays live and every action is attempted, because a
   verified-offline reading can still be wrong and a greyed-out app is worse than an honest refusal. The action
   that cannot complete says so itself, in its own surface.
4. **An action that fails for the network says which network.** `connectivityMessage` answers a failure that
   names nothing — no status, no SQLSTATE, no refusal code: an offline device reads "you are offline", a request
   that never landed reads "could not reach Surf". A failure that names itself keeps its own reason, because
   `navigator.onLine` is read when the error is mapped, not when the request left, and must never overwrite a
   named refusal with a guess about the wire. The one exception is `not_authenticated`, which the client mints
   for itself when a token cannot be refreshed — offline is what produces it. Neither is ever "something went
   wrong".
5. **A write fails fast rather than hanging.** Mutations run at `networkMode: 'always'`, deliberately overriding
   the library default that would pause them: a paused mutation is a button that spins forever and a person who
   does not know whether it went.
6. **A cold offline navigation lands on our page.** A navigation the network never answered gets the
   precached `/offline.html`, which asks the network for nothing; a real page or a real 404 is never shadowed
   by it. The install caches each entry on its own, so one the device could not fetch never fails the install,
   and the first navigation of each worker start fetches whatever the install missed. A landing route never waits
   on a bad network either: past 400 ms the worker answers with its last copy of that document
   (at most a week old, dropped when a sign-in screen is asked for) and refreshes it behind. Content-hashed
   chunks are kept in a cache no deploy purges and which evicts its least recently used past 60 MB.

### Flaky network and retries

7. **A read retries twice, quietly.** Two attempts on the library's backoff, then the surface shows its error
   with a retry the person controls `[loading-and-states §4]`. No spinner outlives the retries.
8. **A refusal is never retried.** `isDeterministicError` stops every 4xx except 408 and 429, and every
   non-retryable database code, because repeating a refusal only delays the sentence that explains it.
9. **Every subsystem's retry is bounded and named.** Realtime backs off exponentially with jitter from 1s to
   30s. The outbox allows three automatic attempts at 5s, 30s and 2 minutes. Auth walks
   `AUTH_RETRY_DELAYS` then settles into `AUTH_DEGRADED_RETRY_MS` forever. A new retry loop states its bound in
   its own constant.
10. **Jitter is mandatory on a shared outage.** A fixed backoff reconnects every client on the same tick and the
    outage repeats itself; `realtimeResubscribeDelayMs` halves the cap and randomises the rest.
11. **The person is never told about a retry that succeeded.** Only the last failure is worth a sentence.

11a. **A request has a deadline.** Every PostgREST and edge-function call carries one (`withRequestDeadline`,
    `SUPABASE_REST_DEADLINE_MS`), and `fetchWithTimeout` is the door for any other call, so a dead zone ends in
    the designed failure instead of a minute of spinner. Signals are combined by `anySignal`, never
    `AbortSignal.any`, which throws below Safari 17.4.

### Reconnection and catch-up

12. **Coming back is when the app asks what it missed.** Realtime has no replay, so a live subscription
    re-runs its own invalidation on reconnect and on foreground, through `useAppForeground` and `useOnline`.
    A subscription that does not backfill silently loses everything that happened while the tab slept. A
    reconnect still waiting out its backoff runs at once on the same two signals: the backoff paces a dead
    network, not a person who is back.
13. **One foreground signal for the whole app.** `useAppForeground` collapses `visibilitychange` and `focus`
    into a single throttled notification, and `useAppBackground` pairs `visibilitychange` with `pagehide`. A
    surface that binds its own listener multiplies every return by the number of surfaces.
14. **Returning refreshes, it does not reshuffle.** `useRefetchOnReconnect` invalidates active queries only, so
    what the person is looking at becomes correct and what they are not looking at waits for its own observer.
15. **Every foregrounding job is targeted.** Session revalidation, profile revalidation, notification backfill,
    presence ping, outbox re-arm and the new-posts count each subscribe for themselves. Nothing invalidates the
    whole cache on return.
16. **A catch-up is a delta where a delta exists.** The new-posts count asks with a `since` timestamp. Realtime
    catch-up is a full invalidation today; a subscription over a list long enough to be expensive states its own
    `since`.

### Queued sends

17. **A message the network refused is still the person's message.** It leaves the conversation cache and lands
    in the outbox, keeps its place in the thread, and reads as undelivered with a retry and a dismiss beside it.
    It is never deleted by the app.
17a. **"Not sent yet" is not "not delivered".** A send waiting on the connection, on this device's keys, or on a
    recipient who has no device yet keeps a clock and one quiet line and raises no toast, because something will
    send it. Red, a retry, a dismiss and exactly one toast are for a send that has run out of automatic tries or
    that the server refused — and a refusal stays red even offline. Waiting on the world costs no try: the
    budget is for attempts the app actually made.
16a. **A message this device cannot open asks for itself.** "Cannot be opened here" is never the end state: the
    device files one silent ask per message (`message_resend_requests`), the sender's client answers it from
    its own archive by appending one device copy, and the bubble opens. The answer is the sender's own device,
    so the ask waits for that device to be opened: `RESEND_GIVE_UP_MS` covers a night, and until it elapses the
    bubble says it is getting the message and offers nothing to do. An ask the sender can never answer — a
    group generation it has rotated past — is refused instead (`refused_at`), which is terminal: the asking
    device stops re-filing and says so at once rather than waiting out a clock. Nothing about the asking, the
    answering, the refusing or the waiting is a toast.
17c. **Nobody presses Retry for something the app can try itself.** The connection returning, the app coming
    back to the foreground, and the recipient publishing a device each give every waiting message its tries
    back, so the automatic budget bounds a burst rather than the message's life. Only a refusal is never
    revived, because the same write is refused every time.
17b. **Nowhere to send it is said before the send, not after.** A DM to someone who has never opened Messages
    carries a note above a composer that stays live, and the message waits for their first device rather than
    spending its tries. The directory announces that device, and the announcement is what drains the outbox.
18. **The outbox survives the process.** It persists into the sealed archive through `saveOutbox`, flushes on
    hide through `useFlushOnHide`, and rehydrates on the next open through `useOutboxHydration`. The copy is
    written when the send STARTS, so a tab that stops existing mid-send loses nothing; an attempt that was in
    flight when the process ended comes back as waiting rather than as failed.
19. **The outbox drains when three things are true**: auth resolved, connectivity verified, something to send.
    It drains oldest first, one conversation at a time and one message at a time, so a returning connection
    never stampedes.
20. **An automatic resend carries an idempotency key.** Without `client_message_id` a retry after a lost
    acknowledgement commits a second copy, so an entry with no key waits for a deliberate tap.
21. **A refusal is terminal and says why.** A block, a revoked permission, a declined request or a moderation
    gate answers the same way every time, so the entry is marked refused, never retried, and the person is told
    once rather than watching a bubble fail on a timer.
22. **Only sends queue.** A like, a follow or a settings toggle offline fails, rolls back and toasts. Queueing
    them would replay stale intent into a world that has moved on.

### Background and foreground

23. **Leaving flushes; it never loses.** Everything held behind a debounce writes on the background signal:
    post and story drafts (`useAutoSaveDraft`), editor projects, analytics batches, feed-learning weights,
    screen-time credit, watch metrics, profile dwell, the encrypted backup and the outbox. A collaborative
    thread's segments (`useDebouncedWrites`) also write on blur and on unmount, and Publish waits for them.
24. **Leaving releases what only a foreground app may hold.** The camera stops, the session ends its span, and
    any capture stream is torn down rather than left claiming the device.
25. **Returning restores rather than restarts.** A camera whose track ended re-initialises, a game clock resets
    its accumulator instead of applying the elapsed hour, and a render watchdog rebases so a backgrounded export
    is not aborted for stalling.
26. **The app-switcher preview shows nothing private.** A conversation blurs behind `protect-blur` on hide and
    clears on show `[colour-and-effects]`.
27. **A background tab does no work that a foreground tab would do.** Polls check `document.hidden` and return.
    The Realtime heartbeat is the one exception, and it ticks in a worker: a hidden tab still has to ring.

### Returning after hours

28. **Nothing moves under the reader, and an accepted offer is kept.** The feed the person left is the feed
    they come back to. New content is an offer, never an insertion: `NewPostsPill` names the count and waits
    to be tapped. The tap then reads back the very posts it named (`list_new_following_posts`) and seats them
    at the top through `prependPostsToFeedCaches`, newest first — before the re-rank behind them runs and
    again after it, so a ranked slate cannot bury what was promised. Pull-to-refresh does the same, because
    the two gestures mean the same thing. Fewer rows than the count means rows that can no longer be read;
    the pill clears on the tap either way, and never sits over a feed that already holds them.
29. **Focus does not refetch.** `refetchOnWindowFocus` is off for the whole app. Coming back to a tab must not
    replace what is under a thumb; freshness comes from the targeted foreground jobs and from the pill.
30. **The persisted cache paints first.** A cold open after hours restores the last-known feed, bookmarks,
    notifications, stories and trending from IndexedDB and refreshes behind them `[speed §6]`.
31. **A new deploy is offered, not forced.** `promptAppUpdate` raises a standing offer, a dot on the account and
    an "Update Surf" row in its menu `[feedback §21]`, which never times out and never sits in the toast stack;
    the app never reloads itself under the person. The one exception is a tab
    asking for code the deploy removed: a `ChunkLoadError` has already replaced the page with an error surface
    no retry can clear, so `recoverFromStaleChunk` reloads it at most once per `STALE_CHUNK_RELOAD_WINDOW_MS`,
    and never over unsaved work. A chunk a background import lost broke nothing on screen: `noteStaleChunk`
    only offers the update, once the deployment confirms a newer build.
32. **Scroll position survives the return.** `(not yet built)` for the feed: nothing persists a feed offset
    across a cold open. Needs the restoration path in `[navigation-and-history]` extended to a persisted
    per-route offset with an age bound, so an hours-old position is not restored into a list that has changed.

### Session expiry without interruption

33. **A session never expires in front of a person.** One policy owns every rotation: `createSessionRefresher`
    refreshes only inside `TOKEN_REFRESH_MARGIN_S` of expiry, through one in-flight promise, on a heartbeat
    that does not trust a throttled background tab, and again on `visibilitychange`, `focus` and `pageshow`.
    auth-js holds `navigator.locks` for a persisted browser session, so two tabs never present one token to
    GoTrue's rotation, and a refusal is not tried again before `SESSION_REFRESH_RETRY_MS`, so a ceiling
    cannot be hammered by a five-second tick.
34. **A request either carries a token or does not go.** When the browser claims a session — a token in hand or
    an `sb-*` cookie — and none can be resolved, the request fails as `NotAuthenticatedError` rather than
    leaving as `anon` and returning 42501. A user restored from `localStorage` is not a session: until
    `sessionConfirmed`, no authenticated read is enabled and the session-only runtime does not mount, and it
    never mounts on a screen `isSessionFreeScreen` names. A 401 is refreshed once and the call replayed; the
    person's action completes and they are told nothing. A keepalive sent as the page leaves cannot refresh,
    so `getLastAccessToken` hands it only a token that is still alive.
35. **The app signs nobody out on a refusal the server did not confirm.** auth-js clears its own storage on any
    non-retryable refresh refusal — a 429 ceiling, a proxy 403, a token already rotated — so a `SIGNED_OUT`
    with a user still on screen is a claim to check: `recoverSession()` asks `/api/auth/session`, re-seats the
    session the server still holds, and only a `dead` verdict ends it. From a page that needs a session
    that leaves for `expiredLoginUrl()` in a single step, never as a toast on whatever button was
    pressed; a public page simply reads as signed out. An unreachable server changes nothing: the cached
    user stays and the ladder retries. A sign-out the person asked for clears this device and tells
    the other tabs of this browser; it revokes nothing on their other devices, which is the separate control
    in Settings that says so.
36. **A transient auth message never becomes copy.** "Not authenticated" while a session is still settling is
    not "session expired": a write waits on `requireSettledUser` and, if it must still refuse, says
    `error.authRequired`. `error.sessionExpired` is kept for a session that is genuinely gone.
37. **A throttled tab is not a device waking up.** Browsers throttle a hidden tab's timers to a tick a minute,
    so `isWakeGap` counts a gap over `SLEEP_THRESHOLD_MS` as a wake only when the tab was visible or the token
    is already expiring — a tick that is merely late rotates nothing. Every request resolves its own token
    before it goes, so a wake needs no gate.

### Drafts

38. **Nothing typed is ever lost.** Every editing surface persists what is in it and restores it on return.
    The interval is a ceiling, not a debounce: `useAutoSaveDraft` writes at least every
    `AUTO_SAVE_DELAY_MS`, so uninterrupted typing saves rather than waiting for a pause that never comes,
    and `flush()` writes on demand — a close, a "Save draft", a backgrounded tab. Autosave is silent:
    `useSaveDraft` never toasts, because a failure the person cannot act on is noise, and it writes through
    the drafts caches rather than invalidating them every few seconds.
39. **A draft is filed where it can survive the device.** Post, story and short drafts are rows in `drafts` and
    follow the account. Message drafts are local for latency and sealed to the author's own devices for
    continuity. Editor projects hold blobs, so they live in IndexedDB and never travel.
40. **A draft is deleted only by publishing it or by the person.** No dismissal, close, navigation or crash
    deletes a draft — and "publishing it" means the write landed, not that the tap was accepted. A composer
    that drops the draft on the tap leaves a refused publish with nothing to recover from. Leaving any
    editor — the composers, the deck, the story canvas, the shorts editor — writes the draft, closes and
    says so once with `toastDraftKept`, whose Discard is the delete; only a capture no draft can hold is
    asked about before it is lost.
41. **Discard means discard.** A confirmation that says a draft is discarded deletes the stored draft in the same
    action. A dialog whose confirm only closes the sheet is a lie the Drafts list then exposes.
42. **Leaving with unsaved work asks once.** `useUnsavedWarning` guards a real `beforeunload`, and only while the
    surface is genuinely dirty. A publish whose bytes are still going up is unsaved work too, and it outlives
    the composer that started it: `holdPublishInFlight` keeps the guard from the tap until the write settles.
43. **What cannot be persisted is not silently persisted.** A canvas holding only a captured clip skips its save,
    and `stripUnpersistableStickerMedia` drops `blob:` media while keeping remote URLs, so a restored draft never
    points at a dead object URL. The query cache holds to the same rule: `writeQueryCache` strips every
    `isGhostId` row, because a seated row's media and its retry both belong to the tab that seated it, and a
    restored one is a permanently-uploading post with a Retry that does nothing.
44. **A cleared draft is a tombstone, not a hole.** Clearing writes an empty value with a timestamp so a
    synced copy from another device cannot resurrect what the person just deleted.

### Continuity across devices

45. **The account carries state about people; the device carries state about itself.** Read position, unread,
    mute, blocks, theme choice and content drafts follow the account. Recent searches, camera settings, media
    quality, the query cache and editor projects stay on the device.
46. **Signing in adopts the account's look once.** The account's theme lands on a new device on first sign-in;
    after that the device's choice wins and writes back, so changing the theme on a phone does not reach across
    and change the laptop mid-session.
47. **A synced draft merges by time, newest wins**, and a draft too large to seal within
    `DRAFT_CIPHERTEXT_MAX_BYTES` stays on its device rather than falling back to plaintext.
48. **Read position is one marker.** `last_read_at` in `conversation_read_state` is written once per read and is
    what every unread count on every device derives from.
49. **History follows the account, not a device.** Any device a person signs in on reads their message
    history without another device being awake, and a local copy that will not open is rebuilt rather
    than mourned. Nothing about how this works is ever the person's problem to understand.

### Concurrent edits

50. **A write that can be made twice states which one wins.** Today only the message draft does, by timestamp.
    Everything else is last-write-wins with no guard.
51. **A shared row carries a version.** `(not yet built)`. A draft, a profile field or a space setting edited on
    two devices silently clobbers itself. Needs the row's `updated_at` sent with the write and refused on
    mismatch, surfaced as a named refusal rather than a lost edit.
52. **A lost edit is never silent.** When a write is refused for staleness, the person keeps their text, is told
    the item changed elsewhere, and is offered the current version. Never a blank field and never a toast alone.

## What each action does with no network

The contract, action by action. "Fails" means the optimistic state rolls back and one line names the reason.

| Action | With no network |
|---|---|
| Read a page already visited | Paints from the persisted cache, banner shows, no refetch storm |
| Read a page never visited | The route's ghost, then the error state with a retry |
| Cold navigation to the app | The browser's own offline page today `[resilience §6]` |
| Like, bookmark, repost, vote | Applies in the frame, then rolls back and names the network |
| Follow, mute, block | Same: applies, rolls back, names the network |
| Send a message | Stays in the thread with a clock and "will send when you are back online", enters the outbox, retries on reconnect, and says nothing else |
| Send a comment or a reply | Fails and names the network; the text stays in the composer |
| Publish a post | Fails and names the network; the draft is already saved |
| Type anything | Persists locally, restores on return |
| Upload media | Fails on its own timeout; the composer keeps the attachment |
| Open a settings toggle | Applies, rolls back, names the network |
| Play cached media | Plays |
| Join a call, go live | Refused with the reason; never a spinner |
| Hit a crash | The report waits in `STORAGE_KEYS.errorOutbox` and leaves on the next connection, so the failure that happened offline is still one the owner sees |

## Enforced by

- `__tests__/components/shared/offline-banner.test.tsx`: rules 1 and 2.
- `__tests__/domain/what-a-failure-is-allowed-to-assert.test.ts`: rule 4 — which failures may keep their own
  reason, and which are the wire's to name.
- `__tests__/utils/client-error-report.test.ts`: the offline row of the table above. A report generated with
  no network is held and flushed on `online` and on the next load, never dropped.
- `__tests__/utils/app-update.test.ts`: rule 31. The prompt appears once, a stale chunk reloads once per
  window rather than on every crash, and one outside any boundary never reloads.
- `__tests__/pwa/an-offline-navigation-lands-on-our-page.test.ts`: rule 6.
- `__tests__/fitness/nothing-fails-silently.test.ts`: rules 4, 17, 22 and 41. Fails when an action promises something
  it cannot perform or fails with no message.
- `__tests__/messaging/outbox-flush.test.ts`, `__tests__/messaging/outbox-durability.test.ts` and
  `__tests__/stores/failed-messages-store.test.ts`: rules 17, 18, 19, 20 and 21.
- `__tests__/messaging/queued-send-states.test.ts`,
  `__tests__/components/message/an-undelivered-bubble-offers-only-retry.test.tsx` and
  `__tests__/components/message/message-input.test.tsx`: rules 17a, 17b and 17c. Each failure resolves to
  waiting or to refused, an offline attempt costs no try, a refusal is red immediately and survives a revive,
  a connection coming back gives every waiting message its tries back, the bubble says what it waits for, an
  in-flight copy comes back as waiting, and the note names the recipient without disabling the box.
- `__tests__/supabase/session-refresh.test.ts`: rules 33, 34, 35 and 37. A throttled tick with a fresh token
  rotates nothing, a real sleep past expiry refreshes once, concurrent callers share one refresh, a refusal
  is only a sign-out once the server confirms it, and a keepalive is never handed an expired token.
- `__tests__/stores/auth-store.test.ts`: rules 34 and 36. A user restored from storage enables no
  authenticated read, and a write waits for the session to settle instead of refusing.
- `__tests__/app/an-auth-screen-runs-no-session-services.test.ts`: rule 34. No session-only service mounts
  on an auth screen or behind an unconfirmed session.
- `__tests__/fitness/offline-disables-no-control.test.ts`: rule 3. Nothing binds a disabled, read-only or
  aria-disabled state to a connectivity reading.
- `__tests__/fitness/a-mutation-fails-fast-offline.test.ts`: rule 5. The client's one mutation default is
  `always`, and no mutation opts back into the library's pausing.
- `__tests__/factories/a-write-is-never-paused-by-the-library.test.tsx`: rule 5, driven rather than read. A
  `useSupaMutation` that declares no `networkMode` still runs and settles with the browser reporting offline.
- `__tests__/fitness/a-realtime-subscription-backfills.test.ts`: rule 12. The hook re-runs the consumer's own
  callback on foreground and on reconnect, and every consumer's callback reaches a read, a cache key or the
  state its render derives from.
- `__tests__/messaging/realtime-recovery.test.ts` and `__tests__/messaging/notification-realtime.test.tsx`:
  rule 12, driven. A reconnect waiting out its backoff runs at once when the tab or the network comes back,
  and the retry it replaced does not also fire.
- `__tests__/video-calling/a-ring-channel-does-not-report-its-own-teardown.test.tsx`: rule 12. The ring stays
  subscribed while the session revalidates.
- `__tests__/realtime/a-new-socket-forgets-the-last-ones-beat.test.ts`: rule 12, driven through the real
  realtime client. A reconnect after a drop with a heartbeat in flight keeps its socket.
- `__tests__/fitness/one-listener-owns-visibilitychange.test.ts`: rule 13. The owner pairs the event with
  `pagehide` for the departure, and every surface that still binds its own is recorded debt.
- `__tests__/fitness/a-poll-checks-that-it-is-visible.test.ts`: rule 27. An interval whose tick reaches the
  network returns early while the tab is hidden, or is armed only while it is visible.
- `__tests__/realtime/a-hidden-tab-keeps-its-socket.test.ts`: rule 27. The browser client beats its Realtime
  socket from a worker wherever the browser has one.
- `__tests__/realtime/a-worker-that-cannot-start-leaves-the-beat-to-the-page.test.ts`: rule 27, driven through
  the real realtime client. A heartbeat worker that fails to start hands the next socket to the page timer, and
  every page's `worker-src` carries `blob:`.
- `__tests__/hooks/use-debounced-writes.test.ts` and `__tests__/hooks/use-collaborative-thread.test.ts`: rule 23
  for a collaborative thread. Words typed inside the debounce reach the thread on blur, on the background
  signal and on unmount, and a Publish pressed inside it posts them.
- `__tests__/fitness/an-editing-surface-persists-what-is-in-it.test.ts`: rules 38 and 40. Autosave writes on
  a ceiling rather than a debounce, flushes on demand and on the background signal, saves silently through the
  drafts caches, and no dismissal, close or unmount deletes a draft.
- `__tests__/fitness/discard-deletes-the-draft.test.ts`: rule 41. Every discard a draft surface offers reaches
  the delete in the same action, whether it is named on the confirm, the cancel or the second outcome.

**Not yet enforced.**

- `a-shared-row-carries-a-version` (51): every table two devices can write sends `updated_at` with the write.
  Waits on the version guard rule 51 asks for; no write sends one today.

## Traps

- **1** `navigator.onLine` false-negatives on flaky wifi, captive portals and VPN transitions; the Supabase client
  deliberately does not read it at all.
- **4** `navigator.onLine` read at map time answers for the moment of the *catch*, not the moment of the
  request; a 429 or a 42501 caught while that flag was stale used to render "you are offline".
- **5** React Query's default `networkMode: 'online'` pauses a mutation offline, so the button spins and nothing
  ever resolves.
- **5** The default is merged by spread, so passing `networkMode: undefined` through is not "unset" — it
  overwrites `always`, and every write built on the factory opted back into pausing without naming it.
- **12** `postgres_changes` has no replay: everything that changed during a backgrounded or offline gap is simply
  missing until something asks for it again.
- **12** A deterministic realtime topic collides when re-subscribing before the previous channel has finished
  leaving, and the client hands back the dying channel; each subscribe takes a unique topic.
- **12** A subscription keyed to a settled session drops on every revalidation, and every tab return after
  thirty seconds starts one: the ring was keyed to `isReady` and went off the air on each.
- **12** realtime-js keeps a heartbeat the dropped socket never answered and, 25 s after the reconnect, closes
  the new and healthy socket for it; `forgetUnansweredBeatOnReconnect` forgets it on every open.
- **13** One alt-tab return fires both `visibilitychange` and `focus`; without a throttle every subscriber
  refetches twice.
- **13** bfcache leaves `visibilityState` at `visible` on the way out, so a flush bound only to
  `visibilitychange` never runs; pair it with `pagehide`.
- **23** A debounced save loses up to its whole window when the tab goes away; the background signal is the
  flush, not a nice-to-have.
- **25** A game or animation clock that keeps accumulating while hidden applies the whole absence in one frame on
  return.
- **27** Realtime closes a socket that has sent nothing for about 66 seconds, and a hidden tab's timers tick about
  once a minute, so a heartbeat on the page's own timers leaves every hidden tab six seconds from losing its ring.
- **27** realtime-js drops a heartbeat worker that fails to start and starts nothing in its place, so every socket
  then drops about once a minute; `fallBackToPageHeartbeat` hands the next socket back to the page timer.
- **35** A spurious `SIGNED_OUT` with a cached user still present is a refresh race, not a sign-out; treating it
  as one logs people out of a working session.
- **41** A discard confirmation that only closes the sheet leaves the autosaved row alive, and it reappears in
  the Drafts list after the person was told it was gone.
- **44** Deleting a synced draft instead of writing an empty tombstone lets the other device's copy restore it on
  the next merge.
