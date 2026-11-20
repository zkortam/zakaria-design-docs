# Shells

Surf ships on phones as two native shells, Surf and Surf Messenger, and each one loads the web app from its own
origin. This document owns what a person meets where the shell, not the page, is drawing: the splash, the
connectivity state, the system's own sheets and dialogs, the bars and insets around the page, and the moments a
shell is too old for what the page asks of it. The feeling it protects is that the app is one product: nothing
about it reads as a website in a frame, and nothing a shell adds looks or speaks differently from the web it
carries.

## Owns and defers

Owns: what native code may draw, the bridge contract and its floor, kill switches, the launch sequence, the
connectivity and update states, the system-chrome claim, the inset writer, the one Back owner, the shells' version
of a permission ask, and where a shell refuses a surface.

Defers: the share sheet and saving to [platform]; the pre-permission card's design to [permissions-and-trust];
history, Back and deep links to [navigation-and-history]; offline, return and drafts to [resilience]; first paint,
install and sign-in to [arrival]; safe areas and the reading column to [layout]; haptics to [feedback].

## Rules

### Capabilities, never screens

1. **The web app is the app.** Every screen, string, rule and flow lives in `app/`, `components/` and `lib/`; a
   shell supplies capabilities. A component never knows what it runs in: it asks `can()` and calls
   `capabilities()`, and `lib/platform/` is the only place a platform is named.
2. **Native UI is only the OS's own surfaces.** The lock screen, a notification, the share sheet, the system call
   screen, the status bar, the store purchase sheet and a system permission dialog. Anything Surf designs is drawn
   by the web, including every state below; the one exception is the native connectivity and update plate, drawn
   before the origin can load.
3. **One ring, one owner.** Where the OS owns the ring (`can('calls', 'start')`), the web's incoming-call overlay
   never renders, in the foreground too: a ring the page hears goes to `capabilities().calls.ring`, named by the
   caller's @handle. Every capture opens through `lib/platform/capture.ts`, which refuses with the `error.inACall`
   fault while the engine's `mediaBusy` holds, so each capture surface shows its "in a call" state.
4. **A surface a shell refuses says so and offers the way out.** Staff tools in a shell render one plate,
   `OpenAdminInBrowser`, whose single action opens the same page in the system browser through
   `capabilities().links.open`. Stripe's embedded payout components cannot run in a webview, so in a shell
   `HostedPayoutSetup` continues setup on Stripe's hosted pages (`connect.openPayoutOnboarding`, back by the
   `?connect=` app link) and sends a live account to the browser to manage. A hidden entry for something that
   exists is never the answer.

### The contract and its floor

5. **A seam is an interface, three arms, a manifest entry and a test.** The web arm lives in
   `lib/platform/adapters/web/`, the native arms are the bridged calls in `lib/platform/adapters/native.ts` and
   their Swift and Kotlin units in `native/`. A method enters `BRIDGE_MANIFEST` only in the commit that finishes all
   three, and `can()` stays false for it until then.
6. **The contract only grows.** An entry is never edited or removed; it is retired with `deprecatedIn` then
   `removedIn`, and `native/contract/bridge-history.json` records every entry ever registered.
7. **Below the floor, the shell keeps working and offers the update.** Between `BLOCKED_BELOW` and
   `MIN_BRIDGE_VERSION`, a feature whose method the shell lacks (`capabilityReach` answers `update`) shows
   `UpdateTheApp`, one sentence and `OpenStoreButton`, never a hidden entry. Below `BLOCKED_BELOW` the native plate
   shows `native.update.required` with the one button to the store listing, decided from the last persisted
   `/api/shell` response before the origin loads; the server serves `/update-required` for any document that still
   arrives, answers 426 on the native routes, and sends that install no push.
8. **A kill switch only takes away.** `/api/shell` answers the compatibility lines, scoped `disabled` entries,
   cost ceilings and the one `notice`; `can()` falls back to the web arm for a switched-off method. Before any
   response has been seen nothing is off; after one, the last good response holds.

### Generated, never typed

9. **A product fact in a native project is generated.** Hosts, bundle ids, strings, permission purposes, the bridge
   contract and the launch timings come from the TypeScript that owns them through `npm run native:generate`. No
   string a person reads is typed in Swift or Kotlin; it arrives from a `native.*` or `permission.purpose.*`
   locale key.
10. **Divergence is data.** What legitimately differs by runtime is a row in a table (`BRIDGE_MANIFEST`,
    `SHELL_GATES`, the purchase policy), never a branch on a platform name.

### Launch and connectivity

11. **The splash is held to the page, never past it.** The native splash wears the theme the person last saw and
    lifts on `bridgeReady`, which `signalReady` sends once the page has presented a frame, beside hydration and not
    after it, with a ceiling of `SHELL_LAUNCH.splashCeilingMs` from first paint on both platforms; both the splash
    and the connectivity state fade over `SHELL_LAUNCH.fadeMs`. Both shells open on `shellStartPath`, the route
    the signed-in person lands on, so a launch never starts with the `/` redirect. The shell fetches `/api/shell`
    at launch and on every return and hands the answer over as `shell.config`; the page only polls for a page left
    open.
12. **No paint is a designed state.** With no first paint by `SHELL_LAUNCH.firstPaintDeadlineMs`, the
    connectivity state takes over: the plate, the app mark and one line, `native.connectivity.unreachable`. A load
    still in flight is left to finish, since a new one would cancel it and lose what was typed; a load that fails
    retries on `SHELL_LAUNCH.retryBackoffMs` and at once when the network returns. A WebView below the feature
    floor shows `native.webview.outdated` with `native.webview.update` instead.

### The system around the page

13. **One claim at a time owns the bars.** `useSystemChrome(claim)` is the only way a surface claims the status
    bar, the home indicator, the deferred edges or the orientation; the theme writer in `lib/theme/chrome.ts` sets
    the default claim, the topmost mounted claim wins field by field, and unmounting restores what was under it.
    A system switch between light and dark reaches the page as `system.colorScheme`, and a change to Reduce
    Transparency, Increase Contrast, Bold Text or the text scale as `system.accessibility`, without a reload. The
    surfaces' claims sit beside the hook: `MEDIA_CHROME` and `mediaChrome` for the viewers, light on media and
    hidden with the viewer's own controls; `captureChrome` for every capture surface, upright on a phone with the
    bottom edge deferred while recording; `callChrome`; `IMMERSIVE_PLAY_CHROME`; and `GESTURE_SURFACE_CHROME`,
    which takes Back's edge for the editor timeline, the canvas editor and the map. Only a screen whose shortest
    side is under `PHONE_SHORTEST_SIDE_MAX` takes an orientation lock, and the top edge is never deferred, so
    Notification Center and Control Center always work.
14. **Android's insets have one writer.** The shell's injector measures bars, the cutout, the keyboard and the
    gesture edges, and `lib/platform/insets.ts` writes them as `--safe-top`, `--safe-right`, `--inset-bottom`,
    `--safe-left` and the `--gesture-left`, `--gesture-right` and `--gesture-bottom` edges, in one frame per change;
    the names are `INSET_PROPERTIES`, generated into the injector. The WebView's container follows the keyboard
    frame by frame while it slides, and the shell links no Capacitor Keyboard plugin on either platform: its
    insets listener and `DISPATCH_MODE_STOP` callback on Android, and its removal of WKWebView's own keyboard
    observers on iOS, sat in front of the shell's `SurfKeyboard`, so the system's own accessory bar stays on iOS.
    It reads `systemBars | displayCutout` and writes
    each edge rounded up to CSS pixels, the bottom and the bottom gesture edge zeroed while the keyboard is up, the
    side gesture edges from `systemGestures`, and it passes the bars down unconsumed, so `env()` agrees from
    WebView 144 and nothing pads twice. iOS writes none: WebKit's `env()` is live there.
15. **A configuration change never reloads the page.** The Android activity declares every configuration change
    it handles itself, so turning, folding, resizing or a keyboard arriving keeps the page, its scroll and
    whatever is typed.
16. **Back has one owner.** On Android the shell's callback is enabled only while there is in-app history or an
    open overlay, and then it is `history.back()`; with nothing behind, the system plays its own back-to-home. A
    surface never listens for the hardware key. While it is enabled, the system's predictive gesture reaches the
    page as `back.progress` and `back.cancel`, and `useSwipeBack` draws the pop under the finger, unless a mounted
    claim sets `backSwipe: false` (`backSwipeAllowed`), which is also how a surface that owns the horizontal
    gesture takes it from the web's swipe-back.
17. **The chooser and the camera are the shell's.** A file input in a shell opens the system chooser, and a
    capture input the system camera, each wired by the shell to the input that asked; the page never draws a
    chooser of its own. They hand over originals `[media §59]`: Android's camera writes to `EXTRA_OUTPUT` at
    `EXTRA_VIDEO_QUALITY` 1, and the app declares the formats it reads (`ORIGINAL_VIDEO_FORMATS`) so the Photo
    Picker does not transcode them.

### Permissions in a shell

18. **Every pre-permission card has exactly one neutral button.** In a shell the card's only action is "Continue",
    which raises the system dialog; declining happens there, and the card closes only by the sheet's own
    dismissal. The reason a dialog shows is the same locale key the generator writes into the native project.

### Native pixels in the page

19. **Native pixels show only through a `NativeSurfaceSlot`.** A call's video is drawn by the shell behind the
    transparent page at the rect `components/ui/native-surface-slot.tsx` holds, in visual-viewport CSS px, measured
    once a frame by `lib/platform/native-surface-slots.ts` and sent only when it moved. While a slot is placed,
    `html[data-native-surface]` hides everything but the slots' host (`data-native-surface-host`), makes the path
    down to it see-through, and the shell paints a black backdrop behind its layers; with none, the page is opaque.

## Enforced by

- `__tests__/fitness/every-capability-has-an-adapter-per-shell.test.ts`: 5, every capability's web arm and every
  registered method's Swift and Kotlin arm.
- `__tests__/components/monetization/hosted-payout-setup.test.tsx`: 4, for payouts.
- `__tests__/fitness/a-bridge-method-bumps-the-contract.test.ts`: 6 and 7, entries recorded, never edited or
  dropped, retired in order, landed above every released shell, and a floor moved only after its soak.
- `__tests__/fitness/boot-shares-the-ua-regex.test.ts`: 1, the boot script's copy of the one user-agent pattern.
- `__tests__/fitness/every-messenger-shell-route-is-served.test.ts`: 4, for the Messenger host's half: every page
  its shell loads is served there.
- `eslint-rules/no-raw-platform-check.mjs`: 1, at zero, for a browser API, a user-agent test, a dialog, a download
  anchor or a `@capacitor/` import outside `lib/platform/`.
- `__tests__/platform/the-core-is-server-safe.test.ts`: 1, a server component can read the platform from the
  request without importing a hook.
- `e2e/shell.spec.ts`: 4, 8 and 14, in the `shell` project against the bridge shim.
- `__tests__/fitness/a-system-chrome-claim-is-the-one-door.test.ts`: 13, the seam has one caller and the stack
  one default.
- `__tests__/fitness/a-permission-card-has-one-neutral-button.test.ts` and
  `__tests__/components/notification/notification-permission-prompt.test.tsx`: 18.
- `__tests__/platform/one-ring-one-owner.test.tsx`: 3, a ring the page hears handed to the shell by @handle.
- `__tests__/platform/native-calls.test.tsx`: 3 and 19, capture refused in a call, a slot published in
  visual-viewport px and the surface cleared with the last slot.

- `__tests__/fitness/a-missing-method-offers-the-update.test.ts` and
  `__tests__/components/shared/update-the-app.test.tsx`: 7, the update offer between the lines.
- `__tests__/fitness/a-blocked-shell-cannot-reach-the-bridge.test.ts` and
  `native/android/app/src/test/java/org/surfsocial/shell/UpdateGateTest.kt`: 7, below `BLOCKED_BELOW`.

**Not yet enforced**

- `a-shell-string-comes-from-a-locale` (9): no literal string in Swift or Kotlin.

## Traps

- 3: Two ring UIs at once, the web overlay under the system call screen, reads as two calls and answers neither.
- 11: A splash that waits on hydration and a second chunk holds the person on a plate while a painted page sits
  under it; the signal is the frame, not React.
- 14: Both Android's injector and Capacitor's own bar handling writing insets pads the page twice on one WebView
  version and not at all on the next.
- 15: An Android activity without the configuration list restarts on rotation and drops a half-typed message.
- 19: Native video sits behind the page, so any painted ancestor of a slot, or any page content under the call
  overlay, covers it: the call is live and the tile reads as a blank hole.
