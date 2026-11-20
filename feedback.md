# Feedback

This document owns everything the product says back: the press under the finger, the buzz, the sound,
the toast, the badge, the choice between undoing a thing and asking about it first, and the order in
which two things that both want the person's attention are allowed to have it. The feeling it protects
is that the product answers. Every tap is acknowledged in its own frame, every completed action says so
once and briefly, every mistake is walked back rather than guarded against, and nothing ever arrives
loudly for something the person did not do.

## Owns and defers

Owns: the press ladder; the haptic vocabulary and the moments that may fire one; the sound catalogue,
when each plays and how mute intent is honoured; toasts as a surface; badges and unread; the undo
versus confirm taxonomy; the interruption priority order.

Defers: press, toast and badge animation to `[motion]`; toast, badge and confirmation wording to
`[copy-and-voice]`; the z-index rungs the priority order rides on to `[tokens]`; the `Dialog`,
`ConfirmDialog` and `Toast` shapes to `[components]`; optimistic mutation and rollback timing to
`[speed]`; permission prompts to `[permissions-and-trust]`; playback audio to `[media]`; live
announcements of the same events to `[accessibility]`.

## Rules

### Press

1. **One press ladder, three rungs.** `press-subtle` for a wide surface, `press` for a
   compact control, `press-firm` for a small glyph. Nothing writes its own `active:scale-*`, no
   `whileTap`, and `group-press-*` names a class Tailwind never generates.

2. **The press answers the control, not the container.** Each utility carries
   `:active:not(:has(:is(button, [role='button'], [role='link'], a):active))`, so a card holding a
   pressed button stays still.

3. **The press is visible on iOS.** `-webkit-tap-highlight-color` is suppressed and WebKit gates
   `:active` behind a touch listener, so the primitives (`Button`, `IconButton`, `UnstyledButton`,
   `Pressable`) carry the press themselves rather than leaving it to a call site's class.

4. **Press is instant down, eased up.** `:is(button, [role='button'], [role='link'], a):active` sets
   `transition-duration: 0s`, so the press lands in the frame the finger does and only the release eases
   back through `press-base`.

5. **Reduced motion keeps the feedback.** `.reduce-motion` swaps the scale for `--press-opacity`.
   A press that disappears under Reduce Motion reads as a dead control, because there is no hover on
   touch and no tap highlight either.

### Haptics

6. **`haptic()` is the only vibration.** `lib/utils/haptics.ts` holds the six styles and their patterns.
   No call site calls `navigator.vibrate`. It answers inside the gesture, before the render it accompanies:
   a shell's native seam is loaded before the first tap, so no buzz waits behind an `import()`.

7. **A haptic answers a physical moment, never a plain tap.** A button's press state `[motion]` is its
   whole answer. No primitive vibrates on click, and no call site adds a `haptic('light')` to a tap: today
   `withHaptic` fires `light` inside `Button`, `IconButton`, `UnstyledButton` and `Pressable` (not yet built)
   and it leaves them, with every redundant call beside it.

8. **A rung names a moment.** Only a moment on the vocabulary below fires a haptic, and the moment fires it,
   not the button that started it: a switch landing, a selection detent in a strip or picker, a swipe or pull
   crossing its threshold, a recording starting or stopping, a send landing, a long press opening a menu, and
   the finish or refusal of work the person waited on. `COALESCE_WINDOW_MS` (60ms) makes an accidental double
   harmless, never acceptable. `Switch` and `Checkbox` wear the `press` class and buzz through `toggleActivation`, so a
   flip answers in its own frame; a disabled one does neither.

9. **Nothing buzzes that the person did not cause.** No haptic on hover, on a scroll, on an arriving
   message, on a poll result, on a realtime event, or inside a loop. A vibration is a reply, never an
   announcement.

10. **Haptics are a device preference.** `getLocalSettings().haptics` gates every call, because the same
    account roams between phones that differ. `startRingVibration` is the only repeating pattern and it
    belongs to one incoming call.

### Sound

11. **Every sound is in one catalogue.** `lib/constants/notification-sounds.ts` names every file, its
    label key and the feature that unlocks it. Nothing constructs an audio path.

12. **A sound plays only for something that arrived while the person was elsewhere.** A notification, a
    message, an incoming call. No sound for a tap, a send, a like, a page, a success or an error.

13. **One thing sounds at a time.** `claimAudioOutput` in `lib/audio/exclusive-playback.ts` is the single
    holder; an arriving player stops the one holding it. A notification chirp, a ringtone and the theme
    ambience each claim it too, so an alert never lands on top of a voice note.

14. **Mute intent is read at play, not at save.** `playNotificationSound` resolves the account's stored
    choice and falls back to the default when the tier that unlocked it has lapsed, so a downgraded
    account stops sounding without its row being rewritten. `none` is a real value that means silence.

15. **A burst is one chirp.** `NOTIFICATION_SOUND_MIN_GAP_MS` (1500ms) coalesces a flurry of arrivals
    into a single sound at `NOTIFICATION_SOUND_VOLUME` (0.5).

16. **A preview runs off a tap.** `previewNotificationSound` and `previewRingtone` are unthrottled
    because they run inside a real gesture, where the autoplay policy always allows them. Any other
    play swallows its rejection rather than throwing.

17. **Ambience yields.** The theme's ambient loop suspends whenever any other element starts playing
    and resumes when it stops, and it never starts until the person has answered its prompt.

17a. **A sound takes the audio session only while it sounds.** `holdAudioSession` in
    `lib/audio/audio-session-policy.ts` is the one writer of the page's session: a chime, a ringtone and
    game audio hold `ambient` (they mix under the person's music and obey the silent switch), a music
    preview holds `playback`, and with no hold the page is back on `auto`. A Web Audio context resumes
    from `interrupted` as well as `suspended` (`isResumable`), or it stays silent after a phone call.

### Toasts

18. **One voice, docked where the thumb is.** Every toast rises from one dock: above the tab bar wherever
    the bar exists (`FLOATING_BAR_BOTTOM`, in compact) and bottom-end beside the rail, so nothing it says covers the
    bar or the header at any width. It sits at `Z_INDEX.toast` (600), above every takeover, below the gate.
    A state that stands rather than passes stands in the same dock beneath the toasts: `OfflineBanner`, through
    `ToastProvider`'s `standing`, on a solid ground `[resilience §2]`. A row a person works in where the dock
    would land registers with `useToastClearance`, and the dock rises above it, so an undo offer never covers
    the controls it undoes: the composer's tool row and every `AiPrompt` do. The row is measured where it is when a toast
    arrives and while one shows (`useToastLift`), never only where it was as it slid in with its sheet.

19. **At most three are on screen.** `MAX_VISIBLE_TOASTS` (3) bounds the stack; the countdown lives in
    the container, not the row, so a queued toast keeps its timer instead of resurfacing with a fresh
    one.

20. **One toast per outcome.** `addToast` drops a duplicate with the same title, description and
    variant. A toast carrying an action is never deduped: two deletions inside five seconds are two
    separate offers to undo.

21. **Two durations, both named.** `TOAST_DURATION_MS` (4s) for an outcome and `TIMING.undo` (5s) for an
    undo offer. A new build is not a toast: `promptAppUpdate` raises a standing offer, a dot on the account
    and an "Update Surf" row in its menu (`UpdateDot`, `UpdateSurfMenuItem`, `UpdateSurfRow`), which waits
    for as long as the person does.

22. **An undo is one line, and its window is drawn.** `undoableToast` renders "Post deleted" and Undo, parted by
    space: no variant icon, no description, no close. The window is the capsule's own fill draining over its duration
    (`toast-drain`, a transform), so there is no number to read. Nothing builds an undo toast by hand;
    `outcomeToast` is for an outcome whose inverse only sometimes exists. An actionable toast pauses, fill
    and all, only under `MEDIA_QUERIES.hoverCapable`, because a touch device may never fire `pointerleave`.
    A change applied from a panel scrolls its row clear of the dock as the undo rises, so the capsule never
    covers what it undoes (`FeedRow` in `lib/utils/feed/feed-proposal.ts`).

23. **A toast never steals focus, and leaves the way it is sent.** `role="status"`, or `role="alert"` for an
    error or a warning, never focused on arrival. A message toast closes on Dismiss; an undo is swiped away
    along the inline axis and leaves along the swipe. One that expires sinks back into the dock on a phone
    and slides off the inline end on a desk.

24. **A rollback always toasts.** An optimistic mutation that the server refuses restores its snapshot
    and names what refused. A silent revert is the defect this rule exists to stop.

24a. **A row is the confirmation only where the person can see it.** A cache accepting an optimistic row
    says a list holds it, never that anyone is looking at it: `isFeedHeadInView` decides, and a publish the
    author is scrolled away from confirms with a toast carrying the way to it. A tier with no undo window
    `[§35]` is the case where nothing else would say anything at all.

### Badges and unread

25. **A dot means "something", a badge means "how many".** `UnreadDot` marks a row that holds unread
    content; `UnreadBadge` carries a count on a navigation destination, capped at `9+`, on
    `COUNT_BADGE` at the `text-badge` size.

26. **Every surface counts the same thing.** The messages badge and the notifications badge each come
    from one hook (`useUnreadMessageCount`, `useUnreadCount`) and every place that shows them renders
    the same number. A second count of the same thing is a bug.

27. **A count clears at one moment.** Reading the thing clears it, everywhere at once, driven by the
    read marker rather than by the surface that displayed it. Opening the list does not clear it;
    reading the row does.

28. **A badge waits a commit.** `useAfterFirstCommit` gates the badges in the chrome, because the
    persisted cache holds a count the server render did not, and a badge that appears and corrects
    itself reads as a phantom.

29. **The tab title, the favicon and the app badge carry the same number** as the tab bar, and clear
    with it. `useAppIconBadge` owns the app badge on every runtime through `capabilities().badge` (the installed
    web app's `setAppBadge`, the iOS icon, Android's launcher count) and re-applies it on every return to the
    foreground; it writes nothing until both counts are known, so a loading count never reads as zero. `(not yet
    built)`: no code writes `document.title`, and no favicon is generated per count.

### Undo and confirm

30. **Reversible acts now.** An action whose inverse exists performs immediately and offers `undoableToast`
    for `TIMING.undo` (5s). The toast is the affordance, so its duration is the window.

31. **Confirm is for the irreversible only.** `ConfirmDialog` asks once, names the thing, and there is no
    second confirmation. Its Cancel is its one way out: the header carries no close glyph beside it
    (`DialogCompactHeader footerDismisses`). Irreversible means the inverse does not exist, the thing leaves this account's
    reach, money moves, a secret is shown for the last time, or other people already saw it.

32. **Never both.** An action that offers an undo does not also ask first, and an action that asks first
    does not pretend to be undoable in its toast.

33. **The undo is a real inverse, not a refetch.** `useUndoableDeletion` calls the restore RPC and
    invalidates, and a failed undo says so. Nothing promises "tap undo to restore it" without a restore. An act
    with no inverse to call (declining a request, erasing search history, deleting a draft) is held instead:
    `holdForUndo` shows the offer and the write is made only when its window closes; `useHeldRemoval` hides
    the row meanwhile. A tab closed inside the window makes no write, so the draft survives rather than being lost.

34. **A destructive act toasts what it did.** After the window closes the record is gone for good, so the
    purge is armed by success and disarmed by the undo, never by a timer that ran anyway.

35. **An undo window may be longer, never shorter.** The publish undo runs for
    `TIER_LIMITS[tier].undoPostSeconds` (0, 30, 60 or 120), and its toast lives exactly that long. A tier
    that grants no window offers no toast rather than an expired one.

### Interruption

36. **Nothing interrupts typing or a drag.** No overlay opens, no focus moves and no list re-orders while
    a field has focus or a finger is down. An arrival waits.

37. **One thing arrives at a time.** A burst and a toast, or a cascade and a dialog, never run at once.

38. **The order is fixed.** Higher on the ladder below may cover lower; lower never covers higher, and
    nothing the app renders covers the gate (`nothing-outranks-the-gate`).

39. **Only a call takes the screen.** Everything else that arrives is a toast or a pill. A live stream
    starting, a mention, a new follower and a finished export are all toasts.

40. **A system prompt follows a tap.** The browser's own permission and share dialogs are only ever
    raised by a control the person pressed, never by an arrival. `[permissions-and-trust]` owns when.

## The haptic vocabulary

| Style | Pattern (ms) | The moments that may fire it |
|---|---|---|
| `light` | 10 | A switch landing. A selection changing within a strip, a grid or a picker. Listening beginning and ending on the AI edge. Never a plain tap. |
| `medium` | 20 | A live activity capsule expanding on long press, a lifted row committing to its gap. A physical threshold or commit: a swipe crossing its action threshold, pull-to-refresh committing, a long press opening a menu or the account switcher, a short paging, a recording starting or stopping, a call accepted or declined, an element dropped on the trash. |
| `heavy` | 30 | Reserved. Nothing fires it today. |
| `success` | 10 · 50 · 10 | A live activity completing and its capsule collapsing. Work the person waited on finished: an export, a generation, a transcription, a liveness step landing. |
| `warning` | 20 · 40 · 20 | Work finished with less than it was asked for. |
| `error` | 30 · 30 · 30 · 30 | Work the person waited on refused. Never a request that merely returned an error. |
| ring | 400 · 200 · 400 · 1200, repeating | An incoming call, through `startRingVibration`, stopped on accept, decline and unmount. |

## The sound catalogue

| Sound | File | When it plays |
|---|---|---|
| Default alert | `/sounds/notification.mp3` | A notification arrives (`playNotificationSound`) |
| Chime, Pop, Bell, Subtle | `/sounds/chime.mp3`, `pop.mp3`, `bell.mp3`, `subtle.mp3` | The same moment, when chosen |
| Aurora, Ripple, Lantern, Signal, Dusk | `/sounds/premium/*.mp3` | The same moment, when chosen and the tier still grants `premium_notification_sounds` |
| None | `null` | Silence, as a real choice |
| Ringtone, default | synthesised (`lib/audio/call-tones.ts`) | An incoming call, repeating at `RINGTONE_PERIOD_MS` |
| Ringtone, chosen | `chime`, `bell`, `subtle`, `aurora`, `ripple` | The same moment; the synthesised tone is the fallback whenever a sample cannot sound |
| Ringback | synthesised, 440 and 480Hz | The caller's own line while the far side rings |
| Theme ambience | `/themes/dream-os.mp3` | A theme that declares one, once the person has said yes, suspended whenever anything else plays |

## The interruption order

| Rank | Surface | Rung | May cover |
|---|---|---|---|
| 1 | The app-switcher privacy blur | `privacyOverlay` 99999 | Everything, including the gate |
| 2 | The parental lock and the wellbeing limit | `gate` 700 | Everything the app renders |
| 3 | A toast, including an undo offer | `toast` 600 | Every takeover, so a refusal always reaches the person |
| 4 | A tooltip | `tooltip` 550 | Fine pointers only |
| 5 | A media viewer | `viewer` 520 | The sheet it opened from |
| 6 | A menu, popover or sheet | `floating` 500 | A takeover it was opened on |
| 7 | The live stage | `liveStage` 450 | The page and its dialogs |
| 8 | An incoming call | `modal` 400 | The page. The only arrival that takes the screen |
| 9 | The floating player, and the live activity capsule just above it | `livePip` 360 | Page content only |
| 10 | The new-posts pill | `inPageChrome` 50 | Nothing. It waits in the page for a tap |

## Enforced by

- `__tests__/fitness/a-press-reads-one-ladder.test.ts` covers 1.
- `__tests__/fitness/a-haptic-names-its-moment.test.ts` covers 6, 7 and 9.
- `__tests__/fitness/a-sound-claims-the-speaker.test.ts` covers 11 and 13.
- `__tests__/audio/audio-session-policy.test.ts` covers 17a.
- `__tests__/fitness/a-toast-carries-a-named-duration.test.ts` covers 20 and 21.
- `__tests__/components/ui/toast.test.tsx` covers 18, 19, 22 and 23.
- `__tests__/components/shared/update-offer.test.tsx` covers 21.
- `__tests__/fitness/nothing-fails-silently.test.ts` covers 24 and 33.
- `__tests__/fitness/an-unread-count-has-one-source.test.ts` covers 26 and 27.
- `__tests__/fitness/a-reversible-act-offers-undo.test.ts` covers 30, 31 and 32.
- `__tests__/fitness/nothing-arrives-while-you-type.test.ts` covers 36.
- `__tests__/fitness/nothing-outranks-the-gate.test.ts` covers 38.

**Not yet enforced**

- 2, 3: `a-press-is-visible-on-a-phone` opens the reference pages on a real device and asserts a pressed
  control changes and its card does not. Only a phone can prove it; CDP touch events never fire `:active`,
  and no source-level read can stand in for a rendered frame under a finger.

## Traps

- 3: `-webkit-tap-highlight-color: transparent` plus WebKit gating `:active` on a touch listener means
  zero press feedback on iOS while every `press` class looks complete on a desktop screenshot.
- 4: a later `transition-*` utility in `@layer utilities` replaces `transition-property` wholesale and
  drops `transform`, which is why the press rule is unlayered.
- 5: the blanket reduce-motion `transition-duration: 0.01ms` removes the travel, so the substitute has to
  be a property that still reads at zero duration.
- 6: iOS has no Vibration API. Clicking a hidden `<input switch>` inside the tap's own gesture is the one
  thing WebKit answers with a haptic, which is what `tapIosSwitch` does.
- 19: only the newest `MAX_VISIBLE_TOASTS` are mounted, so a timer owned by the row would leave a queued
  toast with no countdown and a full duration once the newer ones cleared.
- 20: deduping actionable toasts meant a second deletion inside five seconds offered no undo at all.
- 22: a toast pushed out of the visible window receives no `pointerleave`, so unmounting is what resumes a
  paused countdown.
- 28: the persisted query cache answers before the first fetch, so a badge rendered on the server paint
  shows a number from the last session.
- 34: the media purge is armed on the delete's success and cleared by the undo. Armed on the tap instead,
  an undo restores a row whose media is already gone.
